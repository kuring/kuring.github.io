---
title: 流量控制算法
date: 2018-12-16 01:25:50
tags:
---

限流的方式有多种，每种都有其应用场景。

限制请求的方式包括：

1. 丢弃请求
2. 放在队列中，等有令牌后再请求
3. 走降级逻辑

## 计数器

我之前设计的流控系统，以每秒为单位，如果一秒内超过固定的QPS，则将请求进行降级处理。该算法已经在生产环境中平稳运行了很久，也确实满足了业务的需求。

计数器流控算法简单粗暴，有一个缺点，即流控的单位为秒，但一秒的请求很可能是不均匀的，不能进行更细粒度的控制，也不允许流量存在某种程度的突发。

## 漏桶算法

请求先进入漏桶中，漏桶以一定的速度出水，当水流的速度过大时会直接溢出。

漏桶大小：起到缓冲的作用

漏桶的出水速度：该值固定

## 令牌桶算法

令牌桶算法相比漏桶算法而言，允许请求存在某种程度的突发，常用于网络流量整形和速率限制。

系统会恒定的速度往令牌桶中注入令牌，如果令牌桶中的令牌满后就不再增加。新请求来临时，会拿走一个令牌，如果没有令牌就会限制该请求。

这里的请求可以代表一个网络请求，或者网络的一个字节。

涉及到的变量：

1. 网络请求平均速率r：每隔1/r秒向令牌桶中放入一个令牌，1秒共放入r个令牌
2. 令牌桶的最大大小：令牌桶慢后，再放入的令牌会直接丢弃

令牌相当于操作系统中信号量机制。

业界较为出名的流控工具当属Guava中的RateLimiter，基于令牌桶算法实现。

在实际的代码实现中，并不一定需要一个固定的线程来定期往令牌桶中放入令牌，而是在请求到来时，直接计算得出当前是否还有令牌。比如下面的python代码实现：

```python
import time


class TokenBucket(object):

    # rate是令牌发放速度，capacity是桶的大小
    def __init__(self, rate, capacity):
        self._rate = rate
        self._capacity = capacity
        self._current_amount = 0
        self._last_consume_time = int(time.time())

    # token_amount是发送数据需要的令牌数
    def consume(self, token_amount):
        increment = (int(time.time()) - self._last_consume_time) * self._rate  # 计算从上次发送到这次发送，新发放的令牌数量
        self._current_amount = min(
            increment + self._current_amount, self._capacity)  # 令牌数量不能超过桶的容量
        if token_amount > self._current_amount:  # 如果没有足够的令牌，则不能发送数据
            return False
        self._last_consume_time = int(time.time())
        self._current_amount -= token_amount
        return True
```

## ref

[15行Python代码，帮你理解令牌桶算法](https://juejin.im/post/5ab10045518825557005db65)
