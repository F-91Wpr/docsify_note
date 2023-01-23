## readline

安装：作为 bash 依赖默认安装

[快捷键](https://catonmat.net/ftp/readline-emacs-editing-mode-cheat-sheet.pdf)：

   | Key | Func             |
   | --- | ---------------- |
   | C+a | 移动到行首       |
   | C+e | 移动到行尾       |
   | C+u | 删除光标到行首   |
   | C+k | 删除光标到行尾     |
   | M+# | 注释行并送进历史 |

`C-Ctrl`
`M-Alt`

```shell
vim ~/.inputrc
```

绑定键位

```shell
# M+k:删除整行
"\ek":kill-whole-line
```

使用方向键逐字移动：

```shell
"\e[1;5D": backward-word
"\e[1;5C": forward-word
```

历史搜索功能（小爽）：

```shell
"\e[A": history-search-backward
"\e[B": history-search-forward
```

## htop

![图 10](https://cdn.jsdelivr.net/gh/Z-404/imageHost@main/2023/01/MI_20230107_1673026748930.png)  

## yt-dlp

## ffmpeg

## cmatrix

```shell
sudo apt install cmatrix
cmatrix 
   [-s: Screensaver mode] 
   [-r: rainbow mode]
```

![cmatrix](https://cdn.jsdelivr.net/gh/Z-404/imageHost@main/2023/01/MI_20230105_1672929679268.png)  
             
## screenfetch

```shell
sudo apt install screenfetch
screenfetch
```

![screenfetch](https://cdn.jsdelivr.net/gh/Z-404/imageHost@main/2023/01/MI_20230105_1672928966201.png)  



