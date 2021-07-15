---
layout: post
title: SQL Management Studio 2019 表设计界面增加说明列 
category: Git
tag: [SQL]
---

在建表的时候增加说明字段可以方便导出数据库字典交给对应的DBA

![](/images/images/20210615/SMSSDescription.jpg)

注册表修改项

	Windows Registry Editor Version 5.00

	[HKEY_CURRENT_USER\SOFTWARE\Microsoft\SQL Server Management Studio\18.0_IsoShell\DataProject]
	"SSVPropViewColumnsSQL70"="1,2,6,17;"
	"SSVPropViewColumnsSQL80"="1,2,6,17;"


注意：在设置注册表时，管理器需要是关闭状态。
其中，各数字代表的意思如下：
1. Column Name
2. Data Type
3. Length
4. Precision
5. Scale
6. Allow Nulls
7. Default Value
8. Identity
9. Identity Seed
10. Identity Increment
11. Row GUID
12. Nullable
13. Condensed Type
14. Not for Replication
15. Formula
16. Collation
17. Description
