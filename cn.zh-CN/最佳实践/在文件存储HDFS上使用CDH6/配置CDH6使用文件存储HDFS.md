# 配置CDH6使用文件存储HDFS {#task_1375604 .task}

本文介绍如何配置CDH上的HDFS服务、HIVE服务、SPARK服务、HBase服务来使用文件存储HDFS。

已完成数据迁移，详情请参见[CDH6数据迁移](cn.zh-CN/最佳实践/在文件存储HDFS上使用CDH6/CDH6数据迁移.md#)。

## 配置HDFS服务 {#section_6kl_374_87c .section}

1.  配置链接。 
    1.  在系统主页，选择**配置** \> **高级配置代码段**，进入高级配置代码段页面。
    2.  搜索`core-site.xml`，并选择**HDFS**。
    3.  在core-site.xml 的群集范围高级配置代码段（安全阀）中，添加如下配置项。 

        配置项fs.defaultFS，其值：dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290

        其中f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com为您的文件存储HDFS挂载点域名，需要根据实际情况进行修改。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095809/156767248353804_zh-CN.png)

    4.  单击**保存更改**。
2.  配置mapreduce.application.classpath。 
    1.  选择**YARN \(MR2 Included\)**，在其右侧的操作栏中，单击**配置**。
    2.  在配置页面，搜索mapreduce.application.classpath，查看默认的MR应用程序Classpath。 

        您可以将文件系统HDFS的SDK包放到默认的MR应用程序Classpath下，也可以添加新的存放目录。此处以添加新目录为例。

    3.  单击**+**，添加目录。 建议添加为HDFS服务存放jar包的路径，本案例的jar包的路径为/opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/lib/hadoop-hdfs/\*。

        **说明：** 如果添加了新路径，必选在目录后加上" /\* "或者直接添加文件系统HDFS的SDK包所在的绝对路径。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095809/156767248353805_zh-CN.png)

    4.  添加完成后，单击**保存更改**。
