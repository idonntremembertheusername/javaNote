# 死锁：

1. 互斥使用；
2. 占有且等待；
3. 不可抢占；
4. 循环等待。
   	   

## 解除死锁：

#### 1. 破坏互斥条件

使资源同时访问而非互斥使用，就没有进程会阻塞在资源上，从而不发生死锁。

#### 2. 破坏占有和等待条件

采用静态分配的方式，静态分配的方式是指进程必须在执行之前就申请需要的全部资源，且直至所要的资源全部得到满足后才开始执行。
#### 3.破坏不剥夺条件

剥夺调度能够防止死锁，但是只适用于内存和处理器资源。
方法一：占有资源的进程若要申请新资源，必须主动释放已占有资源，若需要此资源，应该向系统重新申请。
方法二：资源分配管理程序为进程分配新资源时，若有则分配；否则将剥夺此进程已占有的全部资源，并让进程进入等待资源状态，资源充足后再唤醒它重新申请所有所需资源。

#### 4.破坏循环等待条件

给系统的所有资源编号，规定进程请求所需资源的顺序必须按照资源的编号依次进行。
采用层次分配策略，将系统中所有的资源排列到不同层次中：
一个进程得到某层的一个资源后，只能申请较高一层的资源；
当进程释放某层的一个资源时，必须先释放所占有的较高层的资源；
当进程获得某层的一个资源时，如果想申请同层的另一个资源，必须先释放此层中已占有的资源。

------

### 1. 锁类型

- **可重入锁**：广义上的可重入锁指的是可重复可递归调用的锁，在外层使用锁之后，在内层仍然可以使用，并且不发生死锁（前提得是同一个对象或者class），这样的锁就叫做可重入锁。即在执行对象中所有同步方法不用再次获得锁。ReentrantLock和synchronized都是可重入锁。举个简单的例子，当一个线程执行到某个synchronized方法时，比如说method1，而在method1中会调用另外一个synchronized方法method2，此时线程不必重新去申请锁，而是可以直接执行方法method2。
- **可中断锁**：在等待获取锁过程中可中断。synchronized就不是可中断锁，而Lock是可中断锁。
- **公平锁**： 按等待获取锁的线程的等待时间进行获取，等待时间长的具有优先获取锁权利。非公平锁即无法保证锁的获取是按照请求锁的顺序进行的，这样就可能导致某个或者一些线程永远获取不到锁。synchronized是非公平锁，它无法保证等待的线程获取锁的顺序。对于ReentrantLock和ReentrantReadWriteLock，默认情况下是非公平锁，但是可以设置为公平锁。
- **读写锁**：对资源读取和写入的时候拆分为2部分处理，一个读锁和一个写锁。读的时候可以多线程一起读，写的时候必须同步地写。ReadWriteLock就是读写锁，它是一个接口，ReentrantReadWriteLock实现了这个接口。可以通过readLock()获取读锁，通过writeLock()获取写锁。

### 2. 什么是乐观锁和悲观锁

1. **乐观锁**：很乐观，每次去**拿数据**的时候都**认为别人不会修改**，所以不会上锁，但是在**更新**的时候会**去判断**在此期间有没有人去更新这个数据（可以使用版本号等机制）。如果因为**冲突失败就重试**。乐观锁**适用于写比较少的情况下**，即冲突比较少发生，这样可以省去了锁的开销，加大了系统的整个吞吐量。像数据库提供的类似于write_condition机制，其实都是提供的乐观锁。在Java中java.util.concurrent.atomic包下面的原子变量类就是使用了乐观锁的一种实现方式CAS实现的。
2. **悲观锁**：总是假设最坏的情况，每次去**拿数据**的时候都**认为别人会修改**，因此每次**拿数据**的时候都**会上锁**，这样**别人想拿**这个数据就会**阻塞**直到它拿到锁，**效率比较低**。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。再比如Java里面的同步原语synchronized关键字的实现也是悲观锁。

