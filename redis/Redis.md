# Redis

## 1.理解

redis是一个开源的使用C语言编写的支持网络交互的可基于内存也可持久化的k-v数据库（非关系型数据库）

## 2.安装与运行

在网上就能找到相关教程。

安装完成后，进入安装目录src/文件夹中运行命令：

```mysql
##运行redis
./redis-server
##进入安装目录src/文件夹中打开终端或cmd使用 ./redis-cli -p 6379 进行连接redis
./redis-cli -p 6379
##设置key与value
set k1 helloworld
##通过key查找value
get k1
##DBSIZE查看当前数据库的key的数量
DBSIZE
##keys *查看当前数据库所有key
keys *
##SELECT 1切换到1数据库，默认创建了16个数据库，以0为开头
SELECT 1
##Flushdb：清空当前库
##Flushall：清空全部的库
```

操作key

```bash
##删除当前数据库
127.0.0.1:6379> FLUSHDB
OK
##查看当前数据库的所有key
127.0.0.1:6379> keys *
(empty list or set)
##赋值
127.0.0.1:6379> set name aixin
OK
127.0.0.1:6379> keys *
1) "name"
##查看name的key是否存在
127.0.0.1:6379> exists name
(integer) 1
127.0.0.1:6379> set name1 lala
OK
127.0.0.1:6379> keys *
1) "name1"
2) "name"
##将name1移动到数据库1中
127.0.0.1:6379> move name1 1
(integer) 1
127.0.0.1:6379> keys *
1) "name"
##切换数据库
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> keys *
1) "name1"
127.0.0.1:6379[1]> select 0
OK
127.0.0.1:6379> set name1 lala
OK
##给name1设置有效期为5秒。当 key 过期时(生存时间为 0 )，它会被自动删除。
127.0.0.1:6379> expire name1 5
(integer) 1
##查看name1剩余的有效期，-1 表示永不过期，-2 表示已过期
127.0.0.1:6379> ttl name1
(integer) 1
127.0.0.1:6379> ttl name1
(integer) -2
127.0.0.1:6379> keys *
1) "name"
##查看你的key是什么类型的
127.0.0.1:6379> type name
string
127.0.0.1:6379> 
```



压力测试

```bash
./redis-benchmark -h localhost -p 6379 -c 100 -n 100000
##结果
====== PING_INLINE ======
  100000 requests completed in 1.29 seconds
  100 parallel clients
  3 bytes payload
  keep alive: 1

72.87% <= 1 milliseconds
96.67% <= 2 milliseconds
99.14% <= 3 milliseconds
99.55% <= 4 milliseconds
99.78% <= 5 milliseconds
99.84% <= 6 milliseconds
99.90% <= 26 milliseconds
99.91% <= 27 milliseconds
99.97% <= 28 milliseconds
100.00% <= 28 milliseconds
77220.08 requests per second  #每秒处理了70000多的请求
```



## 3.redis5大数据类型

### 3.1、String

String是redis最基本的数据类型，是二进制安全的，可以包含任何数据，包括图片或序列化的对象。

