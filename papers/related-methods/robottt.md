---
type: paper
title: "RoboTTT: Context Scaling for Robot Policies"
short_name: RoboTTT
authors: [Yunfan Jiang, Yevgen Chebotar, Ruijie Zheng, Fengyuan Hu, Yunhao Ge, Jimmy Wu, Tianyuan Dai, Scott Reed, Li Fei-Fei, Yuke Zhu, 'Linxi Jim Fan']
year: 2026
venue: null
paper_url: "https://arxiv.org/abs/2607.15275v1"
code_url: null
project_url: "https://research.nvidia.com/labs/gear/robottt/"
source_urls:
  - "https://arxiv.org/html/2607.15275v1"
  - "https://research.nvidia.com/labs/gear/robottt/"
category: related-methods
series: robot-in-context-learning
tags: [robotics, in-context-learning, test-time-training, transformer, long-horizon, generalization]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "把长时序机器人经验压入可在线梯度更新的fast weights，以sequence action forcing和TBPTT支持8K步上下文。"
---

# RoboTTT：以 fast weights 扩展机器人策略上下文

> **证据边界：**依据 arXiv v1 的第 2–4、6 节与附录阅读；数字为作者报告，未运行代码或复现。下面的 TTT 是小型记忆网络的测试时梯度更新，不能写成“完全不更新参数的 ICL”。

## 一句话总结

RoboTTT 把历史从不断增长的 token 缓存变成固定规模的 fast weights；策略在执行中持续写入、查询这份记忆，以缓解长任务的状态混淆和遮挡。

## 论文明确内容

### 方法：单步注意力与跨步记忆分工

实例基于 GR00T N1.7，在 16 层 DiT 动作头中加入 TTT 层。单步 self/cross-attention 处理当前视觉语言、机器人状态和带噪动作；TTT 沿时间轴处理输入。16 个 register tokens 搬运视觉语言信息，避免把全部视觉 token 都送进记忆网络。TTT 残差经接近零初始化的 tanh gate 注入，减少对预训练能力的初期干扰。

第 2 节式 (1)–(2) 的写入、读取分别为：

$$
W_t=W_{t-1}-\eta\nabla_W\mathcal L_{\rm FW}(f_{W_{t-1}}(K_t),V_t),\qquad O_t=f_{W_t}(Q_t).
$$

这里 $W$ 是 fast weights；投影和初始化 $W_0$ 是通过训练学出的 slow parameters。部署时 slow parameters 冻结，而 $W_t$ 随观察继续更新。每次 rollout 从学到的 $W_0$ 开始，不能把它理解为永久改写基础模型。

### 训练如何让记忆真正服务动作

- **Sequence action forcing：**每个动作 chunk 独立采样 flow-matching 噪声级别，避免整条序列同易同难。
- **TBPTT：**段间保留 fast weights 的值，但截断梯度图；显存受分段长度约束，并非完整 8K 步反传。
- **视频示范：**同任务 human video 与 robot trajectory 串联，视频段只更新记忆，动作监督只施于机器人段。
- **DAgger distillation：**失败动作和人工纠正都写入记忆，但外层 imitation loss 仅监督人工纠正，让模型学习“失败到修正”的过程。

因此 one-shot video 的能力仍依赖离线配对训练；它不是向一个从未学过该接口的 VLA 临时塞视频。

### 实验设置与结果

第 4 节使用 YAM 双臂、四路 RGB，在 Pup Go Car、Circuit、Gear Bot 三个装配任务上评估。分别收集 8、6、5 小时任务数据；Circuit 有 80 种配置，20 种用于训练、60 种留作测试。每任务 20 次评估，耗时更长的 Gear Bot 为 10 次。

| 证据 | 作者报告 | 如何解读 |
|---|---|---|
| 主评估 Fig. 7 | 平均任务完成分数 79%，单步 GR00T 为 42% | “提高 87%”是相对完成度提升，不是成功率增加 87 个百分点 |
| Table 1 完整成功次数 | 9/20、13/20、2/10 | 完整成功与部分阶段得分必须分开；Gear Bot 的 2/10 也意味着仍频繁失败 |
| Fig. 8 上下文扩展 | 8K 为 71.5%，1K 为 43.9% | 此组早于主评估中的 DAgger 训练，不能与 79% 拼接成同一对照 |
| Table 2 人类视频 one-shot | Circuit 完成分数 65%，6/10 全成功；GDN 为 33%、0/10 | 固定指令无法区分目标配置，任务信息确实来自视频 |

### 作者局限

第 6 节指出长上下文训练成本依然上升、TTT 内层目标尚非机器人任务专用，并未消除全部部署失败。固定大小记忆只限制推理状态规模，不等于训练成本或每步计算免费。

## 我的理解

贡献在于把“如何压缩历史”变成可由外层任务目标塑造的学习规则。相比给策略加一张历史图，fast weights 可以存储被遮挡对象和阶段进度；但它仍可能把错误行为写成不良上下文。DAgger 数据的监督掩码，是让失败成为条件而非模仿目标的关键。

## 推测与问题

- 在新接触模式下，增大历史是否仍有用，还是只记住更长的失败轨迹？
- 固定记忆在极长 rollout 中会否遗忘早期关键约束？应按时间间隔和任务阶段分组测试。
- 视频配对中的任务标签、配置重叠和相机线索，需要与“学会全新原语”区分。

## 阅读可信度检查

- [x] 区分部分完成度、完整成功次数和相对增幅。
- [x] 快慢权重与写入公式核对第 2–3 节。
- [x] 未声称复现；未添加未核验的开源训练代码链接。

## 关联笔记

- [机器人 ICL 系列](../../series/robot-in-context-learning.md)
- [WAM-TTT](wam-ttt.md)：同为 fast weights，但在视频侧适配后固定权重执行。
