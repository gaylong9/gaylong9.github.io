---
title: MyBatis-3-映射器文件
mathjax: false
date: 2021-03-23 10:00:35
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

[toc]



# 映射器

由一个接口和XML文件（注解不推荐）组成。映射器中可以配置参数、各类SQL语句、存储过程、缓存、级联等内容，并通过简易的映射规则映射到指定的POJO上，有效消除JDBC底层代码。

## 1. 概述

有以下元素：

* select：查询，可以自定义参数、返回结果集等
* insert：插入，返回一个整数，代表插入的条数
* update：更新，返回整数，代表更新的条数
* delete：删除，返回整数，代表删除的条数
* parameterMap：定义参数映射关系，但即将被删除，不推荐使用
* sql：定义一部分SQL，其他地方引用
* resultMap：描述从结果集中加载对象，提供映射规则
* cache：给定命名空间的缓存配置
* chche-ref：其他命名空间缓存配置的引用

&nbsp;

## 2. select 查询

### 2.1 属性

    1. id：和Mapper的命名空间namespace组合起来是唯一的，供MyBatis调用
   2. parameterType：类全名/别名
   3. parameterMap：即将废弃
   4. resultType：定义类全路径，允许自动匹配时结果集自动映射；或int double等；也可用别名，但不能和resultMap同时使用
   5. resultMap：映射集的引用，和resultType二选一使用，能提供自定义映射规则
   6. flushCache：调用SQL后是否清空之前查询本地缓存和二级缓存，默认false
   7. useCache：启动二级缓存的开关，是否要将此次结果缓存，默认true
   8. timeout：超时参数，s，超时异常
   9. fetchSize：获取记录的总条数设定
   10. statementType：使用JDBC的哪个Statement工作，有STATEMENT（Statement）、PREPARED（PreparedStatement）、CALLABLE（CallableStatement），默认PREPARED
   11. resultSetType：对JDBC的resultSet接口而言，包括FORWARD_ONLY（游标允许向前访问）、SCROLL_SENSITIVE（双向滚动，但不及时更新，若库中数据修改过，不在resultSet中反映）、SCROLL_INSENSITIVE（双向滚动，及时跟踪库更新）
   12. databaseId：参考databaseIDProvider
   13. resultOrdered：仅适用于嵌套结果select，true表示假设包含了嵌套结果集或是分组了，当返回一个主结果行时，就不能引用前面结果集了，确保在获取嵌套的结果集时不至于爆内存；默认false
   14. resultSets：适用于多个结果集的情况，列出执行SQL后每个结果集的名称，逗号分隔

用的最多的是id、parameterType、resultType、resultMap；若要设置缓存还用到flushCache、useCache；其他不常用。

### 2.2 简单应用

eg：统计表中同一姓氏的用户数量：接口中声明方法：`public Integer countUserByFirstName(String firstName);`然后mapper.xml中：

```xml
<select id="countUserByFirstName" parameterType="string" resultType="int">
    select count(*) total from t_user 
    where user_name like concat(#{firstName}, '%')
</select>
```

​	id：配合主元素mapper的namespace属性，联合成为唯一标识

​	parameterType：接受的参数类型，可以是别名或全名

​	resultType：返回类型，同样可以全名或别名

​	#{firstName}：传入的参数

### 2.3 自动映射

      1. MyBatis提供自动映射功能，默认开启，能有效减少大量的映射配置

      2. 基础配置文件中的setting元素中有两个可配置选项autoMappingBehavior或mapUnderscoreToCamelCase，用于控制自动映射和驼峰映射的开关，前者使用的多，灵活，后者严苛，使用不广
      3. autoMappingBehavior的取值：
	1. NONE：不进行自动映射
	  2. PARTIAL：默认，只对没有嵌套结果集进行自动映射
	  3. FULL：对所有结果集进行自动映射
      4. 编写POJO中的属性，如果和表列名一致，就能自动映射；不一致也可在SQL中通过`as`改为一致
      5. 驼峰映射：表列名 user_name，POJO属性名userName，此时可以映射

### 2.4 传入多个参数

1. 使用map接口传递：此时将接口的方法定义为`public List<Book> findBooksByMap(Map<String, Object> parameterMap);`，此时传入映射器的就是一个map对象，使用这个map在SQL中设置参数：

