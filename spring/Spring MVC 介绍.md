Spring MVC 提供了一个 DispatcherServlet 用于开发 Web 应用，在 Servlet2.5 及以下可用 web.xml 配置 `<servlet>` 元素即可

从 Servlet3.0+ 开始，在 Spring MVC 里实现 WebApplicationInitializer 接口便可实现等同与 web.xml 的配置

WebApplicationInitializer 是 Spring 提供用于配置 Servlet3.0+ 配置的接口，实现此接口将会自动被 SpringServletContainerInitializer（用来启动 Servlet3.0 容器）获取到

```
Maven

javaee-web-api 7.0
jstl 1.2
javax.servlet-api 3.1.0
jsp-api 2.2

slf4j-api
log4j
jcl-over-slf4j
logback-classic
logback-core
logback-access
```

```
public class WebInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(MyMvcConfig.class);
        ctx.setServletContext(servletContext);

        Dynamic servlet = servletContext.addServlet("dispatcher", new DispatcherServlet(ctx));
        servlet.addMapping("/");
        servlet.setLoadOnStartup(1);
    }
}
```

Spring Boot 的页面习惯放置方式在 `src/main/resources` 下，对应即 Maven 标准下的 `src/main/webapp/WEB-INF`，访问地址为 `/WEB-INF/classes`

```
@Configuration
@EnableWebMvc
@ComponentScan("com.lian")
public class SpringMvcConfig {
    @Bean
    public InternalResourceViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/classes/views/");
        viewResolver.setSuffix(".jsp");
        viewResolver.setViewClass(JstlView.class);
    }
}
```

@EnableWebMvc 注解会开启一些默认配置，如一些 ViewResolver 或 MessageConverter

Spring MVC 的 ViewResolver 是 Spring MVC 视图（jsp 下就是 html）渲染的核心机制

ViewResolver 的实现类都要实现 ViewResolver 接口，重写 resolveViewName() 方法，该方法返回值是接口 View，View 的职责是使用 model、request、response 对象，并将渲染的视图（html、json、xml、pdf……）返回给浏览器

## Spring MVC 基本配置
Spring MVC 的定制配置需要配置类继承一个 WebMvcConfigurerAdapter 类，并在此类使用 @EnableWebMvc 注解，来开启对 Spring MVC 的配置支持

WebMvcConfigurerAdapter 类也实现了 WebMvcConfigurer 接口，所以 WebMvcConfigurer 的 API 内方法也可以用来配置 MVC

### 静态资源映射

```
@Configuration
@EnableWebMvc
@ComponentScan("com.lian")
public class MyMvcConfig extends WebMvcConfigureAdapter {

    // 静态资源映射
    @Override
    public void addResourceHandler(ResourceHandlerRegistry registry) {

        // addResourceHandler 指文件的放置位置，src/main/resources/assets/
        // addResourceLocations 指对外暴露的访问路径
        registry.addResourceHandler("/assets/**").addResourceLocations("classpath:/assets/");
    }
}
```

### 拦截器
```
@Configuration
@EnableWebMvc
@ComponentScan("com.lian")
public class MyMvcConfig extends WebMvcConfigureAdapter {

    // 注册自定义拦截器类
    @Bean
    public DemoInterceptor demoInterceptor() {
        return new DemoInterceptor();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(demoInterceptor());
    }
}
```

### @ControllerAdvice
通过 @ControllerAdvice 可以将对于控制器的全局配置（@ExceptionHandler、@InitBinder、@ModelAttribute）放置到同一个位置，即对全局有效

@Controller 类中可用 @ExceptionHandler、@InitBinder、@ModelAttribute 注解一个方法，对于该控制器内注解了 @RequestMapping 的方法有效

@ExceptionHandler 用于处理异常

@ModelAttribute 用于绑定键值到 Model

@InitBinder 用于设置 WebDataBinder，WebDataBinder 用于自动绑定前台请求参数到 Model 中

