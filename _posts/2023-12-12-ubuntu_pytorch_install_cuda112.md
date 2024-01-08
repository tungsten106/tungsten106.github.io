---
title: ubuntu给cuda11.2安装pytorch
date: 2023-12-12 16:31:00 +0800
categories: [实践记录]
tags: [python, pytorch]     # TAG names should always be lowercase
img_path: /assets/img/
---

在使用镜像新建了一个`cuda11.2-python3.9` 容器配置环境的过程中需要安装PyTorch。一开始我直接使用 `pip install torch` 来进行安装，但是运行程序时出现报错：

```bash
RuntimeError: The NVIDIA driver on your system is too old (found version 11020). Please update your GPU driver by downloading and installing a new version from the URL: http://www.nvidia.com/Download/index.aspx Alternatively, go to: https://pytorch.org to install a PyTorch version that has been compiled with your version of the CUDA driver.
```

这个错误表明目前系统上安装的 NVIDIA 驱动程序版本太旧，不满足 PyTorch 对 cuda驱动程序的要求。根据提示我们可以选择更新NVIDIA 驱动，或者安装适当版本的PyTorch。这里我选择了更新PyTorch。

首先通过 `nvcc -V` 指令查看cuda版本，得知为版本11.2

在官方安装pytorch2.0的文档中，最低cuda版本为11.8.因此我们需要安装一个旧版pytorch

在官方的旧版pytorch与cuda[版本对应](https://pytorch.org/get-started/previous-versions)中可以找到支持版本11.2以下的最高版本是v1.12.1，通过安装指令：

```bash
# CUDA 10.2
conda install pytorch==1.12.1 torchvision==0.13.1 torchaudio==0.12.1 cudatoolkit=10.2 -c pytorch
```

 