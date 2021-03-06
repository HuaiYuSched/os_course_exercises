#lec8 虚拟内存spoc练习


NOTICE
- 有"w4l2"标记的题是助教要提交到学堂在线上的。
- 有"w4l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。

提前准备

- 完成lec8的视频学习和提交对应的在线练习
- git pull uco
re_os_lab, v9_cpu, os_course_spoc_exercises 　in github repos。这样可以在本机上完成课堂练习。
- 理解如何实现建立页表，给用户态建立页表，在用户态使用虚地址，产生各自也访问错误的基本应对

## 视频相关思考题
### 8.1 虚拟存储的需求背景

 1. 寄存器、高速缓存、内存、外存的访问特征？
  * 位置、是否有软件参与访问、访问频率
  {%s%}
  寄存器位于CPU芯片内部，由编译器或者程序指定其访问和使用，其使用频率最高。
  高速缓存位于CPU的内部，具有多级缓存的多核CPU其L1,L2 级缓存由单核独享，L3级缓存由多核共享。其访问频率低于寄存器。当需要访问内存时，CPU会自动先在高速缓存中寻找相应的内容，如找不到再去内存中寻找，因此没有软件参与。
  内存位于主板上，其访问频率低于高速缓存，由软件显式访问。
  外存访问频率最低，由软件显式访问。
  {%ends%}
 1. 在你写程序时遇到过内存不够的情况吗？尝试过什么解决方法？
  {%s%}
 有。在Linux平台上，使用ulimit调整内存使用大小限制。{%ends%}

### 8.2 覆盖和交换

 1. 什么是覆盖技术？使用覆盖技术的程序开发者的主要工作是什么？

 {%s%}由于程序运行时并非任何时候都要访问程序及数据的各个部分（尤其是大程序），因此可以把用户空间分成一个固定区和若干个覆盖区。将经常活跃的部分放在固定区，其余部分按调用关系分段。首先将那些即将要访问的段放入覆盖区，其他段放在外存中，在需要调用前，系统再将其调入覆盖区，替换覆盖区中原有的段。
 由上可知，开发者需要定义程序之间的调用关系。
{%ends%}
 1. 什么是交换技术？覆盖与交换有什么不同？

 {%s%} 把处于等待状态（或在CPU调度原则下被剥夺运行权利） 的程序从内存移到辅存，把内存空间腾出来，这一过程又叫换出；把准备好竞争CPU运行的程序从辅存移到内存，这一过程又称为换入。
覆盖技术对应的是一个程序，而交换技术是多个程序之间。覆盖掉的程序是不需要再运行的，同时也不需要再载入内存，而交换出去的程序就绪后就会被交换回来。{%ends%}

 1. 如何分析内核模块间的依赖关系？

 {%s%} 在Linux中使用depmod。{%ends%}

 1. 如何内核模块间的函数调用列表？

   {%s%}
在Linux中使用nm命令或利用kallsyms机制，查看内核所需要的所有函数。{%ends%}

### 8.3 局部性原理

 1. 什么是时间局部性、空间局部性和分支局部性？
 {%s%}
时间局部性：被引用过一次的存储器位置在未来会被多次引用.
空间局部性：如果一个存储器的位置被引用，那么将来他附近的位置也会被引用
分支局部性：一个跳转指令的两次执行，很可能跳到相同的内存位置。{%ends%}
 1. 如何提高程序执行时的局部性特征？
  {%s%}
