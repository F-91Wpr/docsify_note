## 





## 常用脚本

### 设置 IP 和主机名

```shell
#!/bin/bash

#改IP
sed -i "/IPADDR/s/.*/IPADDR=\"192.168.218.$1\"/" /etc/sysconfig/network-scripts/ifcfg-ens33

#改Hostname
hostnamectl --static set-hostname BD$1

#重启网络
systemctl restart network
```

### 集群同步脚本`xsync`：

```shell
#!/bin/bash

#1. 判断参数个数
if [ $# -lt 1 ]
then
    echo Not Enough Arguement!
    exit;
fi

#2. 遍历集群所有机器
for host in bd102 bd103 bd104
do
    echo ====================  $host  ====================
    #3. 遍历所有目录，挨个发送

    for file in $@
    do
        #4. 判断文件是否存在
        if [ -e $file ]
            then
                #5. 获取父目录
                pdir=$(cd -P $(dirname $file); pwd)

                #6. 获取当前文件的名称
                fname=$(basename $file)
                ssh $host "mkdir -p $pdir"
                rsync -av $pdir/$fname $host:$pdir
            else
                echo $file does not exists!
        fi
    done
done
```

```shell
# 添加执行属性
chmod +x xsync
# 拷贝到 /bin 目录下
sudo cp ./xsync /bin
# 同步到集群
sudo xsync /bin/xsync
```

### `jpsall`

```shell
#!/bin/bash

for host in bd102 bd103 bd104
do
        echo ================  $host  ================
        ssh $host jps | grep -v Jps
done
```

### 分发命令`xcall`

```shell
#!/bin/bash
pdsh -w "bd102,bd103,bd104" "$*" | awk -F : '{host=$1;$1=null;array[host]=array[host]"\n"$0}END{for (i in array) {print "========  "i"  ========"array[i]}}'
```