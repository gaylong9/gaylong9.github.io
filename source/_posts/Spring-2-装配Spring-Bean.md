---
title: Spring_2.装配Spring_Bean
mathjax: false
date: 2021-03-23 11:20:52
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

[toc]



# 1. XML中依赖注入的3种方式

虽有依赖查找，通过资源定位查找资源，但Spring主要使用依赖注入。

注入的3种方式：

1. 构造器注入
2. setter注入
3. 接口注入

前两种是主要方式，后一种是从别的地方注入，如数据源往往是通过服务器配置，用JNDI的形式通过接口注入Spring IoC中。

## 1.1 构造器注入

本方法基于构造方法实现，构造方法可以有参或无参。Spring通过反射的方式，用构造方法完成注入。

eg：Role

```java
public class Role {
    private Long id;
    private String roleName;
    private String note;
    // getter setter
    public Role(String name, String note) {
        this.roleName = name;
        this.note = note;
    }
}
```

此时仅有一个有参构造方法，故XML配置时要：

```xml
<bean id="role1" class="xxx.Role">
    <constructor-arg index="0" value="总经理"/>
    <constructor-arg index="1" value="公司管理者"/>
</bean>
```

适用于参数较少的情况。

&nbsp;

## 1.2 使用setter注入

当参数过多时，以上xml配置就会复杂。

setter注入是最主流的注入方式，利用Jav Bean规范所定义的setter方法完成注入，灵活可读性高。原理也是反射。

使用本方法时只需一个无参构造方法即可，写好setter方法，随后在XML中：

```xml
<bean id="role" class="xxx.Role">
    <property name="roleName" value="总经理"/>
    <property name="note" value="公司管理者"/>
</bean>
```

如此，Spring通过反射调用无参构造器生成对象，并反射setter方法注入配置值。

&nbsp;

## 1.3 接口注入

有些资源来自系统外，如数据库连接资源在Tomcat配置后，通过JNDI的形式获取（数据源）。

如数据源，配置数据源需要在context.xml中：

```xml
<Context>
    <Resource name="JNDI名称"
              auth="Container"
              type="javax.sql.DataSource"
              driverClassName="com.mysql.jdbc.Driver"
              url="jdbc:mysql://localhost:3306/ssm?..."
              username="jsy"
              password="123"/>
</Context>
```

Spring用JNDI获取Tomcat的数据库连接池：

```xml
<bean class="org.springframework.jndi.JndiObjectFactoryBean">
    <property name="jndiName">
        <value>java:comp/env/jdbc/ssm</value>
    </property>
</bean>
```

&nbsp;

# 2. 装配Bean概述

以上了解了Spring IoC与依赖注入，本节了解完整装配流程（把Bean发布到Spring IoC容器）。

通常使用ApplicationContext的具体实现类作为容器。

Spring提供3种装配方法：

1. XML中显式配置
2. Java的接口和类中实现配置
3. 隐式Bean的发现机制和自动装配原则

具体开发中每种方式都会用到，有以下几点规则：

1. 基于约定优于配置的原则，最优先的应是隐式Bean的发现机制和自动装配原则，能减少开发者的决定权
2. 不能自动装配时优先考虑接口和类中实现配置，避免XML配置泛滥
3. 以上都不可行时，采用XML配置

如所配置的类是自己开发的，能够接触到代码层，就选择前两种；若是第三方类库，无法修改源代码，就XML。

&nbsp;

# 3. XML显式装配

XML文件中引入XML模式（XSD）文件，其中定义配置Bean的一些元素

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```



## 3.1 装配简易值

一个简单的setter注入装配方式

```xml
<bean id="role" class="xxx.Role">
    <property name="roleName" value="总经理"/>
    <property name="note" value="公司管理者"/>
</bean>
```

id：Spring寻找Bean的编号，不过不必须，不指定时Spring自动生成id=“全限定名#{idx}”

class：类全限定名

property：类属性定义，name是属性名

若属性是一个自定义类，就要使用`<property name="" ref=""/>`；ref表示该自定义类之前定义过的bean元素id

&nbsp;

## 3.2 装配集合

如属性是Collection，Properties等，此处以泛型为String举例

```xml
<bean id class>
    <property name="list">
        <list>
            <value>value-list-1</value>
            <value>value-list-2</value>
        </list>
    </property>
    <property name="map">
        <map>
            <entry key="key1" value="value-key-1"/>
            <entry key="key2" value="value-key-2"/>
        </map>
    </property>
    <property name="props">
        <props>
            <prop key="prop1">value-prop-1</prop>
            <prop key="prop2">value-prop-2</prop>
        </props>
    </property>
    <property name="set">
        <set>
            <value>value-set-1</value>
            <value>value-set-2</value>
        </set>
    </property>
    <property name="array">
        <array>
            <value>value-array-1</value>
            <value>value-array-2</value>
        </array>
    </property>
