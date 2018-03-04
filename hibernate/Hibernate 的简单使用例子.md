## 导入相关包
包名 | 解释
---|---
hibernate-core | Hibernate 核心包
antlr | 用于实现从 HQL 到 SQL 的转换
dom4j | 解析 XML 配置文件和 XML 映射文件
hibernate-commons-annotations | Hibernate 注解包
hibernate-jpa-2.1-api | JPA2.1 接口库
jandex | 用来索引 Annotation
javassist | Hibernate 用来实现 PO 字节码的动态生成
jboss-logging | 日志服务通用库
jboss-logging-annotations | 实现带注释的接口的具体类
jboss-transaction-api_1.2_spec | JTA 规范包

其它还包括：
* 数据库的 dirver
* 日志

## 创建 Hibernate XML 配置文件
创建 Hibernate XML 配置文件：
``` xml
<!-- hibernate.cfg.xml -->

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
	"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<!-- 根元素 -->
<hibernate-configuration>

	<session-factory>
	  <!-- property 元素用于配置 Hibernate 连接数据的必要信息 -->
		<!-- 方言 -->
		<property name="hibernate.dialect">org.hibernate.dialect.MySQL5InnoDBDialect</property>
		<!-- 驱动 -->
		<property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
		<!-- url -->
		<property name="hibernate.connection.url">jdbc:mysql://localhost:3306/tang_poetry?characterEncoding=utf-8</property>
		<!-- 用户名 -->
		<property name="hibernate.connection.username">root</property>
		<!-- 密码 -->
		<property name="hibernate.connection.password">123456</property>
		<!-- Hibernate 不推荐采用 DriverManager 来连接数据库，而是推荐数据源( DataSource )来管理数据库连接。如 c3p0 -->
		<!-- 配置C3P0连接池 -->
		<!-- 使用 C3P0 需导入相关 jar 包 -->
		<property name="hibernate.connection.provider_class">org.hibernate.connection.C3P0ConnectionProvider</property>
		<property name="hibernate.c3p0.max_size">10</property>
		<property name="hibernate.c3p0.min_size">5</property>
		<property name="hibernate.c3p0.timeout">5000</property>
		<!-- 缓存 statement 的数量 -->
		<property name="hibernate.c3p0.max_statements">10</property>



		<!-- 控制台是否输出 sql 语句 -->
		<property name="hibernate.show_sql">true</property>
		<!-- 是否将 sql 转化成格式良好的 sql -->
		<property name="hibernate.format_sql">true</property>

		<!-- 罗列映射文件 -->
		<mapping resource="com/lian/bean/Poet.hbm.xml"/>
		<mapping resource="com/lian/bean/Poetry.hbm.xml"/>
	</session-factory>
</hibernate-configuration>
```

## 创建持久化类和对应的 XML 映射文件
Hibernate 持久化类的要求：
* 提供无参数构造器
* 提供一个标识属性——映射数据库表的主键字段
* 每个属性提供 getter setter 方法
* 使用非 final 类
* 重写 equals() 和 hashCode() 方法——对应需要把持久化类放在 Set 中的情况

