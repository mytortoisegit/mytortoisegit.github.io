---
title: redis学习笔记
date: 2024-06-13 15:00:45
tags: redis
---

# 再次学习笔记



Redis是高性能缓存数据库，高速缓存和消息队列代理。

它支持string[字符串](https://www.redis.net.cn/tutorial/3508.html)、hash[哈希表](https://www.redis.net.cn/tutorial/3509.html)、list[列表](https://www.redis.net.cn/tutorial/3510.html)、set[集合](https://www.redis.net.cn/tutorial/3511.html)、zset[有序集合](https://www.redis.net.cn/tutorial/3512.html)，[位图](https://www.redis.net.cn/tutorial/3508.html)，[hyperloglogs](https://www.redis.net.cn/tutorial/3513.html)等数据类型。

内置复制、[Lua脚本](https://www.redis.net.cn/tutorial/3516.html)、LRU收回、[事务](https://www.redis.net.cn/tutorial/3515.html)以及不同级别磁盘持久化功能，

同时通过Redis Sentinel提供高可用，通过Redis Cluster提供自动[分区](https://www.redis.net.cn/tutorial/3524.html)。 

redis在默认情况下 是单进程、单线程、单实例的。

redis特性：速度快，功能丰富，简单稳定，持久化，主从复制，高可用和分布式转移。

主要的5种数据结构，key 一般为简单字符串，符合业务，value比较丰富

key   ------value ---------------string

​				 set 

​				 get 

​			   ----------------hashes

​			   ---------------- lists

​			   ------------------sets

​			  -------------------sorted sets 

<img src="/images/1718595404719.png" alt="数据类型" />



三种方案实现用户信息存储优缺点

1.原生： set user:1:name zhangsan

  		set user:1:age  23

​		set user:1:gender boy

​		优点：简单直观，每个键对应一个值

​		缺点：键数过多，占内存多，用户信息分散，不利于生产

2.将对象序列化存入redis

​		set user:1  serialize(userInfo)

​		优点：编程简单，若使用序列化合理内存使用率高

​		缺点：序列化与反序列化有一定开销，更新属性时，需要把对象反序列化再序列化放进去

3.使用hash类型

​		hmset user:1 name zhangsan age 23 gender boy

​		优点：简单直观，使用合理可减少内存空间消耗

​		缺点：要控制ziplist与hashtable俩种编码转换，且hashtable会消耗更多内存

### String常见场景

​	1.计数功能：编号计数 incr   user:id

​	2.各类场景下的标识号:   incrby serialNo  1000

​	3.集群环境下Session共享 

#### String实现分布式锁 

### HASH常见场景

​		购物车记录：记录购物车id与商品id 以及采购数量， hmset cart:0001  prodId01 2 prodId02 2

​					获取所有购物车商品 hgetall cart:0001

​					查询一共有多少件商品 hlen cart:0001



### List常见场景-数据结构

​		使用list实现队列、栈

​				 队列：lpush   对应rpop

​				 栈：lpush对应lpop

​		订阅号记录：记录用户msg：0001  订阅的每个作者发布的文章id

​					lpush msg:0001   999

​				       lpush msg:0001   1000

​					查询所有订阅号中文章

​					lrange:  msg:0001 0 5

### 集合<set> 使用场景

​	点赞，标签，社交，查询共同兴趣爱好，智能推荐、抽奖

记录点赞信息：给用户user:01 点赞的人包括2、 3、 4、 5

​		           sadd  user:01:dianzan  user:2 user:3 user:4 user:5

​			 获取所有点赞的人smembers  user:01:zan

​			 获取点赞人个数  scard  user:002:zan

用户标签: 给用户添加标签

​		sadd  user:1:fav  basball  fball pq

​		sadd user:2:fav   basball  fball

​		给标签添加用户

​		sadd basball:users user:1  user:2

​		sadd fball:users  user:1 user:2

计算出共同感兴趣的人：

​		sinter user:1:fav  user:2:fav

抽奖：使用活动id作为key

​	  用户004参与活动

​	   sadd   active:001  004

​	  用户005参与活动

​	  sadd   active:001  005	

​	  sadd   active:001  001

​	  sadd   active:001  002

​	  sadd   active:001  003

抽奖操作： 抽2个人中奖 

​		抽完后，被抽到的人就移除

​		spop active:001 2 

​		srandmember active:001 2

朋友圈消息点赞设计：

​	集合与集合取交集

​		sinter setA setB

​	集合与集合取并集

​		sunion setA setB

​	集合与集合取差集

​		sdiff setA setB

​	示例：sadd zan:001 user001

​		   sadd zan:001 user005

​		   sadd zan:001 user002

​		   sadd zan:001 user004

​		   sadd zan:002 user005

​		   sadd zan:002 user001

微博微关系，共同关注，我关注的人他也关注XXX

<img src="/images/1718608742246.png" alt="数据类型" />





### 有序集合(ZSET): 

常用于排行榜，点赞数

<img src="/images/1718609100837.png" alt="作业" />







订单操作练习：

<img src="/images/1718348937509.png" alt="作业" /> 

缓存雪崩：当redis中某一key缓存失效了，大量的访问同时请求这个key时，都去访问数据库导致数据库挂掉

缓存穿透：查询某一个key时在redis中没有，数据库中也没有，被一直查询

### 持久化 

RDB把当前进程数据生成快照（.rdb）文件保存到硬盘的过程，有手动触发与自动触发。

AOF持久化，保存的是所有执行命令.

### redis重启时加载AOF与RDB的顺序：

<img src="/images/1718592500425.png" alt="加载" />



### redis性能测试

<img src="/images/1718677261520.png" alt="加载" />

```
100个并发连接 100000个请求  检测服务器性能
redis-benchmark -h 192.168.42.111 -p 6379 -a 9451a -c 100 -n 10000
测试存取大小为100字节的数据包的性能
redis-benchmark -h 192.168.42.111 -p 6379 -a 9451a  -q  -d 100
只测试set lpush 操作的性能
redis-benchmark -h 192.168.42.111 -p 6379 -a 9451a  -t set,get -n 100000 -q
只测试某些数值存取的性能
redis-benchmark -h 192.168.42.111 -p 6379 -a 9451a -n 100000 -q script load "redis.call('set','foo','bar')"
```

### reids事务

<img src="/images/1718679916072.png" alt="加载" />

### redis高可用

​		高可用意味着冗余，当一台服务器挂掉，可以自动切换到另一台服务器上，不影响客户使用

#### 	一、简单主从

​		缺点：主节点挂掉需要修改配置代码替换主节点

#### 	二、哨兵

​			1.哨兵必须是奇数个，每个哨兵都会监控redis服务端，

​			2.哨兵有三个定时监控任务完成对各个节点的发现和监控

​				1.每个哨兵都会每隔10秒对每个节点发送info收集节点信息

​				2.每个哨兵都会订阅主节点每隔2秒发送一次

​				3.每个哨兵都会每隔1秒发送一次ping

​			3.哨兵主观下线：单个哨兵发送ping超时后，这个哨兵会认为他挂了

​				   客观下线：如果2个以上哨兵发送ping超时后，认为他挂掉，则会进行灾备

​			4.哨兵选举流程：

​				 哨兵A发现某一个服务端有问题，会征求其他哨兵是否这个服务器有问题，如果其他哨兵也提

​				示有问题，哨兵A可能会成为哨兵leader,并处理故障转移。

​						自动处理，将原有的一个从节点变为主节点，将其他从节点匹配信息变更。