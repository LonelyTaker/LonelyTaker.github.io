---
title: Redis
date: 2024-10-06 00:29:24
categories: 后端
tags: Redis
---

# 数据类型

## 字符串（String）

```shell
set key value [NX|XX] [GET] [EX seconds|PX milliseconds|EXAT unix-time-seconds|PXAT unix-time-milliseconds|KEEPTTL]
```

* NX：键不存在的时候设置键值
* XX：键存在的时候设置键值
* GET：返回指定键原本的值，若键不存在时返回`nil`
* EX seconds：以秒为单位的过期时间
* PX milliseconds：以毫秒为单位的过期时间
* EXAT unix-time-seconds：设置以秒为单位的UNIX时间戳对应的时间为过期时间
* PXAT unix-time-milliseconds：设置以毫秒为单位的UNIX时间戳对应的时间为过期时间
* KEEPTTL：保留设置前指定键的生存时间

<br/>

同时设置/获取多个键值：

```shell
mset k1 v1 k2 v2 k3 v3
mget k1 k2 k3
msetnx k1 v1 k4 v4 # 如果其中一项失败，整体失败，k1已存在，所以都不会设置成功
```

获取指定区间范围内的值：

```shell
set k1 abcd1234
getrange k1 0 3 # abcd，类似切片
setrange k1 0 xxxx # xxxx1234，从第0位开始，使用后面的字符串覆盖
```

数值增减（一定要是数字才能进行加减）：

```shell
set k1 100
INCR k1 # 递增
INCRBY k1 2 # 增加指定整数
DECR k1 # 递减
DECRBY k1 2 # 减少指定整数
```

获取字符串长度和内容追加：

```shell
STRLEN k1 # 获取字符串长度1
APPEND k1 xxxx # 追加
```

其他：

```shell
setnx k1 v1 # 只有key不存在时设置key的值
setex k1 10 v1 # 设置key的同时设置过期时间
getset k1 v1 # 先get再set
```

<br/>

## 列表（List）

一个双端链表结构，主要功能有push、pop等，一般用在栈、队列、消息队列等场景

* left、right都可插入添加
* 如果键不存在，创建新的链表
* 如果键已存在，新增内容
* 若果值全移除，对应的键也就消失了

由于底层是个双向链表，对两端的操作性能很高，通过索引下标操作中间节点的性能会较差。

```shell
lpush/rpush
lpop/rpop
lrange k1 0 -1 # 遍历列表
lindex # 按照索引获取元素
llen # 获取列表中元素个数
lrem key 数字N 给定值v1 # 删除N个值等于v1的元素
ltrim key 开始index 结束index # 截取指定范围的值后再赋值给key
rpoplpush 源列表 目标列表
lset key index value
linsert key before/after 已有值 插入的新值
```

<br/>

## 哈希表（Hash）

key-value键值对模式不变，但是value是一个键值对

`Map<String,Map<Object,Object>>`

```shell
hset/hget/hmset/hmget/hgetall/hdel
# hset user id 11 age25
# hset之前只能设置一个值，现在可以设置多个，代替了hmset的功能，hmset已被弃用
hlen # 获取某个key内所有字段数量
hexists key 具体字段
hkeys/hvals key # 罗列key内所有字段/字段值
hincrby/hincrbyfloat
hsetnx
```

<br/>

## 集合（Set）

单值多value，且无重复

```shell
sadd key member [member...] # 添加元素
smembers key # 遍历所有元素
sismember key member # 判断元素是否在集合中
srem key member [member...] # 删除軅
scard # 获取集合内元素个数
srandmember key [数字N] # 从集合中随机展现N个元素，元素不删除
spop key [数字N] # 从集合中随机弹出N个元素，元素删除
smove key1 key2 key1中存在的值 # 将key1中某个值赋予key2
# 集合运算
A:{a,b,c,1,2} B:{1,2,3,a,x}
sdiff A B # 差集
sunion A B # 并集
sinter A B # 交集
sintercard numkeys key [key ...] [LIMIT limit] # redis7新命令
sintercard 2 A B # 返回给定集合的交集产生的集合的基数
```

