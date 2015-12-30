SequoiaSQL手动部署指南
=================

**前提条件**

- 安装SequoiaSQL到要部署的主机上；
- 多个主机上的安装路径、用户名要保持一致；
- 多个主机之间配置好hostname。

**部署**

1. 为所有主机设置SequoiaSQL环境变量。  
检查SequoiaSQL安装路径下的文件sequoiasql_path.sh中的变量GPHOME的路径是否为实际的SequoiaSQL安装路径，不符合的需要修改为实际路径。  
将sequoiasql_path.sh添加到用户的启动脚本~/.bashrc中，以安装路径/opt/sequoiasql为例：
  ```
  if [ -f /opt/sequoiasql/sequoiasql_path.sh ]; then
    . /opt/sequoiasql/sequoiasql_path.sh
  fi
  ```
执行```source ~/.bashrc```以使环境变量在当前环境中生效。

2. 设置各主机之间免密码登录。  
SequoiaSQL集群在初始化、启动和停止的过程中通过SSH执行远程命令，因此需要配置SSH免密码登录，避免过程中输入密码。  
SequoiaSQL提供的ssql命令可以在主机间快速交换SSH秘钥，只需要在主节点所在的主机执行下面的命令，其中hostfile为除主节点之外的所有主机的hostname文件，每行一个hostname。
  ```
  ssql ssh-exkeys -f hostfile
  ```
执行```ssh <hostname>```来检查SSH秘钥交换是否成功。

3. 配置SequoiaSQL。  
SequoiaSQL的配置文件保存在安装路径下的etc目录中。只需要在master节点修改配置文件，在集群初始化阶段配置参数会自动同步到其它节点。  
在sequoiasql-site.xml中根据部署规则设置master、segment的主机和端口号、数据目录、临时目录、HDFS URL等参数。主要参数见下表：

  | 参数 | 说明 |
  | :--: | :--: |
  | hawq_master_address_host | master的hostname |
  | hawq_master_address_port | master的端口号 |
  | hawq_segment_address_port | segment的端口号 |
  | hawq_dfs_url | 访问HDFS的URL |
  | hawq_master_directory | master的数据目录 |
  | hawq_segment_directory | segment的数据目录 |
  | hawq_master_temp_directory | master的临时目录 |
  | hawq_segment_temp_directory | segment的临时目录 |  
在slaves中添加segment节点的hostname，每行一个hostname。  
如果HDFS是以HA方式部署的集群，那么需要修改hdfs-client.xml中的配置，将注释掉的HA部分的配置参数修改为HDFS集群的实际值并去掉注释。 此时注意sequoiasql-site.xml中的参数hawq_dfs_url中的url要与dfs.nameservices保持一致。

4. 初始化SequoiaSQL集群。  
在初始化集群之前要保证hawq_dfs_url的地址是可访问的并且为空，如果hawq_master_directory和hawq_segment_directory使用的不是默认目录，那么需要手动创建。  
在master节点所在的主机上执行如下命令初始化集群：
  ```
  ssql init cluster
  ```

5. 启动SequoiaSQL集群。  
初始化集群成功之后执行如下命令启动集群：
  ```
  ssql start cluster
  ```

6. 验证集群。  
使用psql连接到SequoiaSQL执行操作以确认集群部署正确。
