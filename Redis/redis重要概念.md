<!--
 * @Author: jhliang
 * @Descripttion: In User Settings Edit
 * @LastEditTime: 2021-07-10 17:18:22
 * @Date: 2021-07-10 11:23:16
 * @LastEditors: jhliang
 * @FilePath: \MyBook\Redis\redis重要概念.md
-->
# Redis

## Redis支持的数据结构

Redis的value可以是String，Map，list，Sets和SortedSets。
- 字符串（string）
- 哈希（hash）
- 列表（list）
- 集合（set）
- 有序集合（sorted set）
- 位图 ( Bitmaps )
- 基数统计 ( HyperLogLogs )

### 字符串（string）

```shell
SET name "redis.com.cn"  
OK  
GET name   
"redis.com.cn" 
```

### 哈希（hash）

```shell
HMSET key field1 value1 field2 value2 field3 value3
OK  
HGETALL key
"field1"  
"value1"  
"field2"  
"value2"  
"field3"  
"value3" 
HGET key field1
"value1"
```

### 列表（list）

```shell
LPUSH mylist java go c java python
(integer) 5
LINDEX mylist 1
"java"
LRANGE mylist 0 10
1) "python"
2) "java"
3) "c"
4) "go"
5) "java"
```

### 集合（set）

集合（set）是 Redis 数据库中的无序字符串集合。在 Redis 中，添加，删除和查找的时间复杂度是 O(1)。

```shell
SADD set java go c java python
(integer) 4
SMEMBERS set
1) "c"
2) "go"
3) "python"
4) "java"
```

### 有序集合（sorted set）

```shell
ZADD sortedset 2 java 3 go 1 c 4 python
(integer) 4
ZRANGEBYSCORE sortedset 0 10
1) "c"
2) "java"
3) "go"
4) "python"
```

### 位图 Redis Bitmap

Redis Bitmap 通过类似 map 结构存放 0 或 1 ( bit 位 ) 作为值。

Redis Bitmap 可以用来统计状态，如日活是否浏览过某个东西。

```shell
SETBIT bit4 2 1
(integer) 0
```

### 基数统计 HyperLogLogs

Redis HyperLogLog 可以接受多个元素作为输入，并给出输入元素的基数估算值

- 基数：集合中不同元素的数量，比如 {'apple', 'banana', 'cherry', 'banana', 'apple'} 的基数就是3。
- 估算值：算法给出的基数并不是精确的，可能会比实际稍微多一些或者稍微少一些，但会控制在合 理的范围之内。
  
HyperLogLog 的优点是：即使输入元素的数量或者体积非常非常大，计算基数所需的空间总是固定的、并且是很小的。

```shell
PFADD pf java go c java go
(integer) 1
PFCOUNT pf
(integer) 3 # java go c
```

## Redis的特点

- 高性能： Redis 将所有数据集存储在内存中，可以在入门级 Linux 机器中每秒写（SET）11 万次，读（GET）8.1 万次。Redis 支持 Pipelining 命令，可一次发送多条命令来提高吞吐率，减少通信延迟。
- 持久化：当所有数据都存在于内存中时，可以根据自上次保存以来经过的时间和/或更新次数，使用灵活的策略将更改异步保存在磁盘上。Redis 支持仅附加文件（AOF）持久化模式。
- 数据结构： Redis 支持各种类型的数据结构，例如字符串、散列、集合、列表、带有范围查询的有序集、位图、超级日志和带有半径查询的地理空间索引。
- 原子操作：处理不同数据类型的 Redis 操作是原子操作，因此可以安全地 SET 或 INCR 键，添加和删除集合中的元素等。
- 支持的语言： Redis 支持许多语言，如 C、C++、Erlang、Go、Haskell、Java、JavaScript（Node.js)、Lua、Objective-C、Perl、PHP、Python、R、Ruby、Rust、Scala、Smalltalk 等。
- 主/从复制： Redis 遵循非常简单快速的主/从复制。配置文件中只需要一行来设置它，而 Slave 在 Amazon EC2 实例上完成 10 MM key 集的初始同步只需要 21 秒。
- 分片： Redis 支持分片。与其他键值存储一样，跨多个 Redis 实例分发数据集非常容易。
- 可移植： Redis 是用 C 编写的，适用于大多数 POSIX 系统，如 Linux、BSD、Mac OS X、Solaris 等。
