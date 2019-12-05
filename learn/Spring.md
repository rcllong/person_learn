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


## SpringBoot的自动配置:
### 主要通过@EnableAutoConfiguration,@Conditional,@EnableConfigurationProperties或者@ConfigurationProperties等几个注解来进行自动配置完成的
* @EnableAutoConfiguration:开启自动配置,主要作用就是调用Spring-Core包里的loadFactoryName(),将autoconfig包里的写好的自动配置加载进来
* @Conditional:条件注解,通过判断类路径下有没有相应配置的jar包来确定是否加载和自动配置这个类
* @EnableConfigurationProperties:给自动配置提供具体的配置参数,只需要写在application.properties中,就可以通过映射写入配置类的POJO属性中

#### BeanFactory是Spring中比较原始的Factory,无法支持Spring的插件

* BeanFactory是接口,提供了IOC容器最基本的形式
* FactoryBean也是接口,为IOC容器中Bean的实现提供了更加灵活的方式
* 在Spring中,所有的Bean都是由BeanFactory来进行管理的
* FactoryBean是一个能生产或者修饰对象生成的工厂Bean
* 在Spring中,BeanFactory是IOC容器的核心接口,他的职责包括:实例化,定位,配置应用程序中的对象以及建立这些对象间的依赖
* 一般情况下,Spring通过反射机制利用<bean>的class属性指定实现类实例化Bean,Spring提供了FactoryBean的工厂类接口,用户可以通过实现该接口定制实例化Bean的逻辑
