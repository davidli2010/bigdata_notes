SequoiaSQL PXF手动部署指南
=================

SequoiaSQL通过PXF访问外部数据源，例如HDFS、SequoiaDB。

**前提条件**
- SequoiaSQL集群已正确部署并启动

**部署**

1. 配置参数  
PXF的配置文件在SequoiaSQL安装目录下的pxf/conf目录下。
    - 环境变量  
        修改pxf-env.sh中的JAVA_HOME。  
        PXF_LOGDIR和PXF_RUNDIR是PXF服务用到的目录，在安装SequoiaSQL时已创建这两个目录。如果要修改，那么必须在初始化PXF服务前手动创建目录并赋给SequoiaSQL的用户名和用户组，同时需要修改pxf-log4j.properties。
    - classpath
        修改pxf-private.classpath文件，配置运行PXF所需要的JAVA类路径。例如：
    ```
    /opt/sequoiasql/pxf/conf
    /opt/hadoop-2.7.1//etc/hadoop/
    /opt/hadoop-2.7.1//share/hadoop/hdfs/hadoop-hdfs-2.7.1.jar
    /opt/hadoop-2.7.1//share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.7.1.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/hadoop-auth-2.7.1.jar
    /opt/hadoop-2.7.1//share/hadoop/common/hadoop-common-2.7.1.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/asm-3.2.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/avro-1.7.4.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/commons-cli-1.2.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/commons-codec-1.4.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/commons-collections-3.2.1.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/commons-configuration-1.6.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/commons-io-2.4.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/commons-lang-2.6.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/commons-logging-1.1.3.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/guava-11.0.2.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/htrace-core-3.1.0-incubating.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/jackson-core-asl-1.9.13.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/jackson-mapper-asl-1.9.13.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/jetty-6.1.26.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/jersey-core-1.9.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/jersey-server-1.9.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/log4j-1.2.17.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/protobuf-java-2.5.0.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/slf4j-api-1.7.10.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/snappy-java-1.0.4.1.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/netty-3.6.2.Final.jar
    /opt/hadoop-2.7.1//share/hadoop/common/lib/zookeeper-3.4.6.jar
    /opt/sequoiasql/pxf/lib/pxf-hdfs-*[0-9].jar
    /opt/sequoiasql/pxf/lib/pxf-sequoiadb-*[0-9].jar
    /opt/sequoiasql/pxf/lib/sequoiadb.jar
    ```
2. 启动PXF服务  
在root权限下初始化和启动PXF服务，PXF服务的默认端口号是51200。
    - 初始化PXF服务
    ```
    service pxf-service init
    ```
    - 启动PXF服务
    ```
    service pxf-service start
    ```

3. 在SequoiaSQL中创建表
    - 访问HDFS  
    例：有一个CSV文件，每行3列，前两列为整数，第三列为字符串，保存在HDFS的/pxf/aggregated中。
    ```
    create external table t1(id int, num int, comments varchar) location('pxf://192.168.20.52:51200/pxf/aggregated?PROFILE=HdfsTextSimple') format 'csv';
    ```
    - 访问SequoiaDB  
    例：SequoiaDB地址为：192.168.20.54:50000，集合为foo.bar，记录有整形字段a和b。
    ```
    create external table t2(a int, b int) location('pxf://192.168.20.52:51200/foo.bar?PROFILE=SequoiaDB&sdb=192.168.20.54:50000') format 'custom'(formatter='pxfwritable_import');
    ```

4. 创建表之后就可以使用SQL语法查询该表。不能在表上更新和删除记录。

5. 停止、重启PXF服务，查看PXF服务状态
    ```
    service pxf-service stop
    service pxf-service restart
    service pxf-service status
    ```

6. 卸载SequoiaSQL时备份pxf/conf目录，在重新安装SequoiaSQL时覆盖pxf/conf目录即可。
