---
title: 计算机系统概述
---

## 概述

{% note warn %}
计算、实验和理论已成为科学研究的三大方法
{% endnote %}

{% note info %}
利用计算机实验，再现、预测和发现客观世界运动规律和演化特性的全过程
{% endnote %}


1. 无损伤：模拟真实实验无法进行的事情(海啸、地震、核爆、气候等)
2. 全过程全时空分析：充分了解和细致认识研究对象
3. 低成本、短周期、反复细致地进行

{% note info %}
* 成功的程序员关心其所写的程序性能如何
{% endnote %}

上世纪60~70年代，计算机性能受制于内存容量。程序员尽可能少地用内存提高程序执行速度
<img src="/mse/computer/pic/2.bmp" alt="Smiley face" width="600">

现代，程序员需要理解的是**存储的层次化**特性和**处理器的并行化**特点


{% note warn 决定程序性能的主要因素%}
* 程序中使用的算法——涉及数据结构、算法设计
* 创建程序并翻译成机器指令的软件——涉及编译原理
* 计算机各部件执行效率——涉及计算机原理、操作系统
{% endnote %}

<img src="/mse/computer/pic/3.bmp" alt="Smiley face" width="140" height="200">
<img src="/mse/computer/pic/4.bmp" alt="Smiley face" width="140" height="200">
<img src="/mse/computer/pic/5.bmp" alt="Smiley face" width="140" height="200">
<img src="/mse/computer/pic/6.bmp" alt="Smiley face" width="140" height="200">
<img src="/mse/computer/pic/7.bmp" alt="Smiley face" width="140" height="200">


### 如何理解程序性能

| 软硬件部件                                         | 如何影响性能                         | 讨论的章节   |
| -------------------------------------------------- | ------------------------------------ | ------------ |
| 算法(Algorithm)                                    | 决定源程序语句数量及执行I/O操作数量  | 参考相关资料 |
| 程序语言、编译器(Compiler)和体系结构(Architecture) | 决定每一条源程序语句所对应的机器指令 | 第2章        |
| 处理器(Processor)、存储器系统(Memory System)       | 决定指令的执行速度                   | 第3、4、5章  |
| I/O系统(硬件和操作系统)                            | 决定I/O操作的执行速度                | 第6章        |

## 计算机的工作过程

你看到/编写的是程序，程序的表象之下是什么？
<div class="media">
  <img src="/mse/computer/pic/8.bmp" class="mr-3" alt="Smiley face" width="140" height="200">|
</div>

{% note info Below Your Program%}
“在巴黎，我对当地人讲法语，他们只是瞪着我看；我从来没能让这些白痴理解他们自己的语言”
    ——马克.吐温，《The Innocents Abroad(异国奇遇)》，1869
{% endnote%}

我们对计算机讲什么语言，可以让计算机理解呢？

**简单的软硬件层次化结构**

<img src="/mse/computer/pic/9.bmp" alt="Smiley face" width="140">

1. 隐藏低层层次的实现细节，简化各层次上用户的使用
2. 每一层次都为上一层隐藏了自己的技术细节——“抽象”

{% note info 系统软件: 提供公共服务程序%}
{% endnote %}

| **系统软件** | **提供公共服务程序**                                         |
| ------------ | ------------------------------------------------------------ |
| 操作系统     | 用户程序和硬件的接口，提供计算机资源管理<br/>功能：处理基本的输入输出操作、分配存储空间、为多个程序同时使用计算机提供支持 |
| 编译器       | 将高级语言语句翻译成汇编语言语句的程序                       |
| 汇编器       | 汇编语言是一台计算机指令系统的符号化表示<br/>汇编器将汇编语言中的符号指令翻译成计算机能够识别的二进制指令 |

### 计算机的层次结构

Computer Hierarchy

<img src="/mse/computer/pic/10.jpg" alt="计算机的层次结构">

* 计算机系统层次结构中，指令系统是软/硬件的交界面
* 不同用户工作在不同层次，看到的计算机是不一样的

<img src="/mse/computer/pic/11.jpg" alt="计算机的层次结构">



### 从程序到电子信号

{% note info 分析：如何使用计算机求解一个问题？%}
例：计算整数18和40的和
* 需要将我们对该问题的求解要求通过输入设备告诉计算机；
* 计算机调用相应的运算部件对问题进行求解；
* 并将最后的计算机结果从输出设备输出
{% endnote %}

