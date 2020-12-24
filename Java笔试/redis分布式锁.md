## 分布式锁几大特性

- `互斥性`。在任意时刻，只有一个客户端能持有锁，也叫唯一性。
- `不会发生死锁`。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
- `解铃还须系铃人`。加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了，即不能误解锁。
- `锁不能自己失效`。正常执行程序过程中，锁不能因为某些原因失效。
- `具有容错性`。只要大多数Redis节点正常运行，客户端就能够获取和释放锁。

下面我们举一些实现方式，逐步理解这几大特性。

## 第一种实现方式（初级）

```java
public void wrongRedisLock(Jedis jedis, String lockKey, int expireTime) {
    // 过期时间
    long expires = System.currentTimeMillis() + expireTime;
    String expiresStr = String.valueOf(expires);

    if (jedis.setnx(lockKey, expiresStr) == 1) {
        // 开始执行代码逻辑
    }
}
```

**互斥性**

首先这里使用的是`setnx`这个命令，这个命令的特点就是，如果要设置的key不存在，那么我就可以设置成功。如果key存在，我就设置失败。

这样的特点会保证Redis里只有一个唯一的key，一群客户端同时去设置key时，也只有一个人能设置成功。

因为这个特性，他保证了第一个特性：`互斥性`。

**不会发生死锁**

他这里设置了过期时间，即使客户端宕机的时候，锁也会自动被释放，因为过期时间一到，key就会被自动删除了。

因为这个特性，他保证了第二个特性：`不会发生死锁`。

除了以上两个特性满足外，其他三个特性都没有满足。

## 第二种实现方式（中级）

### 加锁实现

```java
/**
 * 获取分布式锁(加锁代码)
 * @param jedis Redis客户端
 * @param lockKey 锁
 * @param requestId 请求标识
 * @param expireTime 超期时间
 * @return 是否获取成功
 */
public static boolean getDistributedLock(Jedis jedis, String lockKey, String requestId, int expireTime) {

    String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);

    if (LOCK_SUCCESS.equals(result)) {
        return true;
    }
    return false;
}
```

**解铃还须系铃人**

这个加锁与第一种不同之处在于：设置了value值。（value要是唯一能代表客户端的标识）

这个value代表是哪个客户端加的锁，当解锁的时候就需要对比value，看是不是这个客户端加的锁。如果是才能解锁成功，否则解锁失败。

他保证了第三个特性：`解铃还须系铃人`。

### 解锁实现

```java
/**
 * 释放分布式锁(解锁代码)
 * @param jedis Redis客户端
 * @param lockKey 锁
 * @param requestId 请求标识
 * @return 是否释放成功
 */
public static boolean releaseDistributedLock(Jedis jedis, String lockKey, String requestId) {

    String script = "if " +
                        "redis.call('get', KEYS[1]) == ARGV[1]" +
                        "then"+
                        "return redis.call('del', KEYS[1])" +
                    "else" +
                        "return 0 end";

    Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));

    if (RELEASE_SUCCESS.equals(result)) {
        return true;
    }
    return false;
}
```

这里使用的是`Lua脚本`，为什么要使用这个脚本呢？

大家看上面的解锁操作，正常情况下，if/else的解锁操作不是原子性的， 存在并发安全问题。

那么在Redis里执行Lua脚本，能保证这些操作是原子性的，不存在并发安全问题，这就是Lua脚本的作用。

带大家解读以上Lua脚本的意思。

`redis.call`是调用Redis的API方法，这里是调用的get和delete方法，key是`KEYS[1]`这个参数，它相当于一个占位符表达式，真实赋值是方法外传进来的lockKey，下面的`ARGV[1]`也是同理。

`整个Lua脚本连起来的意思就是`：如果通过lockKey获取到的value值等于方法外传进来的值requestId，那么就删除掉lockKey，否则返回0.

这个第二种方式，保证了`互斥性`、`不会发生死锁`、`解铃还须系铃人`，可以说满足了大部分场景的需求，那么第四点和第五点还是没有满足，我们下面继续来看。

## 第三种实现方式（高级）

### Redisson

Redisson是一个在Redis的基础上实现的Java驻内存数据网格（In-Memory Data Grid）。

