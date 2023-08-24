

<center><strong><font face="微软雅黑" size=5>openEuler 进程调度</font></strong></center>

**<center><font face="华文楷体">姓名：麻家乐 &emsp;&emsp;&emsp;&emsp;学号：521021910559 </font>**
</center>

<strong>

</font>

<center><font face="华文楷体" size=5>目录</font></center>

<font face="华文楷体" size=2.5>

&emsp;<font face="华文楷体" size=3>Chapter 1 进程基本概念</font>

&emsp;&emsp;&emsp;1、什么是进程
&emsp;&emsp;&emsp;&emsp;&emsp;1.1 进程的定义
&emsp;&emsp;&emsp;&emsp;&emsp;1.2 进程与线程
&emsp;&emsp;&emsp;2、进程从创建到终止
&emsp;&emsp;&emsp;&emsp;&emsp;2.1 关系图
&emsp;&emsp;&emsp;&emsp;&emsp;2.2 进程的状态
&emsp;&emsp;&emsp;3、进程调度简单介绍
&emsp;&emsp;&emsp;&emsp;&emsp;3.1 什么是进程调度
&emsp;&emsp;&emsp;&emsp;&emsp;3.2 为什么需要进程调度

&emsp;<font face="华文楷体" size=3>Chapter 2 struct task_struct（该部分可以先调过）</font>

&emsp;&emsp;&emsp;1、结构体总体介绍
&emsp;&emsp;&emsp;1.1 为什么要介绍 struct task_struct
&emsp;&emsp;&emsp;&emsp;&emsp;1.2 struct task_struct 重要成员组
&emsp;&emsp;&emsp;3、进程与内核栈
&emsp;&emsp;&emsp;4、进程的亲属关系
&emsp;&emsp;&emsp;5、进程的标识符PID
&emsp;&emsp;&emsp;6、进程的标记flags
&emsp;&emsp;&emsp;7、进程的ptrace
&emsp;&emsp;&emsp;8、进程的统计信息
&emsp;&emsp;&emsp;9、进程的调度
&emsp;&emsp;&emsp;10、struct task_struct 总结

&emsp;<font face="华文楷体" size=3>Chapter 3 进程调度</font>

&emsp;&emsp;&emsp;1、进程的调度流程
&emsp;&emsp;&emsp;&emsp;&emsp;1.1 再次提及进程调度
&emsp;&emsp;&emsp;&emsp;&emsp;1.2 进程调度的流程
&emsp;&emsp;&emsp;&emsp;&emsp;1.3 一些补充
&emsp;&emsp;&emsp;2、struct rq
&emsp;&emsp;&emsp;3、schedule
&emsp;&emsp;&emsp;4、__schedule
&emsp;&emsp;&emsp;5、Chapter3 总结

&emsp;<font face="华文楷体" size=3>Chapter 4 CFS（完全公平调度）介绍</font>

&emsp;&emsp;&emsp;1.1 CFS基本原理概述
&emsp;&emsp;&emsp;&emsp;&emsp;1.2 CFS相关概念
&emsp;&emsp;&emsp;&emsp;&emsp;1.3 CFS算法设计核心
&emsp;&emsp;&emsp;2、struct cfs_rq
&emsp;&emsp;&emsp;3、struct sched_entity
&emsp;&emsp;&emsp;4、struct sched_class
&emsp;&emsp;&emsp;5、struct sched_class fair_sched_class
&emsp;&emsp;&emsp;6、Chapter4 总结 & 完整思维导图

&emsp;<font face="华文楷体" size=3>Chapter 5 选做部分</font>

</font>

<div STYLE="page-break-after: always;"></div>

<br>

<center><font face="华文楷体" size=5>结构体函数文件对照表</font></center>

<br>

<div><center>
<img src=对照表.png width=70% height=70% >
<br>
<strong><font face="华文楷体" size=2>结构体函数文件对照表</font>
</strong>
</center></div>

<div STYLE="page-break-after: always;"></div>

<br>

<center><font face="华文楷体" size=5>Chapter1 进程基本概念</font></center>

<br>

<font face="华文楷体" size=4>1 什么是进程</font>

<font face="华文楷体" size=3>1.1 进程的定义</font>

<font face="华文楷体" size=2.5>

【中文定义】进程是计算机中的一个具有独立功能的程序关于某数据集合上的一次运行活动，是系统进行资源分配的基本单位。
【英文定义】Process is an instance of a computer program that is being executed.

</font>

<font face="华文楷体" size=3>1.2 进程与线程</font>

<font face="华文楷体" size=2.5>

【线程的定义】线程（thread）是操作系统能够进行运算调度的最小单位。
【线程与进程】进程是操作系统资源分配的基本单位，而线程是操作系统调度的基本单位。一个进程可以包含多个线程，这些线程共享该进程的所有资源，如内存空间、文件句柄等。
【 linux 中的 task_struct 】在 linux 中并不区分进程和线程，都是用 task_struct 来抽象。支持多线程的进程是由一组 task_struct 来抽象，而这些 task_struct 会共享一些数据结构。linux 用 thread ID 来唯一标识进程中的线程，对于单线程的进程，process ID 和 thread ID是一样的，对于支持多线程的进程，每个线程有自己的 thread ID，但是所有的线程共享一个PID。
【注】task_struct 是一个庞大的结构体，本文将在 Chapter2 进行详细介绍

</font>

<font face="华文楷体" size=4>2、进程从创建到终止</font>

<font face="华文楷体" size=3>2.1 关系图</font>

<div><center>
<img src=流程图.jpg width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图1-1 进程从创建到终止</font>
</strong>
</center></div>

<font face="华文楷体" size=3>2.2 进程的状态</font>

<font face="华文楷体" size=2.5>

从流程图中可以看出，进程有以下几个状态：
【创建状态】一个新进程被创建时的第一个状态
【就绪状态】进程已经准备好并分配到所需资源，此时，只要分配到CPU就能够立即运行
【阻塞状态】进程由于等待某些事件而暂停执行，如等待I/O操作完成
【执行状态】进程处于正在运行的状态
【终止状态】进程已经完成了它的任务，或者由于某些原因被操作系统强制终止
注：以上状态只是人们为了方便理解而抽象出的五种状态，事实上在linux中并非完全如此划分

</font>

<font face="华文楷体" size=4>3、进程调度简单介绍</font>

<font face="华文楷体" size=3>3.1 什么是进程调度</font>

<font face="华文楷体" size=2.5>

&emsp;&emsp;所谓进程调度，就是指在处于就绪态的一堆进程里，按照一定的调度算法，选出一个进程并给它分配CPU时间让它运行，从而实现多进程的并发执行。

