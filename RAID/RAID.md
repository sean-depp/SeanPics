# Ubuntu18.04软RAID 0 1 5 10建立(附gparted/live使用)

## 目录

* 前言
* 磁盘准备
* 创建RAID 0阵列
> * 格式化RAID
> * 保存RAID
> * 删除RAID

* 创建RAID 1阵列
* 创建RAID 5阵列
* 磁盘测速
* gparted live修改根目录大小
* 创建RAID 10阵列
* 最后

-----

## 前言

> 关于RAID可以参看[维基百科](https://zh.wikipedia.org/wiki/RAID),     或者我推荐这篇[博文](https://blog.51cto.com/zsk1987/1912625),  简单来说,  RAID把多个硬盘组合成为一个逻辑硬盘,  因此,  操作系统只会把它当作一个硬盘. RAID常被用在服务器计算机上,  并且常使用完全相同的硬盘作为组合. 由于硬盘价格的不断下降与RAID功能更加有效地与主板集成,  它也成为普通用户的一个选择,  特别是需要大容量存储空间的工作,  如: 视频与音频制作. 


| RAID等级  | 最少硬碟  | 最大容错  | 可用容量  | 读取效能  | 写入效能  | 安全性  | 目的   | 应用产业   | 
| :-----:  | :--------:   | :-----:   | :-----:   | :-----:   | :-----:   | :-----:  | :----- | :-----  | 
| 单一硬碟        | (参考)         | 0               | 1           | 1     | 1   | 无                           |                         |     |  
| JBOD        | 1         | 0               | n           | 1     | 1   | 无（同RAID 0）                           | 增加容量                         | 个人（暂时）储存备份 | 
| 0           | 2         | 0               | n           | n     | n   | 一个硬碟异常,  全部硬碟都会异常       | 追求最大容量、速度                   | 影片剪接快取用途 |
| 1           | 2         | n-1               |1            | n     |1    | 高,  一个正常即可 | 追求最大安全性 | 个人、企业备份 |
| 5           | 3         | 1               | n-1         | n-1  | n-1  | 高                                     | 追求最大容量、最小预算               | 个人、企业备份 |
| 6           | 4         | 2               | n-2         | n-2| n-2| 安全性较RAID 5高                       | 同RAID 5,  但较安全                   | 个人、企业备份 |     
| 10          | 4|  |  | n |  | 高 | 综合RAID 0/1优点,  理论速度较快 | 大型资料库、伺服器   |
|50|6|   |    |    |   |高   |提升资料安全   |   |
|60  |8  |   |   |  |   |高  |提升资料安全   |   |

1. n代表硬盘总数
2. JBOD（Just a Bunch Of Disks）指将数个物理硬盘,  在操作系统中合并成一个逻辑硬盘,  以直接增加容量
3. 依不同RAID厂商实现算法对于性能表现会有不同,  性能公式仅供参考
4.RAID10、50、60 依实现 Parity 不同公式也不同

> 但是很遗憾,   我的笔记本是没有那么多硬盘的,   为了完成演示,   我只能通过将单个磁盘进行分区来模拟.

-----

## 磁盘准备

> ***注意数据备份!!!***.
> ***注意数据备份!!!***.
> ***注意数据备份!!!***.

> 首先用指令看下目前磁盘情况:

```
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT
```

![](https://user-images.githubusercontent.com/21376904/61098108-65b33500-a490-11e9-9a1d-7956c21aea3e.png)

> 可以看到,  我这里是一个空的465G的固态,  然后被我划出3个5G的分区,  而且未进行文件系统的格式化. 但是你目前手上的磁盘基本不可能是这样的. 所以要先进行处理:
> 首先推荐安装GParted:

```
sudo apt-get install gparted
```

> 然后打开GParted,  这里就可以将分区删除,  然后看到是一整块固态,  然后用fdisk重新分区:

![](https://user-images.githubusercontent.com/21376904/61126893-417f4480-a4e0-11e9-9dbd-ad1148033ba2.png)

![](https://user-images.githubusercontent.com/21376904/61126916-4e039d00-a4e0-11e9-83a1-6ebd3ffa340b.png)

![](https://user-images.githubusercontent.com/21376904/61126918-5065f700-a4e0-11e9-9253-4c61c8511c1f.png)

![](https://user-images.githubusercontent.com/21376904/61098941-4b2e8b00-a493-11e9-98c6-f3b6c7f1dfad.png)

> 这里展示一下分区的操作,  最后分成3个5G的磁盘:

![](https://user-images.githubusercontent.com/21376904/61099284-5d5cf900-a494-11e9-8964-5bae7549ffd2.png)

> 在使用w保存之前,  都是可以用q进行撤销重来的:

![](https://user-images.githubusercontent.com/21376904/61099414-ce9cac00-a494-11e9-8dc8-e03d9fd3fcb7.png)

------

## 创建RAID 0阵列

> RAID 0: striping条带模式 特点: 在读写的时候可以实现并发, 所以相对其读写性能最好, 每个磁盘都保存了完整数据的一部分, 读取也采用并行方式, 磁盘数量越多, 读取和写入速度越快. 因为没有冗余, 一个硬盘坏掉全部数据丢失. 至少两块硬盘才能组成Raid0阵列.  
> 容量: 所有硬盘之和. 磁盘利用率为100%. 

![图片来自互联网](https://user-images.githubusercontent.com/21376904/61118534-82209300-a4cb-11e9-8730-bf912eccf879.gif)

> /dev/md0是磁盘名,  --level=0指的是RAID 0,  --raid-devices=3代表3个磁盘数,  /dev/sda{1, 2, 3}是磁盘名:

```
sudo mdadm --create --verbose /dev/md0 --level=0 --raid-devices=3 /dev/sda{1,2,3}
```

![](https://user-images.githubusercontent.com/21376904/61099593-71552a80-a495-11e9-8ce4-c5029c93923d.png)

> 用指令看下构建情况,  只要没有进度条,  就是构建完成:

```
cat /proc/mdstat
```

![](https://user-images.githubusercontent.com/21376904/61099703-d7da4880-a495-11e9-9bea-d44b9bd407bb.png)

-----

### 格式化RAID

> 这个格式化是通用操作,  包括之后的RAID 1,  RAID 5等等. 然后就是文件系统格式化,  建立文件夹,  挂载三连了:

```
sudo mkfs.ext4 -F /dev/md0
sudo mkdir -p /mnt/md0
sudo mount /dev/md0 /mnt/md0
```

![](https://user-images.githubusercontent.com/21376904/61099817-39021c00-a496-11e9-84b5-9f94d85a8197.png)

> 用```df -h -x devtmpfs -x tmpfs```查看下是否可用:

![](https://user-images.githubusercontent.com/21376904/61099890-7a92c700-a496-11e9-9185-3f609717c10b.png)

------

### 保存RAID

> 这个保存是通用操作,  包括之后的RAID 1,  RAID 5等等. 这样重启之后也会自动挂载. 注意名称上的对应,  因为你的命名可能与我不同:

```
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u
echo '/dev/md0 /mnt/md0 ext4 defaults, nofail, discard 0 0' | sudo tee -a /etc/fstab
```

![](https://user-images.githubusercontent.com/21376904/61100016-10c6ed00-a497-11e9-8ed6-7f26333d37a0.png)

-----

### 删除RAID

> 卸载, 停止RAID.

```
sudo umount /dev/md0
sudo mdadm --stop /dev/md0
```

> 查看下当前磁盘状况:

```
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT
```

![](https://user-images.githubusercontent.com/21376904/61100753-d6128400-a499-11e9-92dc-b9cbafdba3f6.png)

> 删除RAID并重置:

```
sudo mdadm --zero-superblock /dev/sda{1,2,3}
```

> 打开/etc/fstab, 删除之前输入的配置.

```
sudo vim /etc/fstab
```

![](https://user-images.githubusercontent.com/21376904/61100852-373a5780-a49a-11e9-99a2-6d3a3868b65d.png)

> 删除RAID定义:

```
sudo vim /etc/mdadm/mdadm.conf
```

![](https://user-images.githubusercontent.com/21376904/61100944-97c99480-a49a-11e9-9a40-8db1c6233125.png)

> 最后, 更新initramfs:

```
sudo update-initramfs -u
```

> 简单来说, 就是将之前的操作反向操作一波, 如果没有删干净, 会导致启动时出问题, 进入修复模式, 在修复模式中也可以再删除.

----

## 创建RAID 1阵列

![图片来自互联网](https://user-images.githubusercontent.com/21376904/61118501-6ddc9600-a4cb-11e9-9447-b7b4e6b19d39.gif)

> 查看下当前磁盘状况:

```
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT
```

![](https://user-images.githubusercontent.com/21376904/61117459-205f2980-a4c9-11e9-9f1b-08f7306c3746.png)

> 其实和创建RAID 0就差一个level:

```
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=3 /dev/sda{1,2,3}
```

> 我就不重复操作了, 直接跳到RAID 5吧.

-----

## 创建RAID 5阵列

> 要求: 至少3个存储设备

> * 主要好处: 具有更多可用容量的冗余. 
> * 需要注意的事项: 在分配奇偶校验信息时, 一个磁盘的容量将用于奇偶校验.  在处于降级状态时, RAID 5可能会遭受非常差的性能. 

![图片来自互联网](https://user-images.githubusercontent.com/21376904/61117870-1689f600-a4ca-11e9-88fc-6419c2cc309b.gif)

```
sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sda{1,2,3}
```

> RAID 5构建是比较慢的, 这里可以查看状态, 发现有进度条. 当然了, 我故意把大小设置成5G, 设置成200G, 这篇文章就没法写了.

```
cat /proc/mdstat
```

![](https://user-images.githubusercontent.com/21376904/61118176-bd6e9200-a4ca-11e9-9feb-a5b6e89a1f87.png)

> 等待完成.

![](https://user-images.githubusercontent.com/21376904/61118721-e3e0fd00-a4cb-11e9-830e-f6ca9af07002.png)

> 然后请回头查看格式化RAID, 和保存RAID, 不重复写了.

![](https://user-images.githubusercontent.com/21376904/61119177-d6784280-a4cc-11e9-9ded-949b984169a2.png)

## 磁盘测速

> 这里推荐hdparm指令.

```
hdparm -Tt /dev/md0
```

> 这样就可以测速了. 当然了, 我这样测速没什么意义, 因为我是分区然后制成RAID的, 不是通过多个硬盘. 当然了, 也可以看出sda是SATA3固态, sdb是M.2固态或者其他, whatever, 反正和mac的PCIE固态比起来都是弟弟.

![](https://user-images.githubusercontent.com/21376904/61119807-2d324c00-a4ce-11e9-954e-21b1e8b3a061.png)

----

## gparted live修改根目录大小

> 最后是RAID 10, 这是RAID 0和RAID 1的组合, 表现抢眼. 但是至少需要四块磁盘. 而一块硬盘只能分成3个主分区和一个扩展区, 也就是说, 无法靠当前磁盘分配进行演示.

> 思路就是从根目录所在固态借5G主分区出来. 但是根目录是不能再Linux启动的时候修改的, 这里就需要gparted live工具. 其实思路很简单, 就和装系统一样. [这里](https://sourceforge.net/projects/gparted/files/gparted-live-stable/)下载镜像, 用软件[Universal USB Installer](https://www.pendrivelinux.com/universal-usb-installer-easy-as-1-2-3/)进行刻录, 如下图. 然后BIOS进入U盘一路默认, 选择gparted工具, resize大小即可:

![](https://user-images.githubusercontent.com/21376904/61169825-279f3980-a594-11e9-8213-97038f50ce3e.PNG) 

----

## 创建RAID 10阵列

> 创建RAID 10思路也是一样一样的.

![图片来自互联网](https://user-images.githubusercontent.com/21376904/61169973-a34db600-a595-11e9-92e7-c17eeb41a341.gif)

```
sudo mdadm --create --verbose /dev/md0 --level=10 --raid-devices=4 /dev/sda{1,2,3} /dev/sdb3
```

> 等待完成

![](https://user-images.githubusercontent.com/21376904/61169703-2faaa980-a593-11e9-93e3-7882c26428aa.png)

> 再查看下磁盘状态.

![](https://user-images.githubusercontent.com/21376904/61169729-7ef0da00-a593-11e9-9218-46a8359d02de.png)

![](https://user-images.githubusercontent.com/21376904/61170058-9a111900-a596-11e9-9d19-c4943be0b3b3.png)

----

## 最后

> 花费最大精力的就是修改根目录大小, 查阅了很多资料, 也失败了很多次. 总之, 各位如果要下载软件之类, 尽量去官网下载, 避免不必要的麻烦. gparted/live真的是个神器, 不但在Linux好用, 其他OS, 比如macOS也是一样. 顺带解释一下, 为什么刻录的时候用的是Windows, 因为官网推荐的Tuxboot我安装之后打不开, 所以只能放弃. 喜欢记得点赞, 有意见或者建议评论区见哦~

-----





