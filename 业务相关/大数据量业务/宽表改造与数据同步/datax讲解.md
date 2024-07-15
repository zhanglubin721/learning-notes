# datax讲解

## 一. DataX简介

### 1\.1 DataX概述

`DataX` 是阿里巴巴开源的一个`异构数据源离线同步工具`，致力于实现包括`关系型数据库`(<mark>MySQL、Oracle 等)、HDFS、Hive、ODPS、HBase、FTP</mark>

等各种`异构数据源之间`稳定高效的`数据同步`功能。

源码地址：[阿里云DataX源码](<https://github.com/alibaba/DataX>)

### 1\.2 DataX支持的数据源

![请添加图片描述](<image/70f5c9cc0eb342ea9447dc2a63eebca1.png>)

## 二. DataX架构原理

### 2\.1 DataX设计理念

为了解决`异构数据源同步`问题，`DataX` 将复杂的`网状的同步链路`变成了`星型数据链路`，DataX 作为`中间传输载体`负责`连接各种数据源`。当需要接入一个新的数据源的时候，只需要`将此数据源对接`到`DataX`，便能跟已有的数据源做到无缝数据同步。

![请添加图片描述](<image/159c2389f6894118b8061653b4a8f131.png>)

### 2\.2 DataX框架设计

`DataX` 本身作为`离线数据同步框架`，采用 `Framework + plugin架构`构建。将`数据源读取和写入`抽象成为 `Reader/Writer 插件`，纳入到整个同步框架中。

![请添加图片描述](<image/2964df4d36bf4e42868e782d64b2e209.png>)

> 其中：
>
> <mark>Reader</mark>
>
> ：`数据采集`模块，负责采集数据源的数据，将数据`发送给Framework`。
>
> <mark>Writer</mark>
>
> ：`数据写入`模块，负责不断`向Framework取数据`，并将数据`写入到目的端`。
>
> <mark>Framework</mark>
>
> ：用于`连接reader和writer`，作为两者的数据传输通道，并处理`缓冲，流控，并发，数据转换`等核心技术问题。

### 2\.3 DataX运行流程

![请添加图片描述](<image/7092be22f5fb40f3a1d9bb9bbbbdffa3.png>)

> 注：
>
> <mark>Job</mark>
>
> ：`单个数据同步`的作业，称为`一个Job`，`一个Job`启动`一个进程`。
>
> <mark>Task</mark>
>
> ：根据不同数据源的切分策略，一个Job会切分为`多个Task`，`Task`是DataX作业的`最小单元`，`每个Task`负责`一部分数据的同步`工作。
>
> <mark>TaskGroup</mark>
>
> ：`Scheduler调度模块`会对`Task`进行`分组`，每个Task组称为一个`Task Group`。每个Task Group负责`以一定的并发度运行`其所分得的Task，`单个Task Group的并发度为5`。
>
> <mark>Reader–&gt;Channel–&gt;Writer</mark>
>
> ：每个Task启动后，都会固定启动Reader–>Channel–>Writer的线程来完成同步工作。

### 2\.4 DataX调度决策思路

举例来说，用户提交了一个 DataX 作业，并且配置了总的并发度为 20，目的是对一个有 100 张分表的 mysql 数据源进行同步。DataX 的调度决策思路是：

<mark>1）DataX Job 根据分库分表切分策略，将同步工作分成 100 个 Task。</mark>

<mark>2）根据配置的总的并发度 20，以及每个 Task Group 的并发度 5，DataX 计算共需要分配 4 个TaskGroup。</mark>

<mark>3）4 个 TaskGroup 平分 100 个 Task，每一个 TaskGroup 负责运行 25 个 Task。</mark>

> 注：`默认DataX会把Mysql中的1个表切分成1个Task。`

### 2\.5 DataX 与 Sqoop 对比

![请添加图片描述](<image/7825ad7edf9c49a1a41410f21398ec08.png>)

## 三. DataX使用

### 3\.1 DataX使用概述

#### 3\.1.1 DataX任务提交命令

用户只需根据自己同步数据的`数据源`和`目的地`选择`相应的Reader`和 `Writer`，并将 Reader 和 Writer 的信息`配置在一个 json 文件`中，然后执行如下命令提交数据同步任务即可：

![请添加图片描述](<image/a0a05c2206024b01895820ca5dc4959d.png>)

#### 3\.1.2 DataX配置文件格式

使用如下命令查看 `DataX 配置文件模板`：

![请添加图片描述](<image/29e83397e1404fe2b3558e528a53beff.png>)

配置文件模板如下，`json 最外层`是一个 `job`，`job` 包含`setting` 和 `content` 两部分，其中`setting` 用于对`整个 job 进行配置`，`content 用户配置数据源和目的地`。

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "mysqlreader",
                    "parameter": {
                        "column": [],
                        "connection": [
                            {
                                "jdbcUrl": [],
                                "table": []
                            }
                        ],
                        "password": "",
                        "username": "",
                        "where": ""
                    }
                },
                "writer": {
                    "name": "hdfswriter",
                    "parameter": {
                        "column": [],
                        "compress": "",
                        "defaultFS": "",
                        "fieldDelimiter": "",
                        "fileName": "",
                        "fileType": "",
                        "path": "",
                        "writeMode": ""
                    }
                }
            }
        ],
        "setting": {
            "speed": {
                "channel": ""
            }
        }
    }
}
```


![请添加图片描述](<image/e74934b5903a4e3999ed3bf59b973fed.png>)

Reader 和 Writer 的具体参数可参考官方文档，地址如下：[DataX配置](<https://github.com/alibaba/DataX/blob/master/README.md>)

### 3\.2 同步MySQL数据到HDFS案例

案例要求：<mark>同步 gmall 数据库中 base_province 表数据到 HDFS 的/base_province 目录</mark>

需求分析：要实现该功能，需选用 `MySQLReader` 和 `HDFSWriter`，`MySQLReader` 具有`两种模式`分别是 `TableMode` 和 `QuerySQLMode`，`前者`使用 `table,column,where 等属性`声明需要同步的数据；`后者`使用`一条 SQL 查询语句`声明需要同步的数据。

下面分别使用两种模式进行演示。

#### 3\.2.1 MySQLReader之TableMode

**1）编写配置文件**

<mark>（1）创建配置文件 base_province.json</mark>

![请添加图片描述](<image/cca232e8324e4ea391ac8c582cb9c93c.png>)

<mark>(2)内容如下：</mark>

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "mysqlreader",
                    "parameter": {
                        "column": [
                            "id",
                            "name",
                            "region_id",
                            "area_code",
                            "iso_code",
                            "iso_3166_2"
                        ],
                        "where": "id>=3",
                        "connection": [
                            {
                                "jdbcUrl": [
                                    "jdbc:mysql://hadoop102:3306/gmall"
                                ],
                                "table": [
                                    "base_province"
                                ]
                            }
                        ],
                        "password": "000000",
                        "splitPk": "",
                        "username": "root"
                    }
                },
                "writer": {
                    "name": "hdfswriter",
                    "parameter": {
                        "column": [
                            {
                                "name": "id",
                                "type": "bigint"
                            },
                            {
                                "name": "name",
                                "type": "string"
                            },
                            {
                                "name": "region_id",
                                "type": "string"
                            },
                            {
                                "name": "area_code",
                                "type": "string"
                            },
                            {
                                "name": "iso_code",
                                "type": "string"
                            },
                            {
                                "name": "iso_3166_2",
                                "type": "string"
                            }
                        ],
                        "compress": "gzip",
                        "defaultFS": "hdfs://hadoop102:8020",
                        "fieldDelimiter": "\t",
                        "fileName": "base_province",
                        "fileType": "text",
                        "path": "/base_province",
                        "writeMode": "append"
                    }
                }
            }
        ],
        "setting": {
            "speed": {
                "channel": 1
            }
        }
    }
}
```

