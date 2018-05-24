# Oracle基本操作（登陆、用户、表空间、exp/imp、权限）
## 1、登陆 (在windows上CMD下执行)
### 1.1、登陆sys帐户
```sql
SQLPLUS sys AS SYSDBA
```
### 1.2、登陆普通用户
```sql
SQLPLUS 用户名/密码
```
## 2、创建用户一般分为四步
### 2.1、创建临时表空间
```sql
CREATE TEMPORARY TABLESPACE --表示创建的是临时表空间
	SYNCHROMOBILE_TEMP --表空间名称
	TEMPFILE 'D:\ORACLE\ORADATA\ORCL\SYNCHROMOBILE_TEMP.DBF' --表空间文件（以下简称“文件”）存放位置
	SIZE 50M --文件的初始大小
	AUTOEXTEND ON --文件大小是否自动扩展（NO:是）
	NEXT 50M --文件每次扩展大小
	MAXSIZE 500M --文件大小上限
	EXTENT MANAGEMENT LOCAL; --分区分配方案
```
>注释：创建用户之前要先创建`临时表空间`，若不创建则默认的临时表空间为`TEMP`。
### 2.2、创建（数据）表空间
```sql
CREATE TABLESPACE --表示创建正式表空间
	SYNCHROMOBILE 
	DATAFILE 'D:\ORACLE\ORADATA\ORCL\SYNCHROMOBILE.DBF' 
	SIZE 50M 
	AUTOEXTEND ON 
	NEXT 50M 
	MAXSIZE 500M 
	EXTENT MANAGEMENT LOCAL;
```
>注释：创建用户之前要先创建`数据表空间`，若不创建则默认的数据表空间是`system`。
### 2.3、创建用户并指定表空间
```sql
CREATE USER synchromobile --用户名
	IDENTIFIED BY synchromobile --密码
	ACCOUNT UNLOCK --解锁用户
	DEFAULT TABLESPACE SYNCHROMOBILE --指定默认表空间
	TEMPORARY TABLESPACE SYNCHROMOBILE_TEMP; --指定临时表空间
```
### 2.4、给用户授予权限
```sql
GRANT CONNECT,RESOURCE TO synchromobile;
```
## 3、用户相关操作
### 3.1、查找所有用户名
```sql
SELECT USER * FROM DBA_USERS;
```
### 3.2、修改密码
```sql
ALTER USER 用户名 IDENTIFIED BY 新密码;
```
### 3.3、撤销权限
```sql
REVOKE CONNECT, RESOURCE FROM 用户名;
```
### 3.4、删除用户
```sql
DROP USER synchromobile CASCADE;
```
>注释：`CASCADE`表示级联关系也删除掉。
## 4、表空间相关操作
### 4.1、查找所有表空间路径
```sql
SELECT * FROM DBA_DATA_FILES;
```
### 4.2、删除表空间，`CASCADE CONSTRAINT`表示级联关系也删除掉
```sql
DROP TABLESPACE SYNCHROMOBILE INCLUDING CONTENTS AND DATAFILES CASCADE CONSTRAINT;
```
## 5、使用`exp/imp`命令导入/导出`dmp`文件
### 5.1、exp命令 (windows上在CMD下执行)
#### a.将用户表导出到指定路径 D 盘
```sql
exp username/password@ORCL file=d:\testdata.dmp full=y
```
#### b.将用户`system`与`sys`用户的表导出到指定路径 D 盘
```sql
exp system/password@ORCL file=d:\testdata.dmp owner=(system,sys)
```
#### c.将用户中的表 table_A、table_B 导出到指定路径 D 盘
```sql
exp username/password@ORCL file=d:\testdata.dmp tables=(table_A,table_B)
```
#### d.将用户中的表 table1 中的字段 filed1 以"00"打头的数据导出
```sql
exp username/passwor@ORCL filed=d:\testdata.dmp tables=(table1) query=/" where filed1 like '00%'/"
```
>注释：对于压缩可以用`winzip`进行压缩，也可以在上面命令后面加上`compress=y`来实现。
### 5.2、imp命令 (windows上在CMD下执行)
#### a.将 d:/testdata.dmp 中的数据导入数据库中
```sql
imp username/password@ORCL full=y file=d:\testdata.dmp
```
#### b.将 d:\testdata.dmp 中的表table1 导入
```sql
imp username/password@ORCL file=d:\testdata.dmp tables=(table1)
```
#### c.将其他用户的对象导入当前用户
```sql
imp username/password file=d:\testdata.dmp fromuser=user1
imp username/password file=d:\testdata.dmp fromuser=(user1,user2)
```
#### d.将其他用户的对象导入到指定用户
```sql
imp tomcepsp/tomcepsp@ORCL file=d:\Data\jxjy20180522.dmp fromuser=jxjy touser=tomcepsp
imp tomcepsp/tomcepsp file=d:\Data\jxjy20180522.dmp fromuser=(user1,user2) touser=(user3,user4)
```
>注释：以上命令如果出现问题，假设有的表已存在，对该表可以不进行导入，后面添加`ignore=y`。
## 6、权限分类
	DBA：拥有全部特权，是系统最高权限，只有 DBA 才可以创建数据库结构；
	CONNECT：拥有 CONNECT 权限的用户只可以登录 Oracle ，不可以创建实体，不可以创建数据库结构；
	RESOURCE：拥有RESOURCE权限的用户只可以创建实体，不可以创建数据库结构；
	系统权限只能由DBA用户授出：sys, system（最开始只能是这两个用户）；
	对于普通用户：授予 CONNECT , RESOURCE 权限；
	对于DBA管理用户：授予 CONNECT，RESOURCE，DBA 权限。
