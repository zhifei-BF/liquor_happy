[TOC]

# NoSQL数据库概述

NoSQL = Not Only SQL ,意即“不仅仅是SQL",泛指非关系型的数据库。

NoSQL不遵循SQL标准，也不支持ACID原则,且远超于SQL的性能。

使用场景有

- 对数据高并发的读写
- 海量数据的读写
- 对数据高可扩展性

## Memcached

它是很早出现的一种NoSQL数据库。

它的数据都存储在内存中，并且不持久化。

它支持简单的key-value模式，支持类型单一。

一般是作为缓存数据辅助持久化的数据库。

## Redis

几乎覆盖了Memcached的绝大部分功能。

数据都在内存中，支持持久化，主要用作备份恢复。

除了支持简单的key-value模式，还支持多种数据结构的存储，比如：list、hash、set、zset等。

一般是作为缓存数据库辅助持久化的数据库。

## MongDB

高性能、开源、模式自由（schema free)的文档型数据库。

数据都在内存中，如果内存不足，把不常用的数据保存到硬盘。

虽然是key-value模式，但是对value（尤其是json）提供了丰富的查阅功能。

支持二进制数据及大型对象。

可以根据数据的特点替代RDBMS，成为独立的数据库。或者配合RDBMS，存储特定的数据。

# Redis

## Redis 简介

Redis是一个开源的key-value存储系统。

和Memcached类似，它支持存储的value类型相对更多，包括string（字符串），list（链表），set（集合），zset（sorted set-有序集合）和hash（哈希类型）。

这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是具有原子性的。

在此基础上，Redis支持各种不同方式的排序。

与Memcached一样，为了保证效率，数据都是缓存在内存中。区别的是Redis会周期性地把更新的数据写入到磁盘或者把修改操作写入追加的记录文件。

并且在此基础上实现了master-slave（主从）同步。

配合关系型数据库做高速缓存：

- 高频次，热门访问的数据，降低数据库IO
- 分布式架构，做session共享

## Redis安装

redis官网：https://redis.io/

- 下载redis安装包，上传到服务器的/opt目录下

- 解压缩redis压缩包：tar -zxvf redis.tar.gz
- 进入redis解压缩的文件夹下，里面的src是redis的源码，需要先编译才能安装
- 因redis是使用c语言开发的，需要使用c语言的编译命令
- 在redis目录下执行：make 命令时，报错，原因：系统中没有安装C语言的环境
- 安装C语言和C++语言环境

```
yum install -y gcc  //联网安装gcc
yum install -y gcc-c++	//联网安装gcc-c++
gcc -v  //查看gcc版本
g++ -v  //查看gcc-c++版本
```

- make执行错误时，会产生一些错误文件，需要将它们删除掉。

```
make distclean
```

- 然后重新编译

```
make
```

- 编译完成后，执行安装命令。

```
make install
```

- 如果执行redis-server能够看到redis的启动界面，就代表redis安装成功。
- 关闭redis可以使用如下命令：

```linux
ctrl + c
shutdown
```

- redis安装之后，命令存放在/usr/local/bin目录下，命令可以在任意的地方执行。

```
redis-server //redis服务端启动命令
redis-server ./xxx/redis.conf    //指定redis配置文件，然后进行服务端启动
redis-cli  //redis客户端启动命令
redis-cli -h IP地址 -p 6379	//指定ip地址和端口号进行客户端启动
```

- redis默认前台启动（阻塞启动），可以修改为后台启动，具体操作配置文件如下：

```
daemonize yes //即表示为后台启动
```

redis默认有16个库。一般使用默认的索引为0的库（切换库）：

```
select 0
```

查看redis存储数据的条数：

```
dbsize
```

清空当前库（慎用）：

```
flushdb
```

清空redis所有的库：

```
flushall
```

内存数据库存储数据都是在内存中操作，内存操作速度非常快，每个命令执行的时间很短。

redis是单线程+多路IO复用技术。

