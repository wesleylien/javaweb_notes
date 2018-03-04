## 子查询
```
Transaction tx=null;
Session session=null;
try{
session=HibernateUtils.getSession();
tx=session.beginTransaction();

String hql="from Account a where (select count(o) from a.orderList o)=0";
List<Account> list=session.createQuery(hql).list();

for(Account a:list){
  System.out.println(a);
}
tx.commit();
}catch(HibernateException he){
if(tx!=null){
  tx.rollback();
}
he.printStackTrace();
}finally{
HibernateUtils.closeSession(session);
}
```

## 连接查询
### 内连接
```
Transaction tx=null;
Session session=null;
try{
session=HibernateUtils.getSession();
tx=session.beginTransaction();

String hql="select distinct a from Account a inner join a.orderList o with o.order_time>'2015-06-10' where a.id=2";
List<Account> list=session.createQuery(hql).list();

for(Account a:list){
  System.out.println(a);
  Set<Orders> list2=a.getOrderList();
  for(Orders or:list2){
    System.out.println(or);
  }
}
tx.commit();
}catch(HibernateException he){
if(tx!=null){
  tx.rollback();
}
he.printStackTrace();
}finally{
HibernateUtils.closeSession(session);
}
```
### 左连接
```
Transaction tx=null;
Session session=null;
try{
session=HibernateUtils.getSession();
tx=session.beginTransaction();

String hql="select distinct a from Account a left join a.orderList o ";
List<Account> list=session.createQuery(hql).list();

for(Account a:list){
  System.out.println(a);
  Set<Orders> list2=a.getOrderList();
  for(Orders or:list2){
    System.out.println(or);
  }
}
tx.commit();
}catch(HibernateException he){
if(tx!=null){
  tx.rollback();
}
he.printStackTrace();
}finally{
HibernateUtils.closeSession(session);
}
```
### 右连接
```
Transaction tx=null;
Session session=null;
try{
session=HibernateUtils.getSession();
tx=session.beginTransaction();

String hql="select distinct a from Account a right join a.orderList o with o.order_time>'2015-06-10' where a.id=2";
List<Account> list=session.createQuery(hql).list();

for(Account a:list){
  System.out.println(a);
  Set<Orders> list2=a.getOrderList();
  for(Orders or:list2){
    System.out.println(or);
  }
}
tx.commit();
}catch(HibernateException he){
if(tx!=null){
  tx.rollback();
}
he.printStackTrace();
}finally{
HibernateUtils.closeSession(session);
}
```

## 抓取连接查询
fetch 连接，只使用一个查询语句就将相关联的对象或一组值的集合随着它们的父对象的初始化而被初始化

```
// 内连接 抓取连接查询
Transaction tx=null;
Session session=null;
try{
session=HibernateUtils.getSession();
tx=session.beginTransaction();

String hql="select distinct a from Account a right join fetch a.orderList o  where a.id=2";
List<Account> list=session.createQuery(hql).list();

for(Account a:list){
  System.out.println(a);
  Set<Orders> list2=a.getOrderList();
  for(Orders or:list2){
    System.out.println(or);
  }
}
tx.commit();
}catch(HibernateException he){
if(tx!=null){
  tx.rollback();
}
he.printStackTrace();
}finally{
HibernateUtils.closeSession(session);
}
```
