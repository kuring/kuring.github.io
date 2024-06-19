---
title: 微信公众号通过扣子接入大模型
date: 2024-06-19
---


在字节的 AI 发布平台[扣子](https://www.coze.cn/)中提供了创建机器人的功能，并且可以直接对接微信公众号，使用在微信公众号中回复消息，由大模型直接回复的效果。

# 操作步骤
整体操作步骤非常简单，需要申请一个 Coze 的账号和开通微信公众号的开发者功能。

微信公众号的开发者功能在这里配置：
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240619211112.png)

点击`创建 Bot` 按钮，输入 Bot 名称后点击`确认`。
![image.png|570](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240619210355.png)

在机器人设置页面，可以配置模型、选择自己训练的知识库等操作。
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240619210909.png)

设置完成后，点击`发布`，选择`微信公众号（订阅号）`配置功能，设置对应的微信公众号的 AppID。
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240619211019.png)
# 体验
访问公众号直接输入内容，可以看到自动回复内容。
![IMG_3856.PNG.JPG](https://kuring.oss-cn-beijing.aliyuncs.com/images/IMG_3856.PNG.JPG)
