### Spring IoC

#### IoC

IoC 控制反转，把对象创建、依赖反转给容器实现，需要创建一个容器和一种描述让容器知道对象间的依赖关系，Spring 通过 IoC 容器管理对象及其依赖关系。IoC 的主要实现方式是 DI，对象不是从容器中查找依赖的类，而是容器实例化对象时主动为它注入依赖的类。

**基于 XML 的 IoC 初始化**

当创建 ClassPathXmlApplicationContext 时，调用父类 AbstractApplicationContext 的 `refresh` 方法启动整个 IoC 容器对 Bean 定义的载入过程，在创建 IoC 容器前如果已有容器存在，需要销毁，保证使用的是新创建的容器。

容器创建后通过 `loadBeanDefinitions` 方法加载 Bean 配置资源，首先解析配置文件路径，读取配置文件内容，然后通过 XML 解析器将配置信息转换成文档对象，之后按照 Bean 的定义规则解析文档对象。

IoC 容器中注册的 Bean 信息存放在一个 HashMap 中，key 是字符串，值是 BeanDefinition。当配置信息中的 Bean 被解析且被注册到 IoC 容器后，初始化就算完成了。

------

#### DI

**实现**

- **构造方法注入** 

  IoC 容器会检查对象的构造方法，取得它的依赖对象列表，当对象实例化完成时依赖的属性也会成功注入，可以直接使用。缺点是当依赖对象较多时，可能需要多个构造方法。

- **setter 方法注入**

  只需要为依赖对象的属性添加 setter 方法，在描述性上要比构造方法注入强，缺点是无法在对象构造完成后就进入就绪状态。IoC 容器会先实例化 Bean 对象，然后通过反射调用 setter 方法注入属性。

- **注解注入**

  `@Autowired`：自动按类型注入，如果有多个匹配则按照指定 Bean 的 id 查找，需要搭配 `@Qualifier`。

  `@Resource` ：按照 Bean 的 id 注入，如果找不到则会按类型注入。

  `@Value` ：用于注入基本数据类型和 String。

------

#### Bean

**生命周期**

在 IoC 容器的初始化时会对 Bean 定义完成资源定位，加载读取配置并解析，最后将解析的 Bean 信息放在一个 HashMap 集合中。当 IoC 容器初始化后，会创建 Bean 实例并完成依赖注入，注入对象依赖的各种属性值，在初始化时可以指定自定义的初始化方法。经过一系列初始化操作后 Bean 达到可用状态，当使用完成后会调用 destroy 方法进行销毁，此时也可以指定自定义的销毁方法，最终 Bean 被销毁且从容器中移除。

通过配置 bean 标签或注解中的 init-Method 和 destory-Method 属性指定自定义初始化和销毁方法。 

------

**作用域**

通过 scope 属性指定作用域。

| 范围             | 作用域         | 备注                                             |
| ---------------- | -------------- | ------------------------------------------------ |
| 所有 Spring 应用 | singleton      | 默认作用域，每个容器中只有一个唯一的 Bean 实例。 |
|                  | prototype      | 每次 Bean 请求都会创建一个新的实例。             |
| Spring Web 应用  | request        | 为每个请求创建一个新的实例。                     |
|                  | session        | 为每个会话创建一个新的实例。                     |
|                  | global session | 为全局 session 创建一个新的实例。                |

------

**创建**

- **XML** 

  默认无参构造方法，只需指明 bean 标签中的 id 和 class 属性，如果没有无参构造方***报错。

  静态工厂方法，通过 bean 标签的 class 属性指明工厂，factory-method 属性指明方法。

  实例工厂方法，通过 bean 标签的 factory-bean 属性指明工厂，factory-method 属性指明方法。

- **注解** 

  `@Component` 把当前类对象存入 Spring 容器，相当于在 xml 中配置一个 bean 标签。value 属性指定 bean 的 id，默认使用当前类首字母小写的类名。

  `@Controller`，`@Service`，`@Repository` 都是 `@Component` 的衍生注解，作用及属性都一模一样，只是提供了更明确的语义，`@Controller` 用于表现层，`@Service`用于业务层，`@Repository`用于持久层。

  如果想注入第三方类又没有源码，就没法使用 `@Component`，需要用 `@Bean`。被 `@Bean` 注解的方法返回值是一个对象，这个对象由 Spring 的 IoC 容器管理，name 属性用于给对象指定一个名称。

