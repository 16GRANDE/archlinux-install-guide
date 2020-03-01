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
  - 到https://www.archlinux.org/download/网页China镜像源中下载archlinux-**-x86_64.iso文件，可以MD5验证是否被篡改
  - linux系统下制作u盘，使用dd命令:
  ```
  http://www.runoob.com/linux/linux-comm-dd.html
  ```
  - win系统下制作u盘，可以使用UltraISO，rufus，usbwriter

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
  4.配置pacman
    ```
    vim /etc/pacman.conf  (把#Color那一行前面的注释去掉)
    vim /etc/pacman.d/mirrorlist  (寻找中国服务器，将它移动到mirrorlist的最顶上，保存退出，在vim里面可以用宏很方便地把所有的中国服务器移到前面)
                                  (我用的是清华源：## China
                                                  Server = http://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch)
    ```
  5.安装基本包
    - 挂载分区(这一步很重要，特别是引导分区，一定要确认好)
    ```
    mount -L ARCH-ROOT /mnt  (挂载没有标签的分区方法：mount /dev/sda4 /mnt)
    mkdir /mnt/boot/efi  (创建启动分区挂载目录，这个一定要确认好是创建/mnt/boot/efi还是/mnt/boot,我是因为之前安装的manjaro挂载目录就是/boot/efi,所以才创建的是/boot/efi)
    mount -L ARCH-BOOT /mnt/boot/efi
    ```

    - 安装基本包
      ```
      pacstrap /mnt base base-devel linux linux-firmware dhcpcd
      ```
  
  6.挂载Fstab
    生成自动挂载分区的fstab文件
    ```
    genfstab -L /mnt >> /mnt/etc/fstab
    cat /mnt/etc/fstab  (查看生成文件是否正确)
    ```

  7.使用arch-chroot对安装好的系统进行配置
    - 进入arch-chroot
    ```
    arch-chroot /mnt
    ```

    - 设置时区
    ```
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime  (创建链接)
    hwlock --systohc  (同步时间)
    ```

    - 安装必须软件包
    ```
    pacman -S vim dialog wpa_supplicant ntfs-3g networkmanager
    ```

    - 设置Locale
    ```
    vim /etc/locale.gen  (在文件中找到zh_CN.UTF-8 UTF-8,zh_HK.UTF-8 UTF-8,zh_TW.UTF-8 UTF-8,en_US.UTF-8 UTF-8这四行，去掉行首的#号，保存并退出)
    locale-gen
    vim /etc/locale.conf  (在文件的第一行加入以下内容：LANG=en_US.UTF-8)
    ```

    - 设置主机名
    ```
    vim /etc/hostname  (在文件的第一行输入你自己设定的一个myhostname,如：16GARNDE)
    vim /etc/hosts  (在文件末添加如下内容: 127.0.0.1	localhost
                                          ::1		localhost
                                          127.0.1.1	myhostname.localdomain	myhostname)
    ```

    - 设置root密码
    ```
    passwd  (按提示输入确认就行)
    ```

    - 安装Intel-ucode（非IntelCPU可以跳过此步骤）
    ```
    pacman -S intel-ucode
    ```

    - 安装Bootloader
    ```
    pacman -S os-prober ntfs-3g  (这两个包配合Grub检测已经存在的系统，自动设置启动选项)
    pacman -S grub efibootmgr  (接下来的命令为uefi+gpt引导方式配置，其他引导方式请参照参考文档)
    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub  (部署grub)
    grub-mkconfig -o /boot/grub/grub.cfg  (生成配置文件)
    vim /boot/grub/grub.cfg  (检查是否生成正确的配置文件，多系统要查看是否有其他系统入口)
    (如果没有生成正确的配置文件，查看/boot/efi目录是否有initramfs-linux-fallback.img initramfs-linux.img 
    intel-ucode.img vmlinuz-linux这几个文件，如果都没有，说明linux内核没有被正确部署，很有可能是/boot目录没有被正确挂载导致的。
    需要重新挂载，生成配置文件)
    ```

  8.重启
    ```
    exit
    umount /mnt/boot/efi (要先卸载/mnt/boot/efi，再卸载/mnt)
    umount /mnt
    reboot (重启，等到关机后电脑还未重启的瞬间拔掉u盘，让电脑启动移动硬盘的系统)
    ```

  9.享受和折腾属于你自己的archlinux