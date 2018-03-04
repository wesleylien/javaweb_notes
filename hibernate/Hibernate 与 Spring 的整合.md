## 配置 applicationContext.xml 文件
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd>

	<description>Spring公共配置</description>

	<!-- 自动扫描与装配bean -->
	<context:component-scan base-package="com.lian" />

	<!-- 导入外部的 properties 文件 -->
	<context:property-placeholder location="classpath:jdbc.properties"/>

	<!-- 配置 dataSource 和 sessionFactory -->

	<!-- 配置事务 -->

</beans>
```
### 配置 dataSource 和 sessionFactory
* 直接在 spring 的配置文件 applicationContext.xml 中配置
    ```
    <!-- 配置c3p0数据库连接池 -->
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<!-- 数据连接信息 -->
		<property name="driverClass" value="${jdbc.driverClass}"></property>
		<property name="jdbcUrl" value="${jdbc.url}"></property>
		<property name="user" value="${jdbc.user}"></property>
		<property name="password" value="${jdbc.password}"></property>
		<!-- 其他配置 -->
		<!--初始化时获取三个连接，取值应在minPoolSize与maxPoolSize之间。Default: 3 -->
        <property name="initialPoolSize" value="3"></property>
        <!--连接池中保留的最小连接数。Default: 3 -->
        <property name="minPoolSize" value="3"></property>
        <!--连接池中保留的最大连接数。Default: 15 -->
        <property name="maxPoolSize" value="5"></property>
        <!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。Default: 3 -->
        <property name="acquireIncrement" value="3"></property>
        <!-- 控制数据源内加载的PreparedStatements数量。如果maxStatements与maxStatementsPerConnection均为0，则缓存被关闭。Default: 0 -->
        <property name="maxStatements" value="8"></property>
        <!--maxStatementsPerConnection定义了连接池内单个连接所拥有的最大缓存statements数。Default: 0 -->
        <property name="maxStatementsPerConnection" value="5"></property>
        <!--最大空闲时间,1800秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 -->
        <property name="maxIdleTime" value="1800"></property>
	</bean>

	<!-- 配置 sessionFactory -->
	<bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
		<!-- 设置 dataSource 属性 -->
		<property name="dataSource" ref="dataSource"></property>
		<!-- 配置 Hibernate Properties -->
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
				<prop key="hibernate.show_sql">true</prop>
				<prop key="hibernate.format_sql">true</prop>
				<prop key="hbm2ddl.auto">update</prop>
			</props>
		</property>
		<!-- 配置 Mapping Resources -->
 		<property name="mappingLocations" value="classpath:com/lian/entity/*.hbm.xml"></property>
	</bean>
    ```
* 引用 Hibernate 的配置文件 hibernate.cfg.xml
    ```
    <!-- 配置 sessionFactory -->
	<bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
	    <!-- 引用 Hibernate 的配置文件来配置 dataSource 和 Hibernate Properties -->
		<property name="configLocation" value="classpath:hibernate.cfg.xml" />
		<!-- 配置 Mapping Resources -->
 		<property name="mappingLocations" value="classpath:com/lian/entity/*.hbm.xml"></property>
	</bean>
    ```

当相同的配置项都存在时，applicationContext.xml 中的优先级高于 hibernate.cfg.xml

### 配置事务
* XML 配置   
    ```
    <!-- 配置 局部事务管理器 -->
	<bean id="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>

	<!-- 配置 增强事务 Bean -->
	<tx:advice id="txAdvice" transaction-manager="transactionManager">
	    <!-- 配置 详细的事务语义 -->
	    <tx:attributes>
	        <!-- 为不同的方法指定相应的事务语义 -->
	        <!-- 所有以 get 开头的方法都是 read-only 的 -->
	        <tx:method name="get*" read-only="true"/>
	        <!-- 其他方法使用默认的事务设置 -->
	        <tx:method name="*">
	    </tx:attributes>
	</tx:advice>

	<aop:config>
	    <!-- 配置一个切入点 -->
	    <aop:pointcut id="onePointcut" expression="bean(personService)"/>
	    <!-- 指定在 onePointcut 切入点应用 txAdvice 事务增强处理 -->
	    <aop:advisor advice-ref="txAdvice" pointcut-ref="onePointcut"/>
	</aop:config>
    ```
* Annotation   

    ```
    <!-- 配置 局部事务管理器 -->
	<bean id="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>

	<!-- 根据 Annotation 生成事务代理 -->
	<tx:annotation-driven transaction-manager="transactionManager"/>
    ```

    * @Transactional(...)

        属性 | 说明
        ---|---
        name | 当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器
        propagation | 事务的传播行为，默认值为 REQUIRED
        isolation | 事务的隔离度，默认值采用 DEFAULT
        timeout | 事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务
        read-only | 指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true
        rollback-for | 用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔
        no-rollback- for | 抛出 no-rollback-for 指定的异常类型，不回滚事务


[透彻的掌握 Spring 中@transactional 的使用](https://www.ibm.com/developerworks/cn/java/j-master-spring-transactional-use/index.html)


[Spring @Transactional工作原理](http://www.importnew.com/12300.html)


[Spring @Transactional原理及使用](http://tech.lede.com/2017/02/06/rd/server/SpringTransactional/)

## Spring 的 DAO 支持
Spring 提供许多工具类用于对 DAO 实现简化开发步骤

Spring 提供了一致的异常抽象，将原有的 Checked 异常转换包装为 Runtime 异常，因而，编码时无序捕获各种技术中特定的异常

* HibernateTemplate
    ```
    public class PersonDaoImpl implements PersonDao {

        private HibernateTemplate ht = null;

        @Resource(name="sessionFactory")
        protected SessionFactory sessionFactory;

        public SessionFactory getSessionFactory() {
        	return sessionFactory;
        }

        public void setSessionFactory(SessionFactory sessionFactory) {
        	this.sessionFactory = sessionFactory;
        }

        private HibernateTemplate getHibernateTemplate() {
            if (ht == null) {
                ht = new HibernateTemplate(sessionFactory);
            }
            return ht;
        }

        // HibernateTemplate 的使用类似于 Session + Query
        public Person get(Integer id) {
            return getHibernateTemplate().get(Person.class, id);
        }
    }
    ```

    缺点：灵活性不足，不能用 Hibernate API 进行持久化访问

* HibernateCallback 接口
    * HibernateTemplate 的 `.execute(HibernateCallback action)` 和 `.executeFind(HibernateCallback action)` 方法接收一个 HibernateCallback 接口的实现
    * HibernateCallback 接口包含 `doInHibernate(Session session)` 方法
    * 两者结合可解决灵活应用 Hibernate API 的问题
    ```
    public class PersonDaoImpl implements PersonDao {

        private HibernateTemplate ht = null;

        @Resource(name="sessionFactory")
        protected SessionFactory sessionFactory;

        public SessionFactory getSessionFactory() {
        	return sessionFactory;
        }

        public void setSessionFactory(SessionFactory sessionFactory) {
        	this.sessionFactory = sessionFactory;
        }

        private HibernateTemplate getHibernateTemplate() {
            if (ht == null) {
                ht = new HibernateTemplate(sessionFactory);
            }
            return ht;
        }

        public Person get(final Integer id) {
            return getHibernateTemplate().execute(new HibernateCallback() {
                public Object doInHibernate(Session session) throws HibernateException, SQLException {
                    return session.get(Person.class, id);
                }
            });
        }
    }
    ```
* HibernateDaoSupport
    * HibernateDaoSupport 的 `getHibernateTemplate()` 方法返回 HibernateTemplate 对象
    * HibernateDaoSupport 的 `setSessionFactory(SessionFactory sessionFactory)` 可用于接收 Spring 的依赖注入，从而允许使用 sessionFactory 对象
    * HibernateDaoSupport 的 DAO 实现由 HibernateTemplate 对象完成
    * DAO 类的实现继承 HibernateDaoSupport

## Spring 使用声明式事务
* Spring 使用声明式事务来进行统一的事务管理
* 只需要在配置文件中增加事务控制片段，业务逻辑组件的方法将会有事务性
* Spring 的声明式事务支持在不同事务策略之间自由切换

## Hibernate 在 Spring 框架下的 DAO 工程实现
BaseDao.java
```
/**
 * 通用泛型DAO
 */
