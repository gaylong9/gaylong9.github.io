---
title: Redis_2.数据结构与常用命令
mathjax: false
date: 2021-06-11 17:52:22
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

实际工作中多个命令应使用SessionCallback接口，本章仅为演示命令，仅使用RedisTemplate完成。

&nbsp;

# 1. 字符串

一些常用基本命令：

| 命令                   | 说明                        | 备注                                   |
| ---------------------- | --------------------------- | -------------------------------------- |
| set key value          | 设置键值对                  | 最常用的写入命令                       |
| get key                | 通过键获取值                | 最常用的读取命令                       |
| del key                | 通过key删除键值对           | 返回删除数；通用，其他结构也可用此命令 |
| strlen key             | 求value字符串的长度         | 返回长度                               |
| getset key value       | 修改原来的value，并返回旧值 |                                        |
| getrange key start end | 获取子串                    | [start, end]是索引                     |
| append key value       | 新value加入到原value末尾    | 返回新value的长度                      |

将键值的序列化器均设置为StringRedisSerializer，可对value以String的形式操作：

```java
ApplicationContext context = new ClassPathXmlApplicationContext("spring_config.xml");
RedisTemplate template = context.getBean(RedisTemplate.class);
// set
template.opsForValue().set("key2", "value2");
template.opsForValue().set("key3", "value3");
// get
String value2 = (String) template.opsForValue().get("key2");
System.out.println(value2);
// del
template.delete("key2");
value2 = (String) template.opsForValue().get("key2");
System.out.println(value2);
// strlen
Long len3 = template.opsForValue().size("key3");
System.out.println(len3);
// getset
String oldValue3 = (String) template.opsForValue().getAndSet("key3", "newValue3");
System.out.println(oldValue3);
// getrange
String rangeValue3 = template.opsForValue().get("key3", 1, 2);
System.out.println(rangeValue3);
// append
int newLen = template.opsForValue().append("key3", "_append");
System.out.println(newLen);
String newValue3 = (String) template.opsForValue().get("key3");
System.out.println(newValue3);
```

另Redis中数字也用字符串存储，且支持一些简单运算：

| 命令                      | 说明                 | 备注               |
| ------------------------- | -------------------- | ------------------ |
| incr key                  | +1                   | 整数操作           |
| incrby key increment      | +increment           | 整数操作           |
| decr key                  | -1                   | 整数操作           |
| decrby key decrement      | -decrement           | 整数操作           |
| incrbyfloat key increment | +increment（浮点数） | 浮点数或整数的操作 |

因为Redis对数字支持很弱，甚至不支持减法和乘除法，一般是Java获取数字后运算再存入Redis。

```java
template.opsForValue().set("i", "8");
System.out.println((String) template.opsForValue().get("i"));
template.opsForValue().increment("i");
System.out.println((String) template.opsForValue().get("i"));
template.opsForValue().increment("i", 1);
System.out.println((String) template.opsForValue().get("i"));
template.opsForValue().increment("i", 2.3);
System.out.println((String) template.opsForValue().get("i"));
template.opsForValue().increment("i", -2.3);
System.out.println((String) template.opsForValue().get("i"));

template.getConnectionFactory().getConnection().decr(
        template.getKeySerializer().serialize("i"));
System.out.println((String) template.opsForValue().get("i"));
template.getConnectionFactory().getConnection().decrBy(
        template.getKeySerializer().serialize("i"), 6);
System.out.println((String) template.opsForValue().get("i"));
```

Spring对`increment()`方法进行重载，支持自增、加整数/浮点数。

减法RedisTemplate没有支持，书中用上述的复杂代码完成，不过用加法的加负数不也可以吗？

且Spring中`decr()`不支持对浮点数操作。

&nbsp;

# 2. 哈希

Redis中的hash是一种数据结构，一个hash结构可以存储2^32-1个键值对。Hash结构的名字可以看作key，通过它找到Hash结构，内部是String类型的field和value的映射表（内部键值对），即一个hash内有多个field和value的映射。

常用命令如下；