```bash
######################################################################################################################

127.0.0.1:6379> set name aixin  ##赋值操作
OK
127.0.0.1:6379> keys *          ##查看所有的key
1) "name"
127.0.0.1:6379> get name		##根据key查找value
"aixin"
127.0.0.1:6379> exists name		##判断key是否存在
(integer) 1
127.0.0.1:6379> append name 123 ##给name拼接字符串
(integer) 8
127.0.0.1:6379> get name
"aixin123"
127.0.0.1:6379> append name1 ailalalala ##如果key不存在则创建
(integer) 10
127.0.0.1:6379> strlen name		##获取key存放value的长度
(integer) 8

######################################################################################################################
##对于数字增加减少的操作
127.0.0.1:6379> set views 0
OK
127.0.0.1:6379> get views
"0"
127.0.0.1:6379> incr views   ##加1
(integer) 1
127.0.0.1:6379> get views
"1"
127.0.0.1:6379> decr views  ##减一
(integer) 0
127.0.0.1:6379> decr views
(integer) -1
127.0.0.1:6379> get views
"-1"
127.0.0.1:6379> incrby views 5 ##增加指定数量
(integer) 4
127.0.0.1:6379> decrby views 2 ##减少指定数量
(integer) 2

######################################################################################################################
##范围
127.0.0.1:6379> set k1 hello,nihao
OK
127.0.0.1:6379> get k1
"hello,nihao"
127.0.0.1:6379> getrange k1 0 1   ##截取字符串【0，1】
"he"
127.0.0.1:6379> getrange k1 0 -1  ##使用-1代表获取所有字符串
"hello,nihao"

127.0.0.1:6379> set k2 abc111efg222
OK
127.0.0.1:6379> get k2
"abc111efg222"
127.0.0.1:6379> setrange k2 1 666 ##从下标1开始替换666
(integer) 12
127.0.0.1:6379> get k2
"a66611efg222"
######################################################################################################################
# setex:设置过期时间
# setnx:如果不存在才创建 在分布式锁中常常使用
127.0.0.1:6379> setex k1 10 hello  ##将k1赋值为hello，并设置有效期为10秒
OK
127.0.0.1:6379> get k1
"hello"
127.0.0.1:6379> ttl k1  ##十秒后查看有效期以过期
(integer) -2
127.0.0.1:6379> setnx k2 value  ##如果k2不存在则赋值
(integer) 1
127.0.0.1:6379> setnx k2 lala	##如果k2不存在则赋值
(integer) 0
127.0.0.1:6379> get k2  ##可以看到k2还是原来的值
"value"
######################################################################################################################
##批量操作
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3  ##批量设置
OK
127.0.0.1:6379> mget k1 k2 k3			##批量获取
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> msetnx k1 v2 k4 v4     ##如果不存在同时设置多个值，是一个原子性操作，可以看到k4也没有进行赋值。因为k1没有成功所以k4也没有成功
(integer) 0
######################################################################################################################
#对象
#
127.0.0.1:6379> set user:1 {name:lala,age:19} ##设置一个user:1对象，值为json字符来保存一个对象
OK
127.0.0.1:6379> get user
(nil)
127.0.0.1:6379> get user1
(nil)
127.0.0.1:6379> get user:1
"{name:lala,age:19}"

127.0.0.1:6379> mset user:2:name kaka user:2:age 18 ##也可以这样去进行赋值
OK
127.0.0.1:6379> get user:2
(nil)
127.0.0.1:6379> mget user:2
1) (nil)
127.0.0.1:6379> mget user:2:name user:2:age  ##只能通过这种方式进行取值
1) "kaka"
2) "18"
######################################################################################################################
127.0.0.1:6379> getset k1 redis  ##先get后set，如果不存在返回nil
(nil)
127.0.0.1:6379> get k1
"redis"
127.0.0.1:6379> getset k1 mysql ##如果存在返回旧值，设置新值
"redis"
127.0.0.1:6379> get k1
"mysql"



```



### 3.2、Hash

hash，类似Java里的map。

redis里的hash是一个String类型的field和value的映射表。适合存放对象。

```bash
######################################################################################################################
hset myhash f1 v1 #存值
hset myhash f4 v4 f5 v5 #存放多个
hget myhash f2 #获取
hmget myhash f2 f3 #获取多个
hgetall myhash #获取全部数据
hdel myhash f1 #删除hash指定key，对应的value也就消失了
hlen myhash #获取hash的长度
hexists myhash f3#判断hash指定字段key是否存在，不存在为0
hkeys myhsah #获取所有的key
hvals myhash#获取所有的values
hincrby myhash f3 1 #为hash指定key的value值加1
hsetnx myhash f1 123 #如果没有hash不存在发则设置
```



### 3.3、List

redis的list是一个简单的String列表，按照插入的顺序，可以将插入的数据放在列表的头部或尾部。

list的底层是一个链表。

