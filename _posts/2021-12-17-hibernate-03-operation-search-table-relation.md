---
layout: mypost
title: Hibernate学习笔记03 数据库操作 检索 表关系
categories: [Java]
---

### 数据库操作

有好几种方式实现实体类的CRUD操作

1. 对象导航

2. HQL

3. QBC

4. 本地SQL

### 对象导航

1. 查询操作：get(),load()

    获得一个持久态的对象

    根据主键查询：session.get(实体类.class, 主键值)，没找到返回null

    如果主键是int类型最好session.get(实体类.class, new Integer(主键值))

    get并不总是生成sql语句，当一级二级缓存里面没有时会生成sql语句从数据库里面获得

    get和load的区别：

    当取不到数据时get返回null，load抛出异常，因此在使用load时要确认主键id存在，从这一点来讲它没有get方便

    在延迟加载对象时，get方法仍会使用立即加载的方式发送sql语句，并得到已初始化的对象，而使用load根本不会发送sql语句，它返回一个代理对象，知道这个对象被访问时才初始化

2. 添加操作：save()

    session.save(实体类对象)，完成之后实体类变为持久状态，**对它的任何操作都会同步到数据库中，属性改变时会update，因此对一个持久态的实体类做update和save方法是没有意义的**

    根据配置的主键生成的算法，生成一个id，将实体类对象纳入session的内部缓存，事务提交时，清除缓存，将对象持久化到数据库中
    
    注意：如果设置了主键自动增长，当实体类没有设置id(主键时)，存储完成后会把增长之后的id自动赋值给实体类

    如果设置了主键自动增长，并且自己设置了id，存储的过程中id并不生效，而是使用数据库中应当生成的id，完成后id的值会被改变成再数据库中的id

    但是也可以使用save(Object ,id)方法强制指定id

    在调用save方法时并不会立即执行sql，而是等到清理缓存完成才会执行，如果save()之后有修改了实体类的属性，Hibernate将会发送一条insert语句和一条update语句来完成持久化
    因此最好是在对象状态稳定时再调用save方法，可以少执行一条语句。

3. 修改操作：update()

    使用update方法来更新托管状态的实体类。

    或者使用get方法得到持久态的实体类对象，调用对象的set方法改变属性，最后会同步到数据库中。不需要update

4. 删除操作：delete()

    负责删除一个对象，**包括持久对象和托管对对象**

    1. 查询找到实体类对象，然后delete(实体类)

    2. 建立实体类，设置实体类的主键为要删除的，然后delete(实体类)

    批量删除表`Query q = session.createQuery("delete from User");q.executeUpdate();`等价于`delete from t_user`
    效率虽然高，但是后面再做查询时会得到脏数据，即数据库中不存在的数据

5. saveOrUpdate()

    在实际应用中，程序员不会注意到一个实体对象的状态，而对脱管态调用save是不对的。对临时对象调用update也是
    不对的，为了解决这个问题产生了saveOrUpdate，当它是托管状态是系统会调用update方法，是临时对象时调用save方法

### HQL

为Hibernate Query Language的简称。具有与SQL类似的语法规范，只不过SQL针对表中字段进行查询而HQL针对持久化对象，用它来取得对象，而不进行update，delete和insert操作。

1. from子句

    ```java
    Query q = session.createQuery("from User");
	List<User> list = q.list();
    ```

2. select子句

    如果查询两个以上的属性会以数组的形式返回

    ```java
    Query q = session.createQuery("select u.id,u.name from User u");
	List<Object[]> list = q.list();
	
	for(Object[] o:list){
		System.out.println((int)o[0]+" "+(String)o[1]+" "+o.length);
	}
    ```

    但是使用数组比较麻烦，可以将Object[]中的所有成员封装为一个实体类

    ```java
    //注意实体类中要有对应的构造方法
    Query q = session.createQuery("select new User(u.id,u.name) from User u");
    List<User> list = q.list();

    for(User u:list){
        System.out.println(u.toString());
    }
    ```

3. 统计函数查询

    可以在HQL使用函数，经常用的有count() min() max() sum() avg()

    ```java
    Query q = session.createQuery("select count(*) from User");
    //太麻烦，用下面的返回唯一的结果System.out.println(q.list().get(0));
    System.out.println(q.uniqueResult());
    ```