------

**BeanFactory、FactoryBean 和 ApplicationContext 的区别**

BeanFactory 是一个 Bean 工厂，使用简单工厂模式，是 Spring IoC 容器顶级接口，作用是管理 Bean，包括实例化、定位、配置对象及维护对象间的依赖。BeanFactory 属于延迟加载，适合多例模式。

FactoryBean 是一个工厂 Bean，使用工厂方法模式，作用是生产其他 Bean 实例，可以通过实现该接口来自定义实例 Bean 的逻辑。如果一个 Bean 实现了这个接口，那么它就是创建对象的工厂 Bean，而不是 Bean 实例本身。

ApplicationConext 是 BeanFactory 的子接口，扩展了 BeanFactory 的功能，提供了支持国际化文本消息，统一的资源文件读取方式等功能。Bean 的依赖注入在容器初始化时就已经完成，属于立即加载，适合单例模式。

------

#### 注解配置文件

`@Configuration` 指定当前类是一个 Spring 配置类，创建容器时会从该类上加载注解，`value` 属性指定配置类的字节码。

`@ComponentScan` 开启组件扫描，`basePackages` 属性指定要扫描的包。

`@PropertySource` 用于加载 `properties` 文件中的配置。

`@Import` 导入其他配置类，有 `@Import` 的是父配置类，引入的是子配置类，value 属性指定其他配置类的字节码。

------

### Spring AOP 

#### AOP

AOP 面向切面编程，将代码中重复的部分抽取出来，使用动态代理技术，在不修改源码的基础上对方法进行增强。

如果目标对象实现了接口，默认采用 JDK 动态代理，也可以强制使用 CGLib；如果目标对象没有实现接口，采用 CGLib 的方式。

常用场景包括权限认证、自动缓存、错误处理、日志、调试和事务等。

------

#### 相关注解

`@Aspect`：声明被注解的类是一个切面 Bean。

`@Before`：前置通知，指在某个连接点之前执行的通知。

`@After`：后置通知，指某个连接点退出时执行的通知（不论正常返回还是异常退出）。

`@AfterReturning`：返回后通知，指某连接点正常完成之后执行的通知，返回值使用 returning 属性接收。

`@AfterThrowing`：异常通知，指方法异常退出时执行的通知，和 `@AfterReturning` 只会有一个执行，异常使用 throwing 属性接收。

------

#### 相关术语

`Aspect`：切面，一个关注点的模块化，这个关注点可能会横切多个对象。

`Joinpoint`：连接点，程序执行过程中的某一行为，即业务层中的所有方法。

`Advice`：通知，指切面对于某个连接点所产生的动作，包括前置通知、后置通知、返回后通知、异常通知和环绕通知。

`Pointcut`：切入点，指被拦截的连接点，切入点一定是连接点，但连接点不一定是切入点。

`Proxy`：代理，Spring AOP 中有 JDK 动态代理和 CGLib 代理，目标对象实现了接口时采用 JDK 动态代理，反之采用 CGLib 代理。

`Target`：代理的目标对象，指一个或多个切面所通知的对象。

`Weaving` ：织入，指把增强应用到目标对象来创建代理对象的过程。

------

**1.什么是 spring?**

Spring 是个 java 企业级应用的开源开发框架。Spring 主要用来开发 Java 应用，但是有些扩展是针对构建 J2EE 平台的 web 应用。Spring 框架目标是简化 Java 企业级应用开发，并通过 POJO 为基础的编程模型促进良好的编程习惯。

**2.使用 Spring 框架的好处是什么？**

 **轻量：**Spring 是轻量的，基本的版本大约 2MB。 

 **控制反转：**Spring 通过控制反转实现了松散耦合，对象们给出它们的依赖，而不是创建

或查找依赖的对象们。

 **面向切面的编程(AOP)：**Spring 支持面向切面的编程，并且把应用业务逻辑和系统服务

分开。

 **容器：**Spring 包含并管理应用中对象的生命周期和配置。

 **MVC 框架**：Spring 的 WEB 框架是个精心设计的框架，是 Web 框架的一个很好的替代品。

 **事务管理：**Spring 提供一个持续的事务管理接口，可以扩展到上至本地事务下至全局事

