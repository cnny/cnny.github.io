---
layout:     post
title:      "Redis CPU 负载过高问题排查记录"
subtitle:   ""
date:       2023-03-30 14:11:00
author:     "Cann"
header-img: "img/music-05.jpg"
tags:
    - Redis
---

#### 小记：单进程 Redis CPU 负载100%的排查过程

##### 1: 怀疑是否连接数过多

```
    # 查看连接数
    info clients
```

```
    # Clients
    connected_clients:241
    client_recent_max_input_buffer:4
    client_recent_max_output_buffer:0
    blocked_clients:0
```

> 连接数并不多，排除该项


##### 2：怀疑是否内存不足

```
    # 查看内存
    info memory
```

```
    used_memory_human:12.15G
    total_system_memory_human:31.26G
```

> 剩余内存充足，排除该项

##### 3：怀疑存在慢查询

```
    # 查看慢查询日志
    slowlog get [条数]
```

```
     1) 1) (integer) 455249
        2) (integer) 1680136269
        3) (integer) 12109
        4) 1) "ZADD"
           2) "workspace_prod_horizon:supervisors"
           3) "1680136269"
           4) "cx-workspace-q1zU:supervisor-6"
        5) "10.35.4.6:39178"
        6) ""
     2) 1) (integer) 455248
        2) (integer) 1680091982
        3) (integer) 15771
        4) 1) "ZADD"
           2) "workspace_prod_horizon:supervisors"
           3) "1680091982"
           4) "cx-workspace-q1zU:supervisor-5"
        5) "10.35.4.6:39174"
        6) ""
     3) 1) (integer) 455247
        2) (integer) 1680071973
        3) (integer) 31216
        4) 1) "COMMAND"
        5) "10.35.4.3:52512"
        6) ""
     4) 1) (integer) 455246
        2) (integer) 1680056153
        3) (integer) 10814
        4) 1) "LLEN"
           2) "gaoshu_prod_database_queues:location_search_queue"
        5) "10.35.4.3:48930"
        6) ""
```

> PS:
> 1) 日志ID
> 2) 日志时间戳
> 3) 耗时（微秒）
> 4) 执行命令

> 最近并没有慢查询，排除该项

##### 4: 怀疑存在 Bigkey

```
    # 查询各类型中最大的 key
    redis-cli --bigkeys
```

```
    -------- summary -------

    Sampled 163717 keys in the keyspace!
    Total key length in bytes is 9539633 (avg len 58.27)

    Biggest   hash found 'gaoshu_prod_horizon:b0098ae3-b95d-437d-a856-9fb9fc3e3bf9' has 13 fields
    Biggest string found 'cx_passport_prod_database_cx_passport_prod_cache:2w6rVB3ssd3hHl2CGzdwiprv8KJkYaGGaqodU2hm' has 494 bytes
    Biggest    set found 'gaoshu_prod_horizon:measured_jobs' has 26 members
    Biggest   zset found 'gaoshu_prod_horizon:completed_jobs' has 69 members

    0 lists with 0 items (00.00% of keys, avg size 0.00)
    152399 hashs with 609929 fields (93.09% of keys, avg size 4.00)
    11297 strings with 3026432 bytes (06.90% of keys, avg size 267.90)
    0 streams with 0 entries (00.00% of keys, avg size 0.00)
    6 sets with 69 members (00.00% of keys, avg size 11.50)
    15 zsets with 147 members (00.01% of keys, avg size 9.80)
```

>可以看到，最大的 key 也才 494 bytes、69 members，远达不到影响 CPU 的程序，排除该项

>PS：maxmemory-policy = noeviction 时， --bigkeys 命令会执行报错