| 命令                             | 说明                           | 备注                 |
| -------------------------------- | ------------------------------ | -------------------- |
| hdel key field1                  | 删除hash结构中的某个字段       | 可以多字段删除       |
| hexists key fileld               | 判断hash结构中是否存在某个字段 | 返回1或0             |
| hgetall key                      | 获取所有键值                   |                      |
| hincrby key field increment      | 指定某一字段+整数              | 要求该字段本身为整数 |
| hincrbyfloat key field increment | 某一字段+浮点数                | 要求字段本身是数字   |
| hkeys key                        | 返回所有键                     |                      |
| hlen key                         | 键值对数量                     |                      |
| hmget key field1                 | 返回指定键的值                 |                      |
| hmset key field1 value1          | hash中设置多个键值对           |                      |
| hset key field value             | hash中设置键值对               | 对应有hget           |
| hsetnx key field value           | 不存在对应field才设置          |                      |
| hvals key                        | 获取所有value                  |                      |

有几点注意事项：

1. hash结构能存储相当多的键值对，代码中要注意hkeys、hgetall、hvals等获取所有内容的命令
2. 数字的操作，对value有类型要求

在Spring中使用hash时，要先设置hashKeySerializer和hashValueSerializer两属性，此处指定默认序列化器`<property name="defaultSerializer" ref="stringRedisSerializer"/>`，随后是Spring的操作：

```java
String key = "hash";
Map<String, String> map = new HashMap<>(10);
map.put("f1", "val1");
map.put("f2", "val2");
// hmset
template.opsForHash().putAll(key, map);
// hset
template.opsForHash().put(key, "f3", "3");
// hget
System.out.println(template.opsForHash().get(key, "f3"));
// hexists key field
System.out.println(template.opsForHash().hasKey(key, "f3"));
// hgetall
Map keyValMap = template.opsForHash().entries(key);
System.out.println(keyValMap.keySet());
System.out.println(keyValMap.values());
// hincrby
template.opsForHash().increment(key, "f3", 2);
System.out.println(template.opsForHash().get(key, "f3"));
// hincrbyfloat
template.opsForHash().increment(key, "f3", -2.2);
System.out.println(template.opsForHash().get(key, "f3"));
// hvals
List valueList = template.opsForHash().values(key);
System.out.println(valueList);
// hkeys
Set keyList = template.opsForHash().keys(key);
System.out.println(keyList);
// hmget
List valueList2 = template.opsForHash().multiGet(key, keyList);
// hsetnx
System.out.println(template.opsForHash().putIfAbsent(key, "f4", "val4"));
// hdel
System.out.println(template.opsForHash().delete(key, "f1", "f2"));
```

&nbsp;

# 3. 链表

存储多个字符串，有序的，双向的，能够存储2^32-1个节点。

作为一个双向链表，读性能很差，但插入和删除有优势，故使用时需要对场景甄别，判断是否适合链表。

常用命令：命令分为左右两种，以下以左端命令为主，部分命令有右端形式，改为r即可

| 命令                                | 说明                                                         | 备注                                 |
| ----------------------------------- | ------------------------------------------------------------ | ------------------------------------ |
| lpush key node1                     | 左端push                                                     | 若多个节点，则结果会相反             |
| rpush key node1                     | 右端push                                                     | ``                                   |
| lindex key index                    | 左侧开始查找索引index的节点                                  | 返回节点字符串                       |
| llen key                            | 链表长度，返回节点数                                         |                                      |
| lpop key                            | 左端第一个节点删除并返回                                     |                                      |
| linsert key before/after pivot node | 在指定值为pivot的节点前后插入新节点                          | list不存在则报错；没有指定节点返回-1 |
| lpushx list node                    | 若存在key为list的链表，则插入node                            |                                      |
| lrange key start end                | [start, end]节点值                                           |                                      |
| lrem key count value                | count=0，则删除所有值为value的节点；否则删除不大于\|count\|个值为value的节点 | 返回删除个数                         |
| lset key index node                 | 设置索引index的节点值为node                                  |                                      |
| ltrim key start stop                | 只保留[start, stop]的节点                                    |                                      |

以上命令线程不安全，并发易冲突，有以下阻塞命令，会给链表加锁：

| 命令                            | 说明                         | 备注 |
| ------------------------------- | ---------------------------- | ---- |
| blpop key timeout               | 若空，阻塞至超时或有元素     |      |
| rpoplpush key src dest          | 右侧移除，插入到目标链表左侧 |      |
| brpoplpush key src dest timeout | 可以设置超时时间             |      |

以下是Spring中使用：也以左为主