public interface BaseDao<T> {

	/**
	 * 新增一个实例
	 * @param entity 要新增的实例
	 */
	public void save(T entity);

	/**
	 * 根据主键删除一个实例
	 * @param id 主键
	 */
	public void delete(int id);

	/**
	 * 编辑指定实例的详细信息
	 * @param entity 实例
	 */
	public void edit(T entity);

	/**
	 * 根据主键获取对应的实例
	 * @param id 主键值
	 * @return 如果查询成功，返回符合条件的实例;如果查询失败，返回null
	 */
	public T get(Integer id);

	/**
	 * 根据主键获取对应的实例
	 * @param id 主键值
	 * @return 如果查询成功，返回符合条件的实例;如果查询失败，抛出空指针异常
	 */
	public T load(Integer id);

	/**
	 * 获取所有实体实例列表
	 * @return 符合条件的实例列表
	 */
	public List<T> findAll();

	/**
	 * 统计总实体实例的数量
	 * @return 总数量
	 */
	public int totalCount();

	/**
	 * 获取分页列表
	 * @param pageNo 当前页号
	 * @param pageSize 每页要显示的记录数
	 * @return 符合分页条件的分页模型实例
	 */
	public PageModel<T> findByPager(int pageNo, int pageSize);

	/**
	 * 根据指定的SQL语句和参数值执行更新数据的操作
	 * @param sql SQL语句
	 * @param paramValues 参数值数组
	 */
	public void update(String sql);

