

<center><strong><font face="微软雅黑" size=5>openEuler 进程调度</font></strong></center>

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

<center><font face="华文楷体" size=5>完整思维导图</font></center>

<br>

<div><center>
<img src = source/完整思维导图.png width=90% height=90% >
<br>
</center></div>