务（JTA）。

 **异常处理：**Spring 提供方便的 API 把具体技术相关的异常（比如由 JDBC，HibernateorJDO

抛出的）转化为一致的 unchecked 异常。

**3.Spring 由哪些模块组成?**

以下是 Spring 框架的基本模块：

 Coremodule

 Beanmodule

 Contextmodule

 ExpressionLanguagemodule

 JDBCmodule

 ORMmodule

 OXMmodule

 JavaMessagingService(JMS)module

 Transactionmodule

 Webmodule

 Web-Servletmodule 

 Web-Strutsmodule

 Web-Portletmodule

**4.核心容器（应用上下文)模块。**

这是基本的 Spring 模块，提供 spring 框架的基础功能，BeanFactory 是任何以 spring 为

基础的应用的核心。Spring 框架建立在此模块之上，它使 Spring 成为一个容器。

**5.BeanFactory–BeanFactory 实现举例。**

Bean 工厂是工厂模式的一个实现，提供了控制反转功能，用来把应用的配置和依赖从正真

的应用代码中分离。

最常用的 BeanFactory 实现是 XmlBeanFactory 类。

**6.XMLBeanFactory**

最常用的就是 org.springframework.beans.factory.xml.XmlBeanFactory，它根据 XML 文

件中的定义加载 beans。该容器从 XML 文件读取配置元数据并用它去创建一个完全配置的系

统或应用。

**7.解释 AOP 模块**

AOP 模块用于发给我们的 Spring 应用做面向切面的开发，很多支持由 AOP 联盟提供，这样

就确保了 Spring 和其他 AOP 框架的共通性。这个模块将元数据编程引入 Spring。

**8.解释 JDBC 抽象和 DAO 模块。**

通过使用 JDBC 抽象和 DAO 模块，保证数据库代码的简洁，并能避免数据库资源错误关闭导

致的问题，它在各种不同的数据库的错误信息之上，提供了一个统一的异常访问层。它还利

用 Spring 的 AOP 模块给 Spring 应用中的对象提供事务管理服务。

**9.解释对象/关系映射集成模块。**

Spring 通过提供 ORM 模块，支持我们在直接 JDBC 之上使用一个对象/关系映射映射(ORM)工

具，Spring 支持集成主流的 ORM 框架，如 Hiberate,JDO 和 iBATISSQLMaps。Spring 的事务

管理同样支持以上所有 ORM 框架及 JDBC。

**10.解释 WEB 模块。**

Spring 的 WEB 模块是构建在 applicationcontext 模块基础之上，提供一个适合 web 应用的

上下文。这个模块也包括支持多种面向 web 的任务，如透明地处理多个文件上传请求和程序

级请求参数的绑定到你的业务对象。它也有对 JakartaStruts 的支持。

**11.为什么说 Spring 是一个容器？** 

因为用来形容它用来存储单例的 bean 对象这个特性。

**12.Spring 配置文件**

Spring 配置文件是个 XML 文件，这个文件包含了类信息，描述了如何配置它们，以及如何

相互调用。

**13.什么是 SpringIOC 容器？**

SpringIOC 负责创建对象，管理对象（通过依赖注入（DI），装配对象，配置对象，并且管

理这些对象的整个生命周期。

**14.IOC 的优点是什么？**

IOC 或依赖注入把应用的代码量降到最低。它使应用容易测试，单元测试不再需要单例和

JNDI 查找机制。最小的代价和最小的侵入性使松散耦合得以实现。IOC 容器支持加载服务时

的饿汉式初始化和懒加载。

**15.ApplicationContext 通常的实现是什么?** 

 **FileSystemXmlApplicationContext：**此容器从一个 XML 文件中加载 beans 的定义，

XMLBean 配置文件的全路径名必须提供给它的构造函数。

 **ClassPathXmlApplicationContext：**此容器也从一个 XML 文件中加载 beans 的定义，这

里，你需要正确设置 classpath 因为这个容器将在 classpath 里找 bean 配置。

 **WebXmlApplicationContext：**此容器加载一个 XML 文件，此文件定义了一个 WEB 应用的

所有 bean。

