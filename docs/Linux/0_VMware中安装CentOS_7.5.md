## 新建虚拟机

创建新的虚拟机

经典

安装程序光盘映像文件（iso）

虚拟机名称：BD100

位置：%Documents%\Virtual Machines\BD100

处理器：2*2

内存：4096

网络：NAT

IO控制器：LSI Logic（默认）

创建新的虚拟磁盘

磁盘类型：SCSI（默认）

磁盘空间：50G

存储为单个文件

检查一遍：

![图 1](https://cdn.jsdelivr.net/gh/Z-404/imageHost@main/2023/01/MI_20230103_1672733304784.png)  

## 安装 CentOS

确认 ISO 映像文件

开启虚拟机

Install CentOS 7

进入图形化界面：

语言：中文简体

分区：自动分区

KDUMP：禁用

网络和主机名

开始安装

设置 root 密码

查看ip：`ip addr show`


## 网卡、主机名、hosts

1. 配置网卡

    ```shell
    vi /etc/sysconfig/network-scripts/ifcfg-ens33
    ```

    ```shell
    # 静态地址
    BOOTPROTO="static"
    # IP 地址
    IPADDR="192.168.218.100"
    # 子网掩码
    PREFIX=24
    # 网关
    GATEWAY="192.168.218.2"
    # DNS
    DNS1="192.168.218.2"
    ```

    重启生效：

    ```shell
    sudo systemctl restart network
    ```

2. 主机名

    ```shell
    hostname

    hostnamectl --static set-hostname bd100
    ```

3. hosts文件

    ```shell
    vim /etc/hosts

    # 添加
    192.168.218.100 bd100
    192.168.218.102 bd102
    192.168.218.103 bd103
    192.168.218.104 bd104
    ```