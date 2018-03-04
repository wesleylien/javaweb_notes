## 映射文件一览
```
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
    <!-- dynamic-update 指定 update 语句是否在运行时动态生成，并且只更新改变过的字段，默认 false，开启会导致 Hibernate 需要更多时间来生成 sql 语句。打开后，映射文件可以指定乐观锁策略 -->
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

## 根元素
根元素 `<hibernate-mapping>` 可进行一些全局的默认设置

根元素 `<hibernate-mapping>` 可包含多个 `<class>` 子元素，但一般只用于对应一个持久化类

`<hibernate-mapping>` 元素的属性有：

属性 | 解释
---|---
schema | 指定数据库的 Schema 名，如果指定则表名自动添加该 Schema 前缀
catelog | 指定数据库的 Category 名，如果指定则表名自动添加该 Category 前缀
default-cascade | 设置默认的级联风格，默认为 none
default-access | 设置默认的属性访问策略，默认为 property，即 getter 和 setter。field 则通过反射
default-lazy | 设置默认的延迟加载策略，默认为 true
auto-import | 设置是否允许在查询语言中使用非全限定类名，默认为 true。当同一映射文件有两个持久化类要映射，且类名相同（不同包中），应设置为 false
package | 指定包前缀，如果没有指定全限定类名，则默认使用该 package 前缀

## 持久化类的映射
根元素 `<hibernate-mapping>` 下的子元素 `<class>` 用于对应持久化类的映射

`<class>` 元素的属性有：

属性 | 解释
---|---
name | 指定要映射的持久化类名，如果 `<hibernate-mapping>` 元素没有指定 package，则应为全限定类名
schema | 指定数据库的 Schema 名，如果指定则表名自动添加该 Schema 前缀
catelog | 指定数据库的 Category 名，如果指定则表名自动添加该 Category 前缀
lazy | 设置延迟加载策略
table | 指定持久化类的表名，默认以持久化类的类名为表名
discriminator-value | 指定区分不同子类的值，当使用 `<subclass.../>` 元素定义持久化类的继承关系映射时需要使用该属性
polymorphism | 当采用 `<union-subclass.../>` 元素来配置继承映射时，该元素指定是否需要采用隐式多态查询。默认为 implicit
mutable | 指定持久化类的实例是可变对象还是不可变对象，默认为 true
proxy | 指定一个接口，在延迟加载时作为代理使用，也可以指定该类自己的名字
dynamic-update | 指定 update 语句是否在运行时动态生成，并且只更新改变过的字段，默认 false，开启会导致 Hibernate 需要更多时间来生成 sql 语句。打开后，映射文件可以指定乐观锁策略
optimistic-lock | 指定乐观锁定策略，默认为 version
dynamic-insert | 指定 insert 语句是否在运行时动态生成，并且只插入非空字段，默认 false，开启会导致 Hibernate 需要更多时间来生成 sql 语句
select-before-update | 指定是否在 update 前先进行 select 查询，默认为 false。如果为 true，则可保证只有当持久化对象被修改过，才执行 update
where | 指定一个附加的 sql 语句过滤条件
check | 指定一个 sql 表达式，用于为持久化类所对应的表指定一个多行的 Check 约束
persister | 指定一个定制的 ClassPersister
batch-size | 指定根据标识符来抓取实例时每批抓取的实例数，默认为 1
subselect | 该属性用于映射不可变、只读实体。通俗来讲，就是将数据库的子查询映射成持久化对象。当需要使用视图（其实质就是一个查询）来代替数据表时，该属性比较有用

## 主键与普通属性的映射
`<id>` 用于映射标识属性（主键），`<property>` 用于映射普通属性，它们都是 `<class>` 元素的子元素

`<id>` 元素的属性有：

属性 | 解释
---|---
name | 对应持久化类的属性名
type | 指定标识属性的数据类型，可以是 Hibernate 内建类型、Java 类型（全限定类名）。如果没有设置，Hibernate 会自动判断
column | 设置所映射的数据库表的字段名
unsaved-value |
access | 指定访问该标识属性的访问策略

`<id>` 元素的子元素 `<generator>` 元素用于设置主键生成器，用于为持久化实例生成唯一的逻辑主键值

`<generator>` 元素的 class 属性用于指定生成策略，可选值有：increment / identity / sequence / hilo / seqhilo / uuid / guid / native / assigned / select / foreign

`<property>` 元素的属性有：

属性 | 解释
---|---
name | 对应持久化类的属性名
type | 指定标识属性的数据类型，可以是 Hibernate 内建类型、Java 类型（全限定类名）、可序列化的 Java 类类名、自定义类的类名。如果没有设置，Hibernate 会自动判断
column | 设置所映射的数据库表的字段名
update | 设置 Hibernate 生成的 update 语句是否需要包含该字段
insert | 指定访问该标识属性的访问策略
formula | 指定一个 sql 表达式，表示值由表达式计算而来
lazy |
access |
unique |
not-null |
optimistic-lock |
generated |
index | 指定一个字符串的索引名称
unique_key | 指定唯一键的名称
length |
precision | 有效数字位的位数
scale | 小数位数

### formula 属性


## 集合类属性的映射
Hibernate 要求持久化集合值字段必须声明为接口

集合类实例具有值类型的行为：
* 当持久化对象被保存时，这些集合属性会自动被持久化
* 当持久化对象被删除时，这些集合属性对应的记录将被自动删除

集合属性默认采用懒加载

假设集合元素被从一个持久化对象传递到另一个持久化对象，该集合元素对应的记录会从一个表转移到另一个表

两个持久化对象不能共享同一个集合元素的引用

持久化对象的集合属性与在映射文件中的对应关系有：

属性 | 元素
---|---
Set | `<set>`
List | `<list>`
Collection | `<bag>` `<idbag>`
Map | `<map>`
SortedSet | `<set>`
SortedMap | `<map>`
数组 | `<array>`
基本数据类型的数组 | `<rimitive-array>`

集合元素的属性有：

属性 | 解释
---|---
name | 对应持久化类的属性名
table |
schema |
lazy |
inverse |
cascade |
order-by |
sort |
where |
batch-size |
access |
mutable |

`<key>` 子元素用于映射外键，它的属性有：

属性 | 解释
---|---
column |
on-delete |
property-ref |
not-null |
update |
unique |

在 Java 的集合类型中：Set 是无序的没有索引值，List、数组以整数作为索引值，Map 以 key 作为索引值，因此除了 `<set>` 和 `<bag>` 元素外都需要有个子元素指定索引列

用于映射索引列的元素

元素 | 用于映射索引列
---|---
`<list-index>` | List、数组
`<map-key>` | Map、基本数据类型
`<map-key-many-to-many>` | Map、实体引用类型
`<composite-map-key>` | Map、复合数据类型

用于映射集合元素的元素

元素 | 用于映射集合元素
---|---
`<element>` | 基本类型及其包装类、字符串类型、日期类型
`<composite-element>` | 复合类型
`<one-to-many>` |
`<many-to-many>` |

### 集合属性的映射举例
* Set 集合属性操作
    ```
    <set name="hobby" table="hobby_tab">
   		<key column="student_id"></key>
   		<element type="string" column="hobby"></element>
    </set>
    ```
* List 集合属性操作
    ```
    <list name="hobby" table="hobby_tab_list">
  		<key column="student_id"></key>
  		<list-index column="position"></list-index>
  		<element type="string" column="hobby"></element>
  	</list>
    ```
* Collection 集合属性操作
    ```
    private Collection<String> hobby;
    ```
    ```
    <idbag name="hobby" table="hobby_tab_c">
  		<collection-id type="string" column="hobby_id">
  			<generator class="uuid"></generator>
  		</collection-id>
		  <key column="student_id"></key>
  		<element type="string" column="hobby"></element>
  	</idbag>
    ```
* Map 集合属性操作
    ```
    <map name="hobby" table="stu_map_hobby">
  		<key column="student_id"></key>
  		<map-key column="keycolumn" type="string"></map-key>
  		<element type="string" column="hobby"></element>
  	</map>
    ```

## 关联映射
### 1 to 1
* 基于外键的单向 1 to 1    
  基于外键的单向一对一实际上是多对一关联映射的特例    
  采用 `<many-to-one>` 标签，指定多的一端的 unique=true ，这样就限制了多端的多重性为一
  ```
  public class Account {

  	private Integer id;
  	private String name;

  	//主控端引用被控端的对象
  	private Address address;
  }
  ```
  ```
  public class Address {

  	private int id;
  	private String address;
  }
  ```
  Account.hbm.xml:
  ```
  <many-to-one name="address" column="address_id" unique="true"></many-to-one>
  ```
* 基于外键的双向 1 to 1
  基于外键的双向一对一关联映射在基于外键的单向 1 to 1 的基础上，需要在另一端添加 `<one-to-one>` 标签，用 property-ref 来指定反向属性引用
  ```
  public class Account {

  	private Integer id;
  	private String name;

  	//主控端引用被控端的对象
  	private Address address_id;
  }
  ```
  ```
  public class Address {

  	private int id;
  	private String address;

  	//声明一个对主控端对象的引用
    private Account account;
  }
  ```
  Account.hbm.xml:
  ```
  <many-to-one name="address_id" column="address_id" unique="true"></many-to-one>
  ```
  Address.hbm.xml:
  ```
  // 基于外键的双向一对一关联映射需要在一端添加 <one-to-one> 标签，用 property-ref 来指定反向属性引用
  <one-to-one name="account" property-ref="address_id"></one-to-one>
  ```
* 基于主键的单向 1 to 1
  ```
  public class IDCard {

  	private Integer id;
  	private String no;

  	//主控端引用被控端的对象  	
  	private Citizen citizen;
  }
  ```
  ```
  public class Citizen {

  	private int id;
  	private String name;
  }
  ```
  IDCard.hbm.xml:
  ```
  <!--
  constrained 告诉当前主键，你的值时采用另个表中的主键的值
  当前主键对于有关系的另一个表来说就是外键。
  -->
  <one-to-one name="citizen" constrained="true"></one-to-one>
  ```
* 基于主键的双向 1 to 1
  ```
  public class IDCard {

  	private Integer id;
  	private String no;

  	//主控端引用被控端的对象  	
  	private Citizen citizen;
  }
  ```
  ```
  public class Citizen {

  	private int id;
  	private String name;

  	private IDCard idCard;
  }
  ```
  IDCard.hbm.xml:
  ```
  <one-to-one name="citizen" constrained="true"></one-to-one>
  ```
  Citizen.hbm.xml:
  ```
  <!-- 建立一对一关系 -->
  <one-to-one name="idCard" />
  ```

### 单向 1 to many
单向一对多关联映射，在对象关系映射文件中使用 `<one-to-many>` 标签映射，开发中不常见

在数据库中表的关系为：多端(Orders)有一个字段为外键指向一端(Account)
```
public class Account {

