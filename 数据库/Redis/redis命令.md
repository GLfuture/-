[Redis相关命令详解以及原理-VIP-2304.pdf](file:///D:/零声Linux/redis/Redis相关命令详解以及原理-VIP-2304.pdf)

[Redis协议与异步方式-VIP-2304.pdf](file:///D:/零声Linux/redis/Redis协议与异步方式-VIP-2304.pdf)

```
keys *//显示所有的key
set key value
get key
hset key key value(放入一个哈希)//hset role:10001 age 21
hget key key (从左边lpush/rpush) lpush key value(插入一个双端队列，具备插入有序性)

zset key scorce member(key标识，score排序，member唯一)
zrange key start end (遍历有序集合)

flushdb(清空所有数据)

incr key(自增 +1)
incrby key  10(累加 +10)

redis-server --requirepass 123456
```

### 配置文件

```
/etc/redis
```

#### 修改密码

方式一：临时密码
使用CONFIG set requirepass 密码命令来设置密码。设置密码后，客户端连接 redis 服务就需要密码验证，否则无法执行命令。建议配置密码之后退出客户端，重新连接。

127.0.0.1:6379> CONFIG set requirepass "123456"
OK
127.0.0.1:6379> CONFIG get requirepass
1) "requirepass"
2) "123456"

方式二：永久密码
配置文件配置密码，修改默认# requirepass foobared配置，将#去掉，foobared改为自己的密码即可。配置密码之后重新启动 Redis 服务器。

#### 密码无效
上述密码配置方式二在配置之后，会出现密码无效的情况，也就是不输入密码也可以使用。常见的问题是在启动Redis 时没有指定配置了密码的配置文件启动。
有部分情况在指定了配置文件启动之后依旧密码无效，此时可以检查配置文件中是否包含类似如下信息：

user default on nopass ~* &* +@all
1
意思时默认账号不使用密码。将其注释掉或者删掉，重新启动 Redis 密码就能生效了。

#### 密码验证
使用AUTH password命令验证密码

127.0.0.1:6379> AUTH 123456
OK
127.0.0.1:6379> SET mykey "Hello Password"
OK
127.0.0.1:6379> GET mykey
"Hello Password"

### 远程登录

```
 redis-cli -h host -p port -a password
```

### 连接处理

一般采用两条连接，一条用于处理基础命令，一条处理发布订阅

### redis发布订阅模式缺陷

不保证消息一定到达

### 事务

单条连接不需要考虑事务

### acid特性

A.原子性		事务要么成功，要么失败，失败回退

C.一致性		完整检测一致性，逻辑一致性(顺序一致性)

I.隔离性		(解决方法：加锁—>串行执行, MVCC(mysql))

D.持久性		宕机能刷进磁盘

### watch multi exec和lua脚本的差异

watch multi exec事务执行过程中无法拿到中间的值，无法具备逻辑性，而lua在执行事务时可以拿到过程中的值，并处理

lua脚本没有自动回滚机制，需要自己实现

### scan

```
scan index match regex count n
//例:scan 0 match * count 1 ,返回下一个index和key
```

