# E-MapReduce数据迁移 {#task_1495493 .task}

本文介绍如何将E-MapReduce HDFS上的数据迁移到文件存储HDFS。

阿里云E-MapReduce是构建在阿里云云服务器ECS上的开源Hadoop、Spark、Hive、Flink生态大数据PaaS产品。提供用户在云上使用开源技术建设数据仓库、离线批处理、在线流式处理、即时查询、机器学习等场景下的大数据解决方案。

## 准备工作 {#section_e6j_kr5_ahj .section}

1.  开通并创建E-MapRedece集群，详情请参见[创建集群](../../../../cn.zh-CN/快速入门/步骤三：创建集群.md#)。 

    **说明：** 当使用阿里云文件存储HDFS替换E-MapReduce HDFS服务时，您可以选择使用高效云盘、SSD云盘或者本地盘作为Shuffle数据的临时本地存储。具体存储规划可以参考：[../../SP\_159/DNemapreduce1876943/ZH-CN\_TP\_17849\_V10.dita\#concept\_ift\_2n3\_y2b](../../SP_159/DNemapreduce1876943/ZH-CN_TP_17849_V10.dita#concept_ift_2n3_y2b)。

2.  开通文件存储HDFS服务，并创建文件系统实例、添加挂载点，创建权限组和权限组规则，详情请参见[文件存储HDFS快速入门](../../../../cn.zh-CN/快速入门/开通文件存储HDFS服务.md#)。 

    **说明：** 配置挂载点时选择的专有网络和交换机要与E-MapReduce集群侧的配置保持一致。您可以通过以下方法获取：

    1.  登录[阿里云E-MapReduce控制台](https://emr.console.aliyun.com/?spm=a2c4g.11186623.2.8.56cb5168Vl7YY6)。
    2.  在集群管理页面，找到需要挂载文件存储HDFS的目标E-MapReduce集群，单击**管理**。
    3.  单击**集群基础信息**，在**网络信息**区域中获取专有网络和交换机信息。

## 数据迁移 {#section_1ra_agm_4h2 .section}

1.  登录[阿里云E-MapReduce控制台](https://emr.console.aliyun.com/?spm=a2c4g.11186623.2.8.56cb5168Vl7YY6)。
2.  在集群管理页面，找到需要挂载文件存储HDFS的目标E-MapReduce集群，单击**管理**。
3.  配置链接。 
    1.  选择**集群服务** \> **HDFS**，单击配置。
    2.  在服务配置中，选择**core-site**，并单击**自定义配置**。
    3.  新增如下配置项，单击**确定**。 

        -   （必须配置）配置项fs.dfs.impl，其值：com.alibaba.dfs.DistributedFileSystem
        -   （必须配置）配置项fs.AbstractFileSystem.dfs.impl，其值：com.alibaba.dfs.DFS
        -   （可选）配置项io.file.buffer.size，其值：4194304
        -   （可选）配置项dfs.connection.count，其值：1
        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1188169/156499241054183_zh-CN.png)

    4.  确认自定义配置成功后，单击**保存**，在确认保存对话框中，输入执行原因，单击**确定**。 

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1188169/156499241054186_zh-CN.png)

    5.  单击**部署客户端配置**，在确认保存对话框中，输入执行原因，单击**确定**。
4.  执行以下命令，将文件存储HDFS的SDK包（aliyun-sdk-dfs-1.0.2-beta.jar \)，放置到E-MapReduce HDFS服务存放jar包的路径下。 

    ``` {#codeblock_shp_gd0_kx2}
    cp ~/aliyun-sdk-dfs-1.0.2-beta.jar    /opt/apps/ecm/service/hadoop/2.8.5-1.3.1/package/hadoop-2.8.5-1.3.1/share/hadoop/hdfs/
    ```

    在E-MapReduce服务中，对应的路径为/opt/apps/ecm/service/hadoop/x.x.x-x.x.x/package/hadoop-x.x.x-x.x.x/share/hadoop/hdfs。

    **说明：** 集群中的每台机器都需要在相同位置添加该SDK包。

5.  暂停服务。 

    为了保证在数据迁移过程中数据不丢失，需要暂停数据处理服务（如：YARN服务、Hive服务、Spark服务、HBase服务等），HDFS服务仍需保持运行。此处以暂停Spark服务为例进行说明。

    1.  选择**集群服务** \> **Spark**。
    2.  在页面右侧的**操作**栏中，单击**停止 All Components**。
    3.  在执行集群操作对话框中，填写执行原因，并单击**确定**。 

        当组件前面的图标变成红色，表示该组件服务已停止。直到Spark服务运行的组件全部停止后，再进行其他服务的组件停止操作。无需关注xxx Client组件的状态。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1188169/156499241054202_zh-CN.png)

    4.  重复上述步骤，停止剩余服务。 

        **说明：** 只保留HDFS服务正常运行，以方便进行数据迁移。但是如果要迁移的数据量大，请开启YARN服务，以便使用hadoop的数据迁移工具hadoop distcp进行快速地数据迁移。

6.  迁移数据。 

    建议将/user、/hbase、/spark-history 、/apps等服务目录和相关的数据目录全量迁移至文件存储HDFS。

    -   如果涉及将云下集群的数据迁移到云上，请参见[迁移开源HDFS的数据到文件存储HDFS](cn.zh-CN/最佳实践/迁移开源HDFS的数据到文件存储HDFS.md#)。
    -   如果E-MapReduce HDFS文件系统上的数据量较小，可以使用`hadoop fs -cp`命令进行数据迁移。

        为了避免因为权限问题导致数据迁移失败，建议使用root用户执行命令。

        ``` {#codeblock_xzc_lh4_0zx}
        hadoop fs -cp /user  dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/
        ```

        其中f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com为您的文件存储HDFS挂载点域名，请根据实际情况进行修改。

    -   如果E-MapReduce HDFS文件系统上的数据量较大，需要使用数据迁移工具hadoop distcp进行数据迁移，详情请参见[迁移开源HDFS的数据到文件存储HDFS](cn.zh-CN/最佳实践/迁移开源HDFS的数据到文件存储HDFS.md#)。

        ``` {#codeblock_07m_kof_u1p}
        hadoop distcp hdfs://emr-header-1.cluster-xxxx:9000/  dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/
        ```

        其中f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com为您的文件存储HDFS挂载点域名，需要根据实际情况进行修改。


[配置E-MapReduce服务使用文件存储HDFS](cn.zh-CN/最佳实践/在文件存储HDFS上使用E-MapReduce/配置E-MapReduce服务使用文件存储HDFS.md#)