Memcached是串行+多线程的方式。

redis和Memcached三点不同：支持多数据类型、支持持久化、单线程+多路IO复用。

## redis-key操作

查看所有的键：

```
keys *
```

判断某个键是否存在：

```
exists 键名		//返回1代表存在，0代表不存在。
```

查看键对应值的类型：

```
type 键名
```

删除指定的键（键在值在，键亡值亡）：

```
del 键名
```

给键设置过期时间：

```
expire 键名 秒数
```

查看键的过期时间：

```
ttl 键名
/*
返回>=0的数字，代表剩余数字对应的秒数
返回-2，代表键已过期被删除
返回-1，代表键永不过期
*/
```

将键移到某个库中：

```
move 键   库的索引值
```

## redis-五大基本数据类型-string

redis最常使用的数据类型就是string类型。

redis中string类型的大小最大可以为512M，redis的字符串二进制安全。



向redis中存入string键值对：

```
set 键 值   //如果键相同，后设置的值会覆盖之前的值
```

获取redis中指定键的值：

```
get  键名
```

给redis中指定键的值追加内容：

```
append 键名  内容
```

获取键的值的长度：

```
strlen 键
```

当键不存在时才设置键值对（set if not exist）：

```
setnx  键   值
//设置成功是1，设置失败是0.
```

设置值自增1：

```
incr  键
/*
如果键的值已经存在，必须是整型才可以自增。
如果键不存在，默认在0的基础上+1。
*/
```

设置值自减1：

```
decr  键
```

自定义步长增减：

```
incrby 键  步长
decrby 键  步长
```

设置键值对同时设置过期时间（set with expire）：

```
setex 键  秒数  值
```

批量存多个键值对：

```
mset 键 值  键 值 .......
```

批量获取多个键的值：	

```
mget 键1   键2  键3 .......
```

批量设置键值对当键不存在时：

```
msetnx  键1  值1   键2  值2  ......
/*
当有一个键如果存在时，所有的数据都保存失败
只有所有的键都不存在时，才可以存储成功。
*/
```

redis命令具有原子性，命令执行时，必须成功或者失败。



java中的i++是否具有原子操作？

答：不是，执行时，分为三步，先获取i的变量的值，执行i++操作，将计算后的结果设置到内存中。



获取值指定范围内的内容：

```
getrange 键   start  end   // start、end分别是索引。end如果是-1，代表获取到最后。

//举例：
127.0.0.1：6379> set k1  12345678
OK
127.0.0.1：6379> getrange k1 2 5
"3456"
```

替换字符串指定范围的内容：

```
setrange k start replacestr  //start为索引，replacestr为替换的内容

//举例：
127.0.0.1：6379> setrange k1 3 hh
(integer) 8
127.0.0.1：6379> get k1
"123hh45678"
```

设置值并返回被替换的内容：

```
getset k newVal		//获取之前的值，然后设置新值。

//举例：
127.0.0.1：6379> getset k1 hello
"123hh678"
127.0.0.1：6379> get k1
"hello"
```

## redis-五大基本数据类型-list

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

它的底层实际是个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。

有序，元素可以重复。

如果键不存在，创建新的链表；如果键已存在，新增内容；如果值全移除，对应的键也就消失了。

左边的数优先放入栈里使用LPUSH：

```
127.0.0.1：6379> LPUSH list01 1 2 3 4 5
(Integer) 5
127.0.0.1:6379> LRANGE list01 0 -1
1) "5"
2) "4"
3) "3"
4) "2"
5) "1"
```

右边的数优先放入栈里使用RPUSH：

```
127.0.0.1：6379> RPUSH list02 1 2 3 4 5
(Integer) 5
127.0.0.1:6379> LRANGE list02 0 -1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
```

从左边（栈顶）/右边（栈底）吐出一个值。值在键在，值光键亡。

```
lpop/rpop  <key>           
//举例 lpop  list01
```

获取列表中某个索引值为x的值：

```
index  列表名   索引值
```