``` xml
<!-- xxx.hbm.xml -->

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">

<!-- hibernate-mapping 是映射文件的根元素 -->
<!-- schema 指定数据库的 Schema 名，如果指定则表名自动添加该 Schema 前缀 -->
<!-- catelog 指定数据库的 Category 名，如果指定则表名自动添加该 Category 前缀 -->
<!-- default-cascade 设置默认的级联风格，默认为 none -->
<!-- default-access 设置默认的属性访问策略，默认为 property，即 getter 和 setter。field 则通过反射 -->
<!-- default-lazy 设置默认的延迟加载策略，默认为 true -->
<!-- auto-import 设置是否允许在查询语言中使用非全限定类名，默认为 true。当同一映射文件有两个持久化类要映射，且类名相同（不同包中），应设置为 false -->
<!-- package 指定包前缀，如果没有指定全限定类名，则默认使用该 package 前缀 -->
<hibernate-mapping>
    <!-- 可以有多个 class -->
    <!-- name 指定要映射的持久化类名，如果没有指定 package，则应为全限定类名 -->
    <!-- schema -->
    <!-- catelog -->
    <!-- lazy -->
    <!-- table 指定该持久化类映射的数据库的表名，默认以持久化类的类名为表名 -->
    <!-- discriminator-value 指定区分不同子类的值，当使用 <subclass.../> 元素定义持久化类的继承关系映射是需要使用该属性 -->
    <!-- mutable 指定持久化类是可变对象还是不可变对象，默认为 true -->
    <!-- proxy 指定一个接口，在延迟加载时作为代理使用，也可以指定该类自己的名字 -->
    <!-- dynamic-update 指定 update 语句是否在运行时动态生成，并且只更新改变过的字段，默认 false，开启会导致 Hibernate 需要更多时间来生成 sql 语句 -->
    <!-- 打开 dynamic-update 可指定几种乐观锁定策略：version / all / dirty / none。version 使用 version/timestamp 是唯一能够处理 Session 外脱管操作的策略 -->
    <!-- dynamic-insert 指定 insert 语句是否在运行时动态生成，并且只插入非空字段，默认 false -->
    <!-- select-before-update 默认为 false -->
    <!-- polymorphism 当采用 <union-subclass.../> 元素来配置继承映射时，该元素指定是否需要采用隐式多态查询 -->
    <!-- where 指定一个附加的 sql 语句过滤条件 -->
    <!-- persister 指定一个定制的 ClassPersister -->
    <!-- batch-size 指定根据标识符来抓取实例时每批抓取的实例数，默认为 1 -->
    <!-- optimistic-lock 指定乐观锁定策略，默认为 version -->
    <!-- check 指定一个 sql 表达式，用于为持久化类所对应的表指定一个多行的 Check 约束 -->
    <!-- subselect 该属性用于映射不可变、只读实体 -->
	<class name="com.lian.bean.Poet" table="poets">
		<!-- 生成对象唯一的OID标示符 -->
		<!-- name 对应持久化类的属性名 -->
		<!-- type 指定标识属性的数据类型，可以是 Hibernate 内建类型、Java 类型（全限定类名） -->
		<!-- Hibernate 内建类型：integer / string / character / date / timestamp / float / binary / serializable / object / blob -->
		<!-- column 设置所映射的数据库表的字段名 -->
		<!-- unsaved-value -->
		<!-- access -->
		<id name="id" column="id" type="integer">
		    <!-- 主键的生成策略 -->
		    <-- class 生成策略，有：increment / identity / sequence / hilo / seqhilo / uuid / guid / native / assigned / select / foreign -->
			<generator class="native"/>
		</id>
		<!-- 普通属性 -->
		<!-- name -->
		<!-- type 指定标识属性的数据类型，可以是 Hibernate 内建类型、Java 类型（全限定类名）、可序列化的 Java 类类名、自定义类的类名 -->
		<!-- update 设置 Hibernate 生成的 update 语句是否需要包含该字段 -->
		<!-- insert -->
		<!-- formula 指定一个 sql 表达式，表示值由表达式计算而来 -->
		<!-- lazy -->
		<!-- access -->
		<!-- unique 是否唯一 -->
		<!-- not-null 是否为空 -->
		<!-- optimistic-lock 是否需要使用乐观锁 -->
		<!-- generated 字段的值是否由数据库生成，可为：never / insert / always -->
		<!-- index 指定一个字符串的索引名称 -->
		<!-- unique_key 指定唯一键的名称 -->
		<!-- length -->
		<!-- precision 有效数字位的位数 -->
		<!-- scale 小数位数 -->
		<property name="name" type="string"/>
		<property name="created_at" type="timestamp"/>
		<property name="updated_at" type="timestamp"/>


		<!-- 集合类型属性，有：list / set / map / array / primitive-array / bag / idbag。bag 用于映射无序数组，idbag 用于映射无序数组，但为集合增加逻辑次序 -->
		<!-- name -->
		<!-- table 指定保存集合属性的表名，默认与集合属性同名 -->
		<!-- schema -->
		<!-- lazy -->
		<!-- inverse 指定该集合关联的实体在双向关联关系中不控制关联关系 -->
		<!-- cascade 指定对持久化对象的持久化操作(save/update/delete)是否会级联到它所关联的子实体 -->
		<!-- order-by 设置数据库对集合元素排序，值为指定表的指定字段加上 asc 或 desc -->
		<!-- sort 指定集合的排序顺序 -->
		<!-- where -->
		<!-- access -->
		<!-- mutable 指定集合中的元素是否可变 -->
		<list name="" table="">
		    <!-- 映射集合属性数据表的外键列 -->
		    <!-- column -->
		    <!-- on-delete -->
		    <!-- property-ref -->
		    <!-- not-null -->
		    <!-- update -->
		    <!-- unique -->
		    <key column="" not-null=""/>
		    <!-- 映射集合属性数据表的集合索引列 -->
		    <list-index column="list_order"/>
		    <!-- 映射保存集合元素的数据列 -->
		    <element type="" column=""/>
		    <composite-element.../>
			<one-to-many.../>
			<manyto-many.../>
		</list>
		<!-- 数组与 list 的区别是 list 长度可变，数组不可变 -->
		<array name="" table="">
		    <key column="" not-null=""/>
		    <list-index column=""/>
		    <element type="" column=""/>
		    <composite-element.../>
			<one-to-many.../>
			<manyto-many.../>
		</array>
		<!-- set 是无序的，因而无需映射集合元素的索引列 -->
		<!-- sort 属性：当使用 SortedSet 有序集合时设置，有：unsorted 映射有序集合时不排序；natural 映射有序集合时使用自然排序，java.util.Comparator 实现类类名 -->
		<set name="poetries" cascade="all" inverse="true">
			<key column="poet_id"/>
			<map-key>
			<element.../>
			<composite-element.../>
			<one-to-many.../>
			<manyto-many.../>
		</set>
		<!-- bag 既可以映射 list，又可以映射 set，甚至 Collection 集合属性 -->
		<bag name="" table="">
		    <key column="" not-null=""/>
		    <element type="" column="" not-null=""/>
		</bag>
		<map name="" table="">
		    <key column="" not-null=""/>
		    <map-key column="" type="string">
		    <map-key-many-to-many>
			<composite-map-key>
		    <element.../>
			<composite-element.../>
			<one-to-many...>
			<manyto-many...>
		</map>
	</class>
	<!-- 命名查询 -->
	<query name="accountHql">
		<![CDATA[from Account]]>
	</query>
</hibernate-mapping>
```

