#### **安装前准备**

2套CentOS7环境
`10.10.10.21`
`10.10.10.22`
安装包：
` mysql-5.7.23-el7-x86_64.tar.g`

#### **上传安装包至目录`/usr/local/`并解压**

```shell
cd /usr/local/
tar -xvf mysql-5.7.23-el7-x86_64.tar.g
mv mysql-5.7.23-el7-x86_64 mysql
```

#### **创建mysql用户组及用户，并授权**

```shell
groupadd mysql
useradd -g mysql mysql
chown -R mysql /usr/local/mysql
chgrp -R mysql /usr/local/mysql
```

#### **创建配置文件`my.cnf`并添加如下配置**

```shell
[client]
port=3306
socket=/tmp/mysql.sock

[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
socket=/tmp/mysql.sock
log-error=/var/log/mysqld.log
#pid-file=/var/run/mysqld/mysqld.pid
#不区分大小写
lower_case_table_names = 1
```

#### **初始化数据库**

```shell
#手动编辑一下日志文件，什么也不用写，直接保存退出
cd /var/log/
vim mysqld.log

chmod 777 mysqld.log
chown mysql:mysql mysqld.log

/usr/local/mysql/bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --lc_messages_dir=/usr/local/mysql/share --lc_messages=en_US
```

#### **查看初始密码**

```shell
cat /var/log/mysqld.log
#密码在文件的最后一行
#[Note] A temporary password is generated for root@localhost: I3U#HIqPu7yv
#这安装后的初始密码是：I3U#HIqPu7yv

# 或者用如下命令查看密码
grep "password" /var/log/mysqld.log
```

#### **启动mysql服务，修改初始密码**

```shell
/usr/local/mysql/support-files/mysql.server start
/usr/local/mysql/bin/mysql -uroot -p
#输入密码 I3U#HIqPu7yv
set password=password('new_pwd');
grant all privileges on *.* to 'root'@'%' identified by 'new_pwd';
flush privileges;
```

#### **设置开机自启**

```shell
cd /usr/local/mysql/support-files
cp mysql.server /etc/init.d/mysqld
chkconfig --add mysqld
```

ln -s /usr/local/mysql/bin/mysql /usr/bin

至此单机安装已经完成。

------

#### 主从复制

主库IP：`10.10.10.21`

从库IP：`10.10.10.22`

主库创建数据同步用户并授权

```sql
create user repli identified by 'new_password';
grant replication slave on *.* to 'repli'@'%' identified by 'new_password'; 
```

编辑主库my.cnf文件，添加如下内容

```shell
[mysqld]
log-bin=mysql-bin # 开启bin日志
server-id=21 #mysql服务ID(必须唯一)
```

编辑从库my.cnf文件，添加如下内容

```shell
[mysqld]
server-id=21 #mysql服务ID(必须唯一)
```

获取主库binlog文件名称及pos位置信息，执行 `show master status;`

| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
| ---------------- | -------- | ------------ | ---------------- | ----------------- |
| mysql-bin.000002 | 696      |              |                  |                   |

在从节点配置访问主节点的参数信息

```shell
change master to 
master_host='10.10.10.21',
master_user='repli',
master_password='denlaku', 
master_log_file='mysql-bin.000002',
master_log_pos=696;
```

启动从库复制，执行 `start slave;`

停止从库复制，执行  `stop slave;`

