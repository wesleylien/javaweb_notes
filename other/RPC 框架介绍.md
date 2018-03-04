## RPC 基础知识
是什么：
* 是一个协议，程序可使用该协议请求网络中另一台计算机上的某程序的服务，而不需要知道网络细节
* C/S 模式
* 基于传输层协议( 如 TCP/IP 协议 )
* 事件处理模型 (请求、计算、响应)

目的：
* 调用非本机方法
* 不同程序语言之间通信
* 不了解底层通信，像本地方法一样调用

作用：   
* 分布式程序的基础（分布式操作系统，分布式计算，分布式软件设计）
* 垂直应用服务化拆分

特点：
* 封装网络交互
* 远程调用的对象代理
* 支持容器( Spring 、Jetty 等 )
* 可配置、可扩展

## RPC 调用过程


## RPC 框架的设计
有服务提供方即生产者( provider )、服务调用方即消费者( consumer )

消费者：代理层 proxy、序列化层 serialize、通信层 invoke 、容器 Container

生产者：代理层、序列化层、通信层 、容器

## HTTP 协议的 RPC 框架

**序列化层**：consumer 将 请求的类、类中对应的方法、方法对应的参数 进行序列化(1)并通过通信层传到 provider ，provider 将得到的经过序列化的东西进行反序列化(2)，得到 请求的类、类中对应的方法、方法对应的参数 ，从而进行处理，再将处理结果进行序列化(3)，通过通信层传回 consumer ，consumer 将传回的经过序列化的结果进行反序列化(4)

在过程中，这里需要进行序列化和反序列化的步骤有4个，可以定义为4个方法：
```
(1):
/**
 * @param clazz 请求的接口
 * @param method 请求的方法
 * @param param 请求的参数
 * @return
 */
public String consumerRequestFormat(Class clazz,String method,Object param);

(2):
/**
 * @param param 请求参数
 * @return 这里 Request 类封装了 请求的类、请求的方法和请求的参数
 */
public Request providerRequestParse(String param) throws RpcException;

(3):
/**
 * @param param 响应的结果
 * @return
 */
public String providerResponseFormat(Object param);

(4):
/**
 *
 * @param result
 * @return
 */
public <T> T consumerResponseParse(String result);
```
我们将方法分别定义到两个接口：Formatter 和 Parser，并选择用 JSON 协议格式实现该接口（用到 fastjson 包）
```
public class JsonFormatter implements Formatter {

    public static final Formatter formatter = new JsonFormatter();

    public String consumerRequestFormat(Class clazz, String method, Object param) {

        Request request = new Request();
        request.setParam(param);
        request.setClazz(clazz);
        request.setMethod(method);
        return JSON.toJSONString(request, SerializerFeature.WriteClassName);
    }

    public String providerResponseFormat(Object param) {

        return JSON.toJSONString(param, SerializerFeature.WriteClassName);
    }

}
```
```
public class JsonParser implements Parser {

    private static final Logger logger = LoggerFactory.getLogger(JsonParser.class);

    public static final Parser parser = new JsonParser();

    public Request providerRequestParse(String param) throws RpcException {

        try {

            logger.debug("调用参数 {}", param);
            return (Request)JSON.parse(param);
        }
        catch (Exception e) {

            logger.error("转换异常 param = {}", param, e);
            throw new RpcException("",e, RpcExceptionCodeEnum.DATA_PARSER_ERROR.getCode(),param);
        }
    }

    public <T> T consumerResponseParse(String result) {

        return (T)JSON.parse(result);
    }
}
```
**通信层**：consumer 将序列化后的 请求的类、类中对应的方法、方法对应的参数 发送到 provider ，provider 将结果序列化后返回，是一次通信的过程。这里采用 C/S 模式，一般传输层为 TCP， consumer 作为发起请求方，需要知道 provider 的 IP 和端口。provider 作为服务器端，可通过 web 容器来接收处理请求

