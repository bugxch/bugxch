# 极简版《计算机原理》


读书笔记。

<!--more-->

![](https://github.com/bugxch/blogpics/blob/master/201807/cover.jpg?raw=true)


这两周读了日本作者矢泽久雄写的[《程序是怎么跑起来的》](https://book.douban.com/subject/26365491/)，解开了我这个作为通信专业的软件从业者的很多困惑，为了避免日后遗忘，将一些看了这本书之后的问题的解答记录下来。

**Q：电脑的 CPU 中包含哪些部分？各自的作用有哪些？**
**A**：CPU 包含寄存器，控制器，时钟和运算器四种主要的结构。如下图所示

![](https://github.com/bugxch/blogpics/blob/master/201807/computer1.jpg?raw=true)

- 控制器负责将内存上的指令、数据等读入到寄存器，并根据运算的结果控制整个计算机；
- 寄存器用来暂存数据、指令等处理对象，一般 CPU 包含 20～100 个不同的寄存器；
- 时钟负责 CPU 开始计时的时钟信号；
- 运算器负责运算从内存读入寄存器的数据

从程序员的角度来说，CPU 可以看作寄存器的集合。CPU 中包含不同种类的寄存器，各自有不同的功能，如下表所示：

| 种类       | 功能                                                         | 数目 |
| ---------- | ------------------------------------------------------------ | ---- |
| 累加寄存器 | 存储运算中和运算后的数据                                     | 1    |
| 标志寄存器 | 存储运算后的 CPU 状态                                        | 1    |
| 程序计数器 | 存取下一条指令的内存地址                                     | 1    |
| 基址寄存器 | 存储数据内存的起始地址                                       | 多个 |
| 变址寄存器 | 存储基址寄存器的相对地址                                     | 多个 |
| 通用寄存器 | 存储任意数据                                                 | 多个 |
| 指令寄存器 | 存储指令。CPU 内部使用，程序员无法通过程序对寄存器进行读写操作 | 多个 |
| 栈寄存器   | 存储栈区域的起始地址                                         | 多个 |

**Q：一个典型的 C 语言源代码在电脑中运行的基本流程是怎样的？**
**A**：C 语言写成的源代码是高级语言程序，但是 CPU 运行的代码是本地机器语言，因此 C 的源代码并不能立即运行。实际上，一个 C 的源代码需要经过编译、和链接生成. exe 的可执行文件之后，电脑会将. exe 文件的副本复制到内存中再运行，基本的流程如下图所示：

![](https://github.com/bugxch/blogpics/blob/master/201807/computer2.jpg?raw=true)

**Q：内存内部结构如何？内存的数据存取都有哪些数据结构？**
**A**：内存是计算机的主存储器，通过芯片与计算机相连，主要负责存储指令和数据，CPU 通过基址寄存器和变址寄存器读取和写入内存中的数据。内存由连续的长度为 8bit（1 个字节）的基本元素构成，程序启动之后 CPU 的控制寄存器根据时钟信号从内存中读取指令和数据。

存取内存的数据结构包括**数组、栈、堆、队列、链表和二叉树**。我们可以通过指针直接访问和改变对应内存地址中的变量的数值。

- 数组是多个同样类型的数据在内存中连续的排列的形式，可以通过数组的索引访问数组元素；
- 栈可以不通过指定地址和索引对数组元素进行读写。栈由栈底、栈顶描述，一般用来临时保存运算过程中的数据、连接在计算机设备上或者输入输出的数据；
- 队列与栈相似，栈的元素是 FILO，但是队列是 FIFO，队列一般用环形缓冲区实现；
- 链表与数组不同，它在内存中不是连续存储的，每个元素都有一个直接后继，像串珠一样将每个元素串联起来，最大优势是增减元素方便快捷；
- 二叉树中除了最终的子节点之外，每个元素都有两个后继结点，有序二叉树使得搜索变得更有效

**Q：数据和程序是如何保存在计算机中的？**
A：程序和数据是保存在计算机的硬盘中的，但是程序运行需要将机器语言的程序加载到内存，因为 CPU 的程序计数器指定内存地址才能读出程序内容。内存和磁盘因为自身特点的差异，它们之间具有紧密的联系。

- 磁盘缓存。由于磁盘的读取速度较慢，为了加快程序的运行，将磁盘中的部分数据加载到内存中缓存起来，之后在访问同一个数据的时候就直接从内存中读取数据，这样的机制叫**磁盘缓存**；

- 虚拟内存。**虚拟内存**刚好与之相反，在运行比较大的程序或者内存资源比较紧张可以将部分磁盘当作**假想的内存**来用。实现虚拟内存机制需要在磁盘为内存预留空间，并在程序运行时与内存中的内容进行置换（swap），window 中提过**分页式虚拟内存机制**，如下图所示

  ![](https://github.com/bugxch/blogpics/blob/master/201807/vitualmem.jpg?raw=true)

一般虚拟内存的大小与内存相当或者是内存的两倍。

**Q：什么是动态链接和静态链接？二者有何不同？**
**A**：DLL（Dynamic link libary）是在程序运行时候动态加载的文件，[维基百科](https://www.wikiwand.com/zh/%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93)中的解释是

> **動態連結函式庫**（英语：**Dynamic-link library**，缩写为 **DLL**）是[微软公司](https://www.wikiwand.com/zh/%E5%BE%AE%E8%BD%AF%E5%85%AC%E5%8F%B8)在[微软视窗](https://www.wikiwand.com/zh/%E5%BE%AE%E8%BD%AF%E8%A7%86%E7%AA%97)[操作系统](https://www.wikiwand.com/zh/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F)中实现共享[函数库](https://www.wikiwand.com/zh/%E5%87%BD%E6%95%B0%E5%BA%93)概念的一种实作方式。这些函式庫函数的[扩展名](https://www.wikiwand.com/zh/%E6%89%A9%E5%B1%95%E5%90%8D)是`.DLL`、`.OCX`（包含 [ActiveX](https://www.wikiwand.com/zh/ActiveX) 控制的函式庫）或者`.DRV`（舊式的系统[驱动程序](https://www.wikiwand.com/zh/%E9%A9%B1%E5%8A%A8%E7%A8%8B%E5%BA%8F))。

> 所謂動態链接，就是把一些經常會共用的程式碼（靜態链接的 OBJ 程式庫）製作成 DLL 檔，當執行檔呼叫到 DLL 檔內的函數時，Windows 作業系統才會把 DLL 檔載入記憶體內，DLL 檔本身的結構就是可執行檔，當程式需求函數才進行链接。透過動態链接方式，記憶體浪費的情形將可大幅降低。[靜態連結函式庫](https://www.wikiwand.com/zh/%E9%9D%9C%E6%85%8B%E9%80%A3%E7%B5%90%E5%87%BD%E5%BC%8F%E5%BA%AB)則是直接[連結](https://www.wikiwand.com/zh/%E9%93%BE%E6%8E%A5%E5%99%A8)到執行檔。

> DLL 的文件格式与视窗 [EXE](https://www.wikiwand.com/zh/EXE) 文件一样——也就是说，等同于 [32 位](https://www.wikiwand.com/zh/32%E4%BD%8D)视窗的[可移植执行文件](https://www.wikiwand.com/zh/Portable_Executable)（PE）和 [16 位](https://www.wikiwand.com/zh/16%E4%BD%8D)视窗的 New Executable（NE）。作为 EXE 格式，DLL 可以包括[原始碼](https://www.wikiwand.com/zh/%E5%8E%9F%E5%A7%8B%E7%A2%BC)、数据和资源的多种组合。

简单来说，已经编译成汇编语言的程序文件，在进一步链接时如果直接将库文件链接进 exe 可执行文件，则该链接文件就是静态库，如果仅仅在程序运行时才进行链接称为动态链接，链接的目标文件就是动态链接库（windows 中为 dll 文件）。需要说明的是，在链接之后，exe 文件中包含了静态链接库的所有内容，所以会比较大，而动态链接库相对轻巧，并且**动态链接库可以在被多个同时运行的程序所共有，并且保证内存中只有一个 dll 文件中调用函数的副本**，这样就节省了程序运行的空间。实际上 window 操作系统的大部分 API 目标文件是动态链接库，动态链接库一般由导入库导入，导入库中并不存在目标函数的实体，仅仅保存目标函数所在的动态链接库的名称及路径。下面的表格是对两者的总结。

| 链接类型 | 何时链接     | 是否可共享       | 文件类型 | 资源占用 |
| -------- | ------------ | ---------------- | -------- | -------- |
| 静态     | 编译后链接时 | 否               | .a/.lib  | 多       |
| 动态     | 程序运行时   | 可被多个程序共享 | .dll/.so | 少       |

关于动态链接和静态链接的详细介绍请参考博文 [C++ 静态库与动态库](http://www.cnblogs.com/skynet/p/3372855.html)。

**Q：一个 C 语言源程序是如何变成可执行文件（exe）的？又是如何在操作系统中运行的？** **A**：这是个比较大的问题，作者在书中举了个 C 语言的例子。大体来说，C 的源程序需要通过编译器编译成汇编语言（asm 文件），进一步链接需要的库文件（dll 文件）生成可执行文件（exe 文件），最后点击 exe 将可执行文件导入内存运行程序。以`Sample.c`文件为例

```c
#include <stdio.h>
#include <windows.h>

char *title = "messgae box";
double average(double a, double b)
{
    return (a + b)/2.0;
}

int WINAPI WinMain(HINSTANCE h, HINSTANCE d, LPSTR s, int m)
{
    double ave;
    char buff[80];
    ave = average(123,456);

    sprintf(buff, "average value is %f", ave);

    MessageBox(NULL, buff, title, MB_OK);

    return 0;
}
```

1. **编译**该文件，在源文件目录上运行命令`bcc32 -W -c Sample.c`，生成 sample.obj 目标文件；
2. **链接**需要的库文件，运行命令`ilink32 -Tpe -c -x -aa c0w32.obj Sample.obj, Sample.exe,, import32.lib cw32.lib`

需要说明的是，c0w32.obj 文件是与所有程序起始位置相结合的处理内容，称为程序的**启动**。在源程序中，我们**调用**了系统函数 sprintf 和 messagebox，因此，需要将这两个函数对应的库函数（其中的内容与 exe 文件相同，都是本地代码）**链接**进来，告诉链接器去哪里找这两个函数对应的本地代码。

sprintf 的本地代码在 cwlib32.lib 中，编译之后会将它的目标函数合成到 exe 文件中，称为**静态链接**；而 messagebox 的本地代码在库文件 user32.dll 里，使用 import32.dll 是为了告诉连接器 “messagebox 在库文件 user32.dll 中，以及 user32.dll 在哪里”，所以 import32.dll 称为导入库。程序运行时，执行从 DLL 文件调出的 MessageBox() 函数这一信息就会和 exe 文件结合，称为**动态链接**。

源程序到可执行文件的流程如下所示：


![](https://github.com/bugxch/blogpics/blob/master/201807/run.jpg?raw=true)

**Q：可执行文件包含哪些内容？它加载到内存中是什么样子？**
**A**：可执行文件中包含了源程序的变量和函数的虚拟地址，在加载到内存之后需要必要的信息将虚拟地址转换成实际地址，转换需要的信息就在 exe 文件开始的部分，称为再配置信息。exe 文件被加载到内存之后，就将这些虚拟内存转换成实际内存，程序运行中会生成栈和堆，因此在内存中的样子如下图所示

![](https://github.com/bugxch/blogpics/blob/master/201807/memab.jpg?raw=true)

**Q：c，o，a，lib，obj，dll 这些文件分别是什么？他们之间是什么关系？**
**A**：c 是 C 语言的源文件，如博文 [Linux 的. a、.so 和. o 文件](http://blog.csdn.net/chlele0105/article/details/23691147) 中所述

> lib,dll,exe 都算是最终的目标文件，是最终产物。而 c/c++ 属于源代码。源代码和最终目标文件中过渡的就是中间代码 obj，实际上之所以需要中间代码，是你不可能一次得到目标文件。比如说一个 exe 需要很多的 cpp 文件生成。而编译器一次只能编译一个 cpp 文件。这样编译器编译好一个 cpp 以后会将其编译成 obj，当所有必须要的 cpp 都编译成 obj 以后，再统一 link 成所需要的 exe，应该说缺少任意一个 obj 都会导致 exe 的链接失败。而 .o, 是 Linux 目标文件, 相当于 windows 中的. obj 文件，.so 文件为共享库, 是 shared object, 用于动态连接的, 相当于 windows 下的 dll,.a 为静态库, 是好多个. o 合在一起, 用于静态链接

**Q：什么是_BSS 段和_DATA 段？全局变量和局部变量在程序运行时有何不同？**
**A**：这是汇编语言的概念，编译器将高级语言源程序转换成汇编文件 (.asm 文件)，有如下的源文件`sample2.c`

```c
int AddNum(int a, int b)
{
    return a + b;
}

void MyFun()
{
    int c;
    c = AddNum(123,456);
}
```

经过编译之后的汇编文件（软件环境 win10，gcc 编译器）内容如下：

```c
    .file    "sample.c"
    .text
    .globl    _AddNum
    .def    _AddNum;    .scl    2;    .type    32;    .endef
_AddNum:
    pushl    %ebp
    movl    %esp, %ebp
    movl    8(%ebp), %edx
    movl    12(%ebp), %eax
    addl    %edx, %eax
    popl    %ebp
    ret
    .globl    _MyFun
    .def    _MyFun;    .scl    2;    .type    32;    .endef
_MyFun:
    pushl    %ebp
    movl    %esp, %ebp
    subl    $24, %esp
    movl    $456, 4(%esp)
    movl    $123, (%esp)
    call    _AddNum
    movl    %eax, -4(%ebp)
    leave
    ret
    .ident    "GCC: (tdm-1) 4.9.2"
```

汇编程序最接近机器语言，而且其与 C 语言一一对应，所以通过汇编文件就可以了解程序运行的大体情况。从上面的汇编文件，可以看到如下的结果

1. 寄存器 esp 指向栈顶元素地址，每个元素占据 4 个字节的数据；
2. 在每个函数开始的时候，都要将寄存器 ebp 的数据压入栈中进行保护；
3. 上述程序中隐藏的一个关键步骤是在第 21 行，call AddNum 时，计算机已经将 MyFun 函数的下一个指令的地址压入栈中，在调用完 AddNum 时（第 12 行），返回函数 Myfun 时候会自动将栈中的返回指令的地址出栈交给 CPU 的程序计数器，这样就可以实现在调用函数之后仍然返回原来的调用的地方；
4. 函数的入参被保存在栈中，返回值被保存在寄存器里。

```c
int a;
int b;
float fl;

int c = 9;
int d = 10;
int e = 11;
int f = 12;

void MyFun(void)
{
    int a1,b1,c1;
    float fl1;
    a1 = 1;
    b1 = -1;
    fl1 = -99.34;
    c1 = -87;

    a1 = a;
    b1 = b;
    fl1 = fl;
    c1 = c;
}
```

以上的 C 源代码转换成汇编语言是

```c++
    .file    "sample2.c"
    .comm    _a, 4, 2
    .comm    _b, 4, 2
    .comm    _fl, 4, 2
    .globl    _c
    .data
    .align 4
_c:
    .long    9
    .globl    _d
    .align 4
_d:
    .long    10
    .globl    _e
    .align 4
_e:
    .long    11
    .globl    _f
    .align 4
_f:
    .long    12
    .text
    .globl    _MyFun
    .def    _MyFun;    .scl    2;    .type    32;    .endef
_MyFun:
    pushl    %ebp
    movl    %esp, %ebp
    subl    $16, %esp
    movl    $1, -4(%ebp)
    movl    $-1, -8(%ebp)
    movl    LC0, %eax
    movl    %eax, -12(%ebp)
    movl    $-87, -16(%ebp)
    movl    _a, %eax
    movl    %eax, -4(%ebp)
    movl    _b, %eax
    movl    %eax, -8(%ebp)
    movl    _fl, %eax
    movl    %eax, -12(%ebp)
    movl    _c, %eax
    movl    %eax, -16(%ebp)
    leave
    ret
    .section .rdata,"dr"
    .align 4
LC0:
    .long    -1027166700
    .ident    "GCC: (tdm-1) 4.9.2"
```

从中可以看出全局变量保存在. comm 和. globl 段，局部变量保存在寄存器中，因此在程序运行的整个过程中，全局变量可以随时访问，但是局部变量却会在用过之后消失。

关于 windows 的汇编的内容可进一步参考文章[汇编与逆向分析](http://www.mouseos.com/assembly/index.html)

---

以上是此书最干货的部分，书中该介绍了计算机二进制数，和计算机硬件的部分内容，在此略过不提。