<br/>

## 有序集合（ZSet）

在set的基础上，每个val值前加一个score分数值

```shell
zadd key score member [score member...] # 添加元素
zrange key start stop [WITHSCORES] # 按照分数从小到大的顺序返回索引从start到stop之间的所有元素
zrevrange key start stop [WITHSCORES] # 按照分数从大到小的顺序返回索引从start到stop之间的所有元素
zrangebyscore key min max [WITHSCORES] [LIMIT offset count] # 获取指定分数范围的元素，等价于 zrange key min max byscore
# zrangebyscore key (min (max
# 小括号表示不包含（开区间）
zscore key member # 获取元素的分数
zcard key # 获取集合中元素数量
zrem key member # 删除元素
zincrby key increment member # 增加某个元素的分数
zcount key min max # 获取指定分数范围内的元素个数
zmpop numkeys key [key ...] <MIN | MAX> [COUNT count] # redis7新命令，从键名列表中的第一个非空排序机制弹出一个或多个元素
zrank key values # 获取下标值
zrevrank key values # 逆序获取下标值
```

<br/>

>以上为五种基本类型，以下为特殊类型

<br/>

## 地理空间（GEO）

```shell
geoadd # 将指定的地理空间位置（纬度、经度、名称）添加到指定的key中
geopos # 从key里返回所有给定位置元素的位置（经度和纬度）
geodist # 返回两个给定位置之间的距离
georadius # 以给定的经纬度为中心， 找出某一半径内的元素
georadiusbymember # 找出位于指定范围内的元素，中心点是由给定的位置元素决定
geohash # 返回一个或多个位置元素的 Geohash 表示
```

<br/>

## 基数统计（HyperLogLog）

主要用来做基数统计的算法，其优点是在输入元素的数量或者体积非常大时，计算基数所需的空间总是固定的，并且是很小的。

在Redis中，每个HyperLogLog键只需要花费12kb内存，就可以计算接近2^64个不同元素的基数。这和计算基数时，元素越多耗费内存越多的集合形成鲜明对比。

但是，因为HyperLogLog只会根据输入元素来计算基数，不会存储输入元素本身，所以HyperLogLog不能像集合那样，返回输入的各个元素。

基数统计：用于统计一个集合中不重复的元素个数，就是对集合去重复后剩余元素的计算

```shell
pfadd key element [element...] # 添加指定元素到HyperLogLog中
pfcount key [key...] # 返回给定HyperLogLog的基数估算值
pfmerge destkey sourcekey [sourcekey...] # 将多个HyperLogLog合并为一个HyperLogLog
```

<br/>

## 位图（bitmap）

用String类型作为底层数据结构实现的一种统计二值状态（0和1）的数据类型，位图本质是数组，它是基于 String数据类型的按位的操作。该数据由多个二进制位组成，每个二进制位都对应一个偏移量（我们称之为一个索引）

```shell
setbit key offset val # 给指定key的第offset位赋值val
getbit key offset # 获取指定key的第offset位
strlen key # 获取指定key占用的字节长度，不满8位按1字节补全
bitcount key start end # 返回指定key中[start, end]中为1的数量
bitop operation destkey key [key...] # 对不同的二进制存储数据进行位运算（AND、OR、NOT、XOR）
```

<br/>

## 位域（bitfield）

Redis 位域允许您设置、递增和获取任意位长度的整数值。例如，您可以对从无符号 1 位整数到有符号 63 位整数的任何值进行操作。bitfield支持原子读取、写入和增量操作，使其成为管理计数器和类似数值的理想选择。

总结：将一个Redis字符串看作是一个由二进制位组成的数组，并能对变长位宽和任意没有字节对齐的指定整型位进行寻址和修改

<br/>