```bash
######################################################################################################################
#列表的插入与读取L可以当作list也可以解读为left
127.0.0.1:6379> lpush list one  ##在列表的头部（左侧）插入one
(integer) 1
127.0.0.1:6379> lpush list two
(integer) 2
127.0.0.1:6379> lpush list three
(integer) 3
127.0.0.1:6379> lrange list 0 -1 ##获取全部列表的值
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> lrange list 0 1 ##获取列表0~1的值
1) "three"
2) "two"
127.0.0.1:6379> rpush list one ##在列表的尾部（右侧）插入one。这里的R理解为right
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "two"
3) "one"
4) "one"

######################################################################################################################
#移除
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "two"
3) "one"
4) "one"
127.0.0.1:6379> lpop list  ##移除列表第一个元素（最左边）
"three"
127.0.0.1:6379> rpop list ##移除列表最后一个元素（最右边）
"one"
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "one"
######################################################################################################################
#index
127.0.0.1:6379> lindex list 0 ##根据下标获取list中对应的元素
"two"
127.0.0.1:6379> lindex list 1
"one"
127.0.0.1:6379> lindex list 3
(nil)
127.0.0.1:6379> llen list ##获取长度
(integer) 2
######################################################################################################################
#移除操作
127.0.0.1:6379> lrange list 0 -1
1) "four"
2) "three"
3) "two"
4) "one"
5) "one"
6) "two"
7) "one"
127.0.0.1:6379> lrem list 1 one ##移除列表中1个one
(integer) 1
127.0.0.1:6379> lrange list 0 -1 ##可以看到是从最上面的元素开始移除
1) "four"
2) "three"
3) "two"
4) "one"
5) "two"
6) "one"
127.0.0.1:6379> lrem list 2 two##移除列表中2个two
(integer) 2
127.0.0.1:6379> lrange list 0 -1
1) "four"
2) "three"
3) "one"
4) "one"
127.0.0.1:6379> lrem list 2 four##移除列表中2个four,如果列表中没有那么多可以移除的就又多少移除多少
(integer) 1
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "one"
3) "one"
######################################################################################################################
##ltrim
127.0.0.1:6379> lpush list hello hello1 hello2 hello3
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "hello3"
2) "hello2"
3) "hello1"
4) "hello"
127.0.0.1:6379> ltrim list 0 1 ##将当前列表截取为指定下标的列表
OK
127.0.0.1:6379> lrange list 0 -1
1) "hello3"
2) "hello2"
######################################################################################################################
##rpoplpush
127.0.0.1:6379> rpush list hello hello1 hello2 hello3
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "hello"
2) "hello1"
3) "hello2"
4) "hello3"
127.0.0.1:6379> rpoplpush list otherList #将列表最后一个元素移除并放在另一个列表中
"hello3"
127.0.0.1:6379> keys *
1) "otherList"
2) "list"
127.0.0.1:6379> lrange otherList 0 -1
1) "hello3"
######################################################################################################################
##lset修改指定下标元素
127.0.0.1:6379> lrange list 0 -1
1) "hello"
2) "hello1"
3) "hello2"
127.0.0.1:6379> lset list 0 hello0 ##将list中0号下标元素修改为hello0
OK
127.0.0.1:6379> lindex list 0
"hello0"
######################################################################################################################
##linsert
linsert list before hello1 lala #将lala放在list中hello1的前面
linsert list after hello1 lala #将lala放在list中hello1的后面
```



### 3.4、set

redis的set是一个String类型的无序的不可重复的集合，通过HashTable来实现的。

```bash
######################################################################################################################
#存取
127.0.0.1:6379> sadd set1 hello hello #set中存放数据
(integer) 1
(2.31s)
127.0.0.1:6379> smembers set1 #获取set中的所有数据，可以看到不许重复
1) "hello"
sismember set1 hello #判断hello是否存在
scard set1 #获取set的数量
srem set1 hello1 #移除set中的hello1
srandmember set1 1 #随机抽选出1个元素
spop set1 1 #删除指定数量的元素（随机删除）
smove myset ohterSet s2 #将一个指定值移动到另一个set中
sdiff myset yourset #myset想对于yourset的差集
sinter myset yourset#两个集合的交集
sunion myset yourset#两个集合的并集

```

共同关注

### 3.5、ZSet

redis的Zset是一个String类型的不可重复的有序集合。不同的是每一个元素都会关联一个double类型的分数。就是通过这个分数来为集合进行排序，double可重复。

```bash
######################################################################################################################
zadd myzset 1 one 2 two 3 three #添加（one前面的1就代表它的分数）
zrange myzset 0 -1 #获取（range用法同其他的）
zrange myzset 0 -1 withscores#获取所有结果带分数
zrangebyscore myzset 0 4#返回有序集合中指定分数区间的成员列表。有序集成员按分数值递增(从小到大)次序排列
zrangebyscore myzset (0 (4#也可以通过给参数前增加 ( 符号来使用可选的开区间 (小于或大于)。0<score<4
zcard myzset #获取有序集合的成员数
zcount myzset 0 2#计算在有序集合中指定区间分数的成员数
zincrby myzset 5 one#有序集合中对指定成员的分数加上增量 increment
zrem myzset two three#移除有序集合中的一个或多个成员
zrevrange myzset 0 -1#降序配列所有的集合，如果不是-1就是根据下标查找该下标之间的降序排列

```

