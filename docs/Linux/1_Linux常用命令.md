## 常用命令

ps -ef
top
df -h
netstat -tunlp

## 常用工具

grep
sed
awk

cut
sort
## ssh 密钥

```bash
# 生成密钥
ssh-keygen -t rsa   # 三次回车

# 发送公钥
ssh-copy-id hostname
```

## 远程复制

1. scp -r 
   
2. rsync -av

3. xsync 脚本

    ```shell
    #!/bin/bash
   
    #1. 判断参数个数
    if [ $# -lt 1 ]
    then
        echo Not Enough Arguement!
        exit;
    fi
   
    #2. 遍历集群所有机器
    for host in hadoop102 hadoop103 hadoop104
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