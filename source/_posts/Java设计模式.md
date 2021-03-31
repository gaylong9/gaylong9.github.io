---
title: Java设计模式
mathjax: false
date: 2021-03-23 11:19:50
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

[toc]



# Java设计模式

## 1. Java反射技术

反射技术能够配置类的全限定名、方法和参数、完成对象初始化操作、甚至反射某些方法。Spring IoC的基本原理也是反射技术。

Java反射内容繁多，本章重点讲解对象构建和方法的反射调用。Java中反射是通过包java.lang.reflect.\*实现。

### 1.1 通过反射构建对象

1. 无参构造：如ReflectSample类

	```java
	public class ReflectSample {
	    public void sayHello(String name) {
	        System.out.println("Hello " + name);
	    }
	}
	```

	通过反射方式构建对象

	```java
	public ReflectSample getInstance() {
	    ReflectSample obj = null;
	    try {
	        obj = (ReflectSample) Class.forName("xxx.ReflectSample").newInstance();
	    } catch (ClassNotFoundException e) {
	        e.printStackTrace();
	    } 
	    return obj;
	}
	```

	`Class.forName().newInstance()`是给类加载器注册一个类ReflectSample的全限定名，然后通过newInstance方法初始化一个对象。本例是一个无参构造的类。

2. 有参构造：以ReflectSample2类为例：

	```java
	public class ReflectSample2 {
	    private String name;
	    
	    public ReflectSample2 (String name) {
	        this.name = name;
	    }
	    
	    public void sayHello(String name) {
	        System.out.println("Hello " + name);
	    }
	}
	```

	此时的反射构建对象：

	```java
	public ReflectSample2 getInstance() {
	    ReflectSample2 obj = null;
	    try {
	        obj = (ReflectSample2) Class.forName("xxx.ReflectSample2").getConstructor(String.class).newInstance("Tom");
	    } catch (Exception e) {
	        e.printStackTrace();
	    } 
	    return obj;
	}
	```

	本例中的反射，先用forName加载到类的加载器，然后通过getConstructor方法 ，它的参数可以是多个，这里的String.class表示只有一个参数类型为String的构建方法，再用newInstance方法生成对象，实际等价于`obj = new ReflectSample2("Tom")`，只是使用了反射机制。

3. 反射只要配置就可生成对象，可以降低耦合，比较灵活，但比较慢。

&nbsp;

### 1.2 反射方法

使用反射方法前要先获取对象

```java
public Object reflectMethod() {
    Object obj = null;
    ReflectSample target = new ReflectSample();
    try {
        Method method = ReflectSample.class.getMethod("sayHello", String.class);
        obj = method.invoke(target, "Tom");
    } catch (Exception e) {
        e.printStackTrace();
    }
    return obj;
}
```

对于`ReflectSample.class.getMethod()`，若有对象但不知道类时，可以使用`target.getClass().getMethod()`，getMethod中第一个参数是方法名，第二个参数是方法参数类型，是一个列表。

具体反射方法的执行是`method.invoke()`实现，等价于`target.sayHello("Tom")`，若反射的方法有多个参数，则`invoke(target, obj1, obj2…)`即可。

&nbsp;

### 1.3 反射实例

完整通过反射创建对象并调用方法，继续以ReflectSample为例

```java
public Object reflect() {
    ReflectSample obj = null;
    try {
        obj = (ReflectSample) Class.forName("xxx.ReflectSample").newInstance();
        Method method = obj.getClass().getMethod("sayHello", String.class);
        method.invoke(obj, "Tom");
    } catch (Exception e) {
        e.printStackTrace();
    }
    return obj;
}
```

如此完全可以通过配置来完成对象和方法的反射，增强了Java的可配置性和可扩展性，IoC就是基于此法的典型例子。

&nbsp;

## 2. 动态代理模式和责任链模式

1. 动态代理是生成一个代理对象（占位），来代理真实对象，从而控制真实对象的访问
2. 代理模式：调用者与代理对象联系，代理对象与真实对象联系，如此可在访问真实对象的前或后，加入其它逻辑，并且可以判断是否需要访问真实对象等
3. 故代理分两步：代理对象和真实对象建立代理关系、实现代理对象的代理逻辑方法
4. Java有多种动态代理技术，Spring常用JDK（JDK自带功能）和CGLIB（第三方），MyBatis还使用了Javassist