**16.Bean 工厂和 Applicationcontexts 有什么区别？**

Applicationcontexts 提供一种方法处理文本消息，一个通常的做法是加载文件资源（比如

镜像），它们可以向注册为监听器的 bean 发布事件。另外，在容器或容器内的对象上执行

的那些不得不由 bean 工厂以程序化方式处理的操作，可以在 Applicationcontexts 中以声

明的方式处理。Applicationcontexts 实现了 MessageSource 接口，该接口的实现以可插拔

的方式提供获取本地化消息的方法。

**17.一个 Spring 的应用看起来象什么？**

1.  一个定义了一些功能的接口。
2. 这实现包括属性，它的 Setter，getter 方法和函数等。
3.  SpringAOP。 
4. pring 的 XML 配置文件。
5. 使用以上功能的客户端程序。

**18.什么是 Spring 的依赖注入？**

依赖注入，是 IOC 的一个方面，是个通常的概念，它有多种解释。这概念是说你不用创建对

象，而只需要描述它如何被创建。你不在代码里直接组装你的组件和服务，但是要在配置文

件里描述哪些组件需要哪些服务，之后一个容器（IOC 容器）负责把他们组装起来。

**19.有哪些不同类型的 IOC（依赖注入）方式？**

 **构造器依赖注入：**构造器依赖注入通过容器触发一个类的构造器来实现的，该类有一系

列参数，每个参数代表一个对其他类的依赖。

 **Setter 方法注入：**Setter 方法注入是容器通过调用无参构造器或无参 static 工厂方法

实例化 bean 之后，调用该 bean 的 setter 方法，即实现了基于 setter 的依赖注入。

**20.哪种依赖注入方式你建议使用，构造器注入，还是 Setter 方法注入？**

你两种依赖方式都可以使用，构造器注入和 Setter 方法注入。最好的解决方案是用构造器

参数实现强制依赖，setter 方法实现可选依赖。



**SpringBeans**

**21.什么是 Springbeans?**

Springbeans 是那些形成 Spring 应用的主干的 java 对象。它们被 SpringIOC 容器初始化，

装配，和管理。这些 beans 通过容器中配置的元数据创建。比如，以 XML 文件中<bean/>的

形式定义。

Spring 框架定义的 beans 都是单件 beans。在 beantag 中有个属性”singleton”，如果它

被赋为 TRUE，bean 就是单件，否则就是一个 prototypebean。默认是 TRUE，所以所有在 Spring

框架中的 beans 缺省都是单件。

**22.一个 SpringBean 定义包含什么？**

一个 SpringBean 的定义包含容器必知的所有配置元数据，包括如何创建一个 bean，它的生

命周期详情及它的依赖。

**23.如何给 Spring 容器提供配置元数据?**

这里有三种重要的方法给 Spring 容器提供配置元数据。

XML 配置文件。

基于注解的配置。

基于 java 的配置。



**24.你怎样定义类的作用域?**

当定义一个<bean>在 Spring 里，我们还能给这个 bean 声明一个作用域。它可以通过 bean

定义中的 scope 属性来定义。如，当 Spring 要在需要的时候每次生产一个新的 bean 实例，

bean 的 scope 属性被指定为 prototype。另一方面，一个 bean 每次使用的时候必须返回同

一个实例，这个 bean 的 scope 属性必须设为 singleton。



#### **25.解释 Spring 支持的几种 bean 的作用域。**

- **singleton**: 单例模式，当spring创建applicationContext容器的时候，spring会欲初始化所有的该作用域实例，加上lazy-init就可以避免预处理；

- **prototype**：原型模式，每次通过getBean获取该bean就会新产生一个实例，创建后spring将不再对其管理；

  （下面是在web项目下才用到的）

- **request**：搞web的大家都应该明白request的域了吧，就是每次请求都新产生一个实例，和prototype不同就是创建后，接下来的管理，spring依然在监听；

- **session**: 每次会话，同上；

- **global session**: 全局的web域，类似于servlet中的application。



**26.Spring 框架中的单例 bean 是线程安全的吗?**

不，Spring 框架中的单例 bean 不是线程安全的。



