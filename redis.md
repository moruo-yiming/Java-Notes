[toc]



## NoSQL

### 背景：

#### 非关系型数据库用来干嘛？

负载均衡、做缓存--解决CPU、内存压力、解决sesssion共享问题

#### 不同服务器如何共享session?

* 存到客户端，但有安全问题
* 复制多份，占用内存
* 存到缓存数据库，访问速度快

#### 解决IO压力?

对频繁的数据访问放到缓存数据库

#### NoSQL特点

不遵循sql标准；不支持ACID; 性能远超sql；数据存在内存

#### 数据库如何提高访问效率

行式数据库、列式数据库

### Redis

特点：支持持久化，支持多种数据类型

#### 底层原理

redis：单线程(原子性)+多路IO复用

memcached ：多线程 + 锁

#### 准备环境：

##### 虚拟机：安装Vmware

理解网络适配器配置：三种网络连接模式

* 桥接模式：每个Linux系统占用一个IP，容易造成IP冲突，但是能和外部系统通讯
* NAT模式：网络地址转换模式，主机有两个IP，其中一个和虚拟系统构成另一个网络，不占用外部IP（不造成冲突），可以和外部通讯，但无法从外界访问虚拟系统。
* 主机模式：独立系统

##### linux：安装centos7

1. 硬盘分区	分3区 
    /boot引导文件，默认200M即可；       /swap交换区，设跟内存一致；    / 根分区，剩余全部

2. 防火墙操作命令
   centos7	systemctl   start/stop/status   firewalld  开启，关闭，状态
   centos6	service   iptables    start/stop/status  

> 教程：b站搜linux即可

##### reids安装

使用 `gcc --version` 查看是否有gcc环境     安装命令：yum install gcc

官网下载压缩包，使用 `tar -zxvf 压缩包` 解压，`cd`进入解压目录，`make `编译，`make install` 安装。

##### redis启动

> redis安装目录：默认/usr/local/bin，包括有redis-server(redis服务)，redis-cli（redis客户端）		redis-check-aof(用于异常恢复)，redis-sentinel（哨兵模式）等
>
> redis解压目录：有redis配置文件，即redis.conf

前台启动：`/user/local/bin/redis-server`

后台启动：启动时，加上配置文件（**需配置**） `/user/local/bin/redis-server   /路径/redis.conf` 

注：前台启动需要另开一个终端才能使用redis客户端，后台启动需要配置
		建议将配置文件拷贝到其它路径，原先的保留不动，使用的时候也使用拷贝的那个文件。

##### 配置文件redis.conf

`vi /路径/redis.conf`编辑配置文件，查找命令`:/protected`

```.conf
protected-mode no  禁止保护模式
daemonize yes   开启可后台启动
#bind 127.0.   注释掉，允许远程访问
```

默认端口：6379     redis有16个库(0-15)

#### 快速上手

```shell
# String类型操作 	最大512M 
添加set k v		获取get k		setnx相同k不覆盖
数值自增|自减		incr|decr k
自定义步长数值自增|自减	incrby|decrby k step
# 库操作
select切换库	dbsize键数量	keys *查看所有键	  type k键类型	
del k删除	    flushdb清空库	 flushall清空所有库
expire k time设置键的过期时间		ttl k查看剩余时间(-1永不过期，-2已过期)
```

*其它数据集合（用到再去学即可）

