---
layout:     post
title:      "分布式系统简介"  
subtitle:   "分布式系统简介和注意事项"
author:     Sun Jianjiao
header-img: "img/bg/default-bg.jpg"
catalog: true
tags:
    - java

---

# 1. 分布式系统

## 1.1 “单程序 + 单数据库”为什么也是分布式系统
https://mp.weixin.qq.com/s/eCpJkydT87U7UwvzvOdcGA

想象一下，下面的场景是否在“单程序 + 单数据库”项目中出现过？
- log 记录执行成功，但是数据库的数据没发生变化；
- 进程内的缓存数据更新了，但是数据库更新失败了。

因为我们所编写的程序运行时所在的进程，和程序中使用到的数据库所在的进程，并不是同一个。也因此导致了，让这两个进程（系统）完成各自的部分，而后最终完成一件完整的事，变得不再像由单个个体独自完成这件事那么简单。
所以，我们可以这么理解，**涉及多个进程协作才能提供一个完整功能的系统就是“分布式系统”**。

我们在思考“单程序 + 单数据库”项目中遇到的这些问题背后的原因和解决它的过程时，与我们在一个成熟的分布式系统中的遭遇是一样的，例如**数据一致性**。当然，这只是分布式系统核心概念的冰山一角。

**看待一个“分布式系统”的时候，内在胜于表象。以及，只要涉及多个进程协作才能提供一个完整功能的系统，就是“分布式系统”**

## 1.2  价值

**如果你的系统需承载的计算量的增长速度大于摩尔定律的预测，那么在未来的某一个时间点，集中式系统将无法承载你所需的计算量**。
**无论是要以低价格获得普通的性能，还是要以较高的价格获得极高的性能，分布式系统都能够满足。并且受规模效应的影响，系统越大，性价比带来的收益越高**。

- 而这只是一个内在因素，真正推动分布式系统发展的催化剂是“经济”因素。
人们发现，用廉价机器的集合组成的分布式系统，除了可以获得超过 CPU 发展速度的性能外，花费更低，具有更好的性价比，并且还可以根据需要增加或者减少所需机器的数量

- 我们看到了分布式系统相比集中式系统的另一个更明显的优势：更高的可用性。例如，有 10 个能够承载 10000 流量的相同的节点，如果其中的 2 个挂了，只要实际流量不超过 8000，系统依然能够正常运转。

## 1.3 本质
https://mp.weixin.qq.com/s/DXmQb9t29VKUQYGUzL68iQ

而这一切的价值，都是建立在分布式系统的“分治”和“冗余”之上的。

## 1.4 分治
这么做的好处是：问题越小越容易被解决，并且，只要解决了所有子问题，父问题就都可以被解决了。但是，这么做的时候，需要满足一个最重要的条件：**不同分支上的子问题，不能相互依赖，需要各自独立**。因为一旦包含了依赖关系，子问题和父问题之间就失去了可以被“归并”的意义。在软件开发领域，我们把这个概念称为“耦合度”和“内聚度”，这两个度量概念非常重要。

## 1. 5冗余
这里的冗余并不等同于代码的冗余、无意义的重复劳动，而是我们有意去做的、人为增加的重复部分。其目的是容许在一定范围内出现故障，而系统不受影响

单纯地为了备用而做的冗余，最大的弊端是，如果没有出现故障，那么冗余的这部分资源就白白浪费了，不能发挥任何作用。所以，我们才提出了诸如双主多活、读写分离之类的概念，以提高资源利用率。

这样的“分治”方案耦合度和内聚度是否最优，这样做“冗余”带来的收益是否成本能够接受。只要持续带着这些思考，我们就好像拿着一杆秤，基于它，我们就可以去衡量各种变量影响，然后作权衡。比如成本、时间、人员、性能、易维护等等。也可以基于它去判断什么样的框架、组件、协议更适合当前的环境

## 1.6 注意的问题
https://mp.weixin.qq.com/s/SxSfWOn67uv0hwN81SCroA

### 1.6.1 网络并不是可靠的
当前节点所依赖的其他节点由于各种原因无法与之正常通信时，该如何保证其依然能够提供部分或者完整的服务

### 1.6.2 不同节点之间的通信是存在延迟的
所以我们**不能以调用本地方法一样的认知去理解远程调用（RPC）**。在目前的技术环境下，CPU 的运算能力长期保持高速增长，因此两个节点间通讯的大部分场景中，延迟的耗时总是大于目标节点进行逻辑运算的耗时。
由此可得，**并不是将一个集中式系统拆分得越散，系统就越快**。当中存在的延迟，不容忽视。如果是基于提速为目的的拆分，至少是满足下面这样的条件的。

关于这点，我们还需要慎重考虑是否有必要进行“同步”的 RPC 调用，以及尽量的降低调用次数等。这就是为什么我们说，**循环调用不如一次批量调用，反复调用不如调用一次做进程内缓存，同步调用不如异步调用**的道理。