</font>

<font face="华文楷体" size=3>3.2 为什么需要进程调度</font>

<font face="华文楷体" size=2.5>

&emsp;&emsp;由于一个CPU同时只能处理一个进程，而就绪队列里面往往有大量的进程等待CPU资源，因此需要设计进程调度动态地从众多的就绪进程中选择一个最合适的进程来占用CPU资源。
&emsp;&emsp;进程调度的主要目的以及设计进程调度算法的宗旨是为了充分利用计算机系统中的CPU资源，让计算机能够多快好省的完成各种任务。

</font>

<div STYLE="page-break-after: always;"></div>

<br>

<center><font face="华文楷体" size=5>Chapter2 struct task_struct</font></center>

<br>

<font face="华文楷体" size=4>1、结构体总体介绍</font>

<font face="华文楷体" size=3>1.1 为什么要介绍 struct task_struct </font>

<font face="华文楷体" size=2.5>

&emsp;&emsp;linux内核中使用 task_struct 结构来表示一个进程，这个结构体保存了进程的所有信息，所以它非常庞大，在讲解linux内核的进程调度前，我们有必要先初步了解这个 task_struct 中的一些成员

</font>

<font face="华文楷体" size=3>1.2 struct  task_struct 主要成员组 </font>

<div><center>
<img src=task_struct.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-1 struct  task_struct 主要成员组</font>
</strong>
</center></div>

<font face="华文楷体" size=4>2、进程状态</font>

<font face="华文楷体" size=3>2.1 思维导图 </font>

<div><center>
<img src=进程状态.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-2 进程状态思维导图 </font>
</strong>
</center></div>

<font face="华文楷体" size=3>2.2 源代码 </font>

<div><center>
<img src=state.png width=80% height=80% >
<br>
<img src=_state.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-3 进程状态与宏定义源代码  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>2.3 详细介绍 </font>

<font face="华文楷体" size=2.5>

&emsp;&emsp;task_struct 中表示进程状态的变量是 state ，它是一个 long 数据类型的变量。
&emsp;&emsp;当 state = 0 时，表示进程处于 TASK_RUNNING 状态（runnable）。这表示进程可以运行，或者正在运行，或者在运行队列中等待运行。至于如何判断是否正在运行，task_struct 并没有给出相应的变量，不过可以根据CPU上的队列中的指针 *curr来判断。
&emsp;&emsp;当 state = -1 时，表示进程不可运行（unrunnale）。这通常是因为进程被冻结了或正在退出，此时需要通过 task_struct 的标记符号 flags 的值来判断进程所处的状态（后面会提及）。
&emsp;&emsp;当 state > 0 时，表示进程正处于非运行状态（stopped）。非运行状态是一系列状态的集合，它包括TASK_INTERRUPTIBLE、TASK_UNINTERRUPTIBLE 等状态，详细见下图：

<div><center>
<img src=stopped.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-4 stopped 状态</font>
</strong>
</center></div>

</font>

<font face="华文楷体" size=4>3、进程与内核栈</font>

<font face="华文楷体" size=3>3.1 思维导图 </font>

<div><center>
<img src=进程与内核栈.png width=70% height=80% >
<br>
<img src=进程与内核栈2.png width=70% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-5 进程与进程内核栈思维导图  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>3.2 源代码 </font>

<div><center>
<img src=stack.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-6 进程与进程内核栈源代码  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>3.3 详细介绍 </font>

<font face="华文楷体" size=2.5>

&emsp;&emsp;内核在创建进程的时候，会为进程创建相应的堆栈。每个进程会有两个栈，一个用户栈，存在于用户空间，一个内核栈，存在于内核空间。当进程在用户空间运行时，CPU堆栈指针寄存器里面的内容是用户堆栈地址，使用用户栈；当进程在内核空间时，CPU堆栈指针寄存器里面的内容是内核栈空间地址，使用内核栈。
&emsp;&emsp;当进程因为中断或者系统调用而陷入内核态之行时，进程所使用的堆栈也要从用户栈转到内核栈。进程进入内核态后，先把用户态堆栈的地址保存在内核栈之中，然后设置堆栈指针寄存器的内容为内核栈的地址，这样就完成了用户栈向内核栈的转换。当进程从内核态恢复到用户态时，在内核态执行的最后将保存在内核栈里面的用户栈的地址恢复到堆栈指针寄存器即可。这样就实现了内核栈和用户栈的互转。
&emsp;&emsp;那么，我们知道从内核转到用户态时用户栈的地址是在陷入内核的时候保存在内核栈里面的，但是在陷入内核的时候，我们是如何知道内核栈的地址的呢？
关键在进程从用户态转到内核态的时候，进程的内核栈总是空的。这是因为，当进程在用户态运行时，使用的是用户栈，当进程陷入到内核态时，内核栈保存进程在内核态运行的相关信息，但是一旦进程返回到用户态后，内核栈中保存的信息无效，会全部恢复，因此每次进程从用户态陷入内核的时候得到的内核栈都是空的。所以在进程陷入内核的时候，直接把内核栈的栈底地址给堆栈指针寄存器就可以了。
&emsp;&emsp;讲了这么多关于进程内核栈理论方面的内容，那么liunx具体是如何实现二者之间的切换的呢？
&emsp;&emsp;事实上，task_struct 结构体为实现从用户态转到内核态创建了一个 *stack 的指针，通过这个指针（经过一系列函数与计算）就能访问进程的内核栈。而为了实现从内核态转到用户态，linux 在设计内核栈时，事实上创建的是 union thread_union，如下图所示：

<div><center>
<img src=thread_union.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-7 thread_union 源代码  </font>
</strong>
</center></div>

&emsp;&emsp;引入 union 而不是使用 struct 结构是为了让内核栈与 thread_info 位于同一块内存。由于 thread_info 中存在一个 struct task_struct *stack 指针，因此，内核态可以通过调用与其处于同一内存地址下的 thread_info 中的 *stack 指针来实现从内核态转到用户态。


<div><center>
<img src=arch下的thread_info.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-8 arch下thread_info 源代码  </font>
</strong>
</center></div>

&emsp;&emsp;需要注意的是，不同架构的 thread_info 结构体包含的内容是不尽相同的。linux 支持的CPU架构有很多，本实验环境下可以通过访问arch文件夹查看。

<div><center>
<img src=arch.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-9 支持的架构  </font>
</strong>
</center></div>


