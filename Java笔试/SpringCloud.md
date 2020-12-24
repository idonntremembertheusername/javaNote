#### 微服务的优点

各个服务的开发、测试、部署都相互独立，用户服务可以拆分为独立服务，如果用户量很大，可以很容易对其实现负载。

当新需求出现时，使用微服务不再需要考虑各方面的问题，例如兼容性、影响度等。

使用微服务拆分项目后，各个服务之间消除了很多限制，只需要保证对外提供的接口正常可用，而不限制语言和框架等选择。

------

#### 服务治理 Eureka

服务治理由三部分组成：服务提供者、服务消费者、注册中心。

服务注册：在分布式系统架构中，每个微服务在启动时，将自己的信息存储在注册中心。

服务发现：服务消费者从注册中心获取服务提供者的网络信息，通过该信息调用服务。

Spring Cloud 的服务治理使用 Eureka 实现，Eureka 是 Netflix 开源的基于 REST 的服务治理解决方案，Spring Cloud 集成了 Eureka，提供服务注册和服务发现的功能，可以和基于 Spring Boot 搭建的微服务应用轻松完成整合。

------

#### 服务网关 Zuul

Spring Cloud 集成了 Zuul 组件，实现服务网关。

Zuul 是 Netflix 提供的一个开源的 API 网关服务器，是客户端和网站后端所有请求的中间层，对外开放一个 API，将所有请求导入统一的入口，屏蔽了服务端的具体实现逻辑，可以实现方向代理功能，在网关内部实现动态路由、身份认证、IP过滤、数据监控等。

------

#### 负载均衡 Ribbon

Ribbon 是 Netflix 发布的均衡负载器，Spring Cloud 集成了 Ribbon，提供用于对 HTTP 请求进行控制的负载均衡客户端。

负载均衡策略：

在注册中心对 Ribbon 进行注册之后，Ribbon 就可以基于某种负载均衡算法、随机、加权轮询、加权随机等）自动帮助服务消费者调用接口，开发者也可以根据具体需求自定义 Ribbon 负载均衡算法。实际开发中 Spring Clooud Ribbon 需要结合 Spring Cloud Eureka 使用，Eureka 提供所有可以调用的服务提供者列表，Ribbon 基于特定的负载均衡算法从这些服务提供者中选择要调用的实例。

------

#### 服务调用 Feign

Feign 是 Netflix 提供的，一个声明式、模板化的 Web Service 客户端，简化了开发者编写 Web 客户端的操作，开发者可以通过简单的接口和注解来调用 HTTP API。

相比于 Ribbon + RestTemplate 的方式，Feign 可以大大简化代码开发，支持多种注解，包括 Feign 注解、Spring MVC 注解等。

RestTemplate 是 Spring 框架提供的基于 REST 的服务组件，底层是对 HTTP 请求及响应进行了封装，提供了很多访问 REST 服务的方法，简化代码开发。

------

#### 服务配置 Config

Spring Cloud Config 通过服务端可以为多个客户端提供配置服务，既可以将配置文件存储在本地，也可以将配置文件存储在远程的 Git 仓库，创建 Config Server，通过它管理所有的配置文件。

------

#### 服务跟踪 Zipkin

Spring Cloud Zipkin 是一个可以采集并跟踪分布式系统中请求数据的组件，让开发者更直观地监控到请求在各个微服务耗费的时间，Zipkin 包括两部分 Zipkin Server 和 Zipkin Client。

------

#### 服务熔断 Hystrix

**熔断器的作用：**某个服务的单个点的请求故障会导致用户的请求处于阻塞状态，最终的结果就是整个服务的线程资源消耗殆尽。由于服务的依赖性，会导致依赖于该故障服务的其他服务也处于线程阻塞状态，最终导致这些服务的线程资源消耗殆尽 直到不可用，从而导致整个问服务系统都不可用，即雪崩效应。

**设计原则：**服务隔离、服务降级、熔断机制、提供实时监控和报警功能、提供实时配置修改功能。

Hystrix 数据监控需要结合 `Spring Boot Actuator` 使用，Actuator 提供了对服务的数据监控、数据统计，可以通过 `hystirx-stream` 节点获取监控的请求数据，同时提供了可视化监控界面。

当对特定的服务的调用的不可用达到一个**阀值**（Hystrix 是 **5 秒 20 次**） 熔断器将会被打开。

在 Spring Cloud 中可以用 `RestTemplate + Ribbon` 和 `Feign` 来调用。

在Feign的yml配置文件上使用熔断器:

```yml
feign:
  hystrix:
    enabled: true
    command:
        default:  #default全局有效，service id指定应用有效
          execution:
            timeout:
              #如果enabled设置为false，则请求超时交给ribbon控制,为true,则超时作为熔断根据
              enabled: true
            isolation:
              thread:
                timeoutInMilliseconds: 5000 #断路器超时时间，默认1000ms
```



------



#### **SpringCloud如何实现服务的注册和发现**

1. 服务在发布时 指定对应的服务名（服务名包括了IP地址和端口） 将服务注册到注册中心（eureka或者zookeeper）。

2. 这一过程是springcloud自动实现 只需要在main方法添加@EnableDisscoveryClient 同一个服务修改端口就可以启动多个实例。

3. 调用方法：传递服务名称通过注册中心获取所有的可用实例 通过负载均衡策略调用（ribbon和feign）对应的服务。

   ------
   
   

####  **ribbon和feign区别**

- Ribbon添加maven依赖 spring-starter-ribbon 使用@RibbonClient(value="服务名称") 使用RestTemplate调用远程服务对应的方法。
- feign添加maven依赖 spring-starter-feign 服务提供方提供对外接口 调用方使用 在接口上使用@FeignClient("指定服务名")

Ribbon和Feign的区别：

Ribbon和Feign都是用于调用其他服务的，不过方式不同。

1. 启动类使用的注解不同，Ribbon用的是@RibbonClient，Feign用的@EnableFeignClients。
2. 服务的指定位置不同，Ribbon是在@RibbonClient注解上声明，Feign则是在定义抽象方法的接口中使用@FeignClient声明。
3. 调用方式不同，Ribbon需要自己构建http请求，模拟http请求然后使用RestTemplate发送给其他服务，步骤相当繁琐。

​    Feign则是在Ribbon的基础上进行了一次改进，采用接口的方式，将需要调用的其他服务的方法定义成抽象方法即可，

​    不需要自己构建http请求。不过要注意的是抽象方法的注解、方法签名要和提供服务的方法完全一致。