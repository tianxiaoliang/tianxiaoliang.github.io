---
layout: post
title: 难以遏制的人因差错
---

# Go的日志工具之痛
go生态一直没能有一个强势的日志工具，优秀的日志有zap，zerolog，但是API各有各的设计理念，自然是不兼容的

# openlog和seclog的初衷
https://github.com/go-chassis/openlog 

我期望封装一个统一接口来应对不同组织的合规检查，每个团队的日志工具要求都不同，只有通过interface牺牲性能来弥补。我们folk开源日志工具lager，实现了自己的seclog，对接到统一的openlog interface中，做了适配。

倒退5年，go的日志工具更加不成熟，我们folk lager项目，进行安全整改后成为自己的项目https://github.com/go-chassis/seclog，
然而他的老旧的API设计已经被我诟病很久。

一直期望重构一次，我记录了一个issue，并提交了新的一系列PR，花费了大量时间精力来纠正这个问题，而最终还会导致用户使用上的不兼容和整改，为何？

# 问题
先来看看原始issue
https://github.com/go-chassis/go-chassis/issues/889

来看看我们写一个日志用f类API, 省去了fmt.Sprintf大大简化啰嗦的语句，开发人员喜欢
```go
openlogging.Debugf("shuffler %d %d", i, v)
```

然而如果你这么去写，也是完全不会有问题的，可想而知，我们会丢失重要的日志信息，甚至无法定位生产环境问题
```go
openlogging.Debugf("shuffler", i, v)
```

是的，最大的问题在于事后纠正，而不是事前预防，你可以任意的滥用format参数和后面的参数，而无论IDE，静态检查对此都无能为力，代码合入后，只能在运行时肉眼排查。

可以以下面的提交为例，我在重构过程中发现很多f类方法被滥用
https://github.com/go-chassis/go-archaius/pull/121

由于日益增多的代码，很难只通过人眼code review来杜绝错误写法。
本次的重构将解决这类问题


