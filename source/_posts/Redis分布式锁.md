---
title: Redis分布式锁
date: 2024-10-09 14:52:44
categories: 后端
tags: Redis
---

# CAP

分布式系统有三个指标：

-  Consistency（一致性） 
-  Availability（可用性） 
-  Partition tolerance （分区容错性）

在一个分布式系统（指互相连接并共享数据的节点的集合）中，当涉及读写操作时，只能保证一致性（Consistence）、可用性（Availability）、分区容错性（Partition Tolerance）三者中的两个，另外一个必须被牺牲。 

>https://cloud.tencent.com/developer/article/2355483

<br/>

# 分布式锁

## 分布式锁的刚需

* 独占性

  任何时刻只能有且只有一个线程持有

* 高可用

  * 集群环境下，不能因为一个节点宕机而出现获取锁和释放锁失败的情况
  * 高并发情况下，性能依旧良好

* 防死锁

  必须有超时控制机制或者撤销操作，有个兜底终止跳出方案

* 不乱抢

  不能私自释放其他线程获取的锁

* 重入性

  同一个节点的同一个线程如果获取锁之后，它也可以再次获取这个锁

<br/>

## 简单实现

```shell
# 获取锁
SETNX lock thread1
# 释放锁
DEL lock
```

问题：客户端宕机会导致锁一直未被释放，造成死锁。

优化：

```shell
# 获取锁
SETNX lock thread1
# 添加锁的过期时间，避免服务宕机引起的死锁
EXPIRE lock 10
# 释放锁
DEL lock
```

问题：由于`SETNX`和`EXPIRE`并非原子操作，可能存在刚`SETNX`完就宕机的情况，一样会导致死锁。

优化：

```shell
# 获取锁
SET lock thread1 EX 10 NX
# 释放锁
DEL lock
```

<br/>

## 防止误删

上方锁的简易实现中，还存在一定问题：

一般情况下，我们设置的超时时间需要大于业务代码的执行时间，但如果业务代码中出现了异常情况，导致执行完成时间大于锁的超时时间，这就会导致锁过期后被删除，其他线程抢占到锁，如果此时代码执行完了，要去释放锁，就会把其他线程占用的锁给释放了。

修改：

1. 在获取锁时存入线程标识（可以用UUID）
2. 在释放锁时先获取锁种的线程标识，判断是否与当前线程一致：
   * 如果一致则释放
   * 如果不一致则不释放

```shell
# 获取锁
SET lock thread1 EX 10 NX
# 查询锁的线程标识
GET lock
# 如果当前线程为thread1，释放锁
DEL lock
```

>上方为示例解释，线程标识判断需在业务代码中完成

<br/>

## 原子性问题

上方防止误删的优化代码仍有一些问题：

由于 判断线程标识一致性 和 释放锁 的操作不是原子性的，如果在判断线程标识一致性之后，发生了阻塞，此时锁过期被删除，另一个线程抢占到锁，当前线程阻塞结束再去释放锁，又会出现误删除问题。

修改：

* 使用Lua脚本解决多条命令原子性问题

  ```lua
  if redis.call("get",KEYS[1]) == ARGV[1] then
  	return redis.call("del",KEYS[1])
  else
  	return 0
  end
  ```

<br/>

前置知识：

Lua中调用Redis的方式

```lua
redis.call('命令名称', 'key', '其他参数', ...)
redis.call('set', 'name', 'jack')
```

Redis中调用脚本的方式

```shell
EVAL script numkeys key [key ...] arg [arg ...]
EVAL "return redis.call('set', 'name', 'jack')" 0
```

>后面的0是脚本需要的key类型的参数个数。
>
>作用是区分key数组和arg数组，例如`a b c d` ，假设numkeys是2，说明前2个参数a和b是key，后2个参数c和d是arg。

Redis调用脚本传参：

```shell
EVAL "return redis.call('set', KEYS[1], ARGV[1])" 1 name jack
```

<br/>

分布式锁的简单实现（Python）


 ```python
def acquire_lock_with_timeout(conn, lock_name, acquire_timeout=3, lock_timeout=2):
    """
    基于 Redis 实现的分布式锁
    :param conn: Redis 连接
    :param lock_name: 锁的名称
    :param acquire_timeout: 获取锁的超时时间，默认 3 秒
    :param lock_timeout: 锁的超时时间，默认 2 秒
    :return:
    """

    identifier = str(uuid.uuid4())
    lockname = f'lock:{lock_name}'
    lock_timeout = int(math.ceil(lock_timeout))
    end = time.time() + acquire_timeout
    while time.time() < end:
        # 如果不存在这个锁则加锁并设置过期时间，避免死锁
        if conn.set(lockname, identifier, ex=lock_timeout, nx=True):
            return identifier
        time.sleep(0.001)
    return False


def release_lock(conn, lock_name, identifier):
    """
    释放锁
    :param conn: Redis 连接
    :param lockname: 锁的名称
    :param identifier: 锁的标识
    :return:
    """

    unlock_script = """
    if redis.call("get",KEYS[1]) == ARGV[1] then
        return redis.call("del",KEYS[1])
    else
        return 0
    end
    """
    lockname = f'lock:{lock_name}'
    unlock = conn.register_script(unlock_script)
    result = unlock(keys=[lockname], args=[identifier])
    if result:
        return True
    else:
        return False
 ```

<br/>

## 其他问题

基于SETNX实现的分布式锁还存在其他一些问题：

* 不可重入

  同一个线程无法多次获取同一把锁

* 超时释放

  锁超时释放虽然可以避免死锁，但如果业务执行耗时较长，也会导致锁释放，存在安全隐患

* 主从一致性

  如果Redis提供了主从集群，主从同步存在延迟，当主机宕机时，如果从机来不及复制，新上位的主机没有锁数据，就有可能有多个线程获取到锁

现有的成熟工具集合：Redisson

>python可以使用python-redis-lock包，其内部也实现了锁的自动续约

