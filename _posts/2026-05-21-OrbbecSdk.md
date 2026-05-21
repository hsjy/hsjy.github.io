---
title: 奥比中光Dabai SDK安装
date: 2026-05-21 17:20:32 +0800
categories: [工具]
tags: [linux]     # TAG names should always be lowercase
author: <001>
description: 安装Dabai相机驱动
toc: true
comments: true
image:
  path: assets/img/posts/26-2/Orbbec/cover_image.jpg
  alt: Orbbec
pin: false
---

因为手中这款深度相机型号较老（DaBai SN），因而需要按照github仓库SDK v1.0的版本来安装，<https://github.com/orbbec/OrbbecSDK>
​
## Debian方式安装
安装OrbbecSDK：[Gitee](https://gitee.com/orbbecdeveloper/OrbbecSDK/blob/main/doc/tutorial/English/Installation_guidance.md)

下载sdk和Viewer：https://github.com/orbbec/OrbbecSDK/releases
![tool_select](assets/img/posts/26-2/Orbbec/tool_select.png)

这里我选择`OrbbecSDK_v1.10.18_amd64.deb`的版本，下载完成后，进入下载目录
```
sudo dpkg -i OrbbecSDK_v1.10.18_amd64.deb
dpkg -L orbbecsdk
```
接着下载[OrbbecViewer_v1.10.18_202501031427_linux_x64_release.zip](https://github.com/orbbec/OrbbecSDK/releases/download/v1.10.18/OrbbecViewer_v1.10.18_202501031427_linux_x64_release.zip)
解压到本地指定目录
```
unzip ./OrbbecViewer_v1.10.18_202501031427_linux_x64_release.zip -d /mnt/data/users/hs
```
解压后进入该文件夹，运行./OrbbecViewer即可。
![orbbec_viewer](assets/img/posts/26-2/Orbbec/orbbec_viewer.png)

## Python版本的Orbbec SDK构建
下载v1.3.4版本的[pyorbbecsdk](https://github.com/orbbec/pyorbbecsdk/tags)（其他的v1.x.x版本亦可），下载`tar.gz`的源码。
> 由于相机型号较老，v2.0版本不受支持
{: .prompt-warning}

解压到指定目录：
```
tar -zxf ./pyorbbecsdk-1.3.4.tar.gz -C /mnt/data/users/hs
```
进入解压后的目录，阅读readme.md文件，按步骤操作。
1.首先，安装依赖
```
sudo apt-get install python3-dev python3-venv python3-pip python3-opencv
```
2.其次，在编译 Orbbec Python SDK (pyorbbecsdk) 时指定自定义的 Python 环境路径（可选）
> 目的：告诉 CMake 不要使用系统默认找到的 Python 3，而是使用您特定设置的路径，例如Anaconda/Miniconda 环境或特定的 Virtual Environment 中的 Python。
{: .prompt-info}

在解压文件的根目录打开`CMakeLists.txt`文件(nano,vim,文本编辑器等），在`find_package(Python3 REQUIRED COMPONENTS Interpreter Development)`这`一行前加上
```
set(Python3_ROOT_DIR "/home/dell-3660/anaconda3/envs/piper_orbe")# 替换成你Anaconda环境的实际路径
set(pybind11_DIR "${Python3_ROOT_DIR}/lib/python3.10/site-packages/pybind11/share/cmake/pybind11")# 替换成你Pybind11的实际路径
```
> 关于如何找到conda和pybind11实际路径：
> - conda路径：激活环境`conda activate env_name`，然后`which python`查看`python`所在路径，`/bin/python`前面的部分便是`Python3_ROOT_DIR`；
> - pybind11路径：输入`python`进入交互式界面，输入`import pybind11; print(pybind11.__file__)`将得到的路径末尾`__init__.py`替换为`/share/cmake/pybind11`，并将路径前方与`Python3_ROOT_DIR`重合的部分替换为变量名称即可。
{: .prompt-info}

3.开始构建项目。
因为我已提前创建了且位于conda环境中因而省去了虚拟环境构建的部分，进入项目目录。
```
cd pyorbbecsdk-1.3.4
pip3 install -r requirements.txt
mkdir build
cd build
cmake -Dpybind11_DIR=`pybind11-config --cmakedir` ..
make -j4
make install
```
运行部分示例文档
```
cd pyorbbecsdk-1.3.4
export PYTHONPATH=$PYTHONPATH:$(pwd)/install/lib/
sudo bash ./scripts/install_udev_rules.sh
sudo udevadm control --reload-rules && sudo udevadm trigger
python3 examples/depth_viewer.py
python3 examples/color.py
python3 examples/net_device.py # Requires ffmpeg installation for network device
```
