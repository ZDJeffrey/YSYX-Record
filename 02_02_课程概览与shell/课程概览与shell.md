# 课程概览与shell
> 将以下内容一行一行地写入`semester`文件：
> ```bash
> #!/bin/sh
> curl --head --silent https://missing.csail.mit.edu
> ```

```shell
echo '#!/bin/sh' > semester
echo 'curl --head --silent https://missing.csail.mit.edu' >> semester
```
第二条命令使用`>>`将内容追加到文件末尾。

> 执行`semester`文件

文件权限为`-rw-r--r--`，没有执行权限，因此需要添加执行权限。
为所有用户添加执行权限：
```shell
chmod +x semester
```

> 使用`|`和`>`，将`semester`文件输出的最后更改日期信息，写入主目录下的 last-modified.txt 的文件中

最后更改日期信息属性名为`last-modified`，使用`grep`命令提取该属性的所在行并重定向到`last-modified.txt`文件中。
```shell
./semester | grep 'last-modified' > ~/last-modified.txt
```

> 写一段命令来从`/sys`中获取笔记本的电量信息，或者台式机CPU的温度。

- 笔记本电量信息
    ```shell
    cat /sys/class/power_supply/BAT0/capacity
    ```
- 台式机CPU温度
    ```shell
    cat /sys/class/thermal/thermal_zone0/temp
    ```



