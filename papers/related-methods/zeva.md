---
type: paper
title: "Zeva: In-Context Causal Learning for Generalizable Embodied Manipulation"
short_name: Zeva
authors: [Fu Chen, Xin Ding, Bingjia Huang, Xiangyu Li, Mingju Wang, Jiawei He, Kun Li, Wei Sun, Yunxin Liu, Hao Wu, Ting Cao]
year: 2026
venue: null
paper_url: "https://arxiv.org/abs/2608.30880v2"
code_url: null
project_url: "https://air-embodied-brain.github.io/Zeva/"
source_urls:
  - "https://arxiv.org/html/2608.30880v2"
  - "https://air-embodied-brain.github.io/Zeva/"
category: related-methods
series: robot-in-context-learning
tags: [robotics, in-context-learning, world-action-model, generalization, evaluation, long-horizon]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "将机器人自己的动作及其视觉效果编码为双时间尺度记忆，用上下文改变冻结策略的后续行为。"
---

# Zeva：从自身交互结果形成冻结策略的上下文记忆

> **证据边界：**依据2026-09-22的arXiv v2，第III–V节与Algorithm 1、Tables I–III；未复现。论文使用“causal”描述动作—效果交互表征，不能仅凭该命名就推断它解决了未观测混杂或严格因果识别。

## 一句话总结

Zeva 记住“我刚做了什么、世界实际怎样变了”，用这类证据引导下一次尝试；部署改的是记忆内容，基础策略和记忆模块的参数都不更新。

## 论文明确内容

### 交互表征、双时间尺度与读取

第III节将完整交互编码为视觉状态、执行动作chunk及观察到的状态变化。核心式(1)为：

$$
\xi_{\tau+1}=\phi(s_\tau,u_\tau,s_{\tau+1}-s_\tau).
$$

CTE利用GRU将这些输入投影为phase token和interaction signal，训练用效果预测、同任务对比学习、阶段进度约束。它把“当前是什么任务/阶段”和“此前动作产生何种效果”分别组织，而不是保存整段视频文本。

- **BIT：**短期交互trace，保留当前attempt最近的完成交互；重试时清空。
- **PIM：**按phase/effect相似度合并跨attempt经验；同一episode内保留，进入新episode前清空。
- **注入：**以当前phase检索历史effect，经memory encoder生成prompt tokens，条件化扩散动作路径。

训练阶段会学习CTE及条件flow-matching策略；“冻结”仅指部署。观察到动作效果后才写记忆，当前动作不能偷看尚未发生的后果。

### 评估单位决定结论

Episode是固定场景/物体/机器人初始配置，attempt是一次执行。重复尝试会恢复相同初态，只保留PIM；各方法使用相同最大重试预算。不能把累计多次尝试的CSR，直接与一次rollout的成功率比较。

式CSR@K衡量K次内至少成功一次的episode比例：

$$
\operatorname{CSR@K}=\frac1N\sum_i\mathbb I[\max_{1\le k\le K} y_{i,k}=1].
$$

模拟为RoboCasa365-Atomic5五个任务，每任务50个随机episode、重复两次取均值，使用Cosmos3-Nano；真机ChemLab-Evo使用ARX双臂与Cosmos3-Edge，7个任务、每任务20个随机episode。二者基础模型和硬件不同。

### 结果与消融

| 来源 | 作者报告 | 边界 |
|---|---|---|
| Table II Atomic5 | 平均76.8%，Fast-WAM72.4% | +4.4个百分点；不是完整365任务覆盖 |
| Table I ChemLab-Evo | 原子83.3%、短序列70.0%、复杂7.5% | 复杂任务仍很弱，不能只报最容易任务的100% |
| Table I 长任务process score | 平均57.32 | 部分阶段进度，不能写成成功率57.32% |
| Table III | 去BIT或PIM均下降 | 支持短期/跨尝试记忆互补；每格20次 |
| Fig.3 scaling子集 | 四个演化milestone累计26%→45%→70%→73% | 是另选固定episode子集；milestone不是连续attempt编号 |

跨任务检索实验在两项任务上，匹配phase的其他任务经验优于随机替换。它支持所学表征有可迁移结构，但不证明任意任务间迁移或部署过程中永久学会了新技能。

### 作者局限

第V节指出当前经验来自完成任务时的执行尝试，尚未主动选择最有信息的探索动作。正文未将PIM默认保留到新episode，因此“持续自进化”不能扩写成跨所有未来场景永久累积。

## 我的理解

与示范prompt不同，Zeva的上下文主要由机器人自采样并观察效果生成，包含失败也可能有价值。效果标签与当前phase的匹配，决定记忆是帮助恢复还是检索了不合适动作。冻结权重降低灾难性遗忘风险，却不能阻止上下文污染。

## 推测与问题

- 固定初态反复尝试的改善，有多少来自记忆、有多少只是更多执行机会？应报告无记忆同预算CSR对照。
- phase相似性可能来自任务进度而非共享动力学；跨task有效需要拆开检验。
- 接触力、遮挡、物体被移走时，视觉latent差分是否仍能可靠代表动作效果？

## 阅读可信度检查

- [x] 区分episode/attempt、SR/process/CSR及memory重置规则。
- [x] 关键公式和数值核对原文，不将“causal”当作因果识别证明。
- [x] 作者报告与个人问题分开；未复现。

## 关联笔记

- [机器人 ICL 系列](../../series/robot-in-context-learning.md)
- [RoboTTT](robottt.md)
- [HOST](host.md)
