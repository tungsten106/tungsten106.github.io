---
title: 在linux中查看运行指定进程资源占用（cpu+gpu）
date: 2023-12-29 11:44:00 +0800
categories: [实践记录]
tags: [linux]     # TAG names should always be lowercase
img_path: /assets/img/
---


## CPU

### 1. 查找进程号

如果进程较多，输入 `ps -ef | grep ` + 指令关键词 进行搜索。如果运行的是python程序，可以输入 `ps -ef | grep python3`

比如我想查找所有指令中含hello关键词的进程，输入：`ps -ef | grep hello`

输出示例：

```bash
user      52584  75914  0 13:22 pts/9    00:00:00 docker run -it -p 8887:8887 image_hello:v1
user 	  12345  12345  0 13:21 pts/4    00:00:00 python3 hello.py
```

其中第二列为pid

### 2. 查看指定进程号

用top指令查看指定进程（例如我这里查看PID为3833）的进程：

```bash
top -p 3833
```

出现以下内容：

```bash
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s): 30.8 us, 11.3 sy,  0.0 ni, 55.6 id,  2.2 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26359936+total, 10537104 free, 64877176 used, 18818508+buff/cache
KiB Swap:        0 total,        0 free,        0 used. 19780235+avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND             
 12345 root      20   0   11.1g   1.9g 126324 S  1308  0.8 196:31.05 python3  
```

其中 RES代表运行内存，%CPU代表运行占用多少核。每100为1.0核（1.0c），以上示例为13.08c。

#### 其他参数解释

倒数第二行为参数名称，最后一行是参数内容。以上参数解释如下：

1. **PID**：Process ID，进程标识号。这是系统用来唯一标识活动进程的数字。
2. **USER**：该进程所属的用户名称或ID。
3. **PR**：Priority，进程的优先级。它显示了进程的调度优先级，数字越小代表优先级越高。
4. **NI**：Nice value，进程的nice值。这是一个用户设定的优先级值，用来影响进程的调度优先级。正值降低优先级，负值增加优先级。
5. **VIRT**：Virtual Memory Size，虚拟内存大小，单位通常是KiB。它包括进程使用的所有可用内存，包括交换空间、设备映射和分配但未使用的内存。
6. **RES**：Resident Set Size，常驻内存大小。这是该进程已分配的、位于RAM中的非交换区内存的大小，不包括被交换出去的部分。
7. **SHR**：Shared Memory，共享内存大小。指的是可被其他进程共享的内存量。
8. **S**：Process Status，进程状态。常见状态有:
   - `S` (sleeping): 睡眠状态
   - `R` (running): 运行状态
   - `T` (stopped): 停止状态
   - `Z` (zombie): 僵尸状态
9. **%CPU**：该进程占用的CPU百分比。
10. **%MEM**：该进程占用的物理内存百分比。
11. **TIME+**：该进程自启动以来占用的CPU总时间。
12. **COMMAND**：启动进程的命令名称或命令行。



## GPU

- 如果是NVIDIA GPU，可以使用 `nvidia-smi` 命令。它会显示所有NVIDIA GPU的使用情况，包括每个GPU的利用率，以及每个进程的具体GPU使用情况。
- `nvidia-smi`  指令示例输出如下：

```bash
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 460.32.03    Driver Version: 460.32.03    CUDA Version: 11.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  GeForce RTX 3080    Off  | 00000000:01:00.0  On |                  N/A |
| 30%   55C    P2    70W / 320W |   5478MiB / 10018MiB |     28%      Default |
+-------------------------------+----------------------+----------------------+
                                                                             
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      1569      G   /usr/lib/xorg/Xorg                169MiB |
|    0   N/A  N/A      2410      G   /usr/bin/gnome-shell              106MiB |
|    0   N/A  N/A      4021      C   python3                          5201MiB |
+-----------------------------------------------------------------------------+
```

Processes部分显示了当前在GPU上运行的进程列表，通常包括进程ID、使用的GPU、使用的内存等信息。通过**GPU Memory Usage**参数查看每个进程的GPU的显存。

### 实时监控

```shell
nvidia-smi -l 1
```

这里 `1` 可以替换为其他数字，代表每x秒刷新一次。

## Reference

1. [TOP命令参数详解---10分钟学会top用法 - 新盟教育的文章 - 知乎](https://zhuanlan.zhihu.com/p/562569361)

