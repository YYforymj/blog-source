---
title: The Redis Little Book
date: 2018-09-11 20:28:30
categories: redis

---

> 以下内容为 The Redis Little Book 的学习笔记。

<!-- more -->

## 基础

选择数据库，选中编号为 number 的数据库

```
db: select（number）
```

key 组成形式： users:leto （可以理解为名为 leto 的用户，冒号为分隔符用于管理键值）

存储值

```
set users:leto '{"name": "leto", "planet": "dune", "likes": ["spice"]}'
```

获取值

```
get users:leto
```

操作的数据类型根据命令区分，不同的命令操作不同的数据类型。

## 数据结构

- Strings ：字符串，最常用类型

  存储

  ```
  set users:leto '{"name": leto, "planet": dune, "likes": ["spice"]}'
  ```

  获取长度

  ```
  strlen users:leto
  (integer) 50
  ```

  获取子字符串

  ```
  getrange users:leto 31 48
  "\"likes\": [\"spice\"]"
  ```

  追加字符串

  ```
  append users:leto " OVER 9000!!"
  (integer) 62
  ```

  如果字符串可以解析为数字，如

  ```
  set number 1
  ```

  可以直接对数字值进行操作

  ```
  incr number
  (integer) 2
  
  incrby number 3
  (integer) 5
  ```

- Hashes :比较接近类的概念

  存取、属性操作

  ```
  hset users:goku powerlevel 9000
  
  hget users:goku powerlevel
  9000
  
  //一次设置多个属性
  hmset users:goku race saiyan age 737
  
  //一次获取多个属性
  hmget users:goku race powerlevel
  1) "saiyan"
  2) "9000"
  
  //获取所有属性与值
  hgetall users:goku
  1) "powerlevel"
  2) "9000"
  3) "race"
  4) "saiyan"
  5) "age"
  6) "737"
  
  //获取所有属性名称
  hkeys users:goku
  1) "powerlevel"
  2) "race"
  3) "age"
  
  //删除属性
  hdel users:goku age
  ```

- Lists: 类似于队列或者栈，可以 push、pop 操作

  ```
  lpush newusers goku
  ```

- Sets: 类似于 java 中的 set，存储不重复不排序的集合

  存储

  ```
  sadd friends:leto ghanima paul chani jessica
  sadd friends:duncan paul jessica alia
  ```

  查询集合中是否存在

  ```
  sismember friends:leto jessica
  (integer) 1//1为存在
  
  sismember friends:leto vladimir
  (integer) 0//0为不存在
  ```

  查找交集

  ```
  sinter friends:leto friends:duncan
  1) "jessica"
  2) "paul"
  ```

  查找交集并存储

  ```
  sinterstore friends:leto_duncan friends:leto friends:duncan
  (integer) 2
  
  smembers friends:leto_duncan
  1) "jessica"
  2) "paul"
  ```

- Sorted Sets: 介于hashes 与 sets 之间的类型，相当于内部值附加有权值的sets

  存储

  ```
  zadd friends:duncan 70 ghanima 95 paul 95 chani 75 jessica 1 vladimir
  (integer) 5
  ```

  获取区间内值的数量

  ```
  zcount friends:duncan 90 100
  (integer) 2
  ```

  获取某个元素的排名（最高排名为0）

  ```
  zrevrank friends:duncan vladimir
  (integer) 4
  
  zrevrank friends:duncan paul
  (integer) 0
  
  zrevrank friends:duncan chani
  (integer) 1
  ```

## 使用数据结构

- 利用 hash 进行伪多列索引，存储数据：

  ```
  set users:9001 '{"id": 9001, "email": "leto@dune.gov", ...}'
  hset users:lookup:email leto@dune.gov 9001
  ```

  获取数据(Ruby)：

  ```
  id = redis.hget('users:lookup:email', 'leto@dune.gov')
  user = redis.get("users:#{id}")
  ```

