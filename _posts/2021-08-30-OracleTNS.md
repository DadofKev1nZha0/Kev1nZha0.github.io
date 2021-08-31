---
layout: post
title: Oracle ManagedDataAccess Client 连接TNS 
category: Oracle
tag: [Oracle]
---

在使用Oracle.ManagedDataAccess.Client通过TNS连接Oracle数据库的时候并不会自动读取Oracle Client的TNS配置或者是TNS_ADMIN环境变量路径下的配置，而是默认读取程序文件根目录或者app.config目录内的TNS_ADMIN路径下的tnsnames.ora
在建表的时候增加说明字段可以方便导出数据库字典交给对应的DBA

dataSources Section
This section can appear only under a <version> section. The mapping between the different data source aliases and corresponding data descriptors should appear in this section. The following is an example.

	<dataSources>
	<dataSource alias="inst1" descriptor="(DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=sales-server)......)))"/>
	<dataSource alias="inst2" descriptor="(DESCRIPTION= ......)))"/>
	</dataSources>
The following precedence order is followed to resolve the data source alias specified in the Data Source attribute in the connection string.

1.	data source alias in the dataSources section under <oracle.manageddataaccess.client> section in the .NET config file.

2.	data source alias in the tnsnames.ora file at the location specified by TNS_ADMIN in the .NET config file. Locations can consist of either absolute or relative directory paths.

3.	data source alias in the tnsnames.ora file present in the same directory as the .exe.




**参考连接:**

[Data Provider for .NET Developer's Guide](https://docs.oracle.com/cd/E63277_01/win.121/e63268/InstallManagedConfig.htm#ODPNT8161)