  private int id;
  private String accName;

  //对多端对象集合的引用
  private Set<Orders> setOrders;
}
```
```
public class Orders {

  private int id;
  private String orderNum;
  private Date orderTime;
}
```
Account.hbm.xml:
```
<set name="setOrders">
  <key column="acc_id"></key>
  <one-to-many class="com.jikexueyuan.entity.Orders"/>
</set>
```

### 单向 many to 1
单向多对一关联中对象模型中类之间的引用在关系模型中表示为表之间的外键引用，通过 `<many-to-one>` 标签映射多对一关联

```
// 少的一端
public class Dept {

  private int id;
  private String deptName;
}
```
```
// 多的一端
public class Employee {

  private int id;
  private String empName;
  private Date hiredate;
  //对一端的引用
  private Dept dept;
}
```
Employee.hbm.xml:
```
<many-to-one name="dept" column="dept_id"></many-to-one>
```

### 双向 1 to many ( many to 1 )
```
public class Account {

  private int id;
  private String accName;

  //对多端对象集合的引用
  private Set<Orders> setOrders;
}
```
```
public class Orders {

  private int id;
  private String orderNum;
  private Date orderTime;

  private Account account;
}
```
Account.hbm.xml:
```
<set name="setOrders" cascade="all" inverse="true">
  <key column="acc_id"></key>
  <one-to-many class="com.jikexueyuan.entity.Orders"/>
