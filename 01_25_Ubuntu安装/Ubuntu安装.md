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

## apt-get设置代理
```shell
sudo vim /etc/apt/apt.conf.d/proxy.conf
```
填入以下内容：
```
Acquire {
        http::proxy "http://127.0.0.1:7890";
        https::proxy "http://127.0.0.1:7890";
}
```

## 输入法
根据搜狗[官方教程](https://shurufa.sogou.com/linux/guide)安装搜狗输入法。

## 中文字体
在英文模式下，中文部分字体的显示效果奇怪，查看字体配置文件`/etc/fonts/conf.d/64-language-selector-prefer.conf`，发现简体中文字体的优先级较低，优先级最高的为`JP`，调整优先级，将`SC`优先级调整为最高。
```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
	<alias>
		<family>sans-serif</family>
		<prefer>
			<family>Noto Sans CJK SC</family>
			<family>Noto Sans CJK TC</family>
			<family>Noto Sans CJK HK</family>
			<family>Noto Sans CJK JP</family>
			<family>Noto Sans CJK KR</family>
			<family>Lohit Devanagari</family>
			<family>Noto Sans Sinhala</family>
		</prefer>
	</alias>
	<alias>
		<family>serif</family>
		<prefer>
			<family>Noto Serif CJK SC</family>
			<family>Noto Serif CJK TC</family>
			<family>Noto Serif CJK JP</family>
			<family>Noto Serif CJK KR</family>
			<family>Lohit Devanagari</family>
			<family>Noto Serif Sinhala</family>
		</prefer>
	</alias>
	<alias>
		<family>monospace</family>
		<prefer>
			<family>Noto Sans Mono CJK SC</family>
			<family>Noto Sans Mono CJK TC</family>
			<family>Noto Sans Mono CJK HK</family>
			<family>Noto Sans Mono CJK JP</family>
			<family>Noto Sans Mono CJK KR</family>
		</prefer>
	</alias>
</fontconfig>
```


## Grub默认启动项
安装双系统后，Grub默认启动项为Ubuntu，修改默认启动项为Windows。
修改`/etc/default/grub`文件，将`GRUB_DEFAULT`修改为windows对应启动项编号。同时可以修改`GRUB_TIMEOUT`设置选择时间。保存后执行`sudo update-grub`更新。

## Ubuntu关机停止服务时间
Ubuntu关机时，会等待服务停止后再关机，若超时则强制关机。修改`/etc/systemd/system.conf`文件，修改`DefaultTimeoutStopSec`设置超时时间。保存后执行`sudo systemctl daemon-reload`更新。

## NVIDIA显卡驱动安装
由于在安装Ubuntu时使用集成显卡，在无法通过`Softwares&Updates`安装NVIDIA显卡驱动，也无法使用NVIDIA官方驱动安装。并且切换到独显时无法正常使用，因此尝试在集显模式下使用`apt`安装NVIDIA显卡驱动。
```bash
sudo apt install nvidia-driver-535
```
安装完成后，重启系统，切换到独显模式，可以正常使用。

## git配置
- 配置用户名和邮箱
```bash
git config --global user.name "$Your_name"
git config --global user.email $your_email
```
- 配置代理
```bash
# 全局代理配置（使用socks5更安全）
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy https://127.0.0.1:7890
# 仅代理github（推荐）
git config --global http.https://github.com.proxy socks5://127.0.0.1:7890
```

## 双系统时间同步
在Linux中，系统时间会作为UTC时间，再根据时区转换为本地时间。而Windows中，系统时间为本地时间。因此在双系统中，会出现时间不同步的情况。
在Linux中设置系统时钟RTC为本地时间即可解决，执行如下命令：
```bash
timedatectl set-local-rtc 1 --adjust-system-clock
```
其中`--adjust-system-clock`选项表示同时调整系统时钟。


## `poweroff`
> Why executing the `poweroff` command requires superuser privilege in some Linux distributions? Can you provide a scene where bad thing will happen if the `poweroff` command does not require superuser privilege?

`poweroff`会将系统关机，从而使运行的程序被强制终止。若不设置权限，在用户可以随意关闭系统，容易出现误操作或多用户的恶意操作。
若`poweroff`不需要root权限，则病毒程序可以直接通过`poweroff`命令关闭系统，从而影响用户的正常使用。

## `Bash-it`
在默认的`bash`中，并不会显示git分支信息，因此使用`Bash-it`配置`bash`，[github链接](https://github.com/Bash-it/bash-it)

## Flameshot
安装`flameshot`截图工具，在使用时由于屏幕缩放比为150%，导致使用截图时存在闪屏现象，需要在先设置`QT_SCREEN_SCALE_FACTORS`为1.5。
设置快捷键命令为
```shell
env QT_SCREEN_SCALE_FACTORS=1.5 flameshot gui
```
相关issue：[Fails when fractional scaling <> 100%](https://github.com/flameshot-org/flameshot/issues/564)