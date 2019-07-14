通过help命令获取帮助信息

```
ZooKeeper -server host:port cmd args
	stat path [watch]
	set path data [version]
	ls path [watch]
	delquota [-n|-b] path
	ls2 path [watch]
	setAcl path acl
	setquota -n|-b val path 
	history 
	redo cmdno
	printwatches on|off
	delete path [version]
	sync path
	listquota path
	rmr path
	get path [watch]
	create [-s] [-e] path data acl
	addauth scheme auth
	quit 
	getAcl path
	close 
	connect host:port
```



```shell
# 显示指定目录下所有节点
ls /
ls /test
# 创建节点: create /节点名称 数据
create /test 1
# 验证，相当于输入用户名和密码
addauth digest user:pswd
# 设置权限
setAcl /path auth:用户名:密码明文:权限
setAcl /path digest:用户名:密码密文:权限
# 设置配额
setquota -n 5 /test 
```

**4种身份认证方式：**

world：默认方式，相当于全世界都能访问
auth：代表已经认证通过的用户(cli中可以通过addauth digest user:pwd 来添加当前上下文中的授权用户)
digest：即用户名:密码这种方式认证，这也是业务系统中最常用的
ip：使用Ip地址认证

使用密文：setAcl path digest|ip:username:password:c|d|r|w|a

使用明文：

   addauth digest username:password

   setAcl /path auth:username:password:cdrwa

密文命令（执行目录在zk根目录）：

java -cp ./zookeeper-3.4.6.jar:./lib/log4j-1.2.16.jar:./lib/slf4j-api-1.6.1.jar:./lib/slf4j-log4j12-1.6.1.jar org.apache.zookeeper.server.auth.DigestAuthenticationProvider user1:12345

**五种权限：**

CREATE、READ、WRITE、DELETE、ADMIN 也就是 增、删、改、查、管理权限，这5种权限简写为crwda，这5种权限中，delete是指对子节点的删除权限，其它4种权限指对自身节点的操作权限