</bean>
```

若集合中元素是类对象时，如下例（省略了Role和User类的定义）：

```java
public class UserRoleAssembly {
    private Long id;
    private List<Role> list;
    private Set<Role> set;
    private Map<Role, User> map;
    // setter getter
}
```

由于该类中有Role和User的对象，故要先配置Role和User对象，再配置UserRoleAssembly

```xml
<bean id="role1" class>
    <property name="id" value="1"/>
    <property name="roleName" value="role1"/>
</bean>

<bean id="role2" class>
    <property name="id" value="2"/>
    <property name="roleName" value="role2"/>
</bean>

<bean id="user1" class>
    <property name="id" value="1"/>
    <property name="userName" value="user1"/>
</bean>

<bean id="user2" class>
    <property name="id" value="2"/>
    <property name="userName" value="user2"/>
</bean>

<bean id="userRoleAssembly" class>
    <property name="id" value="1"/>
    <property name="list">
        <list>
            <ref bean="role1"/>
            <ref bean="role2"/>
        </list>
    </property>
    <property name="map">
        <map>
            <entry key-ref="role1" value-ref="user1"/>
            <entry key-ref="role2" value-ref="user2"/>
        </map>
    </property>
    <property name="set">
        <set>
            <ref bean="role1"/>
            <ref bean="role2"/>
        </set>
    </property>
</bean>
```

&nbsp;

## 3.3 命名空间装配

xml文件中要引入命名空间和xml模式（xsd）文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="role1" class c:_0="1" c:_1="rolename1" c:_2="rolenote1"/>
    <bean id="role2" class p:id="2" p:roleName="rolename2" p:note="rolenote2"/>
</beans>
```

`xmlns:c="http://www.springframework.org/schema/c"`定义xml命名空间

`c:_0`表示构造方法的第一个参数

`p:id`表示引用属性

复杂属性的装配：继续引入命名空间和xsd，以上例中的UserRoleAssembly为例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/util
http://www.springframework.org/schema/spring-beans.xsd">
    <bean id="role1" class c:_0="1" c:_1="rolename1" c:_2="rolenote1"/>
    <bean id="role2" class p:id="2" p:roleName="rolename2" p:note="rolenote2"/>
    <bean id="user1" class p:id="1" p:userName="username1" p:note="note1"/>
    <bean id="user2" class p:id="2" p:userName="username2" p:note="note2"/>
    
    <util:list id="list">
        <ref bean="role1"/>
        <ref bean="role2"/>
    </util:list>
    
    <util:map id="map">
        <entry key-ref="role1" value-ref="user1"/>
        <entry key-ref="role2" value-ref="user2"/>
    </util:map>
    
    <util:set id="set">
        <ref bean="role1"/>
        <ref bean="role2"/>
    </util:set>
    
    <bean id="userRoleAssembly" class p:id="1" p:list-ref="list" p:map-ref="map" p:set-ref="set"/>
