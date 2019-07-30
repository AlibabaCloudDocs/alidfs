# CDH6数据迁移 {#task_1375534 .task}

本文档介绍如何将CDH中本地HDFS的数据迁移到文件存储HDFS。

在阿里云上创建ECS集群并安装CDH，具体安装方法请参考CDH相关文档。

CDH（Cloudera's Distribution, including Apache Hadoop）是众多 Hadoop 发行版本中的一种，您可以使用文件存储HDFS替换CDH6原有的本地HDFS服务，通过CDH6+文件存储HDFS实现大数据计算在云上的存储与计算分离，应对灵活多变的业务需求的挑战。

1.  登录CDH6的Cloudera Manager管理系统。
2.  配置链接。 
    1.  在系统主页，选择**配置** \> **高级配置代码段**，进入高级配置代码段页面。
    2.  搜索`core-site.xml`，并选择**HDFS**。
    3.  在core-site.xml 的群集范围高级配置代码段（安全阀）区域中，添加如下文件存储HDFS配置项。 

        -   （必须配置）配置项fs.dfs.impl，其值：com.alibaba.dfs.DistributedFileSystem
        -   （必须配置）配置项fs.AbstractFileSystem.dfs.impl，其值：com.alibaba.dfs.DFS
        -   （可选）配置项io.file.buffer.size，其值：4194304
        -   （可选）配置项dfs.connection.count，其值：1
        ![配置项](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095682/156447601053790_zh-CN.png)

    4.  单击**保存更改**。
    5.  返回系统主页，找到**HDFS**，单击重新部署图标，进行重新部署。 

        ![重新部署](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095682/156447601153793_zh-CN.png)

    6.  在过期配置页面，单击**重启过时服务**。
    7.  在重启过时服务页面，单击**立即重启**。
    8.  等待服务全部重启完成，并重新部署客户端配置后，单击**完成**。
3.  执行以下命令将文件系统HDFS的SDK包（aliyun-sdk-dfs-1.0.2-beta.jar \)，放置到CDH HDFS服务上存储jar包的路径下。其中jar包放置目录，请根据实际值替换。 

    ``` {#codeblock_c89_ljz_o8z}
    cp aliyun-sdk-dfs-1.0.2-beta.jar    /opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/lib/hadoop-hdfs/
    ```

    一般情况下，放置到/opt/cloudera/parcels/CDH-x.x.x-x.cdhx.x.x.px.xxxxx/lib/hadoop-hdfs/目录。例如：在CDH 6.0.1中，jar包放置目录为/opt/cloudera/parcels/CDH-6.0.1-1.cdh6.0.1.p0.590678/lib/hadoop-hdfs/。

    **说明：** 集群中的每台机器都需要在相同位置添加该SDK包。

4.  暂停服务。 

    为了保证在更换文件存储系统的过程中文件数据不丢失，需要暂停数据处理服务（如：YARN服务、Hive服务、Spark服务、HBase服务等），HDFS服务仍需保持运行。此处以HBase服务为例进行说明。

    1.  找到**HBase**，在其右侧的操作项中，单击**停止**。
    2.  在停止确认框中，单击**停止**。 当**HBase**前的图标变成灰色，表示该服务完全停止。
    3.  重复上述步骤，停止剩余服务。 

        **说明：** 建议只保留HDFS服务正常运行，以方便进行数据迁移。但是如果要迁移的数据量大，请开启YARN服务，以便使用数据迁移工具hadoop distcp进行快速地数据迁移。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095682/156447601153849_zh-CN.png)

5.  迁移数据。 

    建议将/user、/hbase等服务目录和相关的数据目录全量迁移至文件存储HDFS。

    -   如果涉及将云下集群的数据迁移到云上，请参见[迁移开源HDFS的数据到文件存储HDFS](cn.zh-CN/最佳实践/迁移开源HDFS的数据到文件存储HDFS.md#)。
    -   如果CDH HDFS文件系统上的数据量较小，可以使用`hadoop fs -cp`命令进行数据迁移。

        为了避免因为权限问题导致数据迁移失败，建议切换到hdfs用户执行命令。

        ``` {#codeblock_l62_zi3_jso}
        su hdfs
        hadoop fs -cp /user  dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/
        ```

        其中f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com为您的文件存储HDFS挂载点域名，请根据实际情况进行修改。

    -   如果CDH HDFS文件系统上的数据量较大，需要使用数据迁移工具hadoop distcp进行数据迁移，详情请参见[迁移开源HDFS的数据到文件存储HDFS](cn.zh-CN/最佳实践/迁移开源HDFS的数据到文件存储HDFS.md#)。

        ``` {#codeblock_2ut_qm6_9ea}
        hadoop distcp hdfs://oldclusterip:8020/user  dfs://f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com:10290/
        ```

        其中f-xxxxxxxxxxxxxxx.cn-xxxxxxx.dfs.aliyuncs.com为您的文件存储HDFS挂载点域名，需要根据实际情况进行修改。


[配置CDH6使用文件存储HDFS](cn.zh-CN/最佳实践/在文件存储HDFS上使用CDH6/配置CDH6使用文件存储HDFS.md#)