**2）配置文件说明**

<mark>（1）Reader 参数说明</mark>

![请添加图片描述](<image/d20da5ca83184759860d4ca169516fc3.png>)

<mark>(2) Writer 参数说明</mark>

![请添加图片描述](<image/acd4d65c823040f4a9564bc65e1dfa40.png>)

> 注意事项：
>
> `HFDS Writer` 并未提供 `nullFormat` 参数：也就是`用户并不能自定义 null 值`写`到 HFDS 文件`中的存储格式。默认情况下，`HFDS Writer` 会将 `null 值存储`为`空字符串（''）`，而 `Hive`默认的`null 值存储格式`为`\N`。所以后期将 DataX 同步的文件导入 Hive 表就会出现问题。
>
> 解决该问题的方案有两个：
>
> 一是<mark>修改 DataX HDFS Writer 的源码，增加自定义 null 值存储格式的逻辑</mark>。
>
> 二是<mark>在 Hive 中建表时指定 null 值存储格式为空字符串（’’）</mark>
>
> ,如：

```json
DROP TABLE IF EXISTS base_province;
CREATE EXTERNAL TABLE base_province
(
 `id` STRING COMMENT '编号',
 `name` STRING COMMENT '省份名称',
 `region_id` STRING COMMENT '地区 ID',
 `area_code` STRING COMMENT '地区编码',
 `iso_code` STRING COMMENT '旧版 ISO-3166-2 编码，供可视化使用',
 `iso_3166_2` STRING COMMENT '新版 IOS-3166-2 编码，供可视化使用'
) COMMENT '省份表'
 ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
 NULL DEFINED AS ''
 LOCATION '/base_province/';
```


