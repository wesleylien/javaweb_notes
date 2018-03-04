## SessionFactory
* SessionFactory 是单个数据库映射关系经过编译后的内存镜像
* SessionFactory 的生命周期一般同应用程序的生命周期
* 线程安全
* SessionFactory 是生成 Session 的工厂
* 可以在线程或集群的级别上，为事务之间可以重用的数据提供可选的二级缓存

## Session
* Session 是程序与持久层之间交互操作的一个单线程对象
* 只有处于 Session 管理下的 POJO 才有持久化操作能力

**Session 对象的常用方法：**

方法 | 解释
---|---
save |
persist |
get |
load |
delete |
update |
updateOrSave |
merge |
flush |
setFlushMode |
evict | 清除指定对象
clear |

* Session 对象的 save 和 persist 方法的区别：
	* save 方法返回主键，persist 没有返回
	* save 方法会立即将持久化对象对应的数据插入数据库
	* persist 方法当在一个事务外被调用时，并不立即转化成 insert 语句

* Session 对象的 load 和 get 方法区别：
	* 在延迟加载时，load 返回一个未初始化的代理对象，直到程序调用代理对象的方法时，才访问数据库
	* get 立即访问数据库
	* 没有对应数据时，load 返回 HibernateException 异常，get 返回 null
	* load 或 get 方法都可指定“锁模式”参数(READ / UPGRADE)来代表共享、修改锁

* Session 对象的 update 、updateOrSave 、merge 方法区别：
	* update 可保存脱管对象所做的修改
	* 当不知道对象是否曾经持久化过，可使用 updateOrSave
	* merge 方法不会持久化给定对象，当 Session 存在同持久化表示的持久化对象时，merge 提供的对象将覆盖；没有时，则尝试从数据库加载，或创建新的持久化对象，并返回该对象

### Session 缓存（一级缓存）
* Session 有一个必选的一级缓存，称为一级缓存
* Session 缓存原理
    * 当程序调用Session 的 CRUD 方法时，如果在 Session 的缓存中不存在相应的对象，Hibernate 会把对象加到一级缓存
    * 当清理缓存时，Hibernate 会根据缓存中对象状态的变化来同步更新缓存
* Session缓存的作用
    * 减少访问数据库的频率
    * 保证缓存中的对象与数据库中的数据同步
    * 当缓存中的持久化对象之间存在循环关联关系时，Session会保证不出现访问对象图的死循环，以及由死循环引起的JVM堆栈溢出异常
* 当调用 Transaction 的 commit 方法时，会清理缓存，再向服务器提交事务
* 当显式调用 Session 的 flush 方法时，通过 Session 的 setFlushMode 方法来设定清理缓存的时间点( ALWAYS / AUTO / COMMIT / MANUAL )
* Session 的 evict 方法能将占用大量内存的对象或集合属性从一级缓存中剔除出去。contains 方法可判断某个对象是否在缓存中，clear 方法清除所有缓存

## 持久化对象的状态
持久化对象的状态：持久化、瞬态、托管

* 瞬态：刚 new 出来
* 持久化
  * 持久化实例在数据库中有对应的记录，并拥有一个持久化标识（一般为主键）
  * 持久化对象必须与指定的 Session 关联
  * Hibernate 会检测持久化对象的改动，在当前操作完成时将对象数据写回数据库，不需要手动执行 update

      ```
      Student stu=new Student();
      stu.setName("zhangsan");
      stu.setAge(20);

      session.save(stu);
      // 此时 stu 是持久化对象，修改会保存到数据库
      stu.setAge(30);
      ```
* 脱管
  * Session 关闭或 clear 则对象变成脱管
  * 让脱管对象重新与某个 Session 关联，则重新转变为持久化，脱管期间的改动也不会丢失，也可写入数据库
* 瞬态 -> 持久化
  * save()、saveOrUpdate()
  * get()、load()、find()
* 持久化 -> 瞬态
  * delete()
* 持久化 -> 脱管
  * evict()
  * close()
  * clear()
* 脱管 -> 持久化
  * save()、saveOrUpdate()
  * lock()
