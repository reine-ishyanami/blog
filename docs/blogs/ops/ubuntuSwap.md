---
title: Ubuntu 实例中添加 swap 分区的方法
date: 2024/4/13
tags:
 - Linux
---

# Ubuntu 实例中添加 swap 分区的方法

1. 依次执行一下命令，创建一个空文件，锁定文件的大小

	> 说明：文件的具体大小建议设定为内存的两倍。此处的1K×4M＝4GiB。
	
	```shell
	sudo mkdir -v /var/cache/swap
	cd /var/cache/swap
	sudo dd if=/dev/zero of=swapfile bs=1K count=4M
	```

2. 执行以下命令，将新建的文件转换为 swap 文件

	```shell
	sudo mkswap swapfile
	```
   
3. 执行以下命令，给文件授权

	```shell
	sudo chmod 600 swapfile
	```

4. 执行以下命令，启用 swap 分区
	    
	```shell
	sudo swapon swapfile
	```

5. 执行以下命令，进行验证

	```shell
	swapon -s
	top -bn1 | grep -i swap
	```
	
	输出如下
	
	> KiB Swap: 4194300 total, 4194300 free  

6. 执行以下命令，将该分区设置成开机自启
    
	```shell
	echo "/var/cache/swap/swapfile none swap sw 0 0" | sudo tee -a /etc/fstab
	```

7. 执行以下命令，测试开机是否加载 swap 分区

	```shell
	sudo swapoff swapfile 
	sudo swapon -va
	```

8. 如果您需要修改 swap 使用率，编辑 `/etc/sysctl.conf` 文件，修改 vm.swappiness 值
       
	```shell
	vm.swappiness=[$NUM]
	```
    
	> 说明：$NUM 的取值范围为 0-100，0 表示最大限度使用物理内存，100 表示积极的使用 swap 分区。