### 2.1 JDK动态代理

是JDK自带功能，java.lang.reflect.\*提供的方式，必须**借助一个接口**才能产生代理对象：

```java
// 接口，其中定义的方法将是代理过程中可以代理执行的方法
public interface HelloWorld {
    public void syaHelloWorld();
}

// 实现类，也是代理中的真实对象类
public class HelloWorldImp implements HelloWorld {
    @override
    public void sayHelloWorld() {
        System.out.println("Hello World.");
    }
}
```

接下来需要将代理对象和真实对象绑定，并实现代理对象的代理逻辑方法。JDK动态代理中，要实现代理类必须实现java.lang.reflect.InvocationHandler接口，定义了invoke方法，并提供接口数组用于下挂代理对象：

```java
// 工具类
public class JdkProxyExample implements InvocationHandler {
    // 真实对象
    private Object target = null;
    
    /**
     * 建立代理对象和真实对象的代理关系，并返回代理对象
     * @param target 真实对象
     * @return 代理对象
     */
    public Object bind(Object target) {
        this.target = target;
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
    }
    
    /**
     * 代理方法逻辑
     * @param proxy 代理对象
     * @param method 当前调度方法
     * @param args 当前方法参数
     * @return 代理结果返回
     * @throws Throwable 异常
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("进入代理逻辑方法");
        System.out.println("在调度真实对象之前的服务");
        // 通过反射调用真实对象的方法
        Object obj = method.invoke(target, args);
        System.out.println("在调度真实对象之后的服务");
        return obj;
    }
}
```

第一步建立代理和真实对象联系时，`Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), this)`，第一个参数是真实对象的类加载器；第二个参数是要把生成的动态代理对象下挂在哪些接口，即最开始创建并实现的接口HelloWorld，则代理对象就可以通过`HelloWorld proxy = xxx;`声明；第三个参数是定义实现方法逻辑的代理类，需要实现InvocationHandler的invoke方法，它就是代理逻辑方法的实现方法。

第二步实现代理逻辑方法。invoke实现代理逻辑，有3个参数：proxy代理对象，就是bind生成的对象；method当前调度的犯法；args调度方法的参数。

最后测试此动态代理：

```java
public void testJdkProxy() {
    JdkProxyExample jdk = new JdkProxyExample();
    // 绑定，得到代理对象
    HelloWold proxy = (HelloWorld) jdk.bind(new HelloWorldImp());
    // 已代理，调用逻辑方法
    proxy.sayHelloWorld();
}
```

运行将会输出：

```
进入代理逻辑方法
在调度真实对象之前的服务
Hello World.
在调度真实对象之后的服务
```

&nbsp;

### 2.2 CGLIB动态代理

JDK动态代理只能提供接口才能使用，若不能提供接口，就要用其他第三方技术，如CGLIB动态代理，它只要一个非抽象类就能实现动态代理。

```java
public class CglibProxyExample implements MethodInterceptor {
	/**
	 * 生成CGLIB代理对象 
	 * @param cls -- 真实对象
	 * @return CGLIB代理对象
	 */
	public Object getProxy(Class cls) {
		// CGLIB enhancer增强类对象
		Enhancer enhancer = new Enhancer();
		// 设置增强类型
		enhancer.setSuperclass(cls);
		// 定义代理逻辑对象为当前对象，要求当前对象实现MethodInterceptor方法
		enhancer.setCallback(this);
		// 生成并返回代理对象
		return enhancer.create();
	}

	/**
	 * 代理逻辑方法
	 * @param proxy 代理对象
	 * @param method 方法
	 * @param args 方法参数
	 * @param methodProxy 方法代理
	 * @return 代理逻辑返回
	 * @throws Throwable异常
	 */
	@Override
	public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
		System.err.println("调用真实对象前");
		// CGLIB反射调用真实对象方法
		Object result = methodProxy.invokeSuper(proxy, args);
		System.err.println("调用真实对象后");
		return result;
	}
}
```

