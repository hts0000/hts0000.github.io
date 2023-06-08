# Redis必知必会


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# Redis

## Redis如何通过Key找到对应的Value

## Key的设计原则
- Key不宜太长，大于1024字节会显著影响Key的查找和匹配性能
- Key不宜太短，短Key能减少内存消耗和提升查找性能，需要在可读性和性能直接寻找平衡点
- Key采用`:`区分层级，如`user:1:info`，表示用户编号为1的信息，用`-`或`.`来连接多词，如`comment:4321:reply-to`或`comment:4321:reply.to`表示评论编号4321回复的对象
- Key最大允许的size为512MB

## Redis支持的数据结构
- 字符串(String)
- 列表(List)
- 集合(Set)
- 有序集合(Sorted Set)
- 哈希表(Hash)
- 数据流(Stream)
- 地理空间(Geospatial)
- 基数统计(HyperLogLog)
- 位图(Bitmaps)
- 位域(Bitfields)

## Redis命令执行的结果
执行结果的几种可能
- 成功，返回值1
- 失败，返回值0
- 错误，报错并打印错误信息

## 字符串操作
document: https://redis.io/commands/?group=string

字符串是`Redis`中绝大多数数据结构的底层类型

查看使用说明
```bash
help @string
```

增/改
```bash
SET user:1:info '{"username": "joy", "age": 12}'

# 后面加上EX选项，可以设置Key的过期时间，单位秒
SET user:1:info '{"username": "joy", "age": 12}' EX 10

# 后面加上PX选项，可以设置Key的过期时间，单位毫秒
SET user:1:info '{"username": "joy", "age": 12}' PX 1000

# 后面加上EXAT选项，可以设置Key的在指定的时间戳过期，单位秒
# 下面设置Key在2023-5-23 11:52:10过期
SET user:1:info '{"username": "joy", "age": 12}' EXAT 1684813930

# 后面加上PXAT选项，可以设置Key的在指定的时间戳过期，单位毫秒
# 下面设置Key在2023-05-23 11:53:53.364过期
SET user:1:info '{"username": "joy", "age": 12}' PXAT 1684814033364

# 后面加上NX选项，当Key不存在时创建
SET user:1:info '{"username": "joy", "age": 12}' NX EX 10

# 后面加上XX选项，当Key存在时覆盖
SET user:1:info '{"username": "joy", "age": 12}' XX EX 10

# 加上GET选项，当key存在时，返回旧值，不存在时返回nil
SET user:1:info '{"username": "joy", "age": 18}' GET XX EX 10

# KEEPTTL选项，在修改value时，保持其原有的过期时间
SET user:1:info '{"username": "bob", "age": 18}' GET XX KEEPTTL

# 设置多个KeyValue对
MSET user:1:age 18 user:2:age 19 user:3:age 20
```
查
```bash
GET user:1:info

# 获取多个Key的value
MGET user:1:info user:2:info user:3:info

# 检查Key是否存在，1-存在 0-不存在
EXISTS user:1:info

# 检查Key对应的类型
TYPE user:1:age

# 返回value的长度
STRLEN user:1:info
```
删
```bash
# 支持删除多个
DEL user:1:info user:2:info user:3:info
```
自增/增加任意大小
```bash
# 如果不存在这个key，先创建并设置value为0，然后自增1
# 如果value不为整数或者超过int64的范围，则报错
INCR views:page:2
INCRBY views:page:2 10
```
自减/减少任意大小
```bash
# 如果不存在这个key，先创建并设置value为0，然后自减1
# 如果value不为整数或者超过int64的范围，则报错
DECR views:page:2
DECRBY views:page:2 10
```

### 使用Redis实现分布式锁
Redis的SET命令有个选项是`NX`，表示当Key不存在时才创建，又因为Redis是单线程模型，同一时间点只有一条语句在执行，天然具有原子性，因此可以用来实现分布式锁。

我们用`SET lock-key lock-value NX`命令来创建一个`lock-key`表示一个锁，拿到锁的用户才能继续执行逻辑，拿不到锁的用户自旋等待获取锁。然而这种设计下，拿到锁的程序突然宕机，会形成死锁。

要解决这个问题我们可以加上一个过期时间，`SET lock-key lock-value NX EX 10`命令表示`lock-key`这个锁10秒后会自动释放，这样就避免了因获取到锁的程序宕机无法释放锁形成的死锁问题。

然而又有新的问题。如果过期时间到了，程序1仍然没有执行完主动释放锁，而程序2获取到锁执行的时候，程序1执行完了，然后去释放锁，程序3又获取到了锁，程序2执行完了释放锁。。。如此反复下去，锁根本无法保证数据安全。