获取列表的长度：

```
llen 列表名
```

删除列表中N个value:

```
LREM 列表名  数量   值
```

ITRIM KEY  开始index结束index，截取指定范围的值后再赋值给key

```
LTRIM 列表名  索引值1   索引值2
```

从一个列表的栈底吐出一个数，放入另外一个列表的栈顶：

```
rpoplpush 列表1  列表2
```

设置列表中国某个索引位的值：

```
lset 列表名  	索引值   值
```

在列表中某个值的前面或者后面加入另外一个值：

```
linsert  列表名   before/after  值1   值2 
```

## redis-五大基本数据类型-set

自动去重，无序，底层是value值为null的hash 表。

向集合中添加多个元素，已经存在的元素将被忽略：

```
sadd <key> <value1> <value2> ......
```

获取集合中的元素：

```
smembers <key>
```

判断某个集合中是否含有某个元素，返回1表示有，返回0表示没有：

```
sismember <key> <value>
```

返回集合中元素的个数：

```
scard <key>
```

删除集合中的元素：

```
srem <key> <value1> <value2> ......
```

随机从集合中吐出一个值（会被删除）：

```
spop <key> 
```

随机从该集合中吐出n个值，不会从集合中删除：

```
srandmember <key> <n>
```

把集合中一个值从一个集合移动到另一个集合：

```
smove <source> <destination> <value>
<source> 为要移动值的集合
<destination> 为将要移动到的集合
<value> 为要移动的值
```

返回两个集合的交集元素：

```
sinter <key1> <key2>
```

返回两个集合的并集元素：

```
sunion <key1> <key2>
```

返回两个集合的差集元素（key1中的，不包含key2中的）：

```
sdiff <key1> <key2>
```

## redis-五大基本数据类型-hash

KV模式不变，但V是一个键值对。



向集合中设置值：

```
hset <key> <valueKey> <valueValue>
<valueKey> 为值里面的键
<valueValue> 为值里面的值
```

获取集合中的值里面的值：

```
hget 键  值里面的键
```

向集合中设置多个值：

```
hmset 键 值里面的键1  值里面的值1  值里面的键2  值里面的值2 ......
```

向集合中获取多个值里面的值：

```
hmget 键 值里面的键1 值里面的键2 值里面的键3 ......
```

获取集合中值里的所有键值对：

```
hgetall 键
```

删除集合中值里面的键值对：

```
hdel 键  值里面的键
```

查看集合中键包含键值对的长度：

```
hlen 键
```

判断集合中值里面的某个键是否存在（存在返回1，不存在返回0）：

```
hexists 键  值里面的键
```

获取集合里面所有值里的键：

```
hkeys 键
```

获取集合里面所有值里面的值：

```
hvals 键
```

向集合中值里面的值增加某个整数：

```
hincrby 键 值里面的键  n
n表示为要增加的整数
```

向集合中值里面的值增加某个小数：

```
hincrbyfloat 键 值里面的键 n
```

集合中值里面的键如果不存在则设置值里面的值：

```
hsetnx 键 值里面的键  值里面的值
```

## redis-五大基本数据类型-zset

在set基础上，加一个score值。

之前set是 k1 v1 v2 v3 ，现在zset是k1 score1 v1 score2 v2 。

向zset中添加元素：

```
zadd k1 score1 v1 score2 v2 
```

向zset中取出值：

```
zrange k1 index0 index1
index为索引，当index为-1时，表示取出全部。
```

向zset取出值，并且取出score值：

```
zrange k1 index0 index1 withscore
```

向zset取出值，根据score范围来：

```
zrangebyscore key score1 score2
如果是(score1和(score2，则表示不包含score1和score2。
zrangebyscore key score1 score2 limit index0 count
表示获取score1到score2范围内，索引从index0开始的count个数。
```

删除集合中某个元素：

```
zrem 键  值
```

统计集合中的个数：

```
zcard 键
```

计算score在某个范围内的个数：