</beans>
```

&nbsp;

# 4. 注解装配

减少XML配置，注解既能实现XML的功能，也提供了自动装配的功能。开发者所需做的决定就少了，有利于开发，这就是“约定优于配置”的开发原则。主流是以注解配置为主，XML配置为辅。

对于XML配置，获取Bean是`new ClassPathXmlApplicationContext("xxx.xml").getBean("id")`；对于注解配置，不使用XML，要获取Bean就要建立一个ApplicationConfig类，该类用@Configuration和@ComponentScan注解，内类内提供了一个或多个被@Bean注解的方法，`ctx = new AnnotationConfigApplicationContext(ApplicationConfig.class);`。

Spring中提供了两种方式让IoC容器发现Bean：

1. 组件扫描：通过定义资源的方式，让IoC容器扫描对应包，把Bean装配进来
2. 自动装配：通过注解定义，使一些依赖可以通过注解完成

## 4.1 @Component 装配

POJO:

```java
@Component(value="role")
public class Role{
    @Value("1")
    private Long id;
    @Value("roleName1")
    private String roleName;
    @Value("note1")
    private String note;
    // setter getter
}
```

`@Component`代表IoC把这个类扫描生成Bean实例，value属性代表这个类的id，可简写成`@Component("role")`，若直接空括号，则自动以首字母小写的类名作为id。

`@Value`代表值的注入

还需一个Config类告诉IoC容器去哪里扫描对象：

```java
@ComponentScan
public class PojoConfig {}
```

本Config类无需逻辑，@ComponentScan默认扫描当前包的路径，故这个Config类要和上面的Role放在同一包内。

此时就可以生成IoC容器，注册POJO了：

```java
public class AnnotationMain{
    ApplicationContet ctx = new AnnotationConfigApplicationContext(PojoConfig.class);
    Role role = ctx.getBean(Role.class);
    ...
}
```

可以看出两个弊端：ComponentScan只能扫描当前包，不灵活；注解注入暂时只能简单值。后者之后讲到，现在先看前一个问题。

ComponentScan有两个配置项；

1. basePackages：接收一个Java包的数组，扫描对应的包和子包，装配Bean
2. basePackageClasses：扫描多个类

`@ComponentScan(basePackages={"xxx", "xxx"}, basePackageClasses={xxx.class, xxx.class})`

@ComponentScan注解，虽然可以有多个，但每一个注解都会使IoC容器生成一份实例，若多个注解之间存在POJO重复，就会产生重复对象，故**应只使用一个@ComponentScan注解**，一个注解内如有重复类，也不会多次生成。

两个配置项的选择：basePackages易读，但需要大量重构的项目中造成修改困难。

&nbsp;

## 4.2 @Autowired 自动装配

多数时候**推荐使用自动装配**，可以减小配置的复杂度。

如需自动装配一些String、Integer等简单变量，需要配置XML。

IoC容器先完成Bean的定义和生成，然后寻找资源注入。使用此注解，是在容器生成所有Bean后，若遇到此注解，就寻找对应Bean，注入。即，自动装配，就是容器自己发现对应Bean，自动装配的方式。

以下例子：先定义一个接口，随后在实现类中实现自动装配

```java
public interface RoleService {
    public void printRoleInfo();
}

public class RoleServiceImpl implements RoleService {
    @Autowired
    private Role role = null;
    // getter setter
    
    @Override
    public void printRoleInfo() {
        ...
    }
}
```

RoleServiceImpl中的`@Autowired`表示IoC定位所有Bean后，这个字段需要按类型注入，如此容器就会寻找资源注入。例如代码中定义了Role类的话，容器就会先实例化Role，随后根据Autowired注解找到Role的实例来注入。

`@Autowired(required=false)`表示不一定必须找到注入，找不到也无需报错；如不配置，默认情况下找不到注入就会抛出异常。

另可用于setter配置：

```java
public class RoleServiceImpl implements RoleService {
    private Role role = null;
    @Autowired
    public void setRole(Role role) {
        this.role = role;
    }
    
    @Override
    public void printRoleInfo() {
        ...
    }
}
```

&nbsp;

## 4.3 自动装配的歧义性

如A类有一个属性是B接口，但有两个B接口的实现类B1和B2，此时按类型的自动注入就会抛出异常。有两个注解用于消除歧义：

1. @Primary

	是对于类的注解，表示如有多个满足的类，使用此类优先注入。但只能解决首要性问题，没有解决选择性问题。

2. @Qualifier

	IoC最底层的BeanFactory也定义了按名称查找的方法。使用方法为用Component注解给B1和B2起一个别名，在A中指明要用那个别名：

	```java
	public class A {
	    @Autowired
		@Qualifier("b1")
	    private B b = null;
	    // setter getter
	}
	
	@Component("b1")
	public class B1 {}
	
	@Component("b2")
	public class B2 {}
	```

	&nbsp;

## 4.4 装载带有参数的构造方法类

含参构造方法的注入。继续用Autowired和Qualifier

```java
public RoleController(@Autowired Role role) {
    this.role = role;
}
```

&nbsp;

## 4.5 @Bean 装配Bean

以上都是用Component注解Bean，用Autowired、Qualifier注解属性、方法、参数。对于第三方包，无法自己加入注解，就需要@Bean注解。

`@Bean`：注解到方法上，方法返回的对象将成为Spring的Bean，存放在IoC容器中。

这里所说的方法，可以是自己写的方法，只是返回值是一个第三方对象。

`@Bean(name="..")`还可如此配置为Bean规定别名/id，便可配合Autowired和Qualifier。

&nbsp;

## 4.6 注解自定义Bean的初始化和销毁

使用Bean注解时，可以通过配置指定Bean初始化和销毁时执行的方法

@Bean的配置项：

* name：字符串数组，允许配置多个BeanName
* autowire：是否是一个引用的Bean对象，默认Autowired.NO
* initMethod：自定义初始化方法
* destroyMethod：自定义销毁方法

eg：

```java
public class Role {
    public init() {}
    public destory() {}
}

