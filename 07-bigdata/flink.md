1、下载

2、解压
tar -xvf flink-1.6.3-bin-scala_2.11.tgz

3、配置

conf/flink-conf.yaml文件
jobmanager.rpc.address: 10.10.10.11

slaves文件:
​	10.10.10.12
​	10.10.10.13

4、启动集群
./bin/start-cluster.sh

5、运行例子
开启netcat：nc -l 9000
提交程序：./bin/flink run examples/streaming/SocketWindowWordCount.jar --port 9000 --hostname H02

查看输出：tail -f log/flink-*-taskexecutor-*.out

web界面：http://10.10.10.11:8081

6、停止集群
./bin/stop-cluster.sh