<img src="/mse/computer/pic/12.jpg" alt="计算机的层次结构">

（1）高级语言转换成汇编语言

```c
# 问题的高级语言描述
# 例的C语言描述 
#include <stdio.h>
int main()
{
int c;
c = 18 + 40;
printf(“result is : %d\n”, c);
}
```
当执行这个程序时，计算机内部发生了什么&为什么？

<img src="/mse/computer/pic/13.jpg" alt="从程序到电⼦子信号">

（2）汇编语言转换为机器语言

<img src="/mse/computer/pic/14.jpg" alt="从程序到电⼦子信号">

<img src="/mse/computer/pic/15.jpg" alt="从程序到电⼦子信号">

{% note warn 计算机内部工作过程%}
计算机内部工作过程:逐条执行加载到内存中的二进制机器指令流的过程。

一条指令的执行过程可简单地分为两个操作阶段：
1. 取指阶段，CPU从内存中读取指令，程序计数器保存要被取出的下一条指令的地址，除非遇到跳转指令等情况，否则，PC一般都是在每次取指后加上一个增
量（当前指令的字节数）；
2. 执行阶段，对取出的指令先译码，解释指令的功能，然后执行译码好的指令，这期间可能会读写存储器或端口来获取操作数或者存放结果。

程序的执行过程就是**周期性**和**重复性**地进行**取指令**和**执行指令**两个操作。
{% endnote %}

<img src="/mse/computer/pic/16.jpg" alt="从程序到电⼦子信号">
<img src="/mse/computer/pic/17.jpg" alt="从程序到电⼦子信号">
<img src="/mse/computer/pic/18.jpg" alt="从程序到电⼦子信号">
<img src="/mse/computer/pic/19.jpg" alt="从程序到电⼦子信号">
<img src="/mse/computer/pic/20.jpg" alt="从程序到电⼦子信号">
<img src="/mse/computer/pic/21.jpg" alt="从程序到电⼦子信号">
<img src="/mse/computer/pic/22.jpg" alt="从程序到电⼦子信号">

## 打开计算机的机箱

Under the Covers

{% note info 电⼦子数字计算机ENIAC, 1946%}
| 机器  | 器件   | 加法运算速度 | 程序指令                 | 用途                 |
| ----- | ------ | ------------ | ------------------------ | -------------------- |
| ENIAC | 电子管 | 5000次/秒    | 硬件互连(连接线)编制程序 | 专用（生产火炮射表） |
{% endnote %}

<img src="/mse/computer/pic/23.bmp" alt="Smiley face" width="200" height="150">
<img src="/mse/computer/pic/24.bmp" alt="Smiley face" width="200" height="150">
<img src="/mse/computer/pic/25.bmp" alt="Smiley face" width="200" height="150">

* ENIAC的主要缺点之一：利用开关和连接线来设置程序指令，程序设计麻烦，低效
* ENIAC设计者的思考：将程序指令存储在机器中，由它控制机器的操作？

### 存储程序思想

**关键思想：存储程序**

存储程序原理: 冯·诺依曼第一次书面提出了存储程序原理和存储程序数字计算机

{% note warn 存储程序原理%}
将事先设计好，用以描述计算机解题过程的
**程序如同数据一样**，采用**二进制形式存储**在机器中，
计算机在工作时*自动**高速地从机器中**逐条取出指令加以执行**。
{% endnote %}

计算机与计算器在存储程序方面的不同？
* 操作指令与数据被存放在存储器中
* 控制器自动取指令和数据，控制计算的执行


### 冯诺依曼框架

冯.诺依曼计算机框架

<img src="/mse/computer/pic/26.jpg" alt="冯诺依曼框架">

现代计算机的组成框图

<img src="/mse/computer/pic/27.jpg" alt="现代计算机的组成框图">

非冯.诺依曼模型
1. 神经网络:利用人脑模型思想作为计算范式
2. 基因算法：利用生物学和DNA演化思想开发算法
3. 量子计算：采用量子力学的奇妙思想解决计算问题

<img src="/mse/computer/pic/28.jpg" alt="非冯.诺依曼模型">


### 硬件组成

<img src="/mse/computer/pic/29.jpg" alt="非冯.诺依曼模型">

<img src="/mse/computer/pic/30.jpg" alt="非冯.诺依曼模型">

{% note info 功能部件1——处理器 (CPU - Central Processing Unit)%}
功能：执行程序(Execute programs)
组成：Control Unit + Data path
    * Control Unit(控制单元): 对指令进行译码，产生控制信号
    * Datapath (数据通路): 完成指令的执行