根据程序的时间局部性和空间局部性，对相邻的数据的访问安排其在时间上相邻。
根据分支局部性原理，可以使用编译器的一些扩展如likely和unlikely进行辅助。
{%ends%}
 1. 排序算法的局部性特征？
  * 参考：[九大排序算法再总结](http://blog.csdn.net/xiazdong/article/details/8462393)

### 8.4 虚拟存储概念

 1. 什么是虚拟存储？它与覆盖和交换的区别是什么？它有什么好处和挑战？
{%s%}
虚拟存储就是将一个程序的不常用的部分内存块暂存到外存中。之后由处理器通知操作系统将相应的页面调入内存中。
和覆盖交换相比其并不需要程序员介入。和交换相比，虚拟存储可以交换程序的一部分。
虚拟存储的好处有：物理内存分配的空间不再需要连续，虚拟地址空间的使用也不需要连续。可以提供给用户大于实际物理内存的地址空间。同时支持部分程序的换入换出。
{%ends%}
### 8.5 虚拟页式存储

 1. 什么是虚拟页式存储？缺页中断处理的功能是什么？

{%s%}   将虚拟存储的单位设置为页即可。在页式存储的基础上添加请求调页和页面置换。
缺页中断处理的功能是将对应的在外存中的页面调进内存中来。
{%ends%}
 1. 为了支持虚拟页式存储的实现，页表项有什么修改？
   {%s%}
需要添加一个驻留位，表示该表是否在物理内存中。修改位，表示页是否修改过。访问位，表示最近一段时间内该页是否被访问过。保护位，表示该页的访问权限。{%ends%}
 2. 页式存储和虚拟页式存储的区别是什么？
  {%s%}
 虚拟页式存储支持请求调页和页面置换。{%ends%}

### 8.6 缺页异常

 1. 缺页异常的处理流程？

 {%s%}
https://chyyuu.gitbooks.io/simple_os_book/content/zh/chapter-3/handle_pages_fault.html
{%ends%}

## 个人思考题

### 内存访问局部性的应用程序例子
---
(1)(w4l2)下面是一个体现内存访问局部性好的简单应用程序例子，请参考，在linux中写一个简单应用程序，体现内存局部性差，并给出其执行时间。
```
#include <stdio.h>
#define NUM 1024
#define COUNT 10
int A[NUM][NUM];
void main (void) {
  int i,j,k;
  for (k = 0; k<COUNT; k++)
  for (i = 0; i < NUM; i++)
  for (j = 0; j	 < NUM; j++)
      A[i][j] = i+j;
  printf("%d count computing over!\n",i*j*k);
}
```
可以用下的命令来编译和运行此程序：
```
gcc -O0 -o goodlocality goodlocality.c
time ./goodlocality
```
可以看到其执行时间。

{%s%}
将最内层循环中的ij交换位置。
{%ends%}  

## 小组思考题目
----

### 缺页异常嵌套

（1）缺页异常可用于虚拟内存管理中。如果在中断服务例程中进行缺页异常的处理时，再次出现缺页异常，这时计算机系统（软件或硬件）会如何处理？请给出你的合理设计和解释。
{%s%}
对于计算机硬件来说，异常通常是不可屏蔽的。对于操作系统来说，需要将当前的执行上下文保存起来，去执行新的异常处理程序即可。
https://www.safaribooksonline.com/library/view/understanding-the-linux/0596005652/ch04s03.html
{%ends%}

### 缺页中断次数计算
（2）如果80386机器的一条机器指令(指字长4个字节)，其功能是把一个32位字的数据装入寄存器，指令本身包含了要装入的字所在的32位地址。这个过程最多会引起几次缺页中断？
{%s%}
> 提示：内存中的指令和数据的地址需要考虑地址对齐和不对齐两种情况。需要考虑页目录表项invalid、页表项invalid、TLB缺失等是否会产生中断？    
在X86架构中，TLB不在内存中，因此不属于虚拟内存，TLB的缺失不会产生中断。
加载指令过程。加载指令时，若该指令不对齐，且页目录表项，页表项都invalid，则加载半条指令会产生中断3次，整条指令产生中断6次。同理，加载装入的字页会产生6次中断，一共12次。
{%ends%}

### 虚拟页式存储的地址转换

（3）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持8KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

PTE格式（8 bit） :
```
  VALID | PFN6 ... PFN0
```
PDE格式（8 bit） :
```
  VALID | PT6 ... PT0
```
其
```
VALID==1表示，表示映射存在；VALID==0表示，表示内存映射不存在（有两种情况：a.对应的物理页帧swap out在硬盘上；b.既没有在内存中，页没有在硬盘上，这时页帧号为0x7F）。
PFN6..0:页帧号或外存中的后备页号
PT6..0:页表的物理基址>>5
```

已经建立好了1个页目录表和8个页表，且页目录表的index为0~7的页目录项分别对应了这8个页表。

在[物理内存模拟数据文件](./04-1-spoc-memdiskdata.md)中，给出了4KB物理内存空间和4KBdisk空间的值，PDBR的值。

请手工计算后回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents，the value of addr in phy page OR disk sector。
```
1) Virtual Address 6653:
2) Virtual Address 1c13:
3) Virtual Address 6890:
4) Virtual Address 0af6:
5) Virtual Address 1e6f:
```

请写出一个translation程序（可基于python、ruby、C、C++、LISP、JavaScript等），输入是一个虚拟地址，依据[物理内存模拟数据文件](./04-1-spoc-memdiskdata.md)自动计算出对应的pde index, pde contents, pte index, pte contents，the value of addr in phy page OR disk sector。

**提示:**
```
页大小（page size）为32 Bytes(2^5)
页表项1B

8KB的虚拟地址空间(2^13)
一级页表：2^5
PDBR content: 0xd80（1101_100 0_0000, page 0x6c）

page 6c: e1(1110 0001) b5(1011 0101) a1(1010 0001) c1(1100 0001)
         b3(1011 0011) e4(1110 0100) a6(1010 0110) bd(1011 1101)
二级页表：2^5
页内偏移：2^5

4KB的物理内存空间（physical memory）(2^12)
物理帧号：2^7

Virtual Address 0330(0 00000 11001 1_0000):
  --> pde index:0x0(00000)  pde contents:(0xe1, 11100001, valid 1, pfn 0x61(page 0x61))
  page 6c: e1 b5 a1 c1 b3 e4 a6 bd 7f 7f 7f 7f 7f 7f 7f 7f
           7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f
  page 61: 7c 7f 7f 4e 4a 7f 3b 5a 2a be 7f 6d 7f 66 7f a7
           69 96 7f c8 3a 7f a5 83 07 e3 7f 37 62 30 7f 3f
    --> pte index:0x19(11001)  pte contents:(0xe3, 1 110_0011, valid 1, pfn 0x63)
  page 63: 16 00 0d 15 00 1c 1d 16 02 02 0b 00 0a 00 1e 19
           02 1b 06 06 14 1d 03 00 0b 00 12 1a 05 03 0a 1d
      --> To Physical Address 0xc70(110001110000, 0xc70) --> Value: 02

Virtual Address 1e6f(0 001_11 10_011 0_1111):
  --> pde index:0x7(00111)  pde contents:(0xbd, 10111101, valid 1, pfn 0x3d)
  page 6c: e1 b5 a1 c1 b3 e4 a6 bd 7f 7f 7f 7f 7f 7f 7f 7f
           7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f
  page 3d: f6 7f 5d 4d 7f 04 29 7f 1e 7f ef 51 0c 1c 7f 7f
           7f 76 d1 16 7f 17 ab 55 9a 65 ba 7f 7f 0b 7f 7f
    --> pte index:0x13  pte contents:(0x16, valid 0, pfn 0x16)
  disk 16: 00 0a 15 1a 03 00 09 13 1c 0a 18 03 13 07 17 1c
           0d 15 0a 1a 0c 12 1e 11 0e 02 1d 10 15 14 07 13
      --> To Disk Sector Address 0x2cf(0001011001111) --> Value: 1c
```

## 扩展思考题
---
(1)请分析原理课的缺页异常的处理流程与lab3中的缺页异常的处理流程（分析粒度到函数级别）的异同之处。

(2)在X86-32虚拟页式存储系统中，假定第一级页表的起始地址是0xE8A3 B000，进程地址空间只有第一级页表的4KB在内存。请问这4KB的虚拟地址是多少？它对应的第一级页表项和第二级页表项的物理地址是多少？页表项的内容是什么？

## v9-cpu相关


[challenge]在v9-cpu上，设定物理内存为64MB。在os.c,os2.c,os4.c,os5的基础上实现os6.c，可体现基本虚拟内存管理机制，内核空间的映射关系： kernel_virt_addr=0xc00000000+phy_addr，内核空间大小为64MB，虚拟空间范围为0xc0000000--x0xc4000000, 物理空间范围为0x00000000--x0x04000000；用户空间的映射关系：user_virt_addr=0x40000000+usr_phy_addr，用户空间可用大小为2MB，虚拟空间范围为0x40000000--0x40200000，物理空间范围为0x02000000--x0x02200000，但只建立低地址的1MB的用户空间页表映射。可参考v9-cpu git repo的testing分支中的os.c和mem.h。修改代码为os5.c

- (1)在建立页表后，进入用户态，能够在用户态访问基于用户空间的映射关系
- (2)在用户态和内核态产生各种也访问的错误，并能够通过中断服务例程进行信息提示
- (3)内核通过中断服务例程在感知到用户态访问高地址的空间，且没有超过0x40200000时，内核动态建立页表，确保用户态程序可以正确运行



如果一个用户态进程访问一个合法用户地址，产生内存访问异常后，v9-cpu会把产生异常的pc存在哪里？中断服务例程应该如何设计，可以返回到用户态产生错误的地址再次执行?