3.  更改mapred-site.xml配置。 
    1.  在系统主页，选择**配置** \> **高级配置代码段**，进入高级配置代码段页面。
    2.  搜索mapred-site.xml，并单击**YARN \(MR2 Included\)**。
    3.  在YARN 服务 MapReduce 高级配置代码段（安全阀）中，添加如下配置项。 

        配置项mapreduce.application.framework.path，其值：dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290//user/yarn/mapreduce/mr-framework/3.x.x-cdh6.x.x-mr-framework.tar.gz\#mr-framework

        其中，f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com为您的文件存储HDFS挂载点域名，需要根据实际情况进行修改。3.x.x-cdh6.x.x-mr-framework.tar.gz为该目录下实际存在的文件，根据实际情况进行更改。

        **说明：** 如果3.x.x-cdh6.x.x-mr-framework.tar.gz文件不存在，可能是因为此文件还没有从CDH的HDFS服务同步到文件存储HDFS。您需要将HDFS服务/user/yarn目录下的所有内容同步到文件存储HDFS上，请参考[CDH6数据迁移](cn.zh-CN/最佳实践/在文件存储HDFS上使用CDH6/CDH6数据迁移.md#)。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095809/156767248353822_zh-CN.png)

    4.  单击**保存更改**。
4.  部署新配置并重启服务。 
    1.  返回系统主页，找到**YARN**，单击重新部署图标，进行重新部署。
    2.  在过期配置页面，单击**重启过时服务**。
    3.  在重启过时服务页面，单击**立即重启**。
    4.  等待服务全部重启完成，并重新部署客户端配置后，单击**完成**。

## 配置Hive服务 {#section_06m_039_4o3 .section}

**说明：** 

配置HDFS服务完成后，才能配置Hive服务。

在配置Hive服务之前，请确认/user/hive/目录中的数据已完成全量迁移，迁移方法请参见[迁移开源HDFS的数据到文件存储HDFS](cn.zh-CN/最佳实践/迁移开源HDFS的数据到文件存储HDFS.md#)。

1.  修改元数据。 

    CDH6 Hive服务的元数据存储在Mysql，进入存储Hive元数据的Mysql数据库，修改DBS表和SDS表相应的值，如下所示。

    **说明：** 在进行元数据修改的时候，建议使用root用户，或者其他有权限的用户，避免因为权限问题导致修改失败。其中mysql服务的root用户密码是在搭建CDH服务时设置的密码。

    ``` {#codeblock_4j5_2au_62w}
    MySQL [(none)]> use cdh6hive ;
    
    #修改表“DBS”中的数据
    
    MySQL [cdh6hive]>  select * from DBS ;
    +-------+-----------------------+---------------------------------------------------------------------+--------------------------+------------+------------+
    | DB_ID | DESC                  | DB_LOCATION_URI                                                     | NAME                     | OWNER_NAME | OWNER_TYPE |
    +-------+-----------------------+---------------------------------------------------------------------+--------------------------+------------+------------+
    |     1 | Default Hive database | hdfs://hadoop9:8020/user/hive/warehouse                             | default                  | public     | ROLE       |
    |    13 | NULL                  | hdfs://hadoop9:8020/user/hive/warehouse/analysis_logs.db            | analysis_logs            | root       | USER       |
    |    14 | NULL                  | hdfs://hadoop9:8020/user/hive/warehouse/analysis_logs_report.db     | analysis_logs_report     | root       | USER       |
    |    15 | NULL                  | hdfs://hadoop9:8020/user/hive/warehouse/analysis_logs_report_old.db | analysis_logs_report_old | root       | USER       |
    +-------+-----------------------+---------------------------------------------------------------------+--------------------------+------------+------------+
    4 rows in set (0.00 sec)
    
    MySQL [cdh6hive]> UPDATE DBS SET DB_LOCATION_URI = "dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse"  WHERE DB_ID = 1 ;
    Query OK, 1 row affected (0.00 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
    
    MySQL [cdh6hive]> UPDATE DBS SET DB_LOCATION_URI = "dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db "  WHERE DB_ID = 13 ;
    Query OK, 1 row affected (0.00 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
    
    MySQL [cdh6hive]>
    MySQL [cdh6hive]> UPDATE DBS SET DB_LOCATION_URI = "dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs_report.db"  WHERE DB_ID = 14 ;
    Query OK, 1 row affected (0.00 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
    
    MySQL [cdh6hive]>
    MySQL [cdh6hive]> UPDATE DBS SET DB_LOCATION_URI = "dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs_report_old.db"  WHERE DB_ID = 15 ;
    Query OK, 1 row affected (0.00 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
    
    #修改表“SDS”中的数据
    
    MySQL [cdh6hive]> select *  from  SDS ;
    +-------+-------+---------------------------------------------------------------+---------------+---------------------------+---------------------------------------------------------------------------------------------------------------+-------------+----------------------------------------------------------------+----------+
    | SD_ID | CD_ID | INPUT_FORMAT                                                  | IS_COMPRESSED | IS_STOREDASSUBDIRECTORIES | LOCATION                                                                                                      | NUM_BUCKETS | OUTPUT_FORMAT                                                  | SERDE_ID |
    +-------+-------+---------------------------------------------------------------+---------------+---------------------------+---------------------------------------------------------------------------------------------------------------+-------------+----------------------------------------------------------------+----------+
    |    25 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | hdfs://hadoop9:8020/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned                          |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       25 |
    |    44 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | hdfs://hadoop9:8020/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=15 |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       44 |
    |    45 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | hdfs://hadoop9:8020/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=14 |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       45 |
    |    46 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | hdfs://hadoop9:8020/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=13 |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       46 |
    |    47 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | hdfs://hadoop9:8020/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=12 |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       47 |
    |    48 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | hdfs://hadoop9:8020/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=11 |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       48 |
    |    49 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | hdfs://hadoop9:8020/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=10 |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       49 |
    |    53 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | hdfs://hadoop9:8020/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=9  |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       53 |
    |    54 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | hdfs://hadoop9:8020/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=8  |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       54 |
    +-------+-------+---------------------------------------------------------------+---------------+---------------------------+---------------------------------------------------------------------------------------------------------------+-------------+----------------------------------------------------------------+----------+
    9 rows in set (0.00 sec)
    
    MySQL [cdh6hive]> UPDATE SDS SET LOCATION = "dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned" WHERE  SD_ID = 25 ;
    Query OK, 1 row affected (0.01 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
    
    MySQL [cdh6hive]> UPDATE SDS SET LOCATION = "dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=15" WHERE  SD_ID = 44 ;
    Query OK, 1 row affected (0.01 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
    
    MySQL [cdh6hive]> UPDATE SDS SET LOCATION = "dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=14" WHERE  SD_ID = 45 ;
    Query OK, 1 row affected (0.01 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
    
    MySQL [cdh6hive]> UPDATE SDS SET LOCATION = "dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=13" WHERE  SD_ID = 46 ;
    Query OK, 1 row affected (0.00 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
    
    MySQL [cdh6hive]> UPDATE SDS SET LOCATION = "dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=12" WHERE  SD_ID = 47 ;
    Query OK, 1 row affected (0.00 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
    
    MySQL [cdh6hive]> UPDATE SDS SET LOCATION = "dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=11" WHERE  SD_ID = 48 ;
    Query OK, 1 row affected (0.00 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
    
    MySQL [cdh6hive]> UPDATE SDS SET LOCATION = "dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=10" WHERE  SD_ID = 49 ;
    Query OK, 1 row affected (0.00 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
    
    MySQL [cdh6hive]> UPDATE SDS SET LOCATION = "dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=9" WHERE  SD_ID = 53 ;
    Query OK, 1 row affected (0.01 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
    
    MySQL [cdh6hive]> UPDATE SDS SET LOCATION = "dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=8" WHERE  SD_ID = 54 ;
    Query OK, 1 row affected (0.00 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
    
    MySQL [cdh6hive]>  select *  from  SDS ;
    +-------+-------+---------------------------------------------------------------+---------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+-------------+----------------------------------------------------------------+----------+
    | SD_ID | CD_ID | INPUT_FORMAT                                                  | IS_COMPRESSED | IS_STOREDASSUBDIRECTORIES | LOCATION                                                                                                                                           | NUM_BUCKETS | OUTPUT_FORMAT                                                  | SERDE_ID |
    +-------+-------+---------------------------------------------------------------+---------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+-------------+----------------------------------------------------------------+----------+
    |    25 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned                          |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       25 |
    |    44 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=15 |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       44 |
    |    45 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=14 |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       45 |
    |    46 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=13 |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       46 |
    |    47 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=12 |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       47 |
    |    48 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=11 |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       48 |
    |    49 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=10 |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       49 |
    |    53 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=9  |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       53 |
    |    54 |    14 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned/year=2019/month=7/day=8  |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |       54 |
    +-------+-------+---------------------------------------------------------------+---------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+-------------+----------------------------------------------------------------+----------+
    9 rows in set (0.00 sec)
    ```

2.  重启服务。 
    1.  返回系统主页，找到**Hive**，在其右侧的操作项中，单击**启动**。
    2.  在启动确认框中，单击**启动**。

## 配置Spark服务 {#section_h44_xtq_j8t .section}

**说明：** 

配置HDFS服务完成后，才能配置Spark服务。

配置Spark服务前，请确认/user/spark和/user/history目录中的数据已经完成了全量迁移。迁移方法请参见[ZH-CN\_TP\_1040909\_V1.dita\#task\_1305562](ZH-CN_TP_1040909_V1.dita#task_1305562)。

执行以下命令将文件系统HDFS的SDK包（aliyun-sdk-dfs-1.0.2-beta.jar \)，放置到CDH HDFS服务存储jar包的路径下。其中jar包放置目录，请根据实际值替换。

``` {#codeblock_fq1_a0c_aji}
cp aliyun-sdk-dfs-1.0.2-beta.jar /opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/lib/spark/jars/
```

一般情况下，放置到/opt/cloudera/parcels/CDH-6.x.x-x.cdh6.x.x.px.xxxxxx/lib/spark/jars/目录。例如：在CDH 6.0.1中，jar包放置目录为/opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/lib/spark/jars/。 

**说明：** 集群中的每台机器都需要在相同位置添加该SDK包。

## 配置Hbase服务 {#section_2v5_ql2_u1m .section}

**说明：** 

配置HDFS服务完成后，才能配置HBase服务。

配置Hbase服务前，请确认/hbase目录中的数据已经完成了全量迁移。迁移方法请参见[ZH-CN\_TP\_1040909\_V1.dita\#task\_1305562](ZH-CN_TP_1040909_V1.dita#task_1305562)。

1.  配置链接。 
    1.  在系统主页，选择**配置** \> **高级配置代码段**，进入高级配置代码段页面。
    2.  搜索hbase-site.xml，并选择**HBase** \> **HBase（服务范围）**。
    3.  在hbase-site.xml 的 HBase 服务高级配置代码段（安全阀）和hbase-site.xml 的 HBase 客户端高级配置代码段（安全阀）中，都要添加如下配置项。 

        -   配置项hbase.root.dir，其值：dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/hbase

            其中f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com为您的文件存储HDFS挂载点域名，需要根据实际情况进行修改。

        -   配置项hbase.unsafe.stream.capability.enforce，其值：false
        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095809/156767248353819_zh-CN.png)

2.  部署新配置并重启服务。 
    1.  返回系统主页，找到**Hbase**，单击重新部署图标，进行重新部署。
    2.  在过期配置页面，单击**重启过时服务**。
    3.  在重启过时服务页面，单击**立即重启**。
    4.  等待服务全部重启完成，并重新部署客户端配置后，单击**完成**。

## 关闭HDFS服务 {#section_hi3_ht7_hqb .section}

**说明：** 关闭HDFS服务前，请确认/user/yarn目录中的数据已经完成了全量迁移。迁移方法请参见[ZH-CN\_TP\_1040909\_V1.dita\#task\_1305562](ZH-CN_TP_1040909_V1.dita#task_1305562)。

1.  更改HDFS配置。 
    1.  在系统主页，找到**HDFS**，在其右侧的操作项中，单击**配置**。
    2.  在筛选器中，单击**NameNode**，找到**NameNode 数据目录**配置项，单击**-**，删除挂载磁盘的目录项，并单击**保存更改**。 

        **说明：** 建议只保留/opt/dfs/nn目录，您也可自行选择要保留的目录。

    3.  在筛选器中，单击**DataNode**，找到**DataNode 数据目录**配置项，单击**-**，删除挂载磁盘的目录项，并单击**保存更改**。 

        **说明：** 建议只保留/opt/dfs/dn目录，您也可自行选择要保留的目录。

2.  更改YARN配置。 
    1.  在系统主页，找到**YARN \(MR2 Included\)**，在其右侧的操作项中，单击**配置**。
    2.  在筛选器中，单击**NodeManager**，找到**NodeManager 容器日志目录**配置项，单击**-**，删除挂载磁盘的目录项，并单击**保存更改**。 建议只保留/opt/yarn/container-logs目录，您也可自行选择保留配置项。

        **说明：** 

        建议只保留/opt/yarn/container-logs目录，您也可自行选择要保留的目录。

        如果其他配置项中包含的路径，是要卸载的磁盘的挂载路径，请更换为其他系统路径。

3.  部署新配置并重启服务。 
    1.  返回系统主页，找到**HDFS**，单击重新部署图标，进行重新部署。
    2.  在过期配置页面，单击**重启过时服务**。
    3.  在重启过时服务页面，单击**立即重启**。
    4.  等待服务全部重启完成，并重新部署客户端配置后，单击**完成**。
4.  关闭HDFS服务。 
    1.  返回系统主页，找到**HDFS**，在其右侧的操作项中，单击**停止**。
    2.  在停止确认框中，单击**停止**。 此时，HDFS服务停止，但是其他服务正常运行。

## 验证服务正确性 {#section_9tc_q0u_nh5 .section}

-   hadoop的验证

    使用CDH hadoop中自带的测试包hadoop-mapreduce-examples-3.x.x-cdh6.x.x.jar进行测试。在CDH6.0.1中，该测试包在/opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/jars/下。

    1.  执行以下命令，在/tmp/randomtextwriter目录下生成128M大小的文件。

        ``` {#codeblock_nzq_eu2_2pp}
        yarn  jar /opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/jars/hadoop-mapreduce-examples-3.0.0-cdh6.0.1.jar  randomtextwriter  -D mapreduce.randomtextwriter.totalbytes=134217728  -D mapreduce.job.maps=2 -D mapreduce.job.reduces=2   /tmp/randomtextwriter
        ```

        其中hadoop-mapreduce-examples-3.0.0-cdh6.0.1.jar为cdh6.0.1中的测试包，请根据实际情况修改。

    2.  执行以下命令验证文件是否生成成功，从而验证文件系统实例的连通性。

        ``` {#codeblock_tt0_5g5_qlo}
        hadoop fs -ls dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/tmp/randomtextwriter 
        ```

        其中 f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com为您的文件存储HDFS挂载点域名，请根据实际情况修改。

        -   如果看到\_SUCCESS和part-m-00000两个文件，表示连通成功。如下所示：

            ``` {#codeblock_yf7_gpb_vg3}
            -rw-r-----   2 root root          0 2019-07-11 11:07 /tmp/randomtextwriter/_SUCCESS
            -rw-r-----   2 root root  137780424 2019-07-11 11:07 /tmp/randomtextwriter/part-m-00000
            ```

        -   如果连通失败（如：报错No such file or directory），请参见[../DNALIDFS19100383/ZH-CN\_TP\_159891\_V3.dita\#concept\_185689](../DNALIDFS19100383/ZH-CN_TP_159891_V3.dita#concept_185689)进行排查。
-   Spark的验证

    使用CDH spark中自带的测试包spark-examples\_2.11-2.x.x-cdh6.x.x.jar进行测试。在CDH6.0.1中，该测试包在/opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/jars/下。

    1.  执行以下命令，在/tmp/randomtextwriter目录下生成128M大小的文件。

        ``` {#codeblock_50i_z51_fpg}
        yarn  jar /opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/jars/hadoop-mapreduce-examples-3.0.0-cdh6.0.1.jar  randomtextwriter  -D mapreduce.randomtextwriter.totalbytes=134217728  -D mapreduce.job.maps=4 -D mapreduce.job.reduces=4   /tmp/randomtextwriter
        ```

        其中hadoop-mapreduce-examples-3.0.0-cdh6.0.1.jar为cdh6.0.1中的测试包，请根据实际情况修改。

    2.  使用spark测试包从文件存储HDFS上读取测试文件并按照word count的格式展示。

        ``` {#codeblock_gqh_kg6_432}
        spark-submit   --master yarn --executor-memory 2G --executor-cores 2  --class org.apache.spark.examples.JavaWordCount  /opt/cloudera/parcels/CDH/jars/spark-examples_2.11-2.2.0-cdh6.0.1.jar   /tmp/randomtextwriter
        ```

        如果回显信息类似如下图所示，表示配置成功。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095809/156767248453833_zh-CN.png)

-   Hive的验证
    1.  执行以下命令进入hive命令界面。

        ``` {#codeblock_whm_u63_143}
        [root@hadoop9 ~]# hive
        WARNING: Use "yarn jar" to launch YARN applications.
        SLF4J: Class path contains multiple SLF4J bindings.
        SLF4J: Found binding in [jar:file:/opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/jars/log4j-slf4j-impl-2.8.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
        SLF4J: Found binding in [jar:file:/opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/jars/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
        SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
        SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
        
        Logging initialized using configuration in jar:file:/opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/jars/hive-common-2.1.1-cdh6.0.1.jar!/hive-log4j2.properties Async: false
        
        WARNING: Hive CLI is deprecated and migration to Beeline is recommended.
        ```

    2.  执行以下命令创建测试表。

        ``` {#codeblock_ez7_pq4_ubj}
        hive> create table default.testTable(id int , name string )  row format delimited   fields terminated by '\t'  lines terminated by '\n';
        OK
        Time taken: 0.217 seconds
        ```

    3.  执行以下命令查看测试表。

        如果回显信息中的Location属性对应的值为文件存储HDFS的路径，则表示配置Hive成功。如果不是，请参见[配置Hive服务](#section_06m_039_4o3)进行重新配置。

        ``` {#codeblock_8sw_9ek_4si}
        hive> desc formatted  default.testTable ;
        OK
        # col_name              data_type               comment
        
        id                      int
        name                    string
        
        # Detailed Table Information
        Database:               default
        Owner:                  root
        CreateTime:             Wed Jul 17 15:44:29 CST 2019
        LastAccessTime:         UNKNOWN
        Retention:              0
        Location:               dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse/testtable
        .
        .
        .
        Time taken: 0.079 seconds, Fetched: 33 row(s)
        ```

-   HBase的验证
    1.  执行以下命令进入hbase shell命令界面。

        ``` {#codeblock_qww_1hk_94d}
        [root@hadoop9 ~]# hbase shell
        Java HotSpot(TM) 64-Bit Server VM warning: Using incremental CMS is deprecated and will likely be removed in a future release
        HBase Shell
        Use "help" to get list of supported commands.
        Use "exit" to quit this interactive shell.
        Version 2.0.0-cdh6.0.1, rUnknown, Wed Sep 19 09:14:00 PDT 2018
        Took 0.0036 seconds
        ```

    2.  在HBase中创建测试表。

        ``` {#codeblock_atv_k9z_hqx}
        hbase(main):002:0> create 'hbase_test','info'
        Created table hbase_test
        Took 108.4277 seconds
        => Hbase::Table - hbase_test
        hbase(main):003:0> list
        TABLE
        OnLine_MAXThroughput
        hbase_test
        2 row(s)
        Took 0.0049 seconds
        => ["OnLine_MAXThroughput", "hbase_test"]
        hbase(main):004:0> put 'hbase_test','1', 'info:name' ,'Sariel'
        Took 0.2336 seconds
        hbase(main):005:0> put 'hbase_test','1', 'info:age' ,'22'
        Took 0.0171 seconds
        hbase(main):006:0> put 'hbase_test','1', 'info:industry' ,'IT'
        Took 0.0166 seconds
        hbase(main):007:0>
        ```

    3.  执行以下命令查看文件存储HDFS的/hbase/data/default/路径，如果/hbase/data/default/路径下有hbase\_test目录，则证明配置链接成功。

        ``` {#codeblock_8qi_bat_j5r}
        hadoop fs -ls /hbase/data/default
        ```

        ![验证hbase](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095809/156767248453836_zh-CN.png)


## 常见问题 {#section_0fd_m8o_78r .section}

1.  SecondaryNameNode服务停止。

    在链接配置文件系统HDFS后， CDH中的HDFS服务会报错，报错信息：SecondaryNameNode服务无法启动。这是由于SecondaryNameNode服务需要通过http get方式获取NameNode的fsimage与edits文件进行工作。而文件存储HDFS不提供http服务，所以SecondaryNameNode服务无法启动。此时，CDH内的Spark\\Hive\\Hbase服务已经阿里云文件存储HDFS，CDH自身的HDFS问题不会影响系统运行。

2.  Hive Metastore canary创建Hue HDFS主目录失败。

    当CDH6中安装了HUE服务，则在配置重启hive服务后，可能有报错信息：Hive Metastore canary 创建 hue hdfs 主目录失败。Hive Metastore canary只是一个运行hive运行情况检测程序。HUE服务存在时，Hive Metastore canary会尝试创建hue目录，创建目录前会先检测目录权，要求权限为drwxrwxr-x，但是文件存储HDFS的目录权限为drwxrwxrwx，不符合hive在创建该目录的条件，所以无法创建目录。此错误不会影响文件存储HDFS的使用，也不会影响Hive服务和Hue服务的使用。


[卸载并释放CDH6 HDFS服务使用的云盘](cn.zh-CN/最佳实践/在文件存储HDFS上使用CDH6/卸载并释放CDH6 HDFS服务使用的云盘.md#)