&emsp;&emsp;事实上，有一些架构中的 thread_info 并没有 *stack指针，例如在 arm64 平台上，thread_info 结构体并不包含指向 task_struct 的结构体指针。这是因为，在 arm64 平台上，内核态到用户态的转换是通过 el0 和 el1 两个特权级别来实现的。当进程从用户态切换到内核态时，会从 el0 切换到 el1，然后执行内核代码。当内核代码执行完毕后，会从 el1 切换回到 el0，然后返回用户态。

<div><center>
<img src=arm64下的thread_info.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-10 arm64下thread_info 源代码  </font>
</strong>
</center></div>

</font>

<br>
<font face="华文楷体" size=4>4、进程的亲属关系</font>

<font face="华文楷体" size=3>4.1 思维导图 </font>

<div><center>
<img src=亲属关系.png width=70% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-11 进程的亲属关系思维导图 </font>
</strong>
</center></div>

<font face="华文楷体" size=3>4.2 源代码 </font>

<div><center>
<img src=family.png width=90% height=90% >
<br>
<strong><font face="华文楷体" size=2>图2-12 进程的亲属关系源代码  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>4.3 详细介绍 </font>

<font face="华文楷体" size=2.5> 

- *real_parent：指向创建当前进程的父进程的task_struct指针

- *parent：通常和real_parent相同，但在某些情况下，例如进程被ptrace跟踪时，会被修改为跟踪进程。

- children：链表，包含当前进程的所有子进程的task_struct指针

- sibling：链表，包含当前进程的所有兄弟进程的task_struct指针

- *group_leader：指向当前进程所属线程组的首个线程的task_struct指针

</font>

<font face="华文楷体" size=4>5、进程的标识符 PID</font>

<font face="华文楷体" size=3>5.1 思维导图 </font>

<div><center>
<img src=标识符.png width=60% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-13 进程的标识符思维导图 </font>
</strong>
</center></div>

<font face="华文楷体" size=3>5.2 源代码 </font>

<div><center>
<img src=pid.png width=50% height=90% >
<br>
<strong><font face="华文楷体" size=2>图2-14 进程的标识符源代码  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>5.3 详细介绍 </font>

<font face="华文楷体" size=2.5> 

- pid Process ID，进程号，唯一标识一个进程的task_struct 结构体的数字。
- tgid Thread Group ID，线程组ID，标识一个进程中所有线程的 task_struct 结构体的数字。

- 【重点】创建一个新的进程会有新的PID和TGID，并且2个值相同；创建一个新的线程的时候，也会有新的PID，但它的 tgid 与创建它的进程的 tgid 一致，这样，内核就可以通过 tgid 知道某个 task 属于哪个线程组，也就知道属于哪个进程了。当使用 ps 命令或者 getpid() 等接口查询进程 id 时，内核返回的是 tgid 

<div><center>
<img src=pid-tgid.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-15 pid 与 tgid  </font>
</strong>
</center></div>

</font>

<font face="华文楷体" size=4>6、进程的标记 flags</font>

<font face="华文楷体" size=3>6.1 思维导图 </font>

<div><center>
<img src=标记.png width=60% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-16 进程的标记思维导图 </font>
</strong>
</center></div>

<font face="华文楷体" size=3>6.2 源代码 </font>

<div><center>
<img src=flag.png width=70% height=70% >
<br>
<img src=flags.png width=70% height=70% >
<br>
<strong><font face="华文楷体" size=2>图2-17 进程的标记及其宏定义源代码  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>6.3 详细介绍 </font>

<font face="华文楷体" size=2.5> 

&emsp;&emsp;flags有很多种可能的取值，分别代表不同的状态，每种值所对应的状态如上图宏定义所示，此外，与state类似，flags对应的状态也是可以多种状态的叠加

</font>


<font face="华文楷体" size=4>7、进程的 ptrace </font>

<font face="华文楷体" size=3>7.1 思维导图 </font>

<div><center>
<img src=ptrace.png width=70% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-18 进程的 ptrace 思维导图 </font>
</strong>
</center></div>

<font face="华文楷体" size=3>7.2 源代码 </font>

<div><center>
<img src=flag.png width=70% height=70% >
<br>
<img src=ptrace2.png width=70% height=70% >
<br>
<img src=ptrace3.png width=70% height=70% >
<br>
<strong><font face="华文楷体" size=2>图2-19 进程的 ptrace 源代码  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>7.3 详细介绍 </font>

<font face="华文楷体" size=2.5> 

- ptrace是一个系统调用，它可以让一个进程（跟踪者）观察和控制另一个进程（被跟踪者）的执行，并检查和修改被跟踪者的内存和寄存器。它主要用于实现断点调试和系统调用跟踪。要使用ptrace，首先需要将被跟踪者附加到跟踪者上。
- ptrace：整数值，记录当前进程是否被跟踪，以及跟踪的模式和选项
- ptrace_message：用于传递ptrace事件的附加信息
- ptraced：一个链表，包含当前进程跟踪的所有进程的task_struct指针
- ptrace_entry：一个链表节点，用于将当前进程加入到跟踪者的ptraced链表中

</font>

<font face="华文楷体" size=4>8、进程的统计信息 </font>

<font face="华文楷体" size=3>8.1 思维导图 </font>

<div><center>
<img src=统计信息.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-20 进程的统计信息思维导图 </font>
</strong>
</center></div>

<font face="华文楷体" size=3>8.2 源代码 </font>

<div><center>
<img src=information.png width=70% height=70% >
<br>
<img src=information2.png width=70% height=70% >
<br>
<strong><font face="华文楷体" size=2>图2-21 进程的统计信息源代码  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>8.3 详细介绍 </font>

<font face="华文楷体" size=2.5> 

（一）Time

- utime：user time 用户态消耗CPU时间
- stime：内核态消耗CPU时间
- gtime：虚拟机运行时间
- start_time：进程的启动时间（不包含睡眠）
- real_start_time：进程的启动时间（包含睡眠）
- real_start_time = start_time + 总的睡眠时间
 
（二）Context Switches

- nvcsw：Number of Voluntary Context Switches，自愿（主动）上下文切换，指进程主动放弃CPU，例如等待I/O操作完成。
- long nivcsw：Number of Involuntary Context Switches，非主动（自愿）上下文切换次数，指进程被迫放弃CPU，例如时间片用完或者被更高优先级的进程抢占。
  
（三）Page Fault
&emsp;&emsp;当进程访问它的虚拟地址空间中的PAGE时，如果这个PAGE目前还不在物理内存中，Linux会产生一个hard page fault中断。
&emsp;&emsp;此时，系统需要做以下几件事，然后进程才能访问这部分虚拟地址空间的内存。
&emsp;&emsp;（1）从慢速设备（如磁盘）将对应的数据PAGE读入物理内存
&emsp;&emsp;（2）建立物理内存地址与虚拟地址空间PAGE的映射关系。