核心：ALU(Arithmetic Logic Unit)+Register(寄存器)
    * ALU用来执行算术、逻辑运算
    * 寄存器用来存储临时的数据或控制信息，例如：PC、IR等
{% endnote %}
<img src="/mse/computer/pic/31.bmp" alt="cpu">

{% note info 功能部件2——存储器%}
功能：存储程序和数据
组成(层次化结构 Hierarchies)
内存(Primary memory)： Cache + Main memory (MM)
* Cache(高速缓存)：存放最近使用的数据和指令
* 主存(Main Memory)： 存放被启动的程序中的部分数据和指令
外存(Storage)：磁盘、固态盘、光盘、磁带
磁盘(Magnetic disk)/固态盘：存放系统中所有软件和文档
光盘(Optical disk)、磁带(Tape)：脱机存档
{% endnote %}
<img src="/mse/computer/pic/32.bmp" alt="存储器" width="200" height="150">

{% note info 功能部件3——输入/输出Input/Output (I/O)%}
功能: 各种信息的输入/输出
组成: I/O Controller + I/O Device
* I/O Controller(I/O控制器): 控制外设工作，完成主机和外设之间的通信
* I/O device (I/O设备) : 输入/输出信息，包括：Keyboard、 Printer、 Display ， etcv
{% endnote %}
<img src="/mse/computer/pic/34.jpg" alt="i/o">

{% note info 总线%}
用于各功能部件之间的连接
{% endnote %}

计算机系统总线结构图
<img src="/mse/computer/pic/35.bmp" alt="总线">

常见PC总线图
<img src="/mse/computer/pic/36.bmp" alt="总线">

解剖计算机

<img src="/mse/computer/pic/37.jpg" alt="">

{% note info 思考：一个典型程序的转换处理过程%}
<img src="/mse/computer/pic/38.jpg" alt="">
{% endnote %}


## 计算机性能评价

Computer Performance Evaluation

{% note warn  如何理解“一台计算机性能比另一台好”的含义？%}
不同的性能评价标准会导致不同的结论！
{% endnote %}

### 性能的基本指标

| 衡量计算机性能的基本指标     | 指标组成                                    |
| ---------------------------- | ------------------------------------------- |
| 响应时间(Response Time)      | 执行时间(Execution Time)、等待时间(Latency) |
| 吞吐率(Throughput )          | 带宽(Bandwidth)                             |
| 指令执行速度(MIPS、MFLOPS)   |                                             |
| 功耗( Power)                 |                                             |
| 制造成本(Manufacturing Cost) |                                             |


{% note info 计算机有两种不同的性能%}
| 计算机有两种不同的性能           | 衡量指标                                                     |
| -------------------------------- | ------------------------------------------------------------ |
| Time to do the task              | 响应时间(response time)：完成单个任务所需的总时间<br/>执行时间(execution time)<br/>等待时间或时延(latency) |
| Tasks per day, hour, sec, ns. .. | 吞吐率(throughput)：单位时间内所完成的任务总量<br/>带宽(bandwidth) |
{% endnote %}


{% note info 不同应用场合用户关心的性能不同%}
* 吞吐率高的场合——多媒体应用(音/视频播放要流畅)
* 响应时间短的场合——事务处理系统(响应速度要快)
* 吞吐率高且响应时间短的场合——ATM、文件服务器、Web服务器等
{% endnote %}



{% note warn 计算机性能：响应时间(执行时间)的倒数！%}
性能x = 1 / 执行时间x
{% endnote %}

比较计算机性能时，可以用**响应(执行)时间**来衡量
* 完成同样工作量所需时间最短的那台计算机性能最好
* 处理器时间往往被多个程序共享使用，因此，用户感觉到的程序执行时间并不仅仅是程序真正的执行时间

**系统响应时间(用户感受到的)**
* CPU执行时间：CPU真正花费在程序执行上的时间：
```buildoutcfg
用户CPU时间：用来运行用户代码的时间
系统CPU时间：为执行用户程序而需运行一些操作系统代码的时间
```
* 其他时间：等待I/O操作完成或CPU花在其他用户程序的时间

{% note warn 计算机系统性能≠CPU性能%}
系统性能(System performance)：系统响应时间
CPU性能(CPU performance)：用户CPU时间
**CPU性能：CPU真正⽤用于⽤用户程序执⾏行上的时间**


