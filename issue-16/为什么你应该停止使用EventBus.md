为什么你应该停止使用EventBus
---

> * 原文链接 : [Why you should avoid using an event bus](http://endlesswhileloop.com/blog/2015/06/11/stop-using-event-buses/)
* 原文作者 : [Tony Cosentini](http://endlesswhileloop.com/)
* 译文出自 :  [开发技术前线 www.devtf.cn](http://www.devtf.cn)
* 译者 : [Zhaoyy](https://github.com/Zhaoyy) 


``文中的EventBus多指事件总线这种设计模式，而非EventBus这个具体的类库。``

我经常看到EventBus被作为一种通用模式应用在Android开发中。Otto和EventBus这样的类库经常被用来省去编写不同层之间来封装代码的模板类。尽管EventBus刚开始看起来的确带来了方便，但是很快这些纠缠的事件会被弄成一堆乱麻，很难跟踪更别说调试。

EventBus通常被宣扬可以降低模块之间的耦合度，但是，事实上带给你的是降低耦合度带来的混乱和困惑。

## 嵌套的事件

一个常见的弊端是处理嵌套的事件。不要在事件订阅者里面发布事件，这看起来很容易避免，但是经常会不容易理解。一个订阅者很可能会通过调用其他的方法来间接地抛出其他的事件。这样事件会纠缠成一个复杂地难以置信的球，很难进行调试。

Facebook的Flux框架是另外一个事件驱动的设计框架，但是它[明令禁止发送嵌套事件](https://github.com/facebook/flux/blob/ac1e4970c2a85d5030b65696461c271ba981a2a6/src/Dispatcher.js#L184)。希望Otto和EventBus未来能够检测嵌套事件。

## 以同步的方式处理生产者

（我不认为这跟GreenRobot的EventBus有关系。）

这是另外一个在大代码集中很难处理的常见模式。经常在许多Activity和Fragment里面，我们假定事件可以立即被EventBus里面的订阅者进行处理。它们会设置一个基于事件的属性，以假定这个属性不会为空的方式在其他的生命周期函数中使用。

当你这样做的时候，你对于事件的发送传递作了一些大胆的假设，这些所有的假设并不能都被证明一定是这样或者是明确地被API备注，这些假设都是不可靠的。

当你重构改变这些处理事件的代码的时候会发生什么呢？很可能重构处理事件代码的那个人进行实现的时候并不知道你的这个假设。

问题的根本在于生产者是否处理好一下两个方面：

- 要求把数据作为依赖。
- 处理好数据未获取到时情况。

在事件处理中组件可以把事件数据作为依赖，仅仅是作为是否可用的依赖。这需要一个随时可用的依赖构造函数或者是通过依赖注入注入数据（这个可以继续深入讨论）。

如果还没有请求到任何数据，组件需要处理好这种还未得到信息的情况（对于UI组件通常是处理等待加载的情况）。

## 其他选择

没有类库或者工具会毫无代价的处理好这些问题，但是一些工具或者设计模式会鼓励你使用正确的处理方式来处理这些问题。

正确地使用EventBus可能会避免这些问题。然而，相比大多数其他工具更鼓励实用的做法。尽管使用简单的监听类会增加一些额外的代码，可能会让业务的处理更明确。

对于更加复杂的情况，[RxJava](http://reactivex.io/)提供了更好的解决办法。[Dan Lew有一些非常好的博客来介绍这个框架](http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/)。

**2015-06-14更新**：为了减少偏激我更改了标题（原来的标题是“EventBus 一个反面的设计模式”）。需要重申的是并不是所有的易错问题都是由EventBus导致的。特别是同步获取的问题在任何方式中都有可能发生，但是我认为通过RxJava结合一些配置和可空的注解（``@Nullable``）能够更清晰地处理好这个问题。


