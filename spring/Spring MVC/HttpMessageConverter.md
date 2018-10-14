HttpMessageConverter 是用来处理 request 和 response 里的数据的

Spring 内置了大量的 HttpMessageConverter，如 MappingJackson2HttpMessageConverter、StringHttpMessageConverter……

## 自定义 HttpMessageConverter
```
public class MyHttpMessageConverter extends AbstractHttpMessageConverter<DemoObject> {

    public MyHttpMessageConverter() {

        // 该自定义 HttpMessageConverter 用于处理自定义的媒体类型 application/x-lianwx
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
        // 这里我们处理由 - 隔开的 String，并转成 DemoObject 对象
        String temp = StreamUtils.copyToString(inputMeaasge.getBody(), Charset.forName("UTF-8"));
        String[] tempArr = temp.split("-");
        return new DemoObject(new Long(tempArr[0]), tempArr[1]);
    }

    // 重写 writeInternal 方法处理 response 数据
    @Override
    protected void writeInternal(DemoObject obj, HttpOutputMessage outputMeaasge) throws IOException, HttpMessageNotReadableException {
        // 这里将 DemoObject 实例转成 String
        String out = obj.getId() + "-" + obj.getName();
        outputMeaasge.getBody().write(out.getBytes());
    }
}
```

## 在 Spring 中配置自定义 HttpMessageConverter
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