我们定义了一个接口 Invoker，用于处理通讯层
```
public interface Invoker {
    /**
     * consumer 发起请求调用并返回结果的过程
     * @param request 序列化后的请求报文
     * @param consumerConfig consumer 的配置，主要为 provider 的 url
     * @return provider 返回的序列化后的结果
     * @throws RpcException
     */
    public String request(String request,ConsumerConfig consumerConfig) throws RpcException;

    /**
     * provider 收到请求，将 provider 的结果从请求响应的输出流输出的简单封装
     * @param response 响应报文
     * @param outputStream 输出流
     * @throws RpcException
     */
    public void response(String response,OutputStream outputStream) throws RpcException;
}
```
采用 HTTP 实现这一通信层（这里用到 Apache 的 HttpClient 包）
```
public class HttpInvoker implements Invoker {

    // consumer 发起 HTTP 请求需要 Http 客户端
    private static final HttpClient httpClient = getHttpClient();

    public static final Invoker invoker = new HttpInvoker();

    public String request(String request, ConsumerConfig consumerConfig) throws RpcException {

        // 设置要请求的 provider 的 url
        HttpPost post = new HttpPost(consumerConfig.getUrl());
        post.setHeader("Connection", "Keep-Alive");

        // 设置请求的 parameter 的 key 为 data 的 value 为 consumer 序列化后的请求参数
        List<NameValuePair> params = new ArrayList<NameValuePair>();
        params.add(new BasicNameValuePair("data", request));
        try {
            post.setEntity(new UrlEncodedFormEntity(params,HTTP.UTF_8));

            // consumer 发起请求
            HttpResponse response = httpClient.execute(post);

            // 成功
            if (response.getStatusLine().getStatusCode() == 200) {

                // 返回 provider 返回的序列化后的结果
                return EntityUtils.toString(response.getEntity(),"UTF-8");
            }

            throw new RpcException(RpcExceptionCodeEnum.INVOKE_REQUEST_ERROR.getCode(),request);
        }
        catch (Exception e) {
            throw new RpcException("http 调用异常",e, RpcExceptionCodeEnum.INVOKE_REQUEST_ERROR.getCode(),request);
        }
    }

    public void response(String response, OutputStream outputStream) throws RpcException {

        try {
            outputStream.write(response.getBytes("UTF-8"));
            outputStream.flush();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    /**
     * 设置并返回一个 Http 客户端
     */
    public static HttpClient getHttpClient() {
        PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager();
        //连接池最大生成连接数200
        cm.setMaxTotal(200);
        // 默认设置route最大连接数为20
        cm.setDefaultMaxPerRoute(20);
        // 指定专门的route，设置最大连接数为80
        HttpHost localhost = new HttpHost("localhost", 8080);
        cm.setMaxPerRoute(new HttpRoute(localhost), 50);
        // 创建httpClient
        return HttpClients.custom()
                .setConnectionManager(cm)
                .build();
    }

    public static String EntityToString(final HttpEntity entity, final Charset defaultCharset) throws IOException, ParseException {

        Args.notNull(entity, "Entity");
        final InputStream instream = entity.getContent();
        if (instream == null) {
            return null;
        }
        try {
            Args.check(entity.getContentLength() <= Integer.MAX_VALUE,
                    "HTTP entity too large to be buffered in memory");
            int i = (int)entity.getContentLength();
            if (i < 0) {
                i = 4096;
            }
            Charset charset = null;
            try {
                final ContentType contentType = ContentType.get(entity);
                if (contentType != null) {
                    charset = contentType.getCharset();
                }
            } catch (final UnsupportedCharsetException ex) {
                throw new UnsupportedEncodingException(ex.getMessage());
            }
            if (charset == null) {
                charset = defaultCharset;
            }
            if (charset == null) {
                charset = HTTP.DEF_CONTENT_CHARSET;
            }
            final Reader reader = new InputStreamReader(instream, charset);
            final CharArrayBuffer buffer = new CharArrayBuffer(i);
            final char[] tmp = new char[1024];
            int l;
            while((l = reader.read(tmp)) != -1) {
                buffer.append(tmp, 0, l);
            }
            return buffer.toString();
        } finally {
//            instream.close();
        }
    }
}
```
**代理层**：对于 consumer ，我们需要的是通过动态代理，返回结果，像方法的处理过程在本机执行一样，因此 consumer 的代理类可以实现动态代理中很重要的 InvocationHandler 接口，并在 invoke 方法中进行 请求的类、类中对应的方法、方法对应的参数 的序列化，对 provider 发起请求并得到返回结果，对结果进行反序列化，最后返回反序列化后的结果
```
public class ConsumerProxyFactory implements InvocationHandler {

    //
    private String clazz;
    //
    private ConsumerConfig consumerConfig;
    // 获取 JsonParser 实例
    private Parser parser = JsonParser.parser;
    // 获取 JsonFormatter 实例
    private Formatter formatter = JsonFormatter.formatter;
    // 获取 HttpInvoker 实例
    private Invoker invoker = HttpInvoker.invoker;

    // 利用 Proxy 类动态代理返回代理对象
    public Object create() throws Exception {

        Class interfaceClass = Class.forName(clazz);
        return Proxy.newProxyInstance(interfaceClass.getClassLoader(), new Class[]{interfaceClass}, this);
    }

    // 当代理对象调用方法时执行
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        Class interfaceClass = proxy.getClass().getInterfaces()[0];
        // 序列化
        String request = formatter.consumerRequestFormat(interfaceClass, method.getName(), args[0]);
        // 请求并返回
        String response = invoker.request(req,consumerConfig);
        // 反序列化
        return parser.consumerResponseParse(response);
    }

    // 下面为 getter 和 setter 方法，省略
}
```
对于 provider ，需要的是将收到的请求反序列化后得到 请求的类、类中对应的方法、方法对应的参数 ，并利用反射执行响应的类的方法，最后将处理结果序列化后返回，这些步骤可在 HTTP 收到请求后的处理 handler 中实现。因此 provider 的代理类继承 AbstractHandler ( Jetty 容器的处理 handler ，这里 provider 采用 jetty 作为服务器端的 web 容器 )
```
public class ProviderProxyFactory extends AbstractHandler {

    private static final Logger logger = LoggerFactory.getLogger(ProviderProxyFactory.class);

    // provider 需要将能提供的实例进行注册，这里用一个 Map 对象保存注册结果
    private Map<Class,Object> providers = new ConcurrentHashMap<Class, Object>();

    private static ProviderProxyFactory factory;
    // 获取 JsonParser 实例
    private Parser parser = JsonParser.parser;
    // 获取 JsonFormatter 实例
    private Formatter formatter = JsonFormatter.formatter;
    // 获取 HttpInvoker 实例
    private Invoker invoker = HttpInvoker.invoker;

    /**
     * 构造器，采用默认 config
     *
     */
    public ProviderProxyFactory(Map<Class,Object> providers) {

        if (Container.container == null) {

            new HttpContainer(this).start();
        }
        for (Map.Entry<Class,Object> entry: providers.entrySet()){

            register(entry.getKey(), entry.getValue());
        }

        factory = this;
    }

    /**
     * 构造器
     * ProviderProxyFactory 实例在 provider 上由 Spring 容器负责创建
     * ProviderProxyFactory 实例初始化做两件事：1.启动 Container，接收请求；2.注册服务
     * ProviderConfig 为 provider 的 Container 配置信息，有可供请求的 url，端口
     */
    public ProviderProxyFactory(Map<Class,Object> providers, ProviderConfig providerConfig) {

        // 创建 HttpContainer 实例，HttpContainer 负责启动 Jetty
        if (Container.container == null) {

            new HttpContainer(this, providerConfig).start();
        }
        // 将能服务的类注册到 Map 中
        for (Map.Entry<Class,Object> entry: providers.entrySet()) {

            register(entry.getKey(), entry.getValue());
        }

        factory = this;
    }

    public static ProviderProxyFactory getInstance() {

        return factory;
    }

    public void register(Class clazz, Object object) {

        providers.put(clazz, object);
        logger.info("{} 已经发布", clazz.getSimpleName());
    }


    public void handle(String target, HttpServletRequest request, HttpServletResponse response, int dispatch) throws IOException, ServletException {

        String reqStr = request.getParameter("data");
        try {
            // 将请求参数解析
            Request rpcRequest = parser.providerRequestParse(reqStr);
            // 反射请求
            Object result = rpcRequest.getClazz.getMethod(rpcRequest.getMethod, rpcRequest.getParam().getClass()).invoke(ProviderProxyFactory.getInstance().getBeanByClass(rpcRequest.getClazz()), rpcRequest.getParam());
            // 相应请求
            invoker.response(formatter.providerResponseFormat(result),response.getOutputStream());

        } catch (RpcException e) {

            e.printStackTrace();
        } catch (Throwable e) {

            e.printStackTrace();
        }
    }

    /**
     * 根据 Class 在所注册的实例中查找实例
     */
    public Object getBeanByClass(Class clazz) throws RpcException {

        Object bean =  providers.get(clazz);
        if (bean != null) {

            return bean;
        }
        throw new RpcException(RpcExceptionCodeEnum.NO_BEAN_FOUND.getCode(),clazz);
    }
}
```
**容器**：provider 需要一个 Web 容器用于作服务器端接收 consumer 请求

