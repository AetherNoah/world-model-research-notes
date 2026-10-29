---
type: paper
title: "Astranomy: An Empirical Study of GPT-6 Astra as a Native Robot Policy"
short_name: Astranomy
authors: [Jenai Xuning Yang, Ashna Khetan, Shun Zhang]
year: 2026
venue: NVIDIA technical report
paper_url: "https://research.nvidia.com/labs/srl/projects/astranomy/"
code_url: null
project_url: "https://research.nvidia.com/labs/srl/projects/astranomy/"
source_urls:
  - "https://research.nvidia.com/labs/srl/projects/astranomy/#final-configurations"
category: related-methods
series: agentic-policy
tags: [robotics, embodied-ai, agentic-policy, evaluation, long-horizon]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "固定通用模型，通过动作接口、历史和执行harness的系统消融，区分native policy、agentic policy与WAM协作的收益和成本。"
---

# Astranomy：通用模型成为机器人策略，需要怎样的接口和执行系统？

> **证据边界：**依据 NVIDIA 2026-10-14 技术报告的完整网页，重点核对 §2、§3.4、§4 与 §7。它是推理时接口和系统评估，没有提出新训练目标；下面的闭环表达是阅读用抽象。结果均为作者报告，未独立复现。

## 一句话总结

同一个 Astra，在持久上下文中控制机器人可以比逐次 API 调用更可靠，但成功率提升依赖动作、几何提示、记忆与反馈的共同设计，而且推理时间和 token 成本远高于专用策略。

## 论文明确内容

### 三种控制路径

| 形式 | 谁提出动作 | 上下文和执行方式 |
|---|---|---|
| Direct | Astra | 逐次观测到动作 API；不保留持久对话 |
| Agentic | Astra | 持久 Codex 会话，保留交互历史和运动反馈 |
| Orchestrated | Cosmos3-Nano-Policy 提案，Astra 审核 | 接受策略片段，或由 Astra 改写/接管 |

通用模型不是只输出任务分解：它能指定末端位姿、关节目标等数值动作。Orchestrated 则额外使用一个在 DROID 数据上微调的 WAM，因此不能把其结果解释为“无机器人先验”的直接控制。

### 最终配置，不能和中间消融混用

§3.4 Table 2 给出的最终 Direct 使用绝对 EEF 位姿；90 个任务采用 3–15 步动态动作片段，30 个历史消融任务保留 1–15 步。观测历史为 $t-14,t-7,t$ 三次观测，提供相机和夹爪几何信息。

Agentic 每次给出一个目标，保持 1–5 个控制步；保留完整文字和视觉交互历史、规划提示与 motion feedback，**最终配置关闭可选 native tools**。因此“Agentic 更好”不等于“更多 shell 工具更好”。Orchestrated 每 15 个控制步审核 WAM 建议，使用有界记忆，也关闭可选 native tools。

所有形式默认 medium reasoning、三相机视图，每幅 384×216。它们共享观测—决策—执行—再观测的闭环；这里没有权重更新。记作

$$
a_{t:t+k}=\Pi(o_t,s_t,h_t),\qquad
(o_{t+k},s_{t+k})=\operatorname{Execute}(a_{t:t+k}),
$$

只是统一接口的阅读记号，不是论文新增的优化公式。

### 实验协议

- 主评估是 RoboLab-120，120 个任务、每任务 5 个固定 seed；先在 30 个任务上消融，再评估各形式最强配置。
- DROID 机器人、15 Hz 控制；**模型推理期间模拟器暂停**。
- 无 walltime 或重试次数上限，以环境完成条件或任务动作步数上限结束；每次调用最多 8,192 completion tokens。
- 报告同时提供 SR 和部分完成 Score，两者不能混为一个指标。

### 核对结果与成本

| 最终策略 | RoboLab-120 SR | Score |
|---|---:|---:|
| Direct | 66.3% | 76.7 |
| Agentic | 74.8% | 83.6 |
| Orchestrated | 66.5% | 79.1 |

以上是 Figure 2 的汇总，不是外部硬件实时表现。历史保留明显影响 Agentic；可选工具并未带来额外可靠收益，和 WAM 配合也没有明确超过原生生成。

§4.1 Table 3 中 Direct EEF 平均 18.56 秒/调用，Agentic 为 13.37 秒/调用；Agentic 的推理占估算 walltime 的 97.8%。**这些时延统计也仅基于成功 episode**。Figure 2、§4.2 与 Table A11 按成功 episode 报告 Agentic 30.9M tokens / $36.40、Direct 160k / $1.96。网页开头简写为每 episode，记笔记时采用图注和详细统计的成功样本口径，不把它当作全部尝试的无条件均值。价格与模型接口均是该评估快照。

### 作者承认的局限

每任务仅 5 次，未建立小幅差异的统计显著性；范围以 pick-and-place 为主，不覆盖插装、高动态或极长移动操作。几何提示仍可能导致重复夹爪调整、错误抓取和无效恢复。暂停 physics 将“能完成”与“来得及完成”分离；服务端内部处理不可见，也使模型本体与基础设施难以完全解耦。

## 我的理解

关键变量是控制闭环的设计，而不是一个孤立的模型排行榜。持久历史可能减少重复犯错，却会扩大上下文和重复输入成本；短动作片段提高纠偏机会，也增加调用次数。应同时考虑成功率、推理时间、动作执行时间和每成功任务成本。视频必须注明是否包含推理等待，不能只用 rollout 的流畅程度判断部署性。

## 推测与问题

- 在相同成功率下，事件触发审核能否比逐 chunk 审核更便宜？需要匹配时延和动作预算的实验。
- 真实异步环境中历史保留是否仍有相同收益？报告没有回答。
- Agentic 与 Direct 同时改变多个接口变量；不能把全部收益归因于“推理更强”。

## 阅读可信度检查

- [x] 核对最终配置、Figure 2 与 Table 3。
- [x] 区分完整评估和 30-task 消融，区分成功 episode 与全部尝试。
- [x] 暂停模拟、样本量、外部 baseline 时延硬件差异已记录。
- [x] 未把阅读抽象公式当作原文训练目标；无新增图片。

## 关联笔记

- [Agentic Policy 阅读路线](../../series/agentic-policy.md)
- [RoboDojo 上的直接策略评估](unexpected-robot-policy.md)
- [跨六领域的 Astra 评估](astra-embodied-policies.md)
- [Harness VLA](harness-vla.md)
