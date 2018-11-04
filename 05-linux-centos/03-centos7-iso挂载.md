```shell
# 备份原有的repo文件
mkdir /tmp/repodir
mv /etc/yum.repos.d/* /tmp/repodir

#创建新yum源
cd /etc/yum.repos.d/
vi ops.repo 
# ops.repo文件内容：
[cdrom]
name=local-vcd
baseurl=file:///mnt/cdrom
enabled=1
gpgcheck=0

# 创建挂载目录
mkdir /mnt/cdrom

# 挂载
mount /dev/cdrom /mnt/cdrom
#以只读方式挂载
mount -a

# 设置开机自动挂载
vi /etc/fstab 
	#最后一行添加
	/dev/cdrom /mnt/cdrom iso9660 defaults 0 0

# 查看挂载目录
ll /mnt/cdrom/

# 清除缓存
yum clean all
# 查看yum源
yum repolist

# 安装常用软件
yum install gcc-c++ wget net-tools vim lrzsz tree screen lsof tcpdump -y

```

