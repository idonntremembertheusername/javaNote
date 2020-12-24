

#### 什么是MVC

​    MVC(Model View Controller)是一种软件设计的**框架模式**，它采用**模型(Model)-视图(View)-控制器(controller)的方法把业务逻辑、数据与界面显示分离。**

------

#### 处理流程

1. Web 容器启动时初始化 IoC 容器，加载 Bean 的定义信息并初始化所有单例 Bean，遍历容器中的 Bean，获取每个 Controller 中的所有方法访问的 URL，将 URL 和对应的 Controller 保存到一个 Map 集合中。

2. 所有的请求会转发给 DispatcherServlet 处理，DispatcherServlet 会请求 HandlerMapping 找出容器中被 `@Controller` 修饰的 Bean 以及被 `@RequestMapping` 修饰的方法和类，生成 Handler 和 HandlerInterceptor 并以一个 HandlerExcutionChain 链的形式返回。

3. DispatcherServlet 使用 Handler 找到对应的 HandlerApapter，通过 HandlerApapter 调用 Handler 的方法，将请求参数绑定到方法的形参上，执行方法处理请求并得到逻辑视图 ModelAndView。

4. 使用 ViewResolver 解析 ModelAndView 得到物理视图 View，进行视图渲染，将数据填充到视图中并返回给客户端。

------

#### 组件

`DispatcherServlet`：前端控制器，整个流程控制的核心，负责接收请求并转发给对应的处理组件。

`Handler`：处理器，完成具体业务逻辑。

`HandlerMapping`：处理器映射器，完成 URL 到 Controller 映射。

`HandlerInterceptor`：处理器拦截器，如果需要完成拦截处理可以实现该接口。

`HandlerExecutionChain`：处理器执行链，包括 Handler 和 HandlerInterceptor。

`HandlerAdapter`：处理器适配器，DispatcherServlet 通过 HandlerAdapter 来执行不同的 Handler。

`ModelAndView`：逻辑视图，装载模型数据信息。

`ViewResolver`：视图解析器，将逻辑视图解析为物理视图。

------

#### 常用注解

`@RequtestMapping`：将 URL 请求和方法映射起来，在类和方法定义上都可以添加。

> `value` 属性指定 URL 请求的地址。
>
> `method` 属性限制请求类型，如果没有使用指定方法请求 URL，会报 405 错误。
>
> `params` 属性限制必须提供的参数。

`@RequestParam`：如果 Controller 方法的形参和 URL 参数名不一致可以使用该注解绑定。

> `value` 属性表示 HTTP 请求中的参数名。
>
> `required` 属性设置参数是否必要，默认false。
>
> `defaultValue` 属性指定没有给参数赋值时的默认值。

`@PathVariable`：Spring MVC 支持 RESTful 风格 URL，通过 `@PathVariable`  可以将URL中占位符参数{xxx}绑定到处理器类的方法形参中@PathVariable(“xxx“) 。

```java
@RequestMapping("login/{username}/{passowrd}")
public ModelAndView login(@PathVariable("username") String username,
                          @PathVariable("passowrd") String passowrd){
        ModelAndView mv = new ModelAndView();
        mv.addObject("msg","占位符映射：id:"+ids+";name:"+names);
        mv.setViewName("hello2");
        return mv;
}

```

------

####  **SpringMVC 完成 JSON 操作：** 

> ①. 配置 MappingJacksonHttpMessageConverter
>
> ②. 使用 @RequestBody 注解或 ResponseEntity 作为返回值