### 3. 乐观锁的实现方式（CAS）

乐观锁的实现主要就**两个步骤**：**冲突检测**和**数据更新**。其实现方式有一种比较典型的就是 Compare and Swap ( CAS )。
CAS：CAS是乐观锁技术，当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。
CAS 操作中包含三个操作数 —— 需要读写的内存位置（V）、进行比较的预期原值（A）和拟写入的新值(B)。如果内存位置V的值与预期原值A相匹配，那么处理器会自动将该位置值更新为新值B。否则处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该位置的值。（在 CAS 的一些特殊情况下将仅返回 CAS 是否成功，而不提取当前值。）CAS 有效地说明了“ 我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。 ”这其实和乐观锁的冲突检查+数据更新的原理是一样的。

> 乐观锁是一种思想，CAS是这种思想的一种实现方式。

### 4. CAS的缺点

1. ABA问题
   \> 如果内存地址V初次读取的值是A，并且在准备赋值的时候检查到它的值仍然为A，那我们就能说它的值没有被其他线程改变过了吗？如果在这段期间它的值曾经被改成了B，后来又被改回为A，那CAS操作就会误认为它从来没有被改变过。这个漏洞称为CAS操作的“ABA”问题。ava并发包为了解决这个问题，提供了一个带有标记的原子引用类“AtomicStampedReference”，它可以通过控制变量值的版本来保证CAS的正确性。因此，在使用CAS前要考虑清楚“ABA”问题是否会影响程序并发的正确性，如果需要解决ABA问题，改用传统的互斥同步可能会比原子类更高效。
2. 循环时间长开销很大
   \> 自旋CAS（不成功，就一直循环执行，直到成功）如果长时间不成功，会给CPU带来非常大的执行开销。
3. 只能保证一个共享变量的原子操作。
   \> 当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁来保证原子性。

### 5. 实现一个死锁

什么是死锁：**两个进程都**在**等待对方执行完毕**才能继续往下执行的时候就发生了死锁。结果就是两个进程都陷入了无限的等待中。

产生死锁的**四个必要条件**：

- 互斥条件：一个资源每次只能被一个进程使用。
- 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
- 不剥夺条件:进程已获得的资源，在末使用完之前，不能强行剥夺。
- 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

这四个条件是死锁的必要条件，只要系统发生死锁，这些条件必然成立，而只要上述条件之一不满足，就不会发生死锁。

**考虑如下情形：**

1. 线程A当前持有互斥锁lock1，线程B当前持有互斥锁lock2。
2. 线程A试图获取lock2，因为线程B正持有lock2，因此线程A会阻塞等待线程B对lock2释放。
3. 如果此时线程B也在试图获取lock1，同理线程也会阻塞。
4. 两者都在等待对方所持有但是双方都不释放的锁，这时便会一直阻塞形成死锁。

**死锁的解决方法:**

- 撤消陷于死锁的全部进程；
- 逐个撤消陷于死锁的进程，直到死锁不存在；
- 从陷于死锁的进程中逐个强迫放弃所占用的资源，直至死锁消失。
- 从另外一些进程那里强行剥夺足够数量的资源分配给死锁进程，以解除死锁状态

### 6. 如何确保 N 个线程可以访问 N 个资源同时又不导致死锁？

使用多线程的时候，一种非常简单的**避免死锁的方式**就是：指定**获取锁的顺序**，并**强制**线程按照**指定的顺序获取锁**。因此，如果所有的线程都是以同样的顺序加锁和释放锁，就不会出现死锁了

### 7. volatile关键字

- 对于**可见性、有序性及原子性**问题，通常情况下我们可以通过Synchronized关键字来解决这些个问题。
- 不过如果对Synchronized原理有了解的话，应该知道**Synchronized**是一个**比较重量级**的操作，对系统的性能有比较大的影响，所以，如果有其他解决方案，我们通常都避免使用Synchronized来解决问题。
- 而**volatile**关键字就是Java中提供的**另一种解决**可见性和有序性问题的方案。对于原子性，需要强调一点，也是大家容易误解的一点：**对volatile变量的单次读/写操作可以保证原子性**的，如long和double类型变量，但是并不能保证i++这种操作的原子性，因为本质上**i++是读、写两次操作**。

