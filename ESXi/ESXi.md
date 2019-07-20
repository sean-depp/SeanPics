# ESXi6.7安装流程(2019.7重编版)

## 目录

* 前言
* 准备工作
* 打包与刻录
* 安装
> * Initializing IOV卡住
> * 缺少网卡驱动
> * 安装ESXi6.7
> * Multiboot could not setup the video subsystem

* vSphere Client
> * 添加硬盘
> * 建立虚拟机
> * 显卡直通
* 最后


----------


## 前言

> ESXi直接安装在物理服务器上(裸机), 并将其划分为多个逻辑服务器, 即虚拟机. 相比个人电脑上常见的先装OS, 再装VMware Fusion等虚拟机软件, 再分配空间建立虚拟机. ESXi更多用于服务器, 也更高效能.

----------

## 准备工作

> * win10(Powershell 2.0+, VMware PowerCLI 5.1+)
> * u盘
> * ESXi的zip压缩包
> * 网卡驱动
> * 重打包Powershell脚本ESXi-Customizer-PS
> * u盘刻录软件

> 来一一说明:

> * 使用Powershell 2.0或更高版本的Windows计算机(XP或更高版本), 相比之前介绍的镜像重打包软件ESXi-Customizer, 这次更新使用Powershell脚本ESXi-Customizer-PS, 除了更加geek(zhuang bi)之外, 也可以进行更多操作.
> * 再者就是需要VMware PowerCLI 5.1或更高版本的. 可以通过Powershell直接下载, 类似于Linux的apt-get.