### 1.6.3 带宽是有上限的
举个例子，你的网络都是基于 100M 带宽标准搭建的。那么此时如果每一秒会产生 10240 次数据传输，平均每个数据包大小是 10KB，我们来看下是否会遇到瓶颈。
```
100M 带宽的理论传输速率是 12.5MB/ 秒。
那么现在每一秒需要传输：10240 次 / 秒 * 10KB = 100MB/ 秒。很明显，带宽远远不够了。
```

关于带宽上限，你需要明白这几点：
- 实际环境中的传输速率大小，是由服务商所提供的带宽大小，以及网卡、网卡、交换机、路由器所支持的传输速率中的最小值决定的。
- 内网与外网传输速率是不同的，一般都是内网大于外网。因为服务商所提供的带宽成本更高。
- 同一个局域网内的节点是公用外网出入口的，所以尽可能的缩小在外网传输的数据，以降低占用“独木桥”外网的空间。

### 1.6.4 分布式并不直接意味着是“敏捷”了
可能你曾经有过这样的想法，当在规模较大的集中式系统中工作的时候，每次和许多人在一个代码库里提交代码，老是遇到冲突、排队等待上游模块先开发等等。这时你会想，如果改造成分布式系统，这些问题都没了，工作效率高多了。

答案是否定的，在前两篇文章中也有提到一些。拆分后需要做的额外工作如果没做好，可能会导致不是更快，而是更慢。最典型的现象如：

- **发布更麻烦了**。原先虽然开发麻烦，但是发布简单啊，哪怕用最原始的方式：编译一次，登上服务器，复制黏贴，秒秒钟搞定。而现在需要发布 10 个、20 个、几十个，再这样操作很明显要把人逼疯。

- **排查问题更难了**。原本出了问题，不是在程序就是在数据库。现在还得判断在哪个程序，哪个数据库，是不是要抓狂？

关于这点，你需要秉持着工欲善其事必先利其器的思想。将建设协作相关的辅助性工作与分布式系统同时进行。比如：监控告警系统、配置中心、服务发现，以及批量部署、持续集成，甚至 DevOps 等。

### 1.6.5 数据由一份冗余成多份后如何保持一致
这点其实是由于前面提到的网络因素产生的连带效应。

当遇到数据库压力增大，响应开始变慢的时候，你可能会很容易想到，让 DBA 来做个主从啊。但是，由于网络不可靠、存在延迟、带宽有上限这一系列因素。所以，这个看似只是 Copy 一下工作，需要我们花大量的精力去解决**如何保证在使用不同副本上的数据的时候，都是符合应有的预期的**。
关于这点，业界已经研究了几十年，同时得出了许多具有指导意义的理论和思想。你需要充分的理解这些，并且能够识别出合适的运用场景，就可以解决这个问题。这个概念在软件领域被定义为「数据一致性」

### 1.6.6 整个系统的不同部分可能是异构的
在系统初期，同类技术下的差异往往不太被关注。但是随着分布式系统规模的逐渐扩大，在进入到成熟期的过程中，往往不可避免的会使系统成为异构的，有主动的，也有被动的。

- 被动的。往往需要引入一些新的技术栈或者解决方案来解决当前的问题，因为很多时候不会有资源，也没有必要去“重复造每一个轮子”。

- 主动的。由于规模效应的影响越来越大，某些技术更适合在特定场景下发挥，因此能够降低同等压力下对资源的消耗，就可以明显的降低成本。

关于这点，你要思考如何通过专制的方式进行标准化，来屏蔽这些差异带来的复杂度影响，使得有更多的精力投入到有价值的地方去。专制的特点是“约束”效果，标准化通过专制来进行可以降低许多为了方便而妥协问题。比如每个节点都通过统一的远程调用（RPC）中间件来做“连接”，就将如何连接异构节点的问题交由这个中间件来全权负责，而不是不同的团队各自实现一套。

# 2. redis 分布式锁