```
zcount 键 index0 index
```

获取集合中某个值的索引：

```
zrank 键 值
```

获取集合中某个值的score值：

```
zscore 键 值
```

逆向获取某个值的索引值：

```
zrevrank 键 值
```

逆向获取值：

```
zrevrange 键 index0 index1
```

逆向根据score值获取值：

```
zrevrangebyscore 键 score1 score2
```

## 配置文件介绍

redis的配置文件默认是在redis的根目录下，名为redis.conf。

redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程：

```
daemonize no
```

当redis以守护进程方式运行时，redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定：

```
pidfile /var/run/redis.pid
```

指定redis监听端口，默认端口为6379，redis的开发者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字。

```
port 6379
```

绑定的主机地址：

```
bind 127.0.0.1
```

当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能：

```
timeout  300
```

指定日志记录级别，redis总共支持四个级别：debug、verbose、notice、warning，默认是verbose。

```
loglevel verbose
```

日志记录方式，默认为标准输出，如果配置redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null。

```
logfile stdout
```

设置数据库的数量，默认数据库为0，可以使用SELECT  < dbid > 命令在连接上指定数据库id：

```
databases 16
```

指定本地数据库文件名，默认值为dump.rdb

```
dbfilename dump.rdb
```

指定本地数据库存放目录：

```
dir ./
```

设置redis连接密码，如果配置了连接密码，客户端在连接redis时需要通过AUTH < PASSWROD >命令提供密码，默认关闭。

```
requirepass foobared
```

设置同一时间最大客户端连接数，默认无限制，redis可以同时打开的客户端连接数为redis进程可以打开的最大文件描述符数，如果设置maxclients 0，表示不作限制。当客户端连接数到达限制时，redis会关闭新的连接并向客户端返回max number of clients reached错误信息。

```
maxclients 128
```

指定redis最大内存限制，redis在启动时会把数据加载到内存中，达到最大内存后，redis会向尝试清除已到期或者即将到期的Key，当此方法处理后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。redis新的vm机制，会把Key存放内存，Value会存放再swap区。

```
maxmemory <bytes>
```



当在某个路径下启动redis时，可以使用下面命令获取这个路径：

```
config get dir
```

当redis设置了密码时，可以使用以下命令获取密码：

```
config get requirepass
```

设置密码：

```
config set requirepass "密码"
```

进行密码登录：

```
auth 密码
```

## 持久化之RDB

RDB（redis database）

在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里。

redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能；如果需要进行大规模数据恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。

Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等）数值都和原进程一致。但是是一个全新的进程，并作为原进程的子进程。

RDB保存的是dump.rdb文件。

在redis.conf配置文件中，默认有三种情况可以触发保存RDB（可以对它进行修改）。

```
save <second> <change>

save 900 1		//15分钟改变1次以上
save 300 10 	//5分钟改变10次以上
save 60  10000	//1分钟改变10000次以上

如果设置如下：
save ""
则表示不启用RDB持久化。
```

配置文件中可以对dump.rdb进行修改：

```
dbfilename  dump.rdb
```

当执行shutdown或者flushall 时，都会迅速进行rdb备份。

也可以进行手动备份，执行如下命令：

```
save
```

配置文件中默认在备份时出错，就停止写入；如果配置成no，表示你不在乎数据不一致或者有其他的手段发现和控制。

```
stop-write-on-bgsave-error yes
```

对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能。

```
rdbcompression  yes
```

在存储快照后，还可以让redis使用CRC64算法进行数据校验，但是这样会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能。

```
rdbchecksum  yes
```

Save：save时只管保存，其他不管，全部阻塞。

BGSAVE:redis会在后台异步进行快照操作，快照同时还可以响应客户端请求。可以通过lastsave命令获取最后一次成功执行快照的时间。



备份文件如何进行恢复？

将备份文件（dump.rdb)移动到redis安装目录并启动服务即可。

CONFIG GET dir获取目录



