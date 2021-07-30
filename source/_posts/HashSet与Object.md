---
title: HashSet与Object
date: 2021-07-30 12:00:09
tags: Java
---



<!-- t o c -->

<!-- more -->



[参考1](https://blog.csdn.net/violet_echo_0908/article/details/50167939)

[参考2](https://blog.csdn.net/violet_echo_0908/article/details/50152915)



# 1. 问题

如下这个小例子，使用`HashSet<Loc>`，Loc是一个表示二维坐标的类，`set.add(new Loc(1, 2))`后，`set.contains(new Loc(1, 2))`时返回false。

```java
public class Test {

    static class Loc {
        int x, y;

        public Loc() {}
        
        public Loc(int x, int y) {
            this.x = x;
            this.y = y;
        }
    }

    public static void main(String[] args) {
        HashSet<Loc> set = new HashSet<>();
        set.add(new Loc(1, 2));
        System.out.println(set.contains(new Loc(1, 2))); // false
    }
}
```

IDEA会提示Loc没有重写`equals`和`hashCode`方法。



# 2. 解决

Java中的Hash，使用类的`hashCode()`计算hash值，判断两元素是否相同时使用`equals`，所以Hash数据结构使用类时，对应类要重写这两个方法。

Java中`equals()`和`hashCode()`的规定是：

1. 如果俩对象equal，那么必须有相同的hashCode
2. 即使两个对象有相同的hashCode，它们也不一定equal
3. hashCode()的默认实现是为不同的对象返回不同的整数.有一个设计原则是对于同一个对象，不管内部怎么改变，应该都返回相同的整数

但是为了解决具体问题，如上的Loc例子，重写`equals`和`hashCode`方法时，没有完全照顾到这些规则。

```java
public class Test {

    static class Loc {
        int x, y;

        public Loc() {}

        public Loc(int x, int y) {
            this.x = x;
            this.y = y;
        }

        @Override
        public int hashCode() {
            return x;
        }

        @Override
        public boolean equals(Object obj) {
            if (!(obj instanceof Loc)) {
                return false;
            }
            return ((Loc) obj).x == this.x && ((Loc) obj).y == this.y;
        }
    }

    public static void main(String[] args) {
        HashSet<Loc> set = new HashSet<>();
        set.add(new Loc(1, 2));
        System.out.println(set.contains(new Loc(1, 2)));
		// true
    }
}

```

