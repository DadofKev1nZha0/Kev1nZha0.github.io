---
layout: post
title: SQL自定义端口的连接 
category: SQL
tag: [SQL]
---

众所周知.SQL Server的默认端口是1433

多年之前连接非1433端口的时候SQL连接字符串需要加上自定义的端口号才可以连接成功

比如: 

    public static DataTable QuerySQL(int timeout = 60)
    {
        string SQLConnectionString = "localhost\SQLEXPRESS,8266;database=TEST;uid=sa;pwd=sa";
        string SQLComannd = "select * from TEST";
        DataTable query = new DataTable();
        SqlConnection conn = new SqlConnection();
        if (conn.State == ConnectionState.Open)
            conn.Close();
        conn.ConnectionString = SQLConnectionString;
        conn.Open();
        SqlDataAdapter da_refresh = new SqlDataAdapter(SQLComannd, conn);
        da_refresh.SelectCommand.CommandTimeout = timeout;
        da_refresh.Fill(query);
        conn.Close();
        conn.Dispose();
        return (query);
    }

但是发现了个很奇怪的现象,当我修改SQL的端口号,SqlConnection依然可以连接上.

显然是SqlConnection通过了某种方法获取到了SQL自定义的端口.

开始还以为是.Net Framework 功劳,后来经过查证发现真正实现这个功能的是SQL Server的 SQL Server Browser

SQL Server Browser 会打开UDP 1434 端口,将SQL真正的端口号提供给Client

     UDP    [::]:1434              *:*                                    4256

所以当 SQL Server Browser 禁用的时候,SqlConnection在未申明端口号的时候只会连接到1433.