vim /etc/security/limits.d/20-nproc.conf 

```shell
修改如下内容：
* soft nproc 1024
#修改为
* soft nproc 2048
```

vim /etc/sysctl.conf  

```shell
vm.max_map_count=655360
```

vim /etc/security/limits.conf

```shell
* soft nofile 65536
* hard nofile 65536
* soft nproc 65536
* hard nproc 65536
```