```java
// lpush
template.opsForList().leftPush("list", "node3");
List<String> nodes = Arrays.asList("node2,node".split(","));
template.opsForList().leftPushAll("list", nodes);
// rpush
template.opsForList().rightPush("list", "node4");
// set
template.opsForList().set("list", 0, "node1");
// lindex
System.out.println(template.opsForList().index("list", 0));
// pop
System.out.println(template.opsForList().leftPop("list"));
System.out.println(template.opsForList().rightPop("list"));
// pushx
template.opsForList().leftPushIfPresent("list", "node1");
template.opsForList().rightPushIfPresent("list", "node4");
// llen
long size = template.opsForList().size("list");
// range
System.out.println(template.opsForList().range("list", 0, size-1));
// lrem
nodes = Arrays.asList("node,node".split(","));
template.opsForList().leftPushAll("list", nodes);
template.opsForList().remove("list", 2, "node");
System.out.println(template.opsForList().range("list", 0, size-1));
// linsert较为复杂，需要底层命令
template.getConnectionFactory().getConnection().lInsert(
        "list".getBytes(StandardCharsets.UTF_8),
        RedisListCommands.Position.BEFORE,
        "node2".getBytes(StandardCharsets.UTF_8),
        "before_node".getBytes(StandardCharsets.UTF_8));
System.out.println(template.opsForList().range("list", 0, size-1));

// blpop
template.opsForList().leftPop("list", 1, TimeUnit.SECONDS);
// rpoplpush
template.opsForList().leftPush("list1", "node2InL1");
template.opsForList().leftPush("list1", "node1inL1");
System.out.println(template.opsForList().range("list", 0, 10));
System.out.println(template.opsForList().range("list1", 0, 10));
template.opsForList().rightPopAndLeftPush("list1", "list", 1000, TimeUnit.MILLISECONDS);
System.out.println(template.opsForList().range("list", 0, 10));
System.out.println(template.opsForList().range("list1", 0, 10));

template.delete("list");
template.delete("list1");
```

&nbsp;

# 4. 集合

集合同样是哈希表结构，同样理论存储2^32-1个元素，故集合中的操作复杂度都是O(1)。它有三个特点：无序、不重复、每个元素都是String结构。

Redis还可以对多个集合操作，如交差并集。

常用命令如下：

| 命令                        | 说明                                      | 备注                                   |
| --------------------------- | ----------------------------------------- | -------------------------------------- |
| sadd key member1            | 增加成员                                  |                                        |
| scard key                   | 统计成员数                                |                                        |
| sdiff key1 [key2]           | 两集合差集                                | 若一个集合，就返回所有元素             |
| sdiffstore des key1 [key2]  | sdiff的结果保存到des集合                  |                                        |
| sinter key1 [key2]          | 交集                                      | 同sdiff                                |
| sinterstore des key1 [key2] |                                           |                                        |
| sismember key member        |                                           | 是返回1；不是返回0                     |
| smembers key                | 返回所有成员                              |                                        |
| smove src des member        | 将一个成员从src迁移到des集合              |                                        |
| spop key                    | 随机弹出一个元素                          |                                        |
| srandmember key [count]     | 随机返回一个或多个元素，\|count\|限制总数 | count不填默认1，大于总数就返回整个集合 |
| srem key member1            | 移除元素                                  |                                        |
| sunion key1 [key2]          | 并集                                      | 单key返回所有元素                      |
| sunionstore des key1 [key2] |                                           |                                        |

Spring中使用：

```java
Set set = null;
// sadd
template.boundSetOps("set1").add("v1", "v2");
template.boundSetOps("set2").add("v0", "v1");
// scard
template.opsForSet().size("set1");
// sdiff
set = template.opsForSet().difference("set1", "set2");
// sinter
set = template.opsForSet().intersect("set1", "set2");
// sismember
boolean exits = template.opsForSet().isMember("set1", "v1");
// smembers
set = template.opsForSet().members("set1");
// spop
String val = (String) template.opsForSet().pop("set1");
// srandmember
val = (String) template.opsForSet().randomMember("set1");
List list = template.opsForSet().randomMembers("set1", 2);
// srem
template.opsForSet().remove("set1", "v1");
// sunion
template.opsForSet().union("set1", "set2");
// sdiffstore
template.opsForSet().differenceAndStore("set1", "set2", "diff_set");
```

&nbsp;

# 5. 有序集合

zset

与无序集合的区别就是每个元素还有一个“分数”，是一个浮点数，Redis利用该分数排序。有序集合同样元素唯一、String类型、hash结构，但分数可以相同。元素除了值、分数，还有key属性，由其标示属于哪个集合。

基础命令：