先定义一个抽象类
```
public abstract class Container {

    public static volatile boolean isStart = false;

    public abstract void start();

    public static volatile Container container = null;
}
```
继承该抽象类的 HttpContainer 采用 Jetty 容器
```
public class HttpContainer  extends Container {

    private static final Logger logger = LoggerFactory.getLogger(HttpContainer.class);

    private AbstractHandler httpHandler;
    private ProviderConfig providerConfig;

    public HttpContainer(AbstractHandler httpHandler) {

        this(httpHandler, new ProviderConfig("/invoke",8080));
    }

    public HttpContainer(AbstractHandler httpHandler,ProviderConfig providerConfig) {

        this.httpHandler = httpHandler;
        this.providerConfig = providerConfig;
        Container.container = this;
    }

    public void start() {

        Server server = new Server();
        try {

            SelectChannelConnector connector = new SelectChannelConnector();
            connector.setPort(providerConfig.getPort());
            server.setConnectors(new Connector[]{connector});
            server.setHandler(httpHandler);
            server.start();
        }
        catch (Throwable e) {

            logger.error("容器异常", e);
        }
    }
}
```
用于测试，我们可以定义一些 consumer 和 provider 共有的，包括一个接口
```
public class People {

    private Integer age;

    private Integer sex;

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Integer getSex() {
        return sex;
    }

    public void setSex(Integer sex) {
        this.sex = sex;
    }
}
```
```
public interface SpeakInterface {

    String speak(People people);
}
```
而 provider 则多了 SpeakInterface 接口的实现