官方文档给出了单例的操作方式：[https://redis.io/topics/distlock](https://redis.io/topics/distlock)

结合官方的命令行代码，编写了基于spring-data-redis版本的单例redis分布式锁。

## 2.1 分布式锁的接口

```Java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.stereotype.Component;

import java.util.Collections;
import java.util.concurrent.TimeUnit;

@Component
public class DistributeLock {
    private static final Long RELEASE_SUCCESS = 1L;

    @Autowired
    private StringRedisTemplate redisTemplate;

    // 加锁
    public boolean tryLock(String lockKey, String requestId, long expireTime) {
        return redisTemplate.opsForValue().setIfAbsent(lockKey, requestId,  expireTime, TimeUnit.SECONDS);
    }

    // 解锁
    public Boolean unLock(String lockKey, String requestId) {

        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then " +
                            "return redis.call('del', KEYS[1]) " +
                        "else " +
                            "return 0 " +
                        "end";

        DefaultRedisScript<Boolean> redisScript =new DefaultRedisScript<>();

        redisScript.setScriptText(script);
        redisScript.setResultType(Boolean.class);

        return redisTemplate.execute(redisScript, Collections.singletonList(lockKey), requestId);
    }
}
```

枷锁的时超时时间一定设置的时候传过去，否则如果再设置超时时间之前服务挂掉，会导致死锁。

## 2.2 测试样例代码

```Java
import com.springexample.rediscache.utils.DistributeLock;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.UUID;
import java.util.concurrent.TimeUnit;

@RunWith(SpringRunner.class)
@SpringBootTest
public class DistributeLockTests {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    @Autowired
    DistributeLock distributeLock;

    @Test
    public void redisTemplateTest() {
        String requestId = UUID.randomUUID().toString();
        distributeLock.tryLock("test", requestId, 120);

        // 保存字符串
        stringRedisTemplate.opsForValue().set("aaa", "111", 1, TimeUnit.HOURS);
        Assert.assertEquals("111", stringRedisTemplate.opsForValue().get("aaa"));

        Assert.assertEquals(true, distributeLock.unLock("test", requestId));
    }
}
```

# 3. 分布式事务

## 3.1 提供回滚接口
在服务化架构中，功能 X，需要去协调后端的 A、B 甚至更多的原子服务。那么问题来了，假如 A 和 B 其中一个调用失败了，那可怎么办呢？ 

在笔者的工作中经常遇到这类问题，往往提供了一个 BFF 层来协调调用 A、B 服务。如果有些是需要同步返回结果的，我会尽量按照“串行”的方式去调用。如果调用 A 失败，则不会盲目去调用 B。如果调用 A 成功，而调用 B 失败，会尝试去回滚刚刚对 A 的调用操作。 

当然，有些时候我们不必严格提供单独对应的回滚接口，可以通过传递参数巧妙的实现。

这样的情况，我们会尽量把可提供回滚接口的服务放在前面。举个例子说明：

我们的某个论坛网站，每天登录成功后会奖励用户 5 个积分，但是积分和用户又是两套独立的子系统服务，对应不同的 DB，这控制起来就比较麻烦了。解决思路： 

把登录和加积分的服务调用放在 BFF 层一个本地方法中。
当用户请求登录接口时，先执行加积分操作，加分成功后再执行登录操作
如果登录成功，那当然最好了，积分也加成功了。如果登录失败，则调用加积分对应的回滚接口（执行减积分的操作）。
总结：这种方式缺点比较多，通常在复杂场景下是不推荐使用的，除非是非常简单的场景，非常容易提供回滚，而且依赖的服务也非常少的情况。 

这种实现方式会造成代码量庞大，耦合性高。而且非常有局限性，因为有很多的业务是无法很简单的实现回滚的，如果串行的服务很多，回滚的成本实在太高。 

## 3.2 本地消息表
这种实现方式的思路，其实是源于 ebay，后来通过支付宝等公司的布道，在业内广泛使用。其基本的设计思想是将远程分布式事务拆分成一系列的本地事务。如果不考虑性能及设计优雅，借助关系型数据库中的表即可实现。 

举个经典的跨行转账的例子来描述。 

第一步伪代码如下，扣款 1W，通过本地事务保证了凭证消息插入到消息表中。 


第二步，通知对方银行账户上加 1W 了。那问题来了，如何通知到对方呢？ 

通常采用两种方式：

采用时效性高的 MQ，由对方订阅消息并监听，有消息时自动触发事件
采用定时轮询扫描的方式，去检查消息表的数据。
两种方式其实各有利弊，仅仅依靠 MQ，可能会出现通知失败的问题。而过于频繁的定时轮询，效率也不是最佳的（90% 是无用功）。所以，我们一般会把两种方式结合起来使用。

解决了通知的问题，又有新的问题了。万一这消息有重复被消费，往用户帐号上多加了钱，那岂不是后果很严重？ 

仔细思考，其实我们可以消息消费方，也通过一个“消费状态表”来记录消费状态。在执行“加款”操作之前，检测下该消息（提供标识）是否已经消费过，消费完成后，通过本地事务控制来更新这个“消费状态表”。这样子就避免重复消费的问题。 

总结：上诉的方式是一种非常经典的实现，基本避免了分布式事务，实现了“最终一致性”。但是，关系型数据库的吞吐量和性能方面存在瓶颈，频繁的读写消息会给数据库造成压力。所以，在真正的高并发场景下，该方案也会有瓶颈和限制的。

# 4. CAP/BASE

CAP 理论是分布式系统的一个基础理论，它描述了任何一个分布式系统最多只能满足以下三个特性中的两个：

- 一致性（Consistency）
- 可用性（Availability）
- 分区容忍性（Partition tolerance）

Base:

- Basically Available: 基本可用，允许分区失败
- Soft state: 软状态，接受一段时间的状态不同步
- Eventually consistent：最终一致，保证最终数据的状态是一致的。

当我们再分布式系统中选择了CAP中的A和P后，对于C，我们采用的方式和策略就是保证最终一致。为了更好的扩展性和可用性，一般不会采用强一致性，而是采用最终一致性。