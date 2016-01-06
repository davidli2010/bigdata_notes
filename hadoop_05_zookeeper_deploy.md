Zookeeper手动部署指南
====================

Zookeeper的配置文件保存在安装目录下的conf目录中，可以将zoo_sample.cfg复制为zoo.cfg来快速生成配置文件。

单机模式
--------
进入conf目录创建配置文件zoo.cfg。
```
tickTime=2000
dataDir=/home/hadoop/zookeeper
clientPort=2181
```

参数说明：
- tickTime：Zookeeper中使用的基本时间单位，单位为毫秒
- dataDir：数据目录
- dataLogDir：日志目录，如果没有设置该参数，将使用和dataDir相同的设置
- clientPort：监听客户端连接的端口号

启动server：
```bash
bin/zkServer.sh start
```

启动客户端：
```bash
bin/zkCli.sh -server localhost:2181
```
集群模式
--------
进入conf目录创建配置文件zoo.cfg。
```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/hadoop/zookeeper
clientPort=2181
server.1=ubuntu-52:2888:3888
server.2=ubuntu-54:2888:3888
server.3=ubuntu-56:2888:3888
```

新增了几个参数：
- initLimit：Zookeeper集群中的包含多台server,其中一台为leader, 集群中其余的server为follower。initLimit参数配置初始化连接时,follower和leader之间的最长心跳时间。此时该参数设置为10,说明时间限制为5倍tickTime, 即10*2000=20000ms=20s。
- syncLimit：该参数配置leader和follower之间发送消息,请求和应答的最大时间长度。此时该参数设置为5, 说明时间限制为5倍tickTime, 即10000ms。
- server.X=A:B:C：其中X是一个数字, 表示这是第几号server。A是该server所在主机的hostname或IP地址。 B配置该server和集群中的leader交换消息所使用的端口。C配置选举leader时所使用的端口。

如果server位于同一台主机上，那么要保证每个server的数据目录和使用的端口号是不同的。

在设置的dataDir中新建myid文件,写入一个数字,该数字表示这是第几号server。该数字必须和zoo.cfg文件中的server.X中的X一一对应。

分别启动三个server。客户端可以连接到任意一个server。