- pipeline 类似于批处理batch，在多条数据操作时能够提升效率

- 事务：redis 的命令都是原子操作（因为 redis 是单线程的）。、

  ```
  multi
  hincrby groups:1percent balance -9000000000
  hincrby groups:99percent balance 9000000000
  exec
  ```

  依次输入上述命令即实现了一个事务，关键命令为 `multi` 与 `exec`。事务开始的时候先向Redis服务器发送 MULTI 命令，然后依次发送需要在本次事务中处理的命令，最后再发送 EXEC 命令表示事务命令结束。当输入MULTI命令后，服务器返回OK表示事务开始成功，然后依次输入需要在本次事务中执行的所有命令，每次输入一个命令服务器并不会马上执行，而是返回”QUEUED”，这表示命令已经被服务器接受并且暂时保存起来，最后输入EXEC命令后，本次事务中的所有命令才会被依次执行。如果客户端在发送EXEC命令之前断线了，则服务器会清空事务队列，事务中的所有命令都不会被执行。而一旦客户端发送了EXEC命令之后，事务中的所有命令都会被执行，即使此后客户端断线也没关系，因为服务器已经保存了事务中的所有命令。

  事务错误处理：

  - **语法错误** 就像上面的例子一样，语法错误表示命令不存在或者参数错误
    这种情况需要区分Redis的版本，Redis 2.6.5之前的版本会忽略错误的命令，执行其他正确的命令，2.6.5之后的版本会忽略这个事务中的所有命令，都不执行，就比如上面的例子(使用的Redis版本是2.8的)

  - **运行错误** 运行错误表示命令在执行过程中出现错误，比如用GET命令获取一个散列表类型的键值。
    这种错误在命令执行之前Redis是无法发现的，所以在事务里这样的命令会被Redis接受并执行。如果食物里有一条命令执行错误，其他命令依旧会执行（包括出错之后的命令）。

> 1. Redis服务端是个单线程的架构，不同的Client虽然看似可以同时保持连接，但发出去的命令是序列化执行的，这在通常的数据库理论下是最高级别的隔离
> 2.  用MULTI/EXEC 来把多个命令组装成一次发送，达到原子性
> 3.  用WATCH提供的乐观锁功能，在你EXEC的那一刻，如果被WATCH的键发生过改动，则MULTI到EXEC之间的指令全部不执行，不需要rollback
> 4.  其他回答中提到的DISCARD指令只是用来撤销EXEC之前被暂存的指令，并不是回滚
>
> 作者：顾小宇
> 链接：https://www.zhihu.com/question/35949129/answer/81798934
> 来源：知乎
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

## 一些其他指令

- 过期时间

  ```
  expire pages:about 30  //30s后过期
  expireat pages:about 1356933600  //按照时间戳设置过期
  ttl pages:about //剩余过期时间
  persist pages:about //设置key取消过期
  ```

- 发布与订阅

  先打开一个 redis client ，输入如下指令

  ```
  subscribe warnings //订阅消息
  //下面为输出
  Reading messages... (press Ctrl-C to quit)
  1) "subscribe"
  2) "warnings"
  3) (integer) 1
  ```

  再打开另一个 redis client ，输入如下指令

  ```
  publish warnings "it's over 9000!" //发布消息
  (integer) 1
  ```

  此时可以再第一个 client 中观察到如下输出，实现了发布订阅

  ```
  1) "message"
  2) "warnings"
  3) "it's over 9000!"
  ```

