# 配置E-MapReduce服务使用文件存储HDFS {#task_1495860 .task}

本文介绍如何配置E-MapReduce上的HDFS服务、HIVE服务、SPARK服务、HBase服务来使用文件存储HDFS。

已完成数据迁移，详情请参见[E-MapReduce数据迁移](cn.zh-CN/最佳实践/在文件存储HDFS上使用E-MapReduce/E-MapReduce数据迁移.md#)。

## 配置HDFS服务 {#section_gcy_mxs_h17 .section}

1.  登录[阿里云 E-MapReduce 控制台](https://emr.console.aliyun.com/?spm=a2c4g.11186623.2.8.56cb5168Vl7YY6)。
2.  在集群管理页面，找到需要挂载文件存储HDFS的目标E-MapReduce集群，单击**管理**。
3.  更改配置。 
    1.  选择**集群服务** \> **HDFS**，单击配置。
    2.  在服务配置中，单击**core-site**。
    3.  找到配置项fs.defaultFS，将其值替换为您的文件存储HDFS挂载点域名（dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290）。 

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1188468/156499252554310_zh-CN.png)

    4.  单击**保存**，在确认保存对话框中，输入执行原因，单击**确定**。
    5.  单击**部署客户端配置**，在确认保存对话框中，输入执行原因，单击**确定**。
4.  重启YARN服务。 
    1.  选择**集群服务** \> **YARN**。
    2.  在页面右侧的**操作**栏中，单击**重启 All Components**。

## 配置HIVE服务 {#section_yr9_1e9_y0q .section}

**说明：** 

配置HDFS服务完成后，才能配置Hive服务。

在配置Hive服务之前，请确认/user/hive/目录中的数据已完成全量迁移，迁移方法请参见[迁移开源HDFS的数据到文件存储HDFS](cn.zh-CN/最佳实践/迁移开源HDFS的数据到文件存储HDFS.md#)。

1.  更改配置。 
    1.  选择**集群管理** \> **HIVE**，单击配置。
    2.  在服务配置中，单击**hive-site**。
    3.  找到配置项hive.metastore.warehouse.dir，删除其对应值中的E-MapReduce HDFS文件系统域名，只保留/user/hive/warehouse。 

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1188468/156499252554308_zh-CN.png)

    4.  单击**保存**，在确认保存对话框中，输入执行原因，单击**确定**。
    5.  单击**部署客户端配置**，在确认保存对话框中，输入执行原因，单击**确定**。
2.  修改元数据。 
    1.  在**hivemetastore-site**中，获取数据库相关信息。 
        -   在配置项javax.jdo.option.ConnectionURL中，获取MySQL服务的主机名和元数据存储的数据库。
        -   在配置项javax.jdo.option.ConnectionUserName中，获取MySQL服务的用户名。
        -   在配置项javax.jdo.option.ConnectionPassword中，获取MySQL服务的用户密码。
    2.  Hive的元数据存储在MySQL中，进入存储Hive元数据的MySQL数据库hivemeta中，修改CTLGS表、DBS表和SDS表，如下所示。 

        ``` {#codeblock_t07_wb4_hnr}
        MariaDB [hivemeta]> use hivemeta;
        MariaDB [hivemeta]> select * from CTLGS ;
        +---------+------+---------------------------+-----------------------------------------------------------------------------+
        | CTLG_ID | NAME | DESC                      | LOCATION_URI                                                                |
        +---------+------+---------------------------+-----------------------------------------------------------------------------+
        |       1 | hive | Default catalog, for Hive | hdfs://emr-header-1.cluster-125428:9000/user/hive/warehouse |
        +---------+------+---------------------------+-----------------------------------------------------------------------------+
        1 row in set (0.00 sec)
        
        MariaDB [hivemeta]> UPDATE CTLGS
            -> SET     LOCATION_URI   = 'dfs://f-fad6b94cqnv2.cn-shanghai.dfs.aliyuncs.com:10290/user/hive/warehouse'
            -> WHERE     CTLG_ID = 1;
        Query OK, 0 rows affected (0.00 sec)
        Rows matched: 1  Changed: 0  Warnings: 0
        
        MariaDB [hivemeta]> select * from CTLGS ;
        +---------+------+---------------------------+-----------------------------------------------------------------------------+
        | CTLG_ID | NAME | DESC                      | LOCATION_URI                                                                |
        +---------+------+---------------------------+-----------------------------------------------------------------------------+
        |       1 | hive | Default catalog, for Hive | dfs://f-fad6b94cqnv2.cn-shanghai.dfs.aliyuncs.com:10290/user/hive/warehouse |
        +---------+------+---------------------------+-----------------------------------------------------------------------------+
        1 row in set (0.00 sec)
        
        ##再修改表“DBS”
        
        MariaDB [hivemeta]> select * from  DBS ;
         +-------+-----------------------+-----------------------------------------------------------------------------------------+--------------------------+------------+------------+-----------+
        | DB_ID | DESC                  | DB_LOCATION_URI                                                                         | NAME                     | OWNER_NAME | OWNER_TYPE | CTLG_NAME |
        +-------+-----------------------+-----------------------------------------------------------------------------------------+--------------------------+------------+------------+-----------+
        |     1 | Default Hive database | hdfs://emr-header-1.cluster-125428:9000/user/hive/warehouse                             | default                  | public     | ROLE       | hive      |
        |     2 | NULL                  | hdfs://emr-header-1.cluster-125428:9000/user/hive/warehouse/analysis_logs.db            | analysis_logs            | root       | USER       | hive      |
        |     3 | NULL                  | hdfs://emr-header-1.cluster-125428:9000/user/hive/warehouse/analysis_logs_report.db     | analysis_logs_report     | root       | USER       | hive      |
        |     4 | NULL                  | hdfs://emr-header-1.cluster-125428:9000/user/hive/warehouse/analysis_logs_report_old.db | analysis_logs_report_old | root       | USER       | hive      |
        +-------+-----------------------+-----------------------------------------------------------------------------------------+--------------------------+------------+------------+-----------+
        4 rows in set (0.00 sec)
        
        MariaDB [hivemeta]>UPDATE DBS 
            -> SET    DB_LOCATION_URI = 'dfs://f-fad6b94cqnv2.cn-shanghai.dfs.aliyuncs.com:10290/user/hive/warehouse'
            -> WHERE  DB_ID = 1;
        Query OK, 0 rows affected (0.00 sec)
        Rows matched: 1  Changed: 0  Warnings: 0
        
        MariaDB [hivemeta]>UPDATE DBS 
            -> SET    DB_LOCATION_URI = 'dfs://f-fad6b94cqnv2.cn-shanghai.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db'
            -> WHERE  DB_ID = 1;
        Query OK, 0 rows affected (0.00 sec)
        Rows matched: 1  Changed: 0  Warnings: 0
        
        MariaDB [hivemeta]>UPDATE DBS 
            -> SET    DB_LOCATION_URI = 'dfs://f-fad6b94cqnv2.cn-shanghai.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs_report.db'
            -> WHERE  DB_ID = 1;
        Query OK, 0 rows affected (0.00 sec)
        Rows matched: 1  Changed: 0  Warnings: 0
        
        MariaDB [hivemeta]>UPDATE DBS 
            -> SET    DB_LOCATION_URI = 'dfs://f-fad6b94cqnv2.cn-shanghai.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs_report_old.db'
            -> WHERE  DB_ID = 1;
        Query OK, 0 rows affected (0.00 sec)
        Rows matched: 1  Changed: 0  Warnings: 0
        
        ##修改表“SDS”
        
        MariaDB [hivemeta]> select * from  SDS  ;
        +-------+-------+---------------------------------------------------------------+---------------+---------------------------+-----------------------------------------------------------------------------------------------------------------------------------+-------------+----------------------------------------------------------------+----------+
        | SD_ID | CD_ID | INPUT_FORMAT                                                  | IS_COMPRESSED | IS_STOREDASSUBDIRECTORIES | LOCATION                                                                                                                          | NUM_BUCKETS | OUTPUT_FORMAT                                                  | SERDE_ID |
        +-------+-------+---------------------------------------------------------------+---------------+---------------------------+-----------------------------------------------------------------------------------------------------------------------------------+-------------+----------------------------------------------------------------+----------+
        |     1 |     1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | hdfs://emr-header-1.cluster-125428:9000/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned                          |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |        1 |
        |     2 |     2 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | hdfs://emr-header-1.cluster-125428:9000/user/hive/warehouse/analysis_logs.db/original_log_hz_partitioned                          |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |        2 |
        |     3 |     3 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat |               |                           | hdfs://emr-header-1.cluster-125428:9000/user/hive/warehouse/analysis_logs.db/original_log_sh_partitioned                          |          -1 | org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat |        3 |
        |    29 |    22 | org.apache.hadoop.mapred.TextInputFormat                      |               |                           | hdfs://emr-header-1.cluster-125428:9000/user/hive/warehouse/analysis_logs_report.db/hz_writethroughput_top_daily                  |          -1 | org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat     |       29 |
        ........
        |   548 |    80 | org.apache.hadoop.mapred.TextInputFormat                      |               |                           | hdfs://emr-header-1.cluster-125428:9000/user/hive/warehouse/analysis_logs_report.db/hz_readthroughput_top_yearly                  |          -1 | org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat     |      548 |
        |   549 |    81 | org.apache.hadoop.mapred.TextInputFormat                      |               |                           | hdfs://emr-header-1.cluster-125428:9000/user/hive/warehouse/analysis_logs_report_old.db/hz_writethroughput_top_yearly_20190709    |          -1 | org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat     |      549 |
        |   550 |    82 | org.apache.hadoop.mapred.TextInputFormat                      |               |                           | hdfs://emr-header-1.cluster-125428:9000/user/hive/warehouse/analysis_logs_report.db/hz_writethroughput_top_yearly                 |          -1 | org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat     |      550 |
        +-------+-------+---------------------------------------------------------------+---------------+---------------------------+------------------------------------------------------------------------------------------------------------------------
        536 rows in set (0.00 sec)
        
        MariaDB [hivemeta]> UPDATE SDS   SET  LOCATION = "dfs://f-fad6b94cqnv2.cn-shanghai.dfs.aliyuncs.com:10290/user/hive/warehouse/analysis_logs.db/original_log_bj_partitioned"  WHERE SD_ID = 1;
        Query OK, 0 rows affected (0.01 sec)
        Rows matched: 1  Changed: 0  Warnings: 0
        ......
        ```

3.  在页面右侧的**操作**栏，单击**重启 All Components**，重启服务。

## 配置Spark服务 {#section_4yd_oho_xqh .section}

**说明：** 

配置HDFS服务完成后，才能配置Spark服务。

配置Spark服务前，请确认/spark-history目录中的数据已经完成了全量迁移。迁移方法请参见[ZH-CN\_TP\_1040909\_V1.dita\#task\_1305562](ZH-CN_TP_1040909_V1.dita#task_1305562)。

1.  更改配置。 
    1.  选择**集群服务** \> **Spark**，单击配置。
    2.  在服务配置中，单击**spark-defaults**。
    3.  找到配置项spark\_eventlog\_dir，将其对应的值替换为您文件存储HDFS挂载点域名。 

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1188468/156499252554314_zh-CN.png)

    4.  单击**保存**，在确认保存对话框中，输入执行原因，单击**确定**。
    5.  单击**部署客户端配置**，在确认保存对话框中，输入执行原因，单击**确定**。
2.  放置SDK包。 

    将文件存储HDFS的SDK包（aliyun-sdk-dfs-1.0.2-beta.jar），放置到E-MapReduce Spark服务存放jar包的目录下。

    ``` {#codeblock_3rf_h8m_jdf}
    cp ~/aliyun-sdk-dfs-1.0.2-beta.jar    /opt/apps/ecm/service/spark/2.4.3-1.0.0/package/spark-2.4.3-1.0.0-bin-hadoop2.8/jars/
    ```

    一般情况下，放置到/opt/apps/ecm/service/spark/x.x.x-x.x.x/package/spark-x.x.x-x.x.x-bin-hadoopx.x/jars目录。

    **说明：** 集群中的每台机器都需要添加该SDK包。

3.  在页面右侧的**操作**栏，单击**重启 All Components**，重启服务。

## 配置HBase服务 {#section_wch_8dm_m4r .section}

**说明：** 

配置HDFS服务完成后，才能配置HBase服务。

配置HBase服务前，请确认/hbase目录中的数据已经完成了全量迁移。迁移方法请参见[ZH-CN\_TP\_1040909\_V1.dita\#task\_1305562](ZH-CN_TP_1040909_V1.dita#task_1305562)。

1.  更改配置。 
    1.  选择**集群服务** \> **Hbase**，单击配置。
    2.  在服务配置中，单击**Hbase-site**。
    3.  找到配置项hbase.rootdir，删除其对应值中的E-MapReduce HDFS文件系统域名，只保留/hbase。 

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1188468/156499252554318_zh-CN.png)

    4.  单击**保存**，在确认保存对话框中，输入执行原因，单击**确定**。
    5.  单击**部署客户端配置**，在确认保存对话框中，输入执行原因，单击**确定**。
2.  在页面右侧的**操作**栏，单击**重启 All Components**，重启服务。

## 关闭HDFS服务 {#section_yx6_i7c_x2w .section}

**说明：** 关闭HDFS服务前，请确认原来E-MapReduce HDFS上存储的数据都已经迁移到文件存储HDFS。迁移方法请参见[ZH-CN\_TP\_1040909\_V1.dita\#task\_1305562](ZH-CN_TP_1040909_V1.dita#task_1305562)。

1.  选择**集群服务** \> **HDFS**。
2.  在页面右侧的**操作**栏，单击**重启 All Components**，重启服务。
3.  在执行集群操作对话框中 ，输入执行原因，单击**确定**。

## 验证服务正确性 {#section_lfx_ptw_fgs .section}

-   hadoop的验证

    使用E-MapReduce hadoop中自带的测试包hadoop-mapreduce-examples-2.x.x.jar进行测试。该测试包默认放置在/opt/apps/ecm/service/hadoop/2.x.x-1.x.x/package/hadoop-2.x.x-1.x.x/share/hadoop/mapreduce/目录下。

    1.  执行以下命令，在/tmp/randomtextwriter目录下生成128M大小的文件。

        ``` {#codeblock_1r5_swl_5ou}
        hadoop jar /opt/apps/ecm/service/hadoop/2.8.5-1.3.1/package/hadoop-2.8.5-1.3.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.5.jar  randomtextwriter  -D mapreduce.randomtextwriter.totalbytes=134217728  -D mapreduce.job.maps=2 -D mapreduce.job.reduces=2   /tmp/randomtextwriter
        ```

        其中hadoop-mapreduce-examples-2.8.5.jar为E-MapReduce的测试包，请根据实际情况修改。

    2.  执行以下命令验证文件是否生成成功，从而验证文件系统实例的连通性。

        ``` {#codeblock_065_vyl_x88}
        hadoop fs -ls dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/tmp/randomtextwriter 
        ```

        其中 f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com为您的文件存储HDFS挂载点域名，请根据实际情况修改。

        -   如果看到\_SUCCESS和part-m-00000两个文件，表示连通成功。如下所示：

            ``` {#codeblock_cvm_28e_nim}
            -rwxrwxrwx   3 root root          0 2019-07-26 11:05 dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/tmp/randomtextwriter2/_SUCCESS
            -rwxrwxrwx   3 root root  137774743 2019-07-26 11:05 dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/tmp/randomtextwriter2/part-m-000000
            ```

        -   如果连通失败（如：报错No such file or directory），请参见[../DNALIDFS19100383/ZH-CN\_TP\_159891\_V3.dita\#concept\_185689](../DNALIDFS19100383/ZH-CN_TP_159891_V3.dita#concept_185689)进行排查。
-   Spark的验证

    使用E-MapReduce Spark中自带的测试包spark-examples\_2.x-2.x.x.jar进行测试。该测试包默认放置在/opt/apps/ecm/service/spark/2.x.x-1.0.0/package/spark-2.x.x-1.0.0-bin-hadoop2.8/examples/jars下。

    1.  执行以下命令，在/tmp/randomtextwriter目录下生成128M大小的文件。

        ``` {#codeblock_0rg_y63_vzo}
        hadoop jar /opt/apps/ecm/service/hadoop/2.8.5-1.3.1/package/hadoop-2.8.5-1.3.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.5.jar  randomtextwriter  -D mapreduce.randomtextwriter.totalbytes=134217728  -D mapreduce.job.maps=2 -D mapreduce.job.reduces=2   /tmp/randomtextwriter
        ```

        其中hadoop-mapreduce-examples-2.8.5.jar为E-MapReduce的测试包，请根据实际情况修改。

    2.  使用spark测试包从文件存储HDFS上读取测试文件并按照word count的格式展示。

        ``` {#codeblock_fbg_pi3_33m}
        spark-submit   --master yarn --executor-memory 2G --executor-cores 2  --class org.apache.spark.examples.JavaWordCount  /opt/apps/ecm/service/spark/2.4.3-1.0.0/package/spark-2.4.3-1.0.0-bin-hadoop2.8/examples/jars/spark-examples_2.11-2.4.3.jar   /tmp/randomtextwriter
        ```

        如果回显信息类似如下图所示，表示配置成功。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1188468/156499252654282_zh-CN.png)

-   Hive的验证
    1.  执行以下命令进入Hive命令界面。

        ``` {#codeblock_kfk_ah4_u90}
        [root@hadoop9 ~]# hive
        Hive Session ID = 9d1aeaf3-19d8-461b-952f-6fcfed900e69
        
        Logging initialized using configuration in file:/etc/ecm/hive-conf-3.1.1-1.1.6/hive-log4j2.properties Async: true
        Hive Session ID = 9c564244-96b1-4865-8e0f-21631a2e97f1
        hive>
        ```

    2.  执行以下命令创建测试表。

        ``` {#codeblock_y40_vy1_qv6}
        hive> create table default.testTable(id int , name string )  row format delimited   fields terminated by '\t'  lines terminated by '\n';
        OK
        Time taken: 0.536 seconds
        ```

    3.  执行以下命令查看测试表。

        如果回显信息中的Location属性对应的值为文件存储HDFS的路径，则表示配置Hive成功。如果不是，请参见[配置HIVE服务](#section_yr9_1e9_y0q)进行重新配置。

        ``` {#codeblock_22w_beq_oog}
        hive>  desc formatted  default.testTable ;
        OK
        2019-07-26 11:23:25,133 INFO  [9d1aeaf3-19d8-461b-952f-6fcfed900e69 main] mapred.FileInputFormat: Total input files to process : 1
        # col_name              data_type               comment
        id                      int
        name                    string
        
        # Detailed Table Information
        Database:               default
        OwnerType:              USER
        Owner:                  root
        CreateTime:             Fri Jul 26 11:23:12 CST 2019
        LastAccessTime:         UNKNOWN
        Retention:              0
        Location:               dfs://f-fad6b94cqnv2.cn-shanghai.dfs.aliyuncs.com:10290/user/hive/warehouse/testtable
        Table Type:             MANAGED_TABLE
        Table Parameters:
                COLUMN_STATS_ACCURATE   {\"BASIC_STATS\":\"true\",\"COLUMN_STATS\":{\"id\":\"true\",\"name\":\"true\"}}
                bucketing_version       2
                numFiles                0
                numRows                 0
                rawDataSize             0
                totalSize               0
                transient_lastDdlTime   1564111392
        
        # Storage Information
        SerDe Library:          org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe
        InputFormat:            org.apache.hadoop.mapred.TextInputFormat
        OutputFormat:           org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat
        Compressed:             No
        Num Buckets:            -1
        Bucket Columns:         []
        Sort Columns:           []
        Storage Desc Params:
                field.delim             \t
                line.delim              \n
                serialization.format    \t
        Time taken: 0.134 seconds, Fetched: 34 row(s)
        ```

-   HBase的验证
    1.  执行以下命令进入hbase shell命令界面。

        ``` {#codeblock_65h_4kw_6p7}
        [root@emr-header-1 ~]# hbase shell
        SLF4J: Class path contains multiple SLF4J bindings.
        SLF4J: Found binding in [jar:file:/opt/apps/ecm/service/hbase/1.4.9-1.0.0/package/hbase-1.4.9-1.0.0/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
        SLF4J: Found binding in [jar:file:/opt/apps/ecm/service/hadoop/2.8.5-1.3.1/package/hadoop-2.8.5-1.3.1/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
        SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
        SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
        HBase Shell
        Use "help" to get list of supported commands.
        Use "exit" to quit this interactive shell.
        Version 1.4.9, r8214a16c5d80f077abf1aa01bb312851511a2b15, Thu Jan 31 20:35:22 CST 2019
        ```

    2.  在HBase中创建测试表。

        ``` {#codeblock_wbi_gtq_xs2}
        hbase(main):001:0> create 'hbase_test','info'
        0 row(s) in 1.5620 seconds
        
        => Hbase::Table - hbase_test
        hbase(main):002:0>  put 'hbase_test','1', 'info:name' ,'Sariel'
        0 row(s) in 0.2990 seconds
        
        hbase(main):003:0> put 'hbase_test','1', 'info:age' ,'22'
        0 row(s) in 0.0230 seconds
        
        hbase(main):004:0> put 'hbase_test','1', 'info:industry' ,'IT'
        0 row(s) in 0.0740 seconds
        ```

    3.  执行以下命令查看文件存储HDFS的/hbase/data/default/路径，如果/hbase/data/default/路径下有hbase\_test目录，则证明配置链接成功。

        ``` {#codeblock_l7k_1b6_iga}
        hadoop fs -ls /hbase/data/default
        ```

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1188468/156499252654285_zh-CN.png)


[卸载并释放E-MapReduce HDFS使用的云盘](cn.zh-CN/最佳实践/在文件存储HDFS上使用E-MapReduce/卸载并释放E-MapReduce HDFS使用的云盘.md#)

