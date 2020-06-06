---
title: redis
date: 2019-06-03 11:29:50
tags: redis
---

&emsp;&emsp;redis学习之路。

<!-- more -->

### 简单配置

&emsp;&emsp;使用yum安装的redis，他的配置文件的路径为`/etc/redis.conf`，修改此配置文件以满足我们的要求。

- 绑定IP

```bash
bind 10.15.159.19 127.0.0.1
```

&emsp;&emsp;bind可以绑定多个IP地址，中间用空格分隔，也可以将bind注释掉。注意：<font color="#dd0000">你只能通过bind的IP来访问redis服务</font>。将bind注释掉则可以通过任意IP访问redis服务。

- 端口号

```bash
port 6379
```

- 数据库数量

```bash
databases 16
```

- 连接密码(默认不需要)

```bash
requirepass foobared
```

- 查看 redis 当前使用的配置文件目录

```bash
redis-cli info | grep config
```

### redis 数据结构

&emsp;&emsp;redis 一共有五种数据结构，分别为String、List、hash、Set、Zset。

&emsp;&emsp;原子操作：在多进程（线程）访问共享资源时，能够确保所有其他的进程（线程）都不在同一时间内访问相同的资源。原子操作（atomic operation）是不需要synchronized，这是Java多线程编程的老生常谈了。所谓原子操作是指不会被线程调度机制打断的操作；这种操作一旦开始，就一直运行到结束，中间不会有任何 context switch （切换到另一个线程）。

#### 字符串类型(String)

&emsp;&emsp;空间分配原则：当len小于IMB（1024*1024）时增加字符串分配空间大小为原来的2倍，当len大于等于1M时每次分配 额外多分配1M的空间。取字符串的长度的时间复杂度为O(1)

- 赋值(set key value)

````bash
set hello world
````

- 取值(get key)

```bash
get hello
```

- 自增(incr key)(原子操作)

```bash
incr num
```

- 自减(decr key)(原子操作)

```bash
decr num
```

&emsp;&emsp;自增/自减该key对应的value值都会相应的加/减1，若该key不存在，则先`set key 0`然后执行加/减1操作，最后将执行结果返回。如果该key的值不是数字则报错。

- 自增N(incrby key increment)(原子操作)

```bash
incrby count 5
```

- 自减N(decrby key increment)(原子操作)

```bash
decrby count 5
```

&emsp;&emsp;自增N/自减N同自增/自减。

- 增加浮点数(incrbyfloat key increment)

```bash
incrbyfloat float1 3
```

- 向尾部追加(append key value)

```bash
append hello !
```

&emsp;&emsp;向字符串尾部追加内容，返回追加后的字符串长度，若key不存在，则新增。

- 字符串长度(strlen key)

```bash
strlen hello
```

- 同时设置多个key(mset key value [key value ...])

```bash
mset str1 hehe str2 xixi str3 haha
```

- 同时获取多个key的值(mget key [key ...])

```bash
mget str1 str2 str3
```

#### 列表(list)

&emsp;&emsp;列表类型存储了一个有序的字符串列表。常用的操作是向两端插入新的元素。时间复杂度为 O（1）。结构为一个链表。记录头和尾的地址。看到这里，Redis 数据类型的列表类型一个重大的作用呼之欲出，那就是队列。新来的请求插入到尾部，新处理过的从头部删除。列表类型就是一个下标从 0 开始的数组。由于是链表存储，那么越靠近头和尾的元素操作越快，越靠近中间则越慢.

- 向头部插入(lpush key value [value ...])

```bash
lpush list apple banana pear
```

&emsp;&emsp;返回列表的长度。向头部插入，先向头部插入apple，再向头部插入banana，最后向头部插入pear，所以操作后列表的第一个元素应该是pear。

- 向尾部插入(rpush key value [value])

```bash
rpush list orange grape
```

