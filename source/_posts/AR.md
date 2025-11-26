title: AR
tags: []
categories: []
date: 2025-11-26 22:37:27
---
要复刻（复现）ORB-SLAM2 这个经典的 SLAM 项目，你需要一个 **Linux 环境**（强烈推荐 Ubuntu 16.04, 18.04 或 20.04）。虽然项目文档写的是支持 Ubuntu 12/14/16，但在较新的系统（如 20.04）上编译需要修改少量代码。

以下是基于你提供的文档以及实际开发经验总结的**保姆级复刻步骤**：

-----

### 第一步：环境准备

ORB-SLAM2 高度依赖 Linux 库。如果你使用的是 Windows，建议使用 **WSL2** 或 **虚拟机（VMware/VirtualBox）** 安装 Ubuntu。

**基础工具安装（打开终端执行）：**

```bash
sudo apt-get update
sudo apt-get install git cmake gcc g++ build-essential
```

-----

### 第二步：安装核心依赖库

根据文档，你需要安装以下几个关键库。

#### 1\. 安装 Pangolin (用于可视化)

这是最容易出错的一步，需要从源码编译。

```bash
# 安装 Pangolin 的依赖
sudo apt-get install libglew-dev libpython2.7-dev libboost-dev libboost-thread-dev libboost-filesystem-dev

# 克隆并编译 Pangolin
git clone https://github.com/stevenlovegrove/Pangolin.git
cd Pangolin
mkdir build
cd build
cmake ..
make -j4
sudo make install
```

#### 2\. 安装 OpenCV (图像处理)

Ubuntu 通常自带 OpenCV 库。你可以先尝试安装系统自带版本：

```bash
sudo apt-get install libopencv-dev
```

*注意：ORB-SLAM2 原生支持 OpenCV 2.4 或 3.x。如果你是 Ubuntu 20.04，默认安装的是 OpenCV 4，可能需要修改 `CMakeLists.txt`（见下文“常见坑点”）。*

#### 3\. 安装 Eigen3 (矩阵计算)

```bash
sudo apt-get install libeigen-dev
```

-----

### 第三步：下载与编译 ORB-SLAM2

#### 1\. 克隆代码

```bash
cd ~
git clone https://github.com/raulmur/ORB_SLAM2.git ORB_SLAM2
cd ORB_SLAM2
```

#### 2\. 修改代码（**关键步骤：针对较新系统**）

如果你使用的是 Ubuntu 18.04 或 20.04 及更新的编译器，直接编译会报错 `usleep` 未定义。你需要手动修改几个文件：

找到以下文件：

  * `src/System.cc`
  * `src/Tracking.cc`
  * `src/LocalMapping.cc`
  * `src/LoopClosing.cc`
  * `src/Viewer.cc`
  * `Examples/Monocular/mono_tum.cc` (以及其他你打算运行的 Examples 下的 .cc 文件)

**在这些文件的顶部添加一行代码：**

```cpp
#include <unistd.h>
```

#### 3\. 执行编译脚本

项目提供了一个脚本一键编译第三方库（DBoW2, g2o）和主项目。

```bash
chmod +x build.sh
./build.sh
```

如果编译成功，终端最后会显示 `[100%] Built target ...`，并且在 `lib` 目录下生成 `libORB_SLAM2.so`。

-----

### 第四步：下载数据集并运行

为了验证是否成功，我们需要跑通一个 Demo。这里以 **TUM 数据集（单目模式）** 为例，因为它比较小且容易下载。

#### 1\. 下载数据集

前往 [TUM 数据集官网](https://vision.in.tum.de/data/datasets/rgbd-dataset/download)。
下载 `rgbd_dataset_freiburg1_xyz` (缩写为 fr1\_xyz)。

#### 2\. 运行命令

假设你把下载的数据集解压到了 `Downloads` 文件夹。

回到 `ORB_SLAM2` 项目根目录，执行：

```bash
./Examples/Monocular/mono_tum Vocabulary/ORBvoc.txt Examples/Monocular/TUM1.yaml /home/你的用户名/Downloads/rgbd_dataset_freiburg1_xyz
```

*(注意：路径需要替换为你实际的解压路径)*

**预期结果：**

  * 会弹出一个 Pangolin 的可视化窗口。
  * 你会看到绿色的相机轨迹在移动，红色的特征点在生成。

-----

### 常见坑点与解决方案

1.  **OpenCV 4 报错**

      * **现象**：编译时提示找不到 OpenCV 或者 API 不兼容。
      * **解决**：打开 `CMakeLists.txt`，将 `find_package(OpenCV 3.0 REQUIRED)` 改为 `find_package(OpenCV REQUIRED)`。同时，在代码中某些使用 `CV_LOAD_IMAGE_UNCHANGED` 的地方，可能需要改为 `cv::IMREAD_UNCHANGED`（OpenCV 4 的新标准）。

2.  **编译卡死**

      * **现象**：电脑卡住不动。
      * **解决**：`make -j` 会占用所有 CPU 核心。如果内存不够，请手动修改 `build.sh`，将 `make -j` 改为 `make -j2` 或 `make -j4`。

3.  **ROS 模式**

      * 如果你需要运行 ROS 示例，必须先安装 ROS（如 Melodic 或 Noetic），并配置环境变量 `ROS_PACKAGE_PATH`。然后执行 `./build_ros.sh`。这通常比纯 C++ 版本更复杂，建议先跑通纯 C++ 版本。

### 总结

复刻的核心流程是：**装依赖 (Pangolin 是重点) -\> 改源码 (加 unistd.h) -\> 编译 -\> 下数据 -\> 运行**。

如果你在编译过程中遇到具体的报错信息，可以随时发给我，我帮你分析具体原因。