这里 consumer 和 provider 都使用 Spring 做容器，`@Component` 是用于 Spring 扫描
```
@Component("speakInterface")
public class SpeakInterfaceImpl implements SpeakInterface {

    public String speak(People people)
    {
        if (people.getAge() > 18)
        {
            if (people.getSex() == 0)
            {
                return "男 "+people.getAge()+"岁";
            }else
            {
                return "女  "+people.getAge()+"岁";
            }
        }
        else
        {
            return "小朋友";
        }
    }
}
```
采用 Spring 容器，对于 provider 的 Spring 配置有
```
<context:component-scan base-package="com.lian.rpc" />
<context:annotation-config />

<bean class="com.lian.rpc.proxy.ProviderProxyFactory">
    <constructor-arg name="providers">
        <map key-type="java.lang.Class" value-type="java.lang.Object">
            <entry key="com.lian.rpc.SpeakInterface" value-ref="speakInterface"/>
        </map>
    </constructor-arg>
    <constructor-arg name="providerConfig">
        <bean id="providerConfig" class="com.lian.rpc.invoke.ProviderConfig">
            <property name="port" value="8888"/>
            <property name="target" value="/invoke"/>
        </bean>
    </constructor-arg>
</bean>
```
Spring 以构造器的方式创建 ProviderProxyFactory，即同时启动 Jetty 容器和注册 SpeakInterface 接口

Main 启动 Spring 容器
```
public class Main {

    public static void main(String[] args) throws Exception {

        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath*:spring-*.xml");
        context.start();
        CountDownLatch countDownLatch = new CountDownLatch(1);
        countDownLatch.await();
    }
}
```
对于 consumer 的 Spring 配置有
```
<context:component-scan base-package="com.lian.rpc" />
<context:annotation-config />

<bean id="consumerConfig" class="com.lian.rpc.invoke.ConsumerConfig">
    <property name="url" value="http://127.0.0.1:8888/invoke" />
</bean>

<bean id="speakInterfaceInvoker" class="com.lian.rpc.proxy.ConsumerProxyFactory">
    <property name="consumerConfig" ref="consumerConfig"/>
    <property name="clazz" value="com.lian.rpc.SpeakInterface"/>
</bean>
<bean id="speakInterface" factory-bean="speakInterfaceInvoker" factory-method="create"/>
```
配置了 provider 的 url，使用实例工厂生成 SpeakInterface 的代理对象
```
@Component("peopleController")
public class PeopleController {

    @Resource
    private SpeakInterface speakInterface;

    public String getSpeak(Integer age,Integer sex)
    {
        People people = new People();
        people.setAge(age);
        people.setSex(sex);
        return speakInterface.speak(people);
    }
}
```
