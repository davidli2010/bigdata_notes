HDFS高可用集群手动部署指南
===============

**软件版本**
- Hadoop 2.7.1
- JDK1.7以上

**前提条件**
- 安装Hadoop、JDK到要部署的主机上（可以先在一台主机上修改完配置文件，然后再复制到其它主机上）；
- 多个主机上的安装路径、用户名要保持一致；
- 多个主机之间配置好hostname；
- 多个主机之间配置SSH免密码登录
  - SequoiaSQL提供的ssql命令可以在主机间快速交换SSH秘钥，只需要在主节点所在的主机执行下面的命令，其中hostfile为除主节点之外的所有主机的hostname文件，每行一个hostname。  
  ```bash
  ssql ssh-exkeys -f hostfile
   ```  
   执行```ssh <hostname>```来检查SSH秘钥交换是否成功。

部署HDFS高可用集群（HA cluster）
------------------------------
Hadoop的配置文件位于安装路径下的etc目录。  

1. 配置hadoop-env.sh  
  - 修改JAVA_HOME的路径
  - HADOOP_LOG_DIR指定了Hadoop日志文件的存放路径

2. 配置core-site.xml  
  主要配置参数：

  | 参数 | 说明 |
  | --- | --- |
  | fs.defaultFS | HDFS的URL |
  | hadoop.tmp.dir | Hadoop使用的临时目录路径，默认为/tmp |

  例如：
  ```xml
  <configuration>
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://mycluster</value>
      </property>
      <property>
          <name>hadoop.tmp.dir</name>
          <value>/home/hadoop/data/temp</value>
      </property>
  </configuration>
  ```

3. 配置hdfs-site.xml  
  主要配置参数：

  | 参数 | 说明 |
  | --- | --- |
  | dfs.replication | 数据块的副本数，默认为3 |
  | dfs.namenode.name.dir | namenode数据文件存放路径 |
  | dfs.datanode.data.dir | datanode数据文件存放路径 |
  | dfs.nameservices | 集群的服务名称 |
  | dfs.ha.namenodes.[nameservice ID] | 集群的namenode列表，[nameservice ID]要与nameservices一致 |
  | dfs.namenode.rpc-address.[nameservice ID].[namenode ID] | namenode的远程方法调用的地址 |
  | dfs.namenode.http-address.[nameservice ID].[namenode ID] | namenode的http地址 |
  | dfs.namenode.shared.edits.dir | 为实施HA而存放edits日志的位置 |
  | dfs.journalnode.edits.dir | journal node的数据目录 |

  例如：
  ```xml
  <configuration>
      <property>
          <name>dfs.replication</name>
          <value>3</value>
      </property>
      <property>
          <name>dfs.namenode.name.dir</name>
          <value>file:///home/hadoop/data/name</value>
      </property>
      <property>
          <name>dfs.datanode.data.dir</name>
          <value>file:///home/hadoop/data/data</value>
      </property>
      <property>
          <name>dfs.nameservices</name>
          <value>mycluster</value>
      </property>
      <property>
          <name>dfs.ha.namenodes.mycluster</name>
          <value>nn1,nn2</value>
      </property>
      <property>
          <name>dfs.namenode.rpc-address.mycluster.nn1</name>
          <value>ubuntu-52:9000</value>
      </property>
      <property>
          <name>dfs.namenode.http-address.mycluster.nn1</name>
          <value>ubuntu-52:50070</value>
      </property>
      <property>
          <name>dfs.namenode.rpc-address.mycluster.nn2</name>
          <value>ubuntu-54:9000</value>
      </property>
      <property>
          <name>dfs.namenode.http-address.mycluster.nn2</name>
          <value>ubuntu-54:50070</value>
      </property>
      <property>
          <name>dfs.namenode.shared.edits.dir</name>
          <value>qjournal://ubuntu-52:8485;ubunubuntu-54:8485;ubuntu-56:8485/mycluster</value>
      </property>
      <property>
          <name>dfs.journalnode.edits.dir</name>
          <value>/home/hadoop/data/journal</value>
      </property>
  </configuration>
  ```