4. where子句

    HQL也支持子查询，通过where子句实现这一机制。where子句让用户缩小要返回的实例的列表范围

    ```java
    Query q1 = session.createQuery("from User u where u.id>1");

    Query q2 = session.createQuery("from User u where u.id>?");
    //过时的方法q2.setInteger(0, 1);
    q2.setParameter(0, 1);
    ```

5. order by 子句

    asc升序desc降序

    ```java
    Query q1 = session.createQuery("from User u order by u.id desc");
    ```

6. 分页

    在HQL中没有limit，用hibernate的Query中封装的两个方法来实现

    ```java
    Query q = session.createQuery("from User");
    q.setFirstResult(3);//开始位置,下标从零开始
    q.setMaxResults(3);//每页数目
    List<User> list = q.list();
    ```

7. 连接查询/多表查询

    + inner join 内连接，可以简写为join

        `select xx from xx t1 inner join xx t2 on t1.id=t2.id`

        ```java
        Query q = session.createQuery("from User u inner join u.setRole");
		
		List<Object[]> list = q.list();
		
		for(Object[] os:list){
			System.out.println(os[0]);
			System.out.println(os[1]);
		}
        ```
    
    + left outer join 左外连接
    
    + right outer join 右外连接
    
    + full join 全连接，不常用

    + 迫切内连接

        和内联接的底层实现都是一样的内联接返回的List里面是数组，迫切内联接List里面的是对象

        ```java
        Query<User> q = session.createQuery("from User u inner join fetch u.setRole");
		
		List<User> list = q.list();
		
		for(User u:list){
			System.out.println(u);
		}
        ```

    + 迫切左外连接

### QBC

通过对Criteria对象的设置来筛选查询结果

+ 基本使用

当查询时往往需要设置查询条件。在SQL或者HQL语句中，查询条件往往放入where子句中。
此外Hibernate还还支持Criteria查询，把查询条件封装成Criteria对象

```java
Criteria cr = session.createCriteria(User.class);
cr.add(Restrictions.gt("id", 3));
cr.addOrder(Order.desc("id"));
List<User> list = cr.list();
```
常见的查询限制方法

|方法名|说明|
|:-------------:|:-----:|
|eq|equal，=|
|allEq|参数为Map对象，使用key/value进行多个等于的对比|
|gt|greater-than，>|
|lt|less-than，<|
|le|less-equal，<=|
|between|对应sql的between|
|like|对应sql的like子句，可以使用%等|
|EXACT|字符串精确匹配，相当于like value|
|ANYWHERE|字符串在中间位置，相当于like %value%|
|START|字符串在最前面位置，相当于like value%|
|END|字符串在最后面位置，相当于like %value|
|in|对应sql的in子句|
|and|and关系|
|or|or关系|
|isNull|判断属性是否为空，空返回true|
|isNotNull|与上面相反|
|asc|升序排序|
|desc|降序排序|

+ 分页：setFirstResult，setMaxResults

+ 离线查询

在封装查询提条件之前不用到session

应用场景，有时候需要在servlet中拼接查询条件而不是在dao，可以通过传递DetachedCriteria来完成

```java
DetachedCriteria detachedCriteria = DetachedCriteria.forClass(User.class);
detachedCriteria.addOrder(Order.desc("id"));//添加条件
Criteria cr = detachedCriteria.getExecutableCriteria(session);
//后面和Criteria的操作一样
```

### Native SQL 查询

直接使用本地数据库的sql语言进行查询

```java
SQLQuery sqlQuery = session.createSQLQuery("SELECT * FROM t_user");

sqlQuery.addEntity(User.class);

for(User u:(List<User>)sqlQuery.list()){
	System.out.println(u);
}
```

### 命名查询

可以将查询语句放在映射文件中

```xml
	....
    </class>
	
	<query name="selectAllUser">
		<![CDATA[
			from User
		]]>
	</query>
</hibernate-mapping>
```

```java
Query q = session.createNamedQuery("selectAllUser");	
for(User u:(List<User>)q.list()){
    System.out.println(u);
}
```

### 检索策略

hibernate检索策略分为两类

+ 立即查询：根据id查询，调用get方法，一调用get方法马上发送语句查询数据库

+ 延迟查询：根据id查询，调用load方法，并不会马上执行，只有用到对象里面的值的时候才会发送语句查询数据库

### 两类延迟查询

+ 类级别延迟

+ 关联级别延迟

### 表之间的关系

### 一对多

客户 - 联系人：一个客户可以有多个联系人，但是一个联系人只属于一个客户

