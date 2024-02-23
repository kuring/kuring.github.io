title: 将树莓派 5 打造成 Android TV 电视盒子折腾记 （2）- 遥控器的配置
date: 2024-02-24 00:41:46
tags:
author:
---
在前序文章中，实现了将树莓派 5 刷成了 Android TV 电视盒子，基本的功能已经完备，本文将来介绍如何使用遥控器来控制 Android TV。

- [我要看电视 - 投影仪和电视盒子选型](http://kuring.me/post/watch-tv/)
- [我要看电视 - 将树莓派 5 打造成 Android TV 电视盒子折腾记（1）](http://kuring.me/post/pi5-1/)

树莓派的定位是用作小型计算机使用，而作为电视盒子却必须要具备遥控器来控制的功能。要想通过遥控器来控制树莓派有多种方式可供选择，下面列一下我自己尝试过的方案。
# 使用手机遥控 Android TV
这是最简单成本最低的纯软方案，只要在手机上安装软件就可以来控制 Android TV 了，手机跟 Android TV 在同一个局域网下即可。我使用到了 IOS 上的 App `Remote TV`，软件免费，但有广告。<br />![IMG_2890.PNG](https://kuring.oss-cn-beijing.aliyuncs.com/raspberry/tv-2-1.png)<br />使用上功能完备，比如开关 Android TV 的功能也是完备的。Android TV 的关系操作并非真正的关机，实际上还是在低功耗运行，网络还是可以连接的。也就意味着通过 App 实际上是可以开机的。<br />不方便的地方在于，每次操作需要额外拿起手机打开 App 后才能操作，没有硬件的遥控器来的方便。
# 使用投影仪的遥控器
我使用了爱普生的 TZ2800 投影仪作为播放设备，投影仪的 HDMI 已经支持了 HDMI CEC 功能。这里吐槽下传统企业爱普生，在[官方产品页面](https://www.epson.com.cn/Apps/tech_support/SupportProduct.aspx?p=32158)中很难找到关于该款投影仪的 HDMI 是否支持 CEC 的介绍，或者存在某个地方，但总归我找不到。

HDMI CEC（Consumer Electronics Control） 功能可以实现通过 HDMI 连接让多个设备之间相互控制，可以实现同一个遥控器来控制投影仪和 Android TV 的功能：

1. 当关闭投影仪的时候也可以关闭 Android TV。
2. 当打开投影仪的时候可以打开 Android TV。

要想使用 HDMI CEC 功能，需要在 Android TV 系统中开启相关的功能。<br />![IMG_2893.HEIC](https://kuring.oss-cn-beijing.aliyuncs.com/raspberry/tv-2-2.HEIC)<br />下面的界面用来选择 HDMI CEC 的支持接口，只能支持一个 HDMI 接口。<br />![IMG_2894.HEIC](https://kuring.oss-cn-beijing.aliyuncs.com/raspberry/tv-2-3.HEIC)

由于跟投影仪共用一个遥控器，功能上还是有所受限。比如音量的大小功能，只能操作投影仪，而不能再设置 Android TV 的音量大小。但绝大多数的功能已经具备。
# 使用蓝牙遥控器
家里正巧有个办移动宽带送的电视盒子，该遥控器为蓝牙遥控器。在遥控器的背面正巧有蓝牙配对的方法。<br />![IMG_2891.HEIC](https://kuring.oss-cn-beijing.aliyuncs.com/raspberry/tv-2-4.heic)<br />打开 Android Tv 系统中的 `设置` -> `系统` -> `遥控器和手柄` 功能。同时按住菜单键和返回键即可蓝牙配对。<br />![IMG_2892.HEIC](https://kuring.oss-cn-beijing.aliyuncs.com/raspberry/tv-2-5.HEIC)<br />蓝牙配对成功后，使用过程中功能正常。但发现个致命的缺点：蓝牙会自动断开，而且无法自动连接，下次只能重新配对。我不太确定是否为遥控器的问题，但单这一点已经否定了我使用蓝牙遥控器的方案。

# 使树莓派设备支持遥控器控制
树莓派通过 24 个 GPIO 引脚提供了强大的扩展性，可以接入外部硬件设备来接收遥控器的信号。<br />我们来了解下常见的遥控器的实现原理：

1. 红外遥控器：最为普遍的遥控器，使用红外线发射信号，接收端需要有对应的红外接收模块。在发射信号时需要对准接收端。
2. 无线遥控器：通常采用了 2.4 GHz 无线频段作为传输介质，接收端需要有可以匹配的设备接收信号，比如很多无线鼠标提供了一个很小的 USB 接收端。
3. 蓝牙遥控器：使用蓝牙技术通讯，而蓝牙技术同样采用了 2.4 GHz 的无线频段，但好处在于很多的设备都支持蓝牙协议，不需要接收端再有一个定制化的硬件。

## 接收红外信号
我家里正巧有几个废弃的红外遥控器，因此可以作为树莓派的遥控器来使用。剩下的事情首先需要树莓派通过 GPIO 引脚接收到红外信号。<br />在万能的淘宝上，花了几块钱买到了红外接收器和杜邦线（用来连接红外接收头和 GPIO 引脚），杜邦线需要注意为母对母规格。<br />![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/raspberry/tv-2-6.png)<br />![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/raspberry/tv-2-7.png)

## 红外接收器连接到树莓派
在默认情况下，树莓派 GPIO 引脚的功能并未开启。打开 Android TV 系统中的 `Raspberry Pi settings` -> `IR` -> `Infrared remote`开关。可以看到接收信号的为 GPIO 18 引脚。<br />![IMG_2887.HEIC](https://kuring.oss-cn-beijing.aliyuncs.com/raspberry/tv-2-8.heic)<br />在红外接收器上包含了三个引脚，依次为：OUT（输出信号）、GND（接地信号）、VCC（工作电压），其中红外接收器的 OUT 引脚对应的 GPIO 18 引脚。<br />
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/raspberry/tv-2-9.png)<br />树莓派提供了 3.3V 和 5V 两种工作电压，查看红外信号接收器的工作电压在 2.7V ~ 5.5V 之间，我直接使用了 GPIO 的 5V 的引脚。<br />
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/raspberry/tv-2-10.png)<br />查看树莓派的各个引脚线路图：<br />![来源：https://pinout.vvzero.com/](https://kuring.oss-cn-beijing.aliyuncs.com/raspberry/tv-2-11.png "来源：https://pinout.vvzero.com/")<br />即可得到红外信号接收器跟 GPIO 引脚的对应关系：

| 红外信号接收器引脚 |  GPIO 引脚 |
| --- | --- |
| OUT | 18 |
| GND | 9（就近原则） |
| VCC | 2（4 和 6 被风扇电源占用） |

在将硬件连接好后就可以启动树莓派了。
## 遥控器编码配对
家里可能有多个红外遥控器，那么为什么红外遥控器之间不会出现相互冲突，本来想打开电视，却打开了空调的情况呢？原因是红外遥控器上的每个按键都对应了一个编码，而不同设备的遥控器对应的编码是不同的。电视遥控器发出的开机信号虽然被空调接收到了，但是并不会对该按键的编码做处理，空调也就不会被开机。<br />我从家里随便找了一个红外遥控器也就不可能直接用来控制 Android TV 了，因此需要让 Android TV 可以识别到我的红外遥控器对应的按键编码。<br />要想识别到红外遥控器的编码，需要使用到 Android TV 系统中命令行工具 `ir-keytable`，需要以 ssh 的方式连接 Android TV。Android TV 的连接方式可以参考我之前文章《[我要看电视 - 将树莓派 5 打造成 Android TV 电视盒子折腾记（1）](http://kuring.me/post/pi5-1/)》中的 SSH 章节部分。

在命令行执行 `ir-keytable -p all -t`，并按下遥控器的按键，在命令行中可以获取到类似如下的输出：
```
991.988018: lirc protocol(necx): scancode = 0x835590
991.988023: event type EV_MSC(0x04): scancode = 0x835590
991.988023: event type EV_SYN(0x00).

992.104019: lirc protocol(necx): scancode = 0x835590
992.104025: event type EV_MSC(0x04): scancode = 0x835590
992.104025: event type EV_SYN(0x00).
```
其中 916.968014 表示的事件发生的时间戳。necx 表示使用的为 NEC 扩展协议，NEC 为一种常见的红外编码协议。而 scancode 字段即为我们需要获取的按键编码。EV_SYN 表示一组时间的结束。<br />在上面的命令中，按下遥控器的按键一次，发送出了两个相同信号，也就打印出了两个相同日志块。

打开 Github 项目：[https://github.com/lineage-rpi/android_external_ir-keytable/tree/lineage-18.1/rc_keymaps](https://github.com/lineage-rpi/android_external_ir-keytable/tree/lineage-18.1/rc_keymaps)，可以看到里面有很多编码跟对应键之间的映射关系，因为我的红外遥控器没有数字键，我挑选了一个跟遥控器键比较相似的文件进行重新修改，将其中的编码修改我遥控器对应的编码，并保存到本地文件 rc_keymap.txt 中。我的文件类似如下，其中 KEY_PREVIOUSSONG 和 KEY_PLAYPAUSE 两个按键我的遥控器上没有，我这里未做修改。
```
# table allwinner_ba10_tv_box, type: NEC
0x646dca KEY_UP
0x646d81 KEY_VOLUMEDOWN
0x217 KEY_NEXTSONG
0x646ddc KEY_POWER
0x646dc5 KEY_BACK
0x646dce KEY_OK
0x646dd2 KEY_DOWN
0x646d80 KEY_VOLUMEUP
0x254 KEY_PREVIOUSSONG
0x255 KEY_PLAYPAUSE
0x646d82 KEY_MENU
0x646d88 KEY_HOMEPAGE
0x646dc1 KEY_RIGHT
0x646d99 KEY_LEFT
```
需要注意的是，第一行看似是个注释，实际不是，不要删掉，保留原文件中的格式不变。
## Android TV 系统开机时加载配置
有了遥控器按键和编码之间的映射关系后，需要将其放到 Android TV 系统的 `/boot/rc_keymap.txt` 文件中，让 Android TV 开启自动加载该配置。该文件默认情况下不存在需要创建。

分区 /boot 默认为只读模式，不允许在其中增加文件：
```
127|:/boot # echo "123" > aa
sh: can't create aa: Read-only file system
```

将 /boot 目录重新 mount 为读写模式：
```
$ mount  | grep '/boot '
/dev/block/mmcblk0p1 on /boot type vfat (ro,relatime,fmask=0000,dmask=0000,allow_utime=0022,codepage=437,iocharset=ascii,shortname=mixed,errors=remount-ro)

$ mount -o remount,rw /boot

$ mount  | grep '/boot '
/dev/block/mmcblk0p1 on /boot type vfat (rw,relatime,fmask=0000,dmask=0000,allow_utime=0022,codepage=437,iocharset=ascii,shortname=mixed,errors=remount-ro)
```

创建并将配置写入到文件 /boot/rc_keymap.txt 中：
```
cat << EOF > /boot/rc_keymap.txt
# table allwinner_ba10_tv_box, type: NEC
0x646dca KEY_UP
0x646d81 KEY_VOLUMEDOWN
0x217 KEY_NEXTSONG
0x646ddc KEY_POWER
0x646dc5 KEY_BACK
0x646dce KEY_OK
0x646dd2 KEY_DOWN
0x646d80 KEY_VOLUMEUP
0x254 KEY_PREVIOUSSONG
0x255 KEY_PLAYPAUSE
0x646d82 KEY_MENU
0x646d88 KEY_HOMEPAGE
0x646dc1 KEY_RIGHT
0x646d99 KEY_LEFT
EOF
```

重新将 /boot 分区挂载为只读模式：
```
mount -o remount,ro /boot
```

在执行完后，重新启动系统。在顺利的情况下，即可以通过红外遥控器直接操作 Android TV 系统。
# 总结
到目前为止，我主要在使用的方式为红外遥控器模式，毕竟一个电视盒子还是应该有属于自己的遥控器。Good Luck！