使用了CGLIB的加强者Enhancer，通过设置超类的方法（setSuperclass），通过setCallback方法设置哪个类为它的代理类。其中 参数this意味着是当前对象，就要求用this对象实现接口MethodInterceptor的方法intercept，然后返回代理对象。如此，当前类的intercept方法就是代理逻辑方法。

测试代理：

```java
public void testCGLIBProxy() {
    CglibProxyExample cpe = new CglibProxyExample();
    ReflectSample obj = (ReflectSample)cpe.getProxy(ReflectSample.class);
    obj.sayHello("Tom");
}
```

输出：

```
调用真实对象前
Hello Tom
调用真实对象后
```

JDK与CGLIB动态代理都是用getProxy方法生成代理对象，制定代理的逻辑类。代理逻辑类要实现一个接口的一个方法，那么这个接口定义的方法就是代理对象的逻辑方法，可以控制真实对象的方法。

&nbsp;

### 2.3 拦截器

由于动态代理的难以理解，通过设计者会设计一个拦截器接口供开发者使用，可将动态代理置入黑箱。

用JDK动态代理实现一个拦截器，首先需要定义拦截器接口Interceptor。

```java
public interface Interceptor {
	public boolean before(Object proxy, Object target, Method method, Object[] args);

	public void around(Object proxy, Object target, Method method, Object[] args);

	public void after(Object proxy, Object target, Method method, Object[] args);
}
```

参数proxy代理对象、target真实对象、method方法、args运行方法参数。

before方法返回boolean，在真实对象前调用，返回true时才反射真实对象的方法，若返回false则调用around方法。

在反射真实对象方法或around方法执行后，调用after方法。

接口实现类：

```java
public class MyInterceptor implements Interceptor {
	@Override
	public boolean before(Object proxy, Object target, Method method, Object[] args) {
		System.err.println("反射方法前逻辑");
		return false;// 不反射被代理对象原有方法
	}

	@Override
	public void after(Object proxy, Object target, Method method, Object[] args) {
		System.err.println("反射方法后逻辑。");
	}

	@Override
	public void around(Object proxy, Object target, Method method, Object[] args) {
		System.err.println("取代了被代理对象的方法");
	}
}
```

本类实现了接口的所有方法，使用JDK动态代理，就能实现这些方法的适时调用逻辑。

在JDK动态代理中使用拦截器：

```java
public class InterceptorJdkProxy implements InvocationHandler {

    private Object target; //真实对象
    private String interceptorClass = null;//拦截器全限定名
    
    public InterceptorJdkProxy(Object target, String interceptorClass) {
        this.target = target;
        this.interceptorClass = interceptorClass;
    }

    /**
     * 绑定真实对象并返回一个代理对象
     * @param target 真实对象
     * @return 代理对象
     */
    public static Object bind(Object target, String interceptorClass) {
        //取得代理对象    
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(), 
                new InterceptorJdkProxy(target, interceptorClass));
    }

    @Override
    /**
     * 通过代理对象调用方法，首先进入这个方法.
     * @param proxy --代理对象
     * @param method --方法，被调用方法
     * @param args -- 方法的参数
     */
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (interceptorClass == null) {
            //没有设置拦截器则直接反射原有方法
            return method.invoke(target, args);
        }
        Object result = null;
        //通过反射生成拦截器
        Interceptor interceptor = 
            (Interceptor) Class.forName(interceptorClass).newInstance();
        //调用前置方法
        if (interceptor.before(proxy, target, method, args)) {
            //反射原有对象方法
            result = method.invoke(target, args);
        } else {//返回false执行around方法
            interceptor.around(proxy, target, method, args);
        }
        //调用后置方法
        interceptor.after(proxy, target, method, args);
        return result;
    }
}
```

首先在bind方法中用JDK动态代理绑定了真实对象，返回代理对象。

然后若没有设置拦截器，则直接反射真实对象的方法，随后结束。

若有拦截器，则通过反射生成拦截器，并使用。

测试拦截器：