public class RoleMaker {
    private Role role;
    @Bean(name="role", initMethod="init", destroyMethod="destroy")
    public Role getRole() {return role;}
}
```

&nbsp;

# 5. 装配的混合使用

以上介绍了XML和注解两大类装配Bean的方法。

1. 区别：对于自定义的Bean，主要是用注解；第三方类，则使用XML。如@Bean这个注解，用于第三方类时，不如使用XML简洁易懂。

2. 用法：对于XML配置，获取Bean是`new ClassPathXmlApplicationContext("xxx.xml").getBean("id")`；对于注解配置，不使用XML，要获取Bean就要建立一个ApplicationConfig类，该类用@Configuration和@ComponentScan注解，内类内提供了一个或多个被@Bean注解的get方法，`ctx = new AnnotationConfigApplicationContext(ApplicationConfig.class);`。

3. 混合使用：如是配合ApplicationConfig类使用，最终的IoC容器是通过AnnotationConfigApplicationContext(ApplicationConfig.class)获取的话，在XML中配置的Bean还需要通过注解引入代码中 `@ImportResource({"classpath:spring-dataSource.xml","classpath:xxx.xml"})`，如此才能将XML中的Bean注册进IoC容器，随后便可利用@Autowired进行数据源的自动装配

4. ApplicationConfig组合：如全部配置都注解在一个ApplicationConfig，较为复杂，可创建多个ApplicationConfig123，并`@Import({ApplicationConfig1.class, ApplicationConfig2.class})`引入总ApplicationConfig

5. XML组合：XML同样支持引入其他XML文件 `<import resource="xxx.xml"/>`

6. XML扫描包：`<context:component-scan base-package="xxx"/>`

	

一个数据源例子：

如用@Bean配置数据源，则用@Bean注解一个getDataSource方法，此方法内实例化一个Properties对象，对此对象设置各类数据源属性，最后通过BasicDataSourceFactory.createDataSource(props)创建数据源对象并返回。

如用XML配置数据源：对第三方包的依赖更低，页面去try-catch-finally。

```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource?">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/"/>
    <property name="username" value=""/>
    <property name="password" value=""/>
</bean>
```

随后在ApplicationConfig中@ImportResource，便可在使用SQL的类中@Autowired DataSource。

&nbsp;

# 6. Profile

定义Bean的profile，支持不同数据环境。如开发与测试在不同数据环境下进行。暂不展开。

&nbsp;

# 7. 加载属性文件 properties

## 7.1 用注解加载

`@PropertySource`

1. 属性：

	1. name：字符串，配置此属性配置的名称
	2. value：字符串数组，可以配置多个properties文件
	3. ignoreResourceNotFount：boolean，若找不到对应文件是否忽略，默认false，表示会抛出异常
	4. encoding：编码，默认“”

2. 使用：在ApplicationConfig中`@PropertySource(value={"classpath:xxx.properties"}, ignoreResourceNotFound=true)`，使用时`ctx.getEnvironment().getProperty("xxx");`

3. 属性占位符：`${xxx.xxx} ` 直接得到properties文件的值；上述的通过environment获取无法解决此问题；Spring中推荐使用PropertySourcesPlaceholderConfigurer这个属性文件解析类处理。在ApplicationConfig中：

	```java
	@Configuration
	@PropertySource(...)
	@ComponentScan(...)
	public class ApplicationConfig {
	    @Bean
	    public PropertySourcesPlaceholderConfigurer getPropertySourcesPlaceholderConfigurer() {
	        return new PropertySourcesPlaceholderConfigurer();
	    }
	}
	```

	在需要使用占位符的地方使用Component的`@Value("${xxx.xxx}")`即可实现属性文件用于注入。

&nbsp;

## 7.2 用XML加载

如加载database-config.properties

```xml
<beans ...>
    <context:component-scan base-package="xxx"/>
    <context:property-palceholder ignore-resource-not-found="true" location="classpath:database-config.properties"/>
</beans>
```

ignore-resource-not-found属性表示允许properties文件不存在，不会抛出异常。

location中是配置属性文件路径，可以有多个文件，用逗号分隔。当文件过多时，用以下写法；

```xml
<beans>
    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <array>
                <calue>classpath:database-config.properties</calue>
                <calue>classpath:log4j.properties</calue>
            </array>
        </property>
        <property name="ignoreResourceNotFound" value="true"/>
    </bean>
