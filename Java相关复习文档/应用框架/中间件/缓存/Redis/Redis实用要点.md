# Redis实用

## 危险命令

Redis Keys 命令用于查找所有符合给定模式 pattern 的 key

尽管这个操作的时间复杂度是 O(N), 但是常量时间相当低。例如，在一个普通笔记本上跑Redis，扫描100万个key只要40毫秒。

```redis
命令格式 KEYS pattern
```

**其他危险命令：**

但凡发现时间复杂度为O(N)的命令，都要慎重，不要在生产上随便使用。例如hgetall、lrange、smembers、zrange、sinter等命令，它们并非不能使用，但这些命令的时间复杂度都为O(N)，使用这些命令需要明确N的值，否则也会出现缓存宕机。

1、flushdb 命令用于清空当前数据库中的所有 key

2、flushall 命令用于清空整个 Redis 服务器的数据(删除所有数据库的所有 key )

3、config 客户端连接后可配置服务器

**如何禁用危险命令**

在redis.conf中，在SECURITY这一项中，新增以下配置禁用指定命令：

```
rename-command FLUSHALL ``""``rename-command FLUSHDB ``""``rename-command CONFIG ``""``rename-command KEYS ``""
```

另外，对于flushall命令，需要设置配置文件中appendonly no，否则服务器是无法启动。

如果想要保留命令，但是不轻易使用，可以重命名命令来设定：

```
rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
```

TIP：更改记录到AOF文件或传输到从属服务器的命令的名称可能会导致问题。

**改良建议**

一、如果有这种需求的话可以自己对键值做索引，比如把各种键值存到不同的set里面，分类建立索引，这样就可以很快的得到数据，但是这样也存在一个明显的缺点，就是浪费宝贵的空间，所以还是要合理考虑，当然也可以想办法，比如对于有规律的键值，可以存储他们的始末值等等。

二、针对改良keys和smembers命令也可以使用scan命令