## 流（Stream）

Redis版的MQ消息中间件+阻塞队列

<br/>

# 持久化

将内存中的数据写入磁盘

## RDB

全称`Redis Database`，以指定的时间间隔执行数据集的时间点快照。

把某一时刻的数据和状态以文件的形式写到磁盘上，也就是快照。这样依赖即使故障宕机，快照文件也不会丢失，数据的可靠性也就得到了保证。这个快照文件就称为RDB文件（dump.rdb）

* 自动触发

  6.0.16以下版本配置：

  ```yaml
  ################## SNAPSHOTTING ################
  save 900 1 # 每隔15min，如果超过1个key发生变化，就写一份新的RDB文件
  save 300 10 # 每隔5min，如果超过10个key发生变化，就写一份新的RDB文件
  save 60 10000 # 每隔1min，如果超过10000个key发生变化，就写一份新的RDB文件
  ```

  公式`save m n`表示m秒内数据集存在n次修改时，自动触发保存

  7版本配置：

  ```yaml
  save 3600 1 300 100 60 10000
  ```

* 手动触发

  可以使用`SAVE`或`BGSAVE`命令

  save不会fork子进程，而是直接阻塞主线程来备份数据，直接持久化工作完成，执行期间Redis不能处理其他命令，线上禁止使用

  bgsave会fork一个子进程进行保存，这个操作是后台完成的，这就允许主进程还可以响应客户端请求

<br />

**优点：**

* RDB适合大规模的数据恢复。
* 可以按照业务定时备份。
* RDB文件在内存中的加载速度要比AOF快得多。
* RDB最大限度地提高了Redis性能，因为Redis父进程为了持久化而需要做的唯一工作就是派生一个将完成所有其余工作的子进程。父进程永远不会执行磁盘IO或类似操作。

**缺点：**

* 由于RDB保存有一定时间间隔，如果Redis由于任何原因在没有正确关闭的情况下停止工作，可能存在数据丢失。
* 内存数据的全量同步，如果数据类太大会导致IO严重影响服务性能。

<br />

**触发RDBd的情况：**

* 配置中默认的快照配置（自动触发）
* 手动save/bgsave命令（手动触发）
* 执行flushall/flushdb命令（产生的是个空内容文件）
* 执行shutdown且没有设置开启AOF持久化
* 主从复制时，主节点自动触发

<br />

检查RDB文件：

```shell
redis-check-rdb xxx.rdb
```

禁用快照：

```yaml
sava "" # 配置文件内填写空字符串
```

配置文件中相关的配置信息：

```yaml
save <seconds> <changes> # 自动触发配置
dbfilename # RDB文件名
dir # 存放路径
stop-writes-on-bgsave-error yes # 默认yes，持久化保存出错时是否停止写请求
rdbcompression yes # 默认yes，是否压缩存储。redis会采用LZF算法进行压缩，如果不想消耗CPU进行压缩的话，可以关闭此功能
rdbchunsum yes # 默认yes，在存储快照后还可以让redis用CRC64算法来进行数据校验，但这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能
rdb-del-syns-files no # 默认no，在没有持久化的情况下删除复制中使用的RDB文件
```

<br />

## AOF

全称`Append Only File`，以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来（读操作不记录），只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据。

默认情况下Redis是没有开启AOF的，开启配置：

```yaml
appendonly yes
```

1. 写操作命令到达Redis Server后并不是直接写入AOF文件，会将这些命令先放入`AOF缓存`中进行保存。这里的AOF缓存区实际上是内存中的一片区域，存在的目的是当这些命令达到一定量以后再写入磁盘，避免频繁的磁盘IO操作

2. AOF缓存会根据AOF缓存区`同步文件的三种写回策略`将命令写入磁盘的AOF文件

   写回策略：

   * always：每个写命令执行完立刻同步地将日志写回磁盘
   * everysec（默认）：每秒写回，每个写命令执行完，把日志放入缓冲区，间隔1s写回磁盘
   * no：操作系统控制的写回，每个写命令执行完，只是把日志放入缓冲区，由操作系统决定何时将缓冲区内容写回磁盘