- maj_flt：major page fault，也称为hard page fault, 指需要访问的内存不在虚拟地址空间，也不在物理内存中，因此，系统需要完成上述的（1）与（2）。
注：从swap回到物理内存也是hard page fault。

- min_flt：minor page fault，也称为soft page fault, 指需要访问的内存不在虚拟地址空间，但是在物理内存中。此时，系统只需要完成上述的（2）即可。
注：通常是多个进程访问同一个共享内存中的数据，可能某些进程还没有建立起映射关系，所以访问时会出现 soft page fault

</font>

<font face="华文楷体" size=4>9、进程的调度 </font>

<font face="华文楷体" size=3>9.1 思维导图 </font>

<div><center>
<img src=进程调度.png width=90% height=90% >
<br>
<strong><font face="华文楷体" size=2>图2-22 进程的调度思维导图 </font>
</strong>
</center></div>

<font face="华文楷体" size=3>9.2 源代码 </font>

<div><center>
<img src=进程调度的源代码.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-23 进程调度部分的源代码  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>9.3 详细介绍 </font>

<font face="华文楷体" size=2.5> 

（一）priority

- prio：进程动态优先级，是调度器使用的优先级，它依赖于normal_prio，调度器在运行期间可以根据倾向性调整该值
- static_prio，普通进程的优先级，仅能通过nice修改，nice取值为[-20,19]，static_prio = nice + 120，因此 static_prio 的取值范围为[100,139]。static_prio的值越小优先级越高。
- rt_priority，实时进程的优先级，取值为[0,99]，【注意】rt_priority的值越大优先级越大！
- normal_prio，归一化优先级/正常优先级，normal_prio值是根据调度器类型计算出来的，对于实时进程：normal_prio = 99 - rt_priority，对于非实时进程: normal_prio = static_prio

（二）policy

&emsp;&emsp;在 task_struct 中，policy表示当前进程被调度时所采取的调度策略。调度策略有以下几种：

- SCHED_NORMAL：普通非实时任务的调度策略，它使用完全公平调度（CFS）算法，根据任务的优先级（nice值）和运行时间来分配CPU时间。
- SCHED_FIFO，实时调度策略，使用先进先出（FIFO）算法，即按照任务到达的顺序依次执行，直到任务完成或被更高优先级的任务抢占。该策略不使用时间片，所以同一优先级的任务不会相互抢占。
- SCHED_RR：实时调度策略，使用轮转（RR）算法，即每个任务都有一个固定的时间片，在时间片用完后，就让出CPU给同一优先级的下一个任务。该策略可以保证同一优先级的任务都有公平的机会执行。
- SCHED_BATCH：批处理调度策略，也使用CFS算法，但是抢占的机会比普通任务少，会减少被调度的次数。该策略适合那些批处理工作，比如大量的计算或数据处理。
- SCHED_IDLE：空闲调度策略，也使用CFS算法，但是优先级最低，只有在没有其他任务可以运行时才会执行。该策略适合那些不紧急的后台任务，比如索引或备份。
- SCHED_DEADLINE：基于截止时间的实时调度策略，使用EDF和CBS算法，支持资源预留。每个任务都有一个预算和一个周期，对应于一个重复的请求-服务模型。该策略可以保证每个任务在其截止时间之前完成其预算，否则会产生截止时间超时。该策略适用于有硬实时或软实时需求的任务。

&emsp;&emsp;从源代码中还可以看到：policy 是整型数据，事实上，policy可以取 0，1，2，3，5；这些数字分别对应的调度策略如下：

<div><center>
<img src=调度策略.png width=70% height=70% >
<br>
<strong><font face="华文楷体" size=2>图2-24 调度策略  </font>
</strong>
</center></div>

（三）sched_class
&emsp;&emsp;上述介绍了6种调度策略，事实上，每一种调度策略都会有响应的调度器类，linux共设计了5种调度器类，按照优先级从高到低的顺序如下：

- stop_sched_class：最高优先级的调度器类，用于停止CPU上的其他进程
- dl_sched_class：（deadline）用于实现 SCHED_DEADLINE 调度策略的调度器类
- rt_sched_class：（real time）用于实现 SCHED_FIFO 和 SCHED_RR 调度策略的调度器类
- fair_sched_class：用于实现 SCHED_NORMAL 和 SCHED_BATCH 调度策略的调度器类，也称为完全公平调度器CFS
- idle_sched_class：最低优先级的调度器类，用于实现 SCHED_IDLE 调度策略

&emsp;&emsp;注：task_struct 中 struct sched_class *sched_class 是指向调度器类的指针，struct sched_class 是linux内核抽象出来的一种调度类，是将各种调度器的共性的特征抽象出来封装成的类，关于它的具体信息我们将在下一章节介绍。

（四）sched_entity

- struct sched_entity se ： 当采用 cfs 调度器类调度该进程时，调用的是该调度实体 se
- struct sched_re_entity rt：当采用 rt 调度器类调度该进程时，调用的是该调度实体 rt
- struct sched_dl_entity dl：当采用 dl 调度器类调度该进程时，调用的是该调度实体 dl

&emsp;&emsp;至于stop与idle，由于CPU就绪队列中只有一个stop与idle类型的进程，所以调用时直接调用响应的进程即可，因此无需为这两种类型设计专门的调度实体。
&emsp;&emsp;此外，还需要说明的是，一个进程之所以有3个调度实体，本质上是为了在进程切换调度策略时，可以快速地将其在原先调度器的调度信息转化为当前调度器中的调度信息。
&emsp;&emsp;例如，当一个进程在SCHED_NORMAL和SCHED_FIFO之间切换时，就需要有一个se（sched_entity）和一个rt（sched_rt_entity）来分别存储它在完全公平调度器类和实时调度器类的信息，比如虚拟运行时间、优先级、时间片等。这样，当进程切换调度策略时，就可以快速地恢复它在原来的调度器类中的状态，而不需要重新计算或分配。

</font>

<font face="华文楷体" size=4>10、struct task_struct 总结 </font>

<font face="华文楷体" size=2.5> 

