---
title: 在VisionFive 2（昉·星光2）开发板上基于Opencv进行人脸识别
date: 2023-12-05 01:14:50 +0800
categories: [实践记录]
tags: [python, 人机交互]     # TAG names should always be lowercase
img_path: /assets/img/
---


![starfive](https://doc.rvspace.org/VisionFive2/PB/Image/VisionFive2/VF2photo.png)

此教程的运行主要是在StarFive的VisionFive 2（昉·星光2）开发板上，板子的具体信息或者其他产品参考[官网主页](https://www.starfivetech.com/)；VisionFive 2的开源技术文档和装机教程参考[官方文档](https://doc.rvspace.org/Doc_Center/visionfive_2.html)。

## 1. 在VisionFive 2上安装debian系统

### 将Debian OS烧录到Micro-SD上

这一部分主要参考[官方文档](https://doc.rvspace.org/VisionFive2/PDF/VisionFive2_QSG.pdf)中的*3.3. 将OS烧录到Micro-SD卡上*

【事前准备】准备32g的Micro-SD卡，首先通过外接读卡器或者内置的读卡器将Micro-SD卡接入个人计算机中。格式化Micro-SD卡。

*如果是MacOS用户，Micro-SD卡有可能不会显示读入。可以通过在终端中输入`diskutil list` 来确认目标Micro-SD卡已读入，然后输入`sudo diskutil eraseDisk FAT 32 SDCARD MBRFormat </dev/...>` 来格式化Micro-SD卡。`</dev/...>` 为目标Micro-SD卡的具体名称。*

【下载系统】在starfive提供的[链接](https://debian.starfivetech.com/)上下载最新版本的debian镜像（目前最新版本是202306），将sd文件夹里后缀是.bz2的文件解压。

【烧录步骤】文档中推荐使用BalenaEtcher，点击[下载链接](https://etcher.balena.io/)安装并运行。点击Flash from File，然后选择上一步解压后的.img文件；点击Select target，并选择链接好的Micro-SD卡；最后点击Flash!开始烧录。

### 登陆Debian

在官方文档章节3.4中给出了3种登录方式：

- 通过HDMI使用Xfce桌面环境登录（第22页）
- 通过以太网使用SSH登录（第23页）
- 使用USB转串口转换器连接并登录（第26页）

本教程主要使用**USB转串口连接MacOS电脑**并登录。如果想尝试其他方式可以参考[官方文档](https://doc.rvspace.org/VisionFive2/PDF/VisionFive2_QSG.pdf)。

【硬件连接】参考第30页示意图连接串口，并把USB转串口转换器连接到个人计算机上。

【安装minicom】在MacOS上安装minicom需要使用Homebrew安装，所以需要先安装Homebrew。在终端中输入如下指令

``````zsh
/bin/bash -c "$(curl -fsSL https://gitee.com/ineo6/homebrew-install/raw/master/install.sh)"
``````

然后输入安装minicom的指令

```zsh
brew install minicom
```

【连接串口】首先需要知道通过usb串口连接的设备名称。先通过指令`ls /dev/tty.*` 查看电脑连接的所有串口设备，得到目标设备的id（示例：`/dev/tty.usbmodem102` ，其中usbmodem102是设备的id，前面的`/dev/tty.` 不变。得知设备id之后输入如下指令连接到串口设备：

```zsh
minicom -D /dev/tty.<deviceID> -b 115200
```

【启动设备】输入用户名和密码登录。可以使用root或者user用户名登录，密码都是starfive。

## 2. Python3环境部署

官方已经给出了一键安装包和依赖（包括firefox浏览器、python-opencv等）的shell指令。首先需要下载相应的文件。这一步需要给starfive联网，可以通过以太网接口连接网线。

```zsh
wget https://github.com/starfive-tech/Debian/releases/download/v0.8.0-engineering-release-wayland/install_package_and_dependencies.sh

```

这一步可以通过starfive的终端完成，或者在个人计算机上下载后传输到starfive中（也可以在starfive中新建同名文件并复制所有内容）。运行安装包和依赖的shell文件。

```zsh
chmod +x install_package_and_dependencies.sh
sudo ./install_package_and_dependencies.sh
```

这个安装过程会比较久，可能需要2-3小时。

## 3. 使用opencv进行人脸识别

【硬件配置】本文使用的是usb摄像头，目前版本的starfive暂时不支持pyqt，即无法显示程序读取到的摄像头内容。如果要验证自己摄像头的可用性，可以在终端中输入如下python指令：

```zsh
python3
```

如果上一步python3配置成功，会显示python编程界面。在此界面中输入如下代码：

```python
import cv2

print(cv2.VideoCapture(index=4, apiPreference=cv2.CAP_V4L2)) # usb摄像头
```

这里外接USB摄像头的index默认是4，最好是在终端中输入`sudo v4l2-ctl --list-devices` ，查看USB Camera下面的`/dev/video` 后面的数字来确认index。

如果打印摄像头信息没有报错说明摄像头正常运行。

【人脸识别】成功读取摄像头后就可以进行有关人脸识别的程序编写了。由于无法使用qt，最好是在终端中输入`export QT_QPA_PLATFORM=offscreen ` 指令，使用离屏模式。

以下是一个简单的示例，摄像头会读取信息并在输出中打印True/False来说明是否识别到人脸。新建一个python文档，在终端中输入`nano test1.py` 进入nano编辑器；输入如下代码。

```python
import cv2
import time

# 加载级联分类器模型
face_cascade = cv2.CascadeClassifier("/usr/share/opencv4/haarcascades/haarcascade_frontalface_default.xml")

# 打开摄像头
cap = cv2.VideoCapture(4, cv2.CAP_V4L2) 

# 创建计时器
start_time = time.time()

while True:
    # 读取摄像头图像
    ret, frame = cap.read()

    # 将图像转换为灰度
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    # 检测人脸
    faces = face_cascade.detectMultiScale(gray, scaleFactor=1.1, minNeighbors=5, minSize=(30, 30))

    # 判断是否检测到人脸
    if len(faces) > 0:
        has_face = True
    else:
        has_face = False

    # 输出结果
    print(has_face)
    
    # 检查是否已经过了30秒
    elapsed_time = time.time() - start_time
    if elapsed_time >= 30:
        break

# 等待一段时间确保程序结束
time.sleep(2)

# 结束程序
exit()
```

这里是用的是默认的正脸模型`haarcascade_frontalface_default.xml` 。在`/usr/share/opencv4/haarcascades` 目录下还有其他模型可以进行不同类型的人脸识别，如识别眼睛部位等。

### 实例演示

如这个教程所示，我们制作了与starfive合作的视频，通过面部识别遥控纸花（点此[链接](https://www.bilibili.com/video/BV1Rz4y1J7fQ/)）