```java
public void testInterceptorJDKProxy () {
    HelloWorld proxy = (HelloWorld) InterceptorJdkProxy.bind(new HelloWroldImp(), "xxx.MyInterceptor");
    proxy.sayHelloWorld();
}
```

由于MyInterceptor中，before方法指定返回false，故输出：

```
反射方法前逻辑。
取代了被代理对象的方法。
反射方法后逻辑。
```

&nbsp;

### 2.4 责任链模式

当一个对象在一条链上被多个拦截器拦截处理时，就把这样的设计模式称为责任链模式。此模式下，可将原始对象层层动态代理，形成嵌套结构。即首先建立多个拦截器接口的实现类，代表多个拦截器，先后进行多次代理，使用前一次的代理对象当做后一次的真实对象。如此运行的话，会生成如下结果：

```
拦截器2的before方法
拦截器1的before方法
Hello World，真实对象的方法
拦截器1的after方法
拦截器2的after方法
```

可见，责任链模式优点在于可加入新的拦截器，增加拦截逻辑，但缺点是会增加代理和反射，而它们的性能都不高。

&nbsp;

## 3. 观察者模式 Observer

又称为发布订阅模式，是对象的行为模式。定义了一种一对多的依赖关系，让多个观察者对象同时监视被观察者状态，当被观察者变化时，通知所有观察者，并让它们主动更新自己。

如一个判断条件，决定很多后续执行的情况，若使用简单的if判断，则使得内容很多，需要大量catch以免后续执行之间的妨碍。而观察者模式更易于扩展，责任也更清晰。

本例为商家产品列表如有上新，则各个电商平台观察到后自动上新。

被观察者：需要继承java.util.Observable类，本例是商家的产品列表

```java
public class ProductList extends Observable {
    
    private List<String> productList = null;//产品列表
    private static ProductList instance;//类唯一实例    
    private ProductList() {}//构建方法私有化
    
    /**
     * 取得唯一实例
     * @return 产品列表唯一实例
     */
    public static ProductList getInstance() {
        if (instance == null) {
            instance = new ProductList();
            instance.productList = new ArrayList<String>();
        }
        return instance;
    }
    
    /**
     * 增加观察者（电商接口）
     * @param observer 观察者
     */
    public void addProductListObserver(Observer observer) {
        this.addObserver(observer);
    }
    
    /**
     *  新增产品
     * @param newProduct 新产品 
     */
    public void addProudct(String newProduct) {
        productList.add(newProduct);
        System.err.println("产品列表新增了产品："+newProduct);
        this.setChanged();//设置被观察对象发生变化
        this.notifyObservers(newProduct);//通知观察者，并传递新产品
    }
}
```

构建方法私有化，单例模式，只能通过getInstance方法获得实例。

观察者：需要实现Observer接口的update方法

```java
// 京东接口
public class JingDongObserver implements Observer {
	@Override
    public void update(Observable o, Object product) {
        String newProduct = (String) product;
        System.err.println("发送新产品【"+newProduct+"】同步到京东商城");
    }
}

// 淘宝接口
public class TaoBaoObserver implements Observer {
    @Override
    public void update(Observable o, Object product) {
        String newProduct = (String) product;
        System.err.println("发送新产品【"+newProduct+"】同步到淘宝商城");
    }    
}
```

测试本例：

```java
public static void main(String[] args) {
	ProductList observable = ProductList.getInstance();
	TaoBaoObserver taoBaoObserver = new TaoBaoObserver();
	JingDongObserver jdObserver = new JingDongObserver();
	observable.addObserver(jdObserver);
	observable.addObserver(taoBaoObserver);
	observable.addProudct("产品1");
}
```

得到输出：

```
产品列表新增了产品：产品1
发送新产品【产品1】同步到淘宝商城
发送新产品【产品1】同步到京东商城
```

&nbsp;

## 4. 工厂模式和抽象工厂模式

对于同一大类下的诸多类，用户只需知道自己需要什么类，具体实例化交由工厂实现并返回。

### 4.1普通工厂模式

