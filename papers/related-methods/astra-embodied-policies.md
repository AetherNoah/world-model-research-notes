---
type: paper
title: "Systematically Exploring the Capabilities of GPT-6 Astra as Embodied Policies"
short_name: Astra as Embodied Policies
authors: [Galbot Team]
year: 2026
venue: null
paper_url: "https://arxiv.org/abs/2609.38537"
code_url: "https://github.com/anonymous-report-421/GPT-as-Policy"
project_url: "https://galaxygeneralrobotics.github.io/astra-policy/"
source_urls:
  - "https://arxiv.org/html/2609.38537v1"
  - "https://anonymous-report-421.github.io/public-website/?view=1"
category: related-methods
series: agentic-policy
tags: [robotics, embodied-ai, agentic-policy, evaluation, long-horizon, generalization]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "跨六类具身任务分解Astra的直接控制、策略协作和反馈适应能力，强调完整控制路径、试验预算和推理延迟决定结论边界。"
---

# Astra as Embodied Policies：把推理决策与真实运动贡献分开评价

> **证据边界：**依据 arXiv:2609.38537v1（2026-09-29）的全文、Table 1–13 与附录，以及用户给出的匿名交互报告。未运行代码。作者团队完整名单见论文；这里采用署名 Galbot Team。网页与论文是相关版本，不能计作两个独立复现实验。

## 一句话总结

通用模型可以在部分任务中选择有效目标、准备接触条件、纠正预训练策略，但可靠运动往往由 IK、专用策略或 whole-body controller 提供；数值动作可生成并不意味着机器人能实时、稳定地执行。

## 论文明确内容

### 控制责任分解

论文评估 gripper manipulation、dexterous manipulation、mobile manipulation、navigation、locomotion、humanoid loco-manipulation 六个域。**Direct** 使用 Astra 生成命令配合解析低层执行器；**Hybrid** 额外使用学习型任务策略或全身控制器。导航“向前/转向/停止”和灵巧手逐关节命令的难度不可用一个成功率概括。

双臂 manipulation 的 Hybrid 先获取 $\pi_{0.5}$ 候选关节轨迹，将其通过正向运动学转成可供审核的末端轨迹，再让 Astra 接受片段或输出 EEF 修正。早期交互报告明确候选为 50 步，接受前 1–15 步或修正 1–5 步；两个分支**择一执行**，不是同时叠加控制。不同域还涉及不同反馈守卫与上下文管理，不能把 Hybrid/Direct 的差异全部归因于额外策略本身。

固定模型权重下，反馈修正可以发生在 episode 内；有些研究保留跨 trial 的文字 notes，有些 development 过程还由研究人员更改 prompt、interface 或 tracker。论文明确区分这些机制，它们都不应被默认为同一种 autonomous learning。本文没有统一的新损失函数。

### 协议和结果：每个域各自看

| 实验 | 协议 | 核对结果 |
|---|---|---|
| RoboDojo | 10 task×5 配对实例/方法 | Hybrid 24/50=48%；Direct 13/50=26% |
| RoboLab | 10 task×5，零样本 | Direct 49/50=98%；Hybrid 46/50=92% |
| RoboCasa | 15 task×5、相同初态与20步反馈节奏 | Hybrid 29/75=38.7%；Direct 25/75=33.3%；独立策略17/75=22.7% |
| Navigation | 四个固定子集，各50 episodes/系统 | RxR SR92%，HM3D ObjectNav SR82% |
| Locomotion | 一个course上五次连续适应尝试 | 五次均未到达目标，不能当作五个独立随机实验 |
| HumanoidBench | 30 tasks，两个seed；使用预训练全身controller | 13/30任务超过所列published baselines，并非13/30成功率 |

RoboDojo Table 2 的 Direct Score 37.81 来自 48 个有效 Score 条目，而 SR 使用全部 50 实例；不能由 Score 行反推出成功数。Hybrid 执行步中 85.6% 来自 $\pi_{0.5}$，仅 14.4% 为 Astra 修正；这是动作来源比例，不是贡献归因。RoboCasa 中 Hybrid 也会破坏独立策略本来能完成的任务，平均提升不代表逐任务占优。

导航成功率同样需要看路径效率：例如 HM3D SR82% 对应 SPL43.69%，存在明显绕行。灵巧手直接 in-hand control 难以重构接触，抓稳不等于能持续转动物体。结合 pretrained whole-body controller 后的 humanoid 结果更不能被描述为 Astra 独立输出全身动力学控制。

### 资源和实时性

Table 13 记录 RoboDojo 50 实例条件下 Hybrid 624.8M tokens、Direct 1.132B tokens。RoboCasa 中引入策略虽然减少 Astra 自己产生的动作，却可能增加审核请求。locomotion 最后一轮 30 秒模拟运动对应 250 次同步模型调用，平均39.86秒/调用，**推理时 physics 暂停**。这显示协作未必消除推理开销，也不能把演示的模拟时间当作真实 walltime。

### 匿名网页与 arXiv 版本关系

用户第三个链接标题为 **GPT 6 Astra as an Embodied Policy**，署名 Jiayi Su、Yixin Zheng、Mi Yan、Li Yi、Zhizheng Zhang、He Wang。其内容聚焦双臂 RoboDojo 与单臂 RoboLab，arXiv全文也将其链接为 web report。它展示的 RoboDojo 48%/26%、RoboLab 98%/92% 与扩展论文相应章节一致，因此本库将它作为本篇的**早期/交互版来源**。

早期网页的双臂实验采用 xhigh reasoning、根据公开 $\pi_{0.5}$ 成功率分层选择10个任务，偏重低成功任务；它不是完整 RoboDojo 排行榜，也不是 Astranomy 的 RoboLab-120。网页中的 public policy 值按相同任务/场景比例重加权，不是所有对照在同一新协议下重跑。阅读时分别引用，不从早期网页推导六域通用结论。

## 我的理解

这篇最有价值的是“谁负责什么”：Astra 可以改目标、重定位、判断何时交回控制权，学习型策略负责熟练轨迹，解析执行器负责坐标和运动接口。一个新的控制决定必须经物理结果验证；合理解释、错误诊断、有效修正和稳定重现是四件不同的事。

## 推测与问题

- 如何给混合策略分配因果信用？需要相同初态、相同审核频率与相同动作预算的干预实验，而不是只数修正次数。
- 如果移除暂停机制，性能退化来自观测过期还是低层控制？应加入延迟匹配和异步执行对照。
- 跨 trial 的文字经验是否能在新任务/新初态迁移？当前局部重复成功不构成普遍结论。

## 阅读可信度检查

- [x] 分别核对各域样本量、指标与控制组件。
- [x] 记录Score有效样本差异、动作来源比例和暂停physics。
- [x] 匿名网页作为版本来源合并，没有重复计作独立论文。
- [x] 未添加未核实公式或图片。

## 关联笔记

- [Agentic Policy 系列](../../series/agentic-policy.md)
- [Astranomy](astranomy.md)
- [Unexpected Robot Policy](unexpected-robot-policy.md)
- [Robot-use agents 观点文](robot-use-agents.md)