```
    volatile-lru -> Evict using approximated LRU among the keys with an expire set. 根据LRU算法删除设置过期时间的key

    allkeys-lru -> Evict any(任何的，一些) key using approximated LRU. 根据LRU算法删除任何key

    volatile-lfu -> Evict using approximated LFU among the keys with an expire set. 根据LFU算法删除设置过期时间的key

    allkeys-lfu -> Evict any key using approximated LFU. 根据LFU删除任何key

    volatile-random -> Remove a random key among the ones with an expire set. 随机移除设置过过期时间的key

    allkeys-random -> Remove a random key, any key. 随机移除任何key

    volatile-ttl -> Remove the key with the nearest expire time (minor TTL) 移除即将过期的key(minor TTL)

    noeviction -> Don't evict anything, just return an error on write operations. 当内存不足以容纳新写入数据时，新写入操作会报错。
```

##### 5: 怀疑存在 Hotkey

```
    # 查询访问频率最高的 key
    redis-cli --hotkeys
```

```
    -------- summary -------

    Sampled 163703 keys in the keyspace!
    hot key found with counter: 63  keyname: gaoshu_prod_horizon:supervisors
    hot key found with counter: 40  keyname: bizfin_prod_horizon:supervisors
    hot key found with counter: 32  keyname: workspace_prod_horizon:supervisors
    hot key found with counter: 19  keyname: gaoshu_prod_horizon:supervisor:cx-gaoshu-U8wW:supervisor-6
    hot key found with counter: 18  keyname: gaoshu_prod_horizon:supervisor:cx-gaoshu-U8wW:supervisor-1
    hot key found with counter: 17  keyname: workspace_prod_horizon:masters
    hot key found with counter: 17  keyname: bizfin_prod_horizon:supervisor:cx-gaoshu-OW8N:supervisor-1
    hot key found with counter: 16  keyname: gaoshu_prod_horizon:supervisor:cx-gaoshu-U8wW:supervisor-4
    hot key found with counter: 16  keyname: gaoshu_prod_horizon:master:cx-gaoshu-U8wW
    hot key found with counter: 16  keyname: workspace_prod_horizon:supervisor:cx-workspace-q1zU:supervisor-4
    hot key found with counter: 15  keyname: workspace_prod_horizon:master:cx-workspace-q1zU
    hot key found with counter: 15  keyname: workspace_prod_horizon:supervisor:cx-workspace-q1zU:supervisor-1
    hot key found with counter: 15  keyname: gaoshu_prod_horizon:supervisor:cx-gaoshu-U8wW:supervisor-2
    hot key found with counter: 14  keyname: gaoshu_prod_horizon:supervisor:cx-gaoshu-U8wW:supervisor-3
    hot key found with counter: 14  keyname: gaoshu_prod_horizon:supervisor:cx-gaoshu-U8wW:supervisor-10
    hot key found with counter: 14  keyname: gaoshu_prod_horizon:supervisor:cx-gaoshu-U8wW:supervisor-11
```

> 可以看到，访问频率最高的 key 也才 63 次，并不存在 Hotkey， 排除该项

PS: 同上，maxmemory-policy = noeviction 时， --hotkeys 命令会执行报错

##### 6： 怀疑数据持久化导致的阻塞

执行 `htop`, 可以看到，是 `redis-rdb-bgsave` 进程占用了大量CPU，可以确定，是数据持久化导致的问题

数据持久化占用的是IO，并不影响 CPU，考虑是开启了数据压缩导致的 CPU 占用

```
   # 查看是否开启了数据压缩
   config get rdbcompression
```

```
    1) "rdbcompression"
    2) "yes"
```

> 可以确定就是这个问题

```
    # 关闭数据压缩
    config set rdbcompression "no"
```

> 当然，也可以选择直接关闭数据持久化

```
    config set save ""
```

> PS: save 的参数含义

```
    save 900 1     // 表示每15分钟且至少有1个key改变，就触发一次持久化
    save 300 10    // 表示每5分钟且至少有10个key改变，就触发一次持久化
    save 60 10000  // 表示每60秒至少有10000个key改变，就触发一次持久化
```

>PS: 当然，最好是直接修改 Redis 配置文件
