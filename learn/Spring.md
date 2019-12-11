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



### 自定义 HTTP Servlet（服务端提供接口）

* 首先写一个类 继承 **HttpServlet**  并重写里面的方法
```java
public class PersonController extends HttpServlet {
     /**
      * get方法
      * @param request
      * @param response
     */
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //get方法
    }
    
    /**
     * post方法
     * @param request
     * @param response
     */
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //读取请求的Body
        String s = ReadAsChars2(request);
        System.out.println("body内容：",s);
    }

    /**
     * 读取请求body
     * @param request
     * @return
     */
    private String ReadAsChars2(HttpServletRequest request) {
        InputStream is = null;
        StringBuilder sb = new StringBuilder();
        try {
            is = request.getInputStream();

            byte[] b = new byte[4096];
            for (int n; (n = is.read(b)) != -1; ) {
                sb.append(new String(b, 0, n));
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (null != is) {
                try {
                    is.close();

                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return sb.toString();
    }
}
```
* 其次 在SpringBoot项目中 自定义Configuration，指定接口的路径
```java
public class PersonBeanConfiguration {
    @Bean
    public ServletRegistrationBean getServletRegistrationBean () {
        //绑定Servlet
        ServletRegistrationBean bean = new ServletRegistrationBean(new PersonController());
        //添加服务路径
        bean.addUrlMappings("/personTestUrl");
        return bean;
    }
    
    /**
     * 创建FeignClient
     */
    @Bean
    @ConditionalOnMissingBean
    public Client feignClient(CachingSpringLoadBalancerFactory cachingFactory,
                              SpringClientFactory clientFactory) {
        return new LoadBalancerFeignClient(new Client.Default(null, null),
                cachingFactory, clientFactory);
    }
}

```

### 客户端动态创建FeignClient通过服务名调用服务
* 首先自定义接口  注意： 动态创建的FeignClient 不用加注解@FeignClient("SERVICE-ID")
```java
/**
 * @author: RenChengLong
 */
public interface PersonFeignClient {
    /**
     * 测试
     * @param map 请求的body
     */
    @RequestMapping(value = "/personTestUrl", method = RequestMethod.POST)
    void personTest(@RequestBody Map<String,String> map);
}

```

* 其次 在调用方法中 加入此FeignClient
```java
//导入SpringCloud默认的Feign配置
@Import(FeignClientsConfiguration.class)
public class PersonTest{

    //自定义的FeignClient
    PersonFeignClient personFeignClient;

    //Feign 原生构造器
    Feign.Builder builder;
        
    //创建构造器
    public PersonTest(Decoder decoder, Encoder encoder, Client client, Contract contract) {
        this.builder = Feign.builder().client(client).encoder(encoder).decoder(decoder).contract(contract);
    }    

    public void test() {
        String serviceId="PERSON_TEST";
        Map<String, String> map = new HashMap<>();   
        map.put("parameter1","test1");
        //注意  构建的url  必须添加  http://   
        this.personFeignClient = this.builder.target(PersonFeignClient.class, "http://" + serviceId);
        
        //使用FeignClient请求服务
         this.personFeignClient.personTest(map);
    }

}
```