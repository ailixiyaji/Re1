# Re0:从零开始的逆向学习生活
## 数据类型和寄存器
可以供我们载入（load）或者存储（store）的数据类型可以分为有符号和无符号类型的字，半字，或字节。对这些数据类型的扩展是：半字为-h，-sh，字节为-b或者-sb，字没有扩展。
涉及到的指令集包括

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
## 内存指令，加载和存储
通常，LDR用于将内存数据加载到寄存器中，STR用于从寄存器的值存储到内存地址对应的内存中。
### 立即数用作偏移
![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/fb5d8878-871b-4837-9f09-dc13f368d2ec)
_start:

       ldr r0, adr_var1  将变量var1的内存地址通过标签adr_var1载入R0

       ldr r1, adr_var2  将变量var2的内存地址通过标签adr_var2载入R1

       ldr r2, [r0]      @ 将r0里的值作为地址取出里面的值（0x03）存入寄存器R2 

       str r2, [r1, #2]     寻址模式：偏移模式。将存储在R2里的值（0x03）存放在以r1+2为地址指向的内存空间中。基址寄存器（R1）的值不变

       str r2, [r1, #4]!    寻址模式：先索引模式。将存储在R2里的值（0x03）存放在以r1+4为地址指向的内存空间中，然后基址寄存器（R1）被修改为R1=R1+4

       ldr r3, [r1], #4    寻址模式：后索引模式。将存储在R1里的值作为内存地址取出里面的值存放在以r3中（而非R3+4），然后基址寄存器（R1）被修改为R1=R1+4

       bkpt                  中断，暂停程序
### 寄存器的值用作偏移
![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/f64ed2ba-8781-4b93-bc54-eba57f5508eb)
  ldr r0, adr_var1  透过adr_var1标签，将var1变量的内存地址加载进R0

       ldr r1, adr_var2  透过adr_var2标签，将var2变量的内存地址加载进R1

       ldr r2, [r0]    将R0里的值作为内存地址，把地址里的数值（0x03）载入r2

       str r2, [r1, r2]     寻址模式：偏移寻址。将R2里的值存入：R1+R2（0x03，偏移）的结果所指向的内存空间中。基址寄存器不更新。

       str r2, [r1, r2]!    寻址模式：先索引模式。将R2的值（0x03）存入：R1+R2（0x03，用作偏移）得出的结果所指向的内存空间中。基址寄存器R1更新为R1 = R1+R2。

       ldr r3, [r1], r2  寻址模式：后索引模式。将R2的值作为地址取出里面的值，并将其存入寄存器R3。基址寄存器R1更新为R1=R1+R2。
### 移位寄存器作为偏移
![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/726381d4-24f0-4519-a5c3-de5aa5dd0487)
_start:

              ldr r0, adr_var1      透过adr_var1标签，将var1变量的内存地址加载进R0

              ldr r1, adr_var2      透过adr_var2标签，将var2变量的内存地址加载进R1

              ldr r2, [r0]              将R0里的值作为内存地址，把里面的值（0x03）载入r2

              str r2, [r1, r2, LSL#2]   寻址模式：偏移寻址。将R2里的值（0x03）存入：R1的值+偏移量（R2左移2位后的值）的结果所指向的内存空间中。基址寄存器（R1）不更新。

              str r2, [r1, r2, LSL#2]!  寻址模式：先索引寻址。将R2里的值（0x03）存入：R1的值+偏移量（R2左移2位后的值）的结果所指向的内存空间中。基址寄存器（R1）更新为R1 = R1 + R2<<2。

              ldr r3, [r1], r2, LSL#2   寻址模式：后索引寻址。将R1的值作为地址取出里面的值，并将其存入寄存器R3。基址寄存器R1更新为R1=R1+R2<<2。

              bkpt
### 加壳与脱壳
或可用PEiD对压缩文件进行探测并用UPX解压缩（24/2/27）
### 部分软件使用
***Ollydbg***

F2    设置一个断点（如果断点已经存在，那么断点将被删除）

F4    运行到光标所在行（运行到光标所在行时自动断下）

F7    单步跟踪（如果遇到一个call，则跟踪进入）

F8    单步跟踪（如果遇到一个call，则执行完整个call）

F9    继续执行（运行程序，直到进程退出或遇到下一个断点）


***IDA***

空格    在图形模式和列表视图模式之间切换反汇编视图

F5      将反汇编指令还原为伪代码

x       查看交叉引用

n       对变量名或者函数名进行重命名操作

d       将二进制数据解释为字节/双字/四字

c       将二进制数据解释为代码

a       将二进制数据解释为字符串

***UPX***

compress(压缩)     -(1-9) 默认8/--best

decompress(解压缩)     -d

test     检查文件是否能安全解压

list     提供文件信息

[详情](https://github.com/upx/upx/blob/master/doc/upx.pod)

### 实践
1.***scan with PEiD*** 查看编译信息
***使用OD动态调试*** using string reference find ASCII 查找提示

![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/9432d175-7b58-441b-af5f-fd5dfafe4aa9)

在关键跳转下断点（单击F2）F9运行程序F8跟踪


***IDA静态分析***

![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/cd420c84-3bad-49b8-8bde-fb65ce3ad74d)

点击提示相关字符串X交叉引用

查找F5可生成函数伪代码



2.

![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/a7a9d26f-7703-419a-b755-b0ccf76603b1)

检查得程序经过UPX加壳，需要脱壳后再分析

![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/af58529a-3acd-49bf-a70a-664578176982)

F8跟踪，选择ESP寄存器可知特殊指令位置记为Apocalypse，右击“Follow in Dump”在[hex dump]窗口中选择Apocalypse右击下断点Dword,F9运行，运行停止后删除断点[Debug][Hardware breakpoints]

F7跟踪,运行到Kaslana,再次跟踪到Scharica。在反汇编指令窗口右击[Dump debugged process][dump]保存。[Scharica]是程序入口点信息。

打开ImportREC,选择源文件在OEP中填入[Scharical]点击[IAT AutoSearch][GetImports]可见程序输入表信息，[Show Invalid]查看是否有无效输入表项目，这里没有则[Fix Dump]

![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/d8054aa0-2b77-4a16-a7cb-8e64aae56533)


脱壳完成


3.
在指令代码按X可打开交叉引用窗口查看调用函数的地址，右击在每个参考上设置断点。F9运行，断下的位置即为调用函数的位置

IDA分析，[imports]搜索，X交叉引用，F5还原函数伪代码，查找地址，Restorator查看资源。

![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/ca5cd4fb-b0ca-4089-a3cb-a1259601b8e6)

使用LoadLibrary以及GetProcAddress，动态获取API函数的地址

OD反汇编[Ctrl+G]，F2下断点。F9执行，调试执行到用户代码，

![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/98411476-c707-45f8-8058-e73b1ffab9c6)

可知004015F6处调用MessageBoxA函数。F8单步跟踪

![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/f5b59006-1a3d-4204-8a88-69ad21c21043)

IDA 004015F6处F5，


4.PEiD分析标准算法

![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/7afa08e7-59d7-4452-a3d3-9dfc14cdbb42)

IDA中可[N]重命名。

[;]添加注释。

[/]伪代码添加注释。

双击左键给汇编指令添加注释

![headImg](https://github.com/ailixiyaji/Re1/assets/145940467/6520e615-d4f4-4dd7-b2a7-96e5a42d45d2)









          

