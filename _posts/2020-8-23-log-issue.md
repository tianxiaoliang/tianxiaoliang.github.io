---
layout: post
title: 难以遏制的人因差错
---

# go的日志工具
go生态一直没能有一个强势的日志工具，看上去不错的zap，zerolog，但是API上各有各的设计理念

# openlog
https://github.com/go-chassis/openlog 我期望封装以日志工具来应对不同组织的合规检查，每个团队的日志工具诉求都不同，只有通过interface牺牲日志性能来弥补。

倒退5年，go的日志工具更加不成熟，公司folk的cloudfoundry的lager日志工具，进行安全整改后落地自己的项目https://github.com/go-chassis/seclog，
然而他的老旧的API设计已经被我诟病很久，而我不再期望这样下去。

我记录了一个新的issue，并提交了新的一系列PR，花费了大量时间精力来纠正这个问题，而最终还会导致用户使用上的不兼容和整改，为何？

# 问题

先来看看原始issue
https://github.com/go-chassis/go-chassis/issues/889

来看看我们写一个日志用f类API, 挺好省去了fmt.Sprintf大大简化啰嗦的语句，开发人员喜欢
```go
openlogging.GetLogger().Debugf("shuffler %d %d", i, v)
```
然而如果你这么去写，也是完全不会有问题的，可想而知，我们会丢失重要的日志信息，甚至无法定位现网问题
```go
openlogging.GetLogger().Debugf("shuffler", i, v)
```

是的，最大的问题在于事后纠正，而不是事前预防，你可以任意的滥用format参数和后面的参数，而无论IDE，静态检查对此都无能为力。

可以以下面的提交为例，我在整改的过程中发现了大量滥用f类方法的例子
https://github.com/go-chassis/go-archaius/pull/121

由于日益增多的代码，很难只只通过人眼code review来杜绝错误的代码。我们需要彻底对这种方法进行整改，将问题从此杜绝。让每一个运行期的错误日志，或者详细信息都正常的打印出来


希望大家理解这次不兼容的变更