**27.解释 Spring 框架中 bean 的生命周期。**

 Spring 容器从 XML 文件中读取 bean 的定义，并实例化 bean。 

 Spring 根据 bean 的定义填充所有的属性。

 如果 bean 实现了 BeanNameAware 接口，Spring 传递 bean 的 ID 到 setBeanName 方法。

 如果 Bean 实现了 BeanFactoryAware 接口，Spring 传递 beanfactory 给 setBeanFactory方法。

 如果有任何与 bean 相关联的 BeanPostProcessors，Spring 会在postProcesserBeforeInitialization()方法内调用它们。

 如果 bean 实现 IntializingBean 了，调用它的 afterPropertySet 方法，如果 bean 声明了初始化方法，调用此初始化方法。

 如果有 BeanPostProcessors 和 bean 关联，这些 bean 的postProcessAfterInitialization()方法将被调用。

 如果 bean 实现了 DisposableBean，它将调用 destroy()方法。

**28.哪些是重要的 bean 生命周期方法？你能重载它们吗？**

有两个重要的 bean 生命周期方法，第一个是 setup，它是在容器加载 bean 的时候被调用。

第二个方法是 teardown 它是在容器卸载类的时候被调用。

Thebean 标签有两个重要的属性（init-method 和 destroy-method）。用它们你可以自己定

制初始化和注销方法。它们也有相应的注解（@PostConstruct 和@PreDestroy）。

**29.什么是 Spring 的内部 bean？**

当一个 bean 仅被用作另一个 bean 的属性时，它能被声明为一个内部 bean，为了定义

innerbean，在Spring的基于XML的配置元数据中，可以在<property/>或<constructor-arg/>

元素内使用<bean/>元素，内部 bean 通常是匿名的，它们的 Scope 一般是 prototype。

**30.在 Spring 中如何注入一个 java 集合？**

Spring 提供以下几种集合的配置元素： 

 <list>类型用于注入一列值，允许有相同的值。

 <set>类型用于注入一组值，不允许有相同的值。

 <map>类型用于注入一组键值对，键和值都可以为任意类型。

 <props>类型用于注入一组键值对，键和值都只能为 String 类型。

**31.什么是 bean 装配?**

装配，或 bean 装配是指在 Spring 容器中把 bean 组装到一起，前提是容器需要知道 bean

的依赖关系，如何通过依赖注入来把它们装配到一起。

**32.什么是 bean 的自动装配？**

Spring 容器能够自动装配相互合作的 bean，这意味着容器不需要<constructor-arg>和

<property>配置，能通过 Bean 工厂自动处理 bean 之间的协作。

**33.解释不同方式的自动装配。**

有五种自动装配的方式，可以用来指导 Spring 容器用自动装配方式来进行依赖注入。

 **no**：默认的方式是不进行自动装配，通过显式设置 ref 属性来进行装配。

 **byName：**通过参数名自动装配，Spring 容器在配置文件中发现 bean 的 autowire 属性被

设置成 byname，之后容器试图匹配、装配和该 bean 的属性具有相同名字的 bean。 

 **byType：**通过参数类型自动装配，Spring 容器在配置文件中发现 bean 的 autowire 属

性被设置成 byType，之后容器试图匹配、装配和该 bean 的属性具有相同类型的 bean。

如果有多个 bean 符合条件，则抛出错误。

 **constructor：这个方式类似于** byType，但是要提供给构造器参数，如果没有确定的带

参数的构造器参数类型，将会抛出异常。

 **autodetect：**首先尝试使用 constructor 来自动装配，如果无法工作，则使用 byType 方

式。

**34.自动装配有哪些局限性?**

自动装配的局限性是：

 **重写**：你仍需用<constructor-arg>和<property>配置来定义依赖，意味着总要重写自动装配。

 **基本数据类型**：你不能自动装配简单的属性，如基本数据类型，String 字符串，和类。

 **模糊特性：**自动装配不如显式装配精确，如果有可能，建议使用显式装配。

**35.你可以在 Spring 中注入一个 null 和一个空字符串吗？**

**可以。**



**Spring 注解**

**36.什么是基于 Java 的 Spring 注解配置?给一些注解的例子.**

基于 Java 的配置，允许你在少量的 Java 注解的帮助下，进行你的大部分 Spring 配置而非通过 XML 文件。

