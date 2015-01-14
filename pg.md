---
output: word_document
---

说明：试验设备监控项目中将使用PostgreSQL 9.3

# PostgreSQL 安装及使用

PostgreSQL可以从使用YUM管理的二进制包和源代码包来安装，这种安装方式支持以下Linux发行版（32位和64位，当前版本和之前版本）

- Fedora
- Red Hat Enterprise Linux
- CentOS
- Scientific Linux
- Oracle Enterprise Linux

查看主仓库链接 [http://yum.postgresql.org/](http://yum.postgresql.org/)

## 介绍

配置YUM源

查找并且编辑正在使用的发行版的repo文件，详细定位如下：

Fedora :/etc/yum.repos.d/fedora.repo and /etc/yum.repos.d/fedora-updates.repo，[Fedora]部分

Centos: /etc/yum.repos.d/CentOS-Base.repo,[base]与[updates]部分

Red Hat：/etc/yum/pluginconf.d/rhnplugin.conf，[main]部分
找到这些章节，增加这样一行（否则会引起基础库依赖问题）

	exclude=postgresql*

安装PGDG RPM 文件

对于每一个<发行版、系统架构、数据库版本>的组合，都存在一个对应的PGDGROM文件，浏览http://yum.postgresql.org/查找适合适用于你当前系统的对应RPM文件。比如，在CentOS6 64位上安装PostgreSQL 9.3，就执行

	yum localinstall http://yum.postgresql.org/9.3/redhat/rhel-6-x86_64/pgdg-centos93-9.3-1.noarch.rpm

安装PostgreSQL

列出所有可用的包

	yum list postgres*

要安装PostgreSQL 9.3的基础服务

	yum install postgresql93-server

可以根据需要来安装其他的包。

安装完毕之后的命令

安装完软件包之后，需要对数据库进行配置和初始化，才能使用。在命令行下name的值随着你使用的PostgreSql的版本而变化。在PostgreSQL9.0之后name包含主版本.次版本号，例如PostgreSQL-9.3。在8.x版本下，name的值则保持为postgresql(意味着没有版本号)

## 数据目录

PostgreSQL的数据目录包含了数据库的所有文件。环境变量PGDATA的值经常指向这个目录。
在PostgreSQL9.0及以上版本，默认的数据目录是

	/var/lib/pgsql/<name>/data

比如

/var/lib/pgsql/9.3/data

7.x和8.x版本下，默认的目录是：

/var/lib/pgsql/data/

初始化

安全完成后的第一条命令是在PGDATA中初始化数据库

	service <name> initdb

比如

service postgresql-9.3 initdb

如果以上命令不生效，就尝试直接调用二进制文件来初始化：

/usr/pgsql-y.x/bin/postgresqlyx-setup initdb

比如 在9.3版本下

/usr/pgsql-9.3/bin/postgresql93-setup initdb

开机启动

如果想让PostgreSQL随操作系统启动，执行如下命令：

	chkconfig <name> on

用9.3举例：

chkconfig postgresql-9.3 on

控制服务

要控制数据库服务，使用：

	service <name> <command>

这里的commmand可以是

start,启动数据库
stop,停止数据库
restart,停止/启动数据库，经常在改变核心配置后使用
reload,重新载入pg_hba.conf并保持数据库运行

依然以9.3举例：

	service postgresql-9.3 start

删除

删除所有

	yum erase postgresql93*

或者使用同样的命令删除其他的包。

## 安装以后

首次设置PostgreSQL


# 添加新用户和新数据库

初次安装后，默认生成一个名为postgres的数据库和一个名为postgres的数据库用户。这里需要注意的是，同时还生成了一个名为postgres的Linux系统用户。

下面，使用postgres用户来生成其他用户和新数据库。几种方法可以达到这个目的，本文介绍两种。

## 方法一，使用PostgreSQL控制台。

首先，新建一个Linux新用户，可以取想要的名字，本文为dbuser。

	sudo adduser dbuser

然后，切换到postgres用户。

	sudo su - postgres

下一步，使用psql命令登录PostgreSQL控制台。

	psql

这时相当于系统用户postgres以同名数据库用户的身份，登录数据库，这是不用输入密码的。如果一切正常，系统提示符会变为"postgres=#"，表示这时已经进入了数据库控制台。以下的命令都在控制台内完成。

第一步，使用\password命令，为postgres用户设置一个密码。

	\password postgres

第二步，创建数据库用户dbuser（刚才创建的是Linux系统用户），并设置密码。

	CREATE USER dbuser WITH PASSWORD 'password';

第三步，创建用户数据库，这里为exampledb，并指定所有者为dbuser。

	CREATE DATABASE exampledb OWNER dbuser;

第四步，将exampledb数据库的所有权限都赋予dbuser，否则dbuser只能登录控制台，没有任何数据库操作权限。

	GRANT ALL PRIVILEGES ON DATABASE exampledb to dbuser;

最后，使用\q命令退出控制台（也可以直接按ctrl+D）。

	\q

## 方法二，使用shell命令行。

添加新用户和新数据库，除了在PostgreSQL控制台内，还可以在shell命令行下完成。这是因为PostgreSQL提供了命令行程序createuser和createdb。还是以新建用户dbuser和数据库exampledb为例。

首先，创建数据库用户dbuser，并指定其为超级用户。

	sudo -u postgres createuser --superuser dbuser

然后，登录数据库控制台，设置dbuser用户的密码，完成后退出控制台。

	sudo -u postgres psql
	\password dbuser
	\q

接着，在shell命令行下，创建数据库exampledb，并指定所有者为dbuser。

	sudo -u postgres createdb -O dbuser exampledb

## 方法二，使用pgAdmin III

yum install pgadmin3

远程连接是需要配置

/etc/postgresql/9.3/main/pg_hba.conf

# 登录数据库

添加新用户和新数据库以后，就要以新用户的名义登录数据库，这时使用的是psql命令。

	psql -U dbuser -d exampledb -h 127.0.0.1 -p 5432

上面命令的参数含义如下：-U指定用户，-d指定数据库，-h指定服务器，-p指定端口。

输入上面命令以后，系统会提示输入dbuser用户的密码。输入正确，就可以登录控制台了。

psql命令存在简写形式。如果当前Linux系统用户，同时也是PostgreSQL用户，则可以省略用户名（-U参数的部分）。举例来说，我的Linux系统用户名为ruanyf，且PostgreSQL数据库存在同名用户，则我以ruanyf身份登录Linux系统后，可以直接使用下面的命令登录数据库，且不需要密码。

	psql exampledb

此时，如果PostgreSQL内部还存在与当前系统用户同名的数据库，则连数据库名都可以省略。比如，假定存在一个叫做ruanyf的数据库，则直接键入psql就可以登录该数据库。

	psql

另外，如果要恢复外部数据，可以使用下面的命令。

	psql exampledb < exampledb.sql

# 控制台命令

除了前面已经用到的\password命令（设置密码）和\q命令（退出）以外，控制台还提供一系列其他命令。

	- \h：查看SQL命令的解释，比如\h select。
	- \?：查看psql命令列表。
	- \l：列出所有数据库。
	- \c [database_name]：连接其他数据库。
	- \d：列出当前数据库的所有表格。
	- \d [table_name]：列出某一张表格的结构。
	- \du：列出所有用户。
	- \e：打开文本编辑器。
	- \conninfo：列出当前数据库和连接的信息。

# 数据库操作

基本的数据库操作，就是使用一般的SQL语言。

	# 创建新表 
	CREATE TABLE user_tbl(name VARCHAR(20), signup_date DATE);
	# 插入数据 
	INSERT INTO user_tbl(name, signup_date) VALUES('张三', '2013-12-22');
	# 选择记录 
	SELECT * FROM user_tbl;
	# 更新数据 
	UPDATE user_tbl set name = '李四' WHERE name = '张三';
	# 删除记录 
	DELETE FROM user_tbl WHERE name = '李四' ;
	# 添加栏位 
	ALTER TABLE user_tbl ADD email VARCHAR(40);
	# 更新结构 
	ALTER TABLE user_tbl ALTER COLUMN signup_date SET NOT NULL;
	# 更名栏位 
	ALTER TABLE user_tbl RENAME COLUMN signup_date TO signup;
	# 删除栏位 
	ALTER TABLE user_tbl DROP COLUMN email;
	# 表格更名 
	ALTER TABLE user_tbl RENAME TO backup_tbl;
	# 删除表格 
	DROP TABLE IF EXISTS backup_tbl;
	 

在数据库postgres中创建名为test的schema

 	postgres=# CREATE SCHEMA test;

创建角色(用户)，设置密码

 	postgres=# CREATE USER xxx PASSWORD 'yyy';

为新角色授权

 	postgres=# GRANT ALL ON SCHEMA test TO xxx;

将新Schema中的所有表授权给用户

	postgres=# GRANT ALL ON ALL TABLES IN SCHEMA test TO xxx;

断开连接

	 postgres=# \q

在schema test中创建表test

	 postgres=>CREATE TABLE test.test (coltest varchar(20));
	 CREATE TABLE

新表中插入单条数据

	 postgres=>insert into test.test (coltest) values ('It works!');
	 INSERT 0 1

从表中SELECT

	 postgres=>SELECT * from test.test;
	   coltest  
	 -----------
	  It works!
	 (1 row)

删除test表

	 postgresql=>DROP TABLE test.test;
	 DROP TABLE


更多内容请参见 [英文手册](http://www.postgresql.org/docs/manuals/)