<mark>（3）Setting 参数说明</mark>

![请添加图片描述](<image/d92b3271a6834511bcb5c67db890ff76.png>)

**3）提交任务**

<mark>（1）在 HDFS 创建/base_province 目录</mark>

使用`DataX`向 `HDFS` 同步数据时，需`确保目标路径已存在`：

![请添加图片描述](<image/0c20b70d5fb2489eaa93b33ee1284a49.png>)

登录`HDFS-namenode节点`查看`目录生成`情况:

![请添加图片描述](<image/02c3cebee01e45c48b7462a64d952f95.png>)

<mark>（2）执行如下命令</mark>

![请添加图片描述](<image/8626c9da519941c090b0774cb3a18d7c.png>)

**4）查看结果**

<mark>（1）DataX 打印日志</mark>

![请添加图片描述](<image/0805576ac65c4e4d9d0707cfd167f5f2.png>)

查看`数据库原数据`，发现`原数据`有`34条`，而`DataX打印日志`最终`读出`只有`32条`：

![请添加图片描述](<image/e48e4a9eb5444864b6389674c7c85895.png>)

> 为什么出现上述情况呢？
>
> 原因是，我们的`base_province.json配置文件`中有`where限定条件`，该条件只`筛选表中id&gt;=3的数据`记录，所以`最终读出`的才是`32条！`
>
> ![请添加图片描述](<image/7ccc24cfc1c34cda9d38f221d9cebd94.png>)

<mark>（2）查看 HDFS 文件</mark>

![请添加图片描述](<image/5077b37c93c14da88b54a643241c0e80.png>)

![请添加图片描述](<image/b87bbf659b8946ba9bfecddd95673256.png>)

> 注：<mark>此处需要用管道，借助zcat命令查看数据，否则会出现乱码！</mark>

#### 3\.2.2 MySQLReader之QuerySQLMode

**1）编写配置文件**

<mark>（1）新增配置文件 base_province_sql.json</mark>

![请添加图片描述](<image/a51d92493dc648938fac99038a692518.png>)

