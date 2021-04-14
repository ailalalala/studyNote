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



















































