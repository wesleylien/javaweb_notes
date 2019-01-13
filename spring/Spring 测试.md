## Spring Test
* Spring 通过 Spring TestContext Framework 对集成测试提供顶级支持
* Spring TestContext Framework 不依赖于特定的测试框架（如：JUnit、TestNG……）
* `SpringJUnit4ClassRunner` 类提供了 Spring TestContext Framework 的功能（`SpringRunner` 类也是继承自 `SpringJUnit4ClassRunner`）
* 通过 `@ContextConfiguration` 载入配置类来配置 Spring 容器 Application Context
* 通过 `@ActiveProfiles` 确定 active profile

### Spring Test 简单用例
增加 Spring 测试的依赖包
``` xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-test</artifactId>
    <version>${spring-framework.version}</version>
</dependency>
<dependency>
	<groupId>junit</groupId>
	<artifactId>junit</artifactId>
    <version>4.11</version>
</dependency>
```
配置类
``` java
@Configuration
public class TestConfig {
  @Bean
  @Profile("test")
  public TestBean testBean() {
    return new TestBean("test");
  }

	@Bean
	@Profile("prod")
	public TestBean testBean() {
    return new TestBean("prod");
  }
}
```
Test 类
``` java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes={TestConfig.class})
@ActiveProfiles("test")
public class SpringTestApplicationTests {

  @Autowired
  private TestBean testBean;

	@Test
	public void prodProfilesTest() {
	    Assert.assertEqual("test", testBean.getContent());
	}

}
```

## Spring MVC Test
测试 Web 的一些 Servlet 相关的模拟对象
* `MockMVC`
* `MockHttpServletRequest`
* `MockHttpServletResponse`
* `MockHttpSession`

在 Spring 里，使用 `@WebAppConfiguration` 指定加载 `ApplicationContext` 是一个 `WebApplicationContext`

``` java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes={MyMvcConfig.class})
// 属性指定 Web 资源的位置，默认为 src/main/webapp
@WebAppConfiguration("src/main/resources")
public class SpringTestApplicationTests {

    private MockMvc mockMvc;

    @Autowired
    private DemoService demoService;

    @Autowired
    WebApplicationContext ctx;

    @Autowired
    MockHttpSession session;

    @Autowired
    MockHttpServletRequest request;

    // 在测试开始之前进行的初始化工作
    @Before
    public void setup() {

        // MockMvc 对象的初始化
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.ctx).build();
    }

    @Test
    public void testNormalController throws Exception {
        mockMvc.perform(get("/normal")) // 模拟向 /normal 发送 get 请求
                .andExpect(status().isOk()) // 预期返回状态为200
                .andExpect(view().name("page")) // 预期 View 的名称为 page
                .andExpect(forwardedUrl("/WEB-INF/classes/views/page.jsp")) // 预期页面转向的真正路径为 /WEB-INF/classes/views/page.jsp
                .andExpect(model().attribute("msg", demoService.saySomething()));   // 预期 model 里 msg 的值为 demoService.saySomething() 的返回值
    }

    @Test
    public void testNormalController throws Exception {
        mockMvc.perform(get("/testRest"))
                .andExpect(status().isOk())
                .andExpect(content().contentType("text/plain;charset=UTF-8")) // 预期返回值的媒体类型为 text/plain;charset=UTF-8
                .andExpect(content().string(demoService.saySomething())) // 预期返回值的内容为 demoService.saySomething() 的返回值
    }
}
```

## Spring Boot Test
``` xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```