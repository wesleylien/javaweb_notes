1. Spring 配置文件设置
    ```
    <!-- 使用LocaleResolver接口的实现类实现本地化信息的解析 -->
    <!-- LocaleResolver有三个常用的实现类：
         AcceptHeaderLocaleResolver
         CookieLocaleResolver
         SessionLocaleResolver
    -->
    <bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver">
		<property name="defaultLocale" value="zh"></property>
	</bean>

	<!-- 在拦截器中设置 -->
	<mvc:interceptors>
	    <!-- 使用LocaleChangeInterceptor实现本地化信息的监听 -->
		<bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"></bean>
	</mvc:interceptors>

	<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
		<property name="basename" value="i18n/message" />
		<property name="defaultEncoding" value="UTF-8" />
		<property name="useCodeAsDefaultMessage" value="true" />
	</bean>
    ```
2. 创建国际化资源文件 `message.zh_CN.properties`、`message.en_US.properties`
3. 新建一个 Filter 类，并在 web.xml 中配置，用于获取当前的Locale
    ```
    public class LocaleFilter implements Filter {
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {

        }
        @Override
        public void destroy() {

        }
        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
            String lang = "zh";
            Cookie[] cookies = ((HttpServletRequest)request).getCookies();
            if (cookies != null && cookies.length > 0) {
                for (Cookie cookie : cookies) {
                    if ("statlang".equals(cookie.getName())) {
                        lang = cookie.getValue();
                    }
                }
            }
            ((HttpServletRequest)request).getSession.setAttribute(SessionLocaleResolver.LOCALE_SESSION_ATTRIBUTE_NAME, new Locale(lang));
            chain.doFilter(request, response);
        }
    }
    ```
    ```
    <filter>
        <filter-name>localeFilter</filter-name>
        <filter-class>com.lian.common.filter.LocaleFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>localeFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ```
