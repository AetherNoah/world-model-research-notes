---
type: paper
title: "An Unexpected Robot Policy: Early Evaluations of GPT-6 Astra on RoboDojo and Beyond"
short_name: An Unexpected Robot Policy
authors: [Wenbo Zhang, Kaixuan Wang, Yutao Ouyang, Xiaoyu Huang, Liyang Li, Kailun Su, Weiyang Jin, Wenhao Chai, Haotian Liang, Zhiyang Dou, Yue Chen, Tianxing Chen]
year: 2026
venue: null
paper_url: "https://arxiv.org/abs/2609.24170"
code_url: "https://github.com/RoboProbe/RoboProbe"
project_url: "https://robodojo-benchmark.com/report/gpt-6-astra-eval"
source_urls:
  - "https://arxiv.org/html/2609.24170v1"
category: related-methods
series: agentic-policy
tags: [robotics, embodied-ai, agentic-policy, evaluation, generalization, in-context-learning]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "在不接入学习型运动策略的条件下评估通用LLM直接输出EEF动作，并发现语义泛化、精细控制和示范ICL之间的明显能力差异。"
---

# An Unexpected Robot Policy：直接生成动作不等于可靠的运动能力

> **证据边界：**依据 arXiv:2609.24170v1（2026-09-21）全文，重点为 §3–§4、Table 1/3 与附录 A/C。下面区分作者实验、阅读解释与未证假设。未运行模型或 benchmark。

## 一句话总结

Astra 在 RoboDojo 的轴均值上超过同期公开策略，但其优势集中在语义和开放任务；精密接触、双臂协调仍弱，而且在该协议中加入一次示范反而降低平均成功率。

## 论文明确内容

### 设置与方法

论文将 **LLM as policy** 限定为：模型权重固定、不做机器人任务微调；模型与机器人之间不接入学习型 motor policy。EEF 目标经几何 IK、路径转换执行，允许非学习的后处理，因此并不是原始文本直接驱动电机。

每轮模型读取任务说明、来自 wiki 的任务 recipe、三路 RGB、自身状态和预算。recipe 包含评分阶段，但不直接提供动作序列。文字历史保留观测描述、notes、工具调用与结果，图像只保留最近两轮。模型必须调用 `move_eef` 或 `give_up`，不提供现成的 pick/place/pregrasp 技能。

`move_eef` 可以只指定某只手的部分位置、朝向和夹爪分量；未指定维度维持当前值，双臂可同步运动。软件校验目标并规划关节路径，以 25 Hz 执行。拒绝的请求不运动，返回错误并重试；接受的请求执行后返回到达误差。这不是碰撞安全性的保证。模型没有 metric depth、物体真值位姿、reward state 或布局特定解。

### 没有新训练目标

这是冻结模型的评估，不应为它补写 imitation loss 或 RL reward。关键“学习”问题是上下文中的示范利用和 episode 内反馈纠偏，二者都不等于参数更新。

### Benchmark 协议

Astra 和 GPT-5.5 各在 42 个任务上运行 50 episodes/task，共 2,100 trials/model；DeepSeek-Flash 因成本只运行 10 episodes/task，共 420 trials。均 medium reasoning，默认每 episode 100 次模型调用。比较对象是 2026-09-10 的 40 个公开机器人策略。

**Average 是五个能力轴均值，不是把 2,100 个 episode 简单池化。** Score 是平均过程 reward×100，SR 是完整成功比例；Generalization 同时平均 standard/randomized 条件。公开策略没有用同样 wiki recipe 重跑，因此输入信息并不完全匹配。

### 主要结果

Table 1 中 Astra 为 28.97 Score / 22.48% SR；GPT-5.5 为 1.13 / 0.88%，DeepSeek-Flash 为 2.99 / 1.92%（采样较少）。Astra 的 Precision 轴 SR 仅 4.00%、Long-Horizon 为 8.25%，并不支持“通用LLM已解决机器人控制”的结论。

Table 3 特别值得与 ICL 笔记对照：34 个任务、每任务 10 个匹配布局，共 340 个同条件测试。示范来自**同一任务的不同布局**。

| 条件 | 成功数 | SR |
|---|---:|---:|
| 匹配 zero-shot | 78/340 | 22.9% |
| 图像 + EEF 示范 | 61/340 | 17.9% |
| 文字轨迹示范 | 44/340 | 12.9% |

图像示范下降 5.0 个百分点，文字下降 10.0。不能把这结果推广为“机器人 ICL 无效”：它只评估当前模型、单次异布局示范及具体表示。另有 8 个模型原本能解决的 `general_pickup` 布局上的扰动个例，展示 episode 内修正；这种有筛选的小样本不是独立泛化 benchmark。

### 局限与安全边界

与专用策略对比的输入信息不同；DeepSeek 评估预算不同。real-robot 试验因不安全动作而暂停，说明模拟 benchmark 排名不建立真实安全控制。模拟器内成功、生成代码后在快速控制循环中执行，以及逐次调用LLM直接控制是不同协议；论文后半的钢琴 F1 也不能与 RoboDojo Score 作横向数值比较。

## 我的理解

这个结果更像一张**能力剖面图**：语义推理补上某些专用策略缺口，但动作误差会在接触任务中被放大。一次示范既可能提供行为信息，也可能引入与当前布局冲突的几何信息；要判断“学没学会”，需要任务语义转移、轨迹格式和空间重绑定的独立消融。

## 推测与问题

论文提出 precision bottleneck、布局迁移和上下文格式等可能解释，但没有隔离因果。后续可以固定动作接口，比较零示范、仅任务语义示范、坐标规范化示范与完整视觉动作示范，并同时记录成功和动作误差。不要把个别成功纠偏直接解释为稳定的在线系统辨识。

## 阅读可信度检查

- [x] 核对 §3 的信息边界、轴平均和采样量。
- [x] 核对 Table 1/3 的绝对值与 ICL 负结果。
- [x] 区分 benchmark、筛选扰动案例和真实机器人演示。
- [x] 未引入未核实公式或图片。

## 关联笔记

- [Agentic Policy 系列](../../series/agentic-policy.md)
- [Astranomy](astranomy.md)
- [跨六领域评估](astra-embodied-policies.md)
