# SoC

## char-test
### UART寄存器位置
根据SoC设备地址约定，UART的基地址为0x10000000，根据手册，UART的THR寄存器的偏移量为0x00
### 二进制文件
```bash
riscv64-linux-gnu-gcc -march=rv32e_zicsr -mabi=ilp32e -nostdlib -ffreestanding -static -e _start -Wl,-Ttext=0 -O2 -o char-test char-test.c
riscv64-linux-gnu-objcopy -j .text -O binary char-test char-test.bin
```
其中`-O2`能够避免入口函数的入栈操作，避免在未初始化栈指针时的使用。

### 非换行输出
在没有出现换行符的情况下，verilator仿真`$write`的时候，并不会立即输出，需要添加选项`--autoflush`使其每次调用`$display`等函数时都会立即输出。
在之前总线实现的UART中通过chisel的`printf`函数生成的`$fwrite`并不会有这个问题。

## 添加ysyxSoC的AM运行时环境
### 链接
将LDFLAGS中的_pmem_start赋值为0x20000000，创建新的链接脚本，设置栈顶地址和堆的初始位置为SRAM初始位置(`0x0f000000`)
### putch
仿照char-test的实现，实现字符的串口输出
### 无法运行fib
由于程序存储在ROM中，无法进行写入，而全局变脸的地址就位于ROM中，因此不能对全局变量进行写入


## 智能的链接过程
> 你已经在klib中实现了printf(), 但如果没有在mem-test中调用printf(), 链接后的可执行文件确实不包含printf()的代码. 这种智能的链接方式能够在存储器空间有限的情况下避免生成不必要的代码, 你知道目前的链接流程中有哪些相关的步骤或选项来实现上述功能吗?

在AM和Klib编译时，通过设置编译选项`-ffunction-sections`将各函数放置在独立的section中，通过链接时的`--gc-sections`选项，可以在链接时去除未被调用的section，从而减小生成的二进制文件的大小。


## UART输出字符
在未初始化的状态下，UART的除数锁存器始终为0，导致transmitter不会进行状态转移，无法设置tfifo的pop信号为1，从而在fifo为full时无法再放入更多数据。

## 减少coremark的运行时间
减少coremark迭代次数

## 尝试在flash上执行flash_read()函数
`flash_read`由多条指令组成，而每次取指令都需要通过XIP形式获得，因此每次写入SPI Master中的数据都会在下一条指令取指时被覆盖，无法正常运行。

## 更实际的帧缓冲实现方案
> 通常, 帧缓冲一般是在内存中分配, 通过配置VGA控制器中的部分寄存器, 可以让VGA控制器从内存中读取像素信息.
> 思考一下, 如果在当前的ysyxSoC配置中将帧缓冲分配到内存中, 可能会造成什么问题?

此时无法连续每个周期都输出有效数据，因此不能和CPU使用同样的时钟，需要将时钟进行分频，确保VGA控制模块在需要输出有效数据时，能够在每个时钟周期内都能够从内存读取到数据。