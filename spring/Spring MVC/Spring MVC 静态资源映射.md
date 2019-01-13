## xml 配置方式
``` xml
<mvc:annotation-driven />
<!-- 资源文件所在目录为 /src/main/webapp/static/ -->
<mvc:resources mapping="/static/**" location="/static/" />
```

## Java 配置方式
``` java
@EnableWebMvc
@Configuration
@ComponentScan("com.lian")
public class MyMvcConfig extends WebMvcConfigurerAdapter {

  ...

  @Override
  public void addResourceHandlers(ResourceHandlerResistry registry) {
    // addResourceHandler 指对外暴露的访问路径
    // addResourceLocations 指文件的放置目录
    registry.addResourceHandler("/static/**").addResourceLocations("classpath:/static/");
  }
}
```
