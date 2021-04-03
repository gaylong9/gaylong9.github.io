---
title: Spring-1-Spring_IoC概念
mathjax: false
date: 2021-03-24 11:20:40
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

[toc]



# Spring IoC

## 1. Spring

因其先进的设计理念IoC控制反转和AOP面相切片编程，使它成为Java中最成功的框架。

EJB笨重，通过EJB容器发布Bean，应用通过EJB容器获得Bean，过程缓慢，步骤复杂。

Spring随后兴起，它认为一切Java类都是资源，资源都是Bean，容纳这些Bean的是Spring提供的IoC容器，故Spring是一种基于Bean的编程。对于POJO，提供轻量级和低侵入的编程，可以配置来扩展POJO功能；提供切面编程，消除了try-catch-finally的滥用；提供模板类，整合各个框架和技术。

&nbsp;

## 2. Spring IoC

控制反转。

eg：以前想喝橙汁需要自己买橙子榨汁，现在只需联系饮品店，就能喝到饮品店提供的橙汁。

1. 主动创造：需要创建Blender类、JuiceMaker类，创建对象；在实际任务中，要维护众多关系，并且一些还不熟悉，主动创造比较复杂
2. 被动创造：我们只要提供描述，如这杯橙汁要多少糖、大杯还是小杯等，就可由系统自动创建并返回给调用者

```java
// JuiceMaker
public class JuiceMaker {
    private String shop = null;
    private Source source = null;	// 描述
    // setter getter
    public String makeJuice() {
        String juice = shop + " makes " + source.getSize() + source.getSugar() + source.getFruit();
        return juice;
    }
}

// Source
public class Source {
    private String fruit;
    private String sugar;
    private String size;
    // getter setter
}
```

具体的描述，在XML中提供：这个配置就相当于一个描述实例

```xml
<bean id="source" class="xxx.Source">
    <property name="fruit" value="橙汁"/>
    <property name="sugar" value="多糖"/>
    <property name="size" value="中杯"/>
</bean>
```

JuiceMaker Bean也要配置：

```xml
<bean id="juickMaker" class="xxx.JuickMaker">
    <property name="shop" value="蜜雪冰城"/>
    <property name="source" ref="source"/>
</bean>
```

如此就配置完成，表示要蜜雪冰城为我们做一杯多糖中杯橙汁，最后在Java代码中“下单”：

```java
JuiceMaker maker = (JuiceMaker) ctx.getBean("juickMaker");
String juice = maker.makeJuice();
```

这个过程中，我们只提供了配置信息（订单信息），对于制作过程并不关心。



**Spring IoC阐述：**

1. 控制反转是一种通过描述（XML或注解）并通过第三方去产生或获取特定对象的方式
2. Spring中实现控制反转的是IoC容器，实现方法是依赖注入（Dependency Injection， DI）
3. 如实际开发中，负责A模块的开发者完成接口模块开发后，将服务发布到IoC容器，B模块开发者若要调用，只需提供描述就可得到对应接口。
4. 总之，类的具体实现不需理解，只要知道它有什么用就可以；对象的产生依靠IoC容器，而不是开发者主动的行为。

&nbsp;

## 3. Spring IoC容器

### 3.1 容器的设计

主要基于BeanFactory和ApplicationContext两个接口。其中后者是前者的子接口之一。大多数场景下都将ApplicationContext作为IoC容器。

最底层的接口BeanFactory的一些方法：

* getBean：从容器中获取Bean。多态，多种参数类型，string、Class；但Class类型可以扩展接口也可继承父类，所以有可能使用父类类型无法准确获得实例。
* isSingleton：是否单例，若真表示该Bean在容器中是作为唯一单例存在的；
* isPrototype：若真，表示当从容器中获取Bean，容器就生成一个新实例；默认Spring为Bean创建单例
* getAliases：获取别名

ApplicationContext对BeanFactory进行了扩展，实际使用较多；而ApplicationContext的实现类，虽然功能更多，但是是使用在某一个特定领域。

如在上例做果汁中，Java代码中的ctx，就是通过`Application ctx = new ClassPathXmlApplicationContext("spring-cfg.xml");`创建的，此处的spring-cfg.xml中配置了两个bean元素。

&nbsp;

### 3.2 初始化和依赖注入

Bean的初始化有3步：

1. Resource定位：容器根据开发者的配置，进行资源定位，通过XML或注解配置，就要定位到XML或注解
2. BeanDefinition载入：根据配置，获取POJO，用以生成对应实例
3. BeanDefinition注册：把2中载入的POJO往容器中注册，随后就可通过描述获取到

以上三步结束后，初始化完成，但还没有依赖注入，即没有将配置的资源注入给Bean。

配置项lazy-init需要提一下：默认default（false），表示默认自动初始化Bean；若true，则容器getBean方法时才初始化并依赖注入。

&nbsp;

### 3.3 Spring Bean的生命周期

IoC容器的本质目的就是为了管理Bean。

**初始化步骤：**

1. BeanNameAware.setBeanName()：若Bean实现了本接口方法，就会被调用
2. BeanFactoryAware.setBeanFactory()：如实现，就调用
3. ApplicationContextAware.setApplicaitonContext()：若Bean实现，且容器也要是ApplicationContext接口的实现类，才会调用
4. BeanPostProcessor.postProcessBeforeInitialization()
5. BeanFactoryPostProcessor.afterPropertiesSet()
6. Bean自定义的初始化方法
7. BeanPostProcessor.postProcessAfterInitailization()

完成以上方法调用后，Bean的初始化全部结束，正式存在于IoC容器中，使用者可以从中获取Bean的服务。

**销毁步骤：**（容器销毁要显式调用 `ctx.close();`）

1. DisposableBean.destroy()
2. 自定义销毁方法

另，自定义的init，destory方法，需要在xml中配置指明属性，属性值为方法名，才会运行：

```xml
<bean id="juiceMaker" class="xxx.JuiceMaker" init-method="init" destroy-method="destroy">
    ...
</bean>
```

&nbsp;





































