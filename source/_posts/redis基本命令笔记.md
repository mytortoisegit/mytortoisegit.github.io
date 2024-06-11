---
title: redis基本命令笔记
date: 2024-06-11 18:01:06
tags: redis基础
---

Redis 提供了一系列强大的原生命令，用于操作其数据结构。这些命令按照操作的数据类型和功能可以分类如下：

### 1. 通用命令

- **PING**：测试连接是否正常。
- **ECHO message**：打印消息。
- **SELECT index**：选择数据库。
- **FLUSHDB**：清空当前数据库中的所有数据。
- **FLUSHALL**：清空所有数据库中的所有数据。
- **DBSIZE**：返回当前数据库的键数量。
- **EXISTS key**：检查键是否存在。
- **DEL key**：删除一个或多个键。
- **TYPE key**：返回键的值类型。
- **RENAME key newkey**：重命名键。
- **KEYS pattern**：查找所有符合给定模式的键。
- **MOVE key db**：将键移动到指定的数据库。
- **EXPIRE key seconds**：为键设置过期时间。
- **TTL key**：获取键的剩余生存时间。
- **PERSIST key**：移除键的过期时间。
- **DUMP key**：序列化给定键，并返回被序列化的值。
- **RESTORE key ttl serialized-value**：反序列化给定值，并将它们放到指定的键上。

### 2. 字符串（String）命令

- **SET key value**：设置指定键的值。
- **GET key**：获取指定键的值。
- **INCR key**：将键的值加1。
- **DECR key**：将键的值减1。
- **MGET key1 [key2 ...]**：获取所有给定键的值。
- **MSET key1 value1 [key2 value2 ...]**：同时设置多个键的值。
- **APPEND key value**：将值追加到指定键的值之后。
- **STRLEN key**：获取指定键值的长度。

### 3. 哈希（Hash）命令

- **HSET key field value**：设置哈希表字段的值。
- **HGET key field**：获取哈希表字段的值。
- **HDEL key field [field ...]**：删除一个或多个哈希表字段。
- **HEXISTS key field**：检查哈希表字段是否存在。
- **HGETALL key**：获取哈希表中的所有字段和值。
- **HLEN key**：获取哈希表字段的数量。
- **HMGET key field1 [field2 ...]**：获取所有给定字段的值。
- **HMSET key field1 value1 [field2 value2 ...]**：同时设置多个字段的值。
- **HINCRBY key field increment**：为哈希表字段的值加上指定增量。

### 4. 列表（List）命令

- **LPUSH key value [value ...]**：将一个或多个值插入到列表头部。
- **RPUSH key value [value ...]**：将一个或多个值插入到列表尾部。
- **LPOP key**：移除并返回列表的头元素。
- **RPOP key**：移除并返回列表的尾元素。
- **LLEN key**：获取列表长度。
- **LRANGE key start stop**：获取列表指定范围内的元素。
- **LSET key index value**：设置列表中指定索引的元素的值。
- **LREM key count value**：移除列表中与指定值相等的元素。

### 5. 集合（Set）命令

- **SADD key member [member ...]**：向集合添加一个或多个成员。
- **SREM key member [member ...]**：移除集合中的一个或多个成员。
- **SMEMBERS key**：获取集合中的所有成员。
- **SISMEMBER key member**：判断成员是否在集合中。
- **SCARD key**：获取集合的成员数。
- **SUNION key [key ...]**：返回所有给定集合的并集。
- **SINTER key [key ...]**：返回所有给定集合的交集。
- **SDIFF key [key ...]**：返回第一个集合与其他集合的差集。

### 6. 有序集合（Sorted Set）命令

- **ZADD key score member [score member ...]**：向有序集合添加一个或多个成员，或者更新已存在成员的分数。
- **ZREM key member [member ...]**：移除有序集合中的一个或多个成员。
- **ZRANGE key start stop [WITHSCORES]**：按索引范围获取有序集合的成员。
- **ZRANK key member**：返回有序集合中成员的排名。
- **ZINCRBY key increment member**：有序集合中对指定成员的分数加上增量。
- **ZCARD key**：获取有序集合的成员数。
- **ZCOUNT key min max**：计算在有序集合中指定分数区间内的成员数量。

### 7. 事务（Transaction）命令

- **MULTI**：标记一个事务块的开始。
- **EXEC**：执行所有事务块内的命令。
- **DISCARD**：取消事务块内的所有命令。
- **WATCH key [key ...]**：监视一个或多个键，如果这些键被修改，事务将被中止。
- **UNWATCH**：取消 WATCH 命令对所有键的监视。

### 8. 发布/订阅（Pub/Sub）命令

- **PUBLISH channel message**：将消息发送到指定频道。
- **SUBSCRIBE channel [channel ...]**：订阅一个或多个频道。
- **UNSUBSCRIBE [channel ...]**：退订一个或多个频道。
- **PSUBSCRIBE pattern [pattern ...]**：订阅一个或多个模式匹配的频道。
- **PUNSUBSCRIBE [pattern ...]**：退订一个或多个模式匹配的频道。

### 9. 脚本（Scripting）命令

- **EVAL script numkeys key [key ...] arg [arg ...]**：执行 Lua 脚本。
- **EVALSHA sha1 numkeys key [key ...] arg [arg ...]**：执行缓存的 Lua 脚本。
- **SCRIPT LOAD script**：将脚本加载到脚本缓存中，但不执行。
- **SCRIPT FLUSH**：清除所有已缓存的脚本。
- **SCRIPT EXISTS sha1 [sha1 ...]**：检查是否缓存了给定的脚本。

### 10. 连接（Connection）命令

- **AUTH password**：认证密码。
- **QUIT**：关闭连接。
- **INFO [section]**：获取服务器的信息和统计数据。
- **CONFIG GET parameter**：获取配置参数的值。
- **CONFIG SET parameter value**：设置配置参数的值。
- **MONITOR**：实时监控服务器的所有请求。

这些命令只是 Redis 提供的丰富命令集的一部分。不同的命令可以根据具体的应用场景灵活组合使用，以实现高效的数据存储和访问。完整的命令列表和详细说明可以参考 Redis 官方文档。