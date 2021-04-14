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

## 3.redis5大数据类型

### 3.1、String

String是redis最基本的数据类型，是二进制安全的，可以包含任何数据，包括图片或序列化的对象。

### 3.2、Hash

hash，类似Java里的map。

redis里的hash是一个String类型的field和value的映射表。适合存放对象。

### 3.3、List

redis的list是一个简单的String列表，按照插入的顺序，可以将插入的数据放在列表的头部或尾部。

list的底层是一个链表。

### 3.4、set

redis的set是一个String类型的无序的集合，通过HashTable来实现的。

### 3.5、ZSet

redis的Zset是一个String类型的不可重复的有序集合。不同的是每一个元素都会关联一个double类型的分数。就是通过这个分数来为集合进行排序，double可重复。

## 4.操作Key

```mysql
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
##查看name的key是否村子啊
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















