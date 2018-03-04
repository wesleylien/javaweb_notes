Session 对象的 `.createQuery("HQL语句")` 可获取 Query 对象
```
Query query = session.createQuery("HQL语句");
```
* 此外，将 HQL 查询语句编写在关系映射文件时，在程序中通过 Session 对象的 `.getNameQuery(String nameQuery)` 方法获取对应查询语句的 Query 对象，称为 —— 命名查询
* Query 对象的 `.setFirstResult(int startIndex)` 和 `.setMaxResult(int num)` 方法结合使用可实现分页功能
* Query 对象的 `.list()` 返回 List，`.iterate()` 返回 Iterator 迭代器，`.uniqueResult()` 返回单个持久化对象
* Query 对象的 `.executeUpdate()` 可用于执行更新或者删除（包括批量）
  ```
  // 更新
  String hql="update Account set password='123456' where id=:id";
  Query query=session.createQuery(hql);
  query.setInteger("id", 1);
  int i=query.executeUpdate();

  // 删除
  String hql = "DELETE  Student  WHERE name LIKE :ln";
  Query query = session.createQuery(hql);
  int count = query.executeUpdate();
  ```
* HQL 参数绑定
  ```
  // 使用 ?
  String hql="from Student a where a.id=? and a.stuName=?";

  Query query=session.createQuery(hql);
  query.setInteger(0, 3);
  query.setString(1, "wangwu");
  List<Student> list=(List<Student>)query.list();
  ```
  ```
  // 使用名称
  String hql="from Student a where a.id=:id and a.stuName=:name";

  Query query=session.createQuery(hql);
  query.setInteger("id", 3);
  query.setString("name", "wangwu");
  List<Student> list=(List<Student>)query.list();
  ```
* HQL 语句：普通查询
  ```
  from Student
  ```
* HQL 语句：投影查询（即查询类的某几个属性）
  ```
  SELECT id,name FROM Student
  ```
* HQL 语句：投影查询（即查询类的某几个属性）
  ```
  select new Student(a.stuName,a.age) from Student a
  ```
* HQL 语句：设置别名
  ```
  // AS 可省略
  select stu.id, stu.name from Student AS stu
  ```
* HQL 语句：条件查询    
  在 where 子句中可以指定：
    * . 号
    * 比较运算符：=、>、>=、<、<=、<> 、is null 、is not null
    * 范围运算符：in (值1, 值2 …) ：等于列表中的某一个值、not in(值1, 值2 …) ：不等于列表中的任意一个值、between 值1 and 值2 ：在值1到值2的范围内(包括值1和值2)、not between 值1 and 值2 ：不在值1到值2的范围内
    * 字符串模式匹配： like '字符串模式' （字符串模式中可用“%”代表任意长度的字符串，“\_”代表任意单个字符）
    * 逻辑运算： and (与)、 or (或)、not (非)
    * 用于集合的运算符：is empty、is not empty
  ```
  from Student a where a.id not between 3 and 5
  ```
* HQL 语句：去掉重复记录
  ```
  select distinct s.age,s.stuName from Student s
  ```
* HQL 语句：对结果进行排序
  ```
  // 默认为升序
  FROM  Student  AS  s ORDER BY  s.id  DESC
  ```
* HQL 语句：HQL 函数
  HQL 常见函数：
    * 字符串相关：`upper(s)` 、`lower(s)` 、`concat(s1, s2)` 、`substring(s,offset,length)` 、 `length(s)` 、`trim([[both|leading|trailing] char [from]] s)` 、`locate(search, s, offset)`
    * 数字相关：`abs(n)` 、`sqrt(n)` 、`mod(dividend, divisor)`
    * 集合相关：`size(c)`
    * 日期时间相关：`current_date()` 、 `current_time()` 、 `current_timestamp()` 返回数据库系统的日期、时间、时间戳，`year(d)` 、`month(d)` 、`day(d)` 、`hour(d)` 、`minute(d)` 、`second(d)` 从指定的参数中提取相应的值
  ```
  select new Student(upper(s.stuName)) from Student s
  ```
* HQL 语句：聚合函数    
  `count()` 、 `avg()` 、 `sum()` 、 `max()` 、 `min()`
  ```
  select min(s.age) from Student s
  ```
* HQL 语句：分组函数
  `group by` 、 `having`
  ```
  select count(s.id),s.clazz from Student s group by s.clazz having avg(s.age)>20

  SELECT  s.grade, COUNT(*)  FROM  Student  AS  s GROUP BY  s.grade

  SELECT  s.grade  FROM  Student  AS  s GROUP BY  s.grade HAVING  COUNT(s.id) > 20
  ```
