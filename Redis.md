该视频做的笔记：https://nyimac.gitee.io/2020/06/07/Redis%E5%AD%A6%E4%B9%A0%E6%96%87%E6%A1%A3/#
深入学习Redis后写的读书笔记：https://nyimac.gitee.io/2020/11/08/Redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0/

# Redis

## 五种基本数据类型

### 1.String

* 设置数据         

  `set key value`

  `mset key1 value1 key2 value2`

* 获取数据

  `get key`

  `mget key1 key2`

* 删除数据

  `del key`

* 设置生命周期

  `setex key seconds value`

  `psetex key milliseconds value`

* 向字符串的后面追加字符，如果有就补在后面，没有就新建

  `append key value`

* 增长和减少指令，只有当value为数字时才能增长

  `incr key`

  `incrby key increment`

  `incrbyfloat key increment`

  `decr key`

  `decrby key increment`

###2. hash

* 添加/修改数据

`hset key field value`

* 获取数据

  `hget key field`

​       `hgetall key`

* 删除数据

  `hdel key field`

* 添加/删除多个数据

  `hmset key field1 value1 field2 value2`

* 获取多个数据

  `hmset key field1 field2`

* 获取哈希表中字段的数量

  `hlen key`

* 获取哈希表中是否存在指定的字段

  `hexists key field`

**扩展操作**

* 获取哈希表中所有的字段名或字段值

  `hkeys key`

  `hvals key`

* 设置指定字段的数值增加指定范围的值

  `hincrby key field increment`

  `hincrbyfloat key field increment`



### 3.list

* 添加/修改数据

  `lpush key value1 {value2}`    从左边添加入链表

  `rpush key value1 {value2}`    从右边添加入链表

* 获取数据 

  `lrange key start stop`           起始为0，最后以为是-1

  `lindex key index`

  `llen key`

* 获取并移除数据

  `lpop key`

  `rpop key`

**扩展操作**

* 规定时间内获取并移除数据

  `blpop key1 {key2} timeout`    blpop list1 30

  `brpop key1 {key2} timeout`

* 移除指定数据

  `lrem key count value`  lrem 001 1 d



###4.set

* 添加数据

  `sadd key member1 {member2}`

* 获取全部数据

  `smembers key`

* 删除数据

  `srem key member1 {member2}`

* 获取集合数据总量

  `scard key`

* 判断集合中是否包含指定数据

  `sismember key member`

**扩展操作**

针对一些爱好猜想推荐商品的问题

* 随机获取集合中指定数量的数据

  `srandmember key count`

* 随机获取集合中的某个数据并将该数据移出集合

  `spop key`

* 求两个集合的交、并、差集

  `sinter key1 key2`

  `sunion key1 key2`

  `sdiff key1 key2`

* 求两个集合的交、并、差集并存储到指定集合中

  `sinterstore destination key1 {key2}`

  `sunionstore destination key1 {key2}`

  `sdiffstore destination key1 {key2}`

* 将指定书记从原始集合中移动到目标集合中去

  `smove source destination member`

###5.sorted_set     

###用于排序，优先级等业务

* 添加数据

  `zadd key score1 member1 {score2 member2}`    zadd scores 94 zhangsan     94是排序值

* 获取全部数据

  `zrange key start stop [withscores]`

  `zrevrange key start stop [withscores]`

* 删除数据

  `zrem key member [member ...]`

* 按条件获取数据

  `zrangebyscore key min max [withscores] [limit]`

  `zrevrangebyscore key min max [withscores] [limit]`

* 按条件删除数据

  `zremrangebyrank key start stop`

  `zremrangebyscore key min max`

* 获取集合数据总量

  `zcard key`

  `zcount key min max`

* 集合交、并操作

  `zinterstore destination numkeys key [key ...]`   zinterstore ss 3 s1 s2 s3 

  `zunionstore destination numkeys key [key ...]`    将s1,s2和s3中共有的属性求和放入ss中