| 命令                                                        | 说明                                                         | 备注                                  |
| ----------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------- |
| zadd key score1 value1                                      | 增加成员                                                     | 无key则创建zset                       |
| zcard key                                                   | 获取成员数                                                   |                                       |
| zcount key min max                                          | 根据分数返回成员个数                                         | 默认包含端点，在分数前加“(”表示不包含 |
| zincrby key increment member                                | 给值为member的成员加increment                                |                                       |
| zinterstore desKey numkeys key1                             | 交集，保存至desKey                                           | numKeys表示有几个集合                 |
| zlexcount key min max                                       | 指定值的区间内成员数量                                       | “[”表示包含，“(”不包含                |
| zrange key start stop [withscores]                          | 按分数升序返回成员，可用start stop限制范围；withscores协同分数一起返回 | 包含两端                              |
| zrank key member                                            | 求索引                                                       |                                       |
| zrangebylex key min max [limit offset count]                | 按值升序排列，随后（按索引限制）返回成员                     | “[”包含，“(”不包含                    |
| zrangebyscore key min max [withscores] [limit offset count] | 根据分数求范围                                               |                                       |
| zremrangebyscore key start stop                             | 根据分数区间删除                                             | 按分数排序                            |
| zremrangebyrank key start stop                              | 按分数升序删除                                               |                                       |
| zremrangebylex key min max                                  | 按值删除                                                     |                                       |
| zrevrange key start stop [withscores]                       | 降序排序                                                     |                                       |
| zrevrangebyscore key max min [withscores]                   |                                                              |                                       |
| zrevrank key member                                         |                                                              |                                       |
| zscore key member                                           | 返回成员分数                                                 |                                       |
| zunionstore desKey numKeys key1                             | 并集                                                         |                                       |

zset使用频率不高，使用时要注意区分值和分数。

Spring内部定义了TypedTuple内部接口，内有`getValue()` `getScore()`两方法。默认实现类是DefaultTypedTuple，默认情况下Spring把值和分数封装进这个类。

另外Spring还对范围进行了封装，RedisZSetCommands的内部类Range，有静态`range()`方法，用其生成Range对象，有几个方法，常用的有以下四个：

1. 大于等于 `Range get(min)`
2. 大于 `Range ge(min)`
3. 小于等于 `Range lte(max)`
4. 小于 `Range lt(max)`

限制也有实现类，是RedisZSetCommands的内部类，是一个简单POJO，有offset和count两属性。

部分Spring使用：

```java
Set<ZSetOperations.TypedTuple> set1 = new HashSet<>();
Set<ZSetOperations.TypedTuple> set2 = new HashSet<>();
for (int i = 1, j = 9; i<=9; i++) {
    j--;
    Double score1 = (double) i;
    String value1 = "x" + i;
    Double score2 = (double) j;
    String value2 = j % 2 == 1? "y" + j: "x" + j;
    ZSetOperations.TypedTuple typedTuple1 = new DefaultTypedTuple(value1, score1);
    set1.add(typedTuple1);
    TypedTuple typedTuple2 = new DefaultTypedTuple(value2, score2);
    set2.add(typedTuple2);
}
// zadd
template.opsForZSet().add("zset1", set1);
template.opsForZSet().add("zset2", set2);
// zcard
Long size = template.opsForZSet().zCard("zset1");
// zcount min max, <=score<=
size = template.opsForZSet().count("zset1", 3, 6);
// zrange 不返回分数
Set set = template.opsForZSet().range("zset1", 1, 5);
System.out.println(set);
// zrange withscores
set = template.opsForZSet().rangeWithScores("zset1", 0, -1);
System.out.println(set);
// 区间 zrangebylex
RedisZSetCommands.Range range = RedisZSetCommands.Range.range();
range.lt("x8");
range.gt("x1");
set = template.opsForZSet().rangeByLex("zset1", range);
System.out.println(set);
// 限制
RedisZSetCommands.Limit limit = RedisZSetCommands.Limit.limit();
limit.count(4);
limit.offset(5);
set = template.opsForZSet().rangeByLex("zset1", range, limit);
// zrank
Long rank = template.opsForZSet().rank("zset1", "x4");
```

&nbsp;

# 6. 基数

HyperLogLog

是一种算法，不是存储的数据。优点是在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

| 命令                | 说明   | 备注          |
| ------------------- | ------ | ------------- |
| pfadd key element   | add    | 已存储过返回0 |
| pfcount key         | 基数值 |               |
| pfmerge deskey key1 | 合并   |               |

Spring中其方法都是`opsForHyperLogLog()`的方法，有`add()` `size()` `union()`，使用不多，不再赘述。

&nbsp;

