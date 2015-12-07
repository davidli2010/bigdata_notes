Apache™ Hadoop®
======

- 分布式系统基础架构；
- 实现了一个分布式文件系统HDFS，适合超大数据集，可以流式访问数据；
- 核心设计：HDFS（海量存储）和MapReduce（计算）；
- 受Google的MapReduce和GFS启发；
- 特点：
    - 高可靠性
    - 高扩展性
    - 高效性
    - 高容错性
    - 低成本

**子项目**

- HDFS：分布式文件系统
- MapReduce： 并行计算框架
- HBASE： 分布式NoSQL列数据库，类似BigTable
- Hive：数据仓库工具
- ZooKeeper：分布式锁服务
- Avro：数据序列化格式与传输工具
- Pig：大数据分析平台
- Ambari： Hadoop管理工具，可以快捷的监控、部署和管理集群
- Sqoop：于Hadoop与传统数据库间进行数据的传递
