---
title: "高级编排介绍"
description: "快速了解 FastGPT 高级编排"
icon: "circle"
draft: false
toc: true
weight: 301
---

FastGPT 从 V4 版本开始采用新的交互方式来构建 AI 应用。使用了 Flow 节点编排（工作流）的方式来实现复杂工作流，提高可玩性和扩展性。但同时也提高了上手的门槛，有一定开发背景的用户使用起来会比较容易。

[查看视频教程](https://www.bilibili.com/video/BV1aB4y1Z7Hy/?spm_id_from=333.999.list.card_archive.click&vd_source=903c2b09b7412037c2eddc6a8fb9828b)

![](/imgs/flow-intro1.png)

## 什么是节点？

在程序中，节点可以理解为一个个 Function 或者接口。可以理解为它就是一个**步骤**。将多个节点一个个拼接起来，即可一步步的去实现最终的 AI 输出。

如下图，这是一个最简单的 AI 对话。它由用户输入的问题、聊天记录以及 AI 对话节点组成。

![](/imgs/flow-intro2.png)

执行流程如下：

1. 用户输入问题后，会向服务器发送一个请求，并携带问题。从而得到【用户问题】节点的输出。
2. 根据设置的【最长记录数】来获取数据库中的记录数，从而得到【聊天记录】节点的输出。
   经过上面两个流程，就得到了左侧两个蓝色点的结果。结果会被注入到右侧的【AI】对话节点。
3. 【AI 对话】节点根据传入的聊天记录和用户问题，调用对话接口，从而实现回答。（这里的对话结果输出隐藏了起来，默认只要触发了对话节点，就会往客户端输出内容）

### 节点分类

从功能上，节点可以分为 2 类：

1. **系统节点**：用户引导（配置一些对话框信息）、用户问题（流程入口）。
2. **功能节点**：知识库搜索、AI 对话等剩余节点。（这些节点都有输入和输出，可以自由组合）。

### 节点的组成

每个节点会包含 3 个核心部分：固定参数、外部输入（左边有个圆圈）和输出（右边有个圆圈）。

   ![](/imgs/flow-intro3.png)
   
   - 对话模型、温度、回复上限、系统提示词和限定词为固定参数，同时系统提示词和限定词也可以作为外部输入，意味着如果你有输入流向了系统提示词，那么原本填写的内容就会被**覆盖**。
   - 触发器、引用内容、聊天记录和用户问题则为外部输入，需要从其他节点的输出流入。
   - 回复结束则为该节点的输出。

## 重点 - 工作流是如何运行的

与单出入口的工作流不同，FastGPT的工作流可以指定**不同的入口**，并且没有**固定的出口**，而是以节点运行结束作为出口，如果在一个轮调用中，所有节点都不再允许，则工作流结束。

不过为了方便阅读，大部分时候，我们仍是设置一个模块作为入口，在工作流中，它被叫做`对话入口`。下面我们来看下，工作流是如何运行的，以及每个节点何时被触发执行。

记住3个**节点可执行**的原则：

1. 仅关心**已连接的**外部输入，即左边的圆圈被连接了参数。
2. 当**已连接的**内容都被赋值的时候触发。（这个地方经常会遇到，连接了很多根输入线，但是只要有一个输入没有值，这个节点也不会执行）
3. 可以多个输出连接到一个输入，后续的值会覆盖前面的值。

![](/imgs/workflow_process.png)

### 示例 1：

聊天记录节点会自动执行，因此聊天记录输入会自动赋值。当用户发送问题时，【用户问题】节点会输出值，此时【AI 对话】节点的用户问题输入也会被赋值。两个连接的输入都被赋值后，会执行 【AI 对话】节点。

![](/imgs/flow-intro1.png)

### 例子 2：

下图是一个知识库搜索例子。

1. 历史记录会流入【AI 对话】节点。
2. 用户的问题会流入【知识库搜索】和【AI 对话】节点，由于【AI 对话】节点的触发器和引用内容还是空，此时不会执行。
3. 【知识库搜索】节点仅一个外部输入，并且被赋值，开始执行。
4. 【知识库搜索】结果为空时，“搜索结果不为空”的值为空，不会输出，因此【AI 对话】节点会因为触发器没有赋值而无法执行。而“搜索结果为空”会有输出，流向指定回复的触发器，因此【指定回复】节点进行输出。
5. 【知识库搜索】结果不为空时，“搜索结果不为空”和“引用内容”都有输出，会流向【AI 对话】，此时【AI 对话】的 4 个外部输入都被赋值，开始执行。

![](/imgs/flow-intro4.png)

## 如何连接节点

1. 为了方便识别不同输入输出的类型，FastGPT 给每个节点的输入输出连接点赋予不同的颜色，你可以把相同颜色的连接点连接起来。其中，灰色代表任意类型，可以随意连接。
2. 位于左侧的连接点为输入，右侧的为输出，连接只能将一个输入和输出连接起来，不能连接“输入和输入”或者“输出和输出”。
3. 可以点击连接线中间的 x 来删除连接线。
4. 可以左键点击选中连接线

## 如何阅读？

1. 建议从左往右阅读。
2. 从 **用户问题** 节点开始。用户问题节点，代表的是用户发送了一段文本，触发任务开始。
3. 关注【AI 对话】和【指定回复】节点，这两个节点是输出答案的地方。

## FAQ

### 想合并多个输出结果怎么实现？

1. 文本加工，可以对字符串进行合并。
2. 知识库搜索合并，可以合并多个知识库搜索结果
3. 其他结果，无法直接合并，可以考虑传入到`HTTP`节点中进行合并，使用`[Laf](https://laf.run/)`可以快速实现一个无服务器HTTP接口。

### 节点为什么有2个用户问题

左侧的`用户问题`是指该节点所需的输入。右侧的`用户问题`是为了方便后续的连线，输出的值和传入的用户问题一样。
