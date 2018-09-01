vi /etc/security/limits.d/90-nproc.conf 

```
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
```

vi /etc/security/limits.d/90-nproc.conf 

```
修改如下内容：
* soft nproc 1024
#修改为
* soft nproc 2048
```

vi /etc/sysctl.conf  

```
vm.max_map_count=655360
```