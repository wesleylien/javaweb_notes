在 Spring Boot 中，通过简单的 `@Enable*` 即可开启一项功能的支持，从而避免配置大量的代码

所有的 `@Enable*` 注解都有一个 `@Import` 注解。 `@Import` 注解是用来导入配置类的，这意味着这些自动开启的实现实际上是导入了一些自动配置 Bean

`@Enable*` 系列导入的配置方式分为三种

## 第一类：直接导入配置类
如 `@EnableScheduling` 直接导入配置类 `SchedulingConfiguration`。这个配置类注册了一个 `scheduledAnnotationProcessor` 的 Bean

``` java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(SchedulingConfiguration.class)
@Documented
public @interface EnableScheduling {

}
```

## 第二类：依据条件选择配置类
如 `@EnableAsync` 导入类 `AsyncConfigurationSelector`。`AsyncConfigurationSelector` 通过条件来选择需要导入的配置类

``` java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AsyncConfigurationSelector.class)
public @interface EnableAsync {
  ...
}
```

`AsyncConfigurationSelector` 继承于 `AdviceModelImportSelector`，它的根接口为 `ImportSelector`，这个接口需重写 `selectImport` 方法，在此方法中进行事先条件判断

``` java
public class AsyncConfigurationSelector extends AdviceModeImportSelector<EnableAsync> {

	private static final String ASYNC_EXECUTION_ASPECT_CONFIGURATION_CLASS_NAME =
			"org.springframework.scheduling.aspectj.AspectJAsyncConfiguration";

	/**
	 * {@inheritDoc}
	 * @return {@link ProxyAsyncConfiguration} or {@code AspectJAsyncConfiguration} for
	 * {@code PROXY} and {@code ASPECTJ} values of {@link EnableAsync#mode()}, respectively
	 */
	@Override
	public String[] selectImports(AdviceMode adviceMode) {
		switch (adviceMode) {
			case PROXY:
				return new String[] { ProxyAsyncConfiguration.class.getName() };
			case ASPECTJ:
				return new String[] { ASYNC_EXECUTION_ASPECT_CONFIGURATION_CLASS_NAME };
			default:
				return null;
		}
	}

}
```

## 第三类：动态注册 Bean
如 `@EnableAspectJAutoProxy` 导入类 `AspectJAutoProxyRegister`。`AspectJAutoProxyRegister` 实现了 `ImportBeanDefinitionRegister` 接口。`ImportBeanDefinitionRegister` 的作用是在运行时自动添加 Bean 到加上注解的配置类，通过重写方法 `registerBeanDefinitions`

``` java
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {

	/**
	 * Register, escalate, and configure the AspectJ auto proxy creator based on the value
	 * of the @{@link EnableAspectJAutoProxy#proxyTargetClass()} attribute on the importing
	 * {@code @Configuration} class.
	 */
	@Override
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

		AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);

		AnnotationAttributes enableAspectJAutoProxy =
				AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
		if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
			AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
		}
		if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
			AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
		}
	}

}
```

其中， 参数 `AnnotationMetadata` 用来获得当前配置类上的注解，`BeanDefinitionRegistry` 用来注册 Bean
