title: 红米路由器 AX6000 解锁 SSH
date: 2024-02-11 00:20:15
tags:
author:
---
半年前购买了红米路由器 AX6000，用起来一直非常稳定。因为最近在折腾树莓派，准备将红米路由器上开启 SSH 的功能。最早是准备将路由器刷成 [OpenWRT 系统](https://openwrt.org/toh/xiaomi/redmi_ax6000)，当一旦路由器能够开启 SSH 功能后，刷 OpenWRT 系统的必要性就不是太大了。
> 声明：本文下面操作步骤均为从网络上获取的现有操作步骤。


在电脑的浏览器上登录红米路由器的管理页面，红米路由器管理页面为：[https://miwifi.com/](https://miwifi.com/)，或者 [http://192.168.31.1/](http://192.168.31.1/)。如果本地设置过其他的 DNS 服务器，需要使用 ip 地址的形式访问。
登录后可以获取到当前红米路由器的版本，我的已经是最新的 1.0.67，该版本的固件可以支持开启 SSH 协议。
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/common/ax6000-version.png)
在登录红米路由器管理页面后，查看浏览器的地址栏，`https://miwifi.com/cgi-bin/luci/;stok=5e55c58e949d5419c001bce8288e5a27/web/home#router`，可以看到参数 stok `5e55c58e949d5419c001bce8288e5a27` 即我们需要用到该值。需要注意的是参数 stok 在每次登录时均会发生改变。

用浏览器打开下面的网址，其中 stok 为上面步骤获取到的值。
```
http://192.168.31.1/cgi-bin/luci/;stok=5e55c58e949d5419c001bce8288e5a27/api/misystem/set_sys_time?timezone=%20%27%20%3B%20zz%3D%24%28dd%20if%3D%2Fdev%2Fzero%20bs%3D1%20count%3D2%202%3E%2Fdev%2Fnull%29%20%3B%20printf%20%27%A5%5A%25c%25c%27%20%24zz%20%24zz%20%7C%20mtd%20write%20-%20crash%20%3B%20
```
浏览器如果返回`{"code":0}`，说明该步骤执行成功。

在浏览器执行路由器重启操作，返回 `{"code":0}`，说明该步骤执行成功，此时路由器会发生重启。
```
http://192.168.31.1/cgi-bin/luci/;stok=5e55c58e949d5419c001bce8288e5a27/api/misystem/set_sys_time?timezone=%20%27%20%3b%20reboot%20%3b%20
```

重新登录红米路由器，获取到新的 stok 值 221bcd3ba09b3e4597b0308cc1df18a2。
在浏览器继续访问如下的地址，用来开启 telnet，成功后仍然返回 `{"code":0}`。
```
http://192.168.31.1/cgi-bin/luci/;stok=221bcd3ba09b3e4597b0308cc1df18a2/api/misystem/set_sys_time?timezone=%20%27%20%3B%20bdata%20set%20telnet_en%3D1%20%3B%20bdata%20set%20ssh_en%3D1%20%3B%20bdata%20set%20uart_en%3D1%20%3B%20bdata%20commit%20%3B%20
```

再继续执行重启命令
```
http://192.168.31.1/cgi-bin/luci/;stok=221bcd3ba09b3e4597b0308cc1df18a2/api/misystem/set_sys_time?timezone=%20%27%20%3b%20reboot%20%3b%20
```

在重启完成后，路由器的 telnet 服务已经开启。在终端中执行 `telnet 192.168.31.1` 即可远程连接到小米路由器，而且不需要密码。

在终端中执行如下的命令，用来设置 root 密码，开启并固化 ssh 服务：
```
# 修改 root 密码
echo -e 'wangss19881.\nwangss19881.' | passwd root

# 开启 ssh 服务
nvram set ssh_en=1
nvram set telnet_en=1
nvram set uart_en=1
nvram set boot_wait=on
nvram commit

sed -i 's/channel=.*/channel="debug"/g' /etc/init.d/dropbear
/etc/init.d/dropbear restart

# 设置 ssh 服务开机自启动
mkdir /data/auto_ssh
cd /data/auto_ssh
curl -O https://fastly.jsdelivr.net/gh/lemoeo/AX6S@main/auto_ssh.sh
chmod +x auto_ssh.sh
uci set firewall.auto_ssh=include
uci set firewall.auto_ssh.type='script'
uci set firewall.auto_ssh.path='/data/auto_ssh/auto_ssh.sh'
uci set firewall.auto_ssh.enabled='1'
uci commit firewall

# 设置时区
uci set system.@system[0].timezone='CST-8'
uci set system.@system[0].webtimezone='CST-8'
uci set system.@system[0].timezoneindex='2.84'
uci commit

# 关闭开发模式
mtd erase crash

reboot
```
在终端中执行 `ssh -o HostkeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa  root@192.168.31.1`即可 ssh 连接到路由器。
# 资料
[https://www.youtube.com/watch?v=u5Qg4zqj_V0](https://www.youtube.com/watch?v=u5Qg4zqj_V0)
[https://uzbox.com/tech/openwrt/ax6000.html](https://uzbox.com/tech/openwrt/ax6000.html)
[https://github.com/kjfx/AX6000/releases/tag/RedmiAX6000](https://github.com/kjfx/AX6000/releases/tag/RedmiAX6000)