sorted_set类型数据的扩展操作

* 获取数据对应的索引(排名)

  `zrank key member`

  `zrevrank key member`

* score值获取与修改

  `zscore key member`

  `zincrby key increment member`

* 获取当前系统时间

  `time`

##key的基本操作

* 删除指定key

  `del key`

* 判断key是否存在

  `exists key`

* 获取key的类型

  `type key`

**key的扩展操作**

* 为指定key设置有效期

  `expire key seconds`

  `pexpire key milliseconds`

  `expireat key timestamp`     linux指令，使用时间戳

  `pexpireat key milliseconds-timestamp`

* 获取key的有效时间

  `ttl key `            time to live     

  `pttl key`

  * XX：具有时效性的数据
  * -1：永久有效的数据
  * -2：已经过期的数据  或  被删除的数据 或未被定义的数据

* 将key从时效性转换为永久性

  `persist key`

查询操作

* 查询key

  `keys pattern`     *匹配任意数量的任意字符，？匹配任意一个字符，[]匹配区域中的一个字符

  例如 keys * 查询所有 keys str?  查询以str开头的字符 

* 为key改名

  `rename key newkey`

  `renamenx key newkey`

* 对所有key排序

  sort key(set,list or sorted_set)

* 对key的其它通用操作查询

  `help @generic`

  

##数据库的基本操作

* 切换数据库

  `select index`

在redis中数据库默认被分为0-15个数据库，不切换默认在0数据库中进行操作

* 退出程序

  `quit`

* 判断是否连接服务器

  `ping`          输入ping回车，响应pong显示连接

* `echo message`

* 数据在数据库中移动

  `move key db`           要确保目标数据库中没有该数据，否则移动失败

* 数据清除

  `flushdb`          清除当前数据库中数据

  `flushall`        清除所有数据库中数据

* 查看该数据库有多少个数据

  `dbsize`



------



##Redis高级应用之**持久化**

RDB、AOP（append only file）是Redis对数据持久化进行保存的一种方式  

RDB根据数据变化进行数据保存  AOP是记录操作变化

**RDB三种启动方式对比**

|      方式      | save指令 | bgsave指令 |   save配置   |
| :------------: | :------: | :--------: | :----------: |
|      读写      |   同步   |    异步    | 同bgsave一样 |
| 阻塞客户端指令 |    是    |     否     |              |
|  额外内存消耗  |    否    |     是     |              |
|   启动新进程   |    否    |     是     |              |



RDB缺点   1.因为是时间间隔进行保存，所以可能会丢失部分数据

​				  2.因为创建fork子进程，会降低CPU性能

​                  3.不同版本间RDB保存下来的数据不兼容。

AOF持久化策略：

always  、everysec 和no（系统管理）

always   每次写入操作都记录到AOF文件中，数据零误差，性能低。

everysec（默认采用）   每秒将缓冲区中的指令同步到AOF文件中，数据准确性较高，性能较高

​               在系统突然宕机的情况下丢失一秒内的数据

no(系统控制)   由操作系统控制每次同步到AOF文件的周期，整体过程不可控

​        

* 手动重写 `bgrewriteaof`

* 自动重写 `auto-aof-rewrite-min-size size`

  ​                 `auto-aof-rewrite-percentage percentage`

RDB VS AOF

|  持久化方式  |        RDB         |        AOF         |
| :----------: | :----------------: | :----------------: |
| 占用存储空间 | 小（数据级：压缩） | 大（指令级：重写） |
|   存储速度   |         慢         |         快         |
|   恢复速度   |         快         |         慢         |
|  数据安全性  |     会丢失数据     |    依据策略决定    |
|   资源消耗   |     高/重量级      |     低/轻量级      |
|  启动优先级  |         低         |         高         |



-----







##高级应用之**事务**

事务开启 

`multi`             

执行事务

`exec`

取消事务

