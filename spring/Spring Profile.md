## Profile
Spring 通过 Profile 为在不同环境下使用不同配置提供了支持

### 设置 profile 的方式
有三种设置 Profile 的方式：
1. 通过设定 Environment 的 ActiveProfiles 来设定当前 context 需要使用的配置环境
  ``` java
  AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();

  // 先设置 profile 再加载配置类，配置类才能根据 profile 注册 Bean
  context.getEnvironment(),setActiveProfiles("prod");
  context.register(ProfileConfig.class);
  // 刷新容器
  context.refresh();
  ```
2. 通过设定 jvm 的 spring.profiles.active 参数来设置配置环境 `-Dspring.profiles.active=prod`
  如用 Tomcat 启动 Spring MVC 则可设置 Tomcat 配置文件 `catalina.sh` 中的 JAVA_OPTS 项来设置 jvm 启动参数
  ```
  JAVA_OPTS=" -Xms1024m -Xmx1024m  -XX:PermSize=512m -XX:MaxPermSize=512m -Dspring.profiles.active=prod"
  ```
3. Web 项目在 Servlet 的 context paramter 中    
  在 Servlet 2.5 及以下( web.xml )：   
  ```
  <servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>spring.profiles.active</param-name>
      <param-value>production</param-value>
    </init-param>
  </servlet>
  ```
  Servlet 3.0 及以上( WebApplicationInitializer 实现类 )：
  ```
  public class WebInit implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext container) throws ServletException {
      container.setInitParameter("spring.profiles.active", "production");
    }
  }
  ```
此外，**在 Spring Test 测试类中**，通过 `@ActiveProfiles` 确定活动的 profile
```
@ActiveProfiles("prod")
```

此外，**在 Spring Boot 中**，假设有两个配置文件 `application-dev.properties` 和 `application-prod.properties` ，则在 `application.properyies` 可指定配置文件
```
spring.profiles.active=prod
```
因为 Spring Boot 打包成 jar 包后可以用命令行运行，因而下面是等价的
```
java -jar xxx.jar --spring.profiles.active=prod
```

## Java 配置类使用 profile 确定是否加载该配置类
```
@Configuration
@Profile("dev")
public class ProfileConfig {
    ...
}
```

## Java 配置类使用 profile 注册不同的 Bean
```
@Configuration
public class ProfileConfig {
    @Bean
    @Profile("dev")
    public Bean1 devBean1() {
        return new Bean1("1");
    }

    @Bean
    @Profile("prod")
    public Bean1 prodBean1() {
        return new Bean1("2");
    }
}
```

## xml 配置使用 profile 确定是否加载
  datasource-dev.xml
  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:jdbc="http://www.springframework.org/schema/jdbc"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans.xsd
              http://www.springframework.org/schema/jdbc
              http://www.springframework.org/schema/jdbc/spring-jdbc.xsd"
              profile="dev">


      <jdbc:embedded-database id="dataSource">
          <jdbc:script location="classpath:schema.sql"/>
          <jdbc:script location="classpath:test-data.sql"/>
      </jdbc:embedded-database>

  </beans>
  ```
  datasource-prod.xml
  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:jdbc="http://www.springframework.org/schema/jdbc"
         xmlns:jee="http://www.springframework.org/schema/jee"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans.xsd
              http://www.springframework.org/schema/jdbc
              http://www.springframework.org/schema/jdbc/spring-jdbc.xsd"
         profile="prod">


      <jee:jndi-lookup jndi-name="jdbc/myDatabase"
                       id="dataSource"
                       resource-ref="true"
                       proxy-interface="javax.sql.DataSource"
          />

  </beans>
  ```

## xml 配置使用 profile 注册不同的 Bean
  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
      xmlns:util="http://www.springframework.org/schema/util"
      xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
          http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
          http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.2.xsd">

      <!-- 定义开发的profile -->
      <beans profile="development">
          <!-- 只扫描开发环境下使用的类 -->
          <context:component-scan base-package="com.panlingxiao.spring.profile.service.dev" />
          <!-- 加载开发使用的配置文件 -->
          <util:properties id="config" location="classpath:dev/config.properties"/>
      </beans>

      <!-- 定义生产使用的profile -->
      <beans profile="produce">
          <!-- 只扫描生产环境下使用的类 -->
          <context:component-scan
              base-package="com.panlingxiao.spring.profile.service.produce" />
          <!-- 加载生产使用的配置文件 -->    
          <util:properties id="config" location="classpath:produce/config.properties"/>
      </beans>
  </beans>
  ```
