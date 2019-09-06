# 文件存储HDFS和数据库MySQL双向数据迁移 {#task_2054655 .task}

本文档介绍如何使用Sqoop工具实现文件存储HDFS和关系型数据库MySQL之间的双向数据迁移。

Sqoop是一款开源的工具，主要用于在Hadoop和结构化数据存储（如关系数据库）之间高效传输批量数据 。既可以将一个关系型数据库（MySQL 、Oracle 、Postgres等）中的数据导入HDFS中，也可以将HDFS的数据导入到关系型数据库中。

## 准备工作 {#section_blv_um3_qos .section}

现在Sqoop分为Sqoop1和Sqoop2，两个版本并不兼容。本案例选择使用sqoop1的稳定版本[Sqoop 1.4.7 版本](http://www.apache.org/dyn/closer.lua/sqoop/1.4.7)。

1.  下载[Sqoop 1.4.7 版本](http://www.apache.org/dyn/closer.lua/sqoop/1.4.7)。
2.  解压安装包。 

    ``` {#codeblock_0yx_kg0_pmz}
    tar -zxf sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz -C /home/hadoop/
    ```

3.  配置环境变量。 
    1.  执行`vim /etc/profile`命令，打开配置文件，添加如下内容。 

        ``` {#codeblock_6ri_4l6_ted}
        export SQOOP_HOME=/home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0
        export PATH=$PATH:$SQOOP_HOME/bin
        ```

    2.  执行`source /etc/profile`命令，使配置生效。
4.  添加数据库驱动。 
    1.  下载MySQL链接包。 

        ``` {#codeblock_pqn_hst_a9v}
        wget http://central.maven.org/maven2/mysql/mysql-connector-java/5.1.38/mysql-connector-java-5.1.38.jar
        ```

    2.  将MySQL链接包存放到Sqoop安装目录的lib目录下。 

        ``` {#codeblock_luy_p1f_c9e}
        cp mysql-connector-java-5.1.38.jar /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/lib/
        ```

5.  修改配置文件。 
    1.  执行如下命令进入/home/hadoop/sqoop-1.4.7.bin\_\_hadoop-2.6.0/conf目录。 

        ``` {#codeblock_t8z_bz2_0ri}
        cd /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/conf
        ```

    2.  执行如下命令复制sqoop-env-template.sh，并命名为sqoop-env.sh。 

        ``` {#codeblock_ry1_prn_s8t}
        cp sqoop-env-template.sh  sqoop-env.sh
        ```

    3.  执行`vim sqoop-env.sh`命令打开配置文件，添加如下内容。 

        ``` {#codeblock_8ad_a51_62w}
        export HADOOP_COMMON_HOME=/home/hadoop/hadoop-2.7.2
        export HADOOP_MAPRED_HOME=/home/hadoop/hadoop-2.7.2
        export HIVE_HOME=/home/hadoop/hive-2.1.0   #若没有安装hive、hbase可不必添加此配置
        export HBASE_HOME=/home/hadoop/hbase-1.2.2   #若没有安装hive、hbase可不必添加此配置
        ```

6.  执行如下命令验证数据库是否连接成功。 

    ``` {#codeblock_f5h_z4g_zli}
    sqoop list-databases --connect jdbc:mysql://<dburi> --username 'username' --password 'password'
    ```

    |参数|说明|
    |--|--|
    |dburi|数据库的访问连接，例如： jdbc:mysql://0.0.0.0:3306/。|
    |username|数据库登录用户名。|
    |password|用户密码。|

    如果回显信息中显示MySQL数据库的名称，则表示连接成功。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1630413/156775574759362_zh-CN.png)


## 将MySQL的数据迁移到HDFS上 {#section_1s9_tj3_2ey .section}

在集群Sqoop节点上，使用`sqoop import`命令将MySQL中的数据迁移到HDFS上。

此处以迁移MySQL中的employee表为例，employee表中已写入如下数据。

``` {#codeblock_kx1_539_t9k}
01,测试用户1,1990-01-01,男
02,测试用户2,1990-12-21,男
03,测试用户3,1990-05-20,男
04,测试用户4,1990-08-06,男
05,测试用户5,1991-12-01,女
```

1.  执行以下命令迁移数据。 

    ``` {#codeblock_ykg_ppl_i6k}
    sqoop import --connect jdbc:mysql://172.x.x.x:3306/sqoop_migrate --username 'userid' --password 'userPW' --table employee  --target-dir /mysql2sqoop/table/sqoop_migrate  --num-mappers 1  --columns "e_id,e_name,e_birth,e_sex"  --direct
    ```

    命令格式：`sqoop import --connect jdbc:mysql://<dburi>/<dbname> --username <username> --password <password> --table <tablename> --check-column <col> --incremental <mode> --last-value <value> --target-dir <hdfs-dir>`

    参数说明如下所示，更多详情请参见[Sqoop Import](http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html?spm=a2c4g.11186623.2.11.3c8864a1l6r2uw#-syntax)。

    |参数|说明|
    |--|--|
    |dburi|数据库的访问连接。例如：jdbc:mysql://172.x.x.x:3306/ 。如果您的访问连接中含有参数，则请加上单引号，例如：'jdbc:mysql://172.x.x.x.235:3306/mydatabase?useUnicode=true&characterEncoding=UTF-8'。|
    |dbname|数据库的名字，例如：user。|
    |username|数据库登录用户名。|
    |password|用户密码。|
    |tablename|MySQL数据库中表的名称。|
    |col|迁移表中列的名称。|
    |mode|该模式决定Sqoop如何定义哪些行为新的行。取值：append或lastmodified。|
    |value|前一个导入中检查列的最大值。|
    |hdfs-dir|HDFS的写入目录，此处以/mysql2sqoop/table/sqoop\_migrate为例。|

2.  检查迁移结果。 
    1.  执行`hadoop fs -ls /mysql2sqoop/table/sqoop_migrate`命令，获取迁移文件，此处以part-m-00000为例。 

        ``` {#codeblock_h6r_8fe_5lv}
        Found 2 items
        -rwxrwxrwx   3 root root          0 2019-08-21 14:42 /mysql2sqoop/table/sqoop_migrate/_SUCCESS
        -rwxrwxrwx   3 root root        200 2019-08-21 14:42 /mysql2sqoop/table/sqoop_migrate/part-m-00000
        ```

    2.  执行`hadoop fs -cat /mysql2sqoop/table/sqoop_migrate/part-m-00000`命令查看文件中的内容。 

        如果part-m-00000文件中有如下内容，则表示迁移成功。

        ``` {#codeblock_b6j_unc_x4p}
        01,测试用户1,1990-01-01,男
        02,测试用户2,1990-12-21,男
        03,测试用户3,1990-05-20,男
        04,测试用户4,1990-08-06,男
        05,测试用户5,1991-12-01,女
        ```


## 将HDFS的数据迁移到MySQL上 {#section_8fx_w5f_0qj .section}

将HDFS的数据迁移到MySQL上，需要先在MySQL上创建好对应HDFS数据结构的表，然后在集群Sqoop节点上使用`sqoop export`命令进行迁移。

此处以迁移HDFS上mysqltest.txt中的数据为例，mysqltest.txt中已写入如下数据。

``` {#codeblock_3u2_b55_ams}
6,测试用户6,2019-08-10,男
7,测试用户7,2019-08-11,男
8,测试用户8,2019-08-12,男
9,测试用户9,2019-08-13,女
10,测试用户10,2019-08-14,女
```

1.  创建数据库。 

    ``` {#codeblock_m4n_83q_58u}
    create database sqoop_migrate;
    ```

2.  使用已创建的数据库。 

    ``` {#codeblock_lma_b41_e14}
    use sqoop_migrate;
    ```

3.  创建表。 

    ``` {#codeblock_nkw_sps_p4p}
    CREATE TABLE `employee` (
      `e_id` varchar(20) NOT NULL DEFAULT '',
      `e_name` varchar(20) NOT NULL DEFAULT '',
      `e_birth` varchar(20) NOT NULL DEFAULT '',
      `e_sex` varchar(10) NOT NULL DEFAULT '',
      PRIMARY KEY (`e_id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8 
    ```

4.  执行以下命令迁移数据。 

    ``` {#codeblock_nf9_m02_92g}
     sqoop export --connect jdbc:mysql://172.0.0.0:3306/sqoop_migrate  --username 'userid' --password 'userPW'  --num-mappers 1  --table employee  --columns "e_id,e_name,e_birth,e_sex"  --export-dir '/sqoop2mysql/table/mysqltest.txt'  --fields-terminated-by ','
    ```

    迁移命令格式：`sqoop export --connect jdbc:mysql://<dburi>/<dbname> --username <username> --password <password> --table <tablename> --export-dir <hdfs-dir>`

    |参数|说明|
    |--|--|
    |dburi|数据库的访问连接。例如：jdbc:mysql://172.x.x.x:3306/ 。如果您的访问连接中含有参数，则请加上单引号，例如：'jdbc:mysql://172.x.x.x.235:3306/mydatabase?useUnicode=true&characterEncoding=UTF-8'。|
    |dbname|数据库的名字，例如：user。|
    |username|数据库登录用户名。|
    |password|用户密码。|
    |tablename|MySQL数据库中表的名称。|
    |hdfs-dir|存放待迁移数据的HDFS目录，此处以/sqoop2mysql/table/mysqltest.txt为例。|

5.  验证迁移结果。 
    1.  执行以下命令进入数据库。 

        ``` {#codeblock_wey_cjr_6fj}
        mysql -uroot -p
        ```

    2.  执行以下命令使用数据库。 

        ``` {#codeblock_p0u_yg0_2tf}
        use  sqoop_migrate;
        ```

    3.  执行`select * from employee;`命令查看表数据。 

        如果表中有如下数据，则表示迁移成功。

        ``` {#codeblock_sj1_nw6_g24}
        ...
        
        | 6   | 测试用户6     | 2019-08-10 | 男    |
        | 7   | 测试用户7     | 2019-08-11 | 男    |
        | 8   | 测试用户8     | 2019-08-12 | 男    |
        | 9   | 测试用户9     | 2019-08-13 | 女    |
        | 10  | 测试用户10     | 2019-08-14 | 女    |
        +------+---------------+------------+-------+
        10 rows in set (0.00 sec)
        ```


## 将MySQL的数据迁移到Hive上 {#section_bcd_26n_lv3 .section}

在集群Sqoop节点上使用`sqoop import`命令可以将MySQL上的数据迁移到Hive上。

此处以迁移MySQL中的employee表为例，employee表中已写入如下数据。

``` {#codeblock_a3j_vpl_m5d}
1,测试用户1,2019-08-10,男
2,测试用户2,2019-08-11,男
3,测试用户3,2019-08-12,男
4,测试用户4,2019-08-13,女
5,测试用户5,2019-08-14,女
```

1.  执行以下命令迁移数据。 

    ``` {#codeblock_062_ve0_w1i}
    sqoop import --connect jdbc:mysql://172.0.0.0:3306/sqoop_migrate --username 'userid' --password 'PW'   --table employee   --hive-import --hive-database default  --create-hive-table --hive-overwrite  -m 1 ;
    ```

    迁移命令格式：`sqoop import --connect jdbc:mysql://<dburi>/<dbname> --username <username> --password <password> --table <tablename> --check-column <col> --incremental <mode> --last-value <value> --fields-terminated-by "\t" --lines-terminated-by "\n" --hive-import --target-dir <hdfs-dir> --hive-table <hive-tablename>`

    |参数|说明|
    |--|--|
    |dburi|数据库的访问连接。例如：jdbc:mysql://172.x.x.x:3306/ 。如果您的访问连接中含有参数，则请加上单引号，例如：'jdbc:mysql://172.x.x.x.235:3306/mydatabase?useUnicode=true&characterEncoding=UTF-8'。|
    |dbname|数据库的名字，例如：user。|
    |username|数据库登录用户名。|
    |password|用户密码。|
    |tablename|MySQL数据库中表的名称。|
    |col|迁移表中列的名称。|
    |mode|该模式决定Sqoop如何定义哪些行为新的行。取值：append或lastmodified。|
    |value|前一个导入中检查列的最大值。|
    |hdfs-dir|HDFS的写入目录。|
    |hive-tablename|对应的Hive中的表名。|

2.  验证迁移结果。 

    执行`select * from default.employee;`命令查看表数据，如果表中有如下数据，则表示迁移成功。

    ``` {#codeblock_mr1_i2n_a59}
    1      测试用户1       2019-08-10      男
    2      测试用户2       2019-08-11      男
    3      测试用户3       2019-08-12      男
    4      测试用户4       2019-08-13      女
    5      测试用户5       2019-08-14      女
    ...
    Time taken: 0.105 seconds, Fetched: 14 row(s)
    ```


## 将Hive的数据迁移到MySQL上 {#section_9pg_6gi_v0w .section}

将Hive的数据迁移到MySQL上，需要先在MySQL上创建好对应Hive数据结构的表，然后在集群Sqoop节点上使用`sqoop export`命令进行迁移。

此处以迁移Hive上hive\_test.txt中的数据为例，hive\_test.txt中已写入如下数据。

``` {#codeblock_lip_z32_ves}
1,测试用户1,2019-08-10,男
2,测试用户2,2019-08-11,男
3,测试用户3,2019-08-12,男
4,测试用户4,2019-08-13,女
5,测试用户5,2019-08-14,女
```

1.  在MySQL上的sqoop\_migrate库中创建好要导入的表。 

    ``` {#codeblock_kz5_t0z_oy7}
    use sqoop_migrate ；
     CREATE TABLE `employeeOnHive`(
      `id` VARCHAR(20),
      `name` VARCHAR(20) NOT NULL DEFAULT '',
      `birth` VARCHAR(20) NOT NULL DEFAULT '',
      `sex` VARCHAR(10) NOT NULL DEFAULT '',
      PRIMARY KEY(`id`)
    );
    ```

2.  执行以下命令迁移数据。 

    ``` {#codeblock_4mw_2hu_ich}
    sqoop export --connect jdbc:mysql://172.0.0.0:3306/sqoop_migrate --username 'userid' --password 'userPW'   --table employeeOnHive -m 1  --fields-terminated-by ','  --export-dir /user/hive/warehouse/employeeonhive
    ```

    迁移命令格式：`sqoop export --connect jdbc:mysql://<dburi>/<dbname> --username <username> --password <password> --table <tablename> --export-dir <hive-dir> --fields-terminated-by <Splitter>`

    |参数|说明|
    |--|--|
    |dburi|数据库的访问连接。例如：jdbc:mysql://172.x.x.x:3306/ 。如果您的访问连接中含有参数，则请加上单引号，例如：'jdbc:mysql://172.x.x.x.235:3306/mydatabase?useUnicode=true&characterEncoding=UTF-8'。|
    |dbname|数据库的名字，例如：user。|
    |username|数据库登录用户名。|
    |password|用户密码。|
    |tablename|MySQL数据库中表的名称。|
    |hive-dir|存放待迁移数据的HDFS目录，此处以/sqoop2mysql/table/mysqltest.txt为例。|
    |Splitter|Hive中表中数据分隔符。hive默认为'\\001'。|

3.  验证迁移结果。 
    1.  执行以下进入数据库。 

        ``` {#codeblock_bdl_2wa_ci8}
        mysql -uroot -p
        ```

    2.  执行以下命令使用数据库。 

        ``` {#codeblock_h7r_23m_8h6}
        use  sqoop_migrate;
        ```

    3.  执行`select * from sqoop_migrate.employeeOnHive;`命令查看表数据。 

        如果表中有如下数据，则表示迁移成功。

        ``` {#codeblock_3pm_o2p_pxg}
        +----+---------------+------------+-----+
        | id | name          | birth      | sex |
        +----+---------------+------------+-----+
        | 1  | 测试用户1     | 2019-08-10 | 男  |
        | 2  | 测试用户2     | 2019-08-11 | 男  |
        | 3  | 测试用户3     | 2019-08-12 | 男  |
        | 4  | 测试用户4     | 2019-08-13 | 女  |
        | 5  | 测试用户5     | 2019-08-14 | 女  |
        +----+---------------+------------+-----+
        5 rows in set (0.00 sec)
        ```