它不仅提供了一系列的分布式的Java常用对象，还实现了可重入锁（Reentrant Lock）、公平锁（Fair Lock、联锁（MultiLock）、 红锁（RedLock）、 读写锁（ReadWriteLock）等，还提供了许多分布式服务。

Redisson提供了使用Redis的最简单和最便捷的方法。Redisson的宗旨是促进使用者对Redis的关注分离（Separation of Concern），从而让使用者能够将精力更集中地放在处理业务逻辑上。

用法举例

```java
public void testReentrantLock(RedissonClient redisson){

  RLock lock = redisson.getLock("anyLock");
  try{
      // 1. 最常见的使用方法
      //lock.lock();

      // 2. 支持过期解锁功能,10秒钟以后自动解锁, 无需调用unlock方法手动解锁
      //lock.lock(10, TimeUnit.SECONDS);

      // 3. 尝试加锁，最多等待3秒，上锁以后10秒自动解锁
      boolean res = lock.tryLock(3, 10, TimeUnit.SECONDS);
      if(res){    //成功
          // do your business

      }
  } catch (InterruptedException e) {
      e.printStackTrace();
  } finally {
      lock.unlock();
  }
}
```

大家可以看到和我们的ReentrantLock用法上类似，我们来读读他的源码吧。

### 重点方法罗列

```java
public class RedissonLock {
    //----------------------Lock接口方法-----------------------

    /**
     * 加锁 锁的有效期默认30秒
     */
    void lock();
    /**
     * tryLock()方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true，如果获取失败（即锁已被其他线程获取），则返回false.
     */
    boolean tryLock();
    /**
     * tryLock(long time, TimeUnit unit)方法和tryLock()方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，
     * 在时间期限之内如果还拿不到锁，就返回false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。
     *
     * @param time 等待时间
     * @param unit 时间单位 小时、分、秒、毫秒等
     */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    /**
     * 解锁
     */
    void unlock();
    /**
     * 中断锁 表示该锁可以被中断 假如A和B同时调这个方法，A获取锁，B为获取锁，那么B线程可以通过
     * Thread.currentThread().interrupt(); 方法真正中断该线程
     */
    void lockInterruptibly();

    /**
     * 加锁 上面是默认30秒这里可以手动设置锁的有效时间
     *
     * @param leaseTime 锁有效时间
     * @param unit      时间单位 小时、分、秒、毫秒等
     */
    void lock(long leaseTime, TimeUnit unit);
    /**
     * 这里比上面多一个参数，多添加一个锁的有效时间
     *
     * @param waitTime  等待时间
     * @param leaseTime 锁有效时间
     * @param unit      时间单位 小时、分、秒、毫秒等
     */
    boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException;
    /**
     * 检验该锁是否被线程使用，如果被使用返回True
     */
    boolean isLocked();
    /**
     * 检查当前线程是否获得此锁（这个和上面的区别就是该方法可以判断是否当前线程获得此锁，而不是此锁是否被线程占有）
     * 这个比上面那个实用
     */
    boolean isHeldByCurrentThread();
    /**
     * 中断锁 和上面中断锁差不多，只是这里如果获得锁成功,添加锁的有效时间
     * @param leaseTime  锁有效时间
     * @param unit       时间单位 小时、分、秒、毫秒等
     */
    void lockInterruptibly(long leaseTime, TimeUnit unit);  
}
```

下面我们讲其中一种加锁方式：tryLock，其余的大家可以自己看看，原理都差不多。

### tryLock加锁源码解读

大家先看看加锁流程图