系统性能 = 系统相应时间 = CPU执行时间 + 其他时间 
系统性能 = 用户CPU时间 + 系统CPU时间 + 其他时间
CPU性能 = 用户CPU时间
{% endnote %}


### CPU执行时间

Computer Performance Evaluation

{% note warn 经典的CPU性能公式%}
<img src="/mse/computer/pic/39.jpg" alt="">
{% endnote %}

评价CPU性能的最重要指标——CPU执行时间

```
一个程序的CPU执行时间 = 执行一个程序需要的(计算机)基本时间单元的数量 × 基本时间单元
一个程序的CPU执行时间 = 一个程序的CPU时钟周期数÷时钟频率

一个程序的CPU时钟周期数＝ 程序的指令数 ×每条指令的平均时钟周期数
其中：每条指令的平均时钟周期数：CPI(avg. clock Cycles Per Instr)

一个程序的CPU时间 ＝ 指令数/程序 × CPI × 时钟周期时间
```

> 时钟周期（Clock cycle）

* 所有计算机都有一个固定频率的硬件时钟，决定各种硬件事件发生和执行的时间和顺序
* 硬件时钟所产生的离散时间间隔称为时钟周期

> 时钟频率(Clock rate)

* 时钟周期的倒数

```buildoutcfg
例：时钟周期为250ps，对应的时钟频率为多少？
时钟频率＝1/时钟周期＝1/250ps＝4GHz
```


{% note info 基本的性能指标及其测量单位%}

| 性能指标              | 测量单位               |
| --------------------- | ---------------------- |
| 程序的CPU执行执行时间 | 程序执行的秒数         |
| 指令数                | 程序执行的指令数       |
| CPI                   | 每条指令平均时钟周期数 |
| 时钟周期时间          | 每个时钟周期的秒数     |
{% endnote %}



| 影响因素       | 指令数 | CPI  | 时钟周期时间 |
| -------------- | ------ | ---- | ------------ |
| 算法和编程语言 | √      | √    |              |
| 编译程序       | √      | √    |              |
| 指令集体系结构 | √      | √    | √            |
| 计算机组成     |        | √    | √            |
| 实现技术       |        |      | √            |


{% note info ISA、计算机组成(Organization)、计算机实现技术(Technology)三个因素如何相关?%}
1. 改变ISA以减少指令条数会使时钟周期变长
2. CPI的减少可能会增加时钟周期的长度
3. 缩短时钟周期可能会增加指令的条数
4. 即使是在同一台机器上解决同一个问题，最少指令条数的程序不一定执行的最快
{% endnote %}

评价性能时，必须在各个因素进行权衡！


### Amdahl定律

{% note warn 硬件设计的基本策略：使最常用的部件执行得更快%}
{% endnote %}

```buildoutcfg
Amdahl定律：
改进后的执行时间 = 受改进影响部分的执行时间/改进提高的倍数 + 不受影响的执行时间
```


{% note info 例题1%}
<img src="/mse/computer/pic/40.jpg" alt="">
{% endnote %}

{% note info 例题2%}
<img src="/mse/computer/pic/41.jpg" alt="">
{% endnote %}


> 动态功耗：CMOS中主要的能量消耗

```buildoutcfg
动态功耗主要是指晶体管翻转时消耗的能量
Power = Capacitive Load×Voltage2 ×Frequency Switched
* 开关频率是时钟速率的函数
* 负载电容是晶体管扇出数和所用技术(决定导线和晶体管)的函数

减少芯片动态功耗的方法
* 降低时钟频率
* 降低工作电压 (从5V降低到1.5V)
但是，低电压使晶体管的漏电流问题更加突出，大约30% ～ 40%的芯片功耗是由于漏电流引起的
```

> 静态功耗：即使芯片内晶体管关闭，仍有漏电流发生

```buildoutcfg
Static Power = Voltage ×Leakage Current
即使晶体管总是关闭，增加晶体管的数量也会增加功耗
```

> 除了性能，也要考察成本！

封装：将芯片固定在塑胶或陶瓷基座上，把芯片上蚀刻出来的引线与基座底部伸出的引脚连接，盖上盖板并封焊成芯片

与芯片成本有关的因素
* 圆晶价格
* 圆晶所含芯片数
* 芯片合格率

<img src="/mse/computer/pic/42.jpg" alt="">
<img src="/mse/computer/pic/43.jpg" alt="">
