HDFS手动部署指南
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

部署HDFS集群（cluster）
------------------------------
Hadoop的配置文件位于安装路径下的etc目录。  

1. 配置hadoop-env.sh  
  - 修改JAVA_HOME的路径

2. 配置core-site.xml  
  主要配置参数：

  | 参数 | 说明 |
  | --- | --- |
  | fs.defaultFS | HDFS的URL |
  | hadoop.tmp.dir | Hadoop使用的临时目录路径，默认为/tmp |
  | fs.checkpoint.period | secondary namenode唤醒检查点的周期，默认为3600秒 |
  | fs.checkpoint.size | secondary namenode在edits文件超过该大小时唤醒检查点，默认为67108864字节 |  
  例如：
  ```xml
  <configuration>
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://192.168.20.52:9000</value>
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
  | dfs.namenode.secondary.http_address | secondary namenode的http访问地址 |
  | dfs.datanode.data.dir | datanode数据文件存放路径 |  
  例如：
  ```xml
  <configuration>
      <property>
          <name>dfs.replication</name>
          <value>1</value>
      </property>
      <property>
          <name>dfs.namenode.name.dir</name>
          <value>file:///home/hadoop/data/name</value>
      </property>
      <property>
          <name>dfs.datanode.data.dir</name>
          <value>file:///home/hadoop/data/data</value>
      </property>
  </configuration>
  ```

4. 修改slaves文件
  slaves中存放namenode节点的hostname，每行一个。
  例如：
  ```
  ubuntu-52
  ubuntu-54
  ```

5. 向所有要部署HDFS的主机复制配置文件，保证所有主机上的配置都是一致的。

6. 格式化namenode
  在namenode所在的主机上执行下面命令格式化namenode：  
  ```bash
  hdfs namenode -format
  ```

7. 启动HDFS集群
  在namenode所在的主机上执行下面命令启动HDFS，其中HADOOP_HOME为Hadoop的安装路径：
  ```bash
  <HADOOP_HOME>/sbin/start-dfs.sh
  ```