`discard`

开启事务后不会立即执行，而是将所有命令添加到队列中，等待exec指令全部执行，或者是

discard全部取消

![](D:\software_adr\Snipaste\data_java\Redis.png)



Redis不提供事务回滚的功能，需要程序员手动回滚事务！！！假如事务中指令的格式没错误，只是执行出错，Redis会照常执行接下来的指令，不会回滚事务！



事务执行-----**锁**

* 对数据添加锁  
* 作用：对数据添加锁之后，如果事务执行之前，发现被监控的数据发生了变化，则终止事务

`watch key1 [key2 ...]`

* **取消所有锁**

`unwatch`



**分布式锁**

* 使用setnx设置一个公共锁

  `set lock-key` value

* 释放公共锁

  `del lock-key`

防止死锁

* 使用expire为锁key添加时间限定，到时间不释放，就会自动放弃锁

  `expire lock-key second`

  `pexpire lock-key millseconds`

--------

##高级应用之删除策略

三种删除方式

1. 定时删除 

   优点：节约内存，时间到点删除，快速删除释放不必要的内存数据

   缺点：CPU压力很大，无论CPU负载量多高，都会占用CPU，影响redis服务器的响应时间

   总结：用处理器性能换取存储空间（拿时间换空间）

2. 惰性删除

   优点：节约CPU性能，使用时发现过期直接删除

   缺点：内存压力大，会产生大量过期数据占用内存

   总结：用内存空间换取处理器性能

3. 定期删除

   折中方案，定期使用部分CPU去处理过期数据，性能高，内存压力小

----

##高级数据类型

###Bitmaps

通过对一个类似String的二进制数据进行操作，对每一位的1赋予给定的含义，使用时通过返回某一位的数值进行判断使用 。注意：假如对高进制位设置值时，会对未赋值的低进制位进行自动补0。

**Bitmaps基本操作：**

* 设置值

  `setbit key offset value`   

* 获取值

  `getbit key offset`           get bits 2  获取bits数据的第3位

**Bitmaps扩展操作:**

* 对指定key按位进行交、并、非、异或操作，并将结果保存到destKey中

  `bitop op destKey key1 [key2 ...]`

  * and:交
  * or:或
  * not:非
  * xor:异或

* 统计指定key中1的数量

  `bitcount key [start end]`

应用于信息的状态统计中

---

###HyperLogLog类型的基本操作

* 添加数据

  `pfadd key element [element ...]`

* 统计数据

  `pfcount key [key ...]`

* 合并数据

  `pfmerge destkey sourcekey [sourcekey]`

对数据进行去重操作，相同数据不会重复存储，适用于独立数据的统计

----

###GEO类型的基本操作

* 添加坐标

  `geoadd key longitude latitude [longtitude latitude]`

* 获取坐标点

  `geopos key member [member]`

* 计算坐标点之间的距离

  `geodist key member1 member2 [unit(单位)]`

* 根据坐标求范围内的其它数据

  `georadius key longtitude latitude radis m|km|ft|mi [withcoord] [withdist] [withhash] [count count]`

* 根据点求范围内的其它数据

  `georadiusbymember key member radius m|km|ft|mi [withcoord] [withdist] [withhash] [count count]`

* 获取指定点对应的坐标hash值

  `geohash key member [member ...]`

用于地理位置计算，例如百度地图，微信摇一摇

-----

##Redis集群之主从复制

![](D:\software_adr\Snipaste\data_java\Master&Slave.png)

----

一个master可以对应多个slave,一个slave对应一个master。

* 主从复制的三个阶段：
  1. 建立连接阶段
  2. 数据同步阶段
  3. 命令传播阶段

主从复制的工作流程：

1. 全量复制
2. 部分复制
   1. 复制id runid
   2. 复制缓存区（复制积压缓存区）
   3. 复制偏移量offset

3.心跳机制  

维护着日常数据传输交换
