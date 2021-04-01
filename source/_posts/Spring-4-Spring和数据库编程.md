---
title: Spring-4-Spring和数据库编程
mathjax: false
date: 2021-04-01 21:16:16
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

[toc]

&nbsp;

数据库相关内容，Spring自身提供了JDBC模板模式，即jdbcTemplate，还提供了TransactionTemplate支持事务的模板，但这些都不常用。对于持久层，通常配合Hibernate或MyBatis。对于前者，Spring提供了HibernateTemplate；后者虽由于版本原因没有直接支持，但MyBatis社区已开发了介入Spring的开发包，提供了SqlSessionTemplate，甚至使用中可以省去SqlSessionTemplate，直接使用接口编程。

&nbsp;

# 1. 配置数据库资源

通常配置为数据库连接池，可以通过Spring内部提供类、第三方数据库连接池、Web服务器JNDI数据源。对于这些第三方类，一般通过XML配置。

## 1.1 简单数据库配置

Spring内部提供的类，`org.springframework.jdbc.datasource.SimpleDriverDataSource`，简单但不支持数据库连接池，故通常用于测试。

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource"> 
	<property name="username" value="root" /> 
    <property name="password" value="123456" /> 
    <property name="driverClass" value="com.mysql.jdbc.Driver" /> 
    <property name="url" value="jdbc:mysql://localhost:3306/chapter12" /> 
</bean>
```

&nbsp;

## 1.2 使用第三方数据库连接池

如使用DBCP数据库连接池时，下载包后，就可使用

```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
	<property name="driverClassName" value="com.mysql.jdbc.Driver" />
	<property name="url" value="jdbc:mysql://localhost:3306/chapter12" />
	<property name="username" value="root" />
	<property name="password" value="123456" />
	<!--连接池的最大数据库连接数 -->
	<property name="maxActive" value="255" />
	<!--最大等待连接中的数量 -->
	<property name="maxIdle" value="5" />
	<!--最大等待毫秒数 -->
	<property name="maxWait" value="10000" />
</bean>
```

&nbsp;

## 1.3 使用JNDI数据库连接池

在Tomcat、WebLogic等JavaEE服务器上配置数据源，存在一个JNDI名称，可通过Spring提供的JNDI机制获取对应数据源。如已在Tomcat配置了JNDI为jdbc/chap的数据源，则可在Web应用中获取：

```xml
<bean id="dataSource" class="org.springframework.jndi.JndiObjectFacotryBean">
    <property name="jndiName" value="java:comp/env/jdbc/chap"/>
</bean>
```

&nbsp;

# 2. 解决代码失控 jdbcTemplate

jdbcTemplate是Spring针对JDBC代码失控提供的解决方案，给予常用技术模板化编程，减少工作量，但最终效果并不理想。此外，jdbcTemplate原生不支持事务，需引入事务管理器才支持。若事务交由事务管理器处理，则数据库连接从事务管理器获取，并且jdbcTemplate的关闭连接请求也由事务管理器决定。

配置好数据源后即可配置jdbcTemplate：

```xml
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
	<property name="dataSource" ref="dataSource" />
</bean>
```

随后即可操作jdbcTemplate，对数据库进行操作：

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("spring-cfg.xml");
jdbcTemplate jdbcTemplate = ctx.getBean(jdbcTemplate.class);
String sql = "select * from t_role where id = " + id;
Role role = jdbcTemplate.queryForObject(sql, (ResultSet rs, int rownum) ->{
    Role result = new Role();
    result.setId(rs.getLong("id"));
    result.setRoleName(rs.getString("role_name"));
    result.setNote(rs.getString("note"));
    return result;
});
```

使用了jdbcTemplate的queryForObject的方法，包含SQL和RowMapper接口两个参数。无需关闭资源等反锁代码，jdbcTemplate内部已实现。

&nbsp;

## 2.1 jdbcTemplate的增删改查

