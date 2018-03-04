[Hibernate 二级缓存 总结整理](http://jinnianshilongnian.iteye.com/blog/1525884)

* 可以在**进程或集群的级别**上为事务之间可重用的数据提供二级缓存
* SessionFactory 的二级缓存是全局性的，所有 Session 都共享这个二级缓存。默认关闭
* 查询时使用缓存的实现过程为：首先查询一级缓存中是否具有需要的数据，如果没有，查询二级缓存，如果二级缓存中也没有，此时再执行查询数据库的工作
* Hibernate 并不为二级缓存提供实现方法，而是为 Hibernate 第三方缓存插件整合了接口
* 可以为每一个 **持久化类** 或每一个 **集合** 进行二级缓存的配置
* 删除、更新、增加数据的时候，同时更新缓存。Hibernate 二级缓存策略，是针对于 ID 查询的缓存策略，对于条件查询则毫无作用，为此，Hibernate 提供了针对条件查询的Query Cache 查询缓存
* 二级缓存的并发访问策略
    * 事务型：隔离级别最高 —— 适用于经常被读，很少修改的数据。可以防止脏读和不可重读的并发问题
    * 读写型：提供了 Read Commited 事务隔离级别 —— 适用于经常被读，很少修改的数据。可以防止脏读
    * 非严格读写型：适用于应用程序只是偶尔更新的数据
    * 只读型：隔离级别最低 —— 适用于重来不会修改的数据
* 适合放置在二级缓存的数据
    * 很少被修改的数据
    * 不是很重要的数据，允许出现偶尔并发的数据
    * 不会被并发访问的数据
    * 参考数据 —— 指的是供应用参考的常量数据，它的实例数目有限，它的实例会被许多其他类的实例引用，实例极少或者从来不会被修改
* 不适合存放到第二级缓存的数据
    * 经常被修改的数据
    * 财务数据，绝对不允许出现并发
    * 与其他应用共享的数据
* 二级缓存的使用
    * 添加对应的二级缓存 jar 包
    * Hibernate 配置文件
        ```
        <!-- 开启二级缓存 -->
        <property name="hibernate.cache.use_second_level_cache">true</property>
        <!-- 设置二级缓存实现类 -->
        <!-- 常见的有 EhCache 、OSCache 、SwarmCache 、JBossCache 等 -->
        <property name="hibernate.cache.provider_class">org.hibernate.cache.EhCacheProvider</property>
        ```
    * 选择需要使用二级缓存的持久化类，设置它的命名缓存的并发访问策略
        在映射文件中，在 `<class>` 的子元素下添加
        ```
        <!-- 配置二级缓存 -->
        <!-- usage 属性，指二级缓存策略，有：read-only / read-write / nonstrict-read-write（不严格读写缓存） / transcational（事务性缓存） -->
        <!-- region 属性，指定二级缓存的去域名，默认为类或者集合的名字 -->
        <!-- include 属性，all 包含所有属性，non-lazy 仅包含非延迟加载的属性 -->
        <cache usage="read-only" />
        ```
    * 新建缓存配置文件：ehcache.xml
        ```
        <ehcache>
            <!-- Sets the path to the directory where cache .data files are created.

                 If the path is a Java System Property it is replaced by
                 its value in the running VM.

                 The following properties are translated:
                 user.home - User's home directory
                 user.dir - User's current working directory
                 java.io.tmpdir - Default temp file path -->
            <diskStore path="/Users/slw/Documents/"/>


            <!--Default Cache configuration. These will applied to caches programmatically created through
                the CacheManager.

                The following attributes are required for defaultCache:

                maxInMemory       - Sets the maximum number of objects that will be created in memory
                eternal           - Sets whether elements are eternal. If eternal,  timeouts are ignored and the element
                                    is never expired.
                timeToIdleSeconds - Sets the time to idle for an element before it expires. Is only used
                                    if the element is not eternal. Idle time is now - last accessed time
                timeToLiveSeconds - Sets the time to live for an element before it expires. Is only used
                                    if the element is not eternal. TTL is now - creation time
                overflowToDisk    - Sets whether elements can overflow to disk when the in-memory cache
                                    has reached the maxInMemory limit.

                -->
            <defaultCache
                maxElementsInMemory="10000"
                eternal="false"
                timeToIdleSeconds="120"
                timeToLiveSeconds="120"
                overflowToDisk="true"
                />

            <!--Predefined caches.  Add your cache configuration settings here.
                If you do not have a configuration for your cache a WARNING will be issued when the
                CacheManager starts

                The following attributes are required for defaultCache:

                name              - Sets the name of the cache. This is used to identify the cache. It must be unique.
                maxInMemory       - Sets the maximum number of objects that will be created in memory
                eternal           - Sets whether elements are eternal. If eternal,  timeouts are ignored and the element
                                    is never expired.
                timeToIdleSeconds - Sets the time to idle for an element before it expires. Is only used
                                    if the element is not eternal. Idle time is now - last accessed time
                timeToLiveSeconds - Sets the time to live for an element before it expires. Is only used
                                    if the element is not eternal. TTL is now - creation time
                overflowToDisk    - Sets whether elements can overflow to disk when the in-memory cache
                                    has reached the maxInMemory limit.

                -->

            <!-- Sample cache named sampleCache1
                This cache contains a maximum in memory of 10000 elements, and will expire
                an element if it is idle for more than 5 minutes and lives for more than
                10 minutes.

                If there are more than 10000 elements it will overflow to the
                disk cache, which in this configuration will go to wherever java.io.tmp is
                defined on your system. On a standard Linux system this will be /tmp"
                -->
            <cache name="com.lian.entity.Account"
                maxElementsInMemory="10000"
                eternal="false"
                timeToIdleSeconds="300"
                timeToLiveSeconds="600"
                overflowToDisk="true"
                />

        	 <!-- 设置默认的查询缓存区域的属性 -->
            <cache name="org.hibernate.cache.StandardQueryCache"
        		maxElementsInMemory="50"
        		eternal="false"
        		timeToIdleSeconds="3600"
        		timeToLiveSeconds="7200"
        		overflowToDisk="true"/>
            <!-- 设置时间戳缓存区域的属性 -->
            <cache name="org.hibernate.cache.UpdateTimestampsCache"
        		maxElementsInMemory="5000"
        		eternal="true"
        		overflowToDisk="true"/>
            <!-- 设置自定义命名查询缓存区域的属性,一般可不配置 -->
            <cache name="myCacheRegion"
                maxElementsInMemory="1000"
                eternal="false"
                timeToIdleSeconds="300"
                timeToLiveSeconds="600"
                overflowToDisk="true"/>
        </ehcache>
        ```
### 查询缓存
* 对于某个**条件查询语**句经常使用相同的条件值进行查询，就可以启用查询缓存
* 只有当经常使用同样的参数值进行条件查询时，才起作用（查询缓存缓存的 key 为 hql 或 sql ）。所以，大多数查询没有必要使用查询缓存
* 查询缓存的使用
    * Hibernate 配置文件
        ```
        <!-- 打开查询缓存 -->
		<property name="hibernate.cache.use_query_cache">true</property>
        ```
    * 使用
        ```
        // 条件查询
        String hql="from Account where name like :lname";
		Query query=session.createQuery(hql);
		query.setString("lname", "zhang%");

		// 启用查询缓存
		query.setCacheable(true);

		//指定所使用的查询缓存策略
		query.setCacheRegion("myCacheRegion");

		List<Account> list=(List<Account>)query.list();
        ```

* SessionFactory 对象的 `getCache` 方法返回一个 Cache 对象
* 开启二级缓存的统计功能
    * 在 Hibernate 的配置文件中添加
        ```
        <property name="hibernate.generate_statistics">true</property>
        <property name="hibernate.cache.use_structured_entries">true</property>
        ```
    * `sessionFactory.getStatistics().getSecondLevelCacheStatistics("持久化类完整路径")` 方法返回的对象提供一些工具可分析二级缓存效果

### 在 JTA 环境使用缓存策略（未完成）

### 在集群环境使用缓存策略（未完成）