以@Configuration 注解为例，它用来标记类可以当做一个 bean 的定义，被 SpringIOC 容器

使用。另一个例子是@Bean 注解，它表示此方法将要返回一个对象，作为一个 bean 注册进

Spring 应用上下文。

**37.什么是基于注解的容器配置?**

相对于 XML 文件，注解型的配置依赖于通过字节码元数据装配组件，而非尖括号的声明。

开发者通过在相应的类，方法或属性上使用注解的方式，直接组件类中进行配置，而不是使

用 xml 表述 bean 的装配关系。

**38.怎样开启注解装配？**

注解装配在默认情况下是不开启的，为了使用注解装配，我们必须在 Spring 配置文件中配

置<context:annotation-config/>元素。

**39.@Required 注解**

这个注解表明 bean 的属性必须在配置的时候设置，通过一个 bean 定义的显式的属性值或通

过自动装配，若@Required 注解的 bean 属性未被设置，容器将抛出BeanInitializationException。

**40.@Autowired 注解**

@Autowired 注解提供了更细粒度的控制，包括在何处以及如何完成自动装配。它的用法和

@Required 一样，修饰 setter 方法、构造器、属性或者具有任意名称和/或多个参数的 PN

方法。

**41.@Qualifier 注解**

当有多个相同类型的 bean 却只有一个需要自动装配时，将@Qualifier 注解和@Autowire 注

解结合使用以消除这种混淆，指定需要装配的确切的 bean。



**Spring 数据访问**

**42.在 Spring 框架中如何更有效地使用 JDBC?**

使用 SpringJDBC 框架，资源管理和错误处理的代价都会被减轻。所以开发者只需写

statements 和 queries 从数据存取数据，JDBC 也可以在 Spring 框架提供的模板类的帮助下

更有效地被使用，这个模板叫 JdbcTemplate（例子见这里 here）

**43.JdbcTemplate**

JdbcTemplate 类提供了很多便利的方法解决诸如把数据库数据转变成基本数据类型或对象，

执行写好的或可调用的数据库操作语句，提供自定义的数据错误处理。

**44.Spring 对 DAO 的支持**

Spring 对数据访问对象（DAO）的支持旨在简化它和数据访问技术如 JDBC，HibernateorJDO

结合使用。这使我们可以方便切换持久层。编码时也不用担心会捕获每种技术特有的异常。



**45.使用 Spring 通过什么方式访问 Hibernate?**

在 Spring 中有两种方式访问 Hibernate： 

 控制反转 HibernateTemplate 和 Callback。 

 继承 HibernateDAOSupport 提供一个 AOP 拦截器。

**46.Spring 支持的 ORM**

Spring 支持以下 ORM： 

 Hibernate

 iBatis

 JPA(JavaPersistenceAPI)

 TopLink

 JDO(JavaDataObjects)

 OJB

**47.如何通过 HibernateDaoSupport 将 Spring 和 Hibernate 结合起来？**

用 Spring 的 SessionFactory 调用 LocalSessionFactory。集成过程分三步：

 配置 theHibernateSessionFactory。 

 继承 HibernateDaoSupport 实现一个 DAO。 

 在 AOP 支持的事务中装配。

**48.Spring 支持的事务管理类型**

Spring 支持两种类型的事务管理：

 **编程式事务管理**：这意味你通过编程的方式管理事务，给你带来极大的灵活性，但是难维护。

 **声明式事务管理：**这意味着你可以将业务代码和事务管理分离，你只需用注解和 XML 配置来管理事务。

**49.Spring 框架的事务管理有哪些优点？**

 它为不同的事务 API 如 JTA，JDBC，Hibernate，JPA 和 JDO，提供一个不变的编程模式。

 它为编程式事务管理提供了一套简单的 API 而不是一些复杂的事务 API 如 

 它支持声明式事务管理。

 它和 Spring 各种数据访问抽象层很好得集成。

**50.你更倾向用那种事务管理类型？**

大多数 Spring 框架的用户选择声明式事务管理，因为它对应用代码的影响最小，因此更符

合一个无侵入的轻量级容器的思想。声明式事务管理要优于编程式事务管理，虽然比编程式

事务管理（这种方式允许你通过代码控制事务）少了一点灵活性。

​                  

