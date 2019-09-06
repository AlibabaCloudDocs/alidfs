# SDK示例 {#concept_2039229 .concept}

本文列出了创建目录、删除、上传文件、下载文件、显示目录、写入文件，读取文件、测试等操作的SDK示例，您可以参考示例工程开发您的应用。

## 背景信息 {#section_a7c_n03_xy2 .section}

文件存储HDFS提供对Apache Hadoop FileSystem API的兼容，您可以参考[Hadoop FileSystem API](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/fs/FileSystem.html)进行开发。

**说明：** 目前，部分Hadoop FileSystem API的兼容还未在文件存储HDFS SDK中提供，详情请参见[使用限制](../../../../cn.zh-CN/产品简介/使用限制.md#)。

## 准备工作 {#section_3gw_4gh_toy .section}

1.  已完成文件存储HDFS的配置，详情请参见[快速入门](../../../../cn.zh-CN/快速入门/开始使用文件存储HDFS.md#)。
2.  已安装SDK，详情请参见[安装](cn.zh-CN/SDK 参考/安装.md#)。
3.  在计算节点上安装JDK，版本不能低于1.8。
4.  在计算节点上安装hadoop版本建议不低于2.7.2。
5.  配置maven的依赖配置。

    ``` {#codeblock_nro_n53_80m}
        <dependency>
                <groupId>org.apache.hadoop</groupId>
                <artifactId>hadoop-client</artifactId>
                <version>2.7.2</version>
                <!--hadoop版本建议不低于 2.7.2 -->
        </dependency>
    ```


## 创建目录 {#section_xm9_hlt_lkn .section}

-   示例

    ``` {#codeblock_632_6ql_om9}
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.FileSystem;
    import org.apache.hadoop.fs.Path;
    
    import java.io.IOException;
    
    public class exampleMkDir {
    
        public static void main(String[] args) throws IOException {
    
            if (args.length != 1) {
                System.out.println("本类为创建目录示例类，需要传入一个要创建目录路径的参数。\n" +
                        "例如:  hadoop  jar hdfs_example-1.0-SNAPSHOT.jar com.alibaba.dfs.examples.exampleMkDir /tmp/hdfs_test");
                System.exit(-1);
            }
    
            String fileName = args[0];
    
            //设置操作hdfs文件系统的用户，用户可自行设置
            //注意这里设置的用户，最好和在linux上运行该代码的用户一致
            System.setProperty("HADOOP_USER_NAME", "root");
    
            //创建连接配置
            Configuration hdfsConf = new Configuration();
    
            //创建文件系统实例对象
            FileSystem hadoopFS = FileSystem.get(hdfsConf);
            //在文件存储hdfs上创建目录
            boolean mkdirsSuccess = hadoopFS.mkdirs(new Path(fileName));
    
            //使用完后，请释放资源，否则可能导致Resource busy错误
            hadoopFS.close();
    
            System.out.println("创建是否成功：" + mkdirsSuccess);
            System.out.println("用户可以使用hadoop命令查看创建的目录，例如 ：hadoop fs -ls  " + fileName);
        }
    }
    ```

-   运行示例

    如果创建成功，则执行`hadoop fs -ls /tmp/hdfs_test`命令无返回值。否则会报错No such file or directory。

    ![创建目录运行示例](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1618218/156775669359036_zh-CN.png)


## 移动或者重命名 {#section_dv4_sxt_c9o .section}

-   示例

    ``` {#codeblock_ef7_cir_xxy}
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.FileSystem;
    import org.apache.hadoop.fs.Path;
    
    import java.io.IOException;
    
    public class exampleRename {
    
        public static void main(String[] args) throws IOException {
    
            if (args.length != 2) {
                System.out.println("本类为移动或者重命名示例类，需要传入两个的路径参数。第一个为需要移动或重命名的路径，第二个为移动或重命名后的路径 \n" +
                        "例如： hadoop  jar hdfs_example-1.0-SNAPSHOT.jar com.alibaba.dfs.examples.exampleRename /tmp/hdfs_test  /tmp/hdfs_test2");
                System.exit(-1);
            }
    
            String fileName = args[0];
            String fileRename = args[1];
    
            //设置操作hdfs文件系统的用户，用户可自行设置
            //注意这里设置的用户，最好和在linux上运行该代码的用户一致
            System.setProperty("HADOOP_USER_NAME", "root");
    
    
            //创建连接配置
            Configuration hdfsConf = new Configuration();
    
            //创建文件系统实例对象
            FileSystem hadoopFS = FileSystem.get(hdfsConf);
    
             if (hadoopFS.exists(new Path(fileName))) {
                System.out.println("文件存储hdfs上不存在" + fileName);
            }
    
            boolean renameSuccess = hadoopFS.rename(new Path(fileName), new Path(fileRename));
            System.out.println("改名是否成功" + renameSuccess);
            System.out.println("用户可以使用hadoop命令查看移动或者重命名后文件，例如 ：hadoop fs -ls  " + fileRename);
    
            //使用完后，请释放资源，否则可能导致Resource busy错误
            hadoopFS.close();
    
            System.out.println("改名是否成功"+renameSuccess);
            System.out.println("用户可以使用hadoop命令查看移动或者重命名后文件，例如 ：hadoop fs -ls  " + fileRename);
        }
    }
    ```

-   运行示例
    -   重命名

        如果重命名成功，则执行`hadoop fs -ls /tmp/hdfs_test2`命令无返回值。否则会报错No such file or directory。

        ![重命名运行示例](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1618218/156775669359037_zh-CN.png)

    -   移动

        如果移动成功，则执行`hadoop fs -ls /tmp/hdfs_test2`命令，可以看到/tmp/hdfs\_test2目录下有内容。

        ![移动运行示例](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1618218/156775669359038_zh-CN.png)


## 删除 {#section_w3v_4wq_n6i .section}

-   请求

    ``` {#codeblock_ndv_5m3_ozo}
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.FileSystem;
    import org.apache.hadoop.fs.Path;
    
    import java.io.IOException;
    
    public class exampleDelete {
    
        public static void main(String[] args) throws IOException {
    
            if (args.length != 1) {
                System.out.println("本类为删除示例类，需要传入一个要删除的路径。\n" +
                        "例如： hadoop  jar hdfs_example-1.0-SNAPSHOT.jar com.alibaba.dfs.examples.exampleDelete /dfs_test2 ");
                System.exit(-1);
            }
    
            String fileName = args[0];
    
            //设置操作hdfs文件系统的用户，用户可自行设置
            //注意这里设置的用户，最好和在linux上运行该代码的用户一致
            System.setProperty("HADOOP_USER_NAME", "root");
    
            //创建连接配置
            Configuration hdfsConf = new Configuration();
    
            //创建文件系统实例对象
            FileSystem hadoopFS = FileSystem.get(hdfsConf);
    
            if (hadoopFS.exists(new Path(fileName))) {
                boolean deleteSuccess = hadoopFS.delete(new Path(fileName), true);
                System.out.println("删除是否成功:" + deleteSuccess);
                System.out.println("用户可以使用hadoop命令查看是否存在删除的目录，例如 ：hadoop fs -ls  " + fileName);
    
            } else {
                System.out.println("hdfs文件系统上不存在文件：" + fileName);
            }
    
            //使用完后，请释放资源，否则可能导致Resource busy错误
            hadoopFS.close();
    
        }
    }
    ```

-   运行示例

    如果删除成功，则执行`hadoop fs -ls /tmp/hdfs_test2`命令将返回ls: \`/tmp/hdfs\_test2': No such file or directory。

    ![删除运行示例](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1618218/156775669359039_zh-CN.png)


## 上传文件 {#section_yoq_8jd_ux6 .section}

-   示例

    ``` {#codeblock_l4v_2wb_v2f}
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.FileSystem;
    import org.apache.hadoop.fs.Path;
    
    import java.io.IOException;
    
    public class exampleUp {
    
        public static void main(String[] args) throws IOException {
    
            if (args.length != 2) {
                System.out.println("本类为上传文件(将本地文件上传到hdfs上)示例类，需要传入两个的路径。第一个为需要上传的本地文件路径，第二个为上传到hdfs上的文件路径 \n" +
                        "例如:  hadoop  jar hdfs_example-1.0-SNAPSHOT.jar com.alibaba.dfs.examples.exampleUp /root/test_local.txt  /tmp/test_hdfs.txt ");
                System.exit(-1);
            }
    
            String localFilename = args[0];
            String fileName = args[1];
    
            //设置操作hdfs文件系统的用户，用户可自行设置
            //注意这里设置的用户，最好和在linux上运行该代码的用户一致
            System.setProperty("HADOOP_USER_NAME", "root");
    
            //创建连接配置
            Configuration hdfsConf = new Configuration();
    
            //创建文件系统实例对象
            FileSystem hadoopFS = FileSystem.get(hdfsConf);
    
            if (!hadoopFS.exists(new Path(fileName))) {
                hadoopFS.copyFromLocalFile(new Path(localFilename), new Path(fileName));
                System.out.println("上传是否成功" + hadoopFS.exists(new Path(fileName)));
                System.out.println("用户可以使用hadoop命令查看上传的文件，例如 ：hadoop fs -ls  " + fileName);
            } else {
                System.out.println("上传失败，在文件存储hdfs上已经存在该文件");
            }
    
            //使用完后，请释放资源，否则可能导致Resource busy错误
            hadoopFS.close();
    
        }
    }
    ```

-   运行示例

    如果上传成功，则执行`hadoop fs -ls /tmp/test_hdfs.txt`命令可以在文件存储HDFS上看到该文件。

    ![上传文件](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1618218/156775669359044_zh-CN.png)


## 下载文件 {#section_whn_unl_wn6 .section}

-   示例

    ``` {#codeblock_f56_33v_ept}
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.FileSystem;
    import org.apache.hadoop.fs.Path;
    
    import java.io.File;
    import java.io.IOException;
    
    public class exampleDown {
        public static void main(String[] args) throws IOException {
    
            if (args.length != 2) {
                System.out.println("本类为下载文件(将hdfs上的文件下载到本地)示例类，需要传入两个的路径。第一个为需要下载的hdfs文件路径，第二个为下载到本地的路径 \n" +
                        "例如:  hadoop  jar hdfs_example-1.0-SNAPSHOT.jar com.alibaba.dfs.examples.exampleDown /tmp/test_hdfs.txt  ./test_hdfs.txt ");
                System.exit(-1);
            }
    
            String fileName = args[0];
            String localFilename = args[1];
    
            //设置操作hdfs文件系统的用户，用户可自行设置
            //注意这里设置的用户，最好和在linux上运行该代码的用户一致
            System.setProperty("HADOOP_USER_NAME", "root");
    
            //创建连接配置
            Configuration hdfsConf = new Configuration();
    
            //创建文件系统实例对象
            FileSystem hadoopFS = FileSystem.get(hdfsConf);
            if (!new File(localFilename).exists()) {
                hadoopFS.copyToLocalFile(new Path(fileName), new Path(localFilename));
                System.out.println("下载是否成功" + new File(localFilename).exists());
                System.out.println("用户可以在本地目录" + localFilename + "下看到下载到的文件。");
            } else {
                System.out.println("下载失败，在本地已经存在该文件");
            }
    
            //使用完后，请释放资源，否则可能导致Resource busy错误
            hadoopFS.close();
    
        }
    }
    ```

-   运行示例

    如果下载成功，可以在本地看到该文件。

    ![下载文件](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1618218/156775669359047_zh-CN.png)


## 显示目录 {#section_yve_gk0_kdn .section}

-   示例

    ``` {#codeblock_8yh_5ej_0a6}
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.FileStatus;
    import org.apache.hadoop.fs.FileSystem;
    import org.apache.hadoop.fs.Path;
    
    import java.io.IOException;
    
    public class exampleLs {
    
        public static void main(String[] args) throws IOException {
    
            if (args.length != 1) {
                System.out.println("本类为查看文件存储hdfs上目录信息示例类，需要传入一个要查看的路径。 \n" +
                        "例如:  hadoop  jar hdfs_example-1.0-SNAPSHOT.jar com.alibaba.dfs.examples.exampleLs / ");
                System.exit(-1);
            }
    
            String fileName = args[0];
    
            //设置操作hdfs文件系统的用户，用户可自行设置
            //注意这里设置的用户，最好和在linux上运行该代码的用户一致
            System.setProperty("HADOOP_USER_NAME", "root");
    
            //创建连接配置
            Configuration hdfsConf = new Configuration();
    
            //创建文件系统实例对象
            FileSystem hadoopFS = FileSystem.get(hdfsConf);
    
            FileStatus[] fileStatuses = hadoopFS.listStatus(new Path(fileName));
            for (FileStatus file : fileStatuses) {
                System.out.print(file.getPermission() + "\t");
                System.out.print(file.getOwner() + "\t");
                System.out.print(file.getGroup() + "\t");
                System.out.print(file.getAccessTime() + "\t");
                System.out.print(file.getPath() + "\n");
            }
    
            //使用完后，请释放资源，否则可能导致Resource busy错误
            hadoopFS.close();
            System.out.println("\n\n用户可以使用hadoop命令查看文件存储hdfs "+ fileName +" 目录下的内容，例如 ：hadoop fs -ls " + fileName);
        }
    }
    ```

-   运行示例

    如果该方法运行成功，则执行`hadoop fs -ls /`命令的返回结果和exampleLs返回结果大致相同。

    ![显示目录](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1618218/156775669359050_zh-CN.png)


## 写入文件 {#section_y1f_42v_ooc .section}

-   示例

    ``` {#codeblock_5iz_jd9_nst}
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.FSDataOutputStream;
    import org.apache.hadoop.fs.FileSystem;
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.io.IOUtils;
    
    import java.io.BufferedInputStream;
    import java.io.ByteArrayInputStream;
    import java.io.IOException;
    import java.io.InputStream;
    
    public class exampleWrite {
    
        public static void main(String[] args) throws IOException {
    
            if (args.length != 2) {
                System.out.println("本类为写入hdfs文件系统上的文件示例类，需要传入两个的参数。第一个为的hdfs文件上一个写入文件的路径，第二个为写入文件中的内容 \n" +
                        "例如:  hadoop  jar hdfs_example-1.0-SNAPSHOT.jar com.alibaba.dfs.examples.exampleWrite  /tmp/test_write.txt 测试输入到hdfs上的文件的内容 ");
                System.exit(-1);
            }
    
            String filename = args[0];
            String msg = args[1] + "\n";
    
            //设置操作hdfs文件系统的用户，用户可自行设置
            //注意这里设置的用户，最好和在linux上运行该代码的用户一致
            System.setProperty("HADOOP_USER_NAME", "root");
    
            //创建连接配置
            Configuration hdfsConf = new Configuration();
    
            //创建文件系统实例对象
            FileSystem hadoopFS = FileSystem.get(hdfsConf);
            //创建文件流
            InputStream in = new BufferedInputStream(new ByteArrayInputStream(msg.getBytes("UTF-8")));
    
            //在hdfs上创建一个测试文件
            FSDataOutputStream out = hadoopFS.create(new Path(filename), true);
            //利用IOUtils.copyBytes进行写入
            IOUtils.copyBytes(in, out, 1024 * 8, true);
            System.out.println("已经写入hdfs文件系统上的" + filename + "文件");
            System.out.println("用户可以使用hadoop命令查看写入文件的内容 ：hadoop fs -cat  " + filename);
    
            //使用完后，请释放资源，否则可能导致Resource busy错误
            hadoopFS.close();
    
        }
    }
    ```

-   运行示例

    若该方法运行成功，则执行`hadoop fs -cat /tmp/test_write.txt`命令的返回结果和exampleWrite写入该文件的内容相同。

    ![写入文件](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1618218/156775669459051_zh-CN.png)


## 读取文件 {#section_ld5_lce_sqf .section}

-   示例

    ``` {#codeblock_oew_5oc_d1c}
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.FSDataInputStream;
    import org.apache.hadoop.fs.FileSystem;
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.io.IOUtils;
    
    import java.io.IOException;
    import java.io.OutputStream;
    
    public class exampleRead {
        public static void main(String[] args) throws IOException {
    
            if (args.length != 1) {
                System.out.println("本类为读取hdfs文件系统上的文件示例类，需要传入一个文件存储hdfs上的文件路径的参数。 \n" +
                        "例如:  hadoop  jar hdfs_example-1.0-SNAPSHOT.jar com.alibaba.dfs.examples.exampleRead /tmp/test_write.txt");
                System.exit(-1);
            }
    
            String filename = args[0];
    
            //设置操作hdfs文件系统的用户，用户可自行设置
            //注意这里设置的用户，最好和在linux上运行该代码的用户一致
            System.setProperty("HADOOP_USER_NAME", "root");
    
            //创建连接配置
            Configuration hdfsConf = new Configuration();
    
            //创建文件系统实例对象
            FileSystem hadoopFS = FileSystem.get(hdfsConf);
    
            if (hadoopFS.exists(new Path(filename))) {
                //创建文件流
                FSDataInputStream in = hadoopFS.open(new Path(filename));
                OutputStream out = System.out;
    
                IOUtils.copyBytes(in, out, 1024 * 8, );
    
                System.out.println("\n 用户可以使用hadoop命令查看文件的内容 ，例如：hadoop fs -cat  " + filename);
            } else {
                System.out.println("hdfs文件系统上不存在文件：" + filename);
            }
    
            //使用完后，请释放资源，否则可能导致Resource busy错误
            hadoopFS.close();
        }
    }
    ```

-   运行示例

    如果该方法运行成功，则执行`hadoop fs -cat /tmp/test_write.txt`命令的返回结果和exampleRead打印的内容相同。

    ![读取文件](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1618218/156775669459052_zh-CN.png)


## 整体测试 {#section_h4q_29x_ufn .section}

-   示例

    ``` {#codeblock_k4k_pgy_8w5}
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.*;
    import org.apache.hadoop.io.IOUtils;
    
    import java.io.*;
    
    public class exampleAll {
    
        public static void main(String[] args) throws Exception {
    
            System.out.println("该测试类是包含本测试包中的所有API测试\n\n");
    
            //设置操作hdfs文件系统的用户，用户可自行设置
            //注意这里设置的用户，最好和在linux上运行该代码的用户一致
            System.setProperty("HADOOP_USER_NAME", "root");
    
            //创建连接配置
            Configuration hdfsConf = new Configuration();
    
            //创建文件系统实例对象
            FileSystem hadoopFS = FileSystem.get(hdfsConf);
    
            System.out.println("在文件存储hdfs上创建目录/hdfs_mkdirTest");
            System.out.println("创建是否成功：" + hadoopFS.mkdirs(new Path("/hdfs_mkdirTest")) + "\n");
            Thread.sleep(2 * 1000);
    
            System.out.println("在文件存储hdfs上修改目录/hdfs_mkdirTest 为 /hdfs_renameTest ");
            System.out.println("改名是否成功: " + hadoopFS.rename(new Path("/hdfs_mkdirTest"), new Path("/hdfs_renameTest")) + "\n");
            Thread.sleep(2 * 1000);
    
            System.out.println("在文件存储hdfs上删除目录 /hdfs_renameTest ");
            System.out.println("删除是否成功: " + hadoopFS.delete(new Path("/hdfs_renameTest"), true) + "\n");
            Thread.sleep(2 * 1000);
    
            new File("./hdfs_upTest.txt").delete();
            new File("./hdfs_downTest.txt").delete();
    
            new File("./hdfs_upTest.txt").createNewFile();
            System.out.println("向文件存储hdfs的/hdfs_Test中，上传本地文件 ./hdfs_upTest.txt ");
            hadoopFS.mkdirs(new Path("/hdfs_Test"));
            hadoopFS.copyFromLocalFile(new Path("./hdfs_upTest.txt"), new Path("/hdfs_Test/hdfs_upTest.txt"));
            System.out.println("上传是否成功: " + hadoopFS.exists(new Path("/hdfs_Test/hdfs_upTest.txt")) + "\n");
            Thread.sleep(2 * 1000);
    
            System.out.println("从文件存储hdfs的/hdfs_Test中，下载文件hdfs_upTest.txt到当前目录 ");
            hadoopFS.copyToLocalFile(new Path("/hdfs_Test/hdfs_upTest.txt"), new Path("./hdfs_downTest.txt"));
            System.out.println("下载是否成功: " + new File("./hdfs_downTest.txt").exists() + "\n");
            Thread.sleep(2 * 1000);
    
    
            System.out.println("向文件存储hdfs上的/hdfs_Test/hdfs_writeTest.txt文件中写入测试内容");
            InputStream win = new BufferedInputStream(new ByteArrayInputStream(("测试写入文件存储hdfs 1 \n" +
                    "测试写入文件存储hdfs 2 \n" +
                    "测试写入文件存储hdfs 3 \n").getBytes("UTF-8")));
            FSDataOutputStream wout = hadoopFS.create(new Path("/hdfs_Test/hdfs_writeTest.txt"), true);
            IOUtils.copyBytes(win, wout, 1024 * 8, true);
            System.out.println("已经写入到/hdfs_Test/hdfs_writeTest.txt文件\n");
    
            Thread.sleep(2 * 1000);
    
            System.out.println("从文件存储hdfs上的/hdfs_Test/hdfs_writeTest.txt文件中读取之前写入的内容");
            FSDataInputStream rin = hadoopFS.open(new Path("/hdfs_Test/hdfs_writeTest.txt"));
            OutputStream rout = System.out;
            System.out.println("/hdfs_Test/hdfs_writeTest.txt文件中的內容为：");
            IOUtils.copyBytes(rin, rout, 1024 * 8, false);
            System.out.println();
    
    
            System.out.println("查看文件存储hdfs上 /hdfs_Test 目录 ");
            FileStatus[] fileStatuses = hadoopFS.listStatus(new Path("/hdfs_Test"));
            for (FileStatus file : fileStatuses) {
                System.out.print(file.getPermission() + "\t");
                System.out.print(file.getOwner() + "\t");
                System.out.print(file.getGroup() + "\t");
                System.out.print(file.getPath() + "\n");
            }
            System.out.println();
            System.out.println("测试完毕！");
    
            //使用完后，请释放资源，否则可能导致Resource busy错误
            hadoopFS.close();
            rout.close();
            rin.close();
    
        }
    }
    ```

-   运行示例

    ![整体测试](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1618218/156775669459053_zh-CN.png)