2. ```xml
	<select id="findBooksByMap" parameterType="map" resultType="book">
	    select id, book_name as bookName from books where book_name like concat('%', #{bookName}, '%') and id like concat('%', #{id}, '%')
	</select>
	```

	SQL中#{bookName}是键，要在调用方法前填充map传入。但因map可读性差，不能限定传入参数类型，故使用不多

3. 使用注解传递：`public List<Book> findBooksByAnnotation(@Param("roleName") String roleName, @Param("id") String id);`，可读性大大提高，在SQL中使用参数时，花括号内填入注解的内容即可，且此时不用指定parameterType属性，MyBatis可以自动探索

4. 使用JavaBean传递：参数过多时注解繁琐，可用一个POJO代表参数

	```java
	public class BookParams {
	    private String bookName;
	    private String id;
	    // getter setter
	}
	```

	此时接口方法定义：`public List<Book> findBooksByBean(BookParams bookParam);`；映射器中parameterType指定该JavaBean全名，花括号内用bean的属性即可

### 2.5 resultMap映射结果集

自动映射规则简单，无法定义多的属性。复杂映射使用resultMap，首先需要定义resultMap元素，然后可在select中使用。

```xml
<mapper>
    <resultMap id="roleMap" type="role">
        <id property="id" column="id"/>
        <result property="roleName" column="role_name"/>
    </resultMap>
    <select id="getRole" parameterType="long" resultMap="roleMap">
    	sql
    </select>
</mapper>
```

resultMap元素定义了一个roleMap，id是其标识，type标识要映射的类。

子元素id是主键，result代表其他属性；property代表POJO的属性名，column是SQL的列名，如此便建立了映射联系。

### 2.6 分页参数RowBounds

1. 对查询到的结果取部分。RowBounds类内有offset和limit两个属性。
	1. offset表示偏移量，即从第几行开始读取，默认0。
	2. limit表示限制条数，默认Integer.MAX。
2. 要使用RowBounds，只需在接口参数最后增加一个`RowBounds rowBounds`即可，在映射文件中无需特别设置，MyBatis自动识别应用。
3. 小数据量适用，大量时用插件。

&nbsp;

## 3. insert 插入

### 3.1 属性

* id：SQL标号，标识
* parameterType：参数类型
* flushCache：是否刷新缓存，默认true表示插入后刷新一二级缓存
* timeout：s
* StatementType：STATEMENT（Statement）, PREPARED（PreparedStatement, CALLABLE（CALLABLEStatement），默认PREPARED
* useGeneratedKeys：是否启用Jdbc的getGeneratedKeys方法来取出数据库自动生成的主键（如MySQL的自增主键），默认false
* keyProperty：（仅insert和update）唯一标记一个恶属性，通过useGeneratedKeys或insert的selectKey子元素设置它的键值
* keyColumn：（仅insert和update）
* databaseId

insert后，返回一个整数表示影响了个条数。

### 3.2 简单应用

```xml
<insert id="insertRole" parameterType="role">
    insert into t_role (role_name, note) values(#{roleName},#{note})
</insert>
```

### 3.3 主键回填

1. 意义：MySQL中对主键有自增功能，故上述例子中没有插入id主键。但有时需要利用插入的元组的主键，就需要获取到库自增的主键的值。
2. JDBC中Statement执行插入后，可以通过getGeneratedKeys方法获得生成的主键。MyBatis中的insert有开关属性useGeneratedKeys，默认false，打开后需要配置属性keyProperty或keyColumn，用于告诉系统将回填的主键填入哪个属性

```xml
<insert id="" parameterType="" useGeneratedKeys="true" keyProperty="id">
	sql
</insert>
```

如此，就将主键id回填到POJO的id属性内。

### 3.4 自定义主键

1. 意义；有时主键不需自增，而是依赖某些规则，如本例的表空时id为1，不空时id为当前id+3
2. MyBatis中此功能依赖于selectKey元素
3. selectKey有order属性，其值BEFORE/AFTER表示此主键定义是在SQL前或后执行

```xml
<insert id parameterType>
    <selectKey keyProperty="id" resultType="long" order="BEFORE">
        select if (max(id)=null, 1, max(id)+3) from t_role
    </selectKey>
    sql
</insert>
```

&nbsp;

## 4. update和delete

属性类似上面，且同样返回整数表示影响的行数。

&nbsp;