防止重排序

> 问题：操作系统可以对指令进行重排序，多线程环境下就可能将一个未初始化的对象引用暴露出来，从而导致不可预料的结果
> 解决原理：volatile关键字通过提供“内存屏障”的方式来防止指令被重排序，为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。
>
> 1. 在每个volatile写操作前插入StoreStore屏障，在写操作后插入StoreLoad屏障。
> 2. 在每个volatile读操作前插入LoadLoad屏障，在读操作后插入LoadStore屏障。

实现可见性

> 问题：可见性问题主要指一个线程修改了共享变量值，而另一个线程却看不到
> 解决原理：
>
> 1. 修改volatile变量时会强制将修改后的值刷新的主内存中。
> 2. 修改volatile变量后会导致其他线程工作内存中对应的变量值失效。因此，再读取该变量值的时候就需要重新从读取主内存中的值。

注：volatile并不保证变量更新的原子性

### 8. volatile使用建议

相对于synchronized块的代码锁，**volatile**应该是提供了一个**轻量级的针对共享变量的锁**，当我们在多个线程间使用共享变量进行通信的时候需要考虑将共享变量用volatile来修饰。

volatile是**一种稍弱的同步机制**，在访问volatile变量时**不会执行加锁**操作，也就**不会执行线程阻塞**，因此volatile变量是一种比synchronized关键字更轻量级的同步机制。
**使用建议**：在两个或者更多的线程需要访问的成员变量上使用volatile。当要访问的**变量已在synchronized代码块中**，或者为常量时，**没必要使用volatile**。
由于使用**volatile屏蔽掉了JVM中必要的代码优化**，所以在**效率上比较低**，因此一定在必要时才使用此关键字。

### 9. volatile和synchronized区别

1. **volatile不会进行加锁操作**：
   volatile变量是一种稍弱的同步机制在访问volatile变量时不会执行加锁操作，因此也就不会使执行线程阻塞，因此volatile变量是一种比synchronized关键字更轻量级的同步机制。
2. **volatile变量作用类似于同步变量读写操作：**
   从内存可见性的角度看，写入volatile变量相当于退出同步代码块，而读取volatile变量相当于进入同步代码块。
3. **volatile不如synchronized安全：**
   在代码中如果过度依赖volatile变量来控制状态的可见性，通常会比使用锁的代码更脆弱，也更难以理解。仅当volatile变量能简化代码的实现以及对同步策略的验证时，才应该使用它。一般来说，用同步机制会更安全些。
4. **volatile无法同时保证内存可见性和原子性：**
   加锁机制（即同步机制）既可以确保可见性又可以确保原子性，而volatile变量只能确保可见性，原因是声明为volatile的简单变量如果当前值与该变量以前的值相关，那么volatile关键字不起作用，也就是说如下的表达式都不是原子操作：“count++”、“count = count+1”。

**当且仅当满足以下所有条件时，才应该使用volatile变量**：

1. 对变量的写入操作不依赖变量的当前值，或者你能确保只有单个线程更新变量的值。
2. 该变量没有包含在具有其他变量的不变式中。

总结：在需要同步的时候，第一选择应该是synchronized关键字，这是最安全的方式，尝试其他任何方式都是有风险的。尤其在、jdK1.5之后，对synchronized同步机制做了很多优化，如：自适应的自旋锁、锁粗化、锁消除、轻量级锁等，使得它的性能明显有了很大的提升。

### 10. synchronized

synchronized可以保证方法或者代码块在运行时，**同一时刻只有一个方法**可以进入到临界区，同时它还可以**保证共享变量的内存可见性**。Synchronized主要有以下三个作用：保证互斥性、保证可见性、保证顺序性。

