---
layout: post
title: SQL Server Alwayson 账号同步
category: SQL
tag: [SQL]
---

在做SQL Server Alwayson 迁移的时候当使用SQL账号时,主库加上权限,备库无法同步权限,使用域账号则没有这个问题

本能的想到可能是因为主备库是分别添加的SQL账号,账号的SID不一致,导致的SQL在备库加权限的时候失败.

那么如何复制一个完全相同的账号去备库,参考微软提供的方案:

1. 在主库建立生成账号语句的存储过程

        USE master
        GO
        IF OBJECT_ID ('sp_hexadecimal') IS NOT NULL
        DROP PROCEDURE sp_hexadecimal
        GO
        CREATE PROCEDURE sp_hexadecimal
        @binvalue varbinary(256),
        @hexvalue varchar (514) OUTPUT
        AS
        DECLARE @charvalue varchar (514)
        DECLARE @i int
        DECLARE @length int
        DECLARE @hexstring char(16)
        SELECT @charvalue = '0x'
        SELECT @i = 1
        SELECT @length = DATALENGTH (@binvalue)
        SELECT @hexstring = '0123456789ABCDEF'
        WHILE (@i <= @length)
        BEGIN
        DECLARE @tempint int
        DECLARE @firstint int
        DECLARE @secondint int
        SELECT @tempint = CONVERT(int, SUBSTRING(@binvalue,@i,1))
        SELECT @firstint = FLOOR(@tempint/16)
        SELECT @secondint = @tempint - (@firstint*16)
        SELECT @charvalue = @charvalue +
        SUBSTRING(@hexstring, @firstint+1, 1) +
        SUBSTRING(@hexstring, @secondint+1, 1)
        SELECT @i = @i + 1
        END

        SELECT @hexvalue = @charvalue
        GO

        IF OBJECT_ID ('sp_help_revlogin') IS NOT NULL
        DROP PROCEDURE sp_help_revlogin
        GO
        CREATE PROCEDURE sp_help_revlogin @login_name sysname = NULL AS
        DECLARE @name sysname
        DECLARE @type varchar (1)
        DECLARE @hasaccess int
        DECLARE @denylogin int
        DECLARE @is_disabled int
        DECLARE @PWD_varbinary varbinary (256)
        DECLARE @PWD_string varchar (514)
        DECLARE @SID_varbinary varbinary (85)
        DECLARE @SID_string varchar (514)
        DECLARE @tmpstr varchar (1024)
        DECLARE @is_policy_checked varchar (3)
        DECLARE @is_expiration_checked varchar (3)

        DECLARE @defaultdb sysname

        IF (@login_name IS NULL)
        DECLARE login_curs CURSOR FOR

        SELECT p.sid, p.name, p.type, p.is_disabled, p.default_database_name, l.hasaccess, l.denylogin FROM
        sys.server_principals p LEFT JOIN sys.syslogins l
        ON ( l.name = p.name ) WHERE p.type IN ( 'S', 'G', 'U' ) AND p.name <> 'sa'
        ELSE
        DECLARE login_curs CURSOR FOR

        SELECT p.sid, p.name, p.type, p.is_disabled, p.default_database_name, l.hasaccess, l.denylogin FROM
        sys.server_principals p LEFT JOIN sys.syslogins l
        ON ( l.name = p.name ) WHERE p.type IN ( 'S', 'G', 'U' ) AND p.name = @login_name
        OPEN login_curs

        FETCH NEXT FROM login_curs INTO @SID_varbinary, @name, @type, @is_disabled, @defaultdb, @hasaccess, @denylogin
        IF (@@fetch_status = -1)
        BEGIN
        PRINT 'No login(s) found.'
        CLOSE login_curs
        DEALLOCATE login_curs
        RETURN -1
        END
        SET @tmpstr = '/* sp_help_revlogin script '
        PRINT @tmpstr
        SET @tmpstr = '** Generated ' + CONVERT (varchar, GETDATE()) + ' on ' + @@SERVERNAME + '*/'
        PRINT @tmpstr
        PRINT ''
        WHILE (@@fetch_status <> -1)
        BEGIN
        IF (@@fetch_status <> -2)
        BEGIN
        PRINT ''
        SET @tmpstr = '-- Login: ' + @name
        PRINT @tmpstr
        IF (@type IN ( 'G', 'U'))
        BEGIN -- NT authenticated account/group

        SET @tmpstr = 'CREATE LOGIN ' + QUOTENAME( @name ) + ' FROM WINDOWS WITH DEFAULT_DATABASE = [' + @defaultdb + ']'
        END
        ELSE BEGIN -- SQL Server authentication
        -- obtain password and sid
        SET @PWD_varbinary = CAST( LOGINPROPERTY( @name, 'PasswordHash' ) AS varbinary (256))
        EXEC sp_hexadecimal @PWD_varbinary, @PWD_string OUT
        EXEC sp_hexadecimal @SID_varbinary,@SID_string OUT

        -- obtain password policy state
        SELECT @is_policy_checked = CASE is_policy_checked WHEN 1 THEN 'ON' WHEN 0 THEN 'OFF' ELSE NULL END FROM sys.sql_logins WHERE name = @name
        SELECT @is_expiration_checked = CASE is_expiration_checked WHEN 1 THEN 'ON' WHEN 0 THEN 'OFF' ELSE NULL END FROM sys.sql_logins WHERE name = @name

        SET @tmpstr = 'CREATE LOGIN ' + QUOTENAME( @name ) + ' WITH PASSWORD = ' + @PWD_string + ' HASHED, SID = ' + @SID_string + ', DEFAULT_DATABASE = [' + @defaultdb + ']'

        IF ( @is_policy_checked IS NOT NULL )
        BEGIN
        SET @tmpstr = @tmpstr + ', CHECK_POLICY = ' + @is_policy_checked
        END
        IF ( @is_expiration_checked IS NOT NULL )
        BEGIN
        SET @tmpstr = @tmpstr + ', CHECK_EXPIRATION = ' + @is_expiration_checked
        END
        END
        IF (@denylogin = 1)
        BEGIN -- login is denied access
        SET @tmpstr = @tmpstr + '; DENY CONNECT SQL TO ' + QUOTENAME( @name )
        END
        ELSE IF (@hasaccess = 0)
        BEGIN -- login exists but does not have access
        SET @tmpstr = @tmpstr + '; REVOKE CONNECT SQL TO ' + QUOTENAME( @name )
        END
        IF (@is_disabled = 1)
        BEGIN -- login is disabled
        SET @tmpstr = @tmpstr + '; ALTER LOGIN ' + QUOTENAME( @name ) + ' DISABLE'
        END
        PRINT @tmpstr
        END

        FETCH NEXT FROM login_curs INTO @SID_varbinary, @name, @type, @is_disabled, @defaultdb, @hasaccess, @denylogin
        END
        CLOSE login_curs
        DEALLOCATE login_curs
        RETURN 0
        GO

2. SSMS中选择以文本格式显示结果并执行上面的存储过程

        EXEC sp_help_revlogin

3. 将文本框内输出的语句复制到备库执行

        CREATE LOGIN [sasa] WITH PASSWORD = 0x0200A34FF3AEA1C67E7F7EC5F436607FAF1494F27E26F8B34356A20AA2E2419FB5E21AFC17551805C3822697501783CD2C22206087F5C2594AFB6CEF515D7AD48898B0ACF772 HASHED, SID = 0x5459C58748D86349800D246DA21AF276, DEFAULT_DATABASE = [master], CHECK_POLICY = OFF, CHECK_EXPIRATION = OFF

4. 在主库给账号赋权





**参考连接:**

[在登录实例之间传输登录SQL Server](https://docs.microsoft.com/zh-CN/troubleshoot/sql/security/transfer-logins-passwords-between-instances)