```
@ControllerAdvice
public class ExceptionHandlerAdvice {

    @ExceptionHandler(value = Exception.class)
    public ModelAndView exception(Exception exception, WebRequest request) {
        ModelAndView modelAndView = new ModelAndView("error");
        modelAndView,addObject("errorMessage", exception,getMeaasge());
        return modelAndView;
    }

    @ModelAttribue
    public void addAttribute(Model model) {
        model.addAttribute("msg", "msg from ex");
    }

    @InitBinder
    public void initBinder(WebDataBinder webDataBinder) {

        // 忽略 request param 的 id
        webDataBinder.setDisallowedField("id");
    }
}
```

### 快捷的 ViewController
如果如任何页面处理，只是简单的页面转向
```
@Configuration
@EnableWebMvc
@ComponentScan("com.lian")
public class MyMvcConfig extends WebMvcConfigureAdapter {

    @Override
    public void addViewController(ViewControllerRegistry registry) {
        registry.addViewController("/index").setViewName("/index");
    }
}
```
等同于在 @Controller 控制器中
```
@RequestMapping("/index")
public String hello() {
    return "index";
}
```

### 路径匹配参数配置
在 Spring MVC 中，路径参数如果带 . 的话，. 及后面的值将会被忽略

通过重写 configurePathMatch 方法可不忽略

```
@Configuration
@EnableWebMvc
@ComponentScan("com.lian")
public class MyMvcConfig extends WebMvcConfigureAdapter {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer.setUseSuffixPatternMatch(false);
    }
}
```

## Spring MVC 高级配置

### 文件上传配置
Spring MVC 通过配置一个 MultipartResolver 来上传文件

在控制器中，通过 MultipartFile 对象来接收文件，通过 MultipartFile[] 来接收多个文件上传

```
Maven

commons-fileupload
commons-io
```
```
@Bean
public MultipartResolver multipartResolver() {
    CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver();
    multipartResolver.setMaxUploadSize(1000000);
    return multipartResolver;
}
```
```
@RequestMapping(value="/upload", method=RequestMethod.POST)
public @ResponseBody String upload(MultipartFile file) {
    try {
        FileUtils.writeByteArrayToFile(new File("D:/upload/" + file.getOriginalFilename()), file.getBytes());
        return "ok";
    } catch (IOException e) {
        return "error";
    }
}
```

### 自定义 HttpMessageConverter
HttpMessageConverter 是用来处理 request 和 response 里的数据的

Spring 内置了大量的 HttpMessageConverter

```
public class MyHttpMessageConverter extends AbstractHttpMessageConverter<DemoObject> {

    public MyHttpMessageConverter() {

        // 处理自定义的媒体类型 application/x-lianwx
        super(new MediaType("application", "x-lianwx", Charset.forName("UTF-8")));
    }

    // 自定义 HttpMessageConverter 支持的类
    @Override
    protected boolean supports(Class<?> clazz) {

        // 表明本 HttpMessageConverter 只处理 DemoObject 这个类
        return DemoObject.class.isAssignableFrom(clazz);
    }

    // 重写 readInternal 方法处理 request 数据
    @Override
    protected DemoObject readInternal(Class<? extends DemoObject> clazz, HttpInputMessage inputMeaasge) throws IOException, HttpMessageNotReadableException {

        String temp = StreamUtils.copyToString(inputMeaasge.getBody(), Charset.forName("UTF-8"));
        String[] tempArr = temp.split("-");
        return new DemoObject(new Long(tempArr[0]), tempArr[1]);
    }

    // 重写 writeInternal 方法处理 response 数据
    @Override
    protected void writeInternal(DemoObject obj, HttpOutputMessage outputMeaasge) throws IOException, HttpMessageNotReadableException {

        String out = obj.getId() + "-" + obj.getName();
        outputMeaasge.getBody().write(out.getBytes());
    }
}
```

在 Spring MVC 里注册 HttpMessageConverter 有两种方法：
* configureMessageConverters 会覆盖掉 Spring MVC 默认注册的 HttpMessageConverter
* extendsMessageConverters 仅添加一个自定义的 HttpMessageConverter