RDB的优势：适合大规模的数据恢复，对数据完整性和一致性要求不高。

RDB的劣势：在一定间隔时间做一次备份，所以如果redis意外down掉的话，就会丢失最后一次快照后的所有修改；fork的时候，内存中的数据被克隆了一份，大制2倍的膨胀性需要考虑。



如何停止RDB备份：动态所有停止RDB保存规则的方法:

```
redis-cli config set save ""
```

## 持久化之AOF

AOF(append only file)

AOF是以日志的形式来记录每个写操作，将redis执行过的所有写指令记录下来（读操作不记录），只许追加文件当不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

AOF保存的是appendonly.aof文件。

在配置文件中，默认AOF是关闭的。

```
appendonly no
```

默认文件名appendonly.aof。

```
appendfilename "appendonly.aof"
```

当rdb文件和aof文件共存时，会优先读取aof文件，当读取aof文件是损坏文件时，直接redis无法启动。但是我们可以通过如下命令，对aof文件进行修改，把无法识别的命令进行剔除。

```
redis-check-aof --fix appendonly.aof
```

配置文件中aof策略选择：always、everysec（默认）、no

```
appendfsync everysec
always：同步持久化，每次发生数据变更会被立即记录到磁盘，性能较差但数据完整性比较好。
everysec：出厂默认推荐，异步操作，每秒记录，如果一秒内宕机，有数据丢失。
no
```

关于rewrite：AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制，当AOF文件的大小超过所设定的阈值时，redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集，可以使用命令bgrewriteaof。

重写原理：AOF文件持续增长而过大时，会fork出一条新进程来将文件重写（也是先写临时文件最后再rename）。遍历新进程的内存中的数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似。

触发机制：redis会记录上次重写时AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件小于64M时触发。

```
auto-aof-rewrite-percentage 100 //百分百，一倍
auto-aof-rewrite-min-size 64mb
```

重写时是否可以运用Appendfsync，用默认no即可，保证数据安全性。

```
no-appendfsync-on-rewrite no
```

AOF的劣势：相同数据集的数据而言aof文件要远大于rdb文件，恢复速度慢于rdb；aof运行效率要慢于rdb，每秒同步策略效率较好，不同步效率和rdb相同。

## redis的事务

redis的事务可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，按顺序地串行化执行而不会被其他命令插入，不许加塞。

在一个队列中，一次性、顺序性、排他性的执行一系列命令。

**相关命令：**

multi：表示开始进行事务，记录之后的一系列命令到队列中。

exec： 表示提交一系列命令。

discard：表示放弃提交之前输入的一系列命令。

**事务的错误处理：**

- 组队中某个命令出现了报告错误，执行时整个的所有队列都会被取消。（相对于java的编译阶段报错）
- 如果执行阶段某个命令报出了错误，则只有报错的命令不会被执行，而其他的命令都会执行，不会回滚。（相当于java的运行阶段报错）
- 所以通常来说，redis支持事务，但是是部分事务，而非全部事务。

**悲观锁**

悲观锁(Pessimistic Lock), 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。

悲观锁：请求获取到数据时会加锁，其他请求等待锁释放才可以争抢锁使用[高并发写操作为了保证数据安全可以使用， R数据库]

**乐观锁**

乐观锁(Optimistic Lock), 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量。Redis就是利用这种check-and-set机制实现事务的。

乐观锁：对数据加版本号，当请求获取数据时会一起获取到版本号，其他请求读取数据也没有问题，当任意一个请求修改数据时，乐观锁会检查数据的版本号，如果请求数据的版本号和存储数据的版本号一致，可以修改，并提升版本号。如果获取了修改前数据版本号的其他请求再修改数据一定会失败[高并发读操作]

**WATCH key[key...]**

在执行multi之前，先执行watch key1 [key2],可以监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

**unwatch**

- 取消 WATCH 命令对所有 key 的监视。
- 如果在执行 WATCH 命令之后， EXEC 命令或 DISCARD 命令先被执行了的话，那么就不需要再执行 UNWATCH 了。

