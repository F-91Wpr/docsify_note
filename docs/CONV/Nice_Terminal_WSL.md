# 在 WSL with Ubuntu 中配置 pwsh 遇到到问题

## WSL2 无法连接宿主机

在 Windows 中添加一个防火墙规则:

```shell
PS C:\WINDOWS\system32>  New-NetFirewallRule -DisplayName "WSL" -Direction Inbound  -InterfaceAlias "vEthernet (WSL)"  -Action Allow
```

## WSL 配置代理

```shell
export hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*')
export https_proxy="http://${hostip}:10080";
export http_proxy="http://${hostip}:10080";
```

## 安装 Oh My Posh

```shell
# 安装 oh-my-posh
sudo wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/posh-linux-amd64 -O /usr/local/bin/oh-my-posh
sudo chmod +x /usr/local/bin/oh-my-posh

sudo apt install unzip

# 下载主题
mkdir ~/.poshthemes
wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/themes.zip -O ~/.poshthemes/themes.zip
unzip ~/.poshthemes/themes.zip -d ~/.poshthemes
chmod u+rw ~/.poshthemes/*.omp.*
rm ~/.poshthemes/themes.zip
```

## 更改默认 shell

1. 显示当前 shell：

    ```shell
    echo $SHELL
    ```
    
2. 显示已有的 shell:

    ```shell
    cat /etc/shells
    ```

3. 取代bash，设为默认shell

    ```shell
    chsh -s /opt/microsoft/powershell/7/pwsh
    ```

## 改回默认 /usr/bin/bash 不生效

`.bashrc`被 `.bash_profile` 覆盖

删除`.bash_profile`