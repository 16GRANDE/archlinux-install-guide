# archlinux-install-guide
# 前情提要
- 安装参考资料
  - 参考视频
    <https://www.bilibili.com/video/av81146687>

    <https://www.bilibili.com/video/av86485933>
  
  - 参考文档
    <https://github.com/NiiiKlaus/Get-my-Arch-Linux>

    <https://www.viseator.com/2017/05/17/arch_install/>

  - 感谢大佬
    [theniceboy](https://github.com/theniceboy)
    [NiiiKlaus](https://github.com/NiiiKlaus)
    [viseator](https://github.com/viseator)

  - 安装背景
    闲置一块500硬盘，加个硬盘盒做移动硬盘，拿来安装linux系统，插上电脑就可以用linux，之前已经安装了manjaro,想再装一个arch,开机可以切换manjaro,arch,win10。

  - 提醒
    archlinux 每个版本安装方法都可能有略微不同，最好是看过一遍archwiki的官方安装教程，或者是看一下viseator大佬的文档，该记录适用于archlinux-2020.02.01-x86_64.iso，强烈建议正式安装前先拿空硬盘或者虚拟机练下手。

# 目录
- 制作安装u盘
  -到https://www.archlinux.org/download/网页China镜像源中下载archlinux-**-x86_64.iso文件，可以MD5验证是否被篡改
  -linux系统下制作u盘，使用dd命令:
  ```
  http://www.runoob.com/linux/linux-comm-dd.html
  ```
  -win系统下制作u盘，可以使用UltraISO，rufus，usbwriter

- 设置启动顺序
  百度查询自己电脑品牌的进入bios键，进入bios设置u盘为第一启动项

- 系统安装
  1.连接网络(推荐有线网络，无线网络稍微麻烦一点)
    - 有线网络
    ```
    dhcpcd  (动态分配IP地址)
    ```
    - 判断网络是否连通
    ```
    ping www.baidu.com
    ```
    确认网络连通后用快捷键Ctrl+C终止ping命令

    - 无线网络
    无线网络推荐wifi-menu，简单明了
    ```
    wifi-menu
    ```
    如果wifi-menu用不了，可以用wpa_supplicant
    ```
    ip link  (扫描电脑可用的互联网设备)
    ip link set 设备名 up  (设备名为无线网卡名字，例如wlo1)
    iwlist 设备名 scan | grep ESSID  (查看可以连接的wifi，知道wifi的可以忽略)
    wpa_passphrase wifi名字 wifi密码 > internet.conf  (生成配置文件)
    wpa_supplicant -c internet.conf -i 设备名 &  (运行wpa_supplicant)
    dhcpcd &  (动态分配IP地址)
    ping www.baidu.com  (测试是否连通)
    ```

  2.更新系统时间
  ```
  timedatectl set-ntp true
  ```

  3.分区与格式化
    这部分要确认好自己要分的区，毕竟涉及数据，数据无价，输入命令时要谨慎。

    - 确认好自己的引导方式，我采用的是UEFI+GPT,现在大部分电脑都是采用这种引导方式，采用其他引导方式的可以参照参考文档

    - 查看当前分区状况
    ```
    fdisk -l
    ```
    - 确认自己要分的区
      我要分的有三个区：引导分区，SWAP分区，主分区，但是由于我这块硬盘已经安装了manjaro所以已经有了引导分区，所以我只要再分出两个区就行了。

    - 分区强烈推荐cfdisk，简单明了
    ```
    cfdisk 磁盘名  (如/dev/sda)
    ```

    - 不用cfdisk
    ```
    fdisk 磁盘名  (如/dev/sda)
    m  (查看各命令的作用)
    g  (创建一个全新的gpt分区表，如果不是一块空硬盘，则跳过这一步)
    n  (创建一个新的分区/dev/sda3 -- SWAP分区,用于保存内存中的文件以及作为内存的扩展，此分区不需要太大,我嫌硬盘空间大就分了20G）
       (接下来选择分区的编号、起始位置、终止位置（分区大小，可用例如“+300M”的形式)
    n  (创建一个新的分区/dev/sda4 -- 主分区)
    p  (查看待写入分区结果)
    w  (写入)
    ```

    - 定义分区格式
    ```
    mkfs.fat -F32 /dev/sda1   # /dev/sda1为引导分区
    mkfs.ext4 /dev/sda4       # /dev/sda2为主分区
    mkswap /dev/sda3          # /dev/sda3为SWAP分区
    ```

    - 打开swap
    swapon /dev/sda3

    - 给分区打标签(不是安装在移动硬盘上，磁盘名称不会改变的这一步可以跳过)
      由于是移动硬盘，磁盘名称随时会变，比如原本是sdd，换到另外一台电脑就是sdc，这样会导致挂载不到正确的硬盘，然后启动失败。所以我们要给硬盘分区打个标签，就是给个名字给分区例如ARCH-ROOT，这样电脑就知道挂载到标签为ARCH-ROOT的分区。
      ```
      e2label /dev/sda4 ARCH-ROOT  (for ext2/ext3/ext4)
      dosfslabel /dev/sda1 ARCH-BOOT  (for dos(vfat/fat16/fat32) 这个命令要求标签为大写)
      swaplabel -L ARCH-SWAP /dev/sda3  (for swap)
      ntfslabel (option) device [label] (for ntfs 这个命令没用过，相关信息请百度)
      ```
  5.配置pacman
    ```
    vim /etc/pacman.conf  (把#Color那一行前面的注释去掉)
    vim /etc/pacman.d/mirrorlist  (寻找中国服务器，将它移动到mirrorlist的最顶上，保存退出，在vim里面可以用宏很方便地把所有的中国服务器移到前面)
    ```