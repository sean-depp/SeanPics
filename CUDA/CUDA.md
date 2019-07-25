# Ubuntu18.04下CUDA10.x和TensorFlow1.x环境搭建


[Mac和Ubuntu下修改pip源和TensorFlow(CPU)安装](https://www.jianshu.com/p/aa97968de1c2)

-----

## 目录

* 前言
* 开发环境一览
* ~~显卡驱动安装~~

> * ~~下载驱动~~
> * 禁用nouveau
> * ~~安装驱动~~

* 安装CUDA 10.x

> 第一个CUDA程序

* 安装cudnn7.x
* 安装TensorFlow1.x
* 最后

----------

## 前言

> 其实主要是CUDA的安装, 别的都很简单.

----------


## 开发环境一览

* CPU: Intel® Xeon(R) CPU E5-2696 v3
* GPU: NVIDIA GTX 1080Ti
* OS: Ubuntu 18.04.2 LTS 64位

> 用指令看下英伟达显卡:

```
lspci | grep -I nvidia
```

> 当你搭建完成环境之后, 可以用官方案例查看硬件信息, 我的GPU信息显示如下图, 这张表的参数在后续的并行算法设计中是很有用的:

![image](https://user-images.githubusercontent.com/21376904/61846892-65c52300-aedb-11e9-9fd8-cac7f18ef0d8.png)

----------

## ~~显卡驱动安装~~

> **这步其实可以跳过, CUDA包里面带了驱动, 但是我还是给出流程. 注意禁用nouveau这一步还是要的.**

> 最好不要用Ubuntu附加驱动里提供的显卡驱动.
> 可能会遇到一些奇怪的问题, 当然, 锦鲤是不会出问题的(手动滑稽).
> 这是第一个坑点, 大体有三种展现方式:

> * 装完重启进不去系统, 卡住ubuntu加载页面;
> * 无限登录;
> * 装好了, 进入了系统, 然后输入nvidia-smi指令没有任何反应. 正常情况会弹出一张表, 如下所示:

![image](https://user-images.githubusercontent.com/21376904/61847022-c3f20600-aedb-11e9-9c56-12c4098fb348.png)

----------

### 下载驱动

> 我的实操:
> 首先到[官网](https://www.geforce.cn/drivers)下载显卡驱动, 比方说我是GTX 1080Ti, 操作系统是64位Linux, 我就找对应的版本进行下载.

![image](https://user-images.githubusercontent.com/21376904/61842928-7de17600-aecc-11e9-945e-234de71d6484.png)

> 删掉以往的驱动. 注意, 就算你啥都没装, 这步也是无害的.

```
sudo apt-get remove --purge nvidia*
```

> 安装驱动需要的库:

```
sudo apt-get update
sudo apt-get install dkms build-essential linux-headers-generic
```

> 驱动安装可能需要的库:

```
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install libc6:i386
```

> 安装CUDA需要的库:

```
sudo apt-get install freeglut3-dev libx11-dev libxmu-dev libxi-dev libgl1-mesa-glx libglu1-mesa libglu1-mesa-dev
```

----------


### 禁用nouveau

> 打开blacklist.conf, 在最后加入禁用nouveau的设置, 这是一个开源驱动, 如图所示:

```
sudo vim /etc/modprobe.d/blacklist.conf
```
```
blacklist nouveau
blacklist lbm-nouveau
options nouveau modeset=0
alias nouveau off
alias lbm-nouveau off
```

![image](https://user-images.githubusercontent.com/21376904/61847098-fbf94900-aedb-11e9-9d80-ea287b4fa3e6.png)


> 禁用nouveau内核模块

```
echo options nouveau modeset=0
sudo update-initramfs -u
```

> 重启. 如果运行如下指令**没用打印出任何内容**, 恭喜你, 禁用nouveau成功了.

```
lsmod | grep nouveau
```


----------


### 安装驱动

> ~~来到tty1(快捷键ctrl + alt + f1,如果没反应就f1-f7一个个试, 不同Linux, 按键会略有不同). 运行如下指令关闭图形界面.~~我在ubuntu 18.04 LTS是ctrl + alt + f3-f6. 

> 然后注意, 以下指令适用于16.04及以前.

```
sudo service lightdm stop
```

> 这不适用于18.04. 18.04可以如下操作:

> * 关闭用户图形界面

```
sudo systemctl set-default multi-user.target
sudo reboot
```

> * 开启用户图形界面

```
sudo systemctl set-default graphical.target
sudo reboot
```

> ~~安装驱动, 注意**有坑**, 一定要加**-no-opengl-files**, 不加这个就算安装成功, 也会出现**无限登录**问题.~~ 但是在最近几次安装环境的时候, 例如系统是18.04, 驱动是418.43, 这个参数变得无效. 所以如果不能开启安装页面, 可以去掉此参数.

```
sudo chmod u+x NVIDIA-Linux-x86_64-390.87.run 
sudo ./NVIDIA-Linux-x86_64-390.87.run –no-opengl-files
```

![image](https://user-images.githubusercontent.com/21376904/61847999-28629480-aedf-11e9-9913-047c5febce74.png)
![image](https://user-images.githubusercontent.com/21376904/61848008-2f89a280-aedf-11e9-9681-e883ef266fd6.png)


> ~~如果你已经装了, 但是没有加**-no-opengl-files**, 按照如下操作可以救一下.~~ 或者你安装失败了, 有些库缺少了之类的, 可以用以下命令卸载干净重来.

```
sudo ./NVIDIA-Linux-x86_64-390.87.run –uninstall
```

> 顺带一提, 可能会弹出**Unable to find a suitable destination to install 32-bit compatibility libraries on Ubuntu 18.04 Bionic Beaver Linux**的bug, 然后你需要下面三条指令:

```
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install libc6:i386
```

> 并且中途的选项都选no比较好, 指不定卡死在安装哪个奇怪的东西上.

> 重启. 用**nvidia-smi**指令试一下, 如果看到类似下图, 恭喜你, 驱动安装成功. 或者看到附加驱动显示**继续使用手动安装的驱动**.

![image](https://user-images.githubusercontent.com/21376904/61847022-c3f20600-aedb-11e9-9c56-12c4098fb348.png)

> 安装之后在**软件和更新**当中会显示**继续使用手动安装的驱动**. 当然了, 我一般都是ssh访问, 不怎么看界面的.

----------


## 安装CUDA 10.x

> 到[官网](https://developer.nvidia.com/cuda-toolkit-archive)下载要的CUDA版本, 我这里是[10.0](https://developer.nvidia.com/cuda-10.0-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=runfilelocal), 下载runfile(local)版本, 如下图所示:

![image](https://user-images.githubusercontent.com/21376904/61847366-f6503300-aedc-11e9-8a3b-c751bb85bd9e.png)

> md5检测一下, 不合格要重新下载. 下图是我的检测结果:

```
md5sum cuda_10.0.130_410.48_linux.run
```

![image](https://user-images.githubusercontent.com/21376904/61847338-dfa9dc00-aedc-11e9-9a4d-0b7cdd90d815.png)


> ~~再次关闭图形界面~~

```
sudo service lightdm stop
```

> 这不适用于18.04. 18.04可以如下操作:
> * 关闭用户图形界面

```
sudo systemctl set-default multi-user.target
sudo reboot
```

> ~~安装时候依旧要加**-no-opengl-files**参数, 之后一路默认就好.~~ 最好不要安装与OpenGL相关的.

```
sudo sh cuda_10.0.130_410.48_linux.run
```

![image](https://user-images.githubusercontent.com/21376904/61844131-f3e7dc00-aed0-11e9-8160-0e4a7249e798.png)

> 然后会看到**三个installed**.

![image](https://user-images.githubusercontent.com/21376904/61844265-5fca4480-aed1-11e9-8158-05256ab8bf6c.png)


> 添加环境变量

```
vim ~/.bashrc
```

> 最后写入:

```
export CUDA_HOME=/usr/local/cuda
export PATH=$PATH:$CUDA_HOME/bin
export LD_LIBRARY_PATH=/usr/local/cuda-10.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

> 保存退出, 并其生效.

```
source ~/.bashrc
```

> 运行一些检测命令, 如果和我显示的类似, 恭喜你, 环境配置完成. 可以跑一下英伟达提供的学习案例:

```
cat /proc/driver/nvidia/version
```
```
nvcc -V
```

![image](https://user-images.githubusercontent.com/21376904/61846723-ce5fd000-aeda-11e9-8e8d-81a7cca60128.png)

![image](https://user-images.githubusercontent.com/21376904/61846892-65c52300-aedb-11e9-9fd8-cac7f18ef0d8.png)

----------

### 第一个CUDA程序

> 除了官方案例, 可以自己写代码查看设备信息:

```
vim Device.cu
```
```
#include <stdio.h>
int main() {
        int nDevices;

        cudaGetDeviceCount(&nDevices);
        for (int i = 0; i < nDevices; i++) {
                cudaDeviceProp prop;
                cudaGetDeviceProperties(&prop, i);
                printf("Device Num: %d\n", i);
                printf("Device name: %s\n", prop.name);
                printf("Device SM Num: %d\n", prop.multiProcessorCount);
                printf("Share Mem Per Block: %.2fKB\n", prop.sharedMemPerBlock / 1024.0);
                printf("Max Thread Per Block: %d\n", prop.maxThreadsPerBlock);
                printf("Memory Clock Rate (KHz): %d\n",
                   prop.memoryClockRate);
                printf("Memory Bus Width (bits): %d\n",
                   prop.memoryBusWidth);
                printf("Peak Memory Bandwidth (GB/s): %.2f\n\n",
                   2.0 * prop.memoryClockRate * (prop.memoryBusWidth / 8) / 1.0e6);
        }
        return 0;
}
```
```
nvcc Device.cu -o Device.o && ./Device.o
```

![image](https://user-images.githubusercontent.com/21376904/61847457-50e98f00-aedd-11e9-9b3a-410af7421271.png)

----------

## 安装cudnn7.x

> 首先到[官网](https://developer.nvidia.com/rdp/cudnn-download)去下载勾选的4个:

![image](https://user-images.githubusercontent.com/21376904/61847612-e5ec8800-aedd-11e9-86d2-29932f0fc30c.png)

> 然后解压tgz包, 复制文件到cuda环境, 接着安装deb包.

```
tar -zxvf cudnn-10.0-linux-x64-v7.5.0.56.tgz
sudo cp cuda/include/cudnn.h /usr/local/cuda/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*

sudo dpkg -i libcudnn7_7.5.0.56-1+cuda10.0_amd64.deb
sudo dpkg -i libcudnn7-dev_7.5.0.56-1+cuda10.0_amd64.deb
sudo dpkg -i libcudnn7-doc_7.5.0.56-1+cuda10.0_amd64.deb
```

![image](https://user-images.githubusercontent.com/21376904/61849813-15eb5980-aee5-11e9-8c91-5ade0fe7c885.png)

> 这样就完成安装了, 用个小栗子来测试下吧, 结果如图示:

```
cp -r /usr/src/cudnn_samples_v7/ ~ && cd ~/cudnn_samples_v7/mnistCUDNN
make clean && make && ./mnistCUDNN
```

![image](https://user-images.githubusercontent.com/21376904/61850008-b3df2400-aee5-11e9-8022-8cef6a7bd2a7.png)

----------

## 安装TensorFlow1.x

> 注意TF要求GPU算力在3.5以上, 可以去英伟达官网查看自己笔记本的算力, 当然了, 你用CPU也行的.

```
sudo apt-get install python-pip python3-pip python-dev
sudo pip3 install tensorflow-gpu
```

![image](https://user-images.githubusercontent.com/21376904/61850506-11c03b80-aee7-11e9-8bfb-539048eb54ca.png)

> 最后我给出一个测试例子:

```
#!/usr/bin/python3

import tensorflow as tf
# Just disables the warning, doesn't enable AVX/FMA
import os
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'

hello = tf.constant("Hello, tf!")
sess = tf.Session()
printf (sess.run(hello))
```

![image](https://user-images.githubusercontent.com/21376904/61850673-6fed1e80-aee7-11e9-9da3-1ab27521eaa3.png)

----------


## 最后

> 关于CPU版本, 可以参看[Mac和Ubuntu下修改pip源和TensorFlow(CPU)安装](https://www.jianshu.com/p/aa97968de1c2). 喜欢记得点赞哦, 有意见或者建议评论区见~


----------
