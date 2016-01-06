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

部署HDFS高可用集群（cluster ha）
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
          <value>file:///home/hadoop/data/journal</value>
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

7. 格式化namenode  
  在namenode所在的主机上执行下面命令格式化namenode：  
  ```bash
  hdfs namenode -format
  ```

8. 启动namenode  
  在namenode所在的主机上执行下面命令启动namenode：
  ```bash
  <HADOOP_HOME>/sbin/hadoop_daemon.sh start namenode
  ```

9. 将namenode切换成active  
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