</set>
```
Orders.hbm.xml:
```
<many-to-one name="account" column="acc_id"></many-to-one>
```

#### 双向 1 to many ( many to 1 ) 自身关联
```
public class Category {

  private int id;
  private String name;

  //如果把Category看成是多的一端
  private Category parent;

  //如果把Category看成是少的一端,则需要对多的一端进行对象集合的引用
  private Set<Category> clist;
}
```
Category.hbm.xml:
```
<set name="clist" inverse="true">
    <key column="parent_id"></key>
    <!-- 配置一对多的关联映射 -->
    <one-to-many class="com.jikexueyuan.entity.Category"/>
</set>
<many-to-one name="parent" column="parent_id"></many-to-one>
```

### many to many
在关系模型中，无法直接表达两个表之间的多对多关系。需要创建一个连接表，它同时参照两个表

```
public class Course {

  private int id;
  private String name;

  //对学生集合的引用
  private Set<Student> stuList;
}
```
```
public class Student {

  private int id;
  private String name;
  private String gender;

  //对课程集合的引用
  private Set<Course> courseList;
}
```
Course.hbm.xml:
```
<set name="stuList" table="stu_course01">
    <key column="course_id"></key>
    <many-to-many column="student_id" class="com.jikexueyuan.entity1.Student"></many-to-many>
</set>
```
Student.hbm.xml:
```
<set name="courseList" table="stu_course01">
  <key column="student_id"></key>

  <many-to-many column="course_id" class="com.jikexueyuan.entity1.Course"></many-to-many>
</set>
```

或者也可以分别进行 双向 1 to many ( many to 1 )
```
public class Course02 {

  private int id;
  private String name;

  //对学生集合的引用
  private Set<StuCourse02> stuList;
}
```
```
public class Student02 {

  private int id;
  private String name;
  private String gender;

  //对课程集合的引用
  private Set<StuCourse02> courseList;
}
```
```
// 学生和课程的关系实体
public class StuCourse02 {

  private int id;
  private double score;

  private Student02 stu;
  private Course02 course;
}
```
Course02.hbm.xml:
```
<set name="stuList">
    <key column="course_id"></key>
    <one-to-many class="com.jikexueyuan.entity2.StuCourse02"/>
</set>
```
Student02.hbm.xml:
```
<set name="courseList">
  <key column="student_id"></key>
  <one-to-many class="com.jikexueyuan.entity2.StuCourse02"/>
</set>
```
StuCourse02.hbm.xml:
```
<many-to-one  name="stu"  column="student_id"/>
<many-to-one name="course"  column="course_id"/>
```

### Hibernate 中关联映射的最佳实践
* 为每个持久实体类写一个映射文件
* 不要用怪异的连接映射
* 偏爱双向关联

## 命名查询的设置

## cascade

持久化类的继承映射
discriminator-value