## 5. sql元素

定义一条SQL的一部分，随后可以引用，方便后续编写SQL。

如表中列多，可将所有列名设为一个sql元素，随后SQL语句中便可通过include引用。

```xml
<mapper>
    <sql id="roleCols">
        id, role_name, note
    </sql>
    <select id parameterType>
        select <include refid="roleCols"/> from t_role where id = #{id}
    </select>
</mapper>
```

甚至支持变量传递，将SQL中的变量传给sql元素，然后sql元素引入SQL执行：

```xml
<sql id="roleCols">
    ${alias}.id, ${alias}.role_name
</sql>
<select id parameterType>
    select
    <include refid="roleCols">
        <property name="alias" value="t_role"/>
    </include>
</select>
```

本例中，include元素将表名t_role赋值给alias变量。

&nbsp;

## 6. 参数

《Java EE互联网轻量级框架整合开发》 MyBatis 5.6，暂不展开。

&nbsp;

## 7. resultMap元素

定义映射规则、级联的更新、定制类型转化器等。主要是一个结果集的映射关系，将SQL映射到JavaBean。MyBatis暂只支持resultMap查询，不支持更新或保存、级联的更新删除修改。

### 7.1 resultMap元素构成

```xml
<resultMap>
    <constructor>
        <idArg/>
        <arg/>
    </constructor>
    <id/>
    <result/>
    <association/>
    <collection/>
    <discriminator>
        <case/>
    </discriminator>
</resultMap>
```

id表示主键。

result配置POJO到SQL列名的映射关系。

Constructor用于配置构造方法。POJO可能没有无参构造方法，就需要把有参构造方法配置进此，如`public RoleBean(Integer id, String roleName)`：

```xml
<constructor>
    <idArg column="id" javaType="int"/>
    <arg column="role_name" javaType="string"/>
</constructor>
```

result和idArg的属性：

* property：映射到列结果的字段或属性
* column：对应SQL的列
* javaType
* jdbcType
* typeHandler

### 7.2 使用map存储结果集

map支持任何select的结果集存储，只需在select元素中配置属性`resultType="map"`即可。但可读性差，一般使用POJO存储。

### 7.3 使用POJO存储结果集

