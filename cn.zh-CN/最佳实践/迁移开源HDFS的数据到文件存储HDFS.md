# 迁移开源HDFS的数据到文件存储HDFS {#task_1305562 .task}

本文档介绍如何将开源HDFS的数据平滑地迁移到文件存储HDFS。

当前业界有很多公司是以Hadoop技术构建数据中心，而越来越多的公司和企业希望将业务顺畅地迁移到云上。文件存储HDFS可以帮助您实现将开源HDFS的数据迁移到云上，并允许您在云上就像在Hadoop分布式文件系统中管理和访问数据。

## 适用范围 {#section_zrb_9lj_du9 .section}

-   非阿里云Hadoop集群中的数据迁移到文件存储HDFS。
-   阿里云ECS自建Hadoop集群中的数据迁移到文件存储HDFS。

## 准备工作 {#section_a0f_rz3_msl .section}

1.  在阿里云ECS创建Hadoop集群。 

    如果您目前的Hadoop集群是搭建在阿里云VPC网络上的阿里云ECS集群，则无需在阿里云ECS上创建新的Hadoop集群。

2.  创建和挂载文件系统至阿里云ECS上的Hadoop集群，并将文件存储HDFS设置为fs.defaultFS，详情请参见[文件存储HDFS快速入门](文件存储HDFS快速入门../DNalidfs1853915/ZH-CN_TP_19062_V3.dita#concept_idt_vvl_z2b)。
3.  验证文件系统和计算节点之间的连通性。 
    1.  执行以下命令，在文件存储HDFS上创建目录（如：/dfs\_links）。 

        ``` {#codeblock_wk9_8b9_ffz}
        hadoop fs -mkdir /dfs_links
        ```

    2.  执行以下命令，验证连通性。 

        ``` {#codeblock_aec_k20_g71}
        hadoop fs -ls dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/dfs_links
        ```

        其中f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com为文件存储HDFS挂载点域名，请根据您的实际情况进行修改。

        如果命令正常执行无输出结果，则表示连通成功。如果连通失败，请参见[创建文件系统实例后，为什么无法访问文件存储HDFS？](../../../../cn.zh-CN/常见问题/一般性问题/创建文件系统实例后，为什么无法访问文件存储HDFS？.md#)进行排查。

4.  准备迁移工具。 

    您可以通过Hadoop社区标准的Distcp工具实现全量或增量的HDFS数据迁移，详细的Distcp工具使用说明请参见[Hadoop Distcp 工具官方说明文档](https://hadoop.apache.org/docs/current/hadoop-distcp/DistCp.html)。

    **说明：** 使用Distcp命令将旧集群数据迁移至文件存储HDFS时，请注意文件存储HDFS不支持以下参数，其它参数使用和[Hadoop Distcp 工具官方说明文档](https://hadoop.apache.org/docs/current/hadoop-distcp/DistCp.html)一致。文件存储HDFS及命令行存在限制的详细信息请参见[../DNalidfs1814297/ZH-CN\_TP\_18933\_V5.dita\#concept\_gjn\_zqx\_y2b](../DNalidfs1814297/ZH-CN_TP_18933_V5.dita#concept_gjn_zqx_y2b)。

    |参数|描述|状态|
    |--|--|--|
    |-p\[rbpax\]|r：replication，b：block-size，p：permission，a：ACL，x：XATTR|不可用|


## 非阿里云自建Hadoop集群数据迁移 {#section_okz_yn0_m64 .section}

非阿里云自建Hadoop集群数据迁移到文件存储HDFS包括以下两种情况。

-   非阿里云自建Hadoop集群与文件存储HDFS可以实现网络互通时， 请按照以下方法进行数据迁移。
    1.  使用阿里云高速通道产品建立原集群和文件存储HDFS所在VPC网络的连通，详情请参见[连接本地IDC](https://help.aliyun.com/document_detail/99972.html?spm=a2c4g.11186623.6.578.3bf960caIt4MDV)。
    2.  新旧集群实现网络互通后，执行以下命令迁移数据。

        ``` {#codeblock_85m_8ar_5x7}
        hadoop distcp  -m 1000 -bandwidth 30 hdfs://oldclusterip:8020/user/hive/warehouse  dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse
        ```

        其中oldclusterip为原自建Hadoop集群namenode的IP地址或者域名，f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com为文件存储HDFS挂载点域名，请根据您的实际情况进行修改。

        **说明：** 为减轻现有集群资源压力，建议确保新旧集群网络连通后，在新挂载文件系统的阿里云Hadoop集群上执行数据迁移命令。

-   非阿里云自建Hadoop集群与文件存储HDFS无法实现网络互通时，请按照以下方法进行数据迁移。
    1.  将非阿里云自建Hadoop集群数据迁移到对象存储OSS，详情请参见[离线迁移教程](https://help.aliyun.com/document_detail/54753.html)。
    2.  将对象存储OSS数据迁移到文件存储HDFS，详情请参见[ZH-CN\_TP\_817105\_V2.dita\#task\_1012751](ZH-CN_TP_817105_V2.dita#task_1012751)。

## 阿里云ECS自建Hadoop集群数据迁移 {#section_430_hkl_f6q .section}

阿里云ECS自建Hadoop集群数据迁移到文件存储HDFS时，包括以下两种情况：

-   阿里云ECS自建Hadoop集群处于经典网络环境时，请按照以下方法进行数据迁移。
    1.  通过阿里云ECS的ClassicLink建立ClassicLink连接，详情请参见[建立 ClassicLink 连接](https://help.aliyun.com/document_detail/65413.html#task-xpf-24c-sdb)。
    2.  执行以下命令迁移数据。

        ``` {#codeblock_2x2_tyy_1nb}
        hadoop distcp  -m 1000 -bandwidth 30 hdfs://oldclusterip:8020/user/hive/warehouse  dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse
        ```

        其中oldclusterip为原自建Hadoop集群namenode的IP地址或者域名，f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com为文件存储HDFS挂载点域名，请根据您的实际情况进行修改。

-   阿里云ECS自建Hadoop集群处于VPC网络环境时，请按照以下方法进行数据迁移。

    阿里云ECS自建Hadoop集群处于VPC网络环境时，可以直接通过VPC网络迁移数据到文件存储HDFS。迁移命令如下所示：

    ``` {#codeblock_cgl_27w_nnr}
    hadoop distcp  -m 1000 -bandwidth 30 hdfs://oldclusterip:8020/user/hive/warehouse  dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/user/hive/warehouse
    ```

    其中oldclusterip为原自建Hadoop集群namenode的IP或者域名，f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com为文件存储HDFS挂载点域名，请根据您的实际情况进行修改。


## 常见问题 {#section_dhi_0ll_5qy .section}

-   整体迁移速度受Hadoop集群与文件存储HDFS之间的带宽、集群规模影响。同时文件越多，checksum需要的时间越长。如果迁移数据量大，建议先尝试迁移几个目录评估下整体时间。如果只能在指定时间段内迁移数据，可以将目录切为几个小目录，依次迁移。
-   一般全量数据同步时，需要一个短暂的业务停写过程，用来启用双写双算或直接将业务切换到新集群上。
-   迁移过程出现异常提示：Cannot obtain block length for LocatedBlock。

    从原生的HDFS往对象存储OSS/文件存储HDFS迁移数据时，可能会遇到这个问题。遇到该问题时，请执行`hdfs fsck / –openforwrite`命令，检查当前是否有文件处于写入状态尚未关闭。

    如果有处于写入状态的文件时，需判断文件是否有效。

    -   如果文件无效，则直接删除文件。

        ``` {#codeblock_395_m6l_r3z}
        hdfs rm <path-of-the-file> 
        ```

    -   如果文件有效，则不能直接删除，请考虑恢复问题文件租约。

        ``` {#codeblock_i91_3so_zil}
        hdfs debug recoverLease -path <path-of-the-file> -retries <retry times>
        ```