10.1 synchronized的三种应用方式

1. **修饰实例方法**，作用于当前实例加锁，进入同步代码前要获得当前实例的锁。

   **实现原理**：指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先持有monitor（虚拟机规范中用的是管程一词）， 然后再执行方法，最后再方法完成(无论是正常完成还是非正常完成)时释放monitor。
   
   ```java
   public synchronized void increase(){
       i++;
   }
   ```
   
2. **修饰静态方法**，作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁

   ```java
   public static synchronized void increase(){
       i++;
   }
   ```

3. **修饰代码块**，指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。实现原理：使用的是monitorenter 和 monitorexit 指令，其中monitorenter指令指向同步代码块的开始位置，monitorexit指令则指明同步代码块的结束位置。

   ```java
   static AccountingSync instance=new AccountingSync();
   
   synchronized(instance){
   
       for(int j=0;j<1000000;j++){
   
           i++;
   
       }
   
   }
   ```

### 11. Lock

**Lock**是一个**接口**，它的的实现类提供了比synchronized更广泛意义上锁操作，它允许用户更灵活的代码结构，更多的不同特效。Lock的实现类主要有**ReentrantLock**和**ReentrantReadWriteLock**。

 

```
Lock lock=new ReentrantLock()；



lock.lock();



try{



    // do something



    // 如果有return要写在try块中



}finally{



    lock.unlock();



}
```

### 12. Lock接口中获取锁的方法

- **void lock()**：lock()方法是平常使用得最多的一个方法，就是用来**获取锁**。如果锁已被其他线程获取，则进行等待。在发生异常时，它不会自动释放锁，要记得在finally块中释放锁，以保证锁一定被被释放，防止死锁的发生。
- **void lockInterruptibly()**：可以**响应中断**，当通过这个方法去获取锁时，如果线程 正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。
- **boolean tryLock()**：有返回值的，它表示**用来尝试获取锁**，如果获取成功，则返回true；如果获取失败（即锁已被其他线程获取），则返回false。
- **boolean tryLock(long time, TimeUnit unit)**：和tryLock()方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false，同时可以响应中断。

### 13. Condition类

- Condition是**Java提供来实现等待/通知的类**，Condition类还提供比wait/notify更丰富的功能，Condition对象是**由lock对象所创建**的。但是**同一个锁可以创建多个Condition的对象**，即创建多个对象监视器。这样的好处就是可以指定唤醒线程。notify唤醒的线程是随机唤醒一个。
- Condition 将 Object 监视器方法**（wait、notify 和 notifyAll）分解成**截然**不同的对象**，以便通过将这些对象**与任意 Lock 实现组合使用**，为每个对象提供多个等待 set （wait-set）。
  其中，**Lock 替代了 synchronized 方法和语句的使用**，**Condition 替代了 Object 监视器方法的使用**。
- 在Condition中，用**await()替换wait()**，用**signal()替换notify()**，用**signalAll()替换notifyAll()**，传统线程的通信方式，Condition都可以实现，这里注意，**Condition是被绑定到Lock上**的，要创建一个Lock的Condition必须用newCondition()方法。

### 14. Condition与Object中的wait, notify, notifyAll区别

1. Condition中的await()方法相当于Object的wait()方法，Condition中的signal()方法相当于Object的notify()方法，Condition中的signalAll()相当于Object的notifyAll()方法。
   不同的是，**Object中的这些方法是和同步锁捆绑使用**的；而**Condition是需要与互斥锁/共享锁捆绑使用**的。
2. Condition它更强大的地方在于：**能够更加精细的控制多线程的休眠与唤醒**。对于同一个锁，我们可以创建多个Condition，在不同的情况下使用不同的Condition。
   例如，假如多线程读/写同一个缓冲区：当向缓冲区中写入数据之后，唤醒”读线程”；当从缓冲区读出数据之后，唤醒”写线程”；并且当缓冲区满的时候，”写线程”需要等待；当缓冲区为空时，”读线程”需要等待。