4. 修改slaves文件
  slaves中存放datanode节点的hostname，每行一个。
  例如：
  ```
  ubuntu-52
  ubuntu-54
  ```

5. 向所有要部署HDFS的主机复制配置文件，保证所有主机上的配置都是一致的。

6. 启动journalnode  
  在所有部署journal node的主机上启动journal node节点：
  ```bash
  <HADOOP_HOME>/sbin/hadoop_daemon.sh start journalnode
  ```

7. 格式化namenode主节点并启动  
  在namenode主节点所在的主机上执行下面命令格式化namenode：  
  ```bash
  hdfs namenode -format
  <HADOOP_HOME>/sbin/hadoop_daemon.sh start namenode
  ```

8. 启动namenode备节点  
  在namenode所在的主机上执行下面命令启动namenode：
  ```bash
  hdfs namenode -bootstrapStandby
  <HADOOP_HOME>/sbin/hadoop_daemon.sh start namenode
  ```

9. 将namenode主节点切换成active  
  ```bash
  hdfs haadmin -transitionToActive <namenode ID>
  ```

10. 启动datanode  
  在namenode所在的主机上执行下面命令启动datanode：
  ```bash
  <HADOOP_HOME>/sbin/hadoop_daemon.sh start datanode
  ```

11. 执行jps命令查看运行的节点  
  jps是java下查看进程的命令，类似于linux的ps命令。要执行jps命令需要将JAVA_HOME/bin设置到PATH路径中。

12. 启动和停止HDFS集群
  在任意主机上执行下面命令关闭HDFS集群：
  ```bash
  <HADOOP_HOME>/sbin/stop-dfs.sh
  ```
  在任意主机上执行下面命令启动HDFS集群，在启动之后需要手动将namenode切换成active：
  ```bash
  <HADOOP_HOME>/sbin/start-dfs.sh
  hdfs haadmin -transitionToActive <namenode ID>
  ```
  hdfs haadmin提供一组方法来检测HA的状态。

Failover手动切换
---------------
1. 在hdfs-site.xml中增加配置参数  

  | 参数 | 说明 |
  | --- | --- |
  | dfs.client.failover.proxy.provider.[nameservice ID] | 客户端判断哪个namenode是active时使用的Java类 |
  | dfs.ha.fencing.methods | 防止多个namenode成为active的方法，有sshfence和shell两种 |
  | dfs.ha.fencing.ssh.private-key-files | 使用sshfence方法时需要指定private key的存放路径  |
  ```xml
  <configuration>
      <property>
          <name>dfs.client.failover.proxy.provider.mycluster</name>
          <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
      </property>
      <property>
          <name>dfs.ha.fencing.methods</name>
          <value>sshfence</value>
      </property>
      <property>
          <name>dfs.ha.fencing.ssh.private-key-files</name>
          <value>/home/hadoop/.ssh/id_rsa</value>
      </property>
  </configuration>
  ```
2. 手动切换active namenode  
  ```
  hdfs haadmin -failover <namenode ID> <namenode ID>
  ```

Failover自动切换
----------------
使用Zookeeper可以实现failover自动切换。

1. 在hdfs-site.xml中增加配置参数  

  | 参数 | 说明 |
  | --- | --- |
  | dfs.ha.automatic-failover.enabled | 是否开启自动failover |
  | ha.zookeeper.quorum | Zookeeper节点列表 |  
  ```xml
  <configuration>
      <property>
          <name>dfs.ha.automatic-failover.enabled</name>
          <value>true</value>
      </property>
      <property>
          <name>ha.zookeeper.quorum</name>
          <value>ubuntu-52:2181,ubuntu-54:2181,ubuntu-56:2181</value>
      </property>
  </configuration>
  ```
2. 在Zookeeper中初始化HA状态  
  在任意一个节点执行下面命令来初始化HA的状态：  
  ```
  hdfs zkfc -formatZK
  ```
3. 配置自动failover之后，执行start-dfs.sh启动集群，不再需要手动切换namenode为active。