3. 随着写入AOF内容的增加为避免文件膨胀，会根据规则进行命令的合并（又称`AOF重写`），从而起到AOF文件压缩的目的

<br/>

**优点：**

* 更好保护数据不丢失、性能高、可做紧急恢复

**缺点：**

* AOF文件通常比相同数据集的等效RDB文件大
* AOF运行效率要慢于RDB

<br/>

**AOFRW**

* 自动触发

  ```yaml
  auto-aof-rewrite-percentage 100
  auto-aof-rewrite-min-size 64mb
  ```

  同时满足上面两个条件才会触发：

  * 根据上次重写后的AOF大小，判断当前AOF大小是否增长了1倍（百分百）
  * 重写时满足的文件大小（64MB）

* 手动触发

  ```shell
  bgrewriteaof
  ```

>AOFRW流程以及MP-AOF的优化可参考：
>
>https://developer.aliyun.com/article/866957#

<br/>

相关配置：

```yaml
appendfsync everysec # 写回策略
appenddirname "appendonlydir" # 日志存放路径
appendfilename "appendonly.aof" # 日志文件名
```

redis7.0后采取MP-AOF的方式存储文件，顾名思义，MP-AOF就是将原本的单个AOF文件拆分为多个AOF文件。在MP-AOF中，我们将AOF分为三类，分别是：

* BASE：表示基础AOF，一般由子进程通过重写产生，该文件最多只有一个
* INCR：表示增量AOF，一般会在AOFRW开始执行时被创建，该文件可能存在多个
* HISTORY：表示历史AOF，它是由BASE和INCR AOF变化而来，每次AOFRW成功完成时，本次AOFRW之前对应的BASE和INCR AOF都将变为HISTORY，HISTORY类型的AOF会被Redis自动删除

为了管理这些文件，我们引入了一个manifest（清单）文件来跟踪、管理这些AOF。同时，为了便于AOF备份和拷贝，我们将所有的AOF文件和manifest文件放入一个单独的文件目录中，由appenddirname配置决定。

<br/>

## RDB+AOF混合持久化

* 默认情况下Redis仅开启了RDB，AOF未开启
* 两者可以同时开启
* 同时开启时，服务重启只会加载AOF，不会加载RDB文件

这是一种推荐的持久化方式，因为：

* 通常情况下AOF文件保存的数据集要比RDB文件保存的数据集完整（RDB间隔较长，服务宕机损失的数据较大）
* RDB适合用于备份数据库（AOF在不断变化不好备份）

开启混合方式：

```yaml
aof-use-rdb-preamble yes
```

RDB镜像做全量持久化，AOF做增量持久化。

先使用RDB进行快照存储，然后使用AOF持久化记录所有的写操作，当重写策略满足或手动触发重写的时候，**将最新的数据存储为新的RDB记录**。这样的话，重启服务会从RDB和AOF两部分恢复数据，既保证了数据完整性，又提高了恢复数据的性能。简单来说：混合持久化方式产生的文件一部分是RDB格式，一部分是AOF格式（AOF包括了RDB头部和AOF混写）

<br/>

# 事务

可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，按顺序串行化执行而不会被其他命令插入。

特点：

* 单独的隔离操作

  Redis的事务仅仅保证事务里的操作会被连续独占的执行，Redis命令执行是单线程架构，在执行完事务内所有指令前是不可能再去同时执行其他客户端的请求的。

* 没有隔离级别的概念

  因为事务提交前任何指令都不会被实际执行，也就不存在“事务内的查询要看事务内的更新，在事务外查询不能看到”这种问题了。

* 不保证原子性

  Redis的事务不保证原子性，也就是不保证所有指令同时成功或同时失败，只有决定是否开始执行全部指令的能力，没有执行到一半回滚的能力。