要解决这个问题，我们可以给锁加上唯一标识，释放锁时判断锁还是不是自己的，如果不是就不执行释放。`SET lock-key unique-id NX EX 10`命令，表示`lock-key`这个锁由`unique-id`这个用户持有，释放锁时可以根据`unique-id`来判断锁是不是自己的。

然而判断锁是否是自己的，和释放锁是两条指令，不是原子操作。极端情况下，程序1判断锁是自己的，发送释放锁请求，如果释放锁请求到的比较晚，这个期间内锁到期被自动释放了，程序2获取到了锁，此时程序1释放锁的请求才到达，就会造成释放的锁是程序2的。

要解决这个问题，我们必须让判断锁和是否锁这两个操作是原子的。Redis中有事务和Lua脚本来保证多条指令的原子性。下面以Lua脚本为例。
```Lua
// 释放锁时，先比较 unique_id 是否相等，避免锁的误释放
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

然而还有问题。如果程序1没运行完，锁被自动释放了，程序2拿到锁开始运行，这不就变成两个程序并行执行了吗？并行情况下保证不了数据竞争啊。这种情况下，光靠Redis解决不了了，需要引入监控机制，如果持有锁的程序还在正常执行，那就不断更新过期时间，直到该程序执行完成或者宕机。`Redisson`框架就是Redis官方推荐的分布式锁方案。

上面谈论的还只限于单机情况，如果是Redis集群，涉及到集群间数据同步问题复杂度又上一个数量级。好在`Redisson`框架仍然支持集群，是一个非常成熟的分布式锁框架。

## 列表操作
document: https://redis.io/commands/?group=list

Redis的列表通过链表实现，插入时间O(1)

增加元素
```bash
# 从右边追加元素
rpush user:1:follower 4 5 6

# 从左边追加元素
lpush my-list 1 2 "hello" "done"
```
获取元素
```bash
# 获取列表长度
LLEN user:1:follower

# 获取全部元素，-1表示最后一个元素，闭区间
LRANGE user:1:follower 0 -1

# 获取指定范围的元素
LRANGE user:1:follower 0 3
LRANGE user:1:follower -5 -1

# pop操作，如果没有则元素返回nil
LPOP user:1:follower
RPOP user:1:follower

# 阻塞式的pop操作，没有元素时阻塞
# 如果给定多个列表，那么会顺序的从第一个非空列表中获取值
# timeout 表示阻塞时间，接受双精度的值，单位为秒，0表示无限
BLPOP list1 list2 ... timeout

# POPPUSH操作，Redis计划使用LMOVE指令代替RPOPLPUSH指令
# 把源列表——user:1:follower左边的元素POP，添加到目标列表——user:1:follower的右边
# 实现了一个循环队列的功能
# LMOVE指令还会返回POP出来的值
LMOVE user:1:follower user:1:follower LEFT RIGHT

# 阻塞式的LMOVE，当源列表没有元素时，阻塞
BLMOVE user:1:follower user:1:follower LEFT RIGHT timeout
```
删除元素
```bash
# 移除[0~3]这个闭区间之外的元素
LTRIM user:1:follower 0 3

# 移除等于value的元素
# count支持3种值
# count=0 表示查找整个列表，移除所有等于value的元素
# count>0 表示从列表头开始查找，移除count个等于value的元素
# count<0 表示从列表尾开始查找，移除count个等于value的元素
LREM user:1:follower count value
```

## 集合
集合是无序集合，Redis集合支持进行集合运算，比如：交集、并集和差集
```bash
# 新增元素
SADD user:1:follow 10 11 12
SADD user:2:follow 11 13 15

# 检查 user1 是否关注 11 号用户，返回1表示关注
SISMEMBER user:1:follow 11

# 获取 user1 和 user2 的共同关注
# 集合运算非常耗时，大集合进行操作时会阻塞Redis很长时间
SINTER user:1:follow user:2:follow

# 多个集合的交集存放到新集合dist中
SINTERSTORE dist user:1:follow user:2:follow user:3:follow

# 获取集合元素个数
SCARD user:1:follow

# 获取集合所有元素，一次性返回所有元素，结果集庞大
SMEMBERS user:1:follow

# 集合迭代器
# cursor 表示游标开始的位置，pattern 表示匹配值得表达式，支持Linux中的匹配，count 表示匹配几个值，默认10个
# 每次调用 SSCAN 命令会返回两个值
# 第一个值是游标下次开始的位置，第二个值是扫描到的值得列表
SSCAN user:1:follow cursor MATCH pattern COUNT count

# 删除集合内元素，成功删除至少一个元素返回1，无元素删除返回0
SREM user:1:follow 10 20 30