```java
public class JdbcTemplateTest {
	public static void main(String[] args) {
	    ApplicationContext ctx = new ClassPathXmlApplicationContext("spring-cfg.xml");
	    JdbcTemplate jdbcTemplate = ctx.getBean(JdbcTemplate.class);
	    
	    JdbcTemplateTest test = new JdbcTemplateTest();
//	    test.getRoleByConnectionCallback(jdbcTemplate, 1L);
//	    test.getRoleByStatementCallback(jdbcTemplate, 1L);
	    test.insertRole(jdbcTemplate);
	    List roleList = test.findRole(jdbcTemplate, "role");
	    System.out.println(roleList.size());
	    Role role = new Role();
	    role.setId(1L);
	    role.setRoleName("update_role_name_1");
	    role.setNote("update_note_1");
	    test.updateRole(jdbcTemplate, role);
	    test.deleteRole(jdbcTemplate, 1L);
	}

	public int insertRole(JdbcTemplate jdbcTemplate) {
	    String roleName = "role_name_1";
	    String note = "note_1";
	    String sql = "insert into t_role(role_name, note) values(?, ?)";
	    return jdbcTemplate.update(sql, roleName, note);
	}

	public int deleteRole(JdbcTemplate jdbcTemplate, Long id) {
	    String sql = "delete from t_role where id=?";
	    return jdbcTemplate.update(sql, id);
	}

	public int updateRole(JdbcTemplate jdbcTemplate, Role role) {
	    String sql = "update t_role set role_name=?, note = ? where id = ?";
	    return jdbcTemplate.update(sql, role.getRoleName(), role.getNote(), role.getId());
	}

	public List<Role> findRole(JdbcTemplate jdbcTemplate, String roleName) {
	    String sql = "select id, role_name, note from t_role where role_name like concat('%',?, '%')";
	    Object[] params = {roleName};
	    List<Role> list = jdbcTemplate.query(sql, params, (ResultSet rs, int rowNum) -> {
	        Role result = new Role();
	        result.setId(rs.getLong("id"));
	        result.setRoleName(rs.getString("role_name"));
	        result.setNote(rs.getString("note"));
	        return result;
	    });
	    return list;
	}
}
```

&nbsp;

## 2.2 执行多条SQL

当要多次执行SQL时，可以使用execute方法。传递ConnectionCallback或StatementCallback等接口回调。

```java
public Role getRoleByConnectionCallback(JdbcTemplate jdbcTemplate, Long id) {
	Role role = null;
	role = jdbcTemplate.execute((Connection con) -> {
		Role result = null;
		String sql = "select id, role_name, note from t_role where id = ?";
		PreparedStatement ps = con.prepareStatement(sql);
		ps.setLong(1, id);
		ResultSet rs = ps.executeQuery();
		while (rs.next()) {
			result = new Role();
			result.setId(rs.getLong("id"));
			result.setNote(rs.getString("note"));
			result.setRoleName(rs.getString("role_name"));
		}
		return result;
	});
	return role;
}

public Role getRoleByStatementCallback(JdbcTemplate jdbcTemplate, Long id) {
	Role role = null;
	role = jdbcTemplate.execute((Statement stmt) -> {
		Role result = null;
		String sql = "select id, role_name, note from t_role where id = " + id;
		ResultSet rs = stmt.executeQuery(sql);
		while (rs.next()) {
			result = new Role();
			result.setId(rs.getLong("id"));
			result.setNote(rs.getString("note"));
			result.setRoleName(rs.getString("role_name"));
		}
		return result;
	});
	return role;
}
```

通过实现这两个接口的方法，获取Connection或Statement对象，便可执行多条SQL。

&nbsp;

# 3. MyBatis-Spring项目

SSM：Spring IoC有效管理各类Java资源，即插即拔；AOP使数据库事务可以委托给Spring处理，消除冗余代码，配合MyBatis灵活可配置的特性。

MyBatis-Spring：分离业务层和模型层，且相性好，甚至可以不用SqlSessionFactory、SqlSession等对象，已封装好。