* 排他性

  Redis会保证一个事务内的命令依次执行，而不会被其他命令插入。

<br />

## 使用方式

```shell
# 开启事务
MULTI
...
EXEC

# 放弃事务
MULTI
...
DISCARD
```

两种错误情况：

* 在`EXEC`之前，任何一个命令语法有错，Redis会直接返回错误，所有的命令都不会执行。
* 在`EXEC`之后，当一个命令执行失败，不会影响到其他命令，其他正确的命令仍然会执行。

<br />

## watch监控

Redis使用Watch来提供乐观锁定

* 悲观锁：每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁
* 乐观锁：每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据

乐观锁策略：提交版本必须大于记录当前版本，才能执行更新

客户端1：

```shell
set balance 100
set k1 v1

WATCH balance # 1
MULTI
set balance 150 # 3
set k1 xxx
EXEC # nil
```

客户端2：

```shell
set balance 120 # 2
```

客户端1被客户端2加塞，客户端1的**整个事务**都会中断\

可以使用unwatch取消监控

```shell
WATCH balance
UNWATCH
```

注意：

* 一旦执行EXEC，之前加的监控锁都会被取消
* 当客户端连接丢失的时候（断连），所有监控都会被取消

<br />

# 管道

问题由来：

1. 客户端向服务端发送命令（发送命令 -> 命令排队 -> 命令执行 -> 命令返回），并监听Socket返回，通常以阻塞模式等待服务端响应
2. 服务端处理命令，并将结果返回客户端

上述两步称为：`Round Trip Time`（简称RTT，数据包往返于两端的时间）

![redis管道-1](redis管道-1.png)

如果同时执行大量的命令，那么就要等待上一条命令应答后再执行，这中间不仅多了RTT，而且还频繁调用系统IO，发送网络请求，同时需要Redis调用多次read()和write()系统方法，系统方法会将数据从用户态转移到内核态，这样就会对进程上下文有较大影响，性能不好。

解决方案：

管道可以一次性发送多条命令给服务端，服务端依次处理完毕后，通过一条响应一次性将结果返回，通过减少客户端与redis的通信次数来实现降低往返延时时间。pipeline实现的原理是队列，先进先出特性就保证数据的顺序性。

![redis管道-2](redis管道-2.png)

<br />

## 管道和原生批量命令对比

原生批量命令：`mset`等

* 原生批量命令是原子性的，pipeline是非原子性的
* 原生批量命令依次只能执行一种命令，pipeline支持批量执行不同命令
* 原生批量命令是服务端实现，而pipeline需要服务端与客户端共同完成

<br />

## 管道和事务对比

* 事务具有原子性，管道不具有原子性

* 执行事务时会阻塞其他命令的执行，而执行管道中的命令不会

  >上面两点个人觉得表述有误，个人理解是事务保证了原子操作，不能加塞，而管道无法保证

* 管道一次性将多条命令发送到服务器，事务是一条一条的发，事务只有在接收到EXEC命令后才会执行，管道不会

<br />

## 注意事项

* 管道缓冲的指令只是会依次执行，不保证原子性，如果执行中指令发生异常，将会继续执行后续的指令
* 使用管道组装的命令不能太多，不然数据量过大客户端阻塞的时间可能过久，同时服务端此时也被迫回复一个队列答复，占用很多内存

<br />

# 发布订阅

>https://blog.csdn.net/weixin_50012581/article/details/132348982

<br />

# 主从复制

* 读写分离
* 容灾恢复
* 数据备份
* 水平扩容支撑高并发

<br />

## 基本操作

配从（库）不配主（库）

```yaml
replicaof 主库IP 主库端口 # 配置文件中配置主数据库
masterauth # 从机访问主机的通行密码，从机必须配置，主机不用
```