**Spring 面向切面编程（AOP）**

**51.解释 AOP**

面向切面的编程，或 AOP，是一种编程技术，允许程序模块化横向切割关注点，或横切典型

的责任划分，如日志和事务管理。

**52.Aspect 切面**

AOP 核心就是切面，它将多个类的通用行为封装成可重用的模块，该模块含有一组 API 提供

横切功能。比如，一个日志模块可以被称作日志的 AOP 切面。根据需求的不同，一个应用程

序可以有若干切面。在 SpringAOP 中，切面通过带有@Aspect 注解的类实现。

**52.在 SpringAOP 中，关注点和横切关注的区别是什么？**

关注点是应用中一个模块的行为，一个关注点可能会被定义成一个我们想实现的一个功能。

横切关注点是一个关注点，此关注点是整个应用都会使用的功能，并影响整个应用，比如日

志，安全和数据传输，几乎应用的每个模块都需要的功能。因此这些都属于横切关注点。

**54.连接点**

连接点代表一个应用程序的某个位置，在这个位置我们可以插入一个 AOP 切面，它实际上是

个应用程序执行 SpringAOP 的位置。

**55.通知**

通知是个在方法执行前或执行后要做的动作，实际上是程序执行时要通过 SpringAOP 框架触

发的代码段。

Spring 切面可以应用五种类型的通知：

 **before**：前置通知，在一个方法执行前被调用。

 **after:**在方法执行之后调用的通知，无论方法执行是否成功。

 **after-returning:**仅当方法成功完成后执行的通知。

 **after-throwing:**在方法抛出异常退出时执行的通知。

 **around:**在方法执行之前和之后调用的通知。

**56.切点**

切入点是一个或一组连接点，通知将在这些位置执行。可以通过表达式或匹配的方式指明切

入点。

**57.什么是引入?**

引入允许我们在已存在的类中增加新的方法和属性。

**58.什么是目标对象?**

被一个或者多个切面所通知的对象。它通常是一个代理对象。也指被通知（advised）对象。

**59.什么是代理?**

代理是通知目标对象后创建的对象。从客户端的角度看，代理对象和目标对象是一样的。

**60.有几种不同类型的自动代理？**

BeanNameAutoProxyCreator

DefaultAdvisorAutoProxyCreator

Metadataautoproxying

**61.什么是织入。什么是织入应用的不同点？**

织入是将切面和到其他应用类型或对象连接或创建一个被通知对象的过程。

织入可以在编译时，加载时，或运行时完成。

**62.解释基于 XMLSchema 方式的切面实现。**

在这种情况下，切面由常规类以及基于 XML 的配置实现。

**63.解释基于注解的切面实现**

在这种情况下(基于@AspectJ 的实现)，涉及到的切面声明的风格与带有 java5 标注的普通

java 类一致。



**Spring 的 MVC**

**64.什么是 Spring 的 MVC 框架？**

Spring 配备构建 Web 应用的全功能 MVC 框架。Spring 可以很便捷地和其他 MVC 框架集成，

如 Struts，Spring 的 MVC 框架用控制反转把业务对象和控制逻辑清晰地隔离。它也允许以

声明的方式把请求参数和业务对象绑定。

**65.DispatcherServlet**

Spring 的 MVC 框架是围绕 DispatcherServlet 来设计的，它用来处理所有的 HTTP 请求和响

应。

**66.WebApplicationContext**

WebApplicationContext 继承了 ApplicationContext 并增加了一些 WEB 应用必备的特有功

能，它不同于一般的 ApplicationContext，因为它能处理主题，并找到被关联的 servlet。

**67.什么是 SpringMVC 框架的控制器？**

控制器提供一个访问应用程序的行为，此行为通常通过服务接口实现。控制器解析用户输入

并将其转换为一个由视图呈现给用户的模型。Spring 用一个非常抽象的方式实现了一个控

制层，允许用户创建多种用途的控制器。

**68.@Controller 注解**

该注解表明该类扮演控制器的角色，Spring 不需要你继承任何其他控制器基类或引用

ServletAPI。

**69.@RequestMapping 注解**

该注解是用来映射一个 URL 到一个类或一个特定的方处理法上。

**70.返回 Json 用什么注解？**

@ResponseBody