# 随机弹出count个元素
SPOP user:1:follow count

# 随机返回count个元素
SRANDMEMBER user:1:follow count
```

## 哈希
```bash
# 添加元素
HSET user:1:info username joy age 18 city LA email xxx@xx.com

# 获取元素
HGET user:1:info username
HMGET user:1:info username age

# 获取所有键值对
HGETALL user:1:info

# 获取所有键
HKEYS user:1:info

# 获取所有值
HVALS user:1:info

# 获取键值对个数
HLEN user:1:info

# 增加数值value的计数
# 给年龄增加10
HINCRBY user:1:info age 10

# 删除键值对
HDEL user:1:info email city
```

## 有序集合
有序集合类似于集合，但是支持给每个元素设置分数，根据分数来排序，如果分数一致，则根据元素的字典序排序
```bash
# 添加元素
ZADD players:rank 100 play1 97 play2 88 play3

# NX子选项，要添加的元素不存在才添加
ZADD players:rank NX 200 play4

# XX子选项，要添加的元素存在才更新
ZADD players:rank XX 150 play1

# LT子选项，要添加的元素存在且score小于200才更新
ZADD players:rank XX LT 200 play1

# GT子选项，要添加的元素存在且score大于100才更新
ZADD players:rank XX GT 100 play1

# 获取元素，基于排名
# 获取前三名的元素和其分数
ZRANGE players:rank 0 2 WITHSCORES

# 获取元素，基于排名
# 获取所有名次的元素和其分数
ZRANGE players:rank 0 -1 WITHSCORES

# 获取元素，基于分数
# 获取分数在 [100 ~ 200] 之间的元素及其分数
ZRANGE players:rank 100 200 BYSCORE WITHSCORES

# 获取元素，基于分数
# 获取分数在 [-inf ~ +inf] 之间的元素及其分数
ZRANGE players:rank -inf +inf BYSCORE WITHSCORES

# REV 子选项，可以反转排序
# 默认从小到大，反转为从大到小
ZRANGE players:rank 0 -1 REV WITHSCORES

# 删除元素
ZREM players:rank play1 play2

# 返回集合元素个数
ZCARD players:rank

# 并集计算(相同元素分值相加)
# numberkeys一共多少个key，WEIGHTS每个key对应的分值乘积
ZUNIONSTORE destkey numberkeys key [key...]

# 交集计算(相同元素分值相加)
# numberkeys一共多少个key，WEIGHTS每个key对应的分值乘积
ZINTERSTORE destkey numberkeys key [key...]
```

## 流
专门用于消息队列的数据结构，支持消息持久化，消息唯一id，消费确认，消费组，重复消费等功能。
```bash
# 往流中添加消息，*表示使用自动生成的id，也可以自己指定
# 添加成功返回消息id，自动生成的消息id已 '-' 分割可以分为两部分
# 前半部分为以毫秒为单位的当前服务器时间戳
# 后半部分为在当前毫秒的第几条数据，0表示当前毫秒的第0条数据
XADD user:1:messages * chat-username user2 chat-user-id 2 chat-user-icon http://chat-user-icon/user/2

# 使用NOMKSTREAM子选项，阻止流不存在时的自动创建
XADD user:2:messages NOMKSTREAM * chat-username user2 chat-user-id 2 chat-user-icon http://chat-user-icon/user/2

# 使用MAXLEN子选项，可以对流进行裁剪
# 如果使用 = 运算符，那么流的长度为min(流原始长度, 指定长度)
XADD user:2:messages MAXLEN = 10 * chat-username user2 chat-user-id 2 chat-user-icon http://chat-user-icon/user/2
# 如果使用 ~ 运算符，那么流的长度，会在保证性能的情况下尽可能>=指定长度
XADD user:2:messages MAXLEN ~ 10 * chat-username user2 chat-user-id 2 chat-user-icon http://chat-user-icon/user/2

# 使用MINID子选项，可以对流进行裁剪，MINID的裁剪基于消息ID进行裁剪
# 如果使用 = 运算符，那么流的最老id为min(流原始最老id, 指定id)
XADD user:2:messages MINID = 1684924529514-0 * chat-username user2 chat-user-id 2 chat-user-icon http://chat-user-icon/user/2

# 读取流中的数据
# 从指定id开始读取后面的所有数据，0表示从头开始
XREAD STREAMS user:2:messages 0

# 使用COUNT子选项，可以指定读取条数
XREAD COUNT 2 STREAMS user:2:messages 0

# 使用BLOCK子选择，可以阻塞指定毫秒时长，等待读取指定消息id之后的数据
XREAD BLOCK 100000 STREAMS user:2:messages 1684925552237-0

