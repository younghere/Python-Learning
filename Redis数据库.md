# Redis数据库默认端口：6379

**Remote Dictionary Server,通常称为数据结构服务器，c语言编写的，数据模型是key-value**

#### redis和memcache比较 

- Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，zset，hash等数据结构的存储。 
- Redis支持master-slave(主-从)模式应用 ：
  - 添加、修改、删除都叫“数据写入”，主服务器可以作为“写入”服务器，而从服务器可以作为 “读取”服务器。主服务器和从服务器要通过一些技术自动来同步。这叫“主从模式”。
- Redis支持数据持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。 
- Redis单个value的最大限制是1GB，memcached只能保存1MB的数据。



## Mac 安装redis

安装教程地址：https://blog.csdn.net/qq_21383435/article/details/80676497

```python
alias lre="sudo /usr/local/redis-4.0.10/bin/redis-server /usr/local/redis-4.0.10/etc/redis.conf"  		#运行redis服务器

#启动命令：
	redis-cli
    
#远程启动redis：
	redis-cli -h host -p port -a password
```



## Redis数据类型

redis的value支持五种数据类型：string(字符串)，hash(哈希)，list(列表)，set(集合)以及zset(sorted set:有序集合)



#### 键(key)

redis通过键命令来管理redis的键

```python
SET key_name value 
DEL key_name 		#删除key
EXISTS key 			#判断key是否存在
EXPIRE key seconds 	#给定key过期时间
RENAME key newkey 	#给key重命名
TYPE key			#返回key所储存的值的类型
```



#### String(字符串)

**string是redis最基本的数据类型**，一个key对应一个value，可以包含任何数据，比如图片或者序列化的对象

```python
SET key_name value	#设置key的值为value

GET key_name	#获取key的value		#一个键最大能储存512MB

GETRANGE key start end #返回 key 中字符串值的子字符

GETSET key value #将给定 key 的值设为 value ，并返回 key 的旧值(old value)

MGET key1 [key2..] #获取所有(一个或多个)给定 key 的值

STRLEN key #返回 key 所储存的字符串值的长度

MSET key value [key value ...]	#同时设置一个或多个 key-value 对

INCRBY key increment #将 key 所储存的值加上给定的增量值（increment）

DECRBY key decrement #key 所储存的值减去给定的减量值（decrement）

APPEND key value	#如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾
```



#### Hash(哈希)

一个键值(key=>value)对集合，hashes存的是**字符串**和**字符串值**之间的映射，比如一个用户要存储其全名、姓氏、年龄等等

```python
HMSET key_name field1 value1 field2 value2
HGET key_name field1
HGET key_name field2
HMGET key field1 [field2] #获取所有给定字段的值

#每个hash可以存储2^32-1键值对(40多亿)
HGETALL key_name   #获取key所有的field

HDEL key field1 [field2] 	#删除一个或多个哈希表字段

HEXISTS key field 	#查看哈希表 key 中，指定的字段是否存在

HINCRBY key field increment #为哈希表 key 中的指定字段的整数值加上增量 increment 

HKEYS key #获取所有哈希表中的字段

HLEN key #获取哈希表中字段的数量

HVALS key #获取哈希表中所有值

HSCAN key cursor [MATCH pattern] [COUNT count] #迭代哈希表中的键值对
```



#### List(列表)

按照插入的顺序排序，可以添加一个元素到列表的头部(左边)或者尾部(右边)，list在底层实现上并不是数组，而是链表。

```python
lpush lala I 	 #lala是创建的一个string
lpush lala AM
lpush lala very
lpush lala HAPPY
lrange lala 0 10  #列出lala键中的元素，包前包后，以0为第一个数的索引			
#列表最多可存储2^32-1元素

BLPOP LIST1 LIST2 .. LISTN TIMEOUT #如果列表为空，返回一个 nil 。 否则，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 key ，第二个元素是被弹出元素的值。

BRPOP key1 [key2 ] timeout #移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止

BRPOPLPUSH source destination timeout #从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止

LINDEX key index #通过索引获取列表中的元素

LINSERT key BEFORE|AFTER pivot value #将值 value 插入到列表 key 当中，位于值 pivot 之前或之后 

LLEN key #获取列表的长度

LPOP key #移出并获取列表的第一个元素

LPUSH key value1 [value2] #将一个或多个值插入到列表头部

LPUSH key value1 [value2] #将一个或多个值插入到列表头部，如果列表不存在，则报错

LPUSHX key value #将一个值插入到已存在的列表头部

LRANGE key start stop #获取列表指定范围内的元素

LREM KEY_NAME COUNT VALUE#根据参数 COUNT 的值，移除列表中与参数 VALUE 相等的元素

LSET key index value #通过索引设置列表元素的值
#L代表列表头，R代表列表尾

RPOP key #移除并获取列表最后一个元素

RPOPLPUSH SOURCE_KEY_NAME DESTINATION_KEY_NAME#移除列表的最后一个元素，并将该元素添加到另一个列表并返回
```



#### Set(集合)

Set是string类型的无序集合，集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是	o(1)