```
@Configuration
@EnableWebMvc
@ComponentScan("com.lian")
public class MyMvcConfig extends WebMvcConfigureAdapter {

    @Bean
    public MyHttpMessageConverter myHttpMessageConverter() {
        return new MyHttpMessageConverter();
    }

    @Override
    public void extendsMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.add(myHttpMessageConverter());
    }
}
```

### 服务器端推送技术
服务器端推送技术的解决方案有：
* 前端使用 Ajax 对服务器轮询消息    
    缺点：轮询频率不好控制，大大增加了服务器的压力
* 当客户端向服务器端发送请求，服务器端会抓住这个请求不放，等有数据更新的时候才返回客户端，当客户端接收到消息后，再向服务器端发送请求   
    * 基于 SSE（Server Send Event 服务器发送事件）的服务器端推送   
        ```
        @Controller
        public class SseController {

            // 媒体类型 text/event-stream 是服务器端的 SSE 支持
            @RequestMapping(value="push", produces="text/event-stream")
            public @ResponseBody String push() {
                Random r = new Random();
                try {
                    Thread.sleep(5000);
                } catch(InterruptedException e) {
                    e.printStackTrace();
                }
                return "test: " + r.nextInt() + "\n\n";
            }
        }
        ```
        ```
        // EventSource 是 SSE 客户端，只有新式的浏览器有
        if (!window.EventSource) {
            var source = new EventSource('push');
            s = '';
            source.addEventListener('message', function(e) {
                s = e.data + "<br/>";
                $("msgFromPush").html(s);
            });
            source.addEventListener('open', function(e) {
                console.log("连接打开");
            }, false);
            source.addEventListener('error', function(e) {
                if (e.readyState == EventSource.CLOSED) {
                    console.log("连接关闭");
                } else {
                    console.log(e.readyState);
                }
            }, false);
        } else {
            console.log("浏览器不支持");
        }
        ```
    * 基于 Servlet3.0+ 的异步方法特性   
        DispatcherServlet 增加异步方法支持
        ```
        public class WebInitializer implements WebApplicationInitializer {

            @Override
            public void onStartup(ServletContext servletContext) throws ServletException {
                AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
                ctx.register(MyMvcConfig.class);
                ctx.setServletContext(servletContext);

                Dynamic servlet = servletContext.addServlet("dispatcher", new DispatcherServlet(ctx));
                servlet.addMapping("/");
                servlet.setLoadOnStartup(1);

                // 开启异步方法支持
                servlet.setAsyncSupported(true);
            }
        }
        ```
        异步任务的实现返回一个 DefferedResult
        ```
        @Controller
        public class AsyncController {
            @Autowired
            PushService pushService;

            @RequestMapping("/defer")
            @ResponseBody
            public DefferedResult<String> deferredCall() {
                return pushService.getAsyncUpdate();
            }
        }
        ```
        在计划任务里更新 DefferedResult
        ```
        @Service
        public class PushService {
            private DeferredResult<String> deferredResult;

            public DeferredResult<String> getAsyncUpdate() {
                deferredResult = new DeferredResult<String>();
                return deferredResult;
            }

            @Scheduled(fixedDelay=5000)
            public void refresh() {
                if (deferredResult != null) {
                    deferredResult.setResult(new Long(System.currentTimeMillis()).toString());
                }
            }
        }
        ```
        ```
        deferred();

        function deferred() {
            $.get('defer', function(data) {
                console.log(data);
                // 一次请求完成后再次向后台发送请求
                deferred();
            })
        }
        ```
