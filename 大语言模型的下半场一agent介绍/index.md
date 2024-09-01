# 大语言模型的下半场(一)——Agent介绍


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# 大语言模型的下半场——Agent介绍

## 什么是Agent？
我们都知道大模型很强大，能够像人类大脑一样思考和推理。但是大模型没有与世界互动的能力，就像人类只有大脑没有四肢，无法使用工具。如何让大模型能够使用工具，完成超越大模型能力边界的事情呢？答案是Agent，也叫做智能体。

OpenAI应用研究主管LilianWeng在[LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/)中总结：`Agent = Memory + Planning + Tools + Action`。

![](https://cdn.jsdelivr.net/gh/hts0000/images/202408251747908.png)

- 记忆(Memory)：记忆分为短期记忆和长期记忆，能够为大模型决策是提供上下文。
- 思考和决策(Planning)：根据用户输入结合上下文进行思考和决策。
- 工具(Tools)：决策如何使用工具，工具可以是任意的API调用结果。
- 行动(Action)：执行决策选择的工具进行行动。

一个简单的Agent就是在不断地执行`执行思考、调用工具、观测结果`，最终返回结果。

以`查询今天广州的天气`为例子，假设我们为大模型提供两种工具：
1. 查询当前时间：`get_current_time() -> time`，无参数输入，返回当前时间。
2. 查询某地天气：`get_location_weather(city, time) -> weather_describe`，输入城市与时间，返回该城市的天气预报。

那么Agent可能的思考过程如下：
1. 理解输入，知道是`期望查询今天广州的天气`。
2. 查看已有的工具，发现`get_location_weather`工具可以查询天气。
3. 观测到`get_location_weather`工具需要两个参数输入——city和time。
4. 观测输入，理解`city`参数为`广州`，`time`参数为`今天`，无具体日期。
5. 思考得知需要查询当前具体时间。
6. 查看已有的工具，发现`get_current_time`工具可以查询当前时间，且无需参数输入。
7. 调用`get_current_time`工具获取当前时间。
8. 将`city=广州`和`time=time`作为参数调用`get_location_weather`工具。
9. 观测`get_location_weather`工具结果，整理天气预报。
10. 思考已经得到最终答案。
11. 将最终答案返回给用户。

我们可以发现Agent的思考过程是一个循环调用的过程，这是一种称之为**ReAct模式的思考链**，其理论基础来源于一篇论文：[ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629)。

ReAct是Reason和Action的结合。ReAct思考链遵循如下模式：
1. 思考和推理问题。
2. 制定行动计划。
3. 执行行动。
4. 观测行动结果。
5. 循环直到任务完成。

![](https://cdn.jsdelivr.net/gh/hts0000/images/202408251824376.png)

## Multi-Agent
通过两个或更多不同角色的Agent，让这些Agent协调完成复杂的工作，这种方式称之为Multi-Agent。

通过角色的定义，还可以让Agent之间拥有对话、竞争以及上下级的概念。

一个注明的例子是由斯坦福大学和Google共同推出的项目——虚拟小镇。小镇由25个Agent组成，各自拥有不同的角色和性格，在虚拟小组中模拟人类世界的爱恨情仇。
![](https://cdn.jsdelivr.net/gh/hts0000/images/202408251850069.png)

## Agent前沿应用
### Auto-GPT
可视化的AI Agent平台，使用流程图的方式来构建Agent。

项目地址：https://github.com/Significant-Gravitas/AutoGPT

### BabyAGI
BabyAGI是一个脚本，是使用另一种称之为**Plan-and-Executor思考链**的方式实现的Agent。通过不断将任务拆解为更小的任务，最终为目标提供一系列可行的步骤。

项目地址：https://github.com/yoheinakajima/babyagi

### ChatDev
清华大学发布的应用，由多个Agent共同经营一家虚拟软件公司，每个Agent负责包括销售、产品、研发、测试、实施等不同角色。

项目地址：https://github.com/OpenBMB/ChatDev

### 虚拟小镇
斯坦福和Google共同推出的Multi-Agent项目，用于模拟和研究人类的交互行为。

论文地址：https://arxiv.org/pdf/2304.03442v1.pdf

项目地址：https://github.com/joonspk-research/generative_agents

## 大语言模型的开发框架
### LangChain
由OpenAI推出的大语言模型开发框架，涵盖了基础设施(LangChain-Core)、开发框架(LangChain/LangGraph)、部署平台(LangServe)、观测平台(LangSmith)、社区插件(LangChain-Community)等一系列工具和平台，帮助开发者更好的进行AI应用的开发。

官方文档：https://python.langchain.com/v0.2/docs/introduction/

### AutoGen
由微软推出的开源框架，允许开发人员通过多个Agent构建AI应用。

项目地址：https://github.com/microsoft/autogen

## 什么限制了Agent的发展？
### 有限的上下文
在不断地思考和工具调用中，Prompt的长度会急剧膨胀，知道超过可输入的上下文上限。

### LLM模型的理解能力
Agent会不断地思考、推理和决策，这非常依赖大模型本身的理解能力。如果大模型无法做出正确的决策，那么后续的工具调用都是空中楼阁。

就目前来看，至少需要70b及以上参数的模型，才能够输出比较质量的结果。

### LLM模型的可靠输出
大模型的输出是随机的，这是大模型的魅力，也是限制。如何获取相对可靠的输出，是调用工具的关键。

### 多模态的感知能力
如何将多模态的感知输入到Agent是更高阶段需要考虑的。我们当然不希望大模型只能理解文本，还希望他能够有更多模的感知能力。比如物理世界中：`视觉`、`听觉`、`触觉`、`嗅觉`和精神世界中的：`情绪`等等。

### 想象力
虽然说现在Agent的应用难点主要是技术无法匹配上人类的想象力，但是在将如何应用Agent上，人类的想象力还是相当匮乏。

## 参考
1. [动画科普AI Agent：大模型之后为何要卷它？](https://www.bilibili.com/video/BV1RC4y1A7ZU/?spm_id_from=333.337.search-card.all.click&vd_source=f597eabf6976e2aab2009735e72a6a40)
