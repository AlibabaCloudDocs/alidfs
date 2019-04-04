# 使用RAM授权访问 {#concept_kbm_zdv_hhb .concept}

您可以使用[RAM](../../../../../cn.zh-CN/产品简介/什么是 RAM？.md#)为子用户授权，使其获得文件存储HDFS的管控操作权限。为了遵循最佳安全实践，强烈建议您使用子用户来操作文件存储HDFS。

## RAM中可授权的文件存储HDFS操作 {#section_lzl_4pm_hfb .section}

在RAM中可以为子用户授予以下文件存储HDFS操作的权限。

|操作（Action）|说明|
|----------|--|
|DescribeRegions|返回所有开服区域的RegionId|
|GetRegion|获取Region可用区的配置|
|CreateFileSystem|创建文件系统|
|DeleteFileSystem|删除已有的文件系统实例|
|ModifyFileSystem|修改文件系统属性|
|GetFileSystem|获取文件系统详细信息|
|ListFileSystems|批量获取文件系统详细信息|
|CreateAccessGroup|创建权限组|
|DeleteAccessGroup|删除权限组|
|ModifyAccessGroup|修改权限组属性|
|GetAccessGroup|获取权限组信息|
|ListAccessGroups|批量获取权限组信息|
|CreateAccessRule|创建权限规则|
|DeleteAccessRule|删除权限规则|
|ModifyAccessRule|修改规则属性|
|GetAccessRule|获取规则详细信息|
|ListAccessRules|批量获取规则|
|CreateMountPoint|创建挂载点|
|DeleteMountPoint|删除挂载点|
|ModifyMountPoint|修改挂载点属性|
|GetMountPoint|获取挂载点详细信息|
|ListMountPoints|批量获取挂载点|

## RAM中可授权的文件存储HDFS管控操作 {#section_hwb_hqm_hfb .section}

在RAM授权策略中，文件存储HDFS支持以下资源抽象：

|Action|Resource|
|------|--------|
|dfs:CreateFileSystem|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/\*|
|dfs:CreateFileSystem|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/\*|
|dfs:DeleteFileSystem|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\}|
|dfs:ModifyFileSystem|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\}|
|dfs:GetFileSystem|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\}|
|dfs:ListFileSystems|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/\*|
|dfs:CreateAccessGroup|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/\*|
|dfs:DeleteAccessGroup|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|
|dfs:ModifyAccessGroup|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|
|dfs:GetAccessGroup|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|
|dfs:ListAccessGroups|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/\*|
|dfs:CreateAccessRule|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|
|dfs:DeleteAccessRule|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|
|dfs:ModifyAccessRule|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|
|dfs:GetAccessRule|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|
|dfs:ListAccessRules|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|
|dfs:CreateMountPoint|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\} acs:vpc:$\{region-id\}:$\{account-id\}:vswitch/$\{vswitch-id\}|
|dfs:DeleteMountPoint|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\}|
|dfs:ModifyMountPoint|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\}|
|dfs:GetMountPoint|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\}|
|dfs:ListMountPoints|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\}|
|dfs:\*|\*|

**说明：** Action中的dfs:\*表示所有文件存储HDFS的管控操作，Resource中的\*表示所有文件存储HDFS的资源。

此外，文件存储HDFS还支持资源粒度授权。

-   挂载点是文件系统实例的附属资源，如需操作挂载点，需要所属文件系统实例的权限。

    **说明：** 创建挂载点时，除了要求文件系统实例的权限外，还需要VPC和VSwitch的权限。

-   权限组规则是权限组的附属资源，如需操作权限组规则，需要所属权限组的权限。

## 授权策略 {#section_jrm_kqm_hfb .section}

文件存储HDFS默认授权策略如下：

|策略|说明|
|:-|:-|
|AliyunHDFSReadOnlyAccess|文件存储HDFS管控系统的只读权限|
|AliyunHDFSFullAccess|文件存储HDFS管控系统的所有权限|

授予子账号文件存储HDFS管控系统只读权限的策略示例如下：

```
{
  "Version": "1",
  "Statement": [
    {
      "Action": [
        "dfs:Get*",
        "dfs:List*"
      ],
      "Resource": "*",
      "Effect": "Allow"
    }
  ]
}
```

授予子账号操作文件存储HDFS文件系统实例的策略示例如下：

```
{
    "Version": "1",
    "Statement": [
       {
          "Effect": "Allow",
          "Action": "dfs:GetFileSystem",
          "Resource": "acs:dfs:*:*:HDFSInstanceId" 
        
    ]
}
```

**说明：** HDFSInstanceId为文件系统实例ID。