![img](https://mmbiz.qpic.cn/mmbiz_png/DB0IfcV7dwibvAgxm5FIb9rVibVog6lU7nBypXd9iczI29Vo0tGmgIaSXh42VmeA39WLYr8JicxWcwuQUWJ3rvx0xA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 整个代码主流程

代码都做了注释，大家可以跟着注释阅读源码

```java
@Override
public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
    //取得最大等待时间
    long time = unit.toMillis(waitTime);
    //记录下当前时间
    long current = System.currentTimeMillis();
    //取得当前线程id（判断是否可重入锁的关键）
    long threadId = Thread.currentThread().getId();
    //1.尝试申请锁，返回还剩余的锁过期时间
    Long ttl = tryAcquire(leaseTime, unit, threadId);
    //2.如果为空，表示申请锁成功
    if (ttl == null) {
        return true;
    }
    //3.申请锁的耗时如果大于等于最大等待时间，则申请锁失败
    time -= System.currentTimeMillis() - current;
    if (time <= 0) {
        /**
         * 通过 promise.trySuccess 设置异步执行的结果为null
         * Promise从Uncompleted-->Completed ,通知 Future 异步执行已完成
         */
        acquireFailed(threadId);
        return false;
    }

    current = System.currentTimeMillis();

    /**
     * 4.订阅锁释放事件，并通过await方法阻塞等待锁释放，有效的解决了无效的锁申请浪费资源的问题：
     * 基于信息量，当锁被其它资源占用时，当前线程通过 Redis 的 channel 订阅锁的释放事件，一旦锁释放会发消息通知待等待的线程进行竞争
     * 当 this.await返回false，说明等待时间已经超出获取锁最大等待时间，取消订阅并返回获取锁失败
     * 当 this.await返回true，进入循环尝试获取锁
     */
    RFuture<RedissonLockEntry> subscribeFuture = subscribe(threadId);
    //await 方法内部是用CountDownLatch来实现阻塞，获取subscribe异步执行的结果（应用了Netty 的 Future）
    if (!await(subscribeFuture, time, TimeUnit.MILLISECONDS)) {
        if (!subscribeFuture.cancel(false)) {
            subscribeFuture.onComplete((res, e) -> {
                if (e == null) {
                    unsubscribe(subscribeFuture, threadId);
                }
            });
        }
        acquireFailed(threadId);
        return false;
    }

    try {
        //计算获取锁的总耗时，如果大于等于最大等待时间，则获取锁失败
        time -= System.currentTimeMillis() - current;
        if (time <= 0) {
            acquireFailed(threadId);
            return false;
        }

        /**
         * 5.收到锁释放的信号后，在最大等待时间之内，循环一次接着一次的尝试获取锁
         * 获取锁成功，则立马返回true，
         * 若在最大等待时间之内还没获取到锁，则认为获取锁失败，返回false结束循环
         */
        while (true) {
            long currentTime = System.currentTimeMillis();
            // 再次尝试申请锁
            ttl = tryAcquire(leaseTime, unit, threadId);
            // 成功获取锁则直接返回true结束循环
            if (ttl == null) {
                return true;
            }

            //超过最大等待时间则返回false结束循环，获取锁失败
            time -= System.currentTimeMillis() - currentTime;
            if (time <= 0) {
                acquireFailed(threadId);
                return false;
            }

            /**
             * 6.阻塞等待锁（通过信号量(共享锁)阻塞,等待解锁消息）：
             */
            currentTime = System.currentTimeMillis();
            if (ttl >= 0 && ttl < time) {
                //如果剩余时间(ttl)小于wait time ,就在 ttl 时间内，从Entry的信号量获取一个许可(除非被中断或者一直没有可用的许可)。
                getEntry(threadId).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
            } else {
                //则就在wait time 时间范围内等待可以通过信号量
                getEntry(threadId).getLatch().tryAcquire(time, TimeUnit.MILLISECONDS);
            }

            //7.更新剩余的等待时间(最大等待时间-已经消耗的阻塞时间)
            time -= System.currentTimeMillis() - currentTime;
            if (time <= 0) {
                acquireFailed(threadId);
                return false;
            }
        }
    } finally {
        //7.无论是否获得锁,都要取消订阅解锁消息
        unsubscribe(subscribeFuture, threadId);
    }
}
```

#### 核心加锁代码

```java
private Long tryAcquire(long waitTime, long leaseTime, TimeUnit unit, long threadId) {
    return get(tryAcquireAsync(waitTime, leaseTime, unit, threadId));
}

private <T> RFuture<Long> tryAcquireAsync(long leaseTime, TimeUnit unit, long threadId) {
    //设置了锁持有时间
   if (leaseTime != -1) {
        return tryLockInnerAsync(leaseTime, unit, threadId, RedisCommands.EVAL_LONG);
    }
    
    //未设置锁持有时间，使用看门狗的默认的30秒
    RFuture<Long> ttlRemainingFuture = tryLockInnerAsync(commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout(), TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_LONG);
    
    // 异步获取结果，如果获取锁成功，则启动定时线程进行锁续约
    ttlRemainingFuture.onComplete((ttlRemaining, e) -> {
        if (e != null) {
            return;
        }

        // 启动WatchDog
        if (ttlRemaining == null) {
            scheduleExpirationRenewal(threadId);
        }
    });
    return ttlRemainingFuture;
}
<T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
    internalLockLeaseTime = unit.toMillis(leaseTime);

    /**
     * 通过 EVAL 命令执行 Lua 脚本获取锁，保证了原子性
     */
    return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, command,
              // 1.如果缓存中的key不存在，则执行 hset 命令(hset key UUID+threadId 1),然后通过 pexpire 命令设置锁的过期时间(即锁的租约时间)
              // 返回空值 nil ，表示获取锁成功
              "if (redis.call('exists', KEYS[1]) == 0) then " +
                  "redis.call('hset', KEYS[1], ARGV[2], 1); " +
                  "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                  "return nil; " +
              "end; " +
               // 如果key已经存在，并且value也匹配，表示是当前线程持有的锁，则执行 hincrby 命令，重入次数加1，并且设置失效时间
              "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                  "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                  "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                  "return nil; " +
              "end; " +
               //如果key已经存在，但是value不匹配，说明锁已经被其他线程持有，通过 pttl 命令获取锁的剩余存活时间并返回，至此获取锁失败
              "return redis.call('pttl', KEYS[1]);",
               //这三个参数分别对应KEYS[1]，ARGV[1]和ARGV[2]
               Collections.<Object>singletonList(getName()), internalLockLeaseTime, getLockName(threadId));
}
```

参数说明：

- KEYS[1]就是Collections.singletonList(getName())，表示分布式锁的key；
- ARGV[1]就是internalLockLeaseTime，即锁的租约时间（持有锁的有效时间），默认30s；
- ARGV[2]就是getLockName(threadId)，是获取锁时set的唯一值 value，即UUID+threadId。

大家注意到`看门狗`那个功能没？scheduleExpirationRenewal(threadId);这个方法的使命就是给锁`续命`。

简单来说就是一个`定时任务`，定时去判断锁还有多久失效，如果快失效了，就把锁的`失效时间延长`。

这里就实现了我们之前所说的第四点：**锁不能自己失效**

### tryLock解锁源码解读

大家先看看加锁流程图

![img](https://mmbiz.qpic.cn/mmbiz_png/DB0IfcV7dwibvAgxm5FIb9rVibVog6lU7nwc1x9OZruV8SfqZtibh45IgDxz4aARUFTFvnNyfOg8wszQm0okuIXibA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 源码

调用关系：`unlock` —> `unlockAsync` —> `unlockInnerAsync`，unlockInnerAsync是解锁的`核心代码`

```java
@Override
public void unlock() {
    try {
        get(unlockAsync(Thread.currentThread().getId()));
    } catch (RedisException e) {
        if (e.getCause() instanceof IllegalMonitorStateException) {
            throw (IllegalMonitorStateException) e.getCause();
        } else {
            throw e;
        }
    }
}

@Override
public RFuture<Void> unlockAsync(long threadId) {
    RPromise<Void> result = new RedissonPromise<Void>();
    RFuture<Boolean> future = unlockInnerAsync(threadId);

    future.onComplete((opStatus, e) -> {
        cancelExpirationRenewal(threadId);

        if (e != null) {
            result.tryFailure(e);
            return;
        }

        if (opStatus == null) {
            IllegalMonitorStateException cause = new IllegalMonitorStateException("attempt to unlock lock, not locked by current thread by node id: "
                                                                                  + id + " thread-id: " + threadId);
            result.tryFailure(cause);
            return;
        }

        result.trySuccess(null);
    });

    return result;
}

// 核心解锁代码
protected RFuture<Boolean> unlockInnerAsync(long threadId) {
    /**
     * 通过 EVAL 命令执行 Lua 脚本获取锁，保证了原子性
     */
    return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
            //如果分布式锁存在，但是value不匹配，表示锁已经被其他线程占用，无权释放锁，那么直接返回空值（解铃还须系铃人）
            "if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then " +
                "return nil;" +
            "end; " +
             //如果value匹配，则就是当前线程占有分布式锁，那么将重入次数减1
            "local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); " +
             //重入次数减1后的值如果大于0，表示分布式锁有重入过，那么只能更新失效时间，还不能删除
            "if (counter > 0) then " +
                "redis.call('pexpire', KEYS[1], ARGV[2]); " +
                "return 0; " +
            "else " +
             //重入次数减1后的值如果为0，这时就可以删除这个KEY，并发布解锁消息，返回1
                "redis.call('del', KEYS[1]); " +
                "redis.call('publish', KEYS[2], ARGV[1]); " +
                "return 1; "+
            "end; " +
            "return nil;",
            //这5个参数分别对应KEYS[1]，KEYS[2]，ARGV[1]，ARGV[2]和ARGV[3]
            Arrays.<Object>asList(getName(), getChannelName()), LockPubSub.UNLOCK_MESSAGE, internalLockLeaseTime, getLockName(threadId));

}
```

**解锁消息通知：**

之前加锁的时候源码里写过，如果没获取锁成功，就监听这个锁，监听它什么时候释放，所以解锁的时候，要发出这个消息通知，让其他想获取锁的客户端知道。

```java
public class LockPubSub extends PublishSubscribe<RedissonLockEntry> {

    public static final Long UNLOCK_MESSAGE = 0L;
    public static final Long READ_UNLOCK_MESSAGE = 1L;

    public LockPubSub(PublishSubscribeService service) {
        super(service);
    }
    
    @Override
    protected RedissonLockEntry createEntry(RPromise<RedissonLockEntry> newPromise) {
        return new RedissonLockEntry(newPromise);
    }

    @Override
    protected void onMessage(RedissonLockEntry value, Long message) {

        /**
         * 判断是否是解锁消息
         */
        if (message.equals(UNLOCK_MESSAGE)) {
            Runnable runnableToExecute = value.getListeners().poll();
            if (runnableToExecute != null) {
                runnableToExecute.run();
            }

            /**
             * 释放一个信号量，唤醒等待的entry.getLatch().tryAcquire去再次尝试申请锁
             */
            value.getLatch().release();
        } else if (message.equals(READ_UNLOCK_MESSAGE)) {
            while (true) {
                /**
                 * 如果还有其他Listeners回调，则也唤醒执行
                 */
                Runnable runnableToExecute = value.getListeners().poll();
                if (runnableToExecute == null) {
                    break;
                }
                runnableToExecute.run();
            }

            value.getLatch().release(value.getLatch().getQueueLength());
        }
    }
}
```

到这里，Redis官方实现的分布式锁源码就讲完了，但是有个问题，它虽然实现了`锁不能自己失效`这个特性，但是容错性方面还是没有实现。

## 容错性场景举例

因为在工作中Redis都是集群部署的，所以要考虑集群节点挂掉的问题。给大家举个例子：

- 1、A客户端请求主节点获取到了锁
- 2、主节点挂掉了，但是还没把锁的信息同步给其他从节点
- 3、由于主节点挂了，这时候开始主从切换，从节点成为主节点继续工作，但是新的主节点上，没有A客户端的加锁信息
- 4、这时候B客户端来加锁，因为目前是一个新的主节点，上面没有其他客户端加锁信息，所以B客户端获取锁成功
- 5、这时候就存在问题了，A和B两个客户端同时都持有锁，同时在执行代码，那么这时候分布式锁就失效了。

这里大家会有疑问了，为啥官方给出一个分布式锁的实现，却不解决这个问题呢，因为发生这种情况的几率不大，而且解决这个问题的成本有点小高。

所以，如果业务场景可以容忍这种小概率的错误，则推荐使用 RedissonLock，如果无法容忍，老哥这里给忍不了的同学留个思考题。

**RedissonRedLock**，这个中文名字叫`红锁`，它可以解决这个集群容错性的问题，这里把它当做思考题留给大家。别偷懒，下去认真学。



