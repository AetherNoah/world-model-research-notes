---
type: paper
title: "Harness VLA: Steering Frozen VLAs into Reliable Manipulation Primitives via Memory-Guided Agents"
short_name: Harness VLA
authors: [Yixian Zhang, Huanming Zhang, Feng Gao, Xiao Li, Zhihao Liu, Yi Nie, Chunyang Zhu, Jiaxing Qiu, Yuchen Yan, Jiyuan Liu, Wenhao Tang, Jiaji Rao, Zhengru Fang, Changxu Wei, Yu Wang, Wenbo Ding, Chao Yu]
year: 2026
venue: null
paper_url: "https://arxiv.org/abs/2607.08448"
code_url: "https://github.com/RLinf/RPent"
project_url: "https://harnessvla.github.io/"
source_urls:
  - "https://arxiv.org/html/2607.08448v5"
category: related-methods
series: agentic-policy
tags: [robotics, embodied-ai, agentic-policy, long-horizon, generalization, evaluation]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "将冻结VLA封装成可重试的局部接触技能，用固定解析primitive、任务记忆和全局失败知识完成语义重绑定、搬运和恢复。"
---

# Harness VLA：让冻结 VLA 成为可调度的接触专家

> **证据边界：**依据 arXiv:2607.08448v5（2026-09-24）全文与 Table 1–7、附录 A–G。网页摘要和 HTML 对 RoboCasa 的提升百分点存在不同表述，下面采用表格绝对数和明确对照，不混合口径。未运行 RPent 或机器人实验。

## 一句话总结

VLA 负责它擅长的局部接触，agent 负责语义重绑定、非接触移动、重试和记忆；核心不是在部署时生成更多技能，而是把同一冻结策略以固定、可审计的接口反复用好。

## 论文明确内容

### 方法与执行循环

单体 VLA 在目标重定向、位置交换或长序列中可能重放训练轨迹；纯解析技能又难处理不规则抓取和受限放置。Harness VLA 将 VLA 作为 `vla_act` primitive，只在需要局部接触时短暂执行。其余阶段用固定解析接口，例如 `move_to`、`move_pose`、`rotate_wrist`、`rotate_pitch`、`set_gripper`、`release`，移动操作另有 `navigate_to` 和 `move_base`。

planner 依据当前 RGB-D、proprioception 和记忆生成结构化 JSON primitive 调用；执行后等待结果、观察是否建立接触或到达目标，再重定位和调用下一个 primitive。这里不是让 agent 任意生成 Python 控制程序，也不是每一步由 LLM 预测底层关节动作。

记忆分两层：**Task Specific Memory** 保存 reference seed 上探索成功的 primitive trace 与语义总结；**Global Memory** 保存可复用成功条件和失败模型。复用的是流程和约束，空间参数必须在当前图像/深度中重新 grounding。附录列有官方成功信号，但禁止读取物体真值位姿和 simulator internals；不能因此把该系统概括为“没有任何环境反馈”。

### 训练目标与参数边界

部署中的 VLA 权重冻结，不等于整个实验从未训练：各 benchmark 使用已发布或作者 post-trained 的 VLA checkpoint。本文主方法不提出联合端到端训练loss。任务seed上的记忆构造也不同于 zero-shot；论文分别报告 bootstrapped memory 和不检索 Task Specific Memory 的设置。

### 结果与协议

| 表格/环境 | 评估量 | 结果 |
|---|---|---|
| Table 2 标准LIBERO | 40 tasks×10 seeds=400 rollouts | Harness CC 96.0%，内部冻结baseline95.3% |
| Table 3 LIBERO-Pro | 8 cells×100 trials=800 | CC82.4%，Codex72.1%，同checkpoint直接策略50.0% |
| Table 4 RoboCasa365 | 50 tasks×5 held-out seeds=250 | Codex57.2%，CC48.8%，直接RLDX-1为30.0% |
| Table 6 RoboTwin C2R | 50 tasks×5 randomized seeds=250 | CC58.4%，Codex58.0%，直接LingBot-VLA50.4% |

LIBERO-Pro 同时覆盖 instruction-redirection(T) 和 position-swap(S)。一些外部比较只报告六个非LIBERO-10 cells；其overall与八个cells的overall并不完全同口径。RoboCasa每任务reference seed只用于记忆bootstrapping，评估用另5个seed。

不要直接抄摘要的“+38.6/+25.4/+27.1pp”：检索时 abs摘要与v5 HTML摘要的RoboCasa数字不一致；Table4的57.2−30.0=27.2pp是相对内部冻结baseline，57.2−35.3=21.9pp是相对表中WorldDreamer，两者也不是同一个对照。表格与版本必须一起记。

### 消融和 Flash Mode

VLA调用上限实验显示少量局部调用便有收益，但多次重新staging和再次调用仍重要，不能将其简化成一次VLA输出后的纯程序执行。Flash Mode把成功流程编译成task card，通过VLM+深度重新定位waypoint后按固定顺序执行，适合固定子目标顺序。Table7有72.63%（581/800），但候选卡是在**报告相同seeds**上重放后选max，属于选择过的结果。Flash时间统计是源轨迹tool duration，而Codex列为planner duration，不能把56.17s与529.23s直接当成端到端加速比。

### 局限

作者指出planner和底层VLA之间反馈仍不充分，尚无环境reward/人类偏好的联合优化，杂乱长任务受细粒度视觉描述限制。真实双Franka实验由操作者恢复初始场景并确认最终成功；这不是完全无人值守的完整评测闭环。memory构造、人工规则和解析技能都有工程成本。

## 我的理解

这是一种职责不对称的层次控制：agent负责“在什么条件下调用”，VLA负责“如何完成局部接触”。它的可靠性来自接口契约、重新grounding和重试，不宜只归因为基础LLM更强。与RoboBRIDGE相比，Harness VLA强调固定词表和记忆；后者的primitive LoRA还涉及技能训练。

## 推测与问题

未来应分别统计reference-seed探索预算、在线规划预算和新seed泛化。自动扩技能可能提高覆盖率，也会破坏固定接口的可审计性；需要有准入验证和反例。Flash卡选择若改成独立验证seed，性能和时延收益还剩多少，仍需实验。

## 阅读可信度检查

- [x] 核对版本、绝对结果、bootstrap/held-out区别。
- [x] 明确冻结策略不等于零训练或零示范。
- [x] 记录摘要数字不一致、Flash选择偏差与时延统计不同。
- [x] 没有添加假公式或未经核实的图。

## 关联笔记

- [Agentic Policy 系列](../../series/agentic-policy.md)
- [RoboBRIDGE](robobridge.md)
- [Astranomy](astranomy.md)