<mark>（2）配置文件内容如下:</mark>

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "mysqlreader",
                    "parameter": {
                        "connection": [
                            {
                                "jdbcUrl": [
                                    "jdbc:mysql://hadoop102:3306/gmall"
                                ],
                                "querySql": [
                                    "select  id,name,region_id,area_code,iso_code,iso_3166_2 from base_province where id>=3"
                                ]
                            }
                        ],
                        "password": "000000",
                        "username": "root"
                    }
                },
                "writer": {
                    "name": "hdfswriter",
                    "parameter": {
                        "column": [
                            {
                                "name": "id",
                                "type": "bigint"
                            },
                            {
                                "name": "name",
                                "type": "string"
                            },
                            {
                                "name": "region_id",
                                "type": "string"
                            },
                            {
                                "name": "area_code",
                                "type": "string"
                            },
                            {
                                "name": "iso_code",
                                "type": "string"
                            },
                            {
                                "name": "iso_3166_2",
                                "type": "string"
                            }
                        ],
                        "compress": "gzip",
                        "defaultFS": "hdfs://hadoop102:8020",
                        "fieldDelimiter": "\t",
                        "fileName": "base_province",
                        "fileType": "text",
                        "path": "/base_province",
                        "writeMode": "append"
                    }
                }
            }
        ],
        "setting": {
            "speed": {
                "channel": 1
            }
        }
    }
}
```

**2）配置文件说明**

<mark>（1）Reader 参数说明</mark>

![请添加图片描述](<image/2649a31b97ff4df7bb9a95964d872be1.png>)

**3）提交任务**

<mark>（1）清空历史数据</mark>

![请添加图片描述](<image/d7a7a82340a64f0cbc07c01957eecc78.png>)

<mark>（2）执行如下命令</mark>

![请添加图片描述](<image/2ba590b2e12e4490a4061e11187156a0.png>)

**4）查看结果**

<mark>（1）DataX 打印日志</mark>

![请添加图片描述](<image/a24764a2bd2b4e91ae9cbc2d5a924f26.png>)

<mark>（2）查看 HDFS 文件</mark>

![请添加图片描述](<image/3ac3774ab8f14db496ddad6b5ea824dd.png>)

> 可以看到`MySQLReader`的`两种模式`最终都达到了`同样的效果`，那么这两种mode有什么`区别`?<br>
>
> 答：`TableMode只能查询一张表`，而`QuerySQLMode`可以通过`join等操作关联查询多张表`，并且可以完成更复杂的查询工作(<mark>通过修改json文件中reader模块中的querySql语句</mark>
>
> ).

#### 3\.2.3 DataX传参

通常情况下，`离线数据同步`任务需要`每日定时重复`执行，故 `HDFS`上的`目标路径`通常会`包含一层日期`，以对每日同步的数据加以区分，也就是说每日同步数据的`目标路径不是固定不变`的，因此 `DataX` 配置文件中 `HDFS Writer` 的`path`参数的值应该是`动态`的。为实现这一效果，就需要使用 `DataX 传参`的功能。

DataX 传参的用法如下，在 `JSON 配置文件`中使用`${param}`引用`参数`，在`提交任务`时使用`-p"-Dparam=value"`传入参数值，具体示例如下。

**1）编写配置文件**

<mark>（1）修改配置文件 base_province.json</mark>

![请添加图片描述](<image/a4487c5dcd6347f5a9327aa8cd142989.png>)

<mark>（2）配置文件内容如下</mark>

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "mysqlreader",
                    "parameter": {
                        "column": [
                            "id",
                            "name",
                            "region_id",
                            "area_code",
                            "iso_code",
                            "iso_3166_2"
                        ],
                        "where": "id>=3",
                        "connection": [
                            {
                                "jdbcUrl": [
                                    "jdbc:mysql://hadoop102:3306/gmall"
                                ],
                                "table": [
                                    "base_province"
                                ]
                            }
                        ],
                        "password": "000000",
                        "splitPk": "",
                        "username": "root"
                    }
                },
                "writer": {
                    "name": "hdfswriter",
                    "parameter": {
                        "column": [
                            {
                                "name": "id",
                                "type": "bigint"
                            },
                            {
                                "name": "name",
                                "type": "string"
                            },
                            {
                                "name": "region_id",
                                "type": "string"
                            },
                            {
                                "name": "area_code",
                                "type": "string"
                            },
                            {
                                "name": "iso_code",
                                "type": "string"
                            },
                            {
                                "name": "iso_3166_2",
                                "type": "string"
                            }
                        ],
                        "compress": "gzip",
                        "defaultFS": "hdfs://hadoop102:8020",
                        "fieldDelimiter": "\t",
                        "fileName": "base_province",
                        "fileType": "text",
                        "path": "/base_province/${dt}",
                        "writeMode": "append"
                    }
                }
            }
        ],
        "setting": {
            "speed": {
                "channel": 1
            }
        }
    }
}
```