	/**
	 * 根据指定的SQL语句和参数值执行单个对象的查询操作
	 * @param sql SQL语句
	 * @param paramValues 参数值
	 * @return 符合条件的实体对象
	 */
	public T findUnique(String sql);
}
```
BaseDaoImpl.java
```
/**
 * 通用DAO接口的实现类
 */
@SuppressWarnings("unchecked")
public class BaseDaoImpl<T> implements BaseDao<T> {

	/**
	 * 对应的持久化类
	 */
	private Class<T> clazz;

	@Resource(name="sessionFactory")
	protected SessionFactory sessionFactory;

	public SessionFactory getSessionFactory() {
		return sessionFactory;
	}

	public void setSessionFactory(SessionFactory sessionFactory) {
		this.sessionFactory = sessionFactory;
	}

	public BaseDaoImpl(){
		//通过反射机制获取子类传递过来的实体类的类型信息
		ParameterizedType type=(ParameterizedType)this.getClass().getGenericSuperclass();
		this.clazz=(Class<T>)type.getActualTypeArguments()[0];
	}

	@Override
	public void save(T entity) {
		Session session = sessionFactory.getCurrentSession();
		session.save(entity);
	}

	@Override
	public void delete(int id) {
		Session session = sessionFactory.getCurrentSession();
		session.delete(get(id));
	}

	@Override
	public void edit(T entity) {
		Session session = sessionFactory.getCurrentSession();
		session.merge(entity);
	}

	@Override
	public T get(Integer id) {
		Session session = sessionFactory.getCurrentSession();
		return (T) session.get(clazz, id);
	}

	@Override
	public T load(Integer id) {
		Session session = sessionFactory.getCurrentSession();
		return (T)session.load(clazz, id);
	}

	@Override
	public List<T> findAll() {
		Session session = sessionFactory.getCurrentSession();
		String hql = "select t from "+clazz.getSimpleName()+" t";
		return (List<T>)session.createQuery(hql).list();
	}

	@Override
	public int totalCount() {
		Session session = sessionFactory.getCurrentSession();
		int count = 0;
		String hql = "select count(t) from "+clazz.getSimpleName()+" t";
		Long temp = (Long)session.createQuery(hql).uniqueResult();
		if(temp != null){
			count = temp.intValue();
		}
		return count;
	}

	@Override
	public PageModel<T> findByPager(int pageNo, int pageSize) {
		Session session = sessionFactory.getCurrentSession();
		PageModel<T> pm = new PageModel<T>(pageNo, pageSize);
		String hql = "select t from "+clazz.getSimpleName()+" t";
		int startRow = (pageNo - 1) * pageSize;
		pm.setDatas(session.createQuery(hql).setFirstResult(startRow).setMaxResults(pageSize).list());
		pm.setRecordCount(totalCount());
		return pm;
	}

	@Override
	public void update(String hql) {
		Session session = sessionFactory.getCurrentSession();
		session.createQuery(hql);
	}

	@Override
	public T findUnique(String hql) {
		Session session = sessionFactory.getCurrentSession();
		return (T)session.createQuery(hql).uniqueResult();
	}
}
```
PageModel.java
```
/**
 * 分页模型类
 */
public class PageModel<T> {

	//当前页号
	private int pageNo=1;
	//每页显示的记录数
	private int pageSize=10;
	//总记录数
	private int recordCount;
	//总页数
	private int pageCount;
	//存放分页数据的集合
	private List<T> datas;

	public PageModel(){

	}

	public PageModel(int pageNo,int pageSize){
		this.pageNo=pageNo;
		this.pageSize=pageSize;
	}

	public int getPageNo() {
		return pageNo;
	}

	public void setPageNo(int pageNo) {
		this.pageNo = pageNo;
	}

	public int getPageSize() {
		return pageSize;
	}

	public void setPageSize(int pageSize) {
		this.pageSize = pageSize;
	}

	public int getRecordCount() {
		return recordCount;
	}

	public void setRecordCount(int recordCount) {
		this.recordCount = recordCount;
	}

	public int getPageCount() {
		if(this.getRecordCount()<=0){
			return 0;
		}else{
			pageCount=(recordCount+pageSize-1)/pageSize;
		}
		return pageCount;
	}

	public void setPageCount(int pageCount) {
		this.pageCount = pageCount;
	}

	public List<T> getDatas() {
		return datas;
	}

	public void setDatas(List<T> datas) {
		this.datas = datas;
	}
}
```