```shell
info replication # 可以查看复制节点的主从关系和配置信息
slaveof 主库IP 主库端口 # 命令配置主数据库。如果该数据库已经是某个主数据库的从数据库，那么会停止和原主数据库的同步关系，转而和新的主数据库同步
slaveof no one # 使当前数据库停止与其他数据库的同步，转为主数据库
```

<br />

## 注意事项

* 从机不可以执行写命令

* 从机意外宕机后，重启后仍有最新数据

* 主机意外宕机后，从机不动，原地待命，等待主机重启。期间从机数据可以正常使用

* 主机恢复后，主从关系不变，从机仍能正常复制

* 从机变更主机，会清除之前数据，重新拷贝最新的主机数据

* 从机变为主机，数据依旧存在，但不再同步

* 上一个slave可以是下一个slave的master，slave同样可以接收其他slaves的连接和同步请求，那么该slave作为了链条中下一个的master，可以有效的分载master的同步压力。

  但是，中间环节的slave仍然不支持写操作。

<br />

## 复制原理和工作流程

1. slave启动，同步初请

   slave启动成功连接到master后会发送一个sync命令。

   slave首次全新连接master，一次完成同步（全量同步）将被自动执行，slave自身原有数据会被master数据清除覆盖。

2. 首次连接，全量复制

   master节点收到sync命令后开始在后台保存快照（即RDB持久化，主从复制会触发RDB），同时收集所有接收到的用于修改数据集命令缓存起来，master节点执行RDB持久化后，将RDB快照文件和所有缓存的命令发送到所有slave，以完成一次完全同步。

   而slave在接收到数据库文件数据后，将其存盘并加载到内存中，从而完成复制初始化。

3. 心跳持续，保持通信

   master会持续发送ping包，保持通信。

   ```yaml
   repl-ping-replica-period 10
   ```

   默认是10s

4. 进入平稳，增量复制

   master继续将新的所有收集到的修改命令自动依次传给slave，完成同步。

5. 从机下线，重连续传

   master会检查backlog里面的offset（master和slave都会保存一个复制的offset还有一个masterId，offset是保存在backlog中的），master只会把已经复制的offset后面的数据复制给slave，类似断点续传。

<br />

## 缺点

* 由于所有的写操作都是在master上操作，然后同步更新到slave上，所以从master同步到slave有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，slave机器数量的增加也会使这个问题更加严重。
* 由于写操作只有在master机器上执行，如果master机器宕机，默认情况下，不会在slave节点中自动重选一个master，所有的写操作都没法完成。

<br />

# 哨兵监控

哨兵巡查监控后台master主机是否故障，如果故障了，根据`投票数`自动将某一个从库转换为新主库，继续对外服务。

哨兵的作用：

* 主从监控

  监控主从redis库运行是否正常

* 消息通知

  哨兵可以将故障转移的结果发送给客户端

* 故障转移

  如果master异常，则会进行主从切换，将其中一个slave作为新master

* 配置中心

  客户端通过连接哨兵来获得当前redis服务的主节点地址

<br />

## sentinel.conf

设置监控的master服务器

```yaml
sentinel monitor <master-name> <ip> <redis-port> <quorum>
sentinel auth-pass <master-name> <password>
```

* `quorum`表示最少有多少个哨兵认可客观下线，同意故障迁移的法定票数（确认客观下线的最少的哨兵数量）

  >我们知道网络是不可靠的，有时候一个sentinel会因为网络堵塞而误以为一个master redis已经宕机了，在sentinel集群环境下需要多个sentinel互相沟通来确认某个master是否真的死了，quorum这个参数是进行客观下线的一个依据，意思是至少quorum个sentinel认为这个master有故障，才会对这个master进行下线以及故障转移。这就保证了公平性和高可用。

其他配置

