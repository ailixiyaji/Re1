![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/89e29523-c128-402a-ac2b-120205eb798f)# Re1
##数据类型和寄存器
>  可以供我们载入（load）或者存储（store）的数据类型可以分为有符号和无符号类型的字，半字，或字节。对这些数据类型的扩展是：半字为-h，-sh，字节为-b或者-sb，字没有扩展。
> 涉及到的指令集包括

       ldr = Load Word  载入字

       ldrh = Load unsigned Half Word 载入无符号半字

       ldrsh = Load signed Half Word 载入有符号半字

       ldrb = Load unsigned Byte 载入无符号字节

       ldrsb = Load signed Bytes 载入有符号字节

 

       str = Store Word 储存字

       strh = Store unsigned Half Word 储存无符号半字

       strsh = Store signed Half Word 储存有符号半字

       strb = Store unsigned Byte 储存无符号字节

       strsb = Store signed Byte 储存有符号字节

一般有30个32位的通用寄存器，而R0-R15是在任意特权模式下都可以访问的寄存器，可以分为两组：通用可特殊功能。
r0-r11为通用，R12是IP寄存器，内部程序调用寄存器。R13，SP，堆栈指针寄存器。R14，LR，连接寄存器。R15，PC，程序计数器。CPSR，当前程序状态寄存器
  R0-R12可以在通常的运算过程中用来存储临时的数据，指针（定位内存）等。以R0为例，当我们执行算数运算或者存储当前函数的返回值时，可以把R0视为累加器。系统调用发生时，R11开始生效，它存储了系统调用数值。R11作为栈指针帮助我们追踪栈的边界（稍后会讲到）。此外，ARM专用的函数调用规则规定了函数的前四个参数应该分别存贮与R0到R3中。
## ARM处理器总是获取当前已经执行的指令的后两条指令的地址。
  ARM处理器有两种工作状态ARM和Thumb。这两种工作状态和运行模式没有任何关系。比如不论是ARM还是Thumb状态的代码都可以运行在用户模式下。这两种工作状态之间最大的差异是指令集，ARM状态的指令长度是32位的，Thumb状态的指令长度是16位的（也可能为32位）。了解如何使用Thumb工作状态对于编写ARM平台的漏洞利用是至关重要的。当我们编写ARM shellcode时，需要使用16 bit的Thumb指令代替32 bit的ARM指令，从而避免在指令中出现’\0’截断。
ARM指令通常跟一到两个操作数，我们使用如下模板描述：

      MNEMONIC{S}{condition} {Rd}, Operand1, Operand2

      其中各个字段的作用如下：

      MNEMONIC     - 指令的助记符如ADD

      {S}                     - 可选的扩展位，如果指令后加了S，将依据计算结果更新CPSR寄存器中相应的FLAG

      {condition}       - 执行条件，如果没有指定，默认为AL(无条件执行)

      {Rd}                  - 目的寄存器，存储指令计算结果

      Operand1        - 第一个操作数，可以是一个寄存器或一个立即数

      Operand2        - 第二个(可变)操作数，可以是一个立即数或寄存器甚至带移位操作的寄存器

       助记符、S扩展位、目的寄存器和第一个操作数的作用很好理解，不多做解释，这里补充解释一下执行条件和第二个操作数。设置了执行条件的指令在执行指令前先校验CPSR寄存器中的标志位，只有标志位的组合匹配所设置的执行条件指令才会被执行。第二个操作数被称为可变操作数，因为它可以被设置为多种形式，包括立即数、寄存器、带移位操作的寄存器，如下所示：

       #123         - 立即数

       Rx           - 寄存器比如R1

       Rx, ASR n    - 对寄存器中的值进行算术右移n位后的值

       Rx, LSL n    - 对寄存器中的值进行逻辑左移n位后的值

       Rx, LSR n    - 对寄存器中的值进行逻辑右移n位后的值

       Rx, ROR n    - 对寄存器中的值进行循环右移n位后的值

       Rx, RRX      - 对寄存器中的值进行带扩展的循环右移1位后的值
### 常见指令
![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/496ccd8c-45ce-451a-8aed-5c839c1f7102)
