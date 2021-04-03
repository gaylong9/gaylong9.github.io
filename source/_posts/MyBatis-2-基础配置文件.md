---
title: MyBatis-2-基础配置文件
mathjax: false
date: 2021-03-23 09:59:46
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

[toc]

# MyBatis配置文件

```xml
<configuration>	<!--配置-->
    <properties/>	<!-- 属性 -->
    <settings/>	<!-- 设置 -->
    <typeAliases/>	<!-- 类型别名 -->
    <typeHandlers/>	<!-- 类型处理器 -->
    <objectFactory/>	<!-- 对象工厂 -->
    <plugins/>	<!-- 插件 -->
    <environments>	<!-- 配置环境 -->
        <environment>	<!-- 环境变量 -->
            <transactionManager/>	<!-- 事务管理器 -->
            <dataSource/>	<!-- 数据源 -->
        </environment>
    </environments>
    <databaseIdProvider/>	<!-- 数据库厂商标识 -->
    <mappers/>	<!-- 映射器 -->
</configuration>
```

配置项顺序不能改变。

对象工厂和数据库厂商标识不常用。

## 1. properties 属性

给系统设置一些运行参数，可放在XML或properties文件中，方便修改，不会引起重新编译。MyBatis有三种方式使用properties：property子元素、properties文件、程序代码传递。

1. property子元素

在上述基础配置文件的`<properties/>`中，改写为：

```xml
<properties>
    <property name="database.driver" value="com.mysql.jdbc.Driver"/>
    <property name="database.url" value="jdbc:mysql://localhost:3306/ssm"/>
    <property name="database.username" value="jsy"/>
    <property name="database.password" value="123"/>
</properties>
...
<dataSource type="POOLED">
    <property name="driver" value="${database.driver}"/>
    <property name="url" value="${database.url}"/>
    <property name="username" value="${database.username}"/>
    <property name="password" value="${database.password}"/>
</dataSource>
```

这里在properties属性中定义了几个子属性，并在数据源中进行调用，实现了一次定义，多次使用。但如果需要的子属性过多，就使基础配置文件过长，应改用properties文件。

2. properties文件

	由于本方法文件简单，逻辑只是键值对，易维护，故比较普遍。

	properties文件：jdbc.properties

```properties
database.driver  = com.mysql.jdbc.Driver
database.url = jdbc:mysql://localhost:3306/ssm
database.username = root
database.password = 123
```

并在基础配置文件的properties属性中引用此文件：

`<properties resource="jdbc.properties"/>`，此后也可在基础配置文件中用`${database.xxx}`引用属性了。

3. 实际生产中，数据库的帐密自然需要加密，要将密文配置进properties文件，并在运行代码时，临时解密再重新配置。这个代码运行时机，是在创建SqlSessionFactory前：

```java
InputStream in = Resources.getResourceAsStream("jdbc.properties");
Properties props = new Properties();
props.load(in);
String username = props.getProperty("database.username");
String password = props.getProperty("database.password");
// 解密
props.put("database.username", CodeUtils.decode(username));
props.put("database.password", CodeUtils.decode(password));
InputStream inputStream = Resource.getResourceAsStream("mybatis-config.xml");
// 使用程序传递的方式覆盖原有properties属性参数
sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream, props);
```

代码首先用Resources读取一个jdbc.properties配置文件，获取加密后的帐密，进行解密并重置。最后使用SqlSessionFactoryBuilder的build方法，传递多个properties参数，覆盖原有配置。

解密代码CodeUtils.decode()需自行构建。

4. 三种配置properties的方法，优先级依次升高，若多种方式同时使用，高优先级会覆盖低优先级。

&nbsp;

## 2. settings 设置

是MyBatis中最复杂的设置，影响底层运行，但默认设置就可运行，故通常简单配置即可。