&emsp;&emsp;在 Chatper2 部分，我主要介绍了关于struct task_struct 结构体变量功能的介绍，将其划分为8个部分
- 进程状态【state 关于 0 进行划分成三种基本状态，非0时可再根据flags 或 数值进行进一步确定】      
- 进程与内核栈【进程通过 *stack 访问内核栈、内核栈通过与其同内存块的thread_info中的 *task 返回进程】     
- 进程的亲属关系【父亲、孩子、兄弟、领队】
- 进程的标识符PID【创建进程：新的pid且tgid=pid、创建线程：新的pid且tgid=创建它的进程的tgid】
- 进程的标记flags【进程的各种状态，类似于日志】
- 进程的ptrace【Debug相关】
- 进程的统计信息【运行时间、上下文切换、缺页中断】
- 进程调度【优先级、调度策略、调度器类】

&emsp;&emsp;事实上，struct task_struct 要比我所列出的这8个部分复杂的多，在这里不做进一步的赘述了，有兴趣的可以参考以下资料

【资料1】https://blog.csdn.net/Quinn0918/article/details/70148159

【资料2】https://blog.csdn.net/m0_74282605/article/details/130034516

【资料3】http://husharp.today/2020/12/06/task-struct-01/
<div><center>
<img src=网图1.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图2-25 资料3提供的图表  </font>
</strong>
</center></div>

</font>

<br>

<div STYLE="page-break-after: always;"></div>

<br>

<center><font face="华文楷体" size=5> Chapter3 进程调度 </font></center>

<br>

<font face="华文楷体" size=4>1、进程的调度流程 </font>

<font face="华文楷体" size=3>1.1 再次提及进程调度 </font>

<font face="华文楷体" size=2.5> 

&emsp;&emsp;在Chatpter1中，我们曾提到过：由于一个CPU同时只能处理一个进程，而就绪队列里面往往有大量的进程等待CPU资源，因此需要设计进程调度动态地从众多的就绪进程中选择一个最合适的进程来占用CPU资源。
&emsp;&emsp;可能现在你还在因为 struct task_struct 的庞杂而困惑，但请把 Chapter2 中的一切都先放在一遍(只要明白它是一个代表进程的结构体就好了，其他的先不去管它)，你现在需要关注的是上一段中的这几个关键字："CPU"、"就绪队列"、"进程"、"调度"。
&emsp;&emsp;事实上，这几个关键字就构成了进程调度的基本概念。它们分别对应着以下的结构体或函数：
&emsp;&emsp;" 队列 " —— struct rq
&emsp;&emsp;" 进程 " —— struct task_struct
&emsp;&emsp;" 调度 " —— schedule

</font>

<font face="华文楷体" size=3>1.2 进程调度的流程 </font>

<font face="华文楷体" size=2.5> 

&emsp;&emsp;很多时候，我们往往因为过度地在意一些细节而使自己陷入困扰，比如你去linux内核里查看有关进程调度的代码时，刚开始就想让自己理清楚那几万行代码的作用是完全不现实的，反而会使自己的大脑一团糟。事实上，我们不用把进程的调度想得有多复杂，只需要先抓住主干再去延伸到分支便可。从整体分析进程的调度，其流程将会很简单：
&emsp;（1）当前CPU内有一个正在使用CPU资源的进程（记为current）以及一个等待使用CPU资源的一些进程组成的 "就绪队列"。
&emsp;（2）硬件电路中有一个硬件定时器，它负责周期性的产生时钟中断（一般为10ms），我们称它为滴答定时器，可以认为，它就是操作系统的心脏。每当产生定时器中断的时候，CPU就会执行中断处理程序。
&emsp;（3）在滴答定时器的中断处理中，我们会通过 "schedule" 函数来判断 current 进程是否需要被抢占，若被抢占， "就绪队列" 中的哪一个进程将使用CPU资源。
&emsp;&emsp;以上就是最基本的进程调度的流程了，说白了就是在每次中断时调用 schedule 函数。


</font>

<font face="华文楷体" size=3>1.3 一些补充 </font>

<font face="华文楷体" size=2.5> 

&emsp;&emsp;可能会有人会产生这样的疑问：好像上面所讲的只是提及了处于就绪队列的进程，这些进程都处于正在运行或等待运行的状态（即state = 0）,那么其他状态的进程在哪里呢？
&emsp;&emsp;事实上，处在阻塞状态的进程并不在运行队列（就绪队列）中，而是处于一个等待队列中。这些进程都在等待某些事件的发生，例如等待I/O操作完成或等待信号量。只有当这些事件发生并完成时，内核会将进程从等待队列中唤醒并将其放回就绪队列中。
&emsp;&emsp;注：等待队列位于内核中。“内核”指的是一个提供硬件抽象层、磁盘及文件系统控制、多任务等功能的系统软件，内核常驻于内存，负责处理各种各样的核心任务，比如I/O、进程管理、内存管理等。（更多关于内核方面的知识请自行查阅相关资料）

</font>


<font face="华文楷体" size=4>2、strcut rq </font>

<font face="华文楷体" size=3>2.1 思维导图 </font>

<div><center>
<img src=rq思维导图.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图3-1 struct rq 重要部分思维导图 </font>
</strong>
</center></div>

<font face="华文楷体" size=3>2.2 源代码 </font>

<div><center>
<img src=struct_rq1.png width=70% height=70% >
<br>
<img src=struct_rq2.png width=70% height=70% >
<br>
<img src=struct_rq3.png width=70% height=70% >
<br>
<strong><font face="华文楷体" size=2>图3-2 struct rq 重要部分源代码  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>2.3 详细介绍 </font>

<font face="华文楷体" size=2.5> 

&emsp;&emsp;每个CPU都会有属于自己的队列rq，该队列包含了所有正在CPU上进行的以及等待该CPU分配资源的进程。根据进程的性质（优先级等），又将这些进程进行分组。结构体struct rq中就记录了分组的情况。
&emsp;&emsp;dl是deadline调度类的运行队列，优先级很高；rt 是实时调度运行队列，该队列中的进程都是实时进程，优先级较高 [0 - 99]；cfs 是cfs调度运行队列，该队列的进程都是普通进程，优先级较低 [100 -139]。
&emsp;&emsp;*curr 是指向当前使用CPU进程的指针， *idle指向的是一个空闲的idle线程，Linux系统初始化时会为每个cpu创建一个idle线程，当没有其他进程需要运行的时候，便运行idle线程； *stop指向的是stop调度类的进程。

</font>

<font face="华文楷体" size=4>3、schedule </font>

<font face="华文楷体" size=3>3.1 源代码 </font>

<div><center>
<img src = schedule.png width=70% height=70% >
<br>
<strong><font face="华文楷体" size=2>图3-3 schedule 源代码  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>3.2 代码分析 </font>

<font face="华文楷体" size=2.5> 

