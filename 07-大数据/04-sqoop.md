```shell
#

# 测试连接是否正常
./sqoop list-databases --connect jdbc:mysql://10.10.10.2:3306 --username root --password denlaku
#
./sqoop help import
# 导入数据至hdfs
./sqoop import -m 1 --connect jdbc:mysql://10.10.10.2:3306/sqoop --username root --password denlaku  --table t_user --target-dir /sqoop/user

# 导入数据至数据库表
./sqoop export -m 1 --connect jdbc:mysql://10.10.10.2:3306/sqoop --username root --password denlaku --table t_user --export-dir /input/sqoop/sqoop.txt --fields-terminated-by ' '
```