排行榜

## 4.三种特殊数据类型

### 4.1、geospatial

这个功能可以推算地理位置的信息，两地之间的距离，方圆几里的人。朋友的定位，附近的人。

![image-20210414233824794](Redis.assets/image-20210414233824794.png)

```bash
######################################################################################################################
#geoadd 用于存储指定的地理空间位置，可以将一个或多个经度(longitude)、纬度(latitude)、位置名称(member)添加到指定的 key 中
geoadd china:city 123.46987 41.80515 shenyang
##geopos 用于从给定的 key 里返回所有指定名称(member)的位置（经度和纬度），不存在的返回 nil。
geopos china:city shenyang
##geodist 用于返回两个给定位置之间的距离。GEODIST key member1 member2 [m|km|ft|mi]
#m ：米，默认单位。
#km ：千米。
#mi ：英里。
#ft ：英尺。
geodist china:city shenyang beijing km
#georadius 以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。
#WITHDIST: 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回
#WITHCOORD: 将位置元素的经度和维度也一并返回。
#COUNT 限定返回的记录数。
#ASC: 查找结果根据距离从近到远排序。DESC: 查找结果根据从远到近排序。
georadius china:city 110 30 2000 km

#georadiusbymember  以给定的位置为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。参数同上
GEORADIUSBYMEMBER china:city shenyang 2000 km

#Redis GEO 使用 geohash 来保存地理位置的坐标。geohash 用于获取一个或多个位置元素的 geohash 值
geohash china:city shenyang#结果：wxrvcdkh8k0
```

GEO底层实现原理就是zset。我们可以使用zset来操作GEO，

```bash
zrange china:city 0 -1 #查看上述地图的所有元素
zrem china:city shanghai#删除上海元素
```

### 4.2、Hyperloglog

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

**什么是基数：**比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

```bash
######################################################################################################################
#加指定元素到 HyperLogLog 中。
pfadd mykey1 123 2 43d f f g dc e trg de e r gd e
#返回给定 HyperLogLog 的基数估算值。
pfcount mykey1
#将多个 HyperLogLog 合并为一个 HyperLogLog
pfmerge mykey3 mykey1 MYKEY
```

### 4.3、Bitmaps

位存储

统计用户信息，活跃不活跃，登录未登录。只要是两个状态的都可以使用。

```bash
######################################################################################################################
#用于对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。
setbit sign 0 1
#命令用于对 key 所储存的字符串值，获取指定偏移量上的位(bit)。
getbit sign 5
#计算字符串中的设置位数（填充计数）。
bitcount sign
```

## 5.redis事务

### 事务理解

redis事务可以理解为一组命令的集合。事务支持一次执行多条命令，一个事务中的命令都会被序列化，会按照顺序串行执行命令，其他客户端的命令不会被插入。

redis事务就是一次性，顺序性，排他性的执行一个队列中的多个命令。

因为将每条命令放入事务中并不会被实际执行，只有当执行exec的时候才会一起执行。所以不存在隔离性的概念。

```bash
127.0.0.1:6379> multi  #开启事务
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2 
QUEUED
127.0.0.1:6379> get k1
QUEUED
127.0.0.1:6379> exec  #执行事务
1) OK
2) OK
3) "v1"
```

redis单条命令是原子性的。但是事务是不保证原子性的。如果一条命令执行失败其他的命令还是可以执行的。（如果是非法的命令是都不会执行的）

```bash
127.0.0.1:6379> set k4 v4
OK
127.0.0.1:6379> multi #开启事务
OK
127.0.0.1:6379> set k5 v5
QUEUED
#可以看到运行结果这条命令因为k4存放的是字符串不能进行增加而执行失败，但是没有影响其他命令的执行
127.0.0.1:6379> incr k4 
QUEUED
127.0.0.1:6379> set k6 v6
QUEUED
127.0.0.1:6379> exec #执行事务
1) OK
2) (error) ERR value is not an integer or out of range 
3) OK
######################################################################
#因为非法的命令整个事务的命令都执行失败
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> seta k5 v5
(error) ERR unknown command 'seta'
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
######################################################################
discard#放弃事务
```

### redis乐观锁

乐观锁：认为任何时候都不会有问题，所以不会加锁，只有在进行操作的时候去进行比较。

悲观锁：认为任何时候都可能会出现问题，所以无论做什么都加锁。

