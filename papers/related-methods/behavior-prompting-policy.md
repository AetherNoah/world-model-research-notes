---
type: paper
title: "What Enables In-Context Behavior Prompting for Manipulation?"
short_name: BPP
authors: [Austin Patel, Ben Pekarek, Joel Enrique Castro Hernandez, Shuran Song]
year: 2026
venue: null
paper_url: "https://arxiv.org/abs/2606.30457v2"
code_url: "https://github.com/real-stanford/behavior_prompting"
project_url: "https://behavior-prompting.github.io/"
source_urls:
  - "https://arxiv.org/html/2606.30457v2"
  - "https://behavior-prompting.github.io/"
category: related-methods
series: robot-in-context-learning
tags: [robotics, in-context-learning, diffusion, generalization, evaluation]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "以含观察、状态和动作的示范作prompt，用分块注意力与扩散动作头研究任务多样性如何形成推理时行为条件化。"
---

# BPP：什么条件让示范真正成为机器人行为的 prompt

> **版本说明：**用户给出的题名《Behavior Prompting Policy: Demonstrations as Prompts for Manipulation》对应同一 arXiv 条目的早期版本。2026-10-05 的 v2 标题改为《What Enables In-Context Behavior Prompting for Manipulation?》，方法名仍为 Behavior Prompting Policy（BPP）。本笔记依据 v2 第 3–5 节与附录 C/F/G/I；未复现。

## 一句话总结

BPP 不在测试时训练新技能，而是把一次示范当作闭环策略的条件；关键实证问题是数据需要多大任务多样性，以及模型究竟读取了示范中的什么。

## 论文明确内容

### Prompt 并非普通 RGB 视频

行为 prompt 含机器人执行空间中的观察 $o$、本体状态 $q$ 和动作 $a$。iPhUMI 是带 iPhone 的手持夹爪接口，可由人演示，但会提供传感运动记录；不能把它概括为“任意互联网视频即可教机器人”。

第 3 节按时间 chunk 下采样观察/状态（通常 1Hz），保留每个 chunk 内完整动作序列，经 attention pooling 得到 $P=[p_0,\ldots,p_n]$。当前观察通过 cross-attention 在示范中检索相关部分，然后由带 FiLM 的 CNN diffusion action decoder 生成动作 chunk。测试时示范只编码一次，权重冻结；当前短历史持续更新，整个 rollout 不必无限保存在上下文。

### 训练与目标

每次从某任务抽一条轨迹作 prompt，再从**同任务另一条、不同环境配置**轨迹抽监督动作。没有显式逐帧对应标注。训练沿用条件动作 diffusion 的去噪监督；prompt 是额外条件而非待模仿的输出。本文方法主体未给出一个应被当成新损失的独立数学公式，本笔记不另造公式。

### 评估协议

- **DrawAnything-Sim：**2,000 个程序生成图形，每任务5条示范；测试50个人类产生的新图形，执行时画板方向与示范不同。
- **DrawAnything-Real：**ARX 机械臂与腕部 iPhone，1,000个训练任务；测试10个图形，其中4个训练内、6个未见。这里的6DoF、遮挡和模拟2D设定不能混比。
- **LIBERO-Gen：**Combination 留出未见的抓取/放置组合；Chain 留出已见单步技能的新组合。因此“未见任务”不等于从零获得全新动作原语。
- 模拟结果通常报告3个随机种子，$\pi_{0.5}$ 为单种子；真机图形为跨任务统计。

### 结果与消融

| v2 来源 | 数字/发现 | 含义 |
|---|---|---|
| Fig. 4 DrawAnything-Sim | BPP误差3.43±0.10；ICRT5.07±0.22 | 是绘图误差，不是成功率 |
| 第4.1节动作头对照 | BPP换成L1头后误差4.84±0.22 | 扩散头解释了大部分相对ICRT增益，不能全归功prompt encoder |
| Appendix C 标准LIBERO | 平均97.48±0.33 | 是额外ID检查，不是LIBERO-Gen新任务泛化分数 |
| Appendix F prompt暂停 | 插入三次各10步暂停，误差升至4.26±0.12 | 示范质量仍会影响性能 |
| Fig. 5 固定数据预算 | 增加任务数优于只增加每任务示范数 | 具体阈值依赖任务空间，不能把绘图约1000任务的趋势推广到所有操控 |

注意力分析显示当前状态会定位示范中的相邻子目标；画布被擦除后，注意力会回退到匹配阶段。它支持“密集子目标查询”的解释，但注意力可视化本身不构成唯一机制的证明。

### 作者局限

第5节指出 prompt 与 rollout 虽改变位置/方向，但未系统改变场景与物体身份；任务多样性门槛因领域而异；LIBERO-Gen 主要重组已见技能。仅三种折衣任务时，语言条件可能更好，BPP有时熟练地执行了错误折法。

## 我的理解

BPP 的数据设计阻止模型只看当前画面就猜任务。对世界模型研究而言，最值得借鉴的是“同一状态允许多种任务、示范必须消除歧义”的训练构造，而非简单把更长视频拼进网络。

## 推测与问题

- 若示范含错误或跳步，模型是否会把不正确策略也当作密集目标？
- 任务标签和 prompt/rollout 采样必须避免同一轨迹泄漏；跨物体身份/接触模式需独立评测。
- 视频、动作、语言的收益应在同等预训练与计算预算下比较。

## 阅读可信度检查

- [x] 核实题名版本变化和BPP/ICRT架构区别。
- [x] 数字对应具体图表，不将绘图误差写成成功率。
- [x] 人类演示接口、任务重组和训练边界已明确；未复现。

## 关联笔记

- [机器人 ICL 系列](../../series/robot-in-context-learning.md)
- [ICRT](icrt.md)
- [StellaVLA](stellavla.md)
