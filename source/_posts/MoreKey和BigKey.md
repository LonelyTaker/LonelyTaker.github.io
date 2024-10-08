---
title: MoreKey和BigKey
date: 2024-10-08 17:18:34
categories: 后端
tags: Redis
---

# MoreKey

## 问题

`keys *`指令有致命的弊端，在实际生产环境中是禁用的。

>这个指令没有offset、limit参数，是要一次性返回所有满足条件的key，由于redis是单线程的，其所有操作都是原子的，而keys算法是遍历算法，复杂度是O(n)，如果实例中有千万级以上的key，这个指令就会导致Redis服务卡顿，所有读写Redis的其他的指令都会被延后甚至会超时报错，可能会引起缓存雪崩甚至数据库宕机。

<br />

## 规避

生产上如何限制`keys */flushdb/flushall`等危险命令以防止误删误用？

修改配置文件：

```yaml
########## SECURITY ##########
rename-command keys ""
rename-command flushdb ""
rename-command flushall ""
```

<br />

## 如何正确查询

### scan

与`scan`相关的指令有`sscan/hscan/zscan`

* `scan`命令用于迭代当前数据库中的数据库键

* `sscan`命令用于迭代集合键中的元素
* `hscan`命令用于迭代哈希键中的键值对
* `zscan`命令用于迭代有序集合中的元素（包括元素成员和元素分值）

<br />

```shell
SCAN cursor [MATCH pattern] [COUNT count]
```

* cursor：游标
* pattern：匹配的模式
* count：指定从数据集里返回多少元素，默认10

SCAN返回一个包含两个元素的数组

* 第一个元素是用于下一次迭代的新游标
* 第二个元素则是一个数组，这个数组中包含了所有被迭代的元素
* 如果新游标返回0表示迭代已结束

>https://redis.com.cn/commands/scan.html

<br />

# BigKey

bigkey是指key对应的value所占的内存空间比较大。

string类型控制在10kb以内，hash、list、set、zset元素个数不要超过5000，否则可以称之为bigkey。

<br />

## 问题

* 内存空间不均（不平衡），集群迁移困难
* 超时触发删除操作，由于Redis单线程，不仅耗时并且会占用资源，也就意味着Redis阻塞的可能性会增大
* 网络流量阻塞

<br />

## 如何发现

* `redis-cli --bigkeys`

  在连接Redis时加入`--bigkeys`参数。

  可以找到Redis实例中5种基本数据类型的最大key。

* `MEMORY USAGE key`

  >https://redis.com.cn/commands/memory-usage.html

<br />

## 如何删除

非字符串的bigkey，不要使用`del`删除，使用`sscan、hscan、zscan`方式渐进式删除，同时要注意防止bigkey过期时间自动删除问题（过期触发`del`操作会造成阻塞）

* string：一般使用`del`，如果过于庞大使用`unlink`
* hash：使用`hscan`每次获取少量field-value，再使用`hdel`删除每个field
* list：使用`ltrim`渐进式逐步删除，直到全部删除
* set：使用`sscan`每次获取部分元素，再使用`srem`命令删除每个元素
* lset：使用`zscan`每次获取部分元素，再使用`zremrangebyrank`命令删除每个元素

<br />

## LAZY FREEING

### 阻塞和非阻塞删除

Redis有两种方式删除键，一种是DEL，是对象的阻塞删除。这意味着服务器停止处理新命令，以便以同步方式回收和对象关联的所有内存。如果删除的键与一个小对象关联，该执行时间非常短。但是如果键与包含数百万个元素的集合值相关联，则服务器可能会阻塞很长时间才能完成操作。

基于上述原因，Redis还提供了非阻塞删除，例如UNLINK（非阻塞DEL）以及FLUSHALL和FLUSHDB命令的ASYNC选项，以便在后台回收内存。这些命令在恒定时间内执行，另一个线程将尽可能地逐步释放后台中的对象。

<br />

### 配置

由于其他操作的副作用，Redis服务器有时不得不删除键或刷新整个数据库，在某些场景下独立于用户调用删除对象，而默认的删除方式是阻塞式的，可以通过配置修改：

```yaml
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
# 下方几个配置可以改为yes，使用非阻塞方式
lazyfree-lazy-server-del no
replica-lazy-flush no
lazyfree-lazy-user-del no
```