&emsp;&emsp;返回列表的长度。向尾部插入，orange，grape，所以操作后列表的最后一个元素应该是grape。

- 列表元素个数(llen key)

```bash
llen list
```

&emsp;&emsp;返回列表的长度。若key不存在，返回0。

- 获取列表的子列表(lrange key start end)

```bash
lrange list 0 -1
```

&emsp;&emsp;返回第 start 个到第 end 个元素的列表。包含 start 和 end。支持负数索引。-1 表示最后一个元素，-2 表示倒数第二个元素。

- 获取指定索引值(lindex key index)

```bash
lindex list 0
```

&emsp;&emsp;若index为负数则从尾部向前索引，-1为最后一个元素的位置。

- 从头部弹出(lpop key)

```bash
lpop list
```

&emsp;&emsp;返回被弹出的元素。

- 从尾部弹出(rpop key)

```bash
rpop list
```

&emsp;&emsp;返回被弹出的元素。

- 删除列表中指定值(lrem key count value)

```bash
lrem list 1 apple
```

&emsp;&emsp;返回删除的数量。count为0，删除所有列表中指定的值；count为正数，从前往后删除count个指定的元素(count大于实际的元素数量时删除所有)；count为负数时，从后往前删除count个指定的元素(count大于实际的元素数量时删除所有)。

- 设置索引和值(lset key index value)

```bash
rpush fruit apple pear banana orange
lset fruit 0 "greet apple"
lrange fruit 0 -1
```

- 保留片段，删除其它(ltrim key start end)

```bash
ltrim fruit 0 -2
```

&emsp;&emsp;保留start到end之间的所有元素，含start和end。其他全部删除。舒服从末尾向前，-1表示最后一个元素的下标值。

- 向列表插入元素(linsert key before|after pivot value)

```bash
linsert fruit before pear apple
linsert fruit after pear apple
```

&emsp;&emsp;从头部开始遍历，直到发现第一个pivot，根据before|after将value插入pivot之前还是之后。

- 把一个列表的一个元素转到另一个列表(rpoplpush source destination)(原子操作)

```bash
rpoplpush fruit list
```

&emsp;&emsp;将source列表的最后一个元素弹出并使用lpush操作插入到destination列表中。

#### 散列类型(hash)

&emsp;&emsp;Redis 是以字典（关联数组）的形式存储的，一个 key 对应一个 value。在字符串类型中，value 只能是一个字符串。那么在散列类型中，value 对应的也是一个字典（关联数组）。那么就可以理解，Redis 的散列类型中，key 对应的 value 是一个二维数组。但是字段的值只可以是字符串。也就是说只能是二维数组，不能有更多的维度。

- 赋值(hset key field value)

```bash
hset map name ferry
```

- 取值(hget key field)

```bash
hget map name
```

- 同一个key多个字段赋值(hmset key field value [field value ...])

```bash
hmset map birthday 1995-04-09 sex male
```

- 同一个KEY多个字段取值(hmget key field [field ...])

```bash
hmget map name birthday sex
```

- 获取KEY的所有字段和所有值(hgetall key)

```bash
hgetall map
```

- 字段是否存在(hexists key field)(存在返回1，否则返回0)

```bash
hexists map name
hexists map age
```

- 当字段不存在时赋值(hsetnx key field value)(原子操作)

```bash
hsetnx map height 175
hsetnx map name jack
```

&emsp;&emsp;若字段存在返回0，否则设置字段的值并返回1。等效于hexists+hset，但该操作是原子操作，线程安全的。

- 自增N(hincrby key field increment)

```bash
hincrby map height 2
```

- 删除字段(hdel key field [field ...])

```bash
hdel map height
```

- 只获取字段名(hkeys key)

```bash
hkeys map
```

- 只获取字段值(hvals key)

```bash
hvals map
```

- 获取字段数量(hlen key)

```bash
hlen map
```

#### 集合类型(set)

