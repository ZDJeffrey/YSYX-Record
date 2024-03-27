# PA1

## `clangd`配置
使用`bear`根据Makefile生成`compile_commands.json`，供`clangd`使用
```bash
sudo apt install bear
bear --output build/compile_commands.json -- <make ... -B>
```
`-B`强行重新编译，避免`bear`无法生成`compile_commands.json`的问题。

## 尝试理解计算机如何计算
```
// PC: instruction    | // label: statement
0: mov  r1, 0         |  pc0: r1 = 0;
1: mov  r2, 0         |  pc1: r2 = 0;
2: addi r2, r2, 1     |  pc2: r2 = r2 + 1;
3: add  r1, r1, r2    |  pc3: r1 = r1 + r2;
4: blt  r2, 100, 2    |  pc4: if (r2 < 100) goto pc2;   // branch if less than
5: jmp 5              |  pc5: goto pc5;
```
计算机通过寄存器暂存数据，需要计算时将寄存器中的数据通过ALU进行计算，计算结果存回寄存器。
使用寄存器`r1`暂存累加结果，使用寄存器`r2`存储累加数，`r2`从0开始，每次循环加1，并将`r1`+`r2`的结果存储在`r1`中，直到`r2`大于等于100时跳出累加循环，`r1`中存储计算结果。

## 从状态机视角理解程序运行
使用状态机描述上述程序运行过程
```
(0,x,x)->(1,0,x)->(2,0,0)->               //初始化
(3,0,1)->(4,1,1)->(2,1,1)->               //第一次循环
(3,1,2)->(4,3,2)->(2,3,2)->               //第二次循环
...
(3,4853,99)->(4,4951,99)->(2,4951,99)->   //第99次循环
(3,4951,100)->(4,4951,100)->              //第100次循环
(5,4951,100)->(5,4951,100)->...           //死循环
```

## 宏与条件编译
```c
#define CHOOSE2nd(a, b, ...) b
#define MUX_WITH_COMMA(contain_comma, a, b) CHOOSE2nd(contain_comma a, b)
#define MUX_MACRO_PROPERTY(p, macro, a, b) MUX_WITH_COMMA(concat(p, macro), a, b)
// define placeholders for some property
#define __P_DEF_0  X,
#define __P_DEF_1  X,
// define some selection functions based on the properties of BOOLEAN macro
#define MUXDEF(macro, X, Y)  MUX_MACRO_PROPERTY(__P_DEF_, macro, X, Y)
```
使用`gcc -E`生成预编译文件，对`MUXDEF`进行解析。以宏定义`A`为例子。
当`#define A 1`或`#define A 0`时，`concat(__P_DEF_, A)`会得到`__P_DEF_1`或`__P_DEF_0`，进而预编译得到`X,`，因此`CHOOSE2nd(contain_comma X, Y)`得到`CHOOSE2nd(X, X, Y)`，最终得到`X`。
当未定义`A`时，`concat(__P_DEF_, A)`会得到`__P_DEF_A`，因此`CHOOSE2nd(contain_comma X, Y)`得到`CHOOSE2nd(__P_DEF_A X, Y)`，最终得到`Y`。
后续的`IFDEF`等宏定义则在此基础上实现各自的功能。

## `init_monitor`函数实现全为函数调用
因为需要兼容各类ISA，通过统一API用于`init_monitor`中调用，可以在不修改代码的情况下切换使用不同的ISA。

## `getopt_long`参数
将`main`函数的`argc`、`argv`作为`getopt_long`的参数，从而解析命令行参数。

## `LOG`宏
```c
#define Log(format, ...) \
    _Log(ANSI_FMT("[%s:%d %s] " format, ANSI_FG_BLUE) "\n", \
        __FILE__, __LINE__, __func__, ## __VA_ARGS__)
```
其中使用`## __VA_ARGS__`目的在于连接`__VA_ARGS__`，`##`可以避免在`__VA_ARGS__`为空时产生多余的逗号。

## `cpu_exec(-1)`
`void cpu_exec(uint64_t n)`中的参数`n`表示指令执行的最大数量，而传入`-1`根据补码0XFFFF FFFF FFFF FFFF会将其转换为`uint64_t`的最大值，即最多执行$2^{64}-1$条指令。

## 程序的结束
通过查看反汇编代码，发现程序实行首先会在`_start`段中进行初始化，将`main`函数的地址传给`__libc_start_main`，然后调用`main`函数，`main`函数返回后会回到`__libc_start_main`，此时还需要进行资源释放等清理工作和异常处理等。因此`main`的`return`语句并非程序的结束。

## 优美地退出程序
`nemu_state.state`的默认值为`NEMU_STOP`，当使用`c`正常执行完指令后，会改为`NEUM_END`，而`q`进行退出时，并不会改变`nemu_state.state`的值。
在`main`函数返回前，会根据`nemu_state`判断返回值，仅当`nemu_state`满足`(nemu_state.state == NEMU_END && nemu_state.halt_ret == 0) ||(nemu_state.state == NEMU_QUIT)`时才返回0。因此在未执行过指令就退出时，因为处于`NEMU_STOP`而返回`-1`。
通过在`cmd_q`中将状态修改为`NEMU_QUIT`即可实现优美地退出程序。

