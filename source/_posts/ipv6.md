title: ipv6
date: 2022-07-14 10:24:00
tags:
author:
---
## ipv6的优势

1. 拥有更大的地址空间。
2. 点对点通讯更方便。由于ipv6地址足够多，可以不再使用NAT功能，避免NAT场景下的各种坑。
3. ip配置方便。每台机器都有一个唯一48位的mac地址，如果再增加一个80位的网段前缀即可组成ipv6地址。因此，在分配ip地址的时候，只需要获取到网段前缀即可获取到完整的ipv6地址了。
4. 局域网内更安全。去掉了arp协议，而是采用Neighbor Discovery协议。

## ipv6的数据包格式

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/ipv6-header.png)

分为报头、扩展报头和上层协议数据单元（PDU）三部分组成。

### 报头

固定为40个字节，相比于 ipv4 的可变长度报文更简洁。

- 版本：固定为4bit，固定值6。

- 业务流类别：8bit，用来表明数据流的通讯类别或者优先级。

- 流标签：20bit，标记ipv6路由器需要特殊处理的数据流，目的是为了让路由器对于同一批数据报文能够按照同样的逻辑来处理。目前该字段的应用场景较少。

- 净荷长度：16bit，扩展头和上次协议数据单元的字节数，不包含报头的固定 40 字节。

- 下一个头：8bit，每个字段值有固定含义，用来表示上层协议。如果上层协议为 tcp，那么该字段的值为 4。

- 跳限制：8bit，即跳数限制，等同于ipv4中的ttl值。

- 源ip地址：128bit

- 目的ip地址：128bit

### 扩展报头

扩展报头的长度任意

## ipv6地址

### 地址表示方法

1. 采用16进制表示法，共128位，分为8组，每组16位，每组用4个16进制表示。各组之间使用`:`分割。例如：1080:0:0:0:8:800:200C:417A。
2. 地址中出现连续的0，可以使用`::`来代替连续的0，一个地址中只能出现一次连续的0。例如上述地址可以表示为：1080::8:800:200C:417A。本地的回环地址可使用 `::1` 表示。
3. 如果ipv6的前面地址全部为0，可能存在包含ipv4地址的场景，可以使用ipv4的十进制表示方法。例如：0:0:0:0:0:0:61.1.133.1或者::61.1.133.1。

ip地址结构包含了64位的网络地址和64位的主机地址，其中64位的网络地址又分为了48位的全球网络标识符和16位的本地子网标识符。

### 网段表示方法

在ipv6中同样有网段的概念，如 2001:0:0:CD30::/60 ，其中前60位为前缀长度，后面的所有位表示接口 ID，使用 `::` 表示，但前面的两个0不能使用 `::` 表示。

### 地址分类

包括了如下地址类型，但跟 ipv4 不同的地方在于没有广播地址。

#### 单播地址

单播地址又可以分为如下类型：

##### 链路本地地址（LLA）

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/ipv6-lla.jpg)

类似于 ipv4 的私网地址段，但比 ipv4 的私网地址段范围更小，仅可以用于本地子网内通讯，不可被路由。以`fe80::/10`开头的地址段。设备要想支持 ipv6，必须要有链路本地地址，且只能设置一个。

在设备启动时，通常该地址会自动设置，也可以手动设置。自动生成的地址通常会根据 mac 地址有关，因为每个设备的 mac 地址都是唯一的。

##### 公网单播地址（GUA）

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/ipv6-uga.jpg)

类似于 ipv4 的公网地址。

地址范围：`2000::3` - `3fff::/16`，即以 2 或者 3 开头，用总 ipv6 地址空间的1/8。

通常情况下，公网路由前缀为 48 位，子网 id 为 16 位，接口 id 为 64 位。

##### 本地唯一单播地址（ULA）

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/ipv6-ula.jpg)

范围：`fc00::/7`，当前唯一有效的前缀为`fd00::/8`。只能在私网内部使用，不能在公网上路由。