- 监控

  先打开一个 redis client ，输入如下指令

  ```
  monitor
  ```

  再打开另一个 redis client ，输入如下指令

  ```
  set my "my own redis"
  ```

  第一个 client 中出现如下输出，实现了对 redis 的监控

  ```
  1536649818.773050 [0 127.0.0.1:56539] "COMMAND"
  1536649833.069995 [0 127.0.0.1:56539] "set" "my" "my own redis"
  ```

  慢日志

  redis的slow log记录了那些执行时间超过规定时长的请求。执行时间不包括I/O操作（比如与客户端进行网络通信等），只是命令的实际执行时间（期间线程会被阻塞，无法服务于其它请求）。 
  有两个参数用于配置slow log： 

  - slowlog-log-slower-than：设定执行时间，单位是毫秒，执行时长超过该时间的命令将会被记入log。-1表示不记录slow log; 0强制记录所有命令。

  ```
  config set slowlog-log-slower-than 0
  ```

  - slowlog-max-len：slow log的长度。最小值为0。如果日志队列已超出最大长度，则最早的记录会被从队列中清除。 
    可以通过编辑redis.conf文件配置以上两个参数。对运行中的redis, 可以通过config get, config set命令动态改变上述两个参数

  slow log是记录在内存中的，所以即使你记录所有的命令（将slowlog-log-slower-than设为0），对性能的影响也很小。 

  - slowlog get: 列出所有slow log 
  - slowlog get N:列出最近N条slow log

- 排序

  对 list 进行排序

  ```
  127.0.0.1:6379> rpush users:leto:guesses 5 9 10 2 4 10 19 2
  (integer) 8
  127.0.0.1:6379> sort users:leto:guesses
  1) "2"
  2) "2"
  3) "4"
  4) "5"
  5) "9"
  6) "10"
  7) "10"
  8) "19"
  ```

  值得注意的是，sort 仅仅是对数据排序后进行输出，原数据的顺序并没有被修改

  ```
  127.0.0.1:6379> lrange  users:leto:guesses 0 -1
  1) "5"
  2) "9"
  3) "10"
  4) "2"
  5) "4"
  6) "10"
  7) "19"
  8) "2"
  ```

  set 按照字母表排序

  ```
  127.0.0.1:6379> sadd friends:ghanima leto paul chani jessica alia duncan
  (integer) 6
  127.0.0.1:6379> sort friends:ghanima limit 0 3 desc alpha
  1) "paul"
  2) "leto" 
  3) "jessica"
  ```

## 权限与管理

- redis 的配置文件是 redis.conf

- redis提供了一个轻量级的认证方式，可以编辑redis.conf配置来启用认证。修改 redis.conf 去掉requirepass前的注释，并修改密码为所需的密码，这个时候尝试登录redis，发现可以登上，但是执行具体命令是提示操作不允许。

- 也可以通过命令行配置密码

  ```
  127.0.0.1:6379[1]> config set requirepass my_redis
  OK
  ```

  但是命令行配置的密码仅限于此次 redis 运行期间使用，如果重启一下redis，则依旧使用 redis.conf 中密码鉴权。

- 危险命令重命名：

  对于 flushdb 、config 等重要命令，可以使用重命名来加强安全等级。例如：

  ```
  rename-command FLUSHALL joYAPNXRPmcarcR4ZDgC81TbdkSmLAzRPmcarcR
  ```

- 最大内存设置

  打开redis配置文件，找到如下段落，设置maxmemory参数，maxmemory是bytes字节类型，注意转换。

  ```
  # In short... if you have slaves attached it is suggested that you set a lower
  # limit for maxmemory so that there is some free RAM on the system for slave
  # output buffers (but this is not needed if the policy is 'noeviction').
  #
  # maxmemory <bytes>
  maxmemory 268435456
  ```

  当 redis 使用内存超过设置的最大值时，会报 out of memeory 异常。通常来讲实际内存达到最大内存的 3/4 时就要考虑加大内存。另外，使用 info 指令可以查看当前内存使用情况。

- 主从复制

  redis的复制功能是支持多个数据库之间的数据同步。一类是主数据库（master）一类是从数据库（slave），主数据库可以进行读写操作，当发生写操作的时候自动将数据同步到从数据库，而从数据库一般是只读的，并接收主数据库同步过来的数据，一个主数据库可以有多个从数据库，而一个从数据库只能有一个主数据库。