```yaml
# 指定多少毫秒后，主节点没有应答哨兵，此时哨兵主观认为主节点下线
sentinel down-after-milliseconds <master-name> <milliseconds>

# 表示允许并行同步的slave个数，当master挂了后，哨兵会选出新的master，此时剩余的slave会向新的master发起同步数据
sentinel parallel-syncs <master-name> <nums>

# 故障转移的超时时间，如果故障转移超过设置的毫秒，表示故障转移失败
sentinel fallover-timeout <master-name> <milliseconds>

# 配置当某一事件发生时所需要执行的脚本
sentinel notification-script <master-name> <script-path>

# 客户端重新配置主节点参数脚本
sentinel client-reconfig-script <master-name> <script-path>
```

>https://blog.csdn.net/m0_63947499/article/details/142059274

<br />

## 注意事项

* 主从关系发生变化后（主机宕机，从机上位），宕机的主机重连后不会变回主机，而是保持从机身份。
* 主从关系发生变化后，`sentinel.conf`、`redis.conf`配置文件都会重写。

<br />

## 运行流程和选举流程

* 主观下线：SDOWN（主观不可用）是单个sentinel自己主观上检测到的关于master的状态，从sentinel的角度来看，如果发送了PING心跳后，在一定时间内没有收到合法回复，就打到了SDOWN的条件。

  `down-after-milliseconds`配置可以设置主观下线的时间长度。

* 客观下线：ODOWN（客观不可用）需要一定数量的sentinel，多个哨兵达成一致意见才能认为一个master客观上不可用。

* 哨兵领导者选举：为了避免多个哨兵节点同时执行故障转移，造成混乱，哨兵集群需要从自己中选举一个哨兵领导者（leader），由它来负责故障转移（failover）的流程。哨兵领导者的选举是基于Raft算法或Paxos算法的。

* 新主节点选举：当哨兵领导者选举出来后，它会从所有的从节点中选择一个最合适的从节点，作为新的主节点。新主节点的选举标准是：优先级最高，复制偏移量最大，运行ID最小。

  * 优先级是通过配置文件指定的，越小表示优先级越高；

  * 复制偏移量表示从节点复制主节点数据的字节数，越大表示数据越完整；

  * 运行ID是Redis每次启动时生成的随机字符串，用于标识不同的Redis实例，按字典序比较大小。

* 重新配置从节点：当新主节点选举出来后，哨兵领导者会向所有的从节点发送命令，让它们与新主节点建立复制关系，更新自己的主节点信息。同时，哨兵领导者也会向所有的哨兵节点发送命令，让它们更新自己的主节点信息，并通知客户端使用新的主节点地址。

>https://blog.csdn.net/TaloyerG/article/details/134958242

<br />

# 集群分片

Redis 集群是一个可以在多个 Redis 节点之间进行数据共享的程序集。

* Redis集群支持多个Master，每个Master又可以挂载多个Slave
  * 读写分离
  * 数据高可用
  * 海量数据的读写存储操作
* 由于Cluster自带Sentinel的故障转移机制，内置了高可用的支持，无需再去使用哨兵功能
* 客户端与Redis的节点连接，不再需要连接集群中的所有节点，只需要任意连接集群中的一个可用节点即可
* 槽位slot负责分配到各个物理服务节点，由对应的集群来负责维护节点、插槽和数据之间的关系

<br />

## 概念

* 槽位

  Redis 集群没有使用一致性hash，而是引入了哈希槽的概念。

  Redis 集群由16384个哈希槽，每个key通过CRC16校验后对16384取模来决定放置哪个槽。集群的每个节点负责一部分哈希槽。

* 分片

  使用Redis集群时我们会将 存储的数据分散到多台Redis机器上，这称为分片。简言之，集群中的每个Redis实例都被认为是整个数据的一个分片。

优势：方便扩容/缩容和数据分片查找

<br />

## 映射算法

* 哈希取余算法
* 一致性哈希算法
* 哈希槽分区算法

>https://www.bilibili.com/video/BV13R4y1v7sP?p=79

<br />

## 为什么最大槽数是16384个？

>https://www.cnblogs.com/rjzheng/p/11430592.html

<br />