```bash
######################################################################
#使用watch来监视数据，watch只针对事务有效果
#连接1
set money 100
set out 0
watch money #监视money字段
multi #开启事务
decrby money 20
incrby out 20

#在执行事务前重新开启另一个连接对money做修改
#连接2
incrBy money 100

#回到连接1做exec操作
exec#可以看到执行失败

```

注意：只要是另一个连接对money进行了修改，不管修改后的值与原来一不一样连接1都会执行失败。在事务执行结束后无论成功还是失败watch都会失效。如果需要监视则重新使用watch。

## 6.jedis

### 使用Java操作redis

1.新建maven项目

2.导入依赖

```xml
<!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.5.2</version>
</dependency>
```

3.代码连接与操作

```java
public class JedisTest {
    public static void main(String[] args) {
        Jedis redisTest = new Jedis("127.0.0.1",6379);//直接连接ip与端口
        System.out.println(redisTest.ping());
        Set<String> keys = redisTest.keys("*");
        for (String key : keys) {
            System.out.println(key);
        }
    }
}
```

Java的操作命令与前面学习的命令都一样。

### jedis事务

```java
public static void main(String[] args) {
      Jedis redisTest = new Jedis("localhost",6379);
      System.out.println(redisTest.ping());
      JSONObject jsonObject = new JSONObject();
      jsonObject.put("name","aixin");
      jsonObject.put("age",18);
      String str = jsonObject.toString();
      Transaction multi = redisTest.multi();
      try {
          multi.set("user",str);
          multi.exec();
      } catch (Exception e) {
          multi.discard();
          e.printStackTrace();
      } finally {
          System.out.println(redisTest.get("user"));
          redisTest.close();
      }
}
```

## 7.springBoot整合redis

在springboot2.X之后，原来使用的jedis被替换为了lettuce

jedis：采用的直连，多个线程操作的话是不安全的。如果要避免不安全就使用redis poll连接池。更像BIO

lettuce：采用netty，实例可以在多个线程进行共享，不存在线程不安全的情况。可以减少线程数据。更像NIO

1.新建项目

2.选择好需要的环境

源码：

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
//查看连个类的源码可以看到使用了条件装配，默认将lettuce装配了进来
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {
    public RedisAutoConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean(name = {"redisTemplate"})//如果容器中没有则创建redisTemplate，也就是说可以自己创建redisTemplate
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
       //直接使用默认的redisTemplate，redistribution对象都是需要序列化的
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    //由于String是redis最常用的类型，所以单独提出了一个组件
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}
```

可以在application.properties里配置redis(spring.redis)

3.测试

```java
//opsForValue,opsForHash,opsForList,opsForSet.......就可以操作对应的数据类型
@Autowired
private RedisTemplate redisTemplate;
@Test
public void test(){
    redisTemplate.opsForValue().set("user","aixin");
    System.out.println(redisTemplate.opsForValue().get("user"));
}
```

在命令行上查询发现key变成了这样

```bash
127.0.0.1:6379> keys *
1) "\xac\xed\x00\x05t\x00\x04user"
```

原因是因为RedisTemplate对key默认使用的是JdkSerializationRedisSerializer序列化工具。

```java
if (this.defaultSerializer == null) {
            this.defaultSerializer = new JdkSerializationRedisSerializer(this.classLoader != null ? this.classLoader : this.getClass().getClassLoader());
}
```

使用StringRedisTemplate就会变的正常了

```java
@Autowired
private StringRedisTemplate stringRedisTemplate;

@Test
public void test(){
    stringRedisTemplate.opsForValue().set("userString","StringTemplate");
    System.out.println(stringRedisTemplate.opsForValue().get("userString"));
}
```

### 关于对象的操作

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private String name;
    private int age;
}
```

```java
@Test
public void userTest() throws JsonProcessingException {
    redisTemplate.opsForValue().set("user1",user);
    System.out.println(redisTemplate.opsForValue().get("user1"));
}
```

运行会报异常，因为User类没有实现序列化接口。将User实现序列化接口之后就可以运行了



### 自定义RedisTemplate

```java
@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
    RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
    redisTemplate.setConnectionFactory(redisConnectionFactory);
    Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
    objectMapper.activateDefaultTyping(LaissezFaireSubTypeValidator.instance ,
            ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
    objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
    jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
    StringRedisSerializer redisSerializer = new StringRedisSerializer();
    redisTemplate.setKeySerializer(redisSerializer);
    redisTemplate.setHashKeySerializer(redisSerializer);
    redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
    redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
    redisTemplate.afterPropertiesSet();
    return redisTemplate;
}
```