（1）struct task_struct *tsk = current; &emsp;// 定义一个指向当前进程的指针tsk
（2）sched_submit_work(tsk); &emsp;// 将当前进程放入调度队列
（3）do { ... } while (need_resched()); &emsp;// 循环执行直到不需要调度
（4）preempt_disable(); &emsp;// 禁止抢占
（5）__schedule(false); &emsp;// 选择下一个进程运行，schedule 函数真正的核心
（6）sched_preempt_enable_no_resched(); &emsp;// 允许抢占但不进行调度
（7）sched_update_worker(tsk)； &emsp;// 更新当前进程的调度状态，包括调度策略、优先级等信息

&emsp;&emsp;schedule( ) 函数只是个外层的封装，实际调用的还是 __schedule( ) 函数。 __schedule( ) 接受一个参数，该参数为 bool 型，false 表示非抢占，自愿调度，而 true 则相反.
&emsp;&emsp;通过源代码可以发现，在调用__schedule( )前是需要关闭抢占的。实际上，在__schedule中会去检查当前进程的抢占计数(位于schedule_debug函数)，确保此次调度是在关闭抢占的情况下进行的，且不能是在中断或原子上下文发生的调用。至于关闭中断，该操作将在__schedule内部完成。
&emsp;&emsp;在调度过程完成之后，即 __schedule 返回后，需要重新打开抢占。

</font>

<font face="华文楷体" size=4>4、__schedule </font>

<font face="华文楷体" size=3>4.1 变量定义与前期准备 </font>

<div><center>
<img src = __schedule第一部分.png width=70% height=70% >
<br>
<strong><font face="华文楷体" size=2>图3-4 变量定义与前期准备  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>4.2 当前运行的进程主动发起调度 </font>

<div><center>
<img src = __schedule第二部分.png width=70% height=70% >
<br>
<strong><font face="华文楷体" size=2>图3-5 当前运行的进程主动发起调度  </font>
</strong>
</center></div>

<font face="华文楷体" size=2.5> 

&emsp;&emsp;preempt 用于判断本次调度是否为抢占调度，如果发生了调度抢占（preempt =1），那么直接跳过，不参与判断，直接调用pick_next_task。抢占调度通常都是处于运行态的任务发起的抢占调度。
&emsp;&emsp;如果本次调度不是抢占调度（preempt = 0），并且该进程的state不等于 TASK_RUNNING （0），也就是不是运行态，处于其他状态，代表此次调度是该进程主动请求调度，主动调用了schedule函数，比如该进程进入了阻塞态。
&emsp;&emsp;如果该进程还有未处理信号，那就需要先处理信号。此时需要将当前进程的状态设置回 TASK_RUNNING，这种情况下，当前进程会重新参与调度，有很大概率选取到的下一个进程依旧是当前进程，从而不执行实际的进程切换。
&emsp;&emsp;若该进程没有未处理的信号，由于进程不是运行态，那么就不能在CFS就绪队列中了，于是就调用 deactivate_task 函数将陷入阻塞态的进程移除CFS就绪队列，并将进程调度实体的 on_rq 成员置0，表示不在CFS就绪队列中了。

</font>

<font face="华文楷体" size=3>4.3  选择下一个将要执行的任务</font>

<div><center>
<img src = __schedule第三部分.png width=70% height=70% >
<br>
<strong><font face="华文楷体" size=2>图3-6 选择下一个将要执行的任务  </font>
</strong>
</center></div>

<font face="华文楷体" size=2.5> 

&emsp;&emsp;选择下一个将要执行的任务的核心函数是pick_next_task( )，该函数所对应的源代码如下所示：

<div><center>
<img src = pick_next_task1.png width=70% height=70% >
<br>
<strong><font face="华文楷体" size=2>图3-7 pick_next_task 源代码第一部分  </font>
</strong>
</center></div>

&emsp;&emsp;linux 认为：当前进程属于 cfs 调度器类或 idle 调度器类是大概率事件。因此，pick_next_task 首先先进行判定当前进程是否属于 cfs 调度器类或 idle 调度器类。
&emsp;&emsp;若否，说明当前进程属于更高级别的调度器类，因此直接调过该部分代码；若是，则进而判断此时CPU运行队列中是否只存在属于cfs调度器类的进程（ nr_running = 所有进程数量；cfs.h_nr_running = cfs调度器类的所有进程数量，若只存在属于cfs调度器类的进程，则 nr_running = cfs.h_nr_running ）。
&emsp;&emsp;若否，说明队列中存在比普通进程更高级别的进程，比如实时进程，此时同样直接调过该部分代码；若是，则说明下一个进程大概率属于cfs调度器类（只有当cfs.h_nr_running = 0时才会进行空闲进程）。
&emsp;&emsp;满足了上述的两个条件后，选择下一个进程的方式就可以直接调用 cfs 调度器类的子函数 fair_sched_class.pick_next_task( )。该函数将在 Chapter4 中进行详细讨论
&emsp;&emsp;假如 fair_sched_class.pick_next_task( ) 函数的返回值为 RETRY_TASK，将指示调用者从高优先级调度类里面选取目标进程（该部分在again后面，所以使用goto  again 直接跳到 again 后面），若返回 NULL，将指示调用者从低优先级调度类（即idle）里面获取目标进程，所以使用 idle 调度器类的子函数 idle_sched_class.pick_next_task( )选择下一个进程。

<div><center>
<img src = pick_next_task2.png width=70% height=70% >
<br>
<strong><font face="华文楷体" size=2>图3-8 pick_next_task 源代码第二部分  </font>
</strong>
</center></div>

&emsp;&emsp;如果确定下一个进程不在 cfs_rq 中选，那就需要依据优先级（stop -> deadline -> realtime -> cfs_rq -> idle）对所有的调度器类进行遍历，找到一个进程之后就返回该进程。

</font>

<font face="华文楷体" size=3>4.4  执行进程切换与收尾工作 </font>

<div><center>
<img src = __schedule第四部分.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图3-9 执行进程切换与收尾工作  </font>
</strong>
</center></div>

<font face="华文楷体" size=2.5> 

&emsp;&emsp;这段代码是一个进程调度的另一个关键步骤。此时我们已经通过第三部分获得了下一进程，若当前进程与下一个进程不相同，就会增加 nr_switches计数器 （上下文交换次数），然后调用context_switch( )函数进行上下文切换。否则，就会更新clock_update_flags标志位，并释放锁。
&emsp;&emsp;很明显，该步骤中最重要的一个函数就是 context_switch( )函数，该函数的源代码如下，本文不对该代码进行进一步解释，有了解需要的请自行查阅相关资料。

