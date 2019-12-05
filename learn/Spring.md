### Spring MVC运行流程
* 用户发送请求至前端控制器 DispatcherServlet
* DispatcherServlet收到请求调用HandlerMapping处理器映射器
* 处理器映射器根据请求url找到具体的处理器,生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet
* DispatcherServlet通过HandlerAdapter处理器适配器调用处理器
* 执行处理器(Controller)
* Controller执行完返回ModelAndView
* HandlerAdapter将Controller执行结果ModelAndView返回给DispatcherServlet
* DispatcherServlet将ModelAndView传给ViewResolver视图解析器
* ViewResolver解析后返回具体View
* DispatcherServlet对View进行渲染视图(即将模型数据填充至视图中)
* DispatcherServlet响应用户

### Spring中Bean的Scope 
* Singleton: 默认  整个Bean容器中只存在一个实例
* Prototype:  多例类型,每次从bean容器中都会获取到一个对应的新实例
* Request: 仅适用于Web环境下的ApplicationContext,表示每个HttpRequest的生命周期都会有一个单独的实例
* Session: 仅适用于Web环境下的ApplicationContext,表示每个HttpSession生命周期只会有一个单独的实例
* globalSession: 表示每一个全局的HttpSession下都会拥有一个单独的实例
* application:   仅适用于Web环境下的ApplicationContext,表示在ServletContext生命周期内会拥有一个单独的实例