持久化对象属性的类型与 Hibernate 映射类型与 SQL 类型对应表

![1](/images/hibernate_1.png)

![2](/images/hibernate_2.png)

![3](/images/hibernate_3.png)

## 创建 log4j 日志配置文件 log4j.properties
```
### direct log messages to stdout ###
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n

### direct messages to file hibernate.log ###
#log4j.appender.file=org.apache.log4j.FileAppender
#log4j.appender.file.File=hibernate.log
#log4j.appender.file.layout=org.apache.log4j.PatternLayout
#log4j.appender.file.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n

### set log levels - for more verbose logging change 'info' to 'debug' ###

log4j.rootLogger=warn, stdout

#log4j.logger.org.hibernate=info
log4j.logger.org.hibernate=debug

### log HQL query parser activity
#log4j.logger.org.hibernate.hql.ast.AST=debug

### log just the SQL
#log4j.logger.org.hibernate.SQL=debug

### log JDBC bind parameters ###
log4j.logger.org.hibernate.type=info
#log4j.logger.org.hibernate.type=debug

### log schema export/update ###
log4j.logger.org.hibernate.tool.hbm2ddl=debug

### log HQL parse trees
#log4j.logger.org.hibernate.hql=debug

### log cache activity ###
#log4j.logger.org.hibernate.cache=debug

### log transaction activity
#log4j.logger.org.hibernate.transaction=debug

### log JDBC resource acquisition
#log4j.logger.org.hibernate.jdbc=debug

### enable the following line if you want to track down connection ###
### leakages when using DriverManagerConnectionProvider ###
#log4j.logger.org.hibernate.connection.DriverManagerConnectionProvider=trace
```

## 创建表
在 Hibernate 的配置文件中，在 `<session-factory>` 的子元素下添加

``` xml
<!-- 指定是否需要 Hibernate 根据映射文件自动创建数据表 -->
<!-- update 表示需要 -->
<!-- create-drop 显示关闭 SessionFactory 时，将 Drop 刚建的数据表 -->
<property name="hbm2ddl.auto">update</property>
```

创建表代码

``` java
Configuration cfg = new Configuration().configure();
SchemaExport se = new SchemaExport(cfg);
se.create(true, true);
```

## 从 XML 配置文件中构建 SessionFactory
``` java
// 默认读取 classpath 目录下的 hibernate.cfg.xml 文件

Configuration config = new Configuration().configure();
ServiceRegistry service = new StandardServiceRegistryBuilder().applySettings(config.getProperties()).build();
SessionFactory factory = config.buildSessionFactory(service);
```

## 从 SessionFactory 中获取 Session
``` java
Session session = null;
Transaction tx = null;
try {
	session = factory.openSession();
  // 开启事务
	tx = session.beginTransaction();

	...

	tx.commit();
} catch(Exception e) {
	tx.rollback();
} finally {
	session.close();
}
```

## Session 的基本使用
```
// 增

Student stu=new Student();
stu.setName("zhangsan");
stu.setAge(20);

session.save(stu);
```

```
// 改

Student stu=(Student)session.get(Student.class, 1);

stu.setName("zhaoliu");
stu.setAge(50);

session.update(stu);
```

```
// 删

Student stu=(Student)session.get(Student.class, 2);

session.delete(stu);
```

```
// 查

Student stu=(Student)session.get(Student.class, 1);
```