POJO存储，就可以使用自动映射，或自定义更复杂的映射，当然这就要先创建一个resultMap元素（映射），见[2.5 resultMap映射结果集](###2.5 resultMap映射结果集)

&nbsp;

## 8. 级联

### 8.1 MyBatis中的级联

分为三种：

1. 鉴别器discriminator：根据某些条件决定采用具体实现类级联的方案
2. 一对一association
3. 一对多collection

本例以学生Student、任务表Task、任务安排表ST、男女体检表Female/MaleForm、学生证表ID说明，学生与任务表一对多、与男女体检表属于鉴别器类型、与学生证一对一。

&nbsp;

### 8.2 建立POJO

为每个实体建立POJO。

男女体检表，多数项目相同，可建立父类后继承：

```java
public abstract class HealthForm {
    private Long id;
    private Long stuId;
    private String heart;
    // setter getter
}

public class FemaleForm extends HealthForm {
    private String uterus;
    // settter getter
}
public class MaleForm extends HealthForm {
    private String prostate;
    // setter getter
}
```

学生证表：

```java
public class IDCard {
    private Long id;
    private Long stuId;
    private String name;
}
```

任务表：

```java
public class Task {
    private Long id;
    private String title;
}
```

任务安排表：

```java
public class ST {
    private Long id;
    private Long stuId;
    private Task task = null;
}
```

学生表：建立父类，并继承男女学生，以适配男女体检单

```java
public class Student {
    private Long id;
    private String name;
    private SexEnum sex = null;
    // 学生证一对一
    private IDCard idCard;
    // 任务，一对多
    private List<Task> stuTasks = null;
    // settter getter
}

public class MaleStudent extends Student {
    private MaleForm maleForm = null;
    // setter getter
}

public class FemaleStudent extends Student {
    private FemaleForm femaleForm = null;
    // setter getter
}
```

综上可见，MyBatis中级联，在POJO中的属性就应体现出级联。

&nbsp;

### 8.3 配置mapper文件

级联核心，对每个实体类创建接口后配置mapper文件。

Task

```xml
<mapper namespace="xxx.TaskMapper">
    <select id="getTask" parameterType="long" resultType="xxx.Task">
        select id, title from t_task where id = #{id}
    </select>            
</mapper>
```

IDCard

```xml
<mapper namespace="xxx.IDCardMapper">
    <select id="getCardByStuId" parameterType="long" resultType="xxx.IDCard">
        select id, stuId, name from t_card where stuId = #{stuId}
    </select>            
</mapper>
```

**ST 任务安排表**：association元素代表一对一级联的开始。

```xml
<mapper namespace="xxx.STMapper">
    <resultMap type="ST" id="STMap">
        <id column="id" property="id"/>
        <result column="stu_id" property="stuId"/>
        <association property="task" column="task_id" select="xxx.TaskMapper.getTask"/>
    </resultMap>
    <select id="getSTByStuId" parameterType="long" resultMap="STMap">
        select id, stuId, name from t_card where stuId = #{stuId}
    </select>            
</mapper>
```

男女体检表：

```xml
<mapper namespace="xxx.MaleFormMapper">
    <select id="getMaleForm" parameterType="long" resultType="xxx.MaleForm">
        select id, stuId, name from t_card where stuId = #{stuId}
    </select>            
</mapper>
```

**学生映射：**本例核心，与其他实体类关联，resultMap元素也可继承

```xml
<mapper namespace="xxx.StudentMapper">
    <resultMap type="xxx.Student" id="student">
        <id column="id" property="id"/>
        <reslut column="name" property="name"/>
        <association property="idCard" column="id" select="xxx.IDCardMapper.getCardByStuId"/>
        <collection property="studentTasks" column="id" select="xxx.STMapper.getSTByStuId"/>
        <discriminator javaType="long" column="sex">
            <case value="1" resultMap="maleFormMapper"/>
            <case value="2" resultMap="famaleFormMapper"/>
        </discriminator>
    </resultMap>
    
    <resultMap type="FemaleStudent" id="femaleFormMapper" extends="student">
        <association property="femaleForm" column="id" select="xxx.FemalFormMapper.getFemaleForm"/>
    </resultMap>
    
    <resultMap type="MaleStudent" id="maleFormMapper" extends="student">
        <association property="maleForm" column="id" select="xxx.MalFormMapper.getMaleForm"/>
    </resultMap>
    
    <select id="getSTByStuId" parameterType="long" resultMap="STMap">
        select id, stuId, name from t_card where stuId = #{stuId}
    </select>            
</mapper>
```

discriminator：column属性代表用此列进行鉴别，resultMap属性表示采用哪个resultMap映射

&nbsp;

### 8.4 N+1问题

以上级联方式，在使用时会引发所有级联共同执行，如只需要部分数据，多余的SQL会浪费资源。如已有N个关联关系完成了级联，只要再加入一个关联关系，就变成了N+1个关联，所有级联SQL都会被执行，但并非所有数据都需要。

为解决此问题，MyBatis提出延迟加载功能，一开始取学生信息时，不将学生证表、体检表、任务表取出，只取学生表和任务安排表信息。当通过学生POJO访问学生证、体检表、任务表时才通过相应SQL取出。

&nbsp;

### 8.5 延迟加载

希望一次性把常用的级联数据取出，而不常用的级联数据等到用时才取出。对于这些不常用的级联表采用延迟加载。

在settings配置中有以下两子元素配置级联：

* lazyLoadingEnabled：全局开关，默认false，开启后所有关联对象都延迟加载；特定关联关系中可设置fetchType覆盖该全局开关
* aggressiveLazyLoading：启用时，对任意延迟属性的调用会使带有延迟加载属性的对象完整加载；反之，每种属性按需加载；默认值从3.4.2后变为false；

前者决定是否开启延迟加载，后者控制是否采用层级加载，都是全局级别配置。

另有fetchType属性，可以针对某个信息设置，本属性在association和collection中可以设置，有两个值：

* eager：获得当前POJO后立即加载对应数据
* lazy：获得当前POJO后延迟加载对应数据

当开启延迟加载，关闭层级加载后，使用此属性即可自定义设置。

&nbsp;

### 8.6 另一种级联

基于SQL表连接的基础上，再次设计。虽能消除N+1问题，但较为复杂，浪费内存，不甚推荐，暂不展开。

&nbsp;

### 8.7 多对多级联

程序中将此拆分成两个一对多级联来处理。

本例中，用户可以是多个角色，一个角色可以有多个用户。

POJO：

```java
public class Role {
    private Long id;
    private String name;
    // 关联用户一对多
    private List<User> users;
    // setter getter
}

public class User {
    private Long id;
    private String name;
    // 关联角色一对多
    private List<Role> roles;
    // setter getter
}
```

Mapper：

```xml
<mapper namespace="xxx.RoleMapper">
    <resultMap type="Role" id="RoleMapper">
        <id column="id" property="id"/>
        <result column="role_name" property="name"/>
        <association property="users" column="id" fetchType="lazy" select="xxx.UserMapper.findUserByRoleId"/>
    </resultMap>
    <select id="findRoleByUserId" parameterType="long" resultMap="RoleMapper">
        select r.id, r.name from t_role r, t_userrole ur where r.id = ur.role_id and r.id=#{userId}
    </select>            
</mapper>

<mapper namespace="xxx.UserMapper">
    <resultMap type="User" id="UserMapper">
        <id column="id" property="id"/>
        <result column="user_name" property="name"/>
        <association property="roles" column="id" fetchType="lazy" select="xxx.RoleMapper.findRoleByUserId"/>
    </resultMap>
    <select id="findUserByRoleId" parameterType="long" resultMap="UserMapper">
        select
    </select>            
</mapper>
```

&nbsp;

## 9. 缓存

MyBatis支持缓存，一般放在内存中，提高性能。数据库位于磁盘，速度慢。一般只将命中率高的数据放在缓存。

### 9.1 一级二级缓存

MyBatis分为一级缓存和二级缓存。

一级缓存是在SqlSession上的缓存，二级缓存是在SqlSessionFactory上的缓存。默认时MyBatis开启一级缓存，且一级缓存不需要POJO可序列化。

一级缓存：用一个SqlSession获取mapper，用这个mapper先后get同一数据，实际上是用一个SqlSession获取同一数据；此时只执行一次SQL语句；当一个SqlSession第一次通过SQL和参数获取对象后，会将其缓存，若下次SQL和参数都没有变化，并且缓存未超时或声明不需刷新时，就可以从缓存中读取数据。

二级缓存：一级缓存位于SqlSession层面，不同SqlSession缓存不能共享；如需共享就要开启二级缓存，位于Factory层，更高一层；只需在映射文件中加入`<cache/>`即可。查询结束后需要SqlSession.commit()提交，才会缓存对象到Factory层面。此时需要可序列化。

&nbsp;

### 9.2 缓存配置与使用

1. 只配置`<cache/>`后，MyBatis会将select的结果缓存，而增删改之后会刷新缓存。

2. `<cache/>`配置属性：

* blocking：默认false，不开启阻塞性缓存，读写时不加入JNI的锁；开启后能保证读写安全性，但降低性能
* readOnly：缓存内容是否只读，默认false；若开启，不会因为多线程读写造成不一致
* eviction：缓存策略，LRU最近最少使用的，即移除最长时间不用的对象；FIFO先入先出；SOFT软引用，移除基于垃圾回收器状态和软引用规则的对象；WEAK弱引用，移除垃圾收集器状态和弱引用规则的对象；默认LRU
* flushInterval：ms，默认null，即不主动刷新，只有增删改后才刷新
* type：自定义缓存类，要实现接口Cache
* size：缓存对象个数，默认1024

3. 如需自定义缓存类就要实现Cache接口，但一般可使用Redis、MongoDB等常用缓存。如有一个Redis实现类RedisCache，则如下配置：

	```xml
	<cache type="xxx.RedisCache">
	    <property name="host" value="localhost"/>
	</cache>
	```

4. 可对SQL单独配置以自定义是否缓存

	```xml
	<select ... flushCache="false" useCache="true"/>
	<insert ... flushCache="true"/>
	```

	此为默认配置，flushCache对四种SQL都有效；useCache仅select特有。

	如其他mapper文件需要使用本mapper文件的cache配置，可以引用：

	`<cache-ref namespace="xxx.Mapper"/>`

&nbsp;

## 10. 存储过程

数据库概念，是库预先编译好的，放在库内存中的程序片段，具有高性能，可重用的特点。定义了3种类型参数：输入、输出、输入输出。

暂不展开。

### 10.1 IN和OUT参数

### 10.2 游标