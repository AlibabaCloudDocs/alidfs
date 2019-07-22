# 文件存储HDFS和对象存储OSS双向数据迁移 {#task_1012751 .task}

本文档介绍文件存储HDFS和对象存储OSS之间的数据迁移过程。您可以将文件存储HDFS数据迁移到对象存储OSS，也可以将对象存储OSS的数据迁移到文件存储HDFS上。

阿里云文件存储HDFS是面向阿里云ECS实例及容器服务等计算资源的文件存储服务。文件存储HDFS允许您就像在Hadoop分布式文件系统中管理和访问数据，并对热数据提供高性能的数据访问能力。对象存储OSS是海量、安全、低成本、高可靠的云存储服务，并提供标准型、归档型等多种存储类型供选择。客户可以在文件存储HDFS和对象存储OSS之间实现数据迁移，从而实现热、温、冷数据的合理分层，在实现对热数据的高性能访问的同时，有效控制存储成本。

## 准备工作 {#section_ouf_urd_km4 .section}

1.  挂载文件系统，详情请参见[../DNalidfs1853915/ZH-CN\_TP\_19052\_V6.dita\#task\_vmg\_jtk\_z2b](../DNalidfs1853915/ZH-CN_TP_19052_V6.dita#task_vmg_jtk_z2b)。
2.  验证文件系统和计算节点之间的连通性。
    1.  执行以下命令，在文件存储HDFS上创建目录（如：/dfs\_links）。

        ``` {#codeblock_ayz_mt9_yvv}
        hadoop fs -mkdir /dfs_links
        ```

    2.  执行以下命令，验证连通性。

        如果命令正常执行无输出结果，则表示连通成功。如果连通失败，请参见[购买文件系统实例后，为什么无法访问文件存储HDFS？](../../../../cn.zh-CN/常见问题/一般性问题/创建文件系统实例后，为什么无法访问文件存储HDFS？.md#)进行排查。

        ``` {#codeblock_dna_wto_7b3}
        hadoop fs -ls dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/dfs_links
        ```

        其中f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com为文件存储HDFS挂载点域名，请根据实际情况进行修改。

    3.  准备迁移工具。
        1.  单击[emr-tools](https://yq.aliyun.com/attachment/download/?spm=5176.100239.blogcont78093.18.BfNz7d&id=1956)下载迁移工具安装包。
        2.  将迁移工具安装包上传计算节点的本地目录。

            **说明：** 该计算节点必须运行着Hadoop的YARN服务，或者是YARN集群中可以提交作业的计算节点。因为[emr-tools](https://yq.aliyun.com/attachment/download/?spm=5176.100239.blogcont78093.18.BfNz7d&id=1956)迁移工具需要借助Hadoop数据迁移工具DistCp实现数据的迁移。

        3.  执行以下命令，解压安装包。

            ``` {#codeblock_yvp_bi1_uky}
            tar jxf emr-tools.tar.bz2
            ```


## 将文件存储HDFS数据迁移到对象存储OSS {#section_8wy_88g_qex .section}

1.  进入emr-tools工具安装包解压后所在的目录，使用`hdfs2oss4emr.sh`脚本将文件存储HDFS上的数据迁移到对象存储OSS上。具体命令如下所示：

    ``` {#codeblock_kcs_zj9_s5p}
    cd emr-tools
    ./hdfs2oss4emr.sh \
    dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/HDFS2OSS/data/data_1000g \
    oss://accessKeyId:accessKeySecret@bucket-name.oss-cn-hangzhou.aliyuncs.com/HDFS2OSS/data/data_1000g
    ```

    参数说明如下表所示。

    |参数|说明|
    |--|--|
    |accessKeyId|访问对象存储OSS API的密钥。获取方式请参见[如何获取如何获取AccessKeyId和AccessKeySecret](https://help.aliyun.com/knowledge_detail/48699.html)|
    |accessKeySecret|
    |bucket-name.oss-cn-hangzhou.aliyuncs.com|对象存储OSS的访问域名，包括bucket名称和endpoint地址。|

    执行以上命令后，系统将启动一个Hadoop MapReduce任务（DistCp）。

2.  任务执行完成后，查看迁移结果。

    如果回显包含如下类似信息，说明迁移成功。

    ``` {#codeblock_764_lnv_rsq}
    19/03/27 08:48:58 INFO mapreduce.Job: Job job_1553599949635_0014 completed successfully
    19/03/27 08:48:59 INFO mapreduce.Job: Counters: 38
            File System Counters
                    FILE: Number of bytes read=0
                    FILE: Number of bytes written=2462230
                    FILE: Number of read operations=0
                    FILE: Number of large read operations=0
                    FILE: Number of write operations=0
                    HDFS: Number of bytes read=1000001748624
                    HDFS: Number of bytes written=0
                    HDFS: Number of read operations=40124
                    HDFS: Number of large read operations=0
                    HDFS: Number of write operations=40
                    OSS: Number of bytes read=0
                    OSS: Number of bytes written=1000000000000
                    OSS: Number of read operations=0
                    OSS: Number of large read operations=0
                    OSS: Number of write operations=0
            Job Counters
                    Launched map tasks=20
                    Other local map tasks=20
                    Total time spent by all maps in occupied slots (ms)=65207738
                    Total time spent by all reduces in occupied slots (ms)=0
                    Total time spent by all map tasks (ms)=65207738
                    Total vcore-milliseconds taken by all map tasks=65207738
                    Total megabyte-milliseconds taken by all map tasks=66772723712
            Map-Reduce Framework
                    Map input records=10002
                    Map output records=0
                    Input split bytes=2740
                    Spilled Records=0
                    Failed Shuffles=0
                    Merged Map outputs=0
                    GC time elapsed (ms)=58169
                    CPU time spent (ms)=5960400
                    Physical memory (bytes) snapshot=4420333568
                    Virtual memory (bytes) snapshot=42971959296
                    Total committed heap usage (bytes)=2411724800
            File Input Format Counters
                    Bytes Read=1745884
            File Output Format Counters
                    Bytes Written=0
            org.apache.hadoop.tools.mapred.CopyMapper$Counter
                    BYTESCOPIED=1000000000000
                    BYTESEXPECTED=1000000000000
                    COPY=10002
    
    copy from dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/path/on/dfs to oss://accessKeyId:accessKeySecret@bucket-name.oss-cn-hangzhou.aliyuncs.com/path/on/oss does succeed !!!
    ```

    迁移完成后，您可以通过osscmd工具，执行以下命令查看对象存储OSS上的数据情况。

    ``` {#codeblock_e8v_z8x_lz7}
    osscmd ls oss://bucket-name/HDFS2OSS/data/data_1000g
    ```


## 将对象存储OSS数据迁移到文件存储HDFS {#section_18m_dfi_q08 .section}

1.  进入emr-tools工具安装包解压后所在的目录，使用`hdfs2oss4emr.sh`脚本将对象存储OSS上的数据迁移到文件存储HDFS上。具体命令如下所示：

    ``` {#codeblock_l5h_bhm_ugh}
    cd emr-tools
    ./hdfs2oss4emr.sh  \
    oss://accessKeyId:accessKeySecret@bucket-name.oss-cn-hangzhou.aliyuncs.com/OSS2HDFS/oss/1000g  \
    dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/OSS2HDFS/data/data_1000g
    ```

    参数说明如下表所示。

    |参数|说明|
    |--|--|
    |accessKeyId|访问对象存储OSS API的密钥。获取方式请参见[如何获取如何获取AccessKeyId和AccessKeySecret](https://help.aliyun.com/knowledge_detail/48699.html)。|
    |accessKeySecret|
    |bucket-name.oss-cn-hangzhou.aliyuncs.com|对象存储OSS的访问域名，包括bucket名称和endpoint地址。|

    执行以上命令后，系统将启动一个Hadoop MapReduce任务（DistCp）。

2.  任务执行完成后，查看迁移结果。

    如果回显包含如下类似信息，说明迁移成功。

    ``` {#codeblock_0hd_d3s_ile}
    19/03/23 21:59:23 INFO mapreduce.Job: Counters: 38
            File System Counters
                    DFS: Number of bytes read=2335687
                    DFS: Number of bytes written=999700000000
                    DFS: Number of read operations=60218
                    DFS: Number of large read operations=0
                    DFS: Number of write operations=20076
                    FILE: Number of bytes read=0
                    FILE: Number of bytes written=2575367
                    FILE: Number of read operations=0
                    FILE: Number of large read operations=0
                    FILE: Number of write operations=0
                    OSS: Number of bytes read=0
                    OSS: Number of bytes written=0
                    OSS: Number of read operations=0
                    OSS: Number of large read operations=0
                    OSS: Number of write operations=0
            Job Counters
                    Launched map tasks=21
                    Other local map tasks=21
                    Total time spent by all maps in occupied slots (ms)=36490484
                    Total time spent by all reduces in occupied slots (ms)=0
                    Total time spent by all map tasks (ms)=36490484
                    Total vcore-milliseconds taken by all map tasks=36490484
                    Total megabyte-milliseconds taken by all map tasks=37366255616
            Map-Reduce Framework
                    Map input records=10018
                    Map output records=0
                    Input split bytes=2856
                    Spilled Records=0
                    Failed Shuffles=0
                    Merged Map outputs=0
                    GC time elapsed (ms)=1064802
                    CPU time spent (ms)=10370840
                    Physical memory (bytes) snapshot=6452363264
                    Virtual memory (bytes) snapshot=45328142336
                    Total committed heap usage (bytes)=4169138176
            File Input Format Counters
                    Bytes Read=2332831
            File Output Format Counters
                   Bytes Written=0
            org.apache.hadoop.tools.mapred.CopyMapper$Counter
                    BYTESCOPIED=999700000000
                    BYTESEXPECTED=999700000000
                    COPY=10018
    copy from oss://accessKeyId:accessKeySecret@bucket-name.oss-cn-hangzhou.aliyuncs.com/path/on/oss to dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/path/on/dfs does succeed !!!
    ```

    迁移完成后，您可以执行以下命令查看文件存储HDFS上的数据情况。

    ``` {#codeblock_swq_ias_kkx}
    hadoop fs -ls   dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/OSS2HDFS/data/data_1000g
    ```


## 常见问题 {#section_94q_0na_3nx .section}

-   迁移过程出现异常提示：Cannot obtain block length for LocatedBlock。

    从原生HDFS往对象存储OSS/文件存储HDFS迁移数据时，可能会遇到这个问题。遇到该问题时，请执行`hdfs fsck / –openforwrite`命令，检查当前是否有文件处于写入状态尚未关闭。

    如果有处于写入状态的文件时，需判断文件是否有效。

    -   如果文件无效，则直接删除文件。

        ``` {#codeblock_4f4_035_wvh}
        hdfs rm <path-of-the-file> 
        ```

    -   如果文件有效，则不能直接删除，请考虑恢复问题文件租约。

        ``` {#codeblock_74r_xk4_frt}
        hdfs debug recoverLease -path <path-of-the-file> -retries <retry times>
        ```

-   对于正在写入的文件，进行迁移时会遗漏掉最新写入的数据吗？

    Hadoop兼容文件系统提供单写者多读者并发语义，针对同一个文件，同一时刻可以有一个写者写入和多个读者读出。以文件存储HDFS到对象存储OSS的数据迁移为例，数据迁移任务打开文件存储HDFS的文件F，根据当前系统状态决定文件F的长度L，将L字节迁移到对象存储OSS。如果在数据迁移过程中，有并发的写者写入，文件F的长度将超过L，但是数据迁移任务无法感知到最新写入的数据。因此，建议您在做数据迁移时，避免往迁移的文件中写入数据。


