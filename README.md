

## openEuler 进程调度

### 目录 

##### Chapter 1 进程基本概念

        1、什么是进程
        1.1 进程的定义
        1.2 进程与线程
        2、进程从创建到终止
        2.1 关系图
        2.2 进程的状态
        3、进程调度简单介绍
        3.1 什么是进程调度
        3.2 为什么需要进程调度


##### Chapter 2 struct task_struct

        1、结构体总体介绍
        1.1 为什么要介绍 struct task_struct
        1.2 struct task_struct 重要成员组
        3、进程与内核栈
        4、进程的亲属关系
        5、进程的标识符PID
        6、进程的标记flags
        7、进程的ptrace
        8、进程的统计信息
        9、进程的调度
        10、struct task_struct 总结


##### Chapter 3 进程调度

        1、进程的调度流程
        1.1 再次提及进程调度
        1.2 进程调度的流程
        1.3 一些补充
        2、struct rq
        3、schedule
        4、__schedule
        5、Chapter3 总结


##### Chapter 4 CFS（完全公平调度）介绍

        1.1 CFS基本原理概述
        1.2 CFS相关概念
        1.3 CFS算法设计核心
        2、struct cfs_rq
        3、struct sched_entity
        4、struct sched_class
        5、struct sched_class fair_sched_class
        6、Chapter4 总结 & 完整思维导图

##### Chapter 5 选做部分


### 完整思维导图
<br>
<div><center>
<img src = source/完整思维导图.png width=90% height=90% >
<br>
</center></div>

