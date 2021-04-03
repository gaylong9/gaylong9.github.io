---
title: MyBatis-1-核心组件
mathjax: false
date: 2021-03-23 09:58:22
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

[toc]



# MyBatis核心组件

[github](https://github.com/mybatis/mybatis-3)

## 1. 持久层概念与MyBatis特点

1.  持久层：将业务数据存到磁盘，具有长期存储能力。一般执行持久任务的都是数据库系统。磁盘空间大且廉价，但速度慢。对于高并发任务可加入Redis等NoSQL。
2.  MyBatis：Java Web应用可以通过本框架访问数据库。
3.  MyBatis优点：
	1.  不屏蔽SQL，可以优化SQL语句
	2.  灵活的映射机制，动态SQL，可以根据情况组装SQL
	3.  提供了使用Mapper的接口编程，只需一个接口和一个xml就能创建映射器，工作简化

&nbsp;

## 2. 核心组件

共有4个核心组件：

1. SqlSessionFactoryBuilder：构造器，根据配置或代码生成SqlSessionFactory，采用的是分步构建的Builder模式（在Java设计模式中有介绍）
2. SqlSessionFactory：工厂接口，负责生成SqlSession，使用工厂模式
3. SqlSession：会话，既可以发送SQL执行返回结果，也可以获取Mapper的接口；当前它不出现在业务逻辑代码中，而是使用MyBatis的SQL Mapper接口编程技术，提高可读性与维护性
4. SQL Mapper：映射器，MyBatis新设计出的组件，由一个接口和xml/注解构成，给出对应的SQL和映射规则，负责发送SQL去执行，并返回结果
5. 注：SQL Mapper只是对SqlSession的改进，二者都能发送SQL给数据库执行

&nbsp;

## 3. SqlSessionFactory 工厂接口

要使用MyBatis，首先要用配置或代码生成SqlSessionFactory，用SqlSessionFactoryBuilder完成。它提供了类org.apache.ibatis.session.Configuration作为引导，Builder模式，分步创建过程就在Configuration中完成。即SqlSessionFactoryBuilder和Configuration共同生成工厂接口。

虽然代码和xml配置均可生成工厂接口，但xml配置后修改起来更方便。配置完成后，MyBatis读取配置文件，通过Configuration对象构建MyBatis的上下文。

本接口有两个实现类：SqlSessionManager和DefaultSqlSessionFactory。通常由后者实现，前者适用于多线程。

每个MyBatis应用都以一个SqlSessionFactory的实例为中心，其唯一作用就是生产MyBatis的核心接口对象SqlSession。因其作用唯一，故常用单例模式。

1. XML构建工厂接口

MyBatis中有两类XML，一类是基础配置文件，通常仅一个，配置一些基本的上下文参数和运行环境；另一类是映射文件，配置映射关系、SQL、参数等信息。

```xml
<!-- 基础配置文件 -->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration   PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <typeAliases><!-- 别名 -->
      <typeAlias alias="role" type="com.learn.ssm.chapter3.pojo.Role"/>
  </typeAliases>
  <!-- 数据库环境 -->
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/chapter3"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
      </dataSource>
    </environment>
  </environments>
  <!-- 映射文件 -->
  <mappers>
    <mapper resource="com/learn/ssm/chapter3/mapper/RoleMapper.xml"/>
    <mapper class="com.learn.ssm.chapter3.mapper.RoleMapper2"/> 
  </mappers>
</configuration>
```

typeAliases给Role类定义别名，在上下文中用别名代替全限定名。

environment定义数据库，dataSource type="POOLED"配置数据库，采用MyBatis提供的连接池方式。

mapper元素引入映射器，若映射器是XML构建，则如上述代码 使用resource属性，若是代码构建，则使用class=“xxx.RoleMapper”。

通过基础配置文件即可生成工厂接口：

```java
SqlSessionFactory factory = null;
String resource = "mybatis-config.xml";
InputStream in;
try {
    in = Resource.getResourceAsStream(resource);
    factory = new SqlSessionFactoryBuilder().build(in);
} catch (IOException e) {
    e.printStackTrace();
}
```

读取基本配置文件，通过SqlSessionFactoryBuilder的build方法创建工厂接口。具体实现繁琐，MyBatis通过Builder模式隐藏了。

2. 代码构建工厂接口

```java
// 数据库连接池信息
PooledDataSource dataSource = new PooledDataSource();
dataSource.setDriver("com.mysql.jdbc.Driver");
dataSource.setUsername("root");
dataSource.setPassword("123");
dataSource.setUrl("jdbc:mysql://localhost:3306/ssm");
dataSource.setDefaultAutoCommit(false);
// MyBatis的JDBC事务方式
TransactionFactory transfactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transfactory, dataSource);
// 创建Configuration对象
Configuration conf = new Configuration(environment);
// 注册MyBatis上下文别名
conf.getTypeAliasRegistry().registerAlias("role", Role.class);
// 加入一个映射器
conf.addMapper(RoleMapper.class);
// 使用Builder构建工厂接口
SqlSessionFactory factory = new new SqlSessionFactoryBuilder().build(conf);
return factory;
```

如果系统修改，需要重新编译代码，本方式并不好。除非有特殊需要，如在配置文件中，需要配置加密过的用户名和密码，需要在生成工厂接口时先解密，才考虑代码构建。

&nbsp;

## 4. SqlSession

MyBatis中SqlSession是核心接口，有两个实现类，DefaultSqlSession和SqlSessionManager。前者单线程，后者多线程。本接口类似JDBC中的Connection对象，代表一个连接资源的启用。

作用：获取Mapper接口、发送SQL给数据库、控制数据库事务。

创建：`SqlSession sqlSession = sqlSessionFactory.openSession();`

用SqlSession控制数据库事务：

```java
SqlSession sqlSession = null;
try {
    sqlSession = sqlSessionFactory.openSession();
    // 增删改查
    sqlSession.commit();
} catch (Exception e) {
    sqlSession.rollback();
} finally {
    if (sqlSession != null) {
        sqlSession.close();
    }
}
```

&nbsp;

## 5. Mapper 映射器

是MyBatis中最重要、最复杂的组件，由一个接口和对应的XML/注解组成，可用于：描述映射规则；提供SQL语句，并可配置SQL参数类型、返回类型、缓存刷新等；配置缓存；提供动态SQL。

映射器只需接口，不用实现类。MyBatis采用动态代理技术，会为接口生成一个代理对象，故仅接口也能运行。

使用映射器前首先需要一个POJO，用于映射查询结果。

```java
public class Role {
    private Long id;
    private string roleName;
    // setter getter
}
```

1. 用XML实现映射器

接口：

```java
public interface RoleMapper {
    public Role getRole(Long id);
}
```

XML：

```xml
<mapper namespace="xxx.RoleMapper">
    <select id="getRole" parameterType="long" resultType="role">
        select id, role_name as roleName from t_role where id = #{id}
    </select>
</mapper>
```

MyBatis默认提供自动映射，只要SQL返回的列名能和POJO对应，就能自动映射为一个POJO对象。

注：XML创建SqlSessionFactory时，配置文件中有一个`<mapper resource=".../RoleMapper.xml"/>`，就是用于指定此处的XML。

2. 用注解实现映射器

注解实现时仅需一个接口，可以通过MyBatis的注解来注入SQL。

```java
public interface RoleMapper {
    @Select("select id, role_name as roleName from t_role where id = #{id}")
    public Role getRole(Long id);
}
```

如果两种方式同时实现，则XML优先级更高。但推荐XML方式。

&nbsp;

## 6. 两种发送SQL方式

1. 用SqlSession发送SQL `Role role = (Role) sqlSession.selectOne("xxx.RoleMapper.getRole", 1L)`，这是iBatis时代留下的方式

2. 用Mapper接口发送SQL 

```java
RoleMapper mapper = sqlSession.getMapper(RoleMapper.class);
Role role = roleMapper.getRole(1L);
```

3. 后一个方式更佳

&nbsp;

## 7. 生命周期

1. SqlSessionFactoryBuilder：用于创建SqlSessionFactory，故创建成功后就应删除
2. SqlSessionFactory：可看作数据库连接池，用于生成SqlSession，故应和MyBatis同样周期。同时为了避免过度消耗数据库连接资源，通常使用单例模式
3. SqlSession：相当于一个数据库连接，故应存活在一次业务请求中，处理结束后应关闭
4. Mapper：由SqlSession创建的接口，故最多和SqlSession一致，请求结束后就销毁

&nbsp;

## 8. 实例

1. 需要的文件：

 2. Book.java POJO
	1. BookMapper.java 映射器接口
	2. BookMapper.xml 映射器XML
	3. mybatis-config.xml 基础配置文件
	4. SqlSessionFactoryUtils.java 工具类，创建工厂，获取SqlSession

3. POJO

```java
public class Book {
    private int id;
    private String title;
    // getter setter
}
```

3. Mapper-接口：

```java
public interface BookMapper {
    public int insertBook (Book book);
    public int deleteBook (int id);
    public int updateBook (Book book);
    public Book getBook (int id);
    public List<Book> findBooks (String bookName);
}
```

4. Mapper-XML：

```xml
<mapper namespace="MyBatis_test.BookMapper">
    <insert id="insertBook" parameterType="book">
        insert into books (id, title) values (#{id}, #{title})
    </insert>

    <delete id="deleteBook" parameterType="string">
        delete from books where id = #{id}
    </delete>

    <update id="updateBook" parameterType="book">
        update books set id = #{id}, title = #{title} where id = #{id}
    </update>

    <select id="getBook" parameterType="string" resultType="book">
        select id, title from books where id = #{id}
    </select>
</mapper>
```

5. 基础配置文件 mybatis_config.xml

```xml
<configuration>
    <typeAliases>
        <typeAlias alias="book" type="MyBatis_test.Book"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/bookdb"/>
                <property name="username" value="jsy"/>
                <property name="password" value="jiumima1870353-"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="book_mapper.xml" />
    </mappers>
</configuration>
```

6. 此时前期设置完成，接下来需要创建工具类用于创建工厂

```java
public class SqlSessionFactoryUtils {
    private final static Class<SqlSessionFactoryUtils> LOCK = SqlSessionFactoryUtils.class;
    private static SqlSessionFactory sqlSessionFactory = null;
    private SqlSessionFactoryUtils() {}
    public static SqlSessionFactory getSqlSessionFactory() {
        synchronized (LOCK) {
            if (sqlSessionFactory != null) {
                return sqlSessionFactory;
            }
            String resource = "mybatis_config.xml";
            InputStream in;
            try {
                in = Resources.getResourceAsStream(resource);
                sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
            } catch (IOException e) {
                e.printStackTrace();
                return null;
            }
            return sqlSessionFactory;
        }
    }
    
    public static SqlSession openSqlSession() {
        if (sqlSessionFactory == null) {
            getSqlSessionFactory();
        }
        return sqlSessionFactory.openSession();
    }
}
```

全部private属性保证单例化；synchronized锁保证多线程中也不会被多次实例化；共同保证工厂对象的唯一性。

openSqlSession方法在此处实现的话，可以实现对外隐藏工厂接口，调用时直接用Utils.openSqlSession()即可获得SqlSession对象。

7. 运行测试：

```java
public class Test {
    public static void main(String[] args) {
        Logger log = Logger.getLogger(String.valueOf(Test.class));
        SqlSession sqlSession = null;
        try {
            sqlSession = SqlSessionFactoryUtils.openSqlSession();
            BookMapper mapper = sqlSession.getMapper(BookMapper.class);
            Book book = mapper.getBook("001");
            log.info(book.getId() + " " + book.getTitle());
        } finally {
            if (sqlSession != null) {
                sqlSession.close();
            }
        }
    }
}
```

&nbsp;