> 以上两个都是[打包脚本官网](https://www.v-front.de/p/esxi-customizer-ps.html#download)给出的要求.

> * u盘用于刻录打包好的ESXi镜像. 
> * 压缩包下载需要一个vmware的账号, [VMware Patch下载门户](https://my.vmware.com/cn/web/vmware/evalcenter?p=free-esxi6)

![image](https://user-images.githubusercontent.com/21376904/61181201-7bbe2280-a655-11e9-913b-537bb0d7b2ea.png)

> * 在安装ESXi的时候, 一般会缺少网卡驱动, 推荐手动打包进去, [网卡驱动下载地址](https://vibsdepot.v-front.de/wiki/index.php/List_of_currently_available_ESXi_packages). 我下的是net55-r8168. 当然也可以依赖Powershell脚本自动下载, 不过, 我推荐手动.

![](https://user-images.githubusercontent.com/21376904/61182798-0e1cf100-a66b-11e9-8e12-a78c44255a7b.png)

> * 镜像重打包脚本ESXi-Customizer-PS用于将驱动打包进压缩包
> * u盘刻录软件软碟通就不说了, 别的刻录软件也行.
> * 不过还是说一句, 珍爱生命, 远离Windows.

----------

## 打包与刻录

> 先来看一眼目录, 建立一个vm包, 放入ps脚本, ESXi压缩包, 建立输出目录和驱动目录, 驱动目录放入网卡驱动.

![image](https://user-images.githubusercontent.com/21376904/61181394-90e88080-a658-11e9-9a10-3f3bebdaaba6.png)
![image](https://user-images.githubusercontent.com/21376904/61181400-a8276e00-a658-11e9-8832-f6d0d12e1daf.png)

> 安装VMware PowerCLI, 你如果没有装VMware PowerCLI, 运行ps也会有提示的, 如下图红字, 然后两条指令就可以安装完成了, 很喜欢这种感觉. 如果Windows 80%的软件安装都能这样, 我或许会减少些对它的厌恶感吧.

```
# 查找模块
Find-Module -Name VMware.PowerCLI

# 安装模块
Install-Module -Name VMware.PowerCLI -Scope CurrentUser
```

![image](https://user-images.githubusercontent.com/21376904/61181467-9397a580-a659-11e9-978d-5c6908733aad.png)

> 然后就是打包操作了. 具体含义就不说了, 各位自己-help查看下, 毕竟我也不是卖脚本的. 总之就是将对应的zip, 驱动名称, 离线驱动目录, 输出目录给呼应上. 最后的-nsc最好加上, 油管的视频教程里面建议的, 说是不加这个会导致打包出错.

```
.\ESXi-Customizer-PS-v2.6.0.ps1 -izip .\ESXi670-201906002.zip -load net55-r8168 -pkgDir .\offline -outDir .\out -nsc
```

![image](https://user-images.githubusercontent.com/21376904/61181520-71eaee00-a65a-11e9-8ec2-f807a27b1721.png)

> 打开软碟通, 刻录. 这个就不说了, 这个也不会怎么进女寝装系统不是(手动滑稽).

![image](https://user-images.githubusercontent.com/21376904/61181577-10774f00-a65b-11e9-8f9d-76527e3aaa7c.png)

----

## 安装

> 进入BIOS, 选择刻录好的U盘.

### Initializing IOV卡住

> 如果你是4代酷睿, 你会遇到第一个bug. 卡在Initializing IOV. 解决方案就是进入之前按住shift + O. 然后输入``` noIOMMU```, 注意最前面有个空格的. 回车.

![image](https://user-images.githubusercontent.com/21376904/61181299-208d2f80-a657-11e9-8dea-5b8f89414a46.jpg)

> 成功进入:

![image](https://user-images.githubusercontent.com/21376904/61181329-87124d80-a657-11e9-88b1-b584c1dc1c4d.jpg)

> 之后安装好了, 要启动之前, 还需要shift + O. 然后输入``` noIOMMU```

> 最后进入系统后, 管理界面 > F2 进入配置 > 登录 > 故障解决选项 > 启用Esxi Shell > alt + F1 > 登录 > 执行:

```
esxcli system settings kernel set --setting=noIOMMU -v TRUE
```

> alt + F2退出, 然后可以再禁用ESXI shell.


----------


### 缺少网卡驱动

> 如果你用的官方镜像, 八成是缺少网卡驱动的. 你会看到下图:

![image](https://user-images.githubusercontent.com/21376904/61182549-67832100-a667-11e9-833b-225f77210d0f.jpg)

----------

### 安装ESXi6.7

> 然后就可以开始真正的安装了, 一路接受和同意, 然后选择硬盘, 这里先选240G的固态, 一会再扩展另一个480G的硬盘:

![image](https://user-images.githubusercontent.com/21376904/61181628-a57a4800-a65b-11e9-8c0c-6f9b3aa2ca11.jpg)

> 装完会如下图, 建议清除安装媒体:

![image](https://user-images.githubusercontent.com/21376904/61181662-fdb14a00-a65b-11e9-9964-81307e58d934.jpg)

----------

### Multiboot could not setup the video subsystem

> 一般来说, 安装完了, 启动之前还会看到这个错误, 主要是分辨率没达到, 需要进入BIOS调整. 找到CSM Configuration > Video, 改成UEFI.

----------

## vSphere Client

> 重启之后通过浏览器输入ip就可以进入vSphere Client.

### 添加硬盘

> 点击存储-数据存储

![image](https://user-images.githubusercontent.com/21376904/61182475-440ba680-a666-11e9-8441-f1e5e1926d05.png)
![image](https://user-images.githubusercontent.com/21376904/61182477-538aef80-a666-11e9-91b1-273a789e0bbd.png)
![image](https://user-images.githubusercontent.com/21376904/61182484-60a7de80-a666-11e9-9812-2562dec1e33e.png)

> 可以看到, 扩容成功了.

![image](https://user-images.githubusercontent.com/21376904/61182472-30f8d680-a666-11e9-83a0-747285561983.png)

----

### 建立虚拟机

> 建立虚拟机和平常的虚拟机软件操作类似, 分两步, 上传镜像到服务器, 然后建立对应虚拟机即可.

![image](https://user-images.githubusercontent.com/21376904/61182535-2ab72a00-a667-11e9-839b-c1e3348ccba3.png)
![image](https://user-images.githubusercontent.com/21376904/61182551-6eaa2f00-a667-11e9-97e8-40eb027df2cd.png)

> 选择镜像

![image](https://user-images.githubusercontent.com/21376904/61182742-2f311200-a66a-11e9-8a54-21f785668303.png)

![image](https://user-images.githubusercontent.com/21376904/61182776-b5e5ef00-a66a-11e9-9bb9-49b32f9ef144.png)

----------

### 显卡直通

> 先安装VMware Tools.
> 然后按照[官方文档指示开启](https://docs.vmware.com/cn/VMware-vSphere/6.7/com.vmware.vsphere.vm_admin.doc/GUID-5B3CAB26-5D06-4A99-92A0-3A04C69CE64B.html). 需要CPU支持Intel Virtualization Technology for Directed I/O (VT-d), 比如说我这里是**双Intel(R) Xeon(R) CPU E5-2696 v3 @ 2.30GHz** + **双GTX1080Ti交火**. 不支持就放弃吧, 支持的话, 进入BIOS开启这两项.

> 关于是否支持VT-d和VT-x, 可以查看[英特尔官方文档](https://ark.intel.com/content/www/cn/zh/ark/products/81060/intel-xeon-processor-e5-2698-v3-40m-cache-2-30-ghz.html), 

![image](https://user-images.githubusercontent.com/21376904/61182617-977ef400-a668-11e9-8c68-f76753bc39b6.png)

![image](https://user-images.githubusercontent.com/21376904/61182620-99e14e00-a668-11e9-8fd5-99b7be872283.png)

![image](https://user-images.githubusercontent.com/21376904/61577778-c260c080-ab1e-11e9-8174-08cd77d700fb.png)


> 我这边已经显卡直通了, 所以是显示活动. 如果是第一次激活直通, 需要勾选-切换直通, 系统进入维护模式-重新引导主机. 然后点击虚拟机, 添加PCI设备.

![image](https://user-images.githubusercontent.com/21376904/61577825-326f4680-ab1f-11e9-9642-2ae8f0eab277.png)

![image](https://user-images.githubusercontent.com/21376904/61577854-bb867d80-ab1f-11e9-9e4b-d0ba4df54655.png)

----------

## 最后

> 整体流程其实并不繁琐, 我花时间最久的地方就是镜像打包和bug修复, 后续还会继续更新这一块, 注意ESXi只有60天试用哦, 记得在过期前添加许可证(导航器-主机-管理-许可). 喜欢记得点赞哦, 有意见或者建议评论区见哦~

---------- 



