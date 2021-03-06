---
layout:     post
title:      "熔断"  
subtitle:   "Hystrix的实现原理"
author:     Sun Jianjiao
header-style: text
catalog: true
tags:
    - java

---

## Hystrix 简介

### Hystrix具备哪些能力/优点

在通过网络依赖服务出现高延迟或者失败时，为系统提供保护和控制
可以进行快速失败，缩短延迟等待时间和快速恢复：当异常的依赖回复正常后，失败的请求所占用的线程会被快速清理，不需要额外等待
提供失败回退（Fallback）和相对优雅的服务降级机制
提供有效的服务容错监控、报警和运维控制手段

### Hystrix 如何解决级联故障/防止服务雪崩

Hystrix将请求的逻辑进行封装，相关逻辑会在独立的线程中执行

Hystrix有自动超时策略，如果外部请求超过阈值，Hystrix会以超时来处理

Hystrix会为每个依赖维护一个线程池，当线程满载，不会进行线程排队，会直接终止操作
Hystrix有熔断机制： 在依赖服务失效比例超过阈值时，手动或者自动地切断服务一段时间

## 二、Hystrix 中基于自反馈调节熔断状态的算法原理

我们可以把熔断器想象为一个保险丝，在电路系统中，一般在所有的家电系统连接外部供电的线路中间都会加一个保险丝，当外部电压过高，达到保险丝的熔点时候，保险丝就会被熔断，从而可以切断家电系统与外部电路的联通，进而保障家电系统不会因为电压过高而损坏。

Hystrix提供的熔断器就有类似功能，当在一定时间段内服务调用方调用服务提供方的服务的次数达到设定的阈值，并且出错的次数也达到设置的出错阈值，就会进行服务降级，让服务调用方之间执行本地设置的降级策略，而不再发起远程调用。但是Hystrix提供的熔断器具有自我反馈，自我恢复的功能，Hystrix会根据调用接口的情况，让熔断器在closed,open,half-open三种状态之间自动切换。

- open状态说明打开熔断，也就是服务调用方执行本地降级策略，不进行远程调用。
- closed状态说明关闭了熔断，这时候服务调用方直接发起远程调用。
- half-open状态，则是一个中间状态，当熔断器处于这种状态时候，直接发起远程调用。

### 三种状态的转换

- closed->open:正常情况下熔断器为closed状态，当访问同一个接口次数超过设定阈值并且错误比例超过设置错误阈值时候，就会打开熔断机制，这时候熔断器状态从closed->open。
open->half-open:当服务接口对应的熔断器状态为open状态时候，所有服务调用方调用该服务方法时候都是执行本地降级方法，那么什么时候才会恢复到远程调用那？Hystrix提供了一种测试策略，也就是设置了一个时间窗口，从熔断器状态变为open状态开始的一个时间窗口内，调用该服务接口时候都委托服务降级方法进行执行。如果时间超过了时间窗口，则把熔断状态- 从open->half-open,这时候服务调用方调用服务接口时候，就可以发起远程调用而不再使用本地降级接口，如果发起远程调用还是失败，则重新设置熔断器状态为open状态，从新记录时间窗口开始时间。
- half-open->closed: 当熔断器状态为half-open,这时候服务调用方调用服务接口时候，就可以发起远程调用而不再使用本地降级接口，如果发起远程调用成功，则重新设置熔断器状态为closed状态。

那么有一个问题，用来判断熔断器从closed->open转换的数据是哪里来的那？其实这个是HystrixCommandMetrics对象来做的，该对象用来存在HystrixCommand的一些指标数据，比如接口调用次数，调用接口失败的次数等等，后面我们会讲解。