**redis事务三特性**

- 单独的隔离操作
  - 事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。 

- 没有隔离级别的概念
  - 队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行。

- 不保证原子性
  - 事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚 。

## redis的发布订阅

redis的发布订阅是进程间的一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接收消息。

先订阅后发布才能收到消息。

**相关命令：**

可以一次订阅多个：

```
SUBSCRIBE c1 c2 c3
// c1 c2 c3即表示频道
```

消息发布：

```
PUBLISH c2 hello-redis
```

订阅多个，通配符*：

```
PSUBSCRIBE new*
//表示订阅new相关的频道
```

收取消息：

```
PUBLISH new1 redis2021
```

## redis的主从复制

主从复制：主机数据更新后根据配置和策略，自动同步到备机的master/slaver机制，Master以写为主，Slave以读为主。

配从不配主。

从库配置：`slaveof 主库ip 主库端口` 每次与master断开之后，都需要重新连接，除非你配置进redis.conf文件。

`info repication`用于查看当前连接的redis是什么角色，master还是slave。

![1623339130954](redis.assets/1623339130954.png)

读写分离

**一主二仆**

- 某个主机变从机，将会备份主机的所有数据。

- 主机死了，从机原地待命
- 主机回来，没任何问题。
- 从机死了，回来以后变主机。

**薪火相传**

上一个Slave可以是下一个slave的Master，Slave同样可以接收其他slaves的连接和同步请求，那么该Slave作为了链条中下一个的master，可以有效减轻master的写压力。

使用`info repication`查看中间那个机器，发现它的角色其实还是slave。

```
Slaveof 新主库ip 新主库端口
```

**反客为主**

当主机挂掉以后，从机想变为主机，可以使用命令：`SLAVEOF no one`(使当前数据库停止与其他数据库的同步，转成主数据库)

**主从复制的原理**

Slave启动成功连接到master后会发送一个sync命令，Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，以完成一次完全同步。

全量复制：而slave服务在接收到数据库文件后，将其存盘并加载到内存中。

增量复制：Master继续将新的所有收集到的修改命令依次传给slave，完成同步。

但是只要是重新连接master，一次完全同步（全量复制）将被自动执行。

**哨兵模式（sentinel）**

哨兵模式就是反客为主的自动版。能够后台监控主机是否故障，如果故障了根据投票数自动从库转换成主库。

在redis.conf相关路径下新建一个文件`touch sentinel.conf`,在配置文件加入下列一行配置：

```
sentinel monitor 被监控的数据库的名字 被监控的数据库ip 被监控的数据库端口 票数
//票数表示主机挂掉后slave秃瓢看让谁接替成为主机，得票数多少后成为主机。
```

启动哨兵：

```
Redis-sentinel sentinel.conf
```

当原主机回来以后，被哨兵监控到，将自动变为现主机的slave。

一组sentinel能同时监控多个Master。

**复制的缺点：**

复制延时：由于所有的写操作都是先在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave机器有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重。

## Jedis连接

jedis所需要的架包：

- commons-pool2-2.4.2.jar
- jedis-2.8.1.jar

连接redis注意事项：

1. 禁用Linux的防火墙：Linux(CentOS7)里执行命令：`systemctl stop/disable firewalld.service  `

2. redis.conf中注释掉bind 127.0.0.1 ,然后 `protected-mode no`



在maven项目中的pom文件引入依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-pool2</artifactId>
        <version>2.6.1</version>
    </dependency>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.9.0</version>
    </dependency>
</dependencies>
```

然后代码测试连接是否成功：

```java

package com.atguigu.jedis;
 
import redis.clients.jedis.Jedis;
 