<div><center>
<img src = context_switch.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图3-10 context_switch 源代码  </font>
</strong>
</center></div>

</font>

<font face="华文楷体" size=4>5、Chapter3 总结 </font>

<font face="华文楷体" size=2.5> 

&emsp;&emsp;Chapter3 主要介绍了进程调度的大体流程，详细介绍了与进程调度直接相关的运行队列（rq）以及调度函数（schedule以及__schedule）。
&emsp;&emsp;相信阅读到这里的你已经对进程调度已经有了基本概念，明白了进程调度本质上是在每个滴答定时器中断时，使用调度函数在运行队列中寻找下一个使用CPU资源的进程并执行进程切换。在下一章中，我将详细介绍运行队列中的cfs队列以及与调度相关的调度器和其中的cfs调度器。

</font>

<div STYLE="page-break-after: always;"></div>

<br>

<center><font face="华文楷体" size=5>Chapter4 CFS（完全公平调度）介绍 </font></center>

<br>

<font face="华文楷体" size=4>1、什么是CFS </font>

<font face="华文楷体" size=3>1.1、CFS基本原理概述 </font>

<font face="华文楷体" size=2.5>

&emsp;&emsp;采用cfs策略的进程队列中的每一个进程都会设置一个虚拟时钟 —— virtual runtime(vruntime)。如果一个进程得以执行，随着执行时间的不断增长，其vruntime也将不断增大，没有得到执行的进程vruntime将保持不变。
&emsp;&emsp;而调度器将会选择最小的vruntime那个进程来执行。这就是所谓的“完全公平”。不同优先级的进程其vruntime增长速度不同，优先级高的进程vruntime增长得慢，所以它可能得到更多的运行机会。

</font>

<font face="华文楷体" size=3>1.2、CFS相关概念

<font face="华文楷体" size=2.5>

- schedule：总的调度函数
- __schedule：调度函数中的核心部分，也是真正的调度函数
- sched_class：总的调度器类，fair_sched_class是它的一个实例化
- fair_sched_class：采用CFS算法（策略）的调度器类
- struct rq：CPU运行（就绪）队列，每个rq队列都包含一个cfs_rq
- struct cfs_rq：采用CFS算法（策略）的运行队列，里面所有的进程都是普通进程，优先级均在[100 - 139] 
- struct task_struct：进程对应的结构体
- struct sched_entity：进程中对应于CFS算法（策略）的调度实体，保存了进程在被CFS调度器调度时的调度信息

</font>

<font face="华文楷体" size=3>1.3、CFS算法设计核心 </font>

<font face="华文楷体" size=2.5>

&emsp;&emsp;cfs调度器每次都会选择最小的vruntime那个进程作为下一个使用CPU资源的进程，那么vruntime该如何计算呢？又如何实现优先级高的进程vruntime增长得慢呢？
&emsp;&emsp;事实上，linux采用如下公式计算 vruntime，优先级越大意味着权重越大，因此采用倒数的方式可以实现优先级优先级高的进程 vruntime 增长得慢。

$$ vruntime = realtime * \frac{NICE\_0\_LOAD}{weight}    $$

&emsp;&emsp;其中，realtime 表示的是真实运行时间，NICE_0_LOAD 表示的是 Nice 值为0对应的权重，weight 表示的是当前进程的权重。

</font>

<font face="华文楷体" size=4>2、struct cfs_rq </font>

<font face="华文楷体" size=3>2.1、思维导图 </font>

<div><center>
<img src = cfs_rq.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图4-1 cfs_rq 思维导图  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>2.2、源代码 </font>

<div><center>
<img src = cfs_rq1.png width=80% height=80% >
<br>
<img src = cfs_rq2.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图3-10 cfs_rq 源代码  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>2.3、详细介绍 </font>

<font face="华文楷体" size=2.5>

- load：cfs_rq上所有调度实体的负载权重之和
- runnable_weight：cfs_rq上所有调度实体的运行权重之和
注1：对应单个调度实体，task se 的 load weight 和 runnable weight 是相等的，但是对于调度组 group se 而言，这两个值是不同的。
注2：runnable_weight的计算涉及到了调度组的概念以及PELT算法，本文暂时不作进一步说明，有需要的请参考下方链接

参考：https://blog.csdn.net/Rong_Toa/article/details/108598418

- nr_running：cfs_rq中第一层的调度实体的数量，也就是红黑树中的节点数。这些调度实体可能是进程或者调度组，但是不包括调度组内的子调度实体
- h_nr_running：cfs_rq中所有调度实体的数量，也就是红黑树中的节点数加上每个调度组节点对应的子cfs_rq中的节点数。
  
注1：假如 cf_rq 中有1个调度组A和2个调度实体，其中调度组A包含了3个调度实体，则nr_running = 3，h_nr_running = 5
注2：可以根据nr_running 与 h_nr_running 是否相等来判断 cfs_rq 中是否有调度组
注3：cfs_rq中的调度组是通过task_group结构表示的，调度组的实质是：调度时候不再以进程作为调度实体，而是以进程组作为调度实体。虽然task group作为一个调度实体来竞争CPU资源，但是task group是一组task se或者group se (task group可以嵌套)的集合，不可能在一个CPU上进行调度，因此task group的调度实际上在各个CPU上都会发生，

- exec_clock：CFS就绪队列的总运行时间
- min_vruntime：CFS就绪队列的红黑树中最左边节点的 vruntime
- task_timeline：红黑树，用于链接调度实体，以调度实体的vruntime 作为关键字排序
- *curr：指向当前正在运行的调度实体的指针
- *next：指向下一个要运行的调度实体的指针
- *last：指向上一个运行过的调度实体的指针
- *skip：指向调过的调度实体的指针

<font face="华文楷体" size=4>3、struct sched_entity </font>

<font face="华文楷体" size=3>3.1、思维导图 </font>

<div><center>
<img src = sched_entity思维导图.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图4-3 struct sched_entity 思维导图  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>3.2、源代码 </font>

<div><center>
<img src = sched_entity.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图4-4 cfs_rq 源代码  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>3.3、详细介绍 </font>

<font face="华文楷体" size=2.5>