自定义RedisUtil

```java
package com.xin.utils;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;

import java.util.Collection;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

@Component
@Slf4j
public class RedisUtil {
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    // =============================1-common============================

    /**
     * 指定缓存失效时间
     *
     * @param key  键
     * @param time 时间(秒)
     * @return
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 根据key 获取过期时间
     *
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }

    /**
     * 判断key是否存在
     *
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 删除缓存
     *
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String... key) {
        if (key != null && key.length > 0) {
            if (key.length == 1) {
                redisTemplate.delete(key[0]);
            } else {
                redisTemplate.delete((Collection<String>) CollectionUtils.arrayToList(key));
            }
        }
    }

    // ============================2-String=============================

    /**
     * 普通缓存获取
     *
     * @param key 键
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }

    /**
     * 普通缓存放入
     *
     * @param key   键
     * @param value 值
     * @return true成功 false失败
     */
    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }

    }

    /**
     * 普通缓存放入并设置时间
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 递增 适用场景： https://blog.csdn.net/y_y_y_k_k_k_k/article/details/79218254 高并发生成订单号，秒杀类的业务逻辑等。。
     *
     * @param key 键
     * @paramby  要增加几(大于0)
     * @return
     */
    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }

    /**
     * 递减
     *
     * @param key 键
     * @paramby  要减少几(小于0)
     * @return
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }

    // ================================3-Map=================================

    /**
     * HashGet
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return 值
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(key, item);
    }

    /**
     * 获取hashKey对应的所有键值
     *
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * HashSet
     *
     * @param key 键
     * @param map 对应多个键值
     * @return true 成功 false 失败
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * HashSet 并设置时间
     *
     * @param key  键
     * @param map  对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @param time  时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 删除hash表中的值
     *
     * @param key  键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(key, item);
    }

    /**
     * 判断hash表中是否有该项的值
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(key, item);
    }

    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     *
     * @param key  键
     * @param item 项
     * @param by   要增加几(大于0)
     * @return
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, by);
    }

    /**
     * hash递减
     *
     * @param key  键
     * @param item 项
     * @param by   要减少记(小于0)
     * @return
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, -by);
    }

    // ============================4-set=============================

    /**
     * 根据key获取Set中的所有值
     *
     * @param key 键
     * @return
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            log.error(key, e);
            return null;
        }
    }

    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key   键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 将数据放入set缓存
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

    /**
     * 将set数据放入缓存
     *
     * @param key    键
     * @param time   时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0)
                expire(key, time);
            return count;
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

    /**
     * 获取set缓存的长度
     *
     * @param key 键
     * @return
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

    /**
     * 移除值为value的
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 移除的个数
     */
    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

    // ============================5-zset=============================

    /**
     * 根据key获取Set中的所有值
     *
     * @param key 键
     * @return
     */
    public Set<Object> zSGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            log.error(key, e);
            return null;
        }
    }

    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key   键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean zSHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    public Boolean zSSet(String key, Object value, double score) {
        try {
            return redisTemplate.opsForZSet().add(key, value, 2);
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 将set数据放入缓存
     *
     * @param key    键
     * @param time   时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long zSSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0)
                expire(key, time);
            return count;
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

    /**
     * 获取set缓存的长度
     *
     * @param key 键
     * @return
     */
    public long zSGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

    /**
     * 移除值为value的
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 移除的个数
     */
    public long zSetRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }
    // ===============================6-list=================================

    /**
     * 获取list缓存的内容
     *
     * @param key   键
     * @param start 开始 0 是第一个元素
     * @param end   结束 -1代表所有值
     * @return
     * @取出来的元素 总数 end-start+1
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            log.error(key, e);
            return null;
        }
    }

    /**
     * 获取list缓存的长度
     *
     * @param key 键
     * @return
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

    /**
     * 通过索引 获取list中的值
     *
     * @param key   键
     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     * @return
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            log.error(key, e);
            return null;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @paramtime  时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @paramtime  时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 根据索引修改list中的某条数据
     *
     * @param key   键
     * @param index 索引
     * @param value 值
     * @return
     */
    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 移除N个值为value
     *
     * @param key   键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */
    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }
}
```

## 8.配置文件详解



## 9.持久化

