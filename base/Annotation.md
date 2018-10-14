


## 组合注解与元注解
所谓元注解即可以注解到别的注解上的注解，被注解的注解成为组合注解，*组合注解具备注解其上的元注解的功能*，例如：
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
@ComponentScan
public @interface NewConfiguration {
  String[] value() default {}; // 元注解 @ComponentScan 有一个 value 属性，组合注解 @NewConfiguration 覆盖 value 属性并赋予默认值
}
```