```python
SADD key member1 [member2] 		#向集合添加一个或多个成员，成功返回1，如果元素已经在集合中返回0，如果key对应的set不存在则返回错误

SCARD key #获取集合的成员数

SDIFF key1 [key2] #返回给定所有集合的差集，即key1中哪些元素是key2没有的

SINTER key1 [key2] #返回给定所有集合的交集

SINTERSTORE DESTINATION_KEY KEY KEY1..KEYN #将给定集合之间的交集存储在指定的集合中。如果指定的集合已经存在，则将其覆盖

SISMEMBER key member #判断member元素是否是集合 key 的成员

SMEMBERS key #返回集合中的所有成员

SMOVE source destination member #将 member 元素从 source 集合移动到 destination 集合

SREM key member1 [member2] #移除集合中一个或多个成员

SUNION key1 [key2] #返回所有给定集合的并集

SUNIONSTORE destination key1 [key2] #所有给定集合的并集存储在 destination 集合中

SSCAN key cursor [MATCH pattern] [COUNT count] #迭代集合中的元素
```



#### zset(sorted set:有序集合)

zset也是一个string类型元素的集合，不允许重复的成员

不同的是每个元素都会关联一个double类型的分数，通过分数来为集合中的成员进行**从小到大**的排序

zset的成员是惟一的，但是分数（score)却是可以重复的，score相同，比较添加的先后顺序

**zadd命令**

```python
ZADD key score1 member1 [score2 member2] #向有序集合添加一个或多个成员，或者更新已存在成员的分数

ZCARD key #获取有序集合的成员数

ZCOUNT key min max #计算在有序集合中指定区间分数的成员数

ZINCRBY key increment member #有序集合中对指定成员的分数加上增量 increment

ZINTERSTORE destination numkeys key [key ...] #计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中

ZRANGE key start stop [WITHSCORES] #通过索引区间返回有序集合成指定区间内的成员,0 -1指的是全部成员

ZLEXCOUNT key min max #在有序集合中计算指定字典区间内成员数量

ZRANK key member #返回有序集合中指定成员的索引

ZREM key member [member ...] #移除有序集合中的一个或多个成员

ZREVRANGE key start stop [WITHSCORES] #返回有序集中指定区间内的成员，通过索引，分数从高到底

ZSCORE key member #返回有序集中，成员的分数值

ZUNIONSTORE destination numkeys key [key ...] #计算给定的一个或多个有序集的并集，并存储在新的 key 中
```

#### HyperLogLog

**基数**：比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数

用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素

| 1    | [ PFADD key element [element ...]添加指定元素到 HyperLogLog 中。 |
| ---- | ------------------------------------------------------------ |
| 2    | [PFCOUNT key [key ...]返回给定 HyperLogLog 的基数估算值。    |
| 3    | [PFMERGE destkey sourcekey [sourcekey ...]将多个 HyperLogLog 合并为一个 HyperLogLog |



#### Redis发布订阅



#### Redis事务

- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

 单个redis命令的执行是原子性的，但是Redis事务的执行不是原子性的，事务可以理解为一个打包的批量执行脚本，中间单条指令的失败不会影响到其他的redis命令的执行。

| 序号 | 命令及描述                                                   |
| ---- | ------------------------------------------------------------ |
| 1    | [DISCARD](http://www.runoob.com/redis/transactions-discard.html)  取消事务，放弃执行事务块内的所有命令。 |
| 2    | [EXEC](http://www.runoob.com/redis/transactions-exec.html)  执行所有事务块内的命令。 |
| 3    | [MULTI](http://www.runoob.com/redis/transactions-multi.html)  标记一个事务块的开始。 |
| 4    | [UNWATCH](http://www.runoob.com/redis/transactions-unwatch.html)  取消 WATCH 命令对所有 key 的监视。 |
| 5    | [WATCH key [key ...\]](http://www.runoob.com/redis/transactions-watch.html)  监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。 |

#### Redis连接

| 序号 | 命令及描述                                                   |
| ---- | ------------------------------------------------------------ |
| 1    | [AUTH password](http://www.runoob.com/redis/connection-auth.html)  验证密码是否正确 |
| 2    | [ECHO message](http://www.runoob.com/redis/connection-echo.html)  打印字符串 |
| 3    | [PING](http://www.runoob.com/redis/connection-ping.html)  查看服务是否运行 |
| 4    | [QUIT](http://www.runoob.com/redis/connection-quit.html)  关闭当前连接 |
| 5    | [SELECT index](http://www.runoob.com/redis/connection-select.html)  切换到指定的数据库 |

#### redis缓存思路：

1. 执行查询
2. 缓存中存在数据 -> 查询缓存 
3. 缓存中不存在数据 -> 查询实时接口



#### redis持久化的两种方式

1. RDB(Redis DataBase):在不同的时间点，将redis存储的数据生成快照并存储到磁盘等介质上
2. AOF(Append Only File):将redis执行过的所有写指令记录下来，在下次redis重新启动时，只要把这些写指令从前到后再重复执行一遍，就可以实现数据恢复了



#### 详细的redis入门教程

https://blog.csdn.net/liqingtx/article/details/60330555