# 读取多个流的消息
# 阻塞100000ms，最多读取100条消息，从user:1:messages和user:2:messages两个流的最新消息开始读，'$' 表示最新消息
XREAD BLOCK 100000 COUNT 100 STREAMS user:1:messages user:2:messages $ $

# 读取流中的所有数据
XRANGE user:2:messages - +

# 使用COUNT子选项，实现迭代器的功能
# 第一次迭代，范围为[- ~ +]，取出第一条数据
# 第二次迭代，范围为(第一条数据id ~ +]，取出第二条数据
# Redis中用'('表示开区间
XRANGE user:2:messages (第一条数据id + COUNT 1

# 支持从某个时间点开始
# 因为数据id是：毫秒时间戳-序号 的格式，因此可以给定时间，来获取时间段内的数据
XRANGE user:2:messages 1684924529514 1684925302364 COUNT 10

# 获取单条数据，两个参数指定一样的数据id即可
XRANGE user:2:messages 1684925552237-0 1684925552237-0

# 流的长度
XLEN user:2:messages

# 删除数据
XDEL user:2:messages 1684925552237-0 1684925552237-1
```
消费组，创建一个从流指定id开始读取的组，组内可以有多个消费者，消费方式为`扇出(fan-out)`，也就是轮流消费。可以创建多个消费组，不同消费组内的消费者可以重复消费同一条数据。
```bash
# 创建一个从user:2:messages流的1684925552237-0数据id之后开始消费的消费组user:2:consumer-group:1
XGROYP CREATE user:2:messages user:2:consumer-group:1 1684925552237-0

# 创建一个从头开始消费的消费组
XGROYP CREATE user:2:messages user:2:consumer-group:1 0

# 创建一个从最后一条数据id之后开始消费的消费组
XGROUP CREATE user:2:messages user:2:consumer-group:1 $

# 消费者consumer:1从消费组中第一条未被消费的数据开始读取
# '>' 表示第一条未被消费的数据，也可以指定数据id
# COUNT子选项指定读取的条数，不指定则一直读取到最后
# 一个消费者消费完数据之后，这个消费组内的数据会被打上"已读取"的标志
# 其他消费这就无法再消费这些数据
XREADGROUP GROUP user:2:consumer-group:1 consumer:1 COUNT 10 STREAMS user:2:messages >

# BLOCK子选项，可以指定阻塞时长，单位ms
XREADGROUP GROUP user:2:consumer-group:1 consumer:1 COUNT 10 BLOCK 10000 STREAMS user:2:messages >

# NOACK子选项
# 消费组中的数据，被读取后会打上已读取的标志。
# 被标记为已读取的消息会被记录在该消费组的PENDING List中，并没有真正的删除。
# 要等到消费者显式的ACK，才会将PENDING List中该数据id删除。
# 这种设计是为了防止消费者读取了数据，但是在进行数据处理时程序崩溃了，数据并没有被真正的消费掉
# 程序恢复后还能够继续消费PENDING List中未ACK的数据
# NOACK子选项的作用是，消费者读取到数据，就立马ACK。使用于允许消息丢失的场景
XREADGROUP GROUP user:2:consumer-group:1 consumer:1 COUNT 10 NOACK STREAMS user:2:messages >

# ACK消息已消费
# user:2:messages流的user:2:consumer-group:1组中给定的数据id已消费
# 这些数据将会从PENDING List中被删除
XACK user:2:messages user:2:consumer-group:1 1684925552237-0 1684925552237-1 1684925552237-2

# 查看PENDING List中未ACK的数据，还能看到是哪个消费者读取了这条数据
XPENDING user:2:messages user:2:consumer-group:1 - + 10
```

## 地理空间
专门用于存储和计算地理坐标的数据结构。存储经纬度坐标，计算并给出指定范围内的坐标有哪些
```bash
# 存储坐标
GEOADD locations:ca NX -122.27652 37.805186 station:1 -122.2674626 37.8062344 station:2

# 计算给定经纬度坐标5km半径内的所有坐标，还支持m为单位
# WITHDIST指示返回坐标距离中心点的距离
# WITHCOORD指示返回搜索到的坐标的经纬度
# ASC|DESC指示按照离给定坐标为中心的距离进行排序，ASC从近到远，DESC反之
GEOSEARCH locations:ca FROMLONLAT -122.2612767 37.7936847 BYRADIUS 5 km ASC WITHDIST WITHCOORD

# 计算给定已存储坐标500m半径内的所有坐标
GEOSEARCH locations:ca FROMMEMBER station:2 BYRADIUS 500 m WITHDIST

# BYBOX子选项，支持在给定长宽的矩形内进行搜索
GEOSEARCH locations:ca FROMMEMBER station:2 BYBOX 400 400 km WITHDIST

# 查询指定坐标的经纬度
GEOPOS locations:ca station:1 station:2

# 查询两个坐标之间的km距离，支持m
GEODIST locations:ca station:1 station:2 KM
```

## 位图
```bash
# 设置值
SETBIT users:login-status 10002 1

# 获取值
GETBIT users:login-status 10002

# 统计值为1的个数，[0 ~ -1]表示全区间
BITCOUNT users:login-status 0 -1

# 统计第一个0/1出现的位置，支持指定区间[0 ~ -1]
# 返回的定位总是从0开始算的绝对定位
# BIT子选项表示以BIT位来计数，100 200 BIT 表示[100 ~ 200]这个区间内第一个0/1出现的位置
# 还支持BYTE以字节来计数，BYTE为默认选项
BITPOS users:login-status 1 100 200 BIT
```

## 设置Key的过期时间
`EXPIRE`指令可以给存在的Key设置过期时间，如果Key不存在或者为空，那么直接返回0
```bash
# 给列表设置10.10秒过期时间
LPUSH users "bob" "joy" "may"
EXPIRE users 10.10

# [NX | XX | GT | LT] 子选项
# NX 子选项表示当Key没有过期时间时，设置才生效
# XX 子选项表示当Key没有过期时，设置才生效
# GT 子选项新过期时间大于Key过期时间时，设置才生效
# LT 子选项新过期时间小于Key过期时间时，设置才生效
```

## 位图、布隆过滤器和布谷鸟过滤器
位图只能存储数值，布隆过滤器则可以存储字符串

布隆过滤器是把要存储的值进行多次Hash取模，映射到位图的某个bit位。下次检查该值是否存在时，同样进行多次Hash取模，如果每一个bit位都存在，则该值可能存在，反之则一定不存在。

布隆过滤器可以用较少的空间来过滤值是否存在，缺点是存在误判率，而且值不能删除，使用时间长了之后，过滤器中大多数位置都被填充上了值，误判率上升。

布隆过滤器在Redis中以Module的形式存在，需要单独安装，下载地址: https://redis.io/resources/modules/

## Redis用途
- 缓存：单纯作为缓存使用，减少DB的压力
- 消息中间件：使用Pub/Sub功能作为消息中间件使用
- 内存数据库：把Redis当作数据库使用，需要配合AOF和RDB的数据持久化，以及哨兵或集群来实现

## Key删除策略
### 过期删除策略
过期删除策略是当Key达到过期时间后，如何对过期的Key进行删除的策略。
#### 定时删除
Key过期时间一到就删除。
- 优点：对内存友好，存活的Key都是有用的
- 缺点：对CPU不友好，频繁的执行删除任务，特别是内存还很空闲而CPU繁忙时，有大批量的删除任务

#### 惰性删除
不主动删除，当访问该Key过期时，将其删除。
- 优点：对CPU友好，只会占用很少的CPU资源
- 缺点：对内存不友好，很多过期Key无法及时删除，如果一个Key过期后永远不访问了，会造成内存泄漏

#### 定期删除
每隔一段时间删除过期Key，配置文件中默认配置为`hz 10`，表示每隔10s取出一批量的Key，删除其中过期的。Redis保证了删除这一批量的过期Key不超过25ms，防止线程卡顿
- 优点：平均了内存负载和CPU负载
- 缺点：两头都不占优

#### Redis过期删除策略
Redis选择「惰性删除+定期删除」这两种策略配和使用，以求在合理使用 CPU 时间和避免内存浪费之间取得平衡

### 内存淘汰策略
内存淘汰策略是当Redis使用的内存超过配置文件设置的最大内存时，如何选择淘汰哪些Key的策略。

Redis默认内存淘汰策略为不淘汰，最大允许使用内存为：
- 32位系统——>3G
- 64位系统——>无限制

**设置Redis最大允许使用内存**：配置文件中配置`maxmemory 4GB/4096MB`
```bash
# 查看已使用的内存
INFO MEMORY

# 查看最大允许使用内存
CONFIG GET MAXMEMORY

# 查看内存淘汰策略
CONFIG GET MAXMEMORY-POLICY
```
Redis支持的所有内存淘汰策略
- noeviction：不淘汰，默认的淘汰策略
- volatile-random：在设置了过期时间的键值中，随机淘汰任意键值
- volatile-ttl：在设置了过期时间的键值中，优先淘汰更早过期的键值
- volatile-lru：在设置了过期时间的键值中，淘汰最久未使用的键值
- volatile-lfu：在设置了过期时间的键值中，淘汰最少使用的键值
- allkeys-random：不关心是否设置过期时间，随机淘汰任意键值
- allkeys-lru：不关心是否设置过期时间，淘汰最久未使用的键值
- allkeys-lfu：不关心是否设置过期时间，淘汰最少使用的键值

**设置Redis内存淘汰策略**：配置文件中配置`maxmemory-policy allkeys-lfu`

## 阿里云Redis产品选型
分为社区版和企业版。社区版完全兼容开源Redis，企业版增强了很多能力和性能。

产品可选：
- 标准架构：主从架构
- 集群架构：Redis的集群模式，数据分片，高可用，主从自动切换，使用于读写高QPS场景
- 读写分离架构：写节点1个，读节点多个，只读节点采用链式复制，使用于读多写少场景

## Redis事务

## 数据持久化、恢复及迁移
### 数据持久化
#### AOF
只追加(Append Only File)，在Redis执行每一条写操作后，将该操作的命令追加到文件中，恢复时就读取并执行该文件的命令，这样就能够完全恢复Redis的数据。

**启用AOF**

AOF持久化默认不开启，在配置文件中添加如下配置，开启AOF
```bash
dir /var/lib/redis
appendonly yes
appenddirname appendonlydir
appendfilename appendonly.aof
# always|everysec|no
appendfsync everysec
```
**AOF的三种回写机制**
- always：写操作完成同步写盘，最大限度保证数据不丢，性能差
- everysec：写操作完成将写指令写入缓冲区，每秒落一次盘，性能适中，可能丢失1s的数据
- no：写操作完成将写指令写入缓冲区，由操作系统控制合适将缓冲区刷入磁盘，性能好，可能丢失很多数据

**AOF重写机制**  
官方文档：https://redis.io/docs/management/persistence/

AOF因为所有写指令都会写入文件，随着时间文件会越来越大，AOF文件过大会导致恢复Redis时时间过长。

Redis提供AOF重写机制，当达到设定的重写阈值(auto-aof-rewrite-min-size 64mb)时，触发重写任务，来压缩AOF文件。

Redis7.0之前的重写任务会读取当前所有的Key，将所有Key和其Value写入到新的AOF文件，等到全部Key记录完成，替换之前的AOF文件，这个替换是原子操作。先写入新文件是为了避免重写失败，造成对原有文件的破坏。

Redis7.0之后的重写任务，主进程会新开一个`.incr`AOF文件继续执行写入，子进程执行重写生成新的`.base`快照文件，写入完成后会执行原子性的替换操作。

AOF重写任务是非常耗时的，所以是非阻塞的，会启用一个新的进程来执行重写任务。

使用进程而非线程的原因在于，线程之间是共享内存的，在修改内存数据时，需要加锁保护，而进程的共享内存是只读的，如果修改会发生写时复制，生成新的副本。

执行`BGREWRITEAOF`命令来主动触发AOF重写程序。

**AOF文件格式**

Redis7.0之前AOF是以单文件的形式存在的，7.0之后改成了多AOF文件，这是不向下兼容的。总共有三类AOF文件：
- appendonly.aof.1.base.rdb/aof：代表初始快照，格式可能是AOF或者RDB的
- appendonly.aof.1.incr.aof：表示从初始快照开始的增量变化aof文件，该文件可能有多个
- appendonly.aof.manifest：以上的文件可以分开存放，manifest清单用于追踪他们的位置

**AOF文件修复**
1. 追加命令被截断了  
如果因为某些原因(比如磁盘满了)，导致AOF文件追加写入时失败了，只写入了半截的命令。可以尝试执行`redis-check-aof --fix <filename>`命令生成自动修复的AOF文件，再使用`diff`命令比较两个文件的差异。

2. 文件损坏了  
执行`redis-check-aof --fix <filename>`命令查看损坏的地方，尝试手动修复。
#### RDB
RDB文件是快照文件，将Redis当前内存存储的数据用二进制的方式存储下来，RDB文件相比AOF文件更加紧凑，只存储了当前数据的最终状态，因此使用RDB文件来恢复数据是非常快的。

**启用RDB**

RDB默认配置为：`save 3600 1 300 100 60 10000`，其含义如下
- 3600 1：3600s内至少有1个数据改变，就保存快照
- 300 100：300s内是少有100个数据改变，就保存快照
- 60 10000：60s内至少有10000个数据改变，就保存快照

如果希望关闭快照，使用配置：`save ""`

**RDB快照生成流程**
1. 基于Redis进程生成子进程
2. 子进程将数据写入临时RDB文件
3. 子进程完成数据写入后将新文件替换旧RDB文件

子进程写入时同样使用到了写入复制技术。

**主动执行快照生成**

主动保存快照文件有两个命令，`SAVE`和`BGSAVE`，他们的区别如下：
- `SAVE`：同步的保存，会阻塞主进程执行
- `BGSAVE`：后台子进程执行保存，不会阻塞主进程

### 数据恢复
如果想要建立一个当前Redis实例的副本，应该使用主从的方式建立。当然也可以使用当前实例的RDB文件来快速启动一个非同步的副本。

### 数据迁移
拷贝RDB或AOF文件来迁移。

## 高可用方案
### 主从
主从能完成数据备份的功能。

从库与主库建立联系，可以在配置文件中加上：`replicaof 192.168.1.1 6379`命令，也可以直接在命令行中执行。主从建立流程和具体步骤如下：
1. 从服务器给主服务器发送`psync ? -1`命令，表示进行数据同步  
因为是第一次同步，不知道主服务器的runID，所以用`?`代替，-1则表示同步点
2. 主服务器响应`FULLRESYNC <runID> <offset>`  
响应内容包括主服务器`runID`和偏移量
3. 主服务器执行`BGSAVE`命令后台生成RDB文件，这期间的写操作同时记录在buffer中
4. RDB文件生成后发送给从服务器，从服务器清空自己的数据，使用RDB文件来重建
5. 从服务器回复重建完成，主服务器将buffer中的写指令发送给从服务器，第一次同步完成，建立长连接
6. 后面的每次同步都通过这个长连接将buffer中的数据同步过去，这个过程是异步的

**增量复制**

如果长连接断开了，需要重连，重连期间的数据如何同步？Redis2.8之后的版本采用的是增量复制。

从服务器执行`psync <runID> <offset>`命令，这次会带上曾经记录的服务器id和同步偏移量，从偏移位置开始增量同步。

之前说过主服务器要同步的指令会先写在buffer中，再异步同步过去。而这个buffer——repl_backlog_buffer缓冲区的大小是有限制的，默认1mb，他是一个环形缓冲区。也就是说超过1mb的部分会循环覆盖旧数据，如果要断开的时间太长，要恢复的offset以及被覆盖了，那就会进行全量同步。为了避免这种情况，应该调大repl_backlog_buffer配置的值。

主从架构是没有自动切换的功能的，自动切换需要使用哨兵。

### 哨兵
哨兵在主从的基础上增加故障切换的功能。基于分布式共识算法的高可用方案。

哨兵服务已经集成在redis-server二进制文件中，使用`redis-server /path/to/sentinel.conf --sentinel`命令来启动一个哨兵。哨兵默认运行端口为`26379`。一个最小的配置文件如下：
```bash
# 该哨兵将会监听mymaster这个master是否异常，
# 如果2个哨兵认为其异常了，将会切换到从节点
# 这里并没有配置其他哨兵节点和Redis从节点的信息，
# 这是因为哨兵会通过Pub/Sub能力，从__sentinel__:hello这个Channel中获取其他哨兵的信息，所以mymaster这个名字必须是唯一的，以便服务发现时多个哨兵达成共识建立通信
# 而主从服务器的信息，则会主动与主服务器通信获取，这些配置将会自动的保存到配置文件中，也就是说这些配置都是实时获取的
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1

# 该哨兵将会监听resque这个master是否异常，
# 如果4个哨兵认为其异常了，将会切换到从节点
sentinel monitor resque 192.168.1.3 6380 4
sentinel down-after-milliseconds resque 10000
sentinel failover-timeout resque 180000
sentinel parallel-syncs resque 5
```

**主从如何切换**

首先哨兵如何知道节点真的挂掉了？这里有两个概念：
- `主观下线`：当前哨兵节点认为其下线了
- `客观下线`：哨兵集群超过半数认为其下线了

所有哨兵节点每隔1s与主从节点发一次心跳包确认其存活，超过`down-after-milliseconds`这个配置规定的时间未回复的节点，该哨兵就将其标记为`主观下线`。此时有两种情况，1.该节点真的挂了无法回复，2.哨兵节点与该节点的网络出问题了，为了避免第二种情况导致的误判，所以需要`客观下线`。

当一个哨兵认为该节点`主观下线`了，就会向其他哨兵发起投票，判断该节点是否下线，如果多数都认为该节点`主观下线`了，那么就会将该节点标记为`客观下线`。这里的多数是可以在配置文件中配置的，称之为`quorum`。

确定该主节点下线之后，需要从其从节点中选择一个升级为主节点。如何选择合适的从节点？有如下考量：
1. 历史网络情况：检查每个从节点主观下线的次数，超过10次的将会被过滤掉
2. 优先级：如果从节点配置了优先级`slave-priority`，那么优先级小的将会被选择
3. 复制进度：优先级一样的情况下，谁复制的进度更快选择谁
4. 节点id：都一样则选择节点id小的那个

选择好主节点了，还需要通知其他从节点切换主节点。哨兵会向所有从节点发送`SLAVEOF`命令，切换主节点。

最后通过Pub/Sub通知客户端发生了主从切换事件，客户端可以通过Sub该Channel来完成事件监听，以便与告警。其实不止主从切换事件，主观下线等事件都可以监听到。事件列表如下：

![](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E5%93%A8%E5%85%B5/%E5%93%A8%E5%85%B5%E9%A2%91%E9%81%93.webp)

旧主节点重新恢复了，重新上线后哨兵会将其恢复为从节点。

**哨兵集群的高可用**

哨兵集群的高可用使用分布式算法来保证，需要保证哨兵集群为奇数个，以便于投票时不发生平票而导致脑裂。哨兵集群要高可用起码需要三个节点，最多容忍(n-1)/2个节点宕机。

### 集群
集群在前面两者的基础上通过数据分片实现横向扩展的功能。

`slot`，称之为插槽，Redis集群最重要的概念。一个最小的Redis集群由三主三从组成，所有写入集群的Key将会经过Hash，分配到16384个slot上，而每个主节点将会分配到等量的slot，以达到均衡写入提高写入性能的目的。16384个slot，意味着Redis集群最多支持16384个主从节点。

如果某个主节点宕机了，那么他的从节点将会升级为主节点。

**横向扩展**

横向扩展，将一个新的主从节点加入集群，需要执行称之为`Rehard(重新分片)`的操作，来保证各个节点的负载均衡。

### 读写分离架构
阿里云上的读写分离架构，本地盘方案采用链式复制，数据同步存在延迟，数据可能不一致，如果要求数据一致，那么应该使用云盘方案的星型复制，或者使用集群。

## 可观测能力
### 监控指标
#### 大Key监控
大Key指的是某个Key对应的Value很大，单线程的Redis读取或操作这个Key时时间过长，影响了其他请求的执行，导致总体QPS下降。

如果在集群中遇到大Key问题，还会发生数据倾斜的现象，大Key所在的节点CPU、内存和带宽的负载明显比其他节点要高。

监控大Key可以有如下方法
- redis-cli --bigkeys：列出各个数据结构Top1的大Key
- 离线分析RDB文件：使用[redis-rdb-tools](https://github.com/sripathikrishnan/redis-rdb-tools?spm=a2c4g.443809.0.0.61e328d7K8mOdA)工具离线扫描RDB持久化文件，对线上性能无影响
- TopKey：阿里云上支持实时获取大Key和热Key

**如何解决大Key**
- 拆分：将存储在一个Hash中的成员，分散存储到多个Hash中
- 删除：清除大Key应该使用`UNLINK <key>`命令，该命令是非阻塞式的

#### 查询历史热Key
热Key指的是某个Key的访问量非常高，其占用了大量的QPS、CPU时间和带宽。热Key同样会导致其他请求的QPS下降，还会发生访问倾斜的现象。如果Key太热，超过Redis实例处理能力，还会出现缓存击穿甚至超卖的情况。

热Key会出现上述问题，原因在于读请求的数量，超过了Redis实例的处理能力，也就是服务器的性能不够。

- redis-cli --hotkeys：列出hotkey
- TopKey：阿里云上支持实时获取大Key和热Key

**如何解决大Key**
- 提高服务器性能：升级硬件，使用更好的CPU、内存
- 架构升级：使用读写分离或者集群架构，提升整体架构处理请求的能力

#### 监控是否发生数据倾斜
监控集群各个节点同一时间段CPU、内存和带宽使用量是否平衡。

### 日志

### 访问量倾斜
某个节点的访问量或QPS明显高于其他节点，导致该节点的CPU使用率和带宽使用率高于其他节点
### 数据量倾斜
某个节点的Key大小大于其他节点的Key，就说明发生数据量倾斜，大多少没有个具体的值，一般是大于20%就认为发生倾斜了。

发生倾斜的原因一般是大Key 热Key或者一些高消耗的命令O(n)复杂度的命令，或者是使用了错误的Key和Hash方式，导致过多的Key被分配到同一节点上

## 缓存击穿 缓存穿透 缓存雪崩

## Benchmark
Redis-benchmark
## 禁用高危命令

## 调优
### 参数调优
https://help.aliyun.com/document_detail/98726.html?spm=a2c4g.65001.0.nextDoc.55804630W2a2ik
### 内核调优
