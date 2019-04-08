# 使用RAM授权访问 {#concept_kbm_zdv_hhb .concept}

您可以使用[RAM](../../../../../cn.zh-CN/产品简介/什么是 RAM？.md#)为子用户授权，使其获得文件存储HDFS的管控操作权限。为了遵循最佳安全实践，强烈建议您使用子用户来操作文件存储HDFS。

## 文件存储HDFS默认授权策略 {#section_qpp_3lw_hhb .section}

文件存储HDFS默认授权策略如下：

|策略|说明|
|:-|:-|
|AliyunHDFSReadOnlyAccess|只读访问文件存储HDFS的权限|
|AliyunHDFSFullAccess|管理文件存储HDFS的权限|

## RAM中可授权的文件存储HDFS管控操作 {#section_hwb_hqm_hfb .section}

在RAM中可以为子用户授予以下文件存储HDFS操作的权限：

|Action|Resource|说明|
|------|--------|--|
|dfs:CreateFileSystem|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/\*|创建文件系统|
|dfs:DeleteFileSystem|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\}|删除已有的文件系统实例|
|dfs:ModifyFileSystem|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\}|修改文件系统属性|
|dfs:GetFileSystem|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\}|获取文件系统详细信息|
|dfs:ListFileSystems|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/\*|批量获取文件系统详细信息|
|dfs:CreateAccessGroup|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/\*|创建权限组|
|dfs:DeleteAccessGroup|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|删除权限组|
|dfs:ModifyAccessGroup|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|修改权限组属性|
|dfs:GetAccessGroup|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|获取权限组信息|
|dfs:ListAccessGroups|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/\*|批量获取权限组信息|
|dfs:CreateAccessRule|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|创建权限规则|
|dfs:DeleteAccessRule|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|删除权限规则|
|dfs:ModifyAccessRule|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|修改规则属性|
|dfs:GetAccessRule|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|获取规则详细信息|
|dfs:ListAccessRules|acs:dfs:$\{region-id\}:$\{account-id\}:accessgroup/$\{access-group-id\}|批量获取规则|
|dfs:CreateMountPoint|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\} acs:vpc:$\{region-id\}:$\{account-id\}:vswitch/$\{vswitch-id\}|创建挂载点|
|dfs:DeleteMountPoint|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\}|删除挂载点|
|dfs:ModifyMountPoint|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\}|修改挂载点属性|
|dfs:GetMountPoint|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\}|获取挂载点详细信息|
|dfs:ListMountPoints|acs:dfs:$\{region-id\}:$\{account-id\}:filesystem/$\{file-system-id\}|批量获取挂载点|
|dfs:\*|\*| -   Action列的dfs:\*表示所有文件存储HDFS的管控操作
-   Resource列的\*表示所有文件存储HDFS的资源

 |

-   挂载点是文件系统实例的附属资源，如需操作挂载点，需要所属文件系统实例的权限。

    **说明：** 创建挂载点时，除了要求文件系统实例的权限外，还需要VPC和VSwitch的权限。

-   权限组规则是权限组的附属资源，如需操作权限组规则，需要所属权限组的权限。

## 授权策略 {#section_jrm_kqm_hfb .section}

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