- load：调度实体的负载权重
- runnable_weight：调度实体的运行权重
- run_node：红黑树的数据节点，使用该rb_node将当前节点挂到红黑树上面，红黑树用于存储就绪队列中的调度实体，按照vruntime排序。
- group_node：链表节点，被链接到percpu的rq->cfs_tasks上，在做CPU之间的负载均衡时，就会从该链表上选出group_node节点作为迁移进程（进一步了解请查阅2.3中的链接）
- on_rq: 标志位，代表当前调度实体是否在就绪队列上
- exec_start: 当前实体上次被调度执行的时间
- sum_exec_runtime: 当前实体总执行时间
- vruntime: 调度实体的虚拟时间
- prev_sum_exec_runtime：截止到上次统计，进程执行的时间，通常，通过sum_exec_runtime - prev_sum_exec_runtime来统计进程本次在CPU上执行了多长时间，以执行某些时间相关的操作

</font>

<font face="华文楷体" size=4>4、struct sched_class </font>

<font face="华文楷体" size=3>4.1、思维导图 </font>

<div><center>
<img src = sched_class.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图4-5 struct sched_entity 思维导图  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>4.2、源代码 </font>

<div><center>
<img src = sched_class2.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图4-6 struct sched_entity 源代码  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>4.3、详细介绍 </font>

<font face="华文楷体" size=2.5>

- const struct  sched_class *next: 指向下一个比其等级低的class。其顺序依次为stop，deadline，real time，fair，idle。
- enqueue_task：将一个task插入到相应的runqueue里面
- dequeue_task：将一个task从runqueue里面删除
- yield_task：放弃CPU执行权限，将任务从执行队列中出队，然后再放入到队列末尾。
- check_preempt_curr：用于检查当前进程是否可以被新的进程抢占
- pick_next_task：选择下一个运行的进程
- put_prev_task：将进程放回到运行队列当中

注：sched_class是Linux内核为不同调度策略定义的调度类，是对调度器公共部分的抽象。它是一个可扩展的调度模块层次结构，可用于实现不同的调度策略。

</font>

<font face="华文楷体" size=4>5、struct sched_class fair_sched_class </font>

<font face="华文楷体" size=3>5.1、实例化源代码 </font>

<div><center>
<img src = fair_sched_class.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图4-7 struct fair_sched_class 源代码  </font>
</strong>
</center></div>

&emsp;&emsp;fair_sched_class 是 sched_class类其中一个实例化，整个fair.c文件都是关于 fair_sched_class 类的函数的定义。
&emsp;&emsp;关于 enqueue_task、dequeue_task、yield_task、check_preempt_curr、put_prev_task这五个函数不作进一步介绍，可以参照其字面意思来理解。
&emsp;&emsp;重点介绍一下pick_next_entity 函数，因为该函数是调度函数 schedule 的重要组成部分。
&emsp;&emsp;pick_next_entity 被 pick_next_task_fair函数封装

<font face="华文楷体" size=3>5.2、pick_next_task_fair源代码 </font>

<div><center>
<img src = fair_pick1.png width=80% height=80% >
<br>
<img src = fair_pick2.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图4-8 pick_next_task_fair 源代码  </font>
</strong>
</center></div>

<font face="华文楷体" size=3>5.3、pick_next_task_fair代码逻辑 </font>

&emsp;（1）如果就绪队列为空，就跳转到idle标签。
&emsp;（2）否则，把之前运行的任务放回就绪队列。
&emsp;（3）然后，用真正的选择函数 pick_next_entity（）从就绪队列中选择最优先的调度实体（sched_entity），可能是一个单个任务或者一个任务组。
&emsp;（4）如果选择的调度实体是一个任务组，就递归地进入它对应的子就绪队列，直到找到一个单个任务。
&emsp;（5）最后，返回找到的任务。

<font face="华文楷体" size=3>5.3、pick_next_entity源代码以及注解 </font>

&emsp;&emsp;只需要理解第一部分即可，后两部分可以忽略。

<div><center>
<img src = fair_pick_next1.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图4-9 pick_next_entity 源代码（一）  </font>
</strong>
</center></div>

<div><center>
<img src = fair_pick_next2.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图4-10 pick_next_entity 源代码（二）  </font>
</strong>
</center></div>

<div><center>
<img src = fair_pick_next3.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图4-11 pick_next_entity 源代码（三）  </font>
</strong>
</center></div>

<font face="华文楷体" size=4>6、Chapter4 总结 & 完整思维导图 </font>

<font face="华文楷体" size=2.5> 

&emsp;&emsp;Chapter4主要介绍了什么是完全公平调度算法（策略）。CFS算法引入了虚拟时间vruntime的概念，每次调度都会选择vruntime值最小的调度实体。CPU内采用CFS调度策略的进程都位于一个cfs_rq类型的队列中，该队列采用红黑树的方式保存了所有的调度实体，并以vruntime值的大小为键值。红黑树的最左端就是vruntime最小值对应的调度实体。红黑树的具体实现比较复杂，需要进一步了解的可以参考以下链接
&emsp;&emsp;参考1：https://blog.csdn.net/xiaofeng10330111/article/details/106080394
&emsp;&emsp;参考2：https://blog.csdn.net/cy973071263/article/details/122543826
&emsp;&emsp;现在，本文已经将进程调度的整体流程大体上讲解完毕，为了帮助理解，以下是进程调度的完整思维导图，帮助大家理解从 Chapter1 到 Chapter4 的所有知识点。

<div><center>
<img src = 完整思维导图.png width=90% height=90% >
<br>
<strong><font face="华文楷体" size=2>图4-12 完整思维导图 </font>
</strong>
</center></div>

</font>

<div STYLE="page-break-after: always;"></div>

<br>

<center><font face="华文楷体" size=5>Chapter5 选做部分 </font></center>

<br>

&emsp;&emsp;修改core.c中—__schedule()函数，修改部分的代码如下图所示

<div><center>
<img src = code.png width=80% height=80% >
<br>
<strong><font face="华文楷体" size=2>图5-1 修改的部分 </font>
</strong>
</center></div>

&emsp;&emsp;printk()函数用于在终端输出进程切换前后的PID值，KERN_DEBUG表示该printk语句优先级最低，因此不会在开机时进行输出，一般情况下也不会在终端输出，只有在终端输入dmesg | tail时才会呈现。

&emsp;&emsp;printk()函数用于在终端输出进程切换前后的PID值，KERN_DEBUG表示该printk语句优先级最低，因此不会在开机时进行输出，一般情况下也不会在终端输出，只有在终端输入dmesg | tail时才会呈现。

&emsp;&emsp;实验结果如下：

<div><center>
<img src = success.png width=60% height=60% >
<br>
<strong><font face="华文楷体" size=2>图5-2 实验结果 </font>
</strong>
</center></div>

&emsp;&emsp;注：这种方式对虚拟机内存要求较高，经测试，内存需要大于等于10GB（比较保险）