+ 映射配置

    在User类(表示客户)中添加

    ```java
    private Set<LinkMan> linkMans = new HashSet<LinkMan>();
        public Set<LinkMan> getLinkMans() {
        return linkMans;
    }
    public void setLinkMans(Set<LinkMan> linkMans) {
        this.linkMans = linkMans;
    }
    ```

    User的映射文件添加

    ```xml
    <!-- 使用set表示一对多 ，column表示外键-->
    <set name="linkMans">
        <key column="user_id"></key>
        <one-to-many class="pojo.LinkMan"/>
    </set>
    ```

    在LinkMan类(联系人)中添加

    ```java
    private User user;

    public User getUser() {
        return user;
    }
    public void setUser(User user) {
        this.user = user;
    }
    ```

    在LinkMan类映射文件添加

    ```xml
    <!-- 这里配置多对一，column为外键名，要与一中的外键名相同 -->
    <many-to-one name="user" class="pojo.User" column="user_id" ></many-to-one>
    ```

    最终会在LinkMan表的属性中多一个和外键名一样的列

+ 操作

    级联保存：添加客户同时添加联系人

    ```java
    User u = new User();
    LinkMan lkm = new LinkMan();
    
    u.getLinkMans().add(lkm);
    
    session.save(u);
    session.save(lkm);
    
    transaction.commit();
    ```

    在客户的映射文件中做配置`<set name="linkMans" cascade="save-update">`,然后就可以只保存客户，联系人会自动保存
    
    级联删除：配置级联删除

    ```xml
    <set name="linkMans" cascade="save-update,delete">
			<key column="user_id"></key>
			<one-to-many class="pojo.LinkMan"/>
	</set>
    ```

    ```java
    User u = session.get(User.class, new Integer(1));
    session.delete(u);		
    transaction.commit();
    ```

    修改操作：修改联系人的所属客户

    下面存在性能问题，联系人会修改两次，双向维护外键，可以让其中一方放弃维护外键
    ```java
    //持久态，自动同步
    User u2 = session.get(User.class, 2);
    LinkMan lkm = session.get(LinkMan.class, 1);
    u2.getLinkMans().add(lkm);
    lkm.setUser(u2);
    transaction.commit();
    ```

    配置放弃维护外键

    ```xml
    <!-- 使用set表示一对多 ，column表示外键,inverse为true表示放弃关系的维护-->
    <set name="linkMans" cascade="save-update,delete" inverse="true">
        <key column="user_id"></key>
        <one-to-many class="pojo.LinkMan"/>
    </set>
    ```

### 多对多

通过建立两张表之间的联系表来实现多对多关系

一个用户有多个角色，一个角色有多个用户

+ 配置

    在User类中添加`private Set<Role> setRole = new HashSet<Role>();`和`get/set`

    User的映射文件中添加

    ```xml
    <set name="setRole" table="user_role">
        <key column="userid"></key>
        <many-to-many class="pojo.Role" column="roleid"></many-to-many>
    </set>
    ```

    在Role类中添加`private Set<User> setUser = new HashSet<User>();`和`get/set`

    Role的映射文件中添加

    ```xml
    <set name="setUser" table="user_role">
        <key column="roleid"></key>
        <many-to-many class="pojo.User" column="userid"></many-to-many>
    </set>
    ```

+ 多对多级联保存

    根据用户保存角色，在User的映射配置的set中加上`cascade="save-update"`

    ```java
    User u = new User();
    u.setName("用户1");

    Role r1 = new Role();
    r1.setName("管理员");
    Role r2 = new Role();
    r2.setName("消费者");

    u.getSetRole().add(r1);
    u.getSetRole().add(r2);

    session.save(u);
    ```

+ 多对多级联删除

    直接删除用户，会把在用户表中删除一条记录，同时在关联表中删除用户id的记录，但是角色表不变

    在映射文件的cascade属性中添加delete值，效果是在删除用户的同时，在关联表中删除用户id的记录，在角色表中删除User关联的角色
    （这样做很危险，可能会清除掉某些其他用户与角色的的关联，一般不用）

    ```java
    User u = session.get(User.class, 1);
	session.delete(u);
    ```

+ 关系表的操作

    为用户添加角色：

    ```java
    User u = session.get(User.class, 1);
    Role r = session.get(Role.class, 2);
    u.getSetRole().add(r);
    ```

    为用户删除角色：

    ```java
    User u = session.get(User.class, 1);
    Role r = session.get(Role.class, 2);
    u.getSetRole().remove(r);
    ```