有一Product接口，有5个类实现了接口，为Product1、Product2、…、Product5，并且由ProductFactory同一管理。用户只需使用工厂中的createProduct方法，并传入所需序号，工厂即可实例化并返回对象。

```java
public class ProductFactory {
    public static Product createProduct (String productNo) {
        switch (productNo) {
            case "1": return new Product1();
            ...
            case "5": return new Product5();
            default: throw new NotSuppportedException("不支持此编号产品");
        }
    }
}
```

&nbsp;

### 4.2 抽象工厂模式

普通工厂，解决了一类对象创建问题，但有时对象复杂，有很多种，同时又细分了几个类别。此时若只用一个工厂，就会造成工厂内部实现逻辑繁杂，这就需要划分工厂，但仍希望留一个统一的工厂给使用者，再由这个总工厂接收任务后下发给具体完成对象创建的分工厂。这里的总工厂并不实际存在，是通过各分厂实现的，称为抽象工厂。如此封装，简化了调用者的使用过程。

抽象工厂模式可以向调用者提供一个接口，使其在不必指定产品的具体情况下，创建多个产品族中的产品对象。

为了统一，首先需要定义一个接口，每个工厂（包括虚拟工厂）都要实现这个接口：

```java
public interface ProductFactoryInter {
    public Product createProduct (String productNo);
}
```

随后创建两个分厂，用于实现工厂的具体实现方法：

```java
public class ProductFactory1 implements ProductFactoryInter {
    public Product createProduct (String productNo) {
        Product product = xxx;	// 本分厂内生产的规则
        return product;
    }
}

public class ProductFactory2 implements ProductFactoryInter {
    public Product createProduct (String productNo) {
        Product product = xxx;	// 本分厂内生产的规则
        return product;
    }
}
```

创建总工厂/抽象工厂，并设定任务分发规则，即productNo编号，1开头则交由分厂1完成，2开头则交由分厂2完成：

```java
public class ProductFactory implements ProductFactoryInter {
    public static Product createProduct (String productNo) {
        char ch = productNo.charAt(0);
        ProductFactoryInter factory = null;
        if (ch == '1') {
            factory = new ProductFactory1();
        } else if (ch == '2') {
            factory = new ProductFactory2();
        }
        if (factory != null) {
            return factory.createProduct(productNo);
        }
        return null;
    }
}
```

如此，就实现了抽象工厂与具体完成任务的分厂之间的联系。调用者只需通过抽象工厂发布任务，抽象工厂自动分发任务给分厂完成并返回结果。各分厂只需维护自己的代码即可。

&nbsp;

## 5. 建造者模式 Builder

属于对象的创建模式，可以将一个产品的属性与产品的生产过程分开，从而使一个建造过程生成具有不同属性的对象。

适用于对象具有较多属性，导致一个产品族中有很多不同产品，使用new或工厂模式时，传入过多参数造成不便。

Builder模式是分步构建对象的模式，是逐个产品进行构建，用一个配置类对这些步骤进行统筹，然后将所有信息交给构建器完成构建对象。

本例中，用门票展示，门票包含普通成年人票、老人票、儿童票、军属票。

首先创建配置类 TicketHelper，帮助我们一步步完成构建对象：

```java
public class TickerHelper {
    public void buildAdult (String info) {
        System.out.println("输出模拟构建成人票逻辑，" + info);
    }
    public void buildChild (String info) {
        System.out.println("输出模拟构建儿童票逻辑，" + info);
    }
    public void buildElderly (String info) {
        System.out.println("输出模拟构建老人票逻辑，" + info);
    }
    public void buildSoldier (String info) {
        System.out.println("输出模拟构建军属票逻辑，" + info);
    }
}
```

然后需要一个构建类 TicketBuilder：

```java
public class TicketBuilder {
    public static Object builder (TicketHelper helper) {
        System.out.println("通过配置类TicketHelper构建门票信息");
        return null;
    }
}
```

完成门票构建：

```java
TicketHelper helper = new TicketHelper();
helper.buildAdult("成人票");
helper.buildChild("儿童票");
helper.buildElderly("老人票");
helper.buildSoldier("军属票");
Object ticket = TicketBuilder.builder(helper);
```

&nbsp;

