# 概述

Redis是基于 C语言编写的开源非关系型内存数据库，可以用作数据库、缓存、消息中间件 
- 键值型
- 单线程
- 低延时、速度快
- 数据持久化
- 支持主存集群、分片集群
- 支持多语言客户端

# 安装

## Windows

1. 下载 Redis 安装包
    - [下载地址](https://github.com/redis-windows/redis-windows)
    - 版本选择
        - 推荐 MSYS2
        - `with-Service` 代表会注册为系统服务，开机自启动
    - 下载完成后解压即可

2. 文件介绍

| 文件名                      | 用途                     |
| --------------------------- | ------------------------ |
| install_redis_service.bat   | 注册服务                 |
| uninstall_redis_service.bat | 卸载服务                 |
| start.bat                   | 快捷启动                 |
| redis-server.exe            | Redis 服务器             |
| RedisService.exe            | 服务运行的进程封装器     |
| redis-cli.exe               | 客户端                   |
| redis.conf                  | 配置文件                 |
| redis-full.conf             | 更完整的配置文件         |
| redis-benchmark.exe         | 压力测试工具             |
| redis-check-aof.exe         | 检查 appendonly.aof 文件 |
| resis-check-rdb.exe         | 检查 dump.rdb 文件       |
| redis-sentinel.exe          | Sentinel 高可用监控组件  |
| sentinel.conf               | Sentinel 的配置文件      |
| .dll                        | 动态链接库               |

3. 注册服务
    - 运行 `install_redis_service.bat`
    - 输入 Redis 安装路径
    - 输入配置文件路径（回车即可）
    - 唯一区别是多了一个 pid 文件

4. 启动
    - 分别运行 `redis-server.exe ` 、`redis-cli.exe` (分别开两个终端)
    -  在客户端中输入 `ping`,显示 `PONG` 安装成功
    -  注：服务器终端不可关闭
    -  可将安装路径配置到环境命令中,可随地使用`redis-server`、`redis-cli` 启动程序

## Ubuntu

### 包管理器

Redis 官网提供指令，配置文件位于`/etc/redis/redis.conf`
```
sudo apt-get install lsb-release curl gpg
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
sudo chmod 644 /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
sudo apt-get update
sudo apt-get install redis
```
### 压缩包

1. [下载地址](https://download.redis.io/releases/)
2. 解压目录
    - `/usr/local` 目录
    - `tar -xzf`
    - `make && make install`
### 配置

修改 redis.conf 文件

```
# 允许访问的地址，默认是127.0.0.1，会导致只能在本地访问。修改为0.0.0.0则可以在任意IP访问，生产环境不要设置为0.0.0.0
bind 0.0.0.0
# 守护进程，修改为yes后即可后台运行
daemonize yes 
# 密码，设置后访问Redis必须输入密码
requirepass 123321

# 监听的端口
port 6379
# 工作目录，默认是当前目录，也就是运行redis-server时的命令，日志、持久化等文件会保存在这个目录
dir .
# 数据库数量，设置为1，代表只使用1个库，默认有16个库，编号0~15
databases 1
# 设置redis能够使用的最大内存
maxmemory 512mb
# 内存满时的淘汰策略
maxmemory-policy allkeys-lru
# 日志文件，默认为空，不记录日志，可以指定日志文件名
logfile "redis.log"

# 持久化配置
# 快照持久化（RDB）
save 900 1      # 900秒内有1次写入则触发快照
save 300 10     # 300秒内有10次写入
save 60 10000   # 60秒内有10000次写入
# 是否压缩，压缩会消耗cpu
rdbcompression yes
# RDB 文件名称
dbfilename dump.rdb

# 追加文件持久化（AOF）
appendonly yes
# AOF 文件名
appendfilename "only.aof"
# 执行频率 立即记录|缓冲区 1 秒写入 | 系统决定
appendfsync [always|everysec|no] 
# 增长超多少百分比自动触放重写
auto-aof-rewrite-percontage 100
# 体积超多少触放重写
auto-aof-rewrite-min-size 64mb
```

### 使用命令

```
# 前台启动
redis-server
# 后台启动
redis-server /path/to/redis.conf
# 优雅停止
redis-cli shutdown
# 强制停止
kill -9 <redis-pid>

redis-cli -h <ip> -p <port> -a <passwd>

AUTH <passwd>
```

### 自启动

```
# 创建systemd服务文件
sudo vim /etc/systemd/system/redis.service

sudo systemctl enable redis
sudo systemctl start redis
```

文件内容
```
[Unit]
Description=Redis In-Memory Data Store
After=network.target
[Service]
User=redis
Group=redis
ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf
ExecStop=/usr/local/bin/redis-cli shutdown
Restart=always
[Install]
WantedBy=multi-user.target
```

## 监控

```
# 查看Redis状态信息
redis-cli info
# 查看内存使用详情
redis-cli info memory
# 查看持久化信息
redis-cli info persistence

# 日志分析
loglevel notice
logfile /var/log/redis/redis.log

# 手动触发RDB备份
redis-cli save       # 阻塞式
redis-cli bgsave     # 后台异步

# 压力测试
redis-benchmark -h 127.0.0.1 -p 6379 -n 100000 -c 32 -q
```


# 数据类型

## 通用命令

| 命令   | 功能                              |
| ------ | --------------------------------- |
| help   | 帮助手册                          |
| keys   | 查找出所有key                     |
| del    | 删除指定key                       |
| exists | 判断key是否存在                   |
| expire | 设置key有效期                     |
| ttl    | 查看剩余有效期（-1 永久 -2 过期） |

## String（字符串）

字符串类型，最简单的存储类型
- strign：普通字符串
- int：整型类型
- float：浮点数

| 命令        | 功能                |
| ----------- | ------------------- |
| set         | 添加或修改键值对    |
| get         | 根据key获取value    |
| mset        | 批量添加            |
| mget        | 批量获取            |
| incr        | 整型key自增         |
| incrby      | 整型key自增指定步长 |
| incrbyfloat | 浮点型自增指定步长  |
| setnx       | 不存在则添加        |
| setex       | 指定有效期          |

### key 结构

Redis key 允许多个单词形成层级结构`[项目名]:[业务名]:[类型]:[id]]`

## Hash （哈希）

散列，value 为无序字典（一个key对应多个字段）

| 命令                 | 功能                           |
| -------------------- | ------------------------------ |
| hset key field value | 添加或修改hash类型key的field值 |
| hget key field       | 获取hash类型key的field值       |
| hmset                | 批量添加                       |
| hmget                | 批量获取                       |
| hgetall              | 获取key中所field和value        |
| hkeys                | 获取key中所有fiels             |
| hvals                | 获取key中所有value             |
| hincrby              | key的字段值自增指定步长        |
| hsetnx               | 不存在则添加key的field值       |

## List （列表）

- 支持双向检索
- 有序
- 元素可重复
- 插入和删除快
- 查询速度一般

| 命令                | 功能                   |
| ------------------- | ---------------------- |
| lpush key  val...   | 从左侧插入元素         |
| lpop                | 从左侧删除元素         |
| rpush               | 从右侧插入元素         |
| rpop                | 从右侧删除元素         |
| lrange key star end | 返回范围内所有元素     |
| blpop,brpop         | 没有元素时等待指定时间 |



## Set （集合）

- 无序
- 元素不可重复
- 查找快
- 支持并集、交集、差集等

| 命令                 | 功能             |
| -------------------- | ---------------- |
| sadd key member...   | 添加元素         |
| srem key member...   | 删除元素         |
| scard key            | 返回元素个数     |
| sismember key member | 判断元素是否存在 |
| smembers             | 获取所有元素     |
| sinter key1 key2     | 求交集           |
| sdiff k1 k2          | 求差集           |
| sunion k1 k2         | 求并集           |


## ZSet （有序集合）

- 可排序
- 元素不可重复
- 查询速度快
- 底层为跳表
  
| 命令                         | 功能                             |
| ---------------------------- | -------------------------------- |
| zadd key score member        | 添加元素，存在则更新score        |
| zrem key member              | 删除元素                         |
| zscore key memeber           | 获取指定元素 score               |
| zrank key member             | 获取元素排名                     |
| zcard key                    | 获取元素个数                     |
| zcount key min max           | 统计socr值在指定范围内的元素个数 |
| zincrby key increment member | 指定元素自增指定步长             |
| zrange key min max           | 获取指定排名范围的元素           |
| zrangebyscore key min max    | 获取指定score范围内的元素        |
| zdiff、zinter、zunion        | 求差、交、并集                   |

注：默认升序，降序（zrev*）

## Geospatial

## Hyperloglog

## Bitmap



# 缓存

## 缓存更新

|          | 内存淘汰                             | 超时剔除 | 主动更新     |
| -------- | ------------------------------------ | -------- | ------------ |
| 说明     | 内存不足时自动淘汰，基于内存淘汰机制 | 添加TTL  | 业务逻辑更新 |
| 一致性   | 差                                   | 一般     | 好           |
| 维护成本 | 无                                   | 低       | 高           |


- 低一致性：内存淘汰
- 高一致性：主动更新+超时剔除

### 主动更新

1. 更新数据库同时跟新缓存
2. 缓存与数据整合为服务
3. 仅操作缓存，异步将缓存写入数据库

#### 操作缓存和数据库的问题

- 删除缓存大于更新缓存
- 单体系统事务合并，分布式采用TCC分布式事务
- 先操作数据库后删除缓存
  
## 缓存穿透

数据在缓存和数据库中都不存在

1. 缓存空对象：数据库不存在则在缓存中写null
   - 优点：实现简单，维护方便
   - 缺点
     - 额外内存消耗
     - 短期不一致 
2. 布隆过滤 
   - 采用布隆过滤器，存在放行，不存在拒绝

## 缓存雪崩

同一时段大量缓存key同时失效或Redis服务宕机，导致大量请求到达数据库

- TLL添加随机值
- Redis集群
- 添加降级限流策略
- 添加多级缓存

## 缓存击穿

部分被高并发访问且缓存重建业务较复杂的的key失效，瞬间给数据库大量压力

1. 互斥锁：访问前后加速
2. 逻辑过期：更新缓存操作由单独线程完成，完成前先使用旧数据

| 解决方案 | 优点                                 | 缺点                                   |
| -------- | ------------------------------------ | -------------------------------------- |
| 互斥锁   | 没有额外内存消耗实现简单，保证一致性 | 线程等待、死锁                         |
| 逻辑过期 | 性能好                               | 不保证一致性，右额外内存消耗，实现复杂 |

Redisson 分布式框架

Stream 消息队列

# 持久化

## RDB

Redis 数据备份文件，内存中的所有数据记录到磁盘
```
save   # 主进程执行，阻塞命令
bgsave # 子进程执行
```
Redis 停机自动执行

### bgsave 基本流程

- fork 主线程，共享内存空间
- 子进程读取内存数据并写入 RDB 文件
- 用新文件替换旧 RDB 文件


### 缺点

- 执行间隔长
- fork 子进程、压缩、写出文件都耗时

## AOF

追加文件，Redis 处理的每个写命令记录入 AOF 文件
`bgrewritead`重写文件，简化命令记录


## RDB vs AOF

|            | RDB          | AOF              |
| ---------- | ------------ | ---------------- |
| 持久化方式 | 定时快照内存 | 记录执行命令     |
| 数据完整性 | 不完整       | 相对完整         |
| 文件大小   | 有压缩，小   | 大               |
| 恢复速度   | 快           | 慢               |
| 优先级     | 低           | 高               |
| 资源占用   | 高           | 低  （重写时高） |
| 场景       | 追求启动速度 | 数据安全性要求高 |

# 主从架构

- 高可用，高并发读

## 搭建

1. 准备不同redis.conf文件
   -  修改端口、工作目录
   -  指定声明IP `replica-announce-ip <ip>`
   -  指定主从关系 `slaveof <masterip> <masterport>`

## 数据同步原理

Replication Id，数据集标记，一致为同一数据集   
offset，偏移量，随repl_baklog中的数据而增大  


1. 第一次同步：全量同步 
   - 同步数据版本信息
   - 发送 RDB 文件，记录RDB期间的执行命令人repl_baklog
   - 发送 repl_baklog 文件中的命令 
2. 从机重启后同步：增量同步 
   - id 一致，offset 不一致
   - 发送偏移量后的命令

## 哨兵机制（Sentinel）

实现主从集群的自动故障恢复，解决主机宕机问题  
- 监控主从机是否按预期工作
- 自动故障恢复，自动替换主机
- 通知客户端故障情况

### 原理

心跳机制监测服务状态，间隔 1s 向每个实例发送 ping 命令
- 主观下线：实例未在规定时间响应
- 客观下线：超 quorun（一半）数理的 Sentnel 认为实例主观下线

选择依据：  
1. 断开时间
2. 优先级
3. offset 值
4. 运行 id

转移步骤：  
1. 选定从机执行slaveof no one
2. 所有节点更新 slaveof
3. 修改故障主机为从机

### 搭建

创建sentinel.conf文件  
```
port <> # sentinel 实例端口
sentinel announce-ip <>  # 声明 ip
sentinel monitor <mymaster> <ip> <port> <quorun> # 集群名称 主机ip 主机端口 客观下线阈值
sentinel down-after-milliseconds <mymaster> <5000> # 超时时间
sentinel failover-timeout <mymaster> <6000>
dir <> # 工作目录
```

启动 `redis-sentinel .conf`

# 分片集群

- 海量数据存储
- 高并发写

## 特点

- 多个主机
- 主机通过 ping 互相监控
- 客户端可任意访问节点

## 搭建

准备 redis.conf 文件
```
# 开启集群功能
cluster-enabled yes
# 指定集群配置文件名称
cluster-config-file <path/nodes.conf>
# 节点心跳超时时间
cluster-node-timeout <>
```

启动全部节点

```
redis-cli --cluster create --cluster-replicas <master_num> <ip:port...> # 创建集群
# 主机数 = 总数/(master_nun+1) 

redis-cli -p <port> clusert nodes # 查看节点状态的

# 连接节点加 -c
```

## 散列插槽

主机节点映射到 0~16383个插槽上，数据key 与插槽绑定，防止主机宕机
-  分配插槽给实例
-  计算哈希值对16384取余
-  余数作为插槽寻找实例


## 集群伸缩

```
# 添加节点
redis-ctl -c add-node
# 分配插槽
redis-ctl -c reshart
```

## 故障转移

```
cluster failover 
```




# 数据结构

![redis_01](https://cdn.jsdelivr.net/gh/Noxryn/PicGallery@main/img/redis_01.png)


## 动态字符串 SDS

普通字符串  
- 长度计算获得
- 非二进制安全
- 不可修改

SDS - 结构体  
```
struct __attribute__ ((_packed__)) sdshdr8 {
    len;  // 已保存字节数
    alloc; // 总申请字节数
    flags; // 头类型
    buf[];
}
```
1. 按 len 读取存储字符长度，O(1)
2. 动态扩容，2*len+1（< 1M），len+1M+1（>1M）- 内存预分配  
3. 减少内存分配次数
4. 二进制安全

## 整数集合 IntSet

基于整数数组，长度可变，有序

```
typedef struct intset {
    encoding; // 编码方式 short，int，long
    length;
    contents[];
}
```
- 整数升序依次保存   
- 编码方式自动升级


## 哈希表 Dict  

```
typedef struct dict {
    dictType *typd; // 函数
    *privdata; // 私有数据
    dictht ht[2]; // 哈希表，一个当前数据 ， 一个空，rehash 使用
}
typedef struct dictht {
    dictEntry ** table; // 哈希表
    size;
    sizemask; // 大小的编码 size - 1
    used; // entry 个数
}

typedef truct dictEntry {
    void * key; // 键
    union {
        void *val
        u64;
        s64;
        d;
    }v; // 值
    *next;
}
```

根据 key 计算 hash 值， hash & sizemask （hash % size）计算索引位置

数组结合单向链表，更新键值对时检查负载因子，触放哈希表扩容/收缩，过程中必然重新创建哈希表、计算索引，称 rehash，（渐进式rehash:每次只操作一个数据，h[0]只删不增，h[1] 只增加）

## 压缩列表 ZipList - > listpack

特殊双端链表。 使用（前节点长度+编码属性+数据）替代指针，内存连续  

增删大数据存在连锁更新问题，不断移动数据扩容

## QuickList

两端链表，节点为 ZipList，提供配置项限制节点中数据最大值

## 跳表 SkipList

- 元素升序排列
- 节点中存在不同跨度的指针

## RedisObject 

任意数据类型的键值被封装为 RedisObject，Redis 对象  
```
type // 对象类型
encoding // 编码方式
lru // 最后被访问时间
refcount // 引用计数器
*ptr   // 数据 
```

# 网络模型  

- 核心业务部分（命令处理）：单线程
- 整体：多线程 v4多线程异步处理耗时任务，v6多线程利用多核CPU  

纯内存操作,执行速度快，多线程不会带来提升   
多线程，上下文切换过多，开销大  
线程完全引入锁影响性能 

AE 封装库 epoll 

IO多路复用+事件派发（多：数据解析，读写，响应）

RESP通信协议  

# 内存回收 

## 过期策略

DB结构，key-value保存在哈希表中，数据库结构中存在TTL字段  key-value+key-ttl

惰性删除：访问时检查key是否过期，进行删除  

周期删除：定时任务，周期性抽样部分进行删除  （>10% 重抽）
- slow:hz决定，耗时低于25%
- fast：事件循环调用beforeSleep函数，耗时低于1ms

## 淘汰策略

主动删除部分key 

- 不允许新写入
- TTL 低
- 随机淘汰
- 设TLL的随机淘汰
- LRU (最近访问)和LFU（访问频率）

淘汰池  

# 优化 

1. key
   - [业务]:[数据名]:[id]
   - 长度小于44字节  
   - 不包含特殊字符
2. value
   - 单个value 小于 10kb
   - 元素数量小于1000

bigkey:网络阻塞、数据倾斜、主线程阻塞、CPU压力
3. 数据类型
    - json
    - 字段打撒
    - hash
hash entry数量过多使用哈希表而不是listpack
string内存占用大  
拆分小hash 
4. 
pipeline 复杂数据类型大量导入

5. 
   - 缓存不持久化
   - AOF持久化优先
   - 手动RDB，数据备份
   - 设置合理阈值
   - 刷盘时不aof 

6.slow select  


  

