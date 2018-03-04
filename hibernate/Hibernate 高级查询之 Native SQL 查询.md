Native SQL：直接使用数据库SQL语句进行查询   

```
Transaction tx=null;
Session session=null;
try{
	session=HibernateUtils.getSession();
	tx=session.beginTransaction();
	String sql="select * from acc_tab";

	// 通过Session.createSQLQuery()来获取SQLQuery接口实例
	SQLQuery query=session.createSQLQuery(sql).addEntity(Account.class);
	List<Account> list=query.list();
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

联合查询：
```
Transaction tx=null;
Session session=null;
try{
	session=HibernateUtils.getSession();
	tx=session.beginTransaction();
	String sql="select {a.*},{o.*} from acc_tab a join order_tab o on a.id=o.acc_id";
	SQLQuery query=session.createSQLQuery(sql).addEntity("a",Account.class).addEntity("o", Orders.class);
	List list=query.list();
	for(int i=0;i<list.size();i++){
		Object[] obj=(Object[])list.get(i);
		Account acc=(Account)obj[0];
		System.out.println(acc);
		Orders order=(Orders)obj[1];
		System.out.println(order);
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

调用存储过程：
```
/**
 * Hibernate调用存储过程SQL
 *
 * CREATE DEFINER=`root`@`localhost` PROCEDURE `findAccounts`(IN idValue int(20))
	begin
 		select * from acc_tab where id > idValue;
	end
 */

Transaction tx=null;
Session session=null;
try{
	session=HibernateUtils.getSession();
	tx=session.beginTransaction();
	SQLQuery query=session.createSQLQuery("{call findAccounts(:ids)}").addEntity(Account.class);
	query.setInteger("ids", 1);
	List<Account> list=query.list();
	for(Account acc:list){
		System.out.println(acc);
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