Redis是内存数据库,如果不将内存中的数据库状态保存到磁盘,那么一旦服务器进程退出,服务器中的数据库也会消失.所以redis提供了持久化功能.

### 9.1、RDB

![image-20210415215633536](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/redis/Redis.assets/image-20210415215633536-8535865.png)



触发机制:

1.配置文件中的save的规则,会自动触发rdb

2.执行flushall命令,也会触发。

3.退出redis,也会触发。

如何恢复rdb文件：只需要将rdb文件放在redis启动目录就可以

优点：

1.适合大规模的数据恢复。

2.对数据的完整性要求不高。

缺点：

1.需要一定的时间间隔进行操作。如果redis意外退出，最后一次修改的数据可能就会消失

2.因为需要一条子进程，需要占用一定内存空间。

### 9.2、AOF（Append Only File）

将我们所有的命令都记录下来，恢复的时候就把这个文件里所有的命令都再执行一遍。

AOF默认不开启，需要手动设置开启。

![image-20210415222828927](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/redis/Redis.assets/image-20210415222828927-8535865.png)

手动设置为yes，然后重启redis，就可以看到aof的文件了。

这个文件就记录了我们写操作的每一条命令。如果aof文件被破坏了，这时候redis启动不起来，redis提供了修复该文件的工具

redis-check-aof

![image-20210415223813573](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/redis/Redis.assets/image-20210415223813573-8535865.png)

可以看到在启动的时候会有提示该文件不正确，可以使用命令进行修复

![image-20210415224118408](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/redis/Redis.assets/image-20210415224118408-8535865.png)

K2就被容错掉了。只保留了正常的命令。

优点：

1.每一次修改都同步命令，文件的完整性更好

缺点：

数据文件相对于rdb文件大，修复的数据比rdb慢

aof运行效率也比rdb慢。

所以默认rdb

## 10.发布订阅

redis发布订阅是一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接收消息。

redis客户端可以订阅任意数量的频道。

![image-20210415225424888](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/redis/Redis.assets/image-20210415225424888-8535865.png)

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