> [Redis 命令参考](http://redisdoc.com/index.html)

#### 消息发布和订阅

订阅名字为myChannel的频道：`subscribe myChannel` 

myChannel频道发布message：`publish myChannel message` 

> 需要打开两个客户端

#### 事务

redis事务是一个单独的隔离操作，事务中的所有命令会被序列化并按顺序执行，执行过程中不受其它客户端的请求影响。

使用：
	multi开启事务，之后的命令会被加入队列--组队过程，直到使用执行或取消命令
	exec执行	discard取消

特点：组队过程中一旦有错误，exec执行时全部失败；exec执行时才有的错误不影响其它正确的操作。

事务三特性：

#### 锁机制

悲观锁：操作共享数据时会上锁，这样保证其它事务无法同时操作共享数据

乐观锁：操作共享数据时，通过查询版本号判断数据是否发生改变。（例如数据对应的版本号为1，操作时检查是否为1，为1则操作并修改版本号为2，不为1说明数据被修改过，当前无法进行操作）

redis中通过watch命令监控共享数据，一旦数据发生变化，就无法对其操作，unwatch取消监控

根本问题：由于读取的共享数据不是最新的。

#### 并发模拟工具

安装：yum install httpd-tools

ab命令模拟测试请求

命令： ab -n 1000 -c 200  http://192.168.101.1:8080/skill   （-n 请求次数	-c  并发次数） 
			192.168.101.1为windos电脑ip

注意：关闭linux中的防火墙

> 实现商品秒杀

### 持久化

#### RDB(Redis Database)

是Redis默认采用的持久化方式，以快照的形式将数据持久化到硬盘中。

##### 触发方式:

* 手动：save或`bgsave`命令

* 自动触发：配置redis.conf，让服务器在一定条件下自动执行bgsave命令

  ```conf
  stop-write-on-bgsave-error yes  磁盘无法写入时则关闭redis(例如磁盘满了)
  rdbcompression yes		rdb文件是否压缩
  save 20 3  				表示10秒内如果有大于等于3个key发生变化，就执行bgsave命令
  dbfilename dump.rdb  	rdb文件名
  dir ./					持久化文件保存路径，包括aof也是如此
  ```
  
  注：shutdown关闭服务，清空数据库时也会自动触发。

##### RDB原理:

save命令会让redis服务器阻塞，直到持久化过程结束。
bgsave命令则采用异步的方式生成快照，Redis首先会通过fork获得一个跟父进程一样的子进程，fork过程父进程会被阻塞，fork之后父进程可以继续响应其它命令，而子进程会将数据保存到临时文件，然后再用临时文件去替换dump.rdb

（bgsave底层：COW(copy on write)写时复制，通过复制一个副本并对副本修改，而不是直接修改原先的）

##### 优缺点:

rdb文件体积小，使得恢复数据速度快；但由于要创建子进程，很消耗内存，没法做到实时的持久化。

#### AOF(Append Only File)

以日志的方式（追加）记录每次的写操作(有修改就记录)。Reids重启时，通过重新执行这些命令来恢复数据。

保证数据持久化的实时性。

##### 使用：

配置redis.conf
```shell
appendonly yes  //开启，redis开启时优选读取的是aof文件，而不是rdb
appendfilename "appendonly.aof"  //aof文件名
```

##### 异常恢复：

文件损坏时，通过 `redis-check-aof --fix  appendonly.aof `命令恢复（原理就是把报异常的命令包括后面的命令都去除）,

##### 持久化策略（同步频率）：

```shell
# 配置redis.conf
appendfsync always	   始终同步，有写入就记入日志
appendfsync everysec	每秒同步（默认）
appendfsync no	把同步时机交给操作系统
```

##### 重写策略（Rewire压缩）

aof采用文件追加的方式，会使文件越来越大，所以新增了重写机制，当aof文件的大小是上一次aof文件的2倍，且aof文件大于64mb，则进行压缩。

手动重写命令：bgrewriteaof

默认配置：
```conf
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

原理：使用rdb快照，以二进制的形式附在aof文件的头部

```.conf
no-appendfsync-on-rewrite
```

##### 持久化流程：

写命令会被追加到AOF缓冲区；缓冲区根据AOF持久化策略将命令同步到磁盘AOF文件；
文件超过一定大小时，会进行文件压缩; Redis重启时，通过加载AOF文件恢复数据。

#### 主从复制

1.介绍：一台主机负责写操作，多台从机负责读操作，数据由主机同步到从机。

2.优缺：读写分离，容灾快速恢复（从机挂了有其它从机）；
			   数据同步时有延迟。

3.使用：

> 需要开启多个窗口，可用多个端口来模拟多个redis服务

为不同的redis服务创建和配置不同文件：例如配置端口为6379的服务，创建redis6379.conf文件

```.conf
include redis.conf 		#引入redis配置文件，减少配置
# 配置进程，用于模拟多个redis服务，该配置会在对应路径下创建该文件，命名随意
pridfile 	./redis-6329.pid
# 持久化文件
dbfilename 6379.rdb
# 指定端口号
port 6379
```

后台启动和进入客户端

```shell
redis-server  ~/redis6379.conf
redis-cli -p 6379  #指定端口号
```

查看服务信息（主机master、从机slave）
```conf
info replication  #默认都是主机
```

设置从机（将主机变为从机），需要指明主机服务器

```conf
slaveof ip port #主机服务器的ip和端口号
```

从机变主机(参见反客为主)

```.conf
slave no one
```

注：主从复制中，主机挂了，通过集群可解决。

#### 一主二从

复制原理：从机启动成功连接到主机后会发送一个同步命令；主机收到命令后进行持久化操作，然后将持久化的文件传送到从机。从机接收文件并读取（首次全量复制）；之后主机一旦有写命令，就会同步到从机。

#### 薪火相传

当主机有很多从机时，可将部分从机(B)交由其它从机(A)管理，从而减轻主机的压力。当然，其它从机(A)仍然无法进行写操作，只是能代替主机同步数据到部分从机(B)，即薪火相传。

#### 反客为主

当主机挂了，一种措施就是把某台从机变为主机，从而维持系统正常。

**哨兵模式**(自动的反客为主)：主机挂时，从机变主机这种操作肯定需要能自动触发，哨兵模式就是利用哨兵（也是服务器）监控后台主机是否故障，如果故障，就选择从机替代主机

实现：

* 配置哨兵配置文件sentinel.conf，名字随意。

  ```conf
  # num代表得有num个哨兵监控到才能执行反客为主，至少一个
  sentinel monitor masterName ip port num	#主机名masterName(随意)、主机ip、主机端口号
  ```

* 启动

  ```conf
  #redis-sentinel
  redis-sentinel  /路径/sentinel.conf  #加上配置文件
  ```

从机选择原则：优先级(redis.conf中的replica-priority  num，num越小越优先)、偏移量(谁的数据全)、runid最小的（每个服务有一个随机的runid）

注意：后面如果主机恢复，重启后会变为从机，因为sentinel会向其发送slaveof命令

### 集群

#### 作用

当redis存储容量不够时，可通过集群扩容；当对服务器进行并发写操作时，可通过集群分担压力。

#### 特点

无中心化的集群配置：任何一台机都可以作为入口访问，如果当前服务器不是处理该命令的，则切换到处理该命令的服务器。

#### 原理

集群实现了对Redis的水平扩容，即启动N个Redis节点，将整个数据库分布存储在这N个节点中，每个节点存储总数据的1/N，通过分区，使得集群中若有部分节点失效，集群也能继续处理命令请求。

不足：多键操作需要添加组才可以操作；不支持事务；

#### 搭建redis集群

> 同样需要开启多个窗口，需要6个，分3组，一主一从

创建和配置6个redis.conf，可以沿用主从复制中的配置，以redis6379.conf为例

```conf
include redis.conf
pridfile 	./redis-6329.pid
dbfilename 6379.rdb
port 6379
cluster-enabled yes		#开启集群
cluster-config-file node-6379.conf	#节点配置文件名，
cluster-node-timeout 15000	#设置节点失联时间，超过该事件，集群自动进行主从切换
```

将6节点合成为集群

redis-cli --cluster create  --cluster-replicas 1  ip:port  ip:port......

 1代表采用最简单的方式，一主一从

客户端进入

```conf
redis-cli cluster -c -p port #任意端口进入 -c采用集群策略连接，操作数据时会自动进行服务器切换
```

**服务器切换解释**

slots--插槽，用于集群中保存键，有16384个，每个键都会对应一个插槽（哈希），可用数组去理解。每个服务器对应一部分插槽，每次添加的键通过计算落在那一部分插槽就切换到哪个服务。

插槽操作

```conf
cluster countkeysinslot 3564  #根据插槽3564获取键的数量
cluster getkeysinslot 3563 num  #根据插槽3563获取对应的键，num代表获取几个
```

多键操作，集群不能直接使用mset，键后面需要携带统一的标识，这样才能使用同一插槽

```conf
mset k1{k} v1 k2{k} v2  # k为标识，也就是插槽与键要一对一
```

当插槽对应的主从都挂掉，redis集群是否都会挂掉（由配置决定redis.conf）
```conf
cluster-require-full-coverage no  不会挂掉，yes则会
```

### 应用问题（理解为主）

#### 缓存穿透

问题：向数据库请求不存在的数据，由于不存在，缓存也不会有，导致请求不通过缓存直接访问数据库。（漏洞攻击）
解决：

* 空值缓存：查询返回为空时也进行缓存
* 设置白名单：那些可访问，那些不行。
* 布隆过滤器：集合判断访问数据是否存在，不存在则拦截，同白名单
* 实时监控：

#### 缓存击穿

问题：缓存中存在请求数据，但key过期了，此时有大量并发请求该数据，可能导致数据库被瞬间压垮。（热点数据）
解决：

* 预先设置热门数据，延长缓存时间
* 实时调整过期时长
* 使用锁：排队请求，例如使用setnx设置锁（存在则返回失败），如果返回成功，放行去查数据库并缓存；如果返回失败，则让但前线程sleep一会再重试。

缓存雪崩

问题：缓存中存在请求数据，但大量的key集中过期了，此时有大量并发请求该数据。类似缓存击穿，不同的是大过期key的数量。
解决：

* 使用锁和队列：同上

* 构建多级缓存：nginx缓存+redis缓存+等
* 设置标志更新缓存：记录key是否过期，过期的话则触发其它线程去更新缓存
* 分散缓存失效：对每个key的缓存失效时间增加1-5分钟的随机值，这样就很难引发大量key集中过期

#### 分布式锁

问题：分布式集群系统，不同服务器存在数据共享问题。

实现互斥锁

```shell
setnx lock value #上锁
expire lock 10 #过期时间
del lock #解锁
```

原子性问题：上锁和设置过期时间时，可能出现上锁后异常导致锁永不过期，出现死锁现象。

```shell
set lock value nx ex 10  #上锁并设置过期时间；nx代表不可重复，等价setnx；ex代表过期时间参数
```

释放其它锁问题：假如某服务器执行完操作，接着进行锁释放操作时，突然发生卡顿，无法操作，并且锁到了过期时间，于是自动释放，此时另一台服务获得锁并进行操作；而卡顿的服务器终于恢复了，继续操作释放锁，但此时释放的时另一台服务器的锁。

```shell
解决：value用唯一值代替，这样就能保证每个锁的value不一样，
这样，释放锁时判断value是否发生变化，就能知道锁是否被释放。
```

释放锁需要原子操作: 由于uuid的比较和释放锁的操作是两步，不是原子操作，这就可能出现释放锁之前，锁刚好因为过期释放，然后其它服务器获得到了锁，但原来的服务器继续执行锁的释放，此时释放的是其它服务器的锁。

```shell
lua脚本解决
```

要求：

* 互斥性，也就是只能有一个客户端持有锁
* 不能出现死锁
* 不能释放别人的锁
* 加锁和解锁需要原子操作

> 实现分布式锁

### 新增功能

#### ACL权限控制

查看所有命令`acl help` 
查看用户列表`acl list`

```shell
user  default    on    nopass     ~*       &*     +@all
     默认用户名   开启   无密码   可操作所有key     可执行所有命令
```

查看权限类别`acl cat` ；查看具体类别对应的命令，例如string，`acl cat string`
新建或修改用户`acl setuser uname  >pwd` ......基本参数具体如下

```shell
uname   用户名
>pwd    设置密码为pwd
on/off  是否开启该用户
~*  	指定可操作的key，例如~a*，表示只能操作包含a的key
+@all   指定可执行的命令，例如+@string表示可操作string的所有命令，-@string则是移除
```

切换用户`auth uname pwd`, 对于默认用户的命令为auth  default  nopass

#### 多线线程IO

```shell
io-threads-do-redis yes  #开启
io-threads 4 
```







### *数据集合（用到再去学即可）

List  k 对应多个v

lpush左插 	rpush右插   lpop左删  rpop右删	rpoplpush右插左  	lrange查看全部

lindex根据下标获取元素		llen长度		linsert在某个值前面或后面插入

lrem删除  	lset修改

Set 

Hash哈希

Zset	有序Set  跳跃表+hash

...

































