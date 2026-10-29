---
type: paper
document_kind: viewpoint
title: "Robot-Use Agents"
short_name: Robot-Use Agents
authors: [Phillip Isola]
year: 2026
venue: Personal essay
paper_url: "https://web.mit.edu/phillipi/www/writing/robot-use-agents.html"
code_url: null
project_url: null
source_urls:
  - "https://web.mit.edu/phillipi/www/writing/robot-use-agents.html"
category: related-methods
series: agentic-policy
tags: [robotics, embodied-ai, agentic-policy]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "从工具使用视角讨论通用云端agent控制机器人的可能传播路径，同时承认时延、可靠性和物理风险尚未解决。"
---

# Robot-Use Agents：把机器人视为通用 agent 可以使用的工具

> **证据边界：**这是Phillip Isola于2026-09-07发表的**观点文章**，不是新算法论文。通读原文及其脚注链接后整理。它没有自己的训练目标、数据集、定量实验或正式算法；本库沿用paper索引模板只是为方便检索，不据此声称它经过会议审稿。

## 一句话总结

如果通用agent能像使用计算器、浏览器和计算机一样调用机器人，那么机器人能力的传播速度可能主要由云端模型与接口决定，而不必每个机体都重新定制完整智能栈。

## 原文明确信息

### 核心论点

作者将robot-use与computer-use类比：robot body是agent可调用的物理工具，通用模型可以承担以往通常划给高层规划之外的更多决策。过去反对通用LLM直接控制的理由包括时延、空间能力不足、因果与物理理解不足；新模型演示使这个方向值得认真检验。

文章对比两种扩散路径：传统方案通常依赖针对机体和任务的本地模型、工程与硬件部署；云端agent借助成熟LLM基础设施，有可能让已有联网设备通过模型或接口更新获得新能力。这里的重点是**智能如何扩散到设备群体**，不只是某台机器人某任务多成功几次。

### 作者同时保留的边界

作者明确承认通用agent的命令间隔较长，既受模型思考也受网络影响；“puppeteering”可能不如成熟机器人系统可靠，专用方案目前仍更高效。这些限制在高风险的物理环境尤为重要。文章提出潜力与风险，并未用实验消除它们。

### 没有的方法与实验

原文没有定义统一action representation、feedback cadence或安全接口，没有报告SR、latency benchmark或训练数据，也没有证明任意联网设备可直接部署。脚注引用SayCan、Code as Policies等先行工作和社区演示，不能把那些实验数字当作这篇文章的新结果。

## 我的理解

这篇适合放在系列开头作为问题框架：从“机器人上运行哪个policy”转向“通用agent如何调用、观测和验证一个物理系统”。但计算机工具可以暂停、撤销或快速重试，物理接触往往不可逆。robot-use需要明确执行权限、动作契约、反馈延迟和停止条件，不能仅凭类比完成部署论证。

真正可验证的主张应拆成：跨机体适配成本是否降低；同一云模型更新能否迁移到多台已有设备；包含网络等待的walltime和恢复成本是否可接受；模型生成的动作是否能被独立验证。

## 推测与问题

“云端更新使能力快速传播”是值得研究的预测，不是已建立的普遍事实。可能的检验是固定硬件与低层控制器，只更换通用agent，比较适配工程量、任务覆盖、失效种类和全链路时延。物理安全还需单独工程验证，不能由语言推理或模拟SR替代。

## 阅读可信度检查

- [x] 文体、作者与日期已核对。
- [x] 未给观点文虚构公式、训练数据、benchmark或接收状态。
- [x] 将传播假说与当前能力证据分开。
- [x] 无新增图片，引用均可回原文。

## 关联笔记

- [Agentic Policy 系列](../../series/agentic-policy.md)
- [Astranomy：完整控制接口和成本评估](astranomy.md)
- [Astra跨领域评估：推理与运动责任](astra-embodied-policies.md)
- [RoboBRIDGE：部署orchestration](robobridge.md)
