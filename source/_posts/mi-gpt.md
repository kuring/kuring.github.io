---
title: 使用 mi-gpt 将小爱音箱接入大模型
date: 2024-06-13
---
无意间发现了开源项目 [mi-gpt](https://github.com/idootop/mi-gpt)，该项目可以将家里的小爱音箱接入到 GPT 中，增强小爱音箱的功能。在跟小爱音箱对话的过程中，可以根据特定的提示词走 GPT 来回答，而不是用小爱音箱原生的回复。

必备条件：
1. 必须有一个小米音箱。
2. 必须要有可以长期运行的服务器，可以是树莓派等设备。
3. 要有一个 OpenAI 的账号，也可以用兼容 ChatGPT API 的国内大模型。
# 部署

部署比较简单，下面为我的部署过程，供大家参考。更详细的信息大家可以直接参考 github 项目中的[相关文档](https://github.com/idootop/mi-gpt/tree/main/docs)。
## 创建配置文件 `.migpt.js`：

参考项目中的文件 [.migpt.example.js](https://github.com/idootop/mi-gpt/blob/main/.migpt.example.js)内容：
```
// 小爱音箱扮演角色的简介
const botProfile = `
性别：女
性格：乖巧可爱
爱好：喜欢搞怪，爱吃醋。
`;

// 小爱音箱主人（你）的简介
const masterProfile = `
性别：男
性格：善良正直
其他：总是舍己为人，是傻妞的主人。
`;

export default {
  bot: {
    name: "傻妞",
    profile: botProfile,
  },
  master: {
    name: "陆小千",
    profile: masterProfile,
  },
  speaker: {
    // 小米 ID
    userId: "12345", // 注意：不是手机号或邮箱，请在「个人信息」-「小米 ID」查看
    // 账号密码
    password: "xxx",
    // 小爱音箱 ID 或在米家中设置的名称
    did: "Redmi小爱触屏音箱8",
    // 当消息以下面的关键词开头时，会调用 AI 来回复消息
    callAIKeywords: ["请", "你", "傻妞"],
    // 当消息以下面的关键词开头时，会进入 AI 唤醒状态
    wakeUpKeywords: ["打开", "进入", "召唤"],
    // 当消息以下面的关键词开头时，会退出 AI 唤醒状态
    exitKeywords: ["关闭", "退出", "再见"],
    
    // 进入 AI 模式的欢迎语
    onEnterAI: ["你好，我是傻妞，很高兴认识你"],
    
    // 退出 AI 模式的提示语
    onExitAI: ["傻妞已退出"],
    
    // AI 开始回答时的提示语
    onAIAsking: ["让我先想想", "请稍等"],
    
    // AI 结束回答时的提示语
    onAIReplied: ["我说完了", "还有其他问题吗"],
    
    // AI 回答异常时的提示语
    onAIError: ["啊哦，出错了，请稍后再试吧！"],
    
    // 无响应一段时间后，多久自动退出唤醒模式（默认 30 秒）
    exitKeepAliveAfter: 30,
    
    // TTS 指令，请到 https://home.miot-spec.com 查询具体指令
    ttsCommand: [3, 1],
    
    // 设备唤醒指令，请到 https://home.miot-spec.com 查询具体指令
    wakeUpCommand: [3, 2],
    
    // 是否启用流式响应，部分小爱音箱型号不支持查询播放状态，此时需要关闭流式响应
    streamResponse: false,
    
    // 查询是否在播放中指令，请到 https://home.miot-spec.com 查询具体指令
    playingCommand: [2, 1, 1], // 默认无需配置此参数，播放出现问题时再尝试开启
  },
};
```

必须要修改的内容涉及到如下字段，其他字段可以根据含义来定义：
1. speaker.userId：小米的用户 ID。
2. speaker.password：小米的账号密码。
3. speaker.did：小爱音箱的 ID 或者小爱音箱在米家的设备名字。
4. ttsCommand：需要设置。如果设置不正常，会导致小爱音箱无法播放 GPT 回复内容的情况。
5. wakeUpCommand：需要设置。
6. streamResponse：在某些音箱设备上需要关闭。我的设备因为无法读完完整的句子，选择了关闭该功能，相关参考：[小爱音箱没有读完整个句子，总是戛然而止](https://github.com/idootop/mi-gpt/blob/main/docs/faq.md#q%E5%B0%8F%E7%88%B1%E9%9F%B3%E7%AE%B1%E6%B2%A1%E6%9C%89%E8%AF%BB%E5%AE%8C%E6%95%B4%E4%B8%AA%E5%8F%A5%E5%AD%90%E6%80%BB%E6%98%AF%E6%88%9B%E7%84%B6%E8%80%8C%E6%AD%A2)。

ttsCommand 和 wakeUpCommand 需要在 https://home.miot-spec.com 页面搜索对应的音箱型号
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240612234942.png)

点击规格后任选一个，选择 `Intelligent Speaker`，其中的 `[3, 1]` 对应的为 ttsCommand，`[3, 2]` 对应的为 wakeUpCommand。
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240612235825.png)

## 创建配置文件 `.env`

该文件中需要配置 OPENAI 的账号信息，我这里直接采用了阿里云的通义千问大模型服务，[API 是完全兼容的](https://help.aliyun.com/zh/dashscope/developer-reference/compatibility-of-openai-with-dashscope/)。

参考文档《[开通DashScope并创建API-KEY](https://help.aliyun.com/zh/dashscope/developer-reference/activate-dashscope-and-create-an-api-key)》 阿里云上开通大模型服务，获取到 API-KEY
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240611230914.png)

参考项目中的文件[.env.example](https://github.com/idootop/mi-gpt/blob/main/.env.example)
```

# OpenAI（也支持通义千问、MoonShot、DeepSeek 等模型）
OPENAI_MODEL=qwen-turbo
OPENAI_API_KEY=获取到的API_KEY
OPENAI_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1

# Azure OpenAI Service（可选）
# OPENAI_API_VERSION=2024-04-01-preview
# AZURE_OPENAI_API_KEY=你的密钥
# AZURE_OPENAI_ENDPOINT=https://你的资源名.openai.azure.com
# AZURE_OPENAI_DEPLOYMENT=你的模型部署名，比如：gpt-35-turbo-instruct

# 提示音效（可选，一般不用填，你也可以换上自己的提示音链接试试看效果）
# AUDIO_SILENT=静音音频链接，示例：https://example.com/slient.wav
# AUDIO_BEEP=默认提示音链接，同上
# AUDIO_ACTIVE=唤醒提示音链接，同上
# AUDIO_ERROR=出错了提示音链接，同上

# Doubao TTS（可选，用于调用第三方 TTS 服务，比如：豆包）
# TTS_DOUBAO=豆包 TTS 接口
# SPEAKERS_DOUBAO=豆包 TTS 音色列表接口
```
主要修改如下两个值：
1. OPENAI_API_KEY：即为上文获取到阿里云[模型服务灵积](https://help.aliyun.com/zh/dashscope/)的 API-KEY。
2. OPENAI_MODEL：支持的模型，可以在《[支持的模型列表](https://help.aliyun.com/zh/dashscope/developer-reference/compatibility-of-openai-with-dashscope/#7f9c78ae99pwz)》中查询模型列表。

## 启动服务

提供了两种方式：docker 和 宿主机 Node.js 方式运行，我自然会选择更加简洁的 docker 方式。

执行 `docker run -d --name mi-gpt --env-file $(pwd)/.env -v $(pwd)/.migpt.js:/app/.migpt.js idootop/mi-gpt:3.1.0` 即可本地运行。

# 功能演示

提问小爱同学：“请问一下太阳的重量是多少”，小爱同学可以顺利的回答出答案。

通过 `docker logs mi-gpt -f` 可以看到如下的输出日志：
```
2024/06/12 16:15:38 Speaker 🔥 请问一下太阳的重量是多少

2024/06/12 16:15:38 Speaker 🔊 让我先想想

2024/06/12 16:15:41 Open AI ✅ Answer: 傻妞: 哦，太阳的重量可大了，它是个恒星，比我们的地球重得多。科学家们用的是质量而不是重量来衡量，太阳的质量大约是地球的333,000倍，真是个超级大块头，想想如果它能变成棉花糖，那得多软多亮啊！不过，太阳对我们来说太遥远了，它的重量咱们还是别去抱了，哈哈。

2024/06/12 16:15:41 Speaker 🔊 傻妞: 哦，太阳的重量可大了，它是个恒星，比我们的地球重得多。科学家们用的是质量而不是重量来衡量，太阳的质量大约是地球的333,000倍，真是个超级大块头，想想如果它能变成棉花糖，那得多软多亮啊！不过，太阳对我们来说太遥远了，它的重量咱们还是别去抱了，哈哈
```