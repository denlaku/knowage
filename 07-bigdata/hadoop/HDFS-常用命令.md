## HDFS常用命令

#### 查看目录的文件

```shell
hdfs dfs -ls /
```

#### 查看目录下的所有文件

```shell
hdfs dfs -ls -R /aa
```

#### 创建文件夹

```shell
hdfs dfs -mkdir aa
```

#### 级联创建文件夹

```shell
hdfs dfs -mkdir -p /aa/bb/cc
```

### 上传文件

```shell
hdfs dfs -put ~/aa.txt /my/upload/
# 或者
hdfs dfs -copyFromLocal ~/aa.txt /my/upload/
```

#### 下载文件

```shell
hdfs dfs -get /my/upload/aa.txt ~/download/aa.txt
# 或者
hdfs dfs -copyToLocal /my/upload/aa.txt ~/download/aa.txt
```

#### 合并下载

```shell
hdfs dfs -getmerge /my/upload/aa.txt /my/upload/bb.txt ~/cc.txt
```

#### 复制

```shell
hdfs dfs -cp /my/upload/aa.txt /my/upload/aa.copy.txt
```

#### 移动

```shell
hdfs dfs -mv /my/upload/aa.txt /my/bak/
```

#### 删除文件或文件夹

```shell
# 删除文件
hdfs dfs -rm /my/upload/aa.txt
# 删除文件夹
hdfs dfs -rm -p -r /my/
```

#### 追加文件

```shell
hdfs dfs -appendToFile ~/dd.txt /my/upload/aa.txt
```

#### 查看文件内容

```shell
hdfs dfs -cat /my/upload/aa.txt
```

#### 改变文件所属的组

```shell
hdfs dfs -chgrp [-R] GROUP URI [URI …]
```

#### 改变文件的权限

```shell
hdfs dfs -chmod 777 /my/upload/aa.txt
```

#### 改变文件的拥有者

```shell
hdfs dfs -chown [-R] [OWNER][:[GROUP]] URI [URI ]
```

#### 显示目录中所有文件的大小

```shell
hdfs dfs -du /
```

#### 显示HDFS系统存储大小

```shell
hdfs dfs -df -h
```

#### 查看集群状态

```shell
hdfs dfsadmin -report
```

#### 文件健康状态检查

```shell
hdfs fsck /
```

