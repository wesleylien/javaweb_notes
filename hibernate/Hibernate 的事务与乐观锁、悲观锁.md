## Hibernate 的事务
Hibernate 本身在设计时并不具备事务处理功能。Hibernate 只是将底层的 JDBCTransaction 或者 JTATransaction 进行了一下封装，在外面套上 Transaction 和 Session 的外壳，其实底层都是通过委托底层的 JDBC 或 JTA 来实现事务的调度功能

### Hibernate 声明 JDBC 事务
在 Hibernate 的配置文件中，在 `<session-factory>` 的子元素下添加
``` xml
<!-- 声明采用 JDBC 事务 -->
<!-- Hibernate 默认采用 JDBC 事务 -->
<!-- JDBC 事务是基于 Connection 连接，因而不能跨越多个数据库连接 -->
<!-- JTA 是事务服务的J2EE解决方案 -->
<property name="transaction.factory_class">org.hibernate.transaction.JDBCTransactionFactory</property>
```

### Hibernate 设置事务的隔离级别
在 Hibernate 的配置文件中，在 `<session-factory>` 的子元素下添加
``` xml
<!-- 每一种隔离级别对应着一个正整数 1 2 4 8
     分别为：Read Uncommitted、Read Committed、Repeatable Read、Serializable
-->
<property name="connection.isolation">2</property>
```

## Hibernate 中乐观锁的应用
Hibernate 在其数据库访问引擎中内置了乐观锁定实现，默认选择 version 方式作为 Hibernate 乐观锁定实现机制

实现乐观锁的步骤：
* 持久化类增加属性：
    ```
    // 提供一个乐观锁的版本号，类型为整数型即可
    private long version;

    public long getVersion() {
        return version;
    }
    // 建议 setter 设置为 private
    private void setVersion(long version) {
        this.version = version;
    }
    ```
* 持久化类对应的映射文件，在 `<class>` 的子元素下添加
    ```
    <!-- 要在 id 元素的后面 -->
    <version name="version" column="v"/>
    ```

测试：
```
session = factory.openSession();
tx = session.beginTransaction();
Account acc1 = (Account)session1.get(Account.class, 1);
session2 = factory.openSession();
tx2 = session2.beginTransaction();
Account acc2 = (Account)session2.get(Account.class, 1);

acc2.setBalance(acc2.getBalance() - 200);
// session2 提交了更新，版本号发生了变化
tx2.commit();
// session1 还是比较老的版本，所以更新失败
acc1.setBalance(acc1.getBalance() - 200);

tx1.commit();
```

## Hibernate 中悲观锁的应用
在应用程序中显式采用数据库系统的独占锁来锁定数据资源

如下几种方法时可能显式指定锁定模式为 `LockOptions.UPGRADE` ：
* 调用Session的get()或load()时
    ```
    session.load(XXX.class, params, LockOptions.UPGRADE);
    ```
* 调用Session的lock()方法时
* 调用Query的setLockMode()方法

它会生成 `select ... for update` 这样的语句来显示指定采用独占锁来锁定查询的记录
