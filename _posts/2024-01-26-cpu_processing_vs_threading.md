---
title: 进程/线程与cpu的关系
date: 2024-01-26 15:54:00 +0800
categories: [学习笔记]
tags: [cs_basics]     # TAG names should always be lowercase
img_path: /assets/img/
---


由于之前处理一个cpu消耗比较大的程序，想了解一下电脑的cpu处理原理

# CPU主要构成
## 逻辑处理器和内核
cpu中有两个概念：processor数量 (NumberOfLogicalProcessors) 和cores数量 (NumberOfCores)。

物理核心数（内核数，cores）决定了cpu可以同时处理多少个独立计算任务，而逻辑处理器数（Processors）决定了在这些核心上执行任务的效率。

可以做一个餐厅工作的比喻，整个餐厅是一个CPU，是完成工作（计算任务）的地方。

- CPU（餐厅）由一个或多个物理核心/内核/Cores（厨房）组成，每个内核（厨房）可以单独工作，同时执行不同的计算任务（不同的菜单）。一个多核CPU相当于一家餐厅有多个厨房。

- 逻辑处理器/Processors相当于厨师。每个内核（厨房）可以有一个或者多个逻辑处理器（厨师）。

### 查看CPU配置 - Windows

在windows系统中可以打开终端，输入以下指令查看：

```bash
wmic cpu get name, numberofcores, numberoflogicalprocessors
```

或者 `Ctrl + Shift + Esc` 快捷键打开任务管理器，转到性能选项卡，在右下角有逻辑处理器（Processors）的数量和内核（cores）数量。

### linux

打开终端, 输入命令 `lscpu` 并按回车。查找“CPU(s)”行，这显示了CPU核心数。还可以查看“核心（Core(s) per socket）”和“线程（Thread(s) per core）”的数量来获取更详细的信息。

### MacOS

打开终端输入指令：

```bash
sysctl -n hw.ncpu
```

和指令

```bash
sysctl hw.physicalcpu hw.logicalcpu
```

来获取物理核心数和逻辑核心数（在超线程技术中）

或者可以通过1. 点击屏幕左上角的苹果菜单，选择“关于本机”。2. 点击“系统报告”。3. 在硬件部分，查找“处理器名称”和“处理器数量”。还可以通过看“核心数”来确定CPU的核心数量。

## 进程vs线程
	- 进程（Process）是一个在内存中运行的应用程序。每个进程都有自己独立的一块内存空间，一个进程可以有多个线程。
	- 线程（Thread）是进程中的一个执行任务（控制单元），负责当前进程中程序的执行。一个进程至少有一个线程，一个进程可以运行多个线程，多个线程可共享数据。

继续之前餐厅的比喻。一个线程就相当于一个厨师（逻辑处理器）做一道菜。假设一个厨房只做一个菜单，一个菜单相当于一个进程，由这个厨房（内核）中的所有厨师（逻辑处理器）共同完成。

## 程序的并行处理

假如我的电脑的配置是10 Cores+12 Processors，这说明电脑可以同时执行10个完全独立的程序或者计算任务，每一个任务在单独的物理核心上运行，并且有一些物理核心支持Intel的超线程技术（Hyper-Threading），允许每个这样的核心同时处理两个线程。

在10 Cores+12 Processors的情况下，可能有8个单线程内核+2个超线程内核，这两个超线程内核可以同时执行两个线程。

程序并行处理的实现分为多进程（在多个核心上分别处理独立任务），和多线程（允许程序创建多个执行线索，可以并行执行）等方法（还有分布式计算，用多个计算机处理一个计算任务，这里不赘述了）

可以使用编程，用程序直接创建和管理线程，在C++，Java等语言中比较常见。注意Python由于全局解释器锁（GIL）的存在，Python的线程并不能真正有效地利用多核处理器进行并行计算，相当于对于CPU密集型任务，Python通常采用多进程来实现并行。


# 并行任务的Python实现

## 多线程
对于I/O密集型任务（网络请求或者文件读写），仍然可以用多线程进行并行处理。

Python中一般使用 `threading` 模块，创建 `threading.Thread` 对象，并将目标函数和参数传递给它。虽然这些线程由于GIL的存在而无法在CPU密集型任务上并行执行，但它们可以在等待I/O时允许其他线程运行。

```python
import threading
import time
import os


def worker(num):
    """线程工作函数"""
    print(f'Worker: {num}, PID: {os.getpid()}', end="\n")


if __name__ == '__main__':
    start = time.time()
    threads = []
    for i in range(5): # 创建5个线程
        t = threading.Thread(target=worker, args=(i,))
        threads.append(t)
        t.start()

    # 等待所有线程完成
    for t in threads:
        t.join()
    end = time.time()
    
    print(f"multithreading: {end - start}")
```

输出：

```
Worker: 0, PID: 20864
Worker: 1, PID: 20864
Worker: 2, PID: 20864
Worker: 3, PID: 20864
Worker: 4, PID: 20864
multithreading: 0.001130819320678711
```



## 多进程
可以使用 `multiprocessing` 模块来绕过GIL限制，创建进程，每个进程都有自己的Python解释器和内存空间，因此它们可以真正并行地运行在多核CPU上。

```python
import multiprocessing
import os
import time

def worker(num):
    """线程工作函数"""
    print(f'Worker: {num}, PID: {os.getpid()}')

if __name__ == '__main__':
    # 创建多个进程
    start = time.time()
    processes = []
    for i in range(5): # 创建5个进程
        p = multiprocessing.Process(target=worker, args=(i,))
        processes.append(p)
        p.start()
    # 等待所有进程完成
    for p in processes:
        p.join()
    end = time.time()    
    print(f"multiprocessing: {end - start}")
    
    start = time.time()
    processes = []
    for i in range(5): # 创建5个进程
        processes.append(worker(i+5))
    end = time.time()
    print(f"singleprocessing: {end - start}")
```

输出：

```
Worker: 0, PID: 15116
Worker: 1, PID: 28236
Worker: 2, PID: 26044
Worker: 3, PID: 18240
Worker: 4, PID: 22648
multiprocessing: 0.1219334602355957
Worker: 5, PID: 6028
Worker: 6, PID: 6028
Worker: 7, PID: 6028
Worker: 8, PID: 6028
Worker: 9, PID: 6028
singleprocessing: 0.001992940902709961
```

其实也可以看出对于简单的任务，多进程不一定比单进程要快。