---
layout: post
title: 难以遏制的人因差错
---

# Go的日志工具之痛
go生态一直没能有一个强势的日志工具，优秀的日志有zap，zerolog，但是API各有各的设计理念，自然是不兼容的

# openlog和seclog的初衷

倒退5年，go的日志工具更加不成熟，我们folk lager项目，进行安全整改后成为自己的项目https://github.com/go-chassis/seclog，
然而他的老旧的API设计已经被我诟病很久。就是以f为结尾的函数调用

后来，为了应对以后的需求变化（合规检查），我新建了一个仓库https://github.com/go-chassis/openlog，
是一个适配层对下面的日志工具做适配，保证所有工程都只调用此库，不对具体的日志工具实现产生耦合，这样seclog可以适配，调用方就与seclog解耦了。


然而“f类函数”的问题还没解决，一直期望重构一次，日志工具存在于工程的各个角落，花费了大量时间精力来重构，而最终还会导致用户使用上的不兼容和整改，是否值得？

# 问题
先来看看原始issue
https://github.com/go-chassis/go-chassis/issues/889

来看看我们写一个日志用f类API, 省去了fmt.Sprintf大大简化啰嗦的语句，开发人员喜欢
```go
openlogging.GetLogger().Debugf("shuffler %d %d", i, v)
```

然而如果你这么去写，也是完全不会有问题的，可想而知，我们会丢失重要的日志信息，甚至无法定位生产环境问题
```go
openlogging.GetLogger().Debugf("shuffler", i, v)
```

是的，最大的问题在于事后纠正，而不是事前预防，你可以任意的滥用format参数和后面的参数，而无论IDE，静态检查对此都无能为力，代码合入后，只能在运行时肉眼排查。

我们再来看一个整改后的原形毕露……，不仔细看还真的看不出来“%s:%s%:s”这个事有问题的写法，应当是“%s:%s:%s”
```go
openlog.Error(fmt.Sprintf("can not close client %s:%s%:s, err [%s]", protocol, service, endpoint, err.Error()))
```
把它放到IDE里，可以看到清晰地告警提示，然而你调用f类函数不会，关键日志信息就这么的丢掉了


可以以下面的提交为例，我在重构过程中发现很多f类方法被滥用
https://github.com/go-chassis/go-archaius/pull/121

由于日益增多的代码，很难只通过人眼code review来杜绝错误写法。
本次的重构将解决这类问题，提升代码的质量