![image-20210415225710006](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/redis/Redis.assets/image-20210415225710006-8535865.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![image-20210415225723307](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/redis/Redis.assets/image-20210415225723307-8535865.png)

```bash
subscribe ailalalala#订阅给定的一个或多个频道的信息。
publish ailalalala hello#将信息发送到指定的频道。只要这个命令执行了上面的命令就可以收到
```

## 11.主从复制

概念：

主从复制，是指将一台redis服务器的数据，复制到其他的redis服务器，前者称为主节点（master/leader），后者称为从节点（slave/follower），数据的赋值是单向的，只能从主节点复制到从节点。master以写为主，slave以读为主。

主从环境配置：

```bash
127.0.0.1:6379> info replication #查看当前库的信息
# Replication
role:master #角色为master
connected_slaves:0 #0个从机
master_replid:c928f45da29d39165ff2b5eee07a9a69e5e50cfe
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

复制多个config文件，然后修改对应配置

1.端口

2.pid名字==不知道为什么我的这个没有，然后后台默认也为no==

3.log文件名字

4.dump.rdb名字

修改之后分别启动三个服务。

![image-20210416000053758](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/redis/Redis.assets/image-20210416000053758-8535865.png)

![image-20210415235925096](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/redis/Redis.assets/image-20210415235925096-8535865.png)

一个从节点即可以当主节点也可以当从节点，但是这个服务还是不可以进行写操作

### 一主二从

默认每台服务都是主机，一般情况下只需要配置从机就可以。

```bash
#在从机上进行设置
slaveof 127.0.0.1 6379 #设置当前服务为127.0.0.1 端口为6379的服务为从机
info replication#可以看到当前的信息就变为从机了

```

真实的配置是在配置文件上配置，那就是永久有效了。

```properties
# Master-Slave replication. Use slaveof to make a Redis instance a copy of
# another Redis server. A few things to understand ASAP about Redis replication.
#
# 1) Redis replication is asynchronous, but you can configure a master to
#    stop accepting writes if it appears to be not connected with at least
#    a given number of slaves.
# 2) Redis slaves are able to perform a partial resynchronization with the
#    master if the replication link is lost for a relatively small amount of
#    time. You may want to configure the replication backlog size (see the next
#    sections of this file) with a sensible value depending on your needs.
# 3) Replication is automatic and does not need user intervention. After a
#    network partition slaves automatically try to reconnect to masters
#    and resynchronize with them.
#
slaveof <masterip> <masterport>

# If the master is password protected (using the "requirepass" configuration
# directive below) it is possible to tell the slave to authenticate before
# starting the replication synchronization process, otherwise the master will
# refuse the slave request.
#
masterauth <master-password>
```

主机可以写，从机只能读。主机中的所有信息和数据，都会被从机保存。

```bash
#在从机上写会报错
127.0.0.1:6381> set k2 v2
(error) READONLY You can't write against a read only replica.
```

如果主机关闭了，从机的状态还是不变的。主机如果回来了，从机依旧可以直接获取主机的信息。

如果从机断开了，从机重启后就会变为主机。如果再次设置为从机就会立马从主机中获取到数据。

**以上都是针对于使用命令修改主从机。**

复制原理：

![image-20210415235343529](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/redis/Redis.assets/image-20210415235343529-8535865.png)

**全量同步**
Redis全量复制一般发生在Slave初始化阶段，这时Slave需要将Master上的所有数据都复制一份。具体步骤如下： 
\- 从服务器连接主服务器，发送SYNC命令； 
\- 主服务器接收到SYNC命名后，开始执行BGSAVE命令生成RDB文件并使用缓冲区记录此后执行的所有写命令； 
\- 主服务器BGSAVE执行完后，向所有从服务器发送快照文件，并在发送期间继续记录被执行的写命令； 
\- 从服务器收到快照文件后丢弃所有旧数据，载入收到的快照； 
\- 主服务器快照发送完毕后开始向从服务器发送缓冲区中的写命令； 
\- 从服务器完成对快照的载入，开始接收命令请求，并执行来自主服务器缓冲区的写命令；

**增量同步**
Redis增量复制是指Slave初始化后开始正常工作时主服务器发生的写操作同步到从服务器的过程。 
增量复制的过程主要是主服务器每执行一个写命令就会向从服务器发送相同的写命令，从服务器接收并执行收到的写命令。



如果主机断开了连接，我们可以使用slaveof no one让执行这条命令的服务变为主机。





### 哨兵模式（Sentinel）

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是**哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。**

<img src="/Users/aixin/Downloads/Redis.assets/image-20210416000856522.png" alt="image-20210416000856522" style="zoom:67%;" />



这里的哨兵有两个作用

- 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。
- 当哨兵监测到master宕机，会自动将slave切换成master，然后通过**发布订阅模式**通知其他的从服务器，修改配置文件，让它们切换主机。

然而一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行监控，这样就形成了多哨兵模式。

![image-20210416001015247](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/redis/Redis.assets/image-20210416001015247-8535865.png)

用文字描述一下**故障切换（failover）**的过程。假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover过程，仅仅是哨兵1主观的认为主服务器不可用，这个现象成为**主观下线**。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行failover操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为**客观下线**。这样对于客户端而言，一切都是透明的。



测试：

一主二从

增加一个哨兵的配置文件

```properties
# 禁止保护模式
protected-mode no
# 配置监听的主服务器，这里sentinel monitor代表监控，mymaster代表服务器的名称，可以自定义，192.168.11.128代表监控的主服务器，6379代表端口，2代表只有两个或两个以上的哨兵认为主服务器不可用的时候，才会进行failover操作。
#如果配置1个哨兵后面的数字就为1
sentinel monitor mymaster 192.168.11.128 6379 2
# sentinel author-pass定义服务的密码，mymaster是服务名称，123456是Redis服务器密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster 123456
```

启动哨兵进程

```bash
# 启动哨兵进程
./redis-sentinel ../sentinel.conf
```

如果主机shutdown，那么哨兵线程会在一段时间内选择一个从机变更为主机。如果主机回来了会变为新主机的从机。

哨兵模式优缺点：

优点：

1.哨兵集群，基于主从复制模式，所有的主从配置优点他都有

2.主从可以切换，故障可以转移，系统的可用性更好

3.哨兵模式就是主从模式的升级，手动到自动，更加健壮

缺点：

1. 部署相对 Redis 主从模式要复杂一些，原理理解更繁琐。
2. 资源浪费，Redis 数据节点中 slave 节点作为备份节点不提供服务。
3. Redis Sentinel 主要是针对 Redis 数据节点中的主节点的高可用切换，对 Redis 的数据节点做失败判定分为主观下线和客观下线两种，对于 Redis 的从节点有对节点做主观下线操作，并不执行故障转移。
4. 不能解决读写分离问题，实现起来相对复杂

