如果采用Object类中的wait(),notify(),notifyAll()实现该缓冲区，当向缓冲区写入数据之后需要唤醒”读线程”时，不可能通过notify()或notifyAll()明确的指定唤醒”读线程”，而只能通过notifyAll唤醒所有线程(但是notifyAll无法区分唤醒的线程是读线程，还是写线程)。 但是，通过Condition，就能明确的指定唤醒读线程。

### 15. synchronized和lock的区别

|          | synchronized                                                 | Lock                                        |
| :------- | :----------------------------------------------------------- | :------------------------------------------ |
| 存在层次 | Java的关键字                                                 | 是一个接口                                  |
| 锁的释放 | 1、以获取锁的线程执行完同步代码，释放锁 2、线程执行发生异常，jvm会让线程释放锁 | 在finally中必须释放锁，不然容易造成线程死锁 |
| 锁的获取 | 假设A线程获得锁，B线程等待。如果A线程阻塞，B线程会一直等待   | Lock可以让等待锁的线程响应中断              |
| 锁状态   | 无法判断                                                     | 可以判断有没有成功获取锁                    |
| 锁类型   | 可重入 不可中断 非公平                                       | 可重入 可中断 公平/非公平                   |

性能方面，**JDK1.5**中，**synchronized是性能低效**的。因为这是一个重量级操作，它对性能最大的影响是阻塞的是实现，挂起线程和恢复线程的操作都需要转入内核态中完成，这些操作给系统的并发性带来了很大的压力。相比之下使用Java提供的**Lock对象，性能更高一些**。多线程环境下，synchronized的吞吐量下降的非常严重，而ReentrankLock则能基本保持在同一个比较稳定的水平上。

到了**JDK1.6**，**synchronize加入了很多优化措施**，有自适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等。导致在JDK1.6上synchronize的性能**并不比Lock差**。官方也表示，他们也更支持synchronize，在未来的版本中还有优化余地，所以还是提倡在synchronized能实现需求的情况下，优先考虑使用synchronized来进行同步。

### 16. 锁的状态

Java SE1.6为了**减少获得锁和释放锁所带来的性能消耗**，引入了“**偏向锁**”和“**轻量级锁**”，所以在Java SE1.6里锁一共有**四种**状态，**无锁状态**，**偏向锁状态**，**轻量级锁状态**和**重量级锁状态**，它会随着竞争情况逐渐升级。**锁可以升级但不能降级**，意味着偏向锁升级成轻量级锁后不能降级成偏向锁。这种锁升级却不能降级的策略，目的是为了**提高获得锁和释放锁的效率**。

16.1 偏向锁

在没有实际竞争的情况下，还能够针对部分场景继续优化。如果不仅仅没有实际竞争，自始至终，使用锁的线程都只有一个，那么，维护轻量级锁都是浪费的。偏向锁的目标是，减少无竞争且只有一个线程使用锁的情况下，使用轻量级锁产生的性能消耗。轻量级锁每次申请、释放锁都至少需要一次CAS，但**偏向锁只有初始化时需要一次CAS**。
“偏向”的意思是，偏向锁假定将来只有第一个申请锁的线程会使用锁（不会有任何线程再来申请锁），因此，只需要在Mark Word中CAS记录owner（本质上也是更新，但初始值为空），如果记录成功，则偏向锁获取成功，记录锁状态为偏向锁，以后当前线程等于owner就可以零成本的直接获得锁；否则，说明有其他线程竞争，膨胀为轻量级锁。
偏向锁无法使用自旋锁优化，因为一旦有其他线程申请锁，就破坏了偏向锁的假定。

16.2 轻量级锁

