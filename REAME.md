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
    dhcpcd(动态分配IP地址)
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
    ip link(扫描电脑可用的互联网设备)
    ip link set 设备名 up(设备名为无线网卡名字，例如wlo1)
    iwlist 设备名 scan | grep ESSID(查看可以连接的wifi，知道wifi的可以忽略)
    wpa_passphrase wifi名字 wifi密码 > internet.conf(生成配置文件)
    wpa_supplicant -c internet.conf -i 设备名 &(运行wpa_supplicant)
    dhcpcd &(动态分配IP地址)
    ping www.baidu.com(测试是否连通)
    ```