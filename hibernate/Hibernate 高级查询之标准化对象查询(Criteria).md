通过面向对象化的设计，将查询条件封装为一个对象。它支持在运行时动态生成查询语句

```
Transaction tx=null;
Session session=null;
try{
	session=HibernateUtils.getSession();
	tx=session.beginTransaction();
	// 创建Criteria接口对象
	Criteria cr=session.createCriteria(Account.class);
	// Critertion接口代表一个查询条件，可以通过它的实现类Restrictions类来产生查询条件，并且还需要通过Criteria的add方法添加到Criteria实例中
	Criterion c1=Restrictions.like("name", "zhang%");
	//cr.add(c1);
	Criterion c2=Restrictions.eq("age", 35);
	//cr.add(c2);
	Criterion c3=Restrictions.and(c1, c2);
	//cr.add(c3);

	// Order类对查询结果进行排序，通过Criteria的addOrder()方法添加到Criteria实例中
	// 排序方式有：Order.desc(String propertyName) / Order.asc(String propertyName)
	cr.addOrder(Order.desc("id"));

	List<Account> list=cr.list();
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

```
// 投影查询
Transaction tx=null;
Session session=null;
try{
	session=HibernateUtils.getSession();
	tx=session.beginTransaction();
	Criteria cr=session.createCriteria(Account.class);

	// Projection接口代表投影查询，它的Projections类提供了一系列产生具体Projection实例的静态方法
	// 通过Criteria的setProjection()方法添加到Criteria实例中
	// 类中的聚合函数有：
    // avg(String propertyName)
    // count(String propertyName)
    // sum(String propertyName)
    // max(String propertyName)
    // min(String propertyName)
    // ...
	cr.setProjection(Projections.max("age"));

	List<Integer> list=(List<Integer>)cr.list();
	for(Integer a:list){
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

```
// 离线查询
Transaction tx=null;
Session session=null;
try{
	session=HibernateUtils.getSession();
	tx=session.beginTransaction();
	DetachedCriteria dc=DetachedCriteria.forClass(Account.class);

	Criteria cr=dc.getExecutableCriteria(session);

	List<Account> list=(List<Account>)cr.list();
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

![Restrictions查询用法](/images/hibernate_4.png)
