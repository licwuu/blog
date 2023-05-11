---
title: redis学习
tags:
  - redis
  - nosql
categories:
  - 学习笔记
cover: /image/redis.svg
abbrlink: 7b25d017
date: 2022-10-13 20:48:01
---

## Redis简介

NoSQL(NoSQL = Not Only SQL )，意即“不仅仅是SQL”，泛指非关系型的数据库。NoSQL 不依赖业务逻辑方式存储，而以简单的key-value模式存储。因此大大的增加了数据库的扩展能力。Redis就是典型的、也是常用的NoSql之一。

**特点：**

+ 不遵循SQL标准。
+ 不支持ACID。
+ 远超于SQL的性能。

**适用场景：**

+ 对数据高并发的读写
+ 海量数据的读写
+ 对数据高可扩展性的
+ **用不着sql的和用了sql也不行的情况，请考虑用NoSql**

**不适用场景：**

+ 需要事务支持
+ 基于sql的结构化查询存储，处理复杂的关系,需要即席查询

## 安装

一般Redis都是部署在Linux环境中,因此不考虑在windows中部署。

1. 准备C语言环境

    ```bash
    yum install centos-release-scl scl-utils-build
    yum install -y devtoolset-8-toolchain
    scl enable devtoolset-8 bash
    # 测试GCC
    gcc --version
    ```

    ![测试GCC](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221013210915.png)