* 双向通信技术 —— WebSocket
    WebSocket 为浏览器和服务器端提供了双工异步通信技术，WebSocket 是通过一个 socket 来实现双工异步通信技术    
    直接使用 WebSocket（或者 SockJS）协议开发显得特别繁琐    
    WebSocket 的子协议 STOMP 是一个更高级别的协议，它使用一个基于帧(frame)的格式来定义消息
    * 广播式    
        广播式即服务器端有消息时，会将消息发送给所有连接了当前 endpoint（节点）的浏览器   
        配置 WebSocket 需要在 Java 配置类上使用 @EnableWebSocketMeaasgeBroker 开启 WebSocket 支持，并继承 AbstractWebSocketMessageBrokerConfigurer 类
        ```
        @Configuration

        // 通过 @EnableWebSocketMeaasgeBroker 注解开启使用 STOMP 协议来传输基于代理（message broker）的消息
        @EnableWebSocketMeaasgeBroker
        public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

            @Override
            public void registerStompEndpoints(StompEndpointRegistry registry) {

                // 注册 STOMP 协议的 endpoint，并映射指定的 url，指定使用 SockJS 协议
                registry.addEndPoint("/endpointTest").withSockJS();
            }

            @Override
            public void configureMessageBroker(MessageBrokerRegistry registry) {

                // 配置消息代理
                registry.enableSimpleBroker("/topic");
            }
        }
        ```
        ```
        @Controller
        public class WsController {

            // 类似于 @RequestMapping，当浏览器向服务器端发送请求时，映射到 /welcome 这个地址
            @MessageMapping("/welcome")
            // 当服务器端有消息时，会向订阅了 /topic/getResponse 这个地址的浏览器发送消息
            @SendTo("/topic/getResponse")
            // TestResponse 和 TestMessage 均为 POJO
            public TestResponse say(TestMessage message) throws Exception {
                Thread.sleep(3000);
                return new TestResponse("welcome, " + message.getName());
            }
        }
        ```
        前端需要添加脚本 stomp.min.js（STOMP 协议的客户端脚本）和 sockjs.min.js（SockJS 协议的客户端脚本）
        ```
        var stompClient = null;

        function connect() {
            // 连接 SockJS 的 endpoint 名为 /endpointTest
            var socket = new SockJS('/endpointTest');
            // 使用 STOMP 子协议的 WebSocket 客户端
            stompClient = Stomp.over(socket);
            // 连接 WebSocket 服务端
            stompClient.connect({}, function(frame) {
                console.log('connected: ' + frame);
                // 订阅 /topic/getResponse 目标发送的消息
                stompClient.subscribe('/topic/getResponse', function(response) {
                    console.log('response: ' + response);
                });
            })
        }

        function sendName(name) {
            // 向 /welcome 目标发送消息
            stompClient.send("/welcome", {}, JSON.stringify({'name':name}));
        }

        function disconnect() {
            if (stompClient != null) {
                stompClient.disconnect();
            }
            console.log('disconnected');
        }
        ```
    * 点对点式
        点对点式是解决广播式不能解决的问题：消息由谁发送，由谁接收    
        ```
        @Configuration
        @EnableWebSocketMeaasgeBroker
        public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

            @Override
            public void registerStompEndpoints(StompEndpointRegistry registry) {

                registry.addEndPoint("/endpointTest").withSockJS();
                registry.addEndPoint("/endpointChat").withSockJS();
            }

            @Override
            public void configureMessageBroker(MessageBrokerRegistry registry) {

                // 配置消息代理
                registry.enableSimpleBroker("/topic", "/queue");
            }
        }
        ```
        ```
        @Controller
        public class WsController {

            // 通过 SimpMessagingTemplate 向浏览器发送消息
            @Autowired
            private SimpMessagingTemplate messagingTemplate;

            // Principal 是 Spring Security 的一个类，通过 Principal 可获得当前用户的信息
            @MessageMapping("/chat")
            public void handleChat(Principal principal, String msg) {
                if (principal.getName().equals("aaa")) {

                    // messagingTemplate.convertAndSendToUser 向用户发送消息，参数为 接收用户，订阅地址，消息内容
                    messagingTemplate.convertAndSendToUser("bbb", "/queue/notifications", principal.getName() + " send :" + msg);
                } else {
                     messagingTemplate.convertAndSendToUser("aaa", "/queue/notifications", principal.getName() + " send :" + msg);
                }
            }
        }
        ```
