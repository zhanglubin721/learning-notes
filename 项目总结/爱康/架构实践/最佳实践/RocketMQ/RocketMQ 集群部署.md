**系统部署架构，使用的模式是 2m-2s-sync**



**broker 机器信息**

- Master 1：172.18.24.5
- Slave 1：172.18.24.6
- Master 2：172.18.24.7
- Slave 2：172.18.24.8

**namesrv 机器信息（这里复用了 3 台主机做 Name Server。正式生产环境应该要考虑配置单独的 3 台机器做 Name Server）**

- Name Server 1：172.18.24.5
- Name Server 2：172.18.24.6
- Name Server 3：172.18.24.7

**内核参数优化**

系统I/O和虚拟内存设置

echo 'vm.overcommit_memory=1' >> /etc/sysctl.conf

echo 'vm.min_free_kbytes=5000000' >> /etc/sysctl.conf

echo 'vm.drop_caches=1' >> /etc/sysctl.conf

echo 'vm.zone_reclaim_mode=0' >> /etc/sysctl.conf

echo 'vm.max_map_count=655360' >> /etc/sysctl.conf

echo 'vm.dirty_background_ratio=50' >> /etc/sysctl.conf

echo 'vm.dirty_ratio=50' >> /etc/sysctl.conf

echo 'vm.page-cluster=3' >> /etc/sysctl.conf

echo 'vm.dirty_writeback_centisecs=360000' >> /etc/sysctl.conf

echo 'vm.swappiness=10' >> /etc/sysctl.conf



系统文件句柄设置

echo 'ulimit -n 1000000' >> /etc/profile

echo 'admin hard nofile 1000000' >> /etc/security/limits.conf



系统I/O调度算法

deadline

**安装 jdk 环境（在 4 台主机 Master 1、Slave 1、Master 2、Slave 2 上操作）**

```bash
// 到 oracle 官网下载好 jdk 安装包
# ls jdk-8u181-linux-x64.tar.gz 
jdk-8u181-linux-x64.tar.gz
# tar xvf jdk-8u181-linux-x64.tar.gz -C /usr/local/
# vim /etc/profile
export JAVA_HOME=/usr/local/jdk1.8.0_181
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
# source /etc/profile
```

**安装 rocketmq（在 4 台主机上操作）**

```cpp
# cd /usr/local/src/
# wget https://mirrors.tuna.tsinghua.edu.cn/apache/rocketmq/4.7.1/rocketmq-all-4.7.1-bin-release.zip
# unzip rocketmq-all-4.7.1-bin-release.zip -d /usr/local/
# cd /usr/local/rocketmq-all-4.7.1-bin-release/
```

**配置 rocketmq broker-a（在 Master 1 上操作）**

```bash
# vim conf/2m-2s-sync/broker-a.properties
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=SYNC_MASTER
flushDiskType=ASYNC_FLUSH
namesrvAddr=172.18.24.5:9876;172.18.24.6:9876;172.18.24.7:9876
```

**配置 rocketmq broker-a-s（在 Slave 1 上操作）**

```bash
# vim conf/2m-2s-sync/broker-a-s.properties
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
namesrvAddr=172.18.24.5:9876;172.18.24.6:9876;172.18.24.7:9876
```

**配置 rocketmq broker-b（在 Master 2 上操作）**

```bash
# vim conf/2m-2s-sync/broker-b.properties
brokerClusterName=DefaultCluster
brokerName=broker-b
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=SYNC_MASTER
flushDiskType=ASYNC_FLUSH
namesrvAddr=172.18.24.5:9876;172.18.24.6:9876;172.18.24.7:9876
```

**配置 rocketmq broker-b-s（在 Slave 2 上操作）**

```bash
# vim conf/2m-2s-sync/broker-b-s.properties
brokerClusterName=DefaultCluster
brokerName=broker-b
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
namesrvAddr=172.18.24.5:9876;172.18.24.6:9876;172.18.24.7:9876
```

**在 4 台主机上操作，修改bin/runserver.sh文件， 更改默认日志路径和JVM大小**

```dart
JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g"
JAVA_OPT="${JAVA_OPT} -XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRefLRUPolicyMSPerMB=0"
JAVA_OPT="${JAVA_OPT} -verbose:gc -Xloggc:${GC_LOG_DIR}/rmq_broker_gc_%p_%t.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime -XX:+PrintAdaptiveSizePolicy"
JAVA_OPT="${JAVA_OPT} -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m"
JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow"
JAVA_OPT="${JAVA_OPT} -XX:+AlwaysPreTouch"
JAVA_OPT="${JAVA_OPT} -XX:MaxDirectMemorySize=15g"
JAVA_OPT="${JAVA_OPT} -XX:-UseLargePages -XX:-UseBiasedLocking"
JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${JAVA_HOME}/jre/lib/ext:${BASE_DIR}/lib:${JAVA_HOME}/lib/ext"
#JAVA_OPT="${JAVA_OPT} -Xdebug -Xrunjdwp:transport=dt_socket,address=9555,server=y,suspend=n"
JAVA_OPT="${JAVA_OPT} ${JAVA_OPT_EXT}"
JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}" 
```

**启动集群**
启动 Name Server，在 3 台 Name Server 上操作

在每台机器上创建目录 mkdir `/data/logs/rocketmqlogs/`

```ruby
nohup sh bin/mqnamesrv > /data/logs/rocketmqlogs/mqnamesrv.log 2>&1 & 
```

启动 Master 1，在 Master 1 上操作

```bash
# sh bin/mqbroker -c conf/2m-2s-sync/broker-a.properties > /data/logs/rocketmqlogs/broker-a.log 2>&1 &
```

启动 Slave 1，在 Slave 1 上操作

```bash
# sh bin/mqbroker -c conf/2m-2s-sync/broker-a-s.properties > /data/logs/rocketmqlogs/broker-a-s.log 2>&1 &
```

启动 Master 2，在 Master 2 上操作

```bash
# sh bin/mqbroker -c conf/2m-2s-sync/broker-b.properties > /data/logs/rocketmqlogs/broker-b.log 2>&1 &
```

启动 Slave 2，在 Slave 2 上操作

```bash
# sh bin/mqbroker -c conf/2m-2s-sync/broker-b-s.properties > /data/logs/rocketmqlogs/broker-b-s.log 2>&1 &
```

**查看 NameServer（9876） 和 Broker（10909、10911、10912） 启动情况，在 4 台主机上操作**

```ruby
# netstat -nltup|egrep "9876|10909|10911|10912"
tcp        0      0 0.0.0.0:9876            0.0.0.0:*               LISTEN      1560/java           
tcp        0      0 0.0.0.0:10909           0.0.0.0:*               LISTEN      1775/java           
tcp        0      0 0.0.0.0:10911           0.0.0.0:*               LISTEN      1775/java           
tcp        0      0 0.0.0.0:10912           0.0.0.0:*               LISTEN      1775/java 
```

**测试集群生产消费情况**

```csharp
// 在 4 台主机上操作
# export NAMESRV_ADDR='172.18.24.5:9876;172.18.24.6:9876;172.18.24.7:9876'
// 在其中 3 台机器上操作，启动 3 个消费者
# sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
// 在其中 1 台机器上操作，启动一个生产者，在 3 个消费者机器就可以看到消费信息
# sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
```