public class JedisTest {
    /**
     * JedisConnectionException: java.net.SocketTimeoutException:
     *      错误排查：
     *             1.1  检查reids是否启动
     *                  xhsell中： ps -aux|grep redis
     *             1.2 检查虚拟机防火墙是否关闭
     *             1.3 redis保护模式必须关闭
     *                  vim  /myredis/redis.conf
     *                     69行 ， 注释bind 127.0.0.1
     *                      88行，protected-mode yes改为no
     *                      重启redis服务
     */
    public static void main(String[] args) {
        //java代码连接redis步骤
        //1、获取连接
        Jedis jedis = new Jedis("192.168.1.130", 6379);
        //2、使用连接对象发送命名操作redis
        String ping = jedis.ping();
        System.out.println("ping = " + ping);
        //3、关闭连接
        jedis.close();
 
    }

}
```

使用代码向redis插入数据**(其他数据类型也是类似)**：

```java
 public static void main(String[] args) {
     //java代码连接redis步骤
     //1、获取连接
     Jedis jedis = new Jedis("192.168.1.130", 6379);
     //插入
     jedis.set("k1","v1");
     jedis.set("k2","v2");
     //获取
     System.out.println(jedis.get("k1"));
     //获取所有
     Set<String> sets = jedis.keys("*");
     System.out.println(sets.size());
}
```

**jedis事务：**

```java
 public static void main(String[] args) {
        Jedis jedis = new Jedis("192.168.1.130", 6379);
        Transaction transaction = jedis.multi();
 		transaction.set("k4","v4");
 		transaction.set("k5","v5");
 		//transaction.exec();提交
     	transaction.discard();//放弃
}
```

**jedis主从复制：**

```java
public static void main(String[] args){
    Jedis jedis_M = new Jedis("192.168.1.130",6379);
    Jedis jedis_S = new Jedis("192.168.1.130",6380);
    jedis_S.slaveof("192.168.1.130",6379);
    jedis_M.set("class","1122");
    String result = jedis_S.get("class");
    System.out.println(result);
}
```

## JedisPool

```java
public class JedisPoolUtil{
    private static volatile  JedisPool jedisPool = null;
    
    private JedisPoolUtil(){}
    
    public static JedisPool getJedisPoolInstance(){
        if(null == jedisPool){
            synchronized(JedisPoolUtil.class){
                if(null == jedisPool){
                	JedisPoolConfig poolConfig = new JedisPoolConfig();
                	poolConfig.setMaxActive(1000);
                	poolConfig.setMaxIdle(32);
                	poolConfig.setMaxWait(100 * 1000);
                	poolConfig.setTestOnBorrow(true);
                	
                    jedisPool = new JedisPool(poolConfig,"127.0.0.1",6379);
                }
            }
        }
        return jedisPool;
    }
    
    public static void release(JedisPool jedisPool, Jedis jedis){
        if(null != jedis){
            jedisPool.returnResourceObject(jedis);
        }
    }
}
```



```java
public static void main(String[] args){
    JedisPool jedisPool = JedisPoolUtil.getJedisPoolInstance();
    //JedisPool jedisPool2 = JedisPoolUtil.getJedisPoolInstance();
    //System.out.println(jedisPool == jedisPool2);
    
    Jedis jedis = null;
    try{
        jedis = jedisPool.getResouce();
        jedis.set("k1","v1");
    }catch(Exception e){
        e.printStackTrace();
    }finally{
        jedisPoolUtil.release(jedisPool,jedis);
    }
}
```

maxActive: 控制一个pool可分配多少个jedis实例，通过pool.getResource()来获取；如果赋值为-1，则表示不限制；如果pool已经分配了maxActive个jedis实例，则此时pool的状态为exhausted。

maxIdle：控制一个pool最多有多少个状态为Idle（空闲）的jedis实例；

maxWait：表示当borrow一个jedis实例时，最大的等待时间，如果超过等待时间，则直接抛JedisConnectionException；

testOnBorrow：获得一个jedis实例的时候是否检查连接可用性（ping（））；如果为true，则得到的jedis实例均是可用的；

testOnReturn：return一个jedis实例给pool时，是否检查连接可用性（ping（））;

