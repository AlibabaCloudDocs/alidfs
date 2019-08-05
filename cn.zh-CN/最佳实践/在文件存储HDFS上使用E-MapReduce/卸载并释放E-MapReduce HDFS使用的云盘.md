# 卸载并释放E-MapReduce HDFS使用的云盘 {#task_1545709 .task}

本文介绍在配置E-MapReduce完成后，如何卸载并释放E-MapReduce HDFS服务使用的云盘。

1.  已完成数据迁移，详情请参见[E-MapReduce数据迁移](cn.zh-CN/最佳实践/在文件存储HDFS上使用E-MapReduce/E-MapReduce数据迁移.md#)。
2.  已配置E-MapReduce使用文件存储HDFS，详情请参见[配置E-MapReduce服务使用文件存储HDFS](cn.zh-CN/最佳实践/在文件存储HDFS上使用E-MapReduce/配置E-MapReduce服务使用文件存储HDFS.md#)。
3.  在卸载磁盘前，请停止E-MapReduce集群中的所有服务，等到卸载磁盘操作完成后再启动。

当E-MapReduce已经成功运行在阿里云文件存储HDFS上时，ECS挂载的云盘只用来存储运算中的临时Shuffle文件，可以选择卸载原来用于构建E-MapReduce HDFS服务的云盘，降低集群的拥有成本。

**说明：** 从数据安全性考虑，数据迁移后建议进行数据完整性校验并让E-MapReduce系统在文件存储HDFS上正常运行一段时间后再卸载和释放云盘。云盘释放以后原有数据将无法找回。集群中的每台机器至少需要保留一块数据盘，通常是/mnt/disk1上挂载的磁盘。

1.  卸载数据盘，详情请参见[卸载数据盘](../../../../cn.zh-CN/块存储/云盘/卸载数据盘.md#)。
2.  释放云盘，详情请参见[释放云盘](../../../../cn.zh-CN/块存储/云盘/释放云盘.md#)。