但，MyBatis-Spring项目不是Spring的子项目。Spring3发布时，MyBatis3并未完成，Spring的orm包中只支持了IBatis。MyBatis社区开发了MyBatis-Spring项目，可以[下载使用](http://mvnrepository.com/artifact/org.mybatis/mybatis-spring)。

配置MS项目步骤：

1. 配置数据源 [第一节内容](# 1. 配置数据库资源)
2. 配置SqlSessionFactory
3. ~~可选配置SqlSessionTemplate（同时配置SqlSessionFactory和SqlSessionTemplate时，SqlSessionTemplate执行优先级更高）~~
4. 配置Mapper：可以配置单个Mapper，也可扫描生成Mapper；此时IoC生成对应接口实例，通过注入获取资源
5. 事务管理

&nbsp;

## 3.1 SqlSessionFactoryBean

SqlSessionFactory是生成SqlSession的基础，MS中提供了SqlSessionFactoryBean支持SqlSessionFactory的配置。该Bean中几乎包含所有MyBatis可配置组件，并提供了setter方法让Spring设置，可通过IoC容器配置。

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="configLocation" value="classpath:mybatis_config.xml"/>
</bean>
```

此处配置了数据源，并引入了一个MyBatis配置文件，若配置内容很简单也不需引入，直接使用IoC注入。一般较复杂的配置还是需要单独配置文件，减低代码复杂性。

随后需要完成MyBatis三件套：mybatis-config.xml、mapper.xml、Mapper接口。

&nbsp;

## 3.2 ~~SqlSessionTemplate~~

**逐渐弃用**  可选配置项，线程安全，确保每个线程使用的SqlSession唯一不冲突，提供了增删改查等常用功能。

```xml
<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
	<constructor-arg ref="sqlSessionFactory" />
	<!-- <constructor-arg value="BATCH"/> -->
</bean>
```

SqlSessionTemplate要通过含参构造器创建对象，常用参数是SqlSessionFactory和MyBatis执行器Executor类型，取值范围SIMPLE、REUSE、BATCH。

使用代码：

```java
SqlSessionTemplate template = ctx.getBean(SqlSessionTemplate.class);
Role role = new Role(1, jsy);
template.insert("xxx.RoleMapper.insertRole", role);
template.selectOne("xxx.RoleMapper.getRole", role.getId());
template.update("xxx.RoleMapper.updateRole", role);
template.delete("xxx.RoleMapper.deleteRole", role.getId());
```

运行观察日志可以发现，每次使用template的方法时，都会新建一个SqlSession，所以是线程安全的。

但因操作方法是在字符串中指定，字符串不包含业务含义，只是功能性代码，不符合OOP，且无法被IDE检查正确性，故被逐渐弃用。

&nbsp;

## 3.3 配置MapperFactoryBean

MyBatis运行时只需要一个Mapper接口，无需提供实现类，实际是由MyBatis创建动态代理对象实现，故Spring也无法为其生成实现类。MS提供了MapperFactoryBean类作为中介，配置它以得到需要的Mapper。

```xml
<bean id="roleMapper" class="org.mybatis.spring.mapper.MapperFactoryBean"> 
    <property name="mapperInterface" value="xxx.RoleMapper" />
    <property name="sqlSessionFactory" ref="sqlSessionFactory" />
    <!--<property name="sqlSessionTemplate" ref="sqlSessionTemplate"/>-->
</bean>
```

随后即可`RoleMapper mapper = ctx.getBean(RoleMapper.class)`获取映射器。

&nbsp;

## 3.4 配置MapperScannerConfigurer

项目较大，有多个Mapper时，用此类以扫描方式生成Mapper，就不用3.3中的MapperFactoryBean逐个配置了。且此法无需指定bean的id，毕竟不会调用这个Bean，只是让此类扫描配置实例化Mapper，getBean时指定xxMapper.class即可。

主要配置项：

* basePackage：Spring自动扫描该包，逐层深入扫描，多个包用逗号隔开
* annotationClass：如果类被此项中配置的注解标识，才进行扫描，建议使用此法
* SqlSessionFactoryBeanName：Spring中定义SqlSessionFactory的Bean名称
* markerInterface：实现了什么接口就认为它是Mapper，需要提供一个公共接口标记

操作：

1. 将Mapper接口添加`@Repository`注解，标识DAO层（Data Access Object 数据访问层），配合annotationClass配置项，如此可以将接口放在不同包下，方便组织

2. XML配置Bean，告诉Spring扫描哪个包，及其他配置项

	```xml
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="xxx.mapper" />
		<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
		<!-- 使用sqlSessionTemplateBeanName将覆盖sqlSessionFactoryBeanName的配置 -->
		<!-- <property name="sqlSessionTemplateBeanName" value="sqlSessionFactory"/> -->
		<!-- 指定标注才扫描成为Mapper -->
		<property name="annotationClass" value="org.springframework.stereotype.Repository" />
	</bean>
	```

3. 虽然可以定义一个无意义纯标记的接口，如BaseMapper，实际用的接口继承此接口，并在XML中配置markerInterface属性，但此法不太推荐。

&nbsp;

## 3.5 测试MS

完成以上步骤（配置数据库资源、利用SqlSessionFactoryBean配置SqlSessionFactory、配置Mapper的Bean），就将Spring和MyBatis联系起来，可用Spring使用MyBatis访问数据库了。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("spring_config.xml");
BookMapper mapper = context.getBean(BookMapper.class);
var book1 = mapper.getBook("001");
System.out.println(book1.getId() + " " + book1.getTitle());
var book3 = new Book("003", "JavaScript DOM编程艺术");
mapper.insertBook(book3);
book3 = mapper.getBook("003");
System.out.println(book3.getId() + " " + book3.getTitle());
```

观察日志可发现，每执行一次Mapper接口的方法，就产生一个新的SqlSession，运行完成后自动关闭。且本例只是简单使用数据库，是非事务状态。有关MS的事务管理，在下一章讲解。

