&emsp;&emsp;redis的集合和列表都可以存储多个字符串，它们之间的不同在于，列表可以存储多个相同的字符串，而集合则通过使用散列表（hashtable）来保证自已存储的每个字符串都是各不相同的(这些散列表只有键，但没有与键相关联的值)，redis中的集合是无序的。

- 增加(sadd key member [member ...])

```bash
sadd s 1 a 2 b 3 c
```

- 删除(srem key member [member ...])

```bash
srem s 1 2 3
```

- 获取集合所有元素(smembers key)

```bash
smembers s
```

- 判断指定集合中是否存在某个元素(sismember key member)

```bash
sismember s a
```

- 差集运算(sdiff key [key ...])

```bash
sadd b 1 2 3 a b c
sdiff a b
sdiff b a
```

- 交集运算(sinter key [key ...])

```bash
sinter s b
```

- 并集运算(sunion key [key ...])

```bash
sunion s b
```

- 差集/交集/并集运算结果存储在一个集合中(sdiffstore/sinterstore/sunionstore destination key [key ...])

```bash
sdiffstore bs b s
sinterstore sbs b s
sunionstore sb s b
```

- 随机获取元素(srandmember key [count])

```bash
srandmember b -10
```

&emsp&emsp;若count不存在，随机从集合中获取一个元素输出；若count等于0，输出空；若count大于0，输出不重复的count个元素（count大于实际数量时输出集合中所有元素）；若count小于0，输出可重复的count个元素。

- 随机弹出一个元素(spop key [count])

```bash
spop b 10
```

&emsp&emsp;若count不存在，随机从集合中弹出一个元素并输出该元素；若count等于0，弹出0个元素并输出空；若count大于0，弹出不重复的count个元素（count大于实际数量时弹出集合中所有元素）并输出这些被弹出的元素；若count小于0，报错`(error) ERR index out of range`。

#### 有序集合(zset)

&emsp;&emsp;有序集合和散列一样，都用于存储键值对：有序集合的键被称为成员（member),每个成员都是各不相同的。有序集合的值则被称为分值（score），分值必须为浮点数。有序集合是redis里面唯一一个既可以根据成员访问元素(这一点和散列一样),又可以根据分值以及分值的排列顺序访问元素的结构。

- 增加(zadd key [NX|XX] [CH] [INCR] score member [score member ...])

```bash
zadd z 1 a 2 b 3 c 4 d 5 e 6 f 7 g
```

- 获取分数(zscore key member)

```bash
zscore z e
```

- 获取排名在某个范围内的元素列表，闭区间(zrange key start stop [withscores])

```bash
zrange z 0 10 withscores
```

&emsp;&emsp;若withscores参数不存在，则按顺序输出范围内的元素，若withscores参数存在，则一行元素，一行分值按顺序输出范围内的元素-分值。若start/stop为-1，则取顺序是最后的元素-分值；为-2，则取顺序是倒数第二的元素-分值；以此类推。

- 获取指定分数范围的元素列表，闭区间,limit参数分页(zrangebyscore key min max [withscores] [limit offset count])

```bash
zrangebyscore z 5 10 withscores limit 1 2
```

- 增加元素分值(zincrby key increment member)

```bash
zincrby z 8 v
```

- 获取元素数量(zcard key)

```bash
zcard z
```

- 获取分值范围内的元素个数(zcount key min max)

```bash
zcount z 0 10
```

- 删除元素(zrem key member [member ...])

```bash
zrem z v z
```

- 删除指定排名范围内的元素(zremrangebyrank key start stop)

```bash
zremrangebyrank z 0 10
```

- 删除指定分值范围内的元素(zremrangebyscore key min max)

```bash
zremrangebyscore z 2 10
```

- 获取元素正序排名(zrank key member)

```bash
zrank z f
```

- 获取元素倒序排名(zrevrank key member)

```bash
zrevrank z a
```