</beans>
```

&nbsp;

# 8. 条件化装配Bean

`@Conditional`，配置类时，类需要实现Condition（org.springframework.context.annotation.Condition）接口。

如在配置数据源时，需要先判断数据源中各项配置是否完整，不完整就不创建数据源。

可在getDataSource方法上注解`@Conditional({DataSourceCondition.class})`，判断工作由实现了Condition接口的DataSourceCondition类完成。

```java
public class DataSourceCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 获取上下文环境，判断是否存在各项properties配置
        Environment env = context.getEnvironment();
        return env.containsProperty("xxx.xxx") && env.containsProperty("xxx.xxx");
    }
}
```

AnnotatedTypeMetadata用于获得Bean的注解信息。

若返回false，getDataSource中由于配置过`@Conditional`，故不会配置数据源。

&nbsp;

# 9. Bean的作用域

通常情况下IoC容器只会Bean生成一个实例，多次get也是同一个对象。

本类功能由Spring提供的作用域决定：

* 单例singleton：默认，整个应用中Spring只生成一个Bean的实例
* 原型prototype：每次注入，或通过IoC容器获取Bean时，都创建新实例
* 会话session：Web应用中使用，会话过程中只创建一个实例
* 请求request：Web应用中使用，一次请求中只创建一个实例

使用：在类上注解`@Scope(ConfigurableBeanFactory.SCOPE_XXX)`

&nbsp;

# 10. Spring EL

Spring表达式，更灵活的注入方式。

拥有诸多功能：

* 使用Bean的id来引用Bean
* 调用指定对象的方法和访问对象属性
* 运算
* 正则表达式匹配
* 集合配置

## 10.1 相关类

1. ExpressionParser接口，有InternalSpelExpressionParser和SpelExpressioniParser实现类。

	简单用法如下：

	```java
	// 表达式解析器
	ExpressionParser parser = new SpelExpressionParser();
	// 设置表达式
	Expression exp = parser.parseExpression("'hello world'");
	String str = (String) exp.getValue();
	// 通过EL访问普通方法
	exp = parser.parseExpression("'hello world'.charAt(0)");
	char ch = (Character) exp.getValue();
	// 通过EL访问getter
	exp = parser.parseExpression("'hello world'.bytes");
	byte[] bytes = (byte[]) exp.getValue();
	// 通过EL访问属性
	exp = parser.parseExpression("'hello world'.bytes.length");
	int length = (Integer) exp.getValue();
	// 创建对象
	exp = parser.parseExpression("new String('abc')");
	String abc = (String) exp.getValue();
	```

2. EvaluationContext接口，变量解析，有StandardEvaluationContext实现类

	```java
	// 创建角色对象并获取属性
	Role role = new Role(1, "name_jsy");
	expression = parser.parseExpression("name");
	String name = (String) expression.getValue(role);
	
	// 变量环境类，将role作为根结点
	EvaluationContext ctx = new StandardEvaluationContext(role);
	// 操作根结点
	parser.parseExpression("name").setValue(ctx, "name_jsyyy");
	// 获取新属性值
	name = parser.parseExpression("name").getValue(ctx, String.class);
	// 调用getter方法
	name = parser.parseExpression("getName()").getValue(ctx, String.class);
	
	// 给变量环境增加内容，并写 读
	var list = new ArrayList<String>();
	list.add("value1");
	list.add("value2");
	ctx.setVariable("list", list);
	parser.parseExpression("#list[1]").setValue(ctx, "update_value2");
	String str = parser.parseExpression("#list[1]").getValue(ctx, String.class);
	```

&nbsp;

## 10.2 Bean的属性和方法

使用EL注入，并且可以方便地用其他Bean的属性注入新Bean

```java
@Component("role")
public class Role {
    @Value("#{1}")
    private int id;
    
    @Value("#{'role_name_1'}")
    private String name;

    // setter getter
}

@Component("elBean")
public class ELBean {
    @Value("#{role}")
    private Role role;
    @value("#{role.id}")
    private int id;
    @Value("#{role.name}")
    private String name;
    // setter getter
}
```

另外EL中使用getter时，如`@Value("#{role.getNote().toString()}")`，为防止null.toString()，可改写为`#{role.getNote()?.toString()}`，问号表示判断非null。

&nbsp;

## 10.3 使用类的静态常量和方法

`@Value("#{Math).PI}")`，Math不用import，其他包外类需要写出全限定名。

`@Value(#{Math}.random()})`调用静态方法

&nbsp;

## 10.4 Spring EL运算

`@Value("#{role.id+1}")`

`@Value("#{role.name+"123"}")`

`@Value("#{role.name eq "jsy"}")`

`@Value("#{role.id == 1}")`

`@Value("#{role.id > 1}")`

`@Value("#{role.id > 1? 5: 1}")`

`@Value("#{role.note? : "note"}")`

&nbsp;







