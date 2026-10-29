---
type: paper
title: "RoboBRIDGE: A Modular Framework for Bridging Policies to Robust Real-World Robotic Agents"
short_name: RoboBRIDGE
authors: [Sihyung Yoon, Minjong Yoo, Sanghyun Ahn, Seojeong Choi, Honguk Woo]
year: 2026
venue: IROS 2026
paper_url: "https://arxiv.org/abs/2607.27881"
code_url: null
project_url: null
source_urls:
  - "https://arxiv.org/html/2607.27881v1"
category: related-methods
series: agentic-policy
tags: [robotics, embodied-ai, agentic-policy, long-horizon, generalization, evaluation]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "用监测、感知、规划、控制和机器人接口五模块给预训练策略补上失败诊断与分层恢复，并通过primitive-specific LoRA缓解域变化。"
---

# RoboBRIDGE：动作预测器如何变成能监测和恢复的机器人 agent

> **证据边界：**依据 arXiv:2607.27881v1（2026-07-30）全文、Algorithm 1、Tables I–IV。arXiv comments注明IROS2026接收；不推断未提供的代码地址。数字为作者报告，未独立复现。

## 一句话总结

把失败检测、场景更新、重规划和可切换局部控制器显式放进执行系统，比仅让VLA持续前向预测更能处理部署异常；但harness本身的模块和规则仍需训练、标定与维护。

## 论文明确内容

### 五模块和两阶段监测

1. **Perceptor** 将观测转成object-centric scene state，异步写入共享buffer。
2. **Planner** 根据指令和场景分解primitive sequence；执行结果偏离时更新后续primitive，而非盲目重放原计划。
3. **Controller** 将primitive转为动作，可用VLA、解析IK或脚本。
4. **Robot Interface** 适配不同机器人的速度、delta动作、IK目标等控制API。
5. **Monitor** 先做低成本成功/失败检查，必要时调用更强诊断模型。恢复目标依次包括retry、regenerate、replan、re-perceive，选择足以修复问题的较小代价层级。

Phase 1监测约5Hz，位于控制循环之外；它不是要求LLM在5Hz上完整推理。异步感知则避免每个低层动作都等待重新理解场景。论文的完整闭环是“观察—计划—执行primitive—检查—选择恢复层级”，而不是一次prompt后整段开环执行。

### Primitive-specific LoRA

作者将任务分成稳定的move/grip等primitive，并在冻结VLA骨干上训练对应adapter。原文Eq.(5)的接口是

$$
f_{\theta+\Delta\theta_k}:(i,\mathbf{s}_t,p_t)\mapsto\mathbf{a}^{(k)}_t,
$$

其中$i$是详细指令，$\mathbf{s}_t$汇合当前观测和机器人状态，$p_t$是primitive标签，$\Delta\theta_k$是专属LoRA。因此应区分“harness不要求重训base weights”和“实验没有训练”：后者不成立。骨干冻结，但adapter需要对应示范。

### 实验设置

仿真覆盖LIBERO与RoboCasa atomic 24 tasks；后者每任务50条人类示范、共1,200条，用SpaceMouse采集。比较SmolVLA、$\pi_{0.5}$、GR00T-N1.5的直接部署和加RoboBRIDGE。真实实验使用Franka Research3与UR7e。

实现中Perceptor固定Florence-2，主Planner使用Claude Opus4.6。GR00T-N1.5-3B对DiT action head和vision projector训练Move/Grip LoRA，rank128、alpha256、dropout0.1；部署采用chunk stride4、EMA平滑、10步denoising、1000步上限。这些执行与训练细节也是实验的一部分，不能只把提升归给“加了agent”。

### 结果和消融

| 原文表格 | 不加harness | 加harness |
|---|---:|---:|
| Table I LIBERO-Object均值 | 4.7% | 10.0% |
| Table I LIBERO-Spatial均值 | 72.4% | 73.5% |
| Table I LIBERO-Long均值 | 10.6% | 20.0% |
| Table II RoboCasa / GR00T | 4.2% | 9.8% |
| Table II RoboCasa / SmolVLA | 3.4% | 6.9% |

这些绝对成功率提醒我们：相对提升与高可靠性不是一回事。不要写成“普遍接近100%”或挑选少数成功任务代表全部24任务。Table III中Opus planner开启Monitor后6.6%→14.7%，小模型增益较小；这是特定消融protocol，不可与Table II主结果拼成一行。Table IV在5个任务上，LoRA控制器15.3%→27.1%，full fine-tuning31.9%→40.0%，支持harness和控制器能力相互补充。

### 局限

遮挡下感知错误会触发错误计划；接触失败可能不可逆地改变物体配置，多次重试也无法补救。作者指出当前primitive偏单臂桌面，难直接覆盖变形物体、双臂和复杂接触；监测阈值及恢复规则仍手工设定，未来需更明确的可行性验证和停止条件。主文未给统一跨模型端到端时延，不能由5Hz monitor推导整个系统实时性。

## 我的理解

真正可复用的是“状态变化应由哪个模块负责、失败应退回到哪一层”这个组织方式。primitive adapter增加局部专门性，harness增加纠错性；两者是不同贡献。与Harness VLA的固定冻结contact primitive相比，RoboBRIDGE对技能粒度、adapter和异步monitor投入更多，比较时必须对齐训练数据、观测信息和恢复预算。

## 推测与问题

强monitor可能更会发现错误，也可能频繁误报、打断本来可成功的动作。后续应同时测false positive/negative、恢复成本、不可逆失败率和最终SR，而不是只报告有/无monitor的差值。需要将数据/adapter改进与orchestration改进做因子分解。

## 阅读可信度检查

- [x] Eq.(5)符号与primitive-specific训练边界已核对。
- [x] 数字来自Tables I–IV，未跨protocol拼接。
- [x] 已说明绝对SR偏低、缺少完整时延结论。
- [x] 真实演示未描述成大规模无监督部署；无新增图片。

## 关联笔记

- [Agentic Policy 系列](../../series/agentic-policy.md)
- [Harness VLA](harness-vla.md)
- [Robot-use agents](robot-use-agents.md)