2. 去[Redis官网](http://redis.io)下载对应的安装包，一般Redis都是部署在Linux环境中，即下载for Linux(redis-xxx.tar.gz），放/opt目录
3. 解压安装

    ```bash
    tar -zxvf redis-6.2.1.tar.gz
    cd redis-6.2.1
    make
    make install
    ```

    > 如果没有准备好C语言编译环境，make 会报错(Jemalloc/jemalloc.h：没有那个文件),只需要执行`make distclean`后重新执行`make`命令即可。

**默认安装目录：**/usr/local/bin
查看默认安装目录常用命令：

+ redis-benchmark:性能测试工具，可以在自己本子运行，看看自己本子性能如何
+ redis-check-aof：修复有问题的AOF文件，rdb和aof后面讲
+ redis-check-dump：修复有问题的dump.rdb文件
+ redis-sentinel：Redis集群使用
+ redis-server：Redis服务器启动命令
+ redis-cli：客户端，操作入口

## 启动

### 前台启动

在安装目录下运行`redis-srver`即可。

### 后台启动

1. 编写配置文件

    拷贝一份redis.conf到其他目录:`cp  /opt/redis-3.2.5/redis.conf  /myredis`,更改配置文件daemonize no改成yes。

2. 通过配置文件启动，执行`redis-server /myredis/redis.conf`

### 客户端连接

```bash
redis-cli -p 6379 # -p 6379可省略
```

### redis关闭

1. redis-cli连接后使用shutdown命令关闭，或者直接redis-cli -p 6379 shutdown
2. kill -9 进程号

## [Redis命令网站1](http://redisdoc.com/index.html)、[Redis命令网站2](http://doc.redisfans.com/)

## Redis数据类型

### 0. Redis 键（key）

  ```bash
  keys * # 查看当前库所有key (匹配：keys *1)
  exists key # 判断某个key是否存在
  type key # 查看你的key对应的值是什么类型
  del key # 删除指定的key数据
  unlink key # 根据value选择非阻塞删除，仅将keys从keyspace元数据中删除，真正的删除会在后续异步操作。
  expire key 10 # 10秒钟：为给定的key设置过期时间
  ttl key # 查看还有多少秒过期，-1表示永不过期，-2表示已过期

  select n # 命令切换数据库n
  dbsize # 查看当前数据库的key的数量
  flushdb # 清空当前库
  flushall # 清空全部库
  ```

### 1. 字符串（String）

#### String简介

String是Redis最基本的类型，一个key对应一个value，类似于Map<String,Object>。String类型是二进制安全的,意味着Redis的string可以包含任何数据。比如jpg图片或者序列化的对象。String类型是Redis最基本的数据类型，一个Redis中字符串value**最多可以是512M**。

#### String常用命令

1. `set key value [EX seconds|PX milliseconds |KEEPTTL] [NX|XX]` 添加键值对
  
    NX：当数据库中key不存在时，可以将key-value添加数据库

    XX：当数据库中key存在时，可以将key-value添加数据库，与NX参数互斥

    EX：key的超时秒数

    PX：key的超时毫秒数，与EX互斥

2. `get  key` 查询对应键值
3. `append key` value 将给定的value  追加到原值的末尾
4. `strlen key` 获得值的长度
5. `setnx key value` 只有在 key 不存在时才能设置key的值，后面可以利用这个特性将其作为分布式锁
6. `incr key` 将 key 中储存的数字值增1，只能对数字值操作，如果为空，新增值为1
7. `decr key` 将 key 中储存的数字值减，只能对数字值操作，如果为空，新增值为-1
8. `incrby/decrby key n` 将 key 中储存的数字值增减n(自定义步长)

    > redis中的incr/decr 不同于java中的i++，它是原子操作。

9. `mset key1 value1 key2 value2 .....` 同时设置一个或多个 key-value队
10. `mget key1 key2 key3 .....` 同时获取一个或多个 value  
11. `msetnx key1 value1 key2 value2  .....` 同时设置一个或多个 key-value 队，当且仅当所有给定 key 都不存在才成功,该操作具有原子性, 有一个失败则都失败
12. `getrange key start end` 命令用于获取存储在指定 key 中字符串的子字符串。字符串的截取范围由 start 和 end 两个偏移量决定(包括 start 和 end 在内)
13. `setrange  key start value` 指定的字符串覆盖给定 key 所储存的字符串值，覆盖的位置从偏移量 start 开始
14. `setex  key 过期时间 value` 设置键值的同时，设置过期时间，单位秒。
15. `getset key value` 以新换旧，设置了新值同时获得旧值。

#### String数据结构

String的数据结构为简单动态字符串(Simple Dynamic String,缩写SDS)。是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配.
![String数据结构](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221013221244.png)
如图中所示，内部为当前字符串实际分配的空间capacity一般要高于实际字符串长度len。当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。需要注意的是字符串最大长度为512M。

### 2. 列表(List)

#### List简介

Redis 列表是简单的字符串列表(单键多值)，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。
它的底层实际是个双向快速链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。

#### List常用命令

1. `lpush/rpush <key> <value1> <value2> <value3> ....` 从左边/右边插入一个或多个值。
2. `lpop/rpop <key>` 从左边/右边吐出一个值。值在键在，值光键亡。
3. `rpoplpush <key1> <key2>` 从key1列表右边吐出一个值，插到key2列表左边。
4. `lrange <key> <start> <stop>` 按照索引下标获得元素(从左到右)，负数从右到左计数
5. `lindex <key><index>` 按照索引下标获得元素(从左到右)
6. `llen <key>` 获得列表长度
7. `linsert <key> before <value> <newvalue>` 在value的前面插入newvalue
8. `lrem <key> <n> <value>` 从左边删除n个value(从左到右)
9. `lset <key> <index> <value>` 将列表key下标为index的值替换成value

#### List数据结构

List的数据结构为快速链表quickList。首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，也即是压缩列表。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。当数据量比较多的时候才会改成quicklist。因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。Redis将链表和ziplist结合起来组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

![redis列表数据结构](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221013224949.png)

### 3. 集合(Set)

#### Set简介

Redis set对外提供的功能与list类似是一个列表的功能，特殊之处在于set是可以自动**排重**的，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。

Redis的Set是string类型的无序集合。它底层其实是一个value为null的hash表，所以添加，删除，查找的**复杂度都是O(1)**。

**常用命令参考：**[Redis Set 命令](https://redis.cwnest.top/set/index.html)

#### Set数据结构

Set数据结构是dict字典，字典是用**哈希表**实现的。

Java中HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。

### 4. 哈希(Hash)

#### Hash简介

Redis hash 是一个键值对集合。

Redis hash的key是一个string类型，值是一个field和value的映射表，它特别适合用于存储对象(类似Java里面的Map<String,Object>)。比如：用户ID为查找的key，存储的value用户对象包含姓名，年龄，生日等信息，有以下的存储方式：

1. 每次修改用户的某个属性需要，先反序列化改好后再序列化回去，**开销较大**。

    ![序列化方式](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221014102051.png)

2. 通过用户ID加属性标签构造不同key,**用户ID数据冗余**。

    ![构造不同key](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221014102237.png)

3. 通过 key(用户ID) + field(属性标签) 就可以操作对应属性数据了，既不需要重复存储数据，也不会带来序列化和并发修改控制的问题

    ![hash结构存储](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221014102309.png)

**常用命令参考：**[Redis Hash 命令](https://redis.cwnest.top/hash/index.html)

#### Hash数据结构

Hash类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。

### 5. 有序集合Zset(sorted set)

#### Zset 简介

Redis有序集合zset为每个成员都关联了一个评分（score）,这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是**唯一**的，但是**评分可以是重复**的 。

因为元素是有序的, 所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。访问有序集合的中间元素也是非常快的,因此你能够使用有序集合作为一个没有重复成员的智能列表。

**常用命令参考：**[Redis Zset 命令](https://redis.cwnest.top/sorted_set/index.html)

#### Zset数据结构

SortedSet(zset)是Redis提供的一个非常特别的数据结构，一方面它等价于Java的数据结构Map<String, Double>，可以给每一个元素value赋予一个权重score，另一方面它又类似于TreeSet，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。

zset底层使用了两个数据结构：

1. **Hash：**hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。
2. **跳跃表：**跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。

#### 什么是跳跃表

有序集合在生活中比较常见，例如根据成绩对学生排名，根据得分对玩家排名等。对于有序集合的底层实现，可以用数组、平衡树、链表等。数组不便元素的插入、删除；平衡树或红黑树虽然效率高但结构复杂；链表查询需要遍历所有效率低。Redis采用的是跳跃表。跳跃表效率堪比红黑树，实现远比红黑树简单。

**实例：**对比有序链表和跳跃表，从链表中查询出51

1. 有序链表

    ![有序链表](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221014103750.png)

    要查找值为51的元素，需要从第一个元素开始依次查找、比较才能找到。共需要6次比较。

2. 跳跃表

    ![跳跃表](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221014103839.png)

    从第2层开始，1节点比51节点小，向后比较。

    21节点比51节点小，继续向后比较，后面就是NULL了，所以从21节点向下到第1层

    在第1层，41节点比51节点小，继续向后，61节点比51节点大，所以从41向下

    在第0层，51节点为要查找的节点，节点被找到，共查找4次。

从此可以看出跳跃表比有序链表效率要高。

### Bitmaps

#### Bitmaps简介

Redis提供了Bitmaps这个“数据类型”可以实现对位的操作,合理地使用操作位能够有效地提高内存使用率和开发效率。Bitmaps本身不是一种数据类型，实际上它就是字符串（key-value），但是它可以对字符串的位进行操作。Bitmaps单独提供了一套命令，所以在Redis中使用Bitmaps和使用字符串的方法不太相同，可以把Bitmaps想象成一个以位为单位的数组，数组的每个单元只能存储0和1，数组的下标在Bitmaps中叫做偏移量。

#### 使用样例

每个独立用户是否访问过网站存放在Bitmaps中， 将访问的用户记做1， 没有访问的用户记做0， 用偏移量作为用户的id。设置键的第offset个位的值（从0算起） ， 假设现在有20个用户，userid=1， 6， 11， 15， 19的用户对网站进行了访问， 那么当前Bitmaps初始化结果如图：
![20221015141156](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221015141156.png)
bitmap还可以进行复合操作， 它可以做多个Bitmaps的and（交集） 、 or（并集） 、 not（非） 、 xor（异或） 操作并将结果保存在目标key中。

注：很多应用的用户id以一个指定数字（例如10000） 开头，直接将用户id和Bitmaps的偏移量对应势必会造成一定的浪费，通常的做法是每次做setbit操作时将用户id减去这个指定数字。在第一次初始化Bitmaps时，假如偏移量非常大，那么整个初始化过程执行会比较慢， 可能会造成Redis的阻塞。

#### Bitmaps与set对比

假设网站有1亿用户， 每天独立访问的用户有5千万， 如果每天用集合类型和Bitmaps分别存储活跃用户可以得到表:
|数据类型| 每个用户id占用空间| 需要存储的用户量| 全部内存量|
|:---:|:---:|:---:|:---:|
|集合类型| 64位| 50000000| 64位*50000000 = 400MB|
|Bitmaps |1位 |100000000 |1位*100000000 = 12.5MB|

很明显， 这种情况下使用Bitmaps能节省很多的内存空间， 尤其是随着时间推移节省的内存还是非常可观的:
|数据类型 |一天 |一个月 |一年|
|:---:|:---:|:---:|
|集合类型 |400MB| 12GB |144GB|
|Bitmaps |12.5MB |375MB |4.5GB|

但Bitmaps并不是万金油，假如该网站每天的独立访问用户很少，例如只有10万（大量的僵尸用户），那么两者的对比如下表所示，很显然，这时候使用Bitmaps就不太合适了，因为基本上大部分位都是0
|数据类型 |每个userid占用空间| 需要存储的用户量| 全部内存量|
|:---:|:---:|:---:|:---:|
|集合类型 |64位| 100000| 64位*100000 = 800KB|
|Bitmaps |1位| 100000000| 1位*100000000 = 12.5MB|

### HyperLogLog

在工作当中，我们经常会遇到与统计相关的功能需求，比如统计网站PV（PageView页面访问量）,可以使用Redis的incr、incrby轻松实现。但像UV（UniqueVisitor，独立访客）、独立IP数、搜索记录数等需要去重和计数的问题如何解决？这种求集合中不重复元素个数的问题称为基数问题。

解决基数问题有很多种方案：

1. 数据存储在MySQL表中，使用distinct count计算不重复个数
2. 使用Redis提供的hash、set、bitmaps等数据结构来处理

以上的方案结果精确，但随着数据不断增加，导致占用空间越来越大，对于非常大的数据集是不切实际的。

能否能够降低一定的精度来平衡存储空间？Redis推出了HyperLogLog

Redis HyperLogLog 是用来做基数统计的算法。HyperLogLog在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。在Redis里面，每个 HyperLogLog键只需要花费12KB内存，就可以计算接近 2^64 个不同元素的基数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为HyperLogLog只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog不能像集合那样，返回输入的各个元素。

> 什么是基数?
> 比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

### Geospatial

Redis 3.2 中增加了对GEO类型的支持。GEO，Geographic，地理信息的缩写。该类型，就是元素的2维坐标，在地图上就是经纬度。redis基于该类型，提供了经纬度设置，查询，范围查询，距离查询，经纬度Hash等常见操作。如：

```bash
# 添加城市经纬度
geoadd china:city 106.50 29.53 chongqing 114.05 22.52 shenzhen 116.38 39.90 beijing 
# 计算城市距离geodist<key><member1><member2>  [m|km|ft|mi ]
geodist china:city chongqing shenzhen  km
```

## Redis配置文件常用配置介绍

**requirepass**：设置密码（！！！）

**Unit单元**：配置大小单位,开头定义了一些基本的度量单位，只支持bytes，不支持bit，大小写不敏感

**INCLUDES包含**：多实例的情况可以把公用的配置文件提取出来，通过`include /对应路径`应用过来即可

**bind**：绑定接受谁的访问请求。默认配置是`bind=127.0.0.1`只接受本机请求，如果不设定且`protected-mode no`，则是无限制接受任何ip地址的访问

**protected-mode**：如果开启了protected-mode，那么在没有设定bind ip且没有设密码的情况下，Redis只允许接受本机的响应，默认开启

**port**：启动端口号，默认6379

**tcp-backlog**：设置tcp的backlog，backlog其实是一个连接队列，backlog队列总和=未完成三次握手队列 + 已经完成三次握手队列。

> 在高并发环境下你需要一个高backlog值来避免慢客户端连接问题。注意Linux内核会将这个值减小到/proc/sys/net/core/somaxconn的值（128），所以需要确认增大/proc/sys/net/core/somaxconn和/proc/sys/net/ipv4/tcp_max_syn_backlog（128）两个值来达到想要的效果

**timeout**：设定一个空闲的客户端维持多少秒会关闭，0表示关闭该功能，即永不关闭

**tcp-keepalive n**: 对访问客户端的一种心跳检测，每n秒检测一次。

**daemonzie**: 是否后台启动，建议yes

**pidfile**：设置存放pid文件的位置，每个实例会产生一个不同的pid文件

**loglevel**：设置日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为notice

**logfile**：设置日志文件名称

**databases**：设定库的数量，默认16

**maxclients**：设置redis同时可以与多少个客户端进行连接，默认情况下为10000个客户端，如果达到了此限制，redis则会拒绝新的连接请求，并且向这些连接请求方发出“max number of clients reached”以作回应

**maxmemory**：设置redis可以使用的内存量。一旦到达内存使用上限，redis将会试图移除内部数据，移除规则可以通过maxmemory-policy来指定

> 建议必须设置，否则，将内存占满，造成服务器宕机
> 如果redis无法根据移除规则来移除内存中的数据，或者设置了“不允许移除”，那么redis则会针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等
> 但是对于无内存申请的指令，仍然会正常响应，比如GET等。如果你的redis是主redis（说明你的redis有从redis），那么在设置内存使用上限时，需要在系统中留出一些内存空间给同步队列缓存，只有在你设置的是“不移除”的情况下，才不用考虑这个因素。

**maxmemory-policy**:指定maxmemory的移除规则，有以下几种：

1. volatile-lru：使用LRU算法移除key，只对设置了过期时间的键；（最近最少使用）
2. allkeys-lru：在所有集合key中，使用LRU算法移除key
3. volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键
4. allkeys-random：在所有集合key中，移除随机的key
5. volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key
6. noeviction(默认)：不进行移除。针对写操作，只是返回错误信息

## Redis的发布和订阅

### 什么是发布订阅

Redis 发布订阅 (pub/sub) 是一种消息通信模式：发送者 (pub) 发送消息，订阅者 (sub) 接收消息(观察者模式)。

Redis客户端可以订阅频道如下图右侧(可以订阅任意数量的频道),当发布者给这个频道发布消息后，消息就会发送给订阅的客户端

![发布订阅样例](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221014113540.png)

### 命令行实现

1. 打开一个客户端订阅channel1：`SUBSCRIBE channel1`
2. 打开另一个客户端，给channel1发布消息hello：`publish channel1 hello`,返回的1是订阅者数量
3. 在第一个客户端可以看到发送的消息

![发布订阅实现样例](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221014114349.png)

> 注：发布的消息没有持久化，如果在订阅的客户端没有收到hello，那就只能收到之后发布的消息

## redis事务

### 事务定义

Redis事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断，其主要作用就是串联多个命令防止别的命令插队。

redis中事务和mysql中事务不太像一个概念；redis中事务通过组队完成多个命令的绑定操作，是为了防止别的命令插队，不会回滚。

### 事务命令(Multi、Exec、discard)

从输入Multi命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入Exec后，Redis会将之前的命令队列中的命令依次执行,组队的过程中可以通过discard来放弃组队。
![事务组队过程](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221014150423.png)

### 事务错误处理

1. 在组队过程中出现错误（如语法错误）会立即提示，不修改命令直接使用exec执行，所有组队的命令都无法执行;

2. 在组队阶段成功，在执行阶段发生错误，只有错误命令不执行，其他命令照常执行，**不会回滚**;

### 事务冲突

样例：有多个人拥有你的账户，同时对其操作就会产生事务冲突。

![image-20221010195207412](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/image-20221010195207412.png)

解决方案：

**悲观锁**：假想每次拿数据后原有数据都会被修改，所以，每次拿数据都给数据上锁，使得别人无法操作该数据。

![悲观锁](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221014162312.png)

> tips:传统关系型数据库的行锁、表锁等都是这种方式。但是这种方式效率很慢。

**乐观锁：**给数据加上一个标识，如版本号等，拿到数据进行操作后，查看现在数据和之前拿到的版本号是否一致；不一致则说明有人操作过数据，需要重新读取数据再进行操作；一致则说明操作期间没人动过数据，直接更新数据和更改版本号即可。**乐观锁适用于多读的应用类型，这样可以提高吞吐量**。Redis就是利用这种check-and-set机制实现事务的。

![乐观锁](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221014162329.png)

**redis 中通过watch命令监听key实现乐观锁:**WATCH key [key ...] ，unwatch命令可以取消监听。

在执行multi之前，先执行watch key1 [key2],可以监视一个(或多个) key ，如果在事务**执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。**

### 事务三大特性

1. **单独的隔离操作**：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断
2. **没有隔离级别的概念**:队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行
3. **不保证原子性**:事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚

## Redis持久化

### 1. RDB(redis database)

#### 什么是RDB

RDB是指在指定的时间间隔内将内存中的数据集快照写入磁盘——快照（Snapshot），它恢复时是将快照文件直接读到内存里，redis会**默认开启rdb**，并将备份文件保存在./dump.rdb中

#### RDB执行流程

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到 一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能 如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。**RDB的缺点是最后一次持久化后的数据可能丢失**。

![RDB执行流程](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221014170805.png)

> Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等） 数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程
>
> 在Linux程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，Linux中引入了“**写时复制技术**”
>
> **一般情况父进程和子进程会共用同一段物理内存**，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。

#### save && bgsave

RDB采用写时复制技术在指定时间间隔将redis数据同步到磁盘中，有save和bgsave两种配置，推荐采用bgsave操作。因为save操作会阻塞，而bgsave是异步的。

格式：`save 秒钟 写操作次数`

如：save 10 1000 则表示10s内有1000次写操作，就触发备份。

#### RDB备份与恢复

备份：先通过config get dir 查询rdb文件的目录 ，将*.rdb的文件拷贝到别的地方即可。

恢复：先把备份的文件拷贝到工作目录下 `cp 备份文件路径 dump.rdb`,再启动Redis, 备份数据会直接加载。

#### RDB优势

+ 适合大规模的数据恢复
+ 对数据完整性和一致性要求不高更适合使用
+ 节省磁盘空间
+ 恢复速度快

#### RDB劣势

+ Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑
+ 虽然Redis在fork时使用了**写时拷贝技术**,但是如果数据庞大时还是比较消耗性能
+ 在备份周期在一定间隔时间做一次备份，所以如果Redis意外down掉的话，就会丢失最后一次快照之后的所有修改。

### 2. AOF(append only file)

#### 什么是AOF

AOF以**日志**的形式来记录每个写操作（增量保存），将Redis执行过的所有写指令记录下来(**读操作不记录**)， **只许追加文件但不可以改写文件**，redis启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。AOF**默认关闭，如果开启默认保存在./appendonly.aof**中。

#### 执行流程

1. 客户端的请求写命令会被append追加到AOF缓冲区内；
2. AOF缓冲区根据AOF持久化策略[always,everysec,no]将操作sync同步到磁盘的AOF文件中；
3. AOF文件大小超过重写策略或手动重写时，会对AOF文件rewrite重写，压缩AOF文件容量；
4. Redis服务重启时，会重新load加载AOF文件中的写操作达到数据恢复的目的；

![AOF持久化流程](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221014172420.png)

> **AOF重写操作**：当aof文件过大，达到设定阈值，redis会对其进行重写压缩操作，将前面的指令转化为RDB文件的格式以减少磁盘占用，并记录转化后文件大小，下次在超过当前大小100%时再次触发重写操作。比如文件大小达到64M时触发了重写操作，压缩到了60M，下次在达到120M时再次触发。

#### 启动、备份、和恢复（异常恢复）

启动：修改默认的appendonly no，改为yes

备份：同RDB一样，都是拷贝备份文件

恢复：将备份文件拷贝到Redis工作目录下，启动系统即加载

异常恢复：如遇到**AOF文件损坏**，可备份文件再通过/usr/local/bin/**redis-check-aof--fix appendonly.aof**进行恢复文件。重启redis，然后重新加载

#### AOF同步频率设置

`appendfsync always`：始终同步，每次Redis的写入都会立刻记入日志；性能较差但数据完整性比较好

`appendfsync everysec`：每秒同步，每秒记入日志一次，如果宕机，本秒的数据可能丢失。

`appendfsync no`：redis不主动进行同步，把同步时机交给操作系统。

#### 优势

+ 备份机制更稳健，丢失数据概率更低。
+ 可读的日志文本，通过操作AOF稳健，可以处理误操作。

#### 劣势

+ 比起RDB占用更多的磁盘空间。
+ 恢复备份速度要慢。
+ 每次读写都同步的话，有一定的性能压力。
+ 存在个别Bug，造成恢复不能。

### 怎么选择？

+ 官方建议两个都启用
+ 如果对数据不敏感，可以单独RDB
+ 如果只做纯内存缓存可以都不使用
+ 不建议单独AOF，可能有Bug

> 如果AOF和RDB同时开启，系统使用谁恢复数据？
>
> 默认会使用AOF恢复数据。因为AOF数据还原度高吧，RDB会丢失最后一次备份之后操作的数据。

#### 官方建议

+ RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储
+ AOF持久化方式记录每次对服务器写的操作,当服务器重启的时候会重新执行这些命令来恢复原始的数据,AOF命令以redis协议追加保存每次写的操作到文件末尾
+ Redis还能对AOF文件进行后台重写,使得AOF文件的体积不至于过大
+ 只做缓存：如果你只希望你的数据在服务器运行的时候存在,你也可以不使用任何持久化方式.
+ 同时开启两种持久化方式
+ 在这种情况下,当redis重启的时候会优先载入AOF文件来恢复原始的数据, 因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整.
+ RDB的数据不实时，同时使用两者时服务器重启也只会找AOF文件。那要不要只使用AOF呢？
+ 建议不要，因为RDB更适合用于备份数据库(AOF在不断变化不好备份)， 快速重启，而且不会有AOF可能潜在的bug，留着作为一个万一的手段。
+ 性能建议

> 因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留`save 900 1`这条规则。
> 如果使用AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了。
> 代价,一是带来了持续的IO，二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。
> 只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上。
> 默认超过原大小100%大小时重写可以改到适当的数值。

## Redis主从复制

### 是什么

主机数据更新后根据配置和策略，自动同步到备机的master/slaver机制，master主写，slaver主读，一般**一主多从** 。

![image-20221011094914723](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/image-20221011094914723.png)

**优势**：读写分离，性能扩展、容灾快速恢复

### 单机搭建一主两从

在redis中更改以下配置,作为公共配置：

```bash
Appendonly no
daemonize yes
```

编写多个不同配置文件（其他两个把6379改为别的端口，如6380、6381），写入以下配置

```bash
include /myredis/redis.conf
pidfile /var/run/redis_6379.pid
port 6379
dbfilename dump6379.rdb
```

启动三台redis服务器,可以通过`redis-cli -p 端口号`分别登录三台服务器，使用`info replication`查看相关信息

```bash
redis-server redis6379.conf
redis-server redis6380.conf
redis-server redis6381.conf
```

配置主从关系：**配从不配主**，连接要作为从机的服务器，输入`slaveof <主机ip><主机port>`，主从关系建立。

### 常用主从搭建

#### 一主两从

![image-20221011142451587](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/image-20221011142451587.png)

+ **从服务器挂掉**之后再次重启，并不会直接作为之前主服务器的从服务器，而是自己成为主服务器，独立出来。 要想重新加入之前的主从复制，需要重新执行slaveof 命令加入。
+ **主服务器挂掉**之后，从服务器还是作为从服务器，原地待命，等主服务器重启之后，仍然保持原有主从关系。

#### 薪火相传

![image-20221011142546642](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/image-20221011142546642.png)

主从服务器挂掉之后关系转变和一主二从一样，风险是一旦某个slave宕机，该slave后面跟的slave都没法备份

#### 反客为主

当一个master宕机后，可以用slaveof no one命令（得手动操作，要自动完成得加入哨兵）将后面的从机变成主机 , 其后面的slave不用做任何修改。

### 主从复制原理

1. Slave启动成功连接到master后会发送一个sync命令
2. Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令， 在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步
3. 全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
4. 增量复制：Master继续将新的所有收集到的修改命令依次传给slave,完成同步
5. 但是只要是重新连接master,一次完全同步（全量复制)将被自动执行

> 全量复制之前由salve主动发起而之后则是由master发起，每次为增量复制。

## 哨兵模式(自动版反客为主)

以上面的一主两从（主：6379，从：6380、6381）例子为基础

1. 在自定义的/myredis目录下新建sentinel.conf文件

2. 配置哨兵，vim sentinel.conf

   ```bash
   sentinemonitor mymaster 127.0.0.1 6379 1
   # 其中 mymaster 为监控对象起的名称，1表示从服务器变成主服务器至少需要有1个哨兵同意。
   ```

3. 启动哨兵:`redis-sentinel sentinel.conf`

4. 加入哨兵后，当主服务器挂掉后，会在从机中选举出新的主服务器，而当原来主服务器恢复后，成为新主服务器的从服务器。

5. 新主机选举规则

   + 选择优先级靠前的：优先级在redis.conf中配置，默认100；
   + 选择偏移量最大的：偏移量指过的原主机数据最全的，也就是最近同步的；
   + 选择runid最小的：每个redis 实例启动后都会随机生成一个40位的runid

## Redis集群

### 什么是集群

Redis 集群实现了对Redis的水平扩容，即启动N个redis节点，将整个数据库分布存储在这N个节点中，每个节点存储总数据的1/N。

Redis 集群通过分区（partition）来提供一定程度的可用性（availability）： 即使集群中有一部分节点失效或者无法进行通讯， 集群也可以继续处理命令请求。

### 集群解决的问题

容量不够，redis如何进行扩容？

并发写操作， redis如何分摊？

主从模式，薪火相传模式，主机宕机，导致ip地址发生变化，应用程序中配置需要修改对应的主机地址、端口等信息。

Redis3.0中为此提供了解决方案——无中心化集群配置，在此之前可以通过代理主机来解决。

### 如何构建集群

在之前的配置中加入或修改一下集群相关配置,创建好对应数量的配置文件（本例中配置最简单的6个redis服务构成一个集群）

```bash
cluster-enabled yes # 打开集群模式
cluster-config-file nodes-6379.conf # 设定节点配置文件名
cluster-node-timeout 15000 # 设定节点失联时间，超过该时间（毫秒），集群自动进行主从切换。
```

> vim可使用%s/6379/6380将所有6379替换为6380

启动服务,此时会在目录下生成nodes-port.conf文件（不是之前的配置文件，你要搞错了），通过ll命令查看是否生成。

```bash
redis -server redis6379.conf
redis -server redis6380.conf
redis -server redis6381.conf
redis -server redis6389.conf
redis -server redis6390.conf
redis -server redis6391.conf
```

合体成集群

```bash
cd  /opt/redis-6.2.1/src
redis-cli --cluster create --cluster-replicas 1 192.168.11.101:6379 192.168.11.101:6380 192.168.11.101:6381 192.168.11.101:6389 192.168.11.101:6390 192.168.11.101:6391
# --replicas 1 采用最简单的方式配置集群，一台主机，一台从机，正好三组。
```

#### 登录

普通方式登录：`redis-cli -p 6379`,操作时会报错MOVED...

应该采用集群方式登录,`redis-cli  -c -p 6379`, -c 采用集群策略连接，设置数据会自动切换到相应的写主机,-p后面端口可填集群中任意端口。登陆后可用`cluster nodes`查看集群信息。

#### redis cluster 如何分配这六个节点?

一个集群至少要有**三个主节**点，选项 --cluster-replicas 1 表示我们希望为集群中的每个主节点创建一个从节点。分配原则**尽量保证每个主数据库运行在不同的IP地址，每个从库和主库不在一个IP地址上。**

### slots（插槽）

一个 Redis 集群包含 16384 个插槽（hash slot）， 数据库中的每个键都属于这 16384 个插槽的其中一个， 集群使用公式 CRC16(key) % 16384 来计算键 key 属于哪个槽， 其中 CRC16(key) 语句用于计算键 key 的 CRC16 校验和 。集群中的每个节点负责处理一部分插槽。 比如， 如果一个集群有3个主节点， 其中：节点 A 负责处理 0 号至 5460 号插槽；节点 B 负责处理 5461 号至 10922 号插槽；节点 C 负责处理 10923 号至 16383 号插槽。CRC16(k1) % 16384 = 5000，则k1会被放在节点A中。

> 为什么插槽数量是16384 ，而不是Master数量或者2^16-1=65535个（CRC16能提供的值）？
>
> redis source code author antirez say ：
>
> 1、正常的心跳包携带节点的完整配置，可以用幂等方式替换旧节点以更新旧配置。 这意味着它们包含原始形式的节点的插槽配置，它使用带有16k插槽的2k空间，但使用65k插槽时将使用高达8k的空间。
> 2、同时，由于其他设计权衡，Redis Cluster不太可能扩展到超过1000个主节点。
> 因此，16k处于正确的范围内，以确保每个主站有足够的插槽，最多1000个主站，但足够小的数字可以轻松地将插槽配置传播为原始位图。 请注意，在小型集群中，位图难以压缩，因为当N很小时，位图将设置插槽/ N位，这是设置的大部分位。

### 在集群中录入值

在redis-cli每次录入、查询键值，redis都会计算出该key应该送往的插槽，如果不是该客户端对应服务器的插槽，redis会报错，并告知应前往的redis实例地址和端口。

redis-cli客户端提供了 –c 参数实现自动重定向，如 redis-cli -c –p 6379 登入后，再录入、查询键值对可以自动重定向。

在集群中，不在一个slot下的键值，是不能使用mget,mset等多键操作。但是可以通过{组名}来定义组的概念,从而实现多键操作，redis会使用组名来计算对应的slot值。如`mset k1{cust} v1  k2{cust} v2 k3{cust} v3 k4{cust} v4`

### 故障恢复

如果主节点下线？从节点能否自动升为主节点？15秒超时后，从节点升为主节点。

主节点恢复后，主从关系会如何？主节点回来变成从机。

如果所有某一段插槽的主从节点都宕掉，redis服务是否还能继续?

如果配置文件中cluster-require-full-coverage 为yes ，那么 ，整个集群都挂掉，如果为no ，那么，只是该段插槽数据全都不能使用，也无法存储。

### 集群好处

+ 实现扩容
+ 分摊压力
+ 无中心化配置相对简单

### 集群不足

+ 不支持多键操作
+ 多键的Redis 事务是不被支持的
+ lua脚本不被支持。
+ 由于集群方案出现较晚，很多公司已经采用了其他的集群方案，而代理或者客户端分#的方案想要迁移至 redis cluster，需要整体迁移而不是逐步过渡，复杂度较大。

## Redis应用问题及解决

### 1. 缓存穿透

#### 问题描述

key 对应的数据在数据源并不存在，每次针对此 key 的请求从缓存获取不到，请求都会压到数据源（数据库），从而可能压垮数据源。比如用一个不存在的用户 id 获取用户信息，不论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库（频繁进行很多非正常的 url 访问）。

![image-20221015131449370](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/image-20221015131449370.png)

#### 缓存穿透现象

+ 应用服务器压力变大
+ redis 命中率降低
+ 一直查询数据库，使得数据库压力太大而压垮

其实 redis 在这个过程中一直平稳运行，崩溃的是我们的数据库（如 MySQL）。

#### 解决方案

1. **对空值缓存**：如果一个查询返回的数据为空（不管是数据是否不存在），我们仍然把这个空结果（null）进行缓存，设置空结果的过期时间会很短，最长不超过五分钟。
2. **设置可访问的名单（白名单）**：使用 bitmaps 类型定义一个可以访问的名单，名单 id 作为 bitmaps 的偏移量，每次访问和 bitmap 里面的 id 进行比较，如果访问 id 不在 bitmaps 里面，进行拦截，不允许访问。
3. **采用布隆过滤器**：布隆过滤器（Bloom Filter）是 1970 年由布隆提出的。它实际上是一个很长的二进制向量 (位图) 和一系列随机映射函数（哈希函数）。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。
4. **进行实时监控**：当发现 Redis 的命中率开始急速降低，需要排查访问对象和访问的数据，和运维人员配合，可以设置黑名单限制服务。

### 缓存击穿

#### 问题描述

key对应的数据存在，但在redis中过期，此时若有**大量并发请求**过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。
![image-20221013172834245](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/image-20221013172834245.png)

#### 缓存击穿现象

1. 数据库访问压力瞬时增加，数据库崩溃
2. redis 里面没有出现大量 key 过期
3. redis 正常运行

#### 解决方案

key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑缓存被“击穿”的问题，可以通过一下方案解决：

1. 预先设置热门数据：在redis高峰访问之前，把一些热门数据提前存入到redis里面，加大这些热门数据key的时长

2. 实时调整：现场监控哪些数据热门，实时调整key的过期时长

3. 使用锁：
   + 在缓存失效的时候（判断拿出来的值为空），不是立即去load db。
   + 先使用某些带成功操作返回值的操作（比如Redis的SETNX）去set一个mutex key，*就是设置一个锁*
   + 当操作返回成功时，再进行load db的操作，并回设缓存,最后删除mutex key；
   + 当操作返回失败，证明有线程在load db，当前线程睡眠一段时间再重试整个get缓存的方法。

 ![使用锁流程](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/image-20221015132311885.png)

### 缓存雪崩

#### 问题描述

key对应的数据存在，但在redis中过期，此时若有**大量并发请求**过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。缓存失效时的雪崩效应对底层系统的冲击非常可怕！

缓存雪崩与缓存击穿的**区别在于这里针对很多key缓存，前者则是某一个key**。

![image-20221015132819943](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/image-20221015132819943.png)

#### 解决方案

1. **构建多级缓存架构**：nginx缓存 + redis缓存 +其他缓存（ehcache等）
2. **使用锁或队列**：用加锁或者队列的方式保证来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上。不适用高并发情况
3. **设置过期标志更新缓存**：记录缓存数据是否过期（设置提前量），如果过期会触发通知另外的线程在后台去更新实际key的缓存。
4. **将缓存失效时间分散开**：比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。

### 分布式锁

#### 问题描述

随着业务发展的需要，原单体单机部署的系统被演化成分布式集群系统后，由于分布式系统多线程、多进程并且分布在不同机器上，这将使原单机部署情况下的并发控制锁策略失效，单纯的Java API并不能提供分布式锁的能力。为了解决这个问题就需要一种跨JVM的互斥机制来控制共享资源的访问，这就是分布式锁要解决的问题！

#### 常用实现方案

1. 基于数据库实现分布式锁
2. 基于缓存（Redis等）
3. 基于Zookeeper

其中通过redis实现性能最高，通过zookeeper实现可靠性最高。

#### Redis实现（通过setnx 实现）

只在键不存在时，才对键进行设置操作 (SET key value NX 效果等同于 SETNX key value )。

![image-20221015133725964](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/image-20221015133725964.png)

**注意事项**：

1. 设置锁的过期时间，防止发生死锁
2. setnx获取锁时，设置一个指定的唯一值（例如：uuid）；释放前获取这个值，判断是否自己的锁，防止误删
3. 需要保证删除锁操作的原子性，可使用LUA脚本

## Redis6新特性

### ACL

Redis ACL是Access Control List（访问控制列表）的缩写，该功能允许根据可以执行的命令和可以访问的键来限制某些连接。在Redis 5版本之前，Redis 安全规则只有密码控制，还有通过rename 来调整高危命令比如 flushdb ， KEYS* ， shutdown 等。Redis 6 则提供ACL的功能对用户进行更细粒度的权限控制 ：

（1）接入权限:用户名和密码

（2）可以执行的命令

（3）可以操作的 KEY

### IO多线程

Redis 6 加入多线程,但跟 Memcached 这种从 IO处理到数据访问多线程的实现模式有些差异。Redis 的多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程。之所以这么设计是不想因为多线程而变得复杂，需要去控制 key、lua、事务，LPUSH/LPOP 等等的并发问题。整体的设计大体如下:

![image-20221015134757872](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/image-20221015134757872.png)

另外，多线程IO默认也是不开启的，需要再配置文件中配置：

```bash
io-threads-do-reads  yes 
io-threads 4
```
