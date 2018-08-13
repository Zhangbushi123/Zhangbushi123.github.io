---
layout:         post
title:          floable 6.2.1 无法获取 Http Task Response
subtitle:       遇见可遇不可求的美好
card-image:     https://ww1.sinaimg.cn/mw690/906cb9dbgw1fayoxw9jh0j20b407e3zn.jpg
date:           2018-04-03 19:00:00
tags:           life
post-card-type: image
---

官方文档中对于 Http Task Response 相关变量的获取介绍 [https://www.flowable.org/docs/userguide/index.html#bpmnHttpTaskConfiguration](https://www.flowable.org/docs/userguide/index.html#bpmnHttpTaskConfiguration) 。文档中说，执行变量名都以 resultVariablePrefix 的计算值作为前缀。例如响应状态码可以在另一个活动中被访问，如 task7.responseStatusCode。这里task7是服务任务的id。要覆盖此行为，根据需要设置 resultVariablePrefix。

使用设计器设计流程，以及相关配置如下：

![986c53b2bd07eb3e19c372378d33f2613ea0542a.png](https://i.loli.net/2018/05/25/5b082ac1cba25.png)

## 错误

流程在执行时，抛出异常错误如下：

```
org.flowable.engine.common.api.FlowableException: Unknown property used in expression: ${hrTask.responseStatusCode == 200}
at org.flowable.engine.common.impl.el.JuelExpression.getValue(JuelExpression.java:50)
at org.flowable.engine.impl.el.UelExpressionCondition.evaluate(UelExpressionCondition.java:37)
at org.flowable.engine.impl.util.condition.ConditionUtil.hasTrueCondition(ConditionUtil.java:47)
...
Caused by: org.flowable.engine.common.impl.javax.el.PropertyNotFoundException: Cannot resolve identifier 'hrTask'
at org.flowable.engine.common.impl.de.odysseus.el.tree.impl.ast.AstIdentifier.eval(AstIdentifier.java:97)
at org.flowable.engine.common.impl.de.odysseus.el.tree.impl.ast.AstProperty.eval(AstProperty.java:68)
at org.flowable.engine.common.impl.de.odysseus.el.tree.impl.ast.AstBinary$SimpleOperator.eval(AstBinary.java:31)
...
```

## 问题分析

程序抛出异常错误，不能识别表达式和 hrTask 变量，代码应该是将 hrTask 作为对象，然后获取 responseStatusCode 属性，然后进行判断表达式的。但是数据存放 http 相关变量是以 前缀.变量 存储的，例如 hrTask.responseStatusCode 存为一个字符串作为变量，导致不能正确获取到流程变量。

## 解决方法

修改 Http Task 的源代码，将 ```execution.setVariable(request.getPrefix() + ".responseProtocol", response.getProtocol())``` 修改为 ```execution.setVariable(request.getPrefix() + "ResponseProtocol", response.getProtocol())``` 这种方式，前缀和变量中间以骆驼峰的方式进行存储，这样 DB 就可以存放一个正常的字符串变量，同时，表达式也可以使用 ```${hrTaskResponseStatusCode == 200}``` 这种方式进行处理。

刚看了一下 flowable 在 github 上 master 的相关代码，这个问题最近已经被修复了，但是还没有出正式版本， github 修改地址：[https://github.com/flowable/flowable-engine/commit/d7554ebb572caa7e66933852362610041baefecc#diff-d66c51ccebf7d3aff06bc6143c8f7bf8](https://github.com/flowable/flowable-engine/commit/d7554ebb572caa7e66933852362610041baefecc#diff-d66c51ccebf7d3aff06bc6143c8f7bf8)。目前项目中使用 6.2.1，仍然自己修改源代码。