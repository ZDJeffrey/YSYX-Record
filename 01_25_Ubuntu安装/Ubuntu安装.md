# Ubuntu安装
## 系统安装
根据博客安装Ubuntu系统，[Ubuntu 22.04 双系统安装和卸载](https://blog.csdn.net/m0_63478913/article/details/125352819)
其中为Swap分配20G，为`/`分配180G。

## 更换清华源
修改`/etc/apt/sources.list`文件，进行如下配置：
```bash
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
# deb-src http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```

## Clash
安装[clash-verge](https://github.com/zzzgydi/clash-verge)，导入配置文件，设置开机启动。

## 输入法
根据搜狗[官方教程](https://shurufa.sogou.com/linux/guide)安装搜狗输入法。

## 中文字体


## Grub默认启动项
安装双系统后，Grub默认启动项为Ubuntu，修改默认启动项为Windows。
修改`/etc/default/grub`文件，将`GRUB_DEFAULT`修改为windows对应启动项编号。同时可以修改`GRUB_TIMEOUT`设置选择时间。保存后执行`sudo update-grub`更新。

## Ubuntu关机停止服务时间
Ubuntu关机时，会等待服务停止后再关机，若超时则强制关机。修改`/etc/systemd/system.conf`文件，修改`DefaultTimeoutStopSec`设置超时时间。保存后执行`sudo systemctl daemon-reload`更新。

## NVIDIA显卡驱动安装
在独显模式下，下载并安装[NVIDIA显卡驱动](https://www.nvidia.cn/Download/index.aspx?lang=cn)。
```bash
sudo bash NVIDIA-Linux-x86_64-535.154.05.run
```
