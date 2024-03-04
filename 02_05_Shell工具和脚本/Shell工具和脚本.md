# Shell工具和脚本
> 阅读`man ls`，然后使用`ls`命令进行如下操作：
> - 所有文件(包括隐藏文件)

```bash
ls -a
```

> - 文件打印以人类可以理解的格式输出(例如，使用454M而不是4542799954)

```bash
ls -lh
```

> - 文件以最近访问顺序排序

```bash
ls -ltu
```
课后习题解答中给出的命令是`ls -lt`, 但是`-t`选项是按照修改时间排序，而不是访问时间。因此应该额外使用`-u`选项。
> - 以彩色文本显示输出结果

```bash
ls --color=auto
```

> 编写两个bash函数 `marco` 和 `polo` 执行下面的操作。 每当你执行 `marco` 时，当前的工作目录应当以某种形式保存，当执行 `polo` 时，无论现在处在什么目录下，都应当 `cd` 回到当时执行 `marco` 的目录。 为了方便debug，你可以把代码写在单独的文件 `marco.sh` 中，并通过 `source marco.sh`命令，（重新）加载函数。

- `marco.sh`
    ```bash
    #!/bin/bash
    marco(){
        pwd > /tmp/missing/marco_history.log
    }
    ```

- `polo.sh`
    ```bash
    #!/bin/bash
    polo(){
        cd "$(cat '/tmp/missing/marco_history.log')"
    }
    ```

> 假设您有一个命令，它很少出错。因此为了在出错时能够对其进行调试，需要花费大量的时间重现错误并捕获输出。 编写一段bash脚本，运行如下的脚本直到它出错，将它的标准输出和标准错误流记录到文件，并在最后输出所有内容。 加分项：报告脚本在失败前共运行了多少次。
> ```bash
> #!/usr/bin/env bash
>
> n=$(( RANDOM % 100 ))
>
> if [[ n -eq 42 ]]; then
>    echo "Something went wrong"
>    >&2 echo "The error was using magic numbers"
>    exit 1
> fi
>
> echo "Everything went according to plan"
>```

```bash
#!/usr/bin/env bash
count=0
echo > out.log

while true
do
  ./buggy.sh &>> out.log
  if [[ $? -ne 0 ]]; then
	cat out.log
	echo "failed after $count times"
	break
  fi
  ((count++))
done
```

> 编写一个命令，它可以递归地查找文件夹中所有的HTML文件，并将它们压缩成zip文件。注意，即使文件名中包含空格，您的命令也应该能够正确执行(提示：查看`xargs`的参数`-d`)

```bash
find . -type f -name '*.html' | xargs -d "\n" tar -cvzf html.zip
```

> 编写一个命令或脚本递归的查找文件夹中最近使用的文件。更通用的做法，你可以按照最近的使用时间列出文件吗？

`find . -type f -print0 | xargs -0 ls -lt | head -1`