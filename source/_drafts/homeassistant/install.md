---
title: Home Assistant 部署
---
术语：
HA：Home Assistant 的简写。
# 什么是 Home Assistant
Home Assistant是一个开源的智能家居自动化平台，可以将多个智能家电和服务集成到一个单一的用户界面中。

主要的特点：
1. **开源和自由**：Home Assistant是完全开源的，这意味着任何人都可以自由下载、使用和修改它的源代码。
2. **丰富的集成**：它支持数千款设备和服务的集成，包括照明、安全系统、温控器、传感器、语音助手等，用户可以通过Home Assistant打造一个全面自动化的智能家居系统。
3. **自动化和场景**：用户可以设置自动化规则和场景，根据一系列条件或者时间对设备进行自动控制。
4. **用户友好的界面**：提供一个可定制的仪表板，可以通过拖放来组织设备和功能，可以非常容易地以视觉化的方式访问和控制其设备。
5. **跨平台支持**：客户端包括了 Web 页面、Android、IOS 等平台的 APP，可以通过手机随时随地访问。

# 你是否需要 Home Assistant
如果你同时具备如下条件，可以尝试使用 HA：
1. 具有一定的探索精神和折腾精神，同时懂一点计算机的技术，最好是会一点编程。HA 是个开源软件，搭建有一定的技术门槛和学习成本。
2. 家里有多种品牌的智能设备需要管理，比如米家、海尔等。
3. 家里有闲置的服务器，需要用来运行 HA 服务，确保服务器是可以长期运行的。服务器可以是 Raspberry Pi、旧版电脑、NAS 等硬件。

# 部署 Home Assistant
## 部署方式的选择
官方提供了如下四种方式：
![image.png|399](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240531234335.png)
1. HA OS：提供了独立的 HA 镜像，适用于独立的服务器或者虚拟机中部署，功能完整。
2. Supervised：以进程的方式运行在 Debian 系的操作系统设备，对 OS 的要求非常高，需要安装较多依赖包，甚至对 OS 的版本都有要求。比如在 ubuntu 24.04 上提示不支持该 OS，因为该 OS 版本过新。
3. Container：Docker容器技术安装Home Assistant Core。插件的安装和升级稍微复杂一些，但功能无缺失。官方的教程很多都是以 Supervised 模式开发的，并不太适合 Container 模式。
4. Core：适用于想要在现有Python环境上完全控制的高级用户。安装了Home Assistant的核心应用程序，不包括GUI配置界面。

我这里采用了 Container 的部署方式，部署和升级都较为简单，下面采用 Container 的方式来部署。

## 启动 HA 容器
在启动容器之前，需要先自行部署 docker。

执行如下命令即可启动 HA 服务：
```
docker run -d \
  --name homeassistant \
  --privileged \
  --restart=unless-stopped \
  -e TZ=Asia/Shanghai \
  -v /data/home-assistant:/config \
  -v /run/dbus:/run/dbus:ro \
  --network=host \
  ghcr.io/home-assistant/home-assistant:stable
```
需要注意下 `-v /data/home-assistant:/config` 参数，要将`/data/home-assistant`目录更换为服务器的目录，该目录为 HA 服务的数据目录。

在浏览器输入 `$ip:8123` 即可访问 HA 的 dashboard，其中 `$ip` 为服务器的 ip 地址。
![image.png|350](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601000539.png)

点击 **创建我的智能家居** 即开始用户创建：
![image.png|350](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601000911.png)

再接下来输入家的位置：
![image.png|434](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601000940.png)

到这一步即完成了相关的操作：
![image.png|477](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601002144.png)

# 安装 HACS
HACS 为 HA 的第三方扩展插件，它为Home Assistant用户提供了一个便捷的方式来发现、安装和管理自定义集成、插件和主题，为 HA 的必装软件。

参考 [HACS 的官方安装文档](https://hacs.xyz/docs/setup/download)，在服务上执行如下命令

```
docker exec -it homeassistant bash
wget -O - https://get.hacs.xyz | bash -
exit
docker restart homeassistant
```

在 **配置** -> **设备与服务** -> **添加集成** 中可以找到对应的插件，
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601003813.png)
在接下来的页面全部打上勾

![image.png|429](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601005820.png)

在接下来的页面会弹出激活页面，需要跳转到 [Github 认证页面]([https://github.com/login/device](https://github.com/login/device))，输入页面上提示的代码后即完成了认证。
![image.png|428](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601010057.png)
在认证完成后，即可在左侧的菜单栏中看到 HACS 页面了。
![image.png|360](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601010213.png)

点击后既可看到 HACS 市场的各类插件。
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601010244.png)

# 使用 HACS 安装插件举例：Config Editor

Config Editor 可以用来编辑 HA 的配置文件，该插件还是非常常用。

我们以 *Config Editor* 插件为例，来介绍通过 HACS 安装插件的过程。在输入框中输入 *Config Editor* 找到对应的插件。

点击右下角的 **DOWNLOAD** 按钮即可下载插件。
![image.png|415](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601011549.png)

下载完成后可以在 配置看到新的提醒，需要重启服务后该插件即可加载成功。
![image.png|437](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601012104.png)

在 **配置** -> **设备与服务** -> **添加集成** 搜索 *Config Editor*
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601012244.png)
提示需要在  configuration.yaml 中添加如下配置，configuration.yaml 位于 /data/home-assistant 目录下：
```
config_editor:
```

修改完成后需要重启 HA，在 **HACS** 中搜索 **Config Editor**，可以发现多了一个 **Config Editor Card** 的组件，选择下载。

接下来通过仪表盘功能将 Config Editor 做一个 dashboard。
在 **配置** -> **仪表盘** -> **添加仪表盘** -> **从新建仪表盘开始** 中，采用如下的配置参数：
![image.png|439](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601233915.png)
此时左侧的菜单中会一个**配置文件编辑器**的菜单，依次点击右上角的**编辑仪表盘** -> **添加卡片**，找到 **Config Editor Card**，并选择添加即可。此时可显示一个配置文件的编辑器卡片。
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240601234657.png)


# 添加设备
## 集成米家
Xiaomi Miot Auto
重启容器
# 集成海尔设备
在 [Github]( https://github.com/banto6/haier) 上下载 haier 插件，我这里使用的为最新的 v1.0.0 版本。该版本采用 Client Id 和 Token 的方式登录。

### 获取海尔 Client Id 和 Token


### 配置 Haier

下载完成并解压缩后，将解压缩后的 custom_components/haier 复制到 HA 的custom_components 目录下，并重启 HA。

在配置 -> 集成 -> 添加集成，找到 Haier，在下面的界面中输入 Haier 的 Client Id 和 Token 信息。
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240530003317.png)


## 配置彩云天气插件
在 HACS 中找到 **Colorfulclouds Weather Card** 插件。
安装[天气预报](https://github.com/hasscc/tianqi?tab=readme-ov-file)插件，里面提供了四种安装方式，我这里直接选择方法一。
配置 -> 添加集成中找到天气预报插件，要填写的内容如下：
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240531001345.png)
在界面编辑器中，添加卡片 -> 自定义卡片中找到 **Colorfulclouds Weather Lovelace**

![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240531001915.png)


# 链接
[玩转智能家居，homeassistant从入门到精通](https://www.bilibili.com/video/BV1h94y1w7oN/?vd_source=a3976009933b557cbb345571f11e76ad)