其全网 id 的部分采用伪随机算法，可以尽最大可可能确保全局的唯一性，从而在两个网络进行并网的时候减少地址冲突的概率。

##### loopback地址

即 `::1`，等同于 ipv4 的 `127.0.0.1/8`

##### 未指定单播地址

即 `::`。

#### 多播地址（Multicast）

又叫组播地址，标识一组节点，目的为组播地址的流量会转发到组内的所有节点，类似于 ipv4 的广播地址。地址范围：`FF00::/8`。

#### 任意播地址（Anycast）

标识一组节点，所有节点的接口分配相同的 ip 地址，目的为组播地址的流量会转发到组内的就近节点。任意播地址没有固定的前缀。

## DNS服务和ipv6

### 双栈请求域名请求顺序

在开启ipv4/ipv6双栈的情况下，域名解析会同时发出A/AAAA请求，发出请求的先后顺序由/etc/resolv.conf的option中的inet6选项决定。

### ipv6与  /etc/hosts文件

通常在/etc/hosts文件中包含了如下的回环地址

```
::1             localhost6.localdomain6 localhost6
```



### 检测域名是否支持ipv6

#### dig aaaa方法

```yaml
dig aaaa  ipv6.google.com.hk

; <<>> DiG 9.10.6 <<>> aaaa ipv6.google.com.hk
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50248
;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;ipv6.google.com.hk.		IN	AAAA

;; ANSWER SECTION:
ipv6.google.com.hk.	21600	IN	CNAME	ipv6.google.com.
ipv6.google.com.	21600	IN	CNAME	ipv6.l.google.com.
ipv6.l.google.com.	300	IN	AAAA	2607:f8b0:4003:c0b::71
ipv6.l.google.com.	300	IN	AAAA	2607:f8b0:4003:c0b::64
ipv6.l.google.com.	300	IN	AAAA	2607:f8b0:4003:c0b::65
ipv6.l.google.com.	300	IN	AAAA	2607:f8b0:4003:c0b::8b

;; Query time: 83 msec
;; SERVER: 30.30.30.30#53(30.30.30.30)
;; WHEN: Mon Jun 20 10:45:37 CST 2022
;; MSG SIZE  rcvd: 209
```

#### 使用网站测试

使用该网址，将域名更换为对应的域名：http://ipv6-test.com/validate.php?url=http://www.microsoft.com

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/ipv6-check.png)

## 启用ipv6

ipv6特性可以设置在整个系统级别或者单个网卡上，默认启用。

### 系统级别

系统级别可以通过内核参数 net.ipv6.conf.all.disable_ipv6 来查看是否启用，如果输出结果为0，说明启用。可以修改该内核参数的值来开启或者关闭系统级别的ipv6特性。

也可以通过修改grub的内核参数来选择开启或者关闭，在  /etc/default/grub 中GRUB_CMDLINE_LINUX追加如下内容，其中xxxxx代表当前已经有的参数：

```yaml
GRUB_CMDLINE_LINUX="xxxxx ipv6.disable=1"
```

### 网卡级别

可以通过 `ifconfig ethx` 命令来查看网卡信息，如果其中包含了inet6，则说明该网卡启用了ipv6特性。

也可以通过 `sysctl net.ipv6.conf.ethx.disable_ipv6` 来查看网卡是否启用。其中ethx为对应的网卡名称。

### nginx支持ipv6

默认情况下，nginx不支持ipv6，要支持ipv6，需要在编译的时候指定 --with-ipv6 选项。

在编译完成后，通过nginx -V 查看需要包含 --with-ipv6 选项。

## 配置 ipv6

ifconfig  eth0  inet6 add 2607:f8b0:4003:c0b::71

## 参考资料

[Linux有问必答：如何在Linux下禁用IPv6](https://developer.aliyun.com/article/87556?spm=a2c6h.13813017.content3.1.7de919b7VnWCcD)



