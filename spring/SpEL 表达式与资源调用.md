Spring EL-Spring 表达式语言，支持在 xml 和注解中使用表达式，类似于 jsp 的 EL 表达式

Spring 开发中涉及到各种资源的调用（如普通文件、网址、配置文件、系统环境变量等），都可以使用 SpEL 表达式实现资源的注入

SpEL 表达式在 xml 配置文件或者 Java 配置类或者注解配置中使用，需要在表达式外围加上 `#{spEL}`

Spring 主要在注解 @Value 的参数中使用 SpEL 表达式

## 注解 @Value 与 SpEL 的使用

* 注入普通字符
    ```
    @Value("普通字符")
    private String normalStr;
    ```
* 注入操作系统属性
    ```
    @Value("#{systemProperties['os.name']}")
    private String osName;
    ```
* 注入表达式运算结果
    ```
    @Value("#{ T(java.lang.Math).random() * 100.0 }")
    private double randomNumber;
    ```
* 注入其他 Bean 的属性
    ```
    @Value("#{bean1.bean1Str}")
    private String fromBean1;
    ```
* 注入文件内容
    ```
    @Value("classpath:com/lian/test.txt")
    private Resource testFile;
    ```
    Resource 资源获取
    ```
    // 这里 IOUtils 使用到 commons-io jar 包
    String str = IOUtils.toString(testFile.getInputStream());
    ```
* 注入网址内容
    ```
    @Value("http://www.baidu.com")
    private Resource testUrl;
    ```
    Resource 资源获取
    ```
    // 这里 IOUtils 使用到 commons-io jar 包
    String str = IOUtils.toString(testUrl.getInputStream());
    ```
* 注入属性文件
    ```
    // 注入配置文件需使用 @PropertySource 指定文件地址
    @PropertySource("classpath:com/lian/test.properties")
    public class ELConfig {
        @Value("${book.name}")
        private String bookName;

        // 要使用 @Value 注入配置文件，需要配置一个 PropertySourcePlaceholderConfigurer 的 Bean
        @Bean
        public static PropertySourcePlaceholderConfigurer propertyConfigure() {
            return new PropertySourcePlaceholderConfigurer();
        }
    }    
    ```
    获取配置文件资源的另一种方式
    ```
    // 注入配置文件需使用 @PropertySource 指定文件地址
    @PropertySource("classpath:com/lian/test.properties")
    public class ELConfig {
        @Autowired
        private Environment environment;
    }
    ```
    ```
    String str = environment.getProperty("book.name");
    ```

在XML或annotation中使用，需要在表达式外围加上#{}
ExpressionParser	EvaluationContext
1. 直接量表达式
2. 在表达式中创建数组
3. 在表达式中创建list
4. 在表达式中访问list map
5. 调用方法
6. 算术、比较、逻辑、赋值、三目等运算符
7. 类型运算符 T()
8. 调用构造器
...
