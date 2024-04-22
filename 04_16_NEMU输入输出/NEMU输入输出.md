# NEMU输入输出

## 理解volatile关键字

```c 
void fun() {
  extern unsigned char _end;  // _end是什么?
  volatile unsigned char *p = &_end;
  *p = 0;
  while(*p != 0xff);
  *p = 0x33;
  *p = 0x34;
  *p = 0x86;
}
```

- `volatile`
  ```asm
  0000000000000000 <fun>:
    0:	f3 0f 1e fa          	endbr64 
    4:	c6 05 00 00 00 00 00 	movb   $0x0,0x0(%rip)        # b <fun+0xb>
    b:	48 8d 15 00 00 00 00 	lea    0x0(%rip),%rdx        # 12 <fun+0x12>
    12:	66 0f 1f 44 00 00    	nopw   0x0(%rax,%rax,1)
    18:	0f b6 02             	movzbl (%rdx),%eax
    1b:	3c ff                	cmp    $0xff,%al
    1d:	75 f9                	jne    18 <fun+0x18>
    1f:	c6 05 00 00 00 00 33 	movb   $0x33,0x0(%rip)        # 26 <fun+0x26>
    26:	c6 05 00 00 00 00 34 	movb   $0x34,0x0(%rip)        # 2d <fun+0x2d>
    2d:	c6 05 00 00 00 00 86 	movb   $0x86,0x0(%rip)        # 34 <fun+0x34>
    34:	c3                   	ret    
  ```
- 无`volatile`
  ```asm
  0000000000000000 <fun>:
    0:	f3 0f 1e fa          	endbr64 
    4:	c6 05 00 00 00 00 00 	movb   $0x0,0x0(%rip)        # b <fun+0xb>
    b:	eb fe                	jmp    b <fun+0xb>
  ```

在不使用`volatile`时编译器认为`_end`并不存在，因此将其相关的代码都进行了优化，使得函数变为了死循环。
如果代码中p指向的地址最终被映射到一个设备寄存器, 去掉volatile可能会使得所有对设备寄存器的访问都被优化掉, 从而导致设备无法正常工作。

## 理解mainargs
- nemu
  nemu中通过编译时指定预处理器宏`MAINARGS`来指定参数内容
  ```
  CFLAGS += -DMAINARGS=\"$(mainargs)\"
  ```
- native
  native中使用`getenv`获取环境变量`mainargs`的值
  ```c
  const char *args = getenv("mainargs");
  ```

## 如何检测多个键同时被按下?
因为当按键松开的时候会发生一个断码，因此可以通过当前检测到通码但未接收到断码来判断按下的按键，从而存储多个按键被按下的状态。

## 神奇的调色板
> 在一些90年代的游戏中(比如仙剑奇侠传), 很多渐出渐入效果都是通过调色板实现的, 聪明的你知道其中的玄机吗?

每一帧渲染前更换颜色渐深或渐浅的调色板，从而改变实际显示的颜色，实现渐入渐出的效果。

## 游戏是如何运行的
> 请你以打字小游戏为例, 结合"程序在计算机上运行"的两个视角, 来剖析打字小游戏究竟是如何在计算机上运行的. 具体地, 当你按下一个字母并命中的时候, 整个计算机系统(NEMU, ISA, AM, 运行时环境, 程序) 是如何协同工作, 从而让打字小游戏实现出"命中"的游戏效果?

1. 当按下一个按键时，NEMU通过ISA每条指令执行后的`device_update`通过NEMU运行时环境中的SDL检测当前获取的键盘扫描码，存入对应的键盘扫描码mmio中。
2. 程序每次循环调用运行时环境的接口`ioe_read`，运行时环境通过AM将函数转换为对应的ISA指令，访问mmio，获得当前键盘扫描码。
3. 程序判断当前按下的键盘扫描码为正确的按键，就更新对应的字符结构体中的状态，将绿色的字符再次通过`ioe_read`更新Frame Buffer。
4. 运行时环境通过AM将函数转换为对应的ISA指令，访问mmio，将对应内容写入Frame Buffer。
5. NEMU在每次运行完一条ISA指令后，通过`device_update`调用`vga_update_screen`根据是否需要同步显示，将Frame Buffer中的内容通过SDL显示到屏幕上。