settings配置项：[完整配置项](https://mybatis.org/mybatis-3/zh/configuration.html#settings) 有很多，但常用的不多，如缓存的cacheEnabled，级联的lazyLoadingEnabled、aggressiveLazyLoading，自动映射的autoMappingBehavior和mapUnderscoreToCamelCase，执行器类型的defaultExecutorType等。

配置方法：

```xml
<settings>
    <setting name="cacheEnabled" value="true"/>
    ...
</settings>
```

&nbsp;

## 3. typeAliases 别名

代替类的全限定名，分为系统定义别名和自定义别名。MyBatis中不区分大小写。别名由类org.apache.ibatis.type.TypeAliasRegistry定义。

1. TypeAliasRegistry类定义别名实现：

此外，Configuration对象本身已对一些常用配置项配置了别名，如：

```java
typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);
typeAliasRegistry.registerAlias("POOLED", PooledDatasourceFactory.class);
```

2. 系统定义别名

	在MyBatis初始化过程中，系统自动初始化了一些[别名](https://mybatis.org/mybatis-3/zh/configuration.html#typeAliases)，基础数据类型用\_前缀，如\_int表示int，而int表示Integer。包括上述Configuration已定义的别名，系统定义别名不可重复定义。

3. 自定义别名

	4.  配置文件中进行自定义，但类多时不方便

```xml
<typeAliases>
	<typeAlias alias="Book" type="xxx.Book"/>
</typeAliases>
```

​	2. 扫描别名:MyBatis将扫描这个包里的类，将其第一个字母小写作为其别名，如Book类将取book别名。另可在类头用注解标注`@Alias("user2")`以防止自动别名时的重名冲突

```xml
<typeAliases>
    <package name="xxx.pojo"/>
</typeAliases>
```

## 4. typeHandler 类型转换器

JDBC中，需要在PreparedStatement对象中设置已预编译过得SQL语句的参数。执行SQL后，会通过ResultSet获取数据。这些操作，MyBatis是通过typeHandler实现的。它的作用就是数据库类型jdbcType和Java类型javaType之间的转换。通常MyBatis会自动探测，不需手动配置，若数据类型特殊，就要自定义typeHandler。

也分为系统typeHandler和自定义typeHandler，系统的大部分场景够用，若数据类型特殊，需要特殊转换规则等就要自定义。

1. 系统定义的typeHandler，[完整](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)
2. 自定义typeHandler：如枚举，需要特殊转化规则。自定义时，需要仿照系统定义时的步骤，实现TypeHandler接口或集成BaseTypeHandler类。本例实现接口：

```java
public class MyTypeHandler implements TypeHandler<String> {
    // 泛型String表示将数据库数据类型转化为String
    @Override
    public void setParameter(PreparedStatement ps, int i, String parameter, jdbcType jdbcType) throws SQLException {
        ps.setString(i, parameter);
    }
    
    @Override
    public String getResult(ResultSet rs, String columnName) throws SQLException {
		return rs.getString(columnName);
    }
    
    @Override
	public String getResult(ResultSet rs, int columnIndex) throws SQLException {
        return rs.getString(columnIndex);
    }
    
    @Override
    public String getResult(CallableStatement cs, int columnIndex) throws SQLException {
        return cs.getString(columnIndex);
    }
}
```

泛型String表示将数据库数据类型转化为String，随后实现设置参数和获取结果集的方法。随后在配置文件中配置：

```xml
<typeHandlers>
    <typeHandler jdbcType="CARCHAR" javaType="string" handler="xxx.MyTypeHandler" />
</typeHandlers>
```

配置完成后系统才会读取到，如此注册表示当jdbcType和javaType能与MyTypeHandler对应时，就启动MyTypeHandler，也可手动启动。

不同启动方式：mapper.xml中

```xml
<resultMap id="bookMapper" type="role">
    <result property="id" column="id"/>
    <result property="bookName" column="book_name" jdbcType="VARCHAR" javaType="string" />
    <result property="note" column="note" typeHandler="xxx.MyTypeHandler"/>
</resultMap>
...
<select id="" paramterType="string" resultMap="bookMapper">
    select id, book_name from books where book_name like concat('%', #{bookName, jdbcType=VARCHAR, javaType=string}, '%')
</select>

<select id="" paramterType="string" resultMap="bookMapper">
     select id, book_name from books where book_name not like concat('%', #{bookName, typeHandler=xxx.MyTypeHandler}, '%')
</select>
```

即，要么指定与自定义typeHandler一致的jdbcType和javaType，要么直接使用typeHandler指定具体实现类。

3. 自定义typeHandler之包扫描：枚举类太多时，自定义typeHandler复杂，采用包扫描，不过这样无法指定jdbcType和javaType，只能使用注解处理`@MappedTypes(String.class) @MappedjdbcTypes(jdbcType.VARCHAR)`

```xml
<typeHandlertype>
    <package name="xxx.typehandler"/>
</typeHandlertype>
```

4. 枚举typeHandler：多数情况下是因为枚举才自定义typeHandler，MyBatis已定义了两个类所谓枚举类型的支持：EnumOrdinalTypeHandler和EnumTypeHandler。使用不多，暂不展开。

5. 文件操作：MyBatis对数据库的Blob字段也进行了支持，提供了一个BlobTypeHandler和ByteArrayTypeHandler，不常用，暂不展开。

&nbsp;

## 5. ObjectFactory 对象工厂

创建结果集时，MyBatis使用一个对象工厂完成创建。默认使用其定义的org.apache.ibatis.reflection.factory.DefaultObjectFactory完成。

如需自定义对象工厂，则要实现结果ObjectFactory或集成DefaultObjectFactory。

```java
public class MyObjectFactory extends DefaultObjectFactory {
    private static final long serialVersionUID = -9955122346740814948L;
    private Object temp = null;
    @Override
    public void setProperties(Properties properties) {
        super.setProperties(properties);
    }
    
    @Override
    public <T> T create(Class<T> type) {
        T result = super.create(type);
        return result;
    }
    
    @Override
    public <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
        T result = super.create(type, constructorArgTypes, constructorArgs);
        temp = result;
        return result;
    }
    
    @Override 
    public <t> boolean isConllection(Class<T> type) {
        return super.isCollection(type);
    }
}
```

配置：

```xml
<objectFactroy type="xxx.MyObjectFactory">
    <property name="prop1" value="value1"/>
</objectFactroy>
```

如此，MyBatis就会采用配置的MyObjectFactory生成结果集对象。

&nbsp;

## 6. plugin 插件

灵活但复杂，会覆盖MyBatis底层对象的核心方法和属性，十分危险，需要清除掌握MyBatis底层的构成和原理才可使用。

&nbsp;

## 7. environments 运行环境

配置数据库信息，可以配置多个数据库`<environment>`，又下分事务管理器transactionManager和数据源dataSource。实际工作中一般使用Spring对数据源和数据库事务进行管理。

1. transactionManager事务管理器

	2.  需要实现接口Transaction，包含getConnection、commit、rollback、close、getTimeout几个方法。

	3. 提供两个实现类JdbcTransaction和ManagedTransaction，对应两种工厂JdbcTransactionFactory和ManagedTransactionFactory，都要实现TransactionFactory接口

		综上，配置时就可配置两种类型的事务管理器

		`<transactionManager type="JDBC"/>`

		`<transactionManager type="MANAGED"/>`

	4. JDBC使用JdbcTransactionFactory生成JdbcTransaction对象，以JDBC的方式对数据库的提交和回滚进行操作。

		MANAGED使用ManagedTransactionFactory生成ManagedTransaction对象，其提交和回滚不用任何操作，而是把事务交给容器处理。默认情况下会关闭连接，而容器不一定希望，故需将closeConnection设为false。

	5. 也可自定义事务管理器 `<transactionManage type="xxx.MyTransactionFactroy"/>`，

```java
public class MyTransactionFactory implements TransactionFactroy {
    @Override
    public void setProperties(Properties prop) {}
    
    @Override
    public Transaction new Transaction(Connection conn) {
        return new MyTransaction(conn);
    }
    
    @Override
    public Transaction newTransaction(DataSource dataSource, TransactionIsolationLevel level, boolean autoCommit) {
        return new MyTransaction(dataSource, level, autoCommit);
    }
}
```

```java
public class MyTransaction extends JdbcTransaction implements Transaction {
    public MyTransaction (DataSource ds, TransactionIsolationLevel level, boolean desiredAutoCommit) {
        super(ds, level, desiredAutoCommit);
    }
    
    public MyTransaction(Connection conn) {
        super(conn);
    }
    
    @Override
    public Connection getConnection() throws SQLException {
        return super.getConnection();
    }
    
    @Override
    public void comit() throws SQLException {
        super.commit();
    }
    
    @Override 
    public void rollback() throws SQLException {
        super.rollback();
    }
    
    @Override 
    public void close() throws SQLException {
        super.close();
    }
    
    @Override 
    public Integer getTimeout()  throws SQLException {
        return super.getTimeout();
    }
    
}
```

2. dataSource 数据源环境
	1. 用于配置数据库，MyBatis中数据库通过Pool鹅的DataSourceFactory、UnpooledDataSourceFactory、JNDIDataSourceFactory三个工厂提供，前两者产生PooledDataSource、UnpooledDataSource对象，而最后一个工厂会根据JNDI的信息拿到外部容器实现的数据库连接对象。无论是哪个工厂，最后生产的对象都是一个实现了DataSource接口的数据库连接对象。
	2. 配置方式 `<dataSource type="UNPOOLED/POOLED/JNDI">`
	3. UNPOOLED：采用非数据库池的管理方式，每次请求都会开打一个新的数据库连接，所以创建慢。下有属性：driver、url、username、password、defaultTransactionIsolationLevel默认连接事务隔离等级，另可传递属性给驱动，如`driver.encoding=UTF8`，会通过`DriverManager.getConnection(url, driverProperties)`传递值为UTF8的encoding属性给驱动
	4. POOLED：初始已有连接，请求时无需再次建立，省时，下还有属性poolMaximumActiveConnections（最大正在使用连接数，10）、poolMaximumIdleConnections（最大空闲连接数）、poolMaximumCheckoutTime（强制返回前池中连接被检出的时间，20s）、poolTimeToWait（获取连接过久就重新尝试获取，20s）、poolPingQuery（发送到数据库的侦测查询，检验连接是否正常工作）、poolPingEnabled（是否启用侦测查询，若开启则需要使用一个可执行的SQL语句设置poolPingQuery属性，false）、poolPingConnectionsNotUsedFor（配置poolPingQuery的使用额度，匹配具体的数据库连接超时时间，默认0）
	5. JNDI：数据源JNDI的实现是为了能在EJB或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个JNDI上下文的引用，此时只需配置两个属性：initial_context（在InitialContext中寻找上下文，可选，若忽略则data_source直接从InitialContext中寻找）、data_source（引用数据源实例位置上下文的路径，若initial_context提供配置时就在其返回的上下文中寻找，若无就在InitialContext中寻找）。此外，可以通过添加前缀“env.”直接把属性传递给InitialContext，如`env.encoding=UTF8`，就会在InitialContext实例化时向构造方法中传递此属性。最后，MyBatis支持第三方数据源，如DBCP，则需要提供一个自定义的DataSourceFactory，实现DataSourceFactory接口，在配置中的type属性处添加类的全限定名

&nbsp;

## 8. databaseIdProvider 数据库厂商标识

主要用于支持多种不同厂商的数据库，但并不常用。如软件提供不同数据库，需要客户决定使用哪个时才用到。

1. 系统默认的databaseIdProvider：

	如使用Oracle和MySQL，则配置：

```xml
<databaseIdProvider type="DB_VENDOR" >
    <property name="Oracle" value="oravle"/>
    <property name="MySQL" value="mysql"/>
    <property name="DB2" value="db2"/>
</databaseIdProvider>
```

name是数据库的嘛形成，如不确定，可使用JDBC创建其数据库连接对象Connection，然后通过代码`connection.getMetaData().getDatabaseProductName()`获取。

value是别名，在MyBatis中通过别名标识一条SQL由哪种数据库运行。

随后在Mapper中改造SQL：

```xml
<selct id="getBook" parameterType="long" resultType="book" databaseId="oracle">
    ...
</selct>
```

MyBatis先找设置了databaseId的SQL语句，若无则找没设置的SQL，若还无则报错。

2. 不使用系统规则，自定义规则，要实现DatabaseIdProvider接口，暂不展开。

&nbsp;

## 9. 引入映射器的方法

映射器是MyBatis中最核心的组件，本节只讨论如何引入映射器。

映射器定义了命名空间namespace的方法，命名空间对应的是一个接口的全路径，而非实现类。

首先定义接口：

```java
public interface BookMapper {
    public Book getBook(int id);
}
```

随后Mapper的XML： bookMapper.xml

```xml
<mapper namespace="xxx.BookMapper">
    <select id="getBook" parameterType="int" resultType="xxx.Book">
        select ...
    </select>
</mapper>
```

以上创建了Mapper，以下是在基础配置文件中引入映射器的几种方法：

1. 用文件路径引入：

```xml
<mappers>
    <mapper resource="xxx.bookMapper.xml"/>
</mappers>
```

2. 用包名引入：`<mapper name="xxx.mapper"/>`

3. 用类注册引入：`<mapper class="xxx.BookMapper"/>` 实为该接口位置

4. 用userMapper.xml引入：`<mapper url="file:///var/mappers/xxx/bookMapper.xml"/>`？

&nbsp;