**2）提交任务**

<mark>（1）创建目标路径</mark>

![请添加图片描述](<image/c81775ca64284343be49d2fc0faf5cba.png>)

<mark>（2）执行如下命令</mark>

![请添加图片描述](<image/5f1e445f5d3444ab993960074f6726c8.png>)

> 注：<mark>此处通过-p"-Ddt=2023-06-19"命令来实现传入到指定目录下。</mark>

**3）查看结果**

![请添加图片描述](<image/874ec23d886d4e048374a9888ba41a1a.png>)

### 3\.3 同步HDFS数据到MySQL案例

案例要求：<mark>同步 HDFS 上的/base_province 目录下的数据到 MySQL gmall 数据库下的test_province 表。</mark>

需求分析：要实现该功能，需选用 `HDFSReader` 和 `MySQLWriter`。

**1）编写配置文件**

<mark>（1）创建配置文件 test_province.json</mark>

![请添加图片描述](<image/bc243e361c604158a297d5b1002dfb7e.png>)

<mark>（2）配置文件内容如下</mark>

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "hdfsreader",
                    "parameter": {
                        "defaultFS": "hdfs://hadoop102:8020",
                        "path": "/base_province",
                        "column": [
                            "*"
                        ],
                        "fileType": "text",
                        "compress": "gzip",
                        "encoding": "UTF-8",
                        "nullFormat": "\\N",
                        "fieldDelimiter": "\t"
                    }
                },
                "writer": {
                    "name": "mysqlwriter",
                    "parameter": {
                        "username": "root",
                        "password": "000000",
                        "connection": [
                            {
                                "table": [
                                    "test_province"
                                ],
                                "jdbcUrl": "jdbc:mysql://hadoop102:3306/gmall?useUnicode=true&characterEncoding=utf-8"
                            }
                        ],
                        "column": [
                            "id",
                            "name",
                            "region_id",
                            "area_code",
                            "iso_code",
                            "iso_3166_2"
                        ],
                        "writeMode": "replace"
                    }
                }
            }
        ],
        "setting": {
            "speed": {
                "channel": 1
            }
        }
    }
}
```

**2）配置文件说明**

<mark>（1）Reader 参数说明</mark>

![请添加图片描述](<image/96c87e52a23247c38ec467a798b230dc.png>)

<mark>（2）Writer 参数说明</mark>

![请添加图片描述](<image/ece2c4a15e694c9c80733bb2991e6db7.png>)

> 注：此处`WriteMode`有`三种模式`，分别如下:
>
> `insert into(insert)`: 当要写入的Mysql表`无主键`时，为`正常写入`；当`有主键`时，一旦`出现重复数据`，会出现`数据重复异常`。
>
> `replace into(replace)` : 要求`写入的Mysql表必须有主键`，且当`主键`存在`重复`时，会`delete`对应`整行数据`，然后`再insert`。
>
> `ON DUPLICATE KEY UPDATE(update)`: 要求`写入的Mysql表必须有主键`，该模式是`在原有数据记录`的基础上`作修改`，而`不做删除`操作。

**3）提交任务**

<mark>（1）在 MySQL 中创建 gmall.test_province 表</mark>

```json
DROP TABLE IF EXISTS `test_province`;
CREATE TABLE `test_province` (
 `id` bigint(20) NOT NULL,
 `name` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT 
NULL,
 `region_id` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT 
NULL,
 `area_code` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT 
NULL,
 `iso_code` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT 
NULL,
 `iso_3166_2` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NULL 
DEFAULT NULL,
 PRIMARY KEY (`id`)
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = 
Dynamic;
```

<mark>（2）执行如下命令</mark>

![请添加图片描述](<image/88fc22473ce14046954a2406748f5c8d.png>)

**4）查看结果**

<mark>（1）DataX 打印日志</mark>

![请添加图片描述](<image/420b3f1cc0a740f3a1770a0574dfc8b0.png>)

<mark>（2）查看 MySQL 目标表数据</mark>

![请添加图片描述](<image/a2faf3d5c03d422ba6145b60e5609ae9.png>)

> <mark>为什么DataX日志中读出记录总数为64，而实际写入Mysql中的数据只有32条呢？</mark>
>
> 由于`DataX`从`HDFS`中`读取`的时候，会从`根目录`依次读取其中`文件`与`子目录`，而`2023-06-19子目录下`又保存有`一份`与根目录下gz文件`同样的文件`，所以`DataX`实际读取的数据为`2*32=64条`：
>
> ![请添加图片描述](<image/79019395da83404f952d36fb46538936.png>)而又因为`Writer写入的策略`选用为`replace`，该策略会`对重复的数据作覆盖`(<mark>仅保留其中一份数据</mark>
>
> )，所以`写入Mysql`中的数据就只有`32条`:<br>
>
> ![请添加图片描述](<image/d51c8b3829ef4fd1a4a017abfbd7ca37.png>)

## 四. DataX优化

### 4\.1 速度控制

DataX3.0 提供了包括`通道(并发)、记录流、字节流`三种`流控模式`，可以随意控制你的作业速度，让你的作业在数据库可以承受的范围内达到最佳的同步速度。

关键优化参数如下：

![请添加图片描述](<https://img-blog.csdnimg.cn/7a8e2d95d1eb46f0965a9fb8ee8be25b.png>)

> 注意事项：
>
> 1\.若`配置`了`总 record 限速`，则`必须配置单个 channel` 的 `record 限速`(<mark>避免类似“数据倾斜”的现象出现</mark>)
>
> 2\.若`配置`了`总 byte 限速`，则`必须配置单个 channe` 的 `byte 限速`(<mark>避免类似“数据倾斜”的现象出现</mark>)
>
> 3\.若`配置`了`总 record 限速`和`总 byte 限速`，`channel 并发数参数`就会`失效`。因为配置了总record 限速和总 byte 限速之后，`实际 channel 并发数是通过计算`得到的：
>
> 计算公式为:
>
> <mark>min(总 byte 限速/单个 channel 的 byte 限速，总 record 限速/单个 channel 的 record 限速)</mark>

### 4\.2 内存调整

当提升 DataX Job 内 `Channel 并发数`时，内存的占用会显著增加，因为 DataX 作为数据交换通道在内存中会缓存较多的数据。例如 Channel 中会有一个 Buffer，作为临时的数据交换的缓冲区，而在部分 Reader 和 Writer 的中，也会存在一些 Buffer，为了`防止 OOM 等错误`，需`调大JVM的堆内存`。

建议将内存设置为 4G 或者 8G，这个也可以根据实际情况来调整。

调整 JVM xms xmx 参数的两种方式：一种是`直接更改 datax.py 脚本`；另一种是`在启动`的时候，`加上对应的参数`，如下：

```json
python datax/bin/datax.py --jvm="-Xms8G -Xmx8G" /path/to/your/job.json
```

