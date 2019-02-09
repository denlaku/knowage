## Hbase的shell操作

```shell
# version返回hbase版本信息
version
# status返回hbase集群的状态信息
status
# exit退出hbase shell
exit
# 关闭hbase集群（与exit不同）
shutdown
# ------------------------------------------------------------------------------------
# help查看某个命令的帮助信息
help 'get' # 获取get命令的帮助信息
# 显示所有的表
list
list 'user'
# create 创建表
create 'user','info','job'
# truncate 重新创建指定表,表里的数据会消失
truncate 'test'
# describe 显示表相关的详细信息
describe 'test'
desc 'test'
# disable 使表无效
disable 'test'
# enable 使表有效
enable 'test'
# drop 删除表（删除表之前必须使用disable使表失效）
drop 'test'
# exists 测试表是否存在
exists 'test'
# alter 修改列族（column family）模式
# 添加新的列族info
alter 'test','info'
alter 'test', NAME => 'info'
# 删除指定的列族info
alter 'test', NAME => 'info', METHOD => 'delete' 
alter 'test','delete'=>'info'
# 设置表为只读
alter 'test', READONLY
# ------------------------------------------------------------------------------------
# 直接把表赋值个某个变量，如
t=create 'user','name','age' # 创建表时赋值
t=get_table 'user' # 获取表并赋值
# scan 查看表的所有数据
scan 'test'
# count 统计表中行的数量
count 'test
# put 向指向的表单元添加值
put '表名','行名','列名','值'
put 'test','row1','name','tom'
# get 查看记录
get '表名','行名' # 获取一行
get 'test','row1'
get '表名','行名','列名' # 获取某一行中的一列
get 'test','row1','name'
# delete 删除指定列的值
delete 'test','row1','name'
# deleteall删除指定行的所有元素值
deleteall 'users','row1'
# incr 增加指定表行或列的值 (HBase计数器)
incr 'users','row1','name:info'
```

