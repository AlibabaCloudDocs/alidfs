# 在文件存储HDFS上使用Apache Spark {#task_2277929 .task}

本文档主要介绍在文件存储HDFS上搭建及使用Apache Spark的方法。

## 准备工作 {#section_5u5_18f_l6u .section}

1.  开通文件存储HDFS服务并创建文件系统实例和挂载点，详情请参见[HDFS快速入门](../../../../cn.zh-CN/快速入门/开通文件存储HDFS服务.md#)。
2.  在计算节点上安装JDK。 版本不能低于1.8。
3.  在计算节点上安装Scala。 Scala下载地址：[官方链接](https://www.scala-lang.org/download/all.html)，其版本要与使用的Apache Spark版本相兼容。
4.  下载Apache Hadoop压缩包。 Apache Hadoop下载地址：[官方链接](https://archive.apache.org/dist/hadoop/common/)。建议您选用的Apache Hadoop版本不低于2.7.2，本文档中使用的Apache Hadoop版本为Apache Hadoop 2.7.2。
5.  下载Apache Spark压缩包。 Apache Spark下载地址：[官方链接](https://archive.apache.org/dist/spark/)。选用Apache Spark版本时请注意该版本要与您当前选用的Apache Hadoop版本相兼容，本文中使用的Apache Spark版本为2.3.0。

**说明：** 本文档的操作步骤中涉及的安装包版本号、文件夹路径，请根据实际情况进行替换。

## 配置Apache Hadoop {#section_e7p_l7k_in4 .section}

1.  执行如下命令解压Apache Hadoop压缩包到指定文件夹。 

    ``` {#codeblock_x89_o6p_8w7}
    tar -zxvf hadoop-2.7.2.tar.gz -C /usr/local/
    ```

2.  修改hadoop-env.sh配置文件。 
    1.  执行如下命令打开hadoop-env.sh配置文件。 

        ``` {#codeblock_uqo_3o4_4hb}
        vim /usr/local/hadoop-2.7.2/etc/hadoop/hadoop-env.sh
        ```

    2.  配置JAVA\_HOME目录，如下所示。 

        ``` {#codeblock_e04_amc_zcf}
        export JAVA_HOME=/usr/java/default
        ```

3.  修改core-site.xml配置文件。 
    1.  执行如下命令打开core-site.xml配置文件。 

        ``` {#codeblock_wgg_l2i_4mc}
        vim /usr/local/hadoop-2.7.2/etc/hadoop/core-site.xml
        ```

    2.  在core-site.xml配置文件中，配置如下信息，详情请参见[挂载文件系统](../../../../cn.zh-CN/快速入门/挂载文件系统.md#)。 

        ``` {#codeblock_69k_tsl_vo0}
        <configuration>
        <property>
             <name>fs.defaultFS</name>
             <value>dfs://x-xxxxxxxx.cn-xxxxx.dfs.aliyuncs.com:10290</value>
             <!-- 该地址填写您的挂载点地址 -->
        </property>
        <property>
             <name>fs.dfs.impl</name>
             <value>com.alibaba.dfs.DistributedFileSystem</value>
        </property>
        <property>
             <name>fs.AbstractFileSystem.dfs.impl</name>
             <value>com.alibaba.dfs.DFS</value>
        </property>
        <property>
             <name>io.file.buffer.size</name>
             <value>8388608</value>
        </property>
        <property>
             <name>alidfs.use.buffer.size.setting</name>
             <value>true</value>
        </property>
        <property>
             <name>dfs.usergroupservice.impl</name>
             <value>com.alibaba.dfs.security.LinuxUserGroupService.class</value>
        </property>
          <property>
             <name>dfs.connection.count</name>
             <value>16</value>
        </property>
        </configuration>
        ```

4.  修改mapred-site.xml配置文件。 
    1.  执行如下命令打开mapred-site.xml配置文件。 

        ``` {#codeblock_igu_g4c_m1f}
        vim /usr/local/hadoop-2.7.2/etc/hadoop/mapred-site.xml
        ```

    2.  在mapred-site.xml配置文件中，配置如下信息。 

        ``` {#codeblock_81r_oc5_4px}
        <configuration>
        <property>
              <name>mapreduce.framework.name</name>
              <value>yarn</value>
        </property>
        </configuration>
        ```

5.  修改yarn-site.xml配置文件。 
    1.  执行如下命令打开yarn-site.xml配置文件。 

        ``` {#codeblock_m3m_0nu_82c}
        vim /usr/local/hadoop-2.7.2/etc/hadoop/yarn-site.xml
        ```

    2.  在yarn-site.xml配置文件中，配置如下信息。 

        ``` {#codeblock_lx7_6mz_z19}
        <configuration>
        <property>
          <name>yarn.resourcemanager.hostname</name>
          <value>xxxx</value>
          <!-- 该地址填写集群中yarn的resourcemanager的hostname -->
        </property>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>spark_shuffle,mapreduce_shuffle</value>
            <!-- 如果不配置spark on yarn,此处只设置为mapreduce_shuffle -->
        </property>
        <property>
            <name>yarn.nodemanager.aux-services.spark_shuffle.class</name>
            <value>org.apache.spark.network.yarn.YarnShuffleService</value>
            <!-- 如果不配置spark on yarn,不配置该项 -->
        </property>
        <property>
          <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
          <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
        <property>
          <name>yarn.nodemanager.vmem-pmem-ratio</name>
          <value>2.1</value>
        </property>
        <property>
          <name>yarn.nodemanager.pmem-check-enabled</name>
          <value>false</value>
        </property>
        <property>
          <name>yarn.nodemanager.vmem-check-enabled</name>
          <value>false</value>
        </property>
        <property>
          <name>yarn.nodemanager.resource.memory-mb</name>
          <value>16384</value>
            <!-- 根据您当前的集群能力进行配置此项 -->
        </property>
        <property>
          <name>yarn.nodemanager.resource.cpu-vcores</name>
          <value>4</value>
             <!-- 根据您当前的集群能力进行配置此项 -->
        </property>
        <property>
          <name>yarn.scheduler.maximum-allocation-vcores</name>
          <value>4</value>
            <!-- 根据您当前的集群能力进行配置此项 -->
        </property>
        <property>
          <name>yarn.scheduler.minimum-allocation-mb</name>
          <value>3584</value>
            <!-- 根据您当前的集群能力进行配置此项 -->
        </property>
        <property>
          <name>yarn.scheduler.maximum-allocation-mb</name>
          <value>14336</value>
            <!-- 根据您当前的集群能力进行配置此项 -->
        </property>
        </configuration>
        ```

6.  修改slaves配置文件。 
    1.  执行如下命令打开slaves配置文件。 

        ``` {#codeblock_on9_gal_9yr}
        vim /usr/local/hadoop-2.7.2/etc/hadoop/slaves
        ```

    2.  在slaves配置文件中，配置如下信息。 

        ``` {#codeblock_ws3_jxb_4w0}
        node1
        node2
        ```

7.  配置环境变量。 
    1.  执行如下命令打开/etc/profile配置文件。 

        ``` {#codeblock_vfa_2ub_q3i}
        vim /etc/profile
        ```

    2.  在/etc/profile配置文件中，配置如下信息。 

        ``` {#codeblock_tcp_ul3_w24}
        export HADOOP_HOME=/usr/local/hadoop-2.7.2
        export HADOOP_CLASSPATH=/usr/local/hadoop-2.7.2/etc/hadoop:/usr/local/hadoop-2.7.2/share/hadoop/common/lib/*:/usr/local/hadoop-2.7.2/share/hadoop/common/*:/usr/local/hadoop-2.7.2/share/hadoop/hdfs:/usr/local/hadoop-2.7.2/share/hadoop/hdfs/lib/*:/usr/local/hadoop-2.7.2/share/hadoop/hdfs/*:/usr/local/hadoop-2.7.2/share/hadoop/yarn/lib/*:/usr/local/hadoop-2.7.2/share/hadoop/yarn/*:/usr/local/hadoop-2.7.2/share/hadoop/mapreduce/lib/*:/usr/local/hadoop-2.7.2/share/hadoop/mapreduce/*:/usr/local/hadoop-2.7.2/contrib/capacity-scheduler/*.jar
        export HADOOP_CONF_DIR=/usr/local/hadoop-2.7.2/etc/hadoop
        ```

    3.  执行如下命令使配置生效。 

        ``` {#codeblock_7vv_mm0_oci}
        source /etc/profile
        ```

8.  执行如下命令配置文件存储HDFS的SDK。 

    您可以单击[此处](https://mvnrepository.com/artifact/com.aliyun.dfs/aliyun-sdk-dfs)，下载文件存储HDFS的SDK （此处以aliyun-sdk-dfs-1.0.3.jar为例），将其部署在Hadoop生态系统组件的CLASSPATH上，详情请参见[挂载文件系统](../../../../cn.zh-CN/快速入门/挂载文件系统.md#)。

    ``` {#codeblock_61z_x19_ywf}
    cp aliyun-sdk-dfs-1.0.3.jar  /usr/local/hadoop-2.7.2/share/hadoop/hdfs
    ```

9.  执行如下命令将$\{HADOOP\_HOME\}文件夹同步到集群的其他节点。 

    ``` {#codeblock_9bk_fnp_q7j}
    scp -r hadoop-2.7.2/ root@node2:/usr/local/
    ```


## 验证Apache Hadoop配置 {#section_bge_8p6_r02 .section}

完成Hadoop配置后，不需要格式化namenode，也不需要使用start-dfs.sh来启动HDFS相关服务。如需使用yarn服务，只需在resourcemanager节点启动yarn服务，具体验证Hadoop配置成功的方法请参见[安装](../../../../cn.zh-CN/SDK 参考/安装.md#)。

``` {#codeblock_nf2_com_a4c}
/usr/local/hadoop-2.7.2/sbin/start-yarn.sh
```

## 配置Apache Spark {#section_vyz_9r4_5s6 .section}

本文档以spark on yarn为例进行搭建说明，spark on yarn的官方配置文档请参见[在Yarn上使用Spark](http://spark.apache.org/docs/2.3.0/running-on-yarn.html)。

1.  执行如下命令解压Apache Spark压缩包。 

    ``` {#codeblock_g6u_0p1_wgf}
    tar -zxvf spark-2.3.0-bin-hadoop2.7.tgz -C /usr/local/
    ```

2.  修改spark-env.sh配置文件。 
    1.  执行如下命令打开spark-env.sh配置文件。 

        ``` {#codeblock_a1p_37n_pvt}
        vim /usr/local/spark-2.3.0-bin-hadoop2.7/conf/spark-env.sh
        ```

    2.  在spark-env.sh配置文件中，配置如下信息。 

        ``` {#codeblock_g83_hj5_adp}
        export JAVA_HOME=/usr/java/default
        export SCALA_HOME=/usr/local/scala-2.11.7
        export SPARK_CONF_DIR=/usr/local/spark-2.3.0-bin-hadoop2.7/conf
        export HADOOP_HOME=/usr/local/hadoop-2.7.2
        export HADOOP_CONF_DIR=/usr/local/hadoop-2.7.2/etc/hadoop
        ```

3.  拷贝jar包。 

    1.  将配置Apache Hadoop章节的[步骤 8](#step_uur_lz1_pl7)中下载的文件存储HDFS SDK（此处以aliyun-sdk-dfs-1.0.3.jar为例）拷贝到Spark配置文件夹的jars目录下。 

        ``` {#codeblock_x1c_ea1_6nb}
        cp aliyun-sdk-dfs-1.0.3.jar /usr/local/spark-2.3.0-bin-hadoop2.7/jars
        ```

    2.  将Spark配置文件夹中yarn目录下的spark-x.x.x-yarn-shuffle.jar包拷贝到当前集群所有节点的yarn/lib目录下。 

        ``` {#codeblock_qcu_545_hdb}
        cp /usr/local/spark-2.3.0-bin-hadoop2.7/yarn/spark-2.3.0-yarn-shuffle.jar /usr/local/hadoop-2.7.2/share/hadoop/yarn
        ```

    **说明：** 

    -   配置spark on yarn模式时，不需要将Spark配置文件夹分发到集群的所有节点，只需要在集群中一台提交任务的节点上配置即可。
    -   配置spark standalone时，需要将文件存储HDFS的SDK（aliyun-sdk-dfs-x.y.z.jar） 拷贝到Spark配置文件夹的jars目录下之后再将Spark的配置文件夹分发到集群所有节点上。

## 验证Apache Spark配置 {#section_2sc_bjp_4m1 .section}

使用Spark读取文件存储HDFS上面的文件进行wordcount计算，将计算结果打印并写入文件存储HDFS。

1.  执行以下命令创建测试数据。 

    ``` {#codeblock_dgm_1lv_p17}
    echo -e "hello,world\nhello,world\nhello,world\nhello,world\nhello,world" > /tmp/words
    ```

    创建完成后，你可以执行`cat /tmp/words`命令查看测试数据是否创建成功。

2.  执行如下命令，在文件存储HDFS上创建文件夹。 

    ``` {#codeblock_rma_yta_tq5}
    /usr/local/hadoop-2.7.2/bin/hadoop fs -mkdir -p /sparktest/input
    /usr/local/hadoop-2.7.2/bin/hadoop fs -mkdir -p /sparktest/output
    ```

3.  执行如下命令将测试数据上传至HDFS上的文件夹。 

    ``` {#codeblock_sf4_f4l_kj2}
    /usr/local/hadoop-2.7.2/bin/hadoop fs -put /tmp/words /sparktest/input
    ```

    上传完成后，您可以执行`/usr/local/hadoop-2.7.2/bin/hadoop fs -cat /sparktest/input/words`命令确认测试数据是否上传成功。

4.  执行以下命令启动spark-shell。 

    ``` {#codeblock_l5y_kpr_jbi}
    /usr/local/spark-2.3.0-bin-hadoop2.7/bin/spark-shell --master yarn 
    --deploy-mode client \
    --driver-cores 1  \
    --driver-memory 1G \
    --executor-memory 1G \
    --num-executors 2 \
    ```

5.  执行以下命令运行wordcount程序。 

    ``` {#codeblock_usx_faj_w3i}
    scala> val res = sc.textFile("dfs://x-xxxxxx.cn-xxxx.dfs.aliyuncs.com:10290/sparktest/input/words").flatMap(_.split(",")).map((_,1)).reduceByKey(_+_)
    scala> res.collect.foreach(println)
    scala> res.saveAsTextFile("dfs://x-xxxxxx.cn-xxxx.dfs.aliyuncs.com:10290/sparktest/output/res")
    ```

    其中，dfs://x-xxxxxx.cn-xxxx.dfs.aliyuncs.com为HDFS文件系统的挂载点，请根据实际情况替换。