轻量级锁是由偏向所升级来的，**偏向锁运行在一个线程进入同步块的情况下，当第二个线程加入锁争用的时候，偏向锁就会升级为轻量级锁**。轻量级锁是在没有多线程竞争的前提下，减少传统的重量级锁使用产生的性能消耗。轻量级锁所适应的场景是线程交替执行同步块的情况，如果存在同一时间访问同一锁的情况，就会导致轻量级锁膨胀为重量级锁。
使用轻量级锁时，不需要申请互斥量，仅仅将Mark Word中的部分字节CAS更新指向线程栈中的Lock Record，如果更新成功，则轻量级锁获取成功，记录锁状态为轻量级锁；否则，说明已经有线程获得了轻量级锁，目前发生了锁竞争（不适合继续使用轻量级锁），接下来膨胀为重量级锁。

16.3 重量级锁

重量锁在JVM中**又叫对象监视器（Monitor）**，它很像C中的Mutex，除了具备Mutex(0|1)互斥的功能，它还负责实现了Semaphore(信号量)的功能，也就是说它至少包含一个竞争锁的队列，和一个信号阻塞队列（wait队列），前者负责做互斥，后一个用于做线程同步。

16.4 自旋锁

自旋锁原理非常简单，如果持有锁的线程能在很短时间内释放锁资源，那么那些等待竞争锁的线程就不需要做内核态和用户态之间的切换进入阻塞挂起状态，它们只需要等一等（自旋），等持有锁的线程释放锁后即可立即获取锁，这样就避免用户线程和内核的切换的消耗。
但是线程自旋是需要消耗cup的，说白了就是让cup在做无用功，如果一直获取不到锁，那线程也不能一直占用cup自旋做无用功，所以需要设定一个自旋等待的最大时间。
如果持有锁的线程执行的时间超过自旋等待的最大时间扔没有释放锁，就会导致其它争用锁的线程在最大等待时间内还是获取不到锁，这时争用线程会停止自旋进入阻塞状态。

16.5 自适应自旋锁

自适应意味着自旋的时间不再固定了，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定：

- 如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很有可能再次成功，进而它将允许自旋等待持续相对更长的时间，比如100个循环。
- 相反的，如果对于某个锁，自旋很少成功获得过，那在以后要获取这个锁时将可能减少自旋时间甚至省略自旋过程，以避免浪费处理器资源。

自适应自旋解决的是“锁竞争时间不确定”的问题。JVM很难感知到确切的锁竞争时间，而交给用户分析就违反了JVM的设计初衷。自适应自旋假定不同线程持有同一个锁对象的时间基本相当，竞争程度趋于稳定，因此，可以根据上一次自旋的时间与结果调整下一次自旋的时间。

16.6 偏向锁、轻量级锁、重量级锁适用于不同的并发场景

**偏向锁**：无实际竞争，且将来只有第一个申请锁的线程会使用锁。
**轻量级锁**：无实际竞争，多个线程交替使用锁；允许短时间的锁竞争。
**重量级锁**：有实际竞争，且锁竞争时间长。
另外，如果锁**竞争时间短**，可以使用**自旋锁**进一步**优化**轻量级锁、重量级锁的性能，减少线程切换。
如果**锁竞争程度逐渐提高**（缓慢），那么**从偏向锁逐步膨胀到重量锁**，能够提高系统的整体性能。

> 锁膨胀的过程：只有一个线程进入临界区（偏向锁），多个线程交替进入临界区（轻量级锁），多线程同时进入临界区（重量级锁）。

### 17. AQS

AQS即是AbstractQueuedSynchronizer，一个**用来构建锁和同步工具的框架**，包括常用的**ReentrantLock**、**CountDownLatch**、**Semaphore**等。
AbstractQueuedSynchronizer是一个**抽象类**，主要是**维护**了一个int类型的state属性和一个非阻塞、先进先出的**线程等待队列**；其中state是用volatile修饰的，保证线程之间的可见性，**队列的入队和出队操作都是无锁操作，基于自旋锁和CAS实现**；另外**AQS分为两种模式**：**独占模式**和**共享模式**，像ReentrantLock是基于独占模式模式实现的，CountDownLatch、CyclicBarrier等是基于共享模式。