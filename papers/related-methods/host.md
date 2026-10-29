---
type: paper
title: "HOST: Robots Acquire Manipulation Skills in Seconds from a Single Human Video"
short_name: HOST
authors: [Guangyan Chen, Meiling Wang, Te Cui, Zichen Zhou, Qi Shao, Shalfun Li, Hang Su, Roy Gan, Hao Wang, Mengyin Fu, Yi Yang, Yufeng Yue]
year: 2026
venue: null
paper_url: "https://arxiv.org/abs/2607.20033v4"
code_url: "https://github.com/CGuangyan-BIT/HOST"
project_url: "https://x2robot.com/pages/host"
source_urls:
  - "https://arxiv.org/html/2607.20033v4"
  - "https://github.com/CGuangyan-BIT/HOST"
category: related-methods
series: robot-in-context-learning
tags: [robotics, in-context-learning, world-action-model, video-prediction, diffusion, generalization]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "以共享任务进度对齐人类视频与机器人轨迹，再级联预测进度、机器人未来观察和动作，实现无任务专属更新的单视频技能条件化。"
---

# HOST：先找到示范中的进度，再预测自己的未来与动作

> **证据边界：**依据2026-08-20的arXiv v4第2、4节和补充材料；实验未复现。“29秒”是作者定义的新技能获取时间，不能直接理解为模型每步延迟、完整任务执行时间或从零训练耗时。

## 一句话总结

HOST 不让策略从完整人类视频直接回归动作，而是建立人—机器人任务进度对应，再按“当前进度→机器人未来画面→动作”的顺序生成输出。

## 论文明确内容

### 两个互相配合的机制

**1. 训练目标与示范未来耦合。**人类与机器人执行同一任务可能不同速，固定时间偏移的监督会把示范和动作错配。第4.2节以共享视觉embedding、Smooth DTW和时间循环一致性学习单调对应；先把当前机器人帧映射到人类示范阶段，再从人类视频的接下来一段反查机器人时间步，构造未来观察/动作监督。它不是仅给输入视频做裁剪，而是重新定义目标。

**2. Self-grounded cascade。**第4.3节以Wan2.2视频expert和动作expert组成MoT；条件含视频window、机器人观察、本体状态及指令。先生成归一化进度token，再生成机器人视角的未来观察，最后以这些输出为条件生成动作。顺序由因果attention和干净/带噪目标token实现。

第4.4节式(17)将推理写为：

$$
\hat y_i=\operatorname{Denoise}(z^d,z^o,c,\hat y_{<i}),\quad i\in\{p,o,a\}.
$$

这是**生成顺序**，不等于从观察数据识别真实物理因果图。对齐网络训练使用时间循环一致性与DTW损失；策略采用加权进度、观察和动作flow-matching目标。

### 数据与部署

Stage 1：193,462条、229个任务的同embodiment机器人轨迹配对，建立跟随过程的能力。Stage 2：5,847段人类示范与同任务机器人轨迹配对。测试只提供一段新任务人类视频，不再做任务专属参数更新；执行中按预测进度移动视频window。

所有真机评估均使用双ARX R5、两腕部加一第三人称RGB相机、20维双臂动作空间。50个新任务用于展示技能覆盖，各任务20次随机初态试验；与baseline、SFT的定量比较另用论文规定的新任务集合，不能混淆任务集合。原始视频没有机器人动作标签，但训练阶段仍依赖大量机器人动作和配对数据。

### 结果与消融

| 来源 | 作者报告 | 解读 |
|---|---|---|
| 第2.4–2.5节/Fig.5–6 | 新任务对比平均成功62%；最强50-demo SFT为56% | 差6个百分点；不是“只看1视频便任意任务100%” |
| Fig.6获取成本 | 平均29秒；相比最快SFT约507倍 | 包含新任务示范收集/离线微调的对照口径，不是推理速度比较 |
| Fig.9目标耦合 | 整段视频0.21；按时间window0.29；进度window0.45；完整耦合0.62 | 支持时间对齐与目标构造，而非纯粹增大上下文 |
| Fig.10A级联 | 直接动作0.34；加定位0.43；并行视觉0.55；视觉→动作级联0.62 | 这些变体已保留目标耦合，与Fig.9是另一组消融 |

保留原有技能的比较基于7个训练内任务。权重不改写使HOST避免了SFT参数覆盖，但仍需正确检索/提供旧任务视频，并不意味着新程序被永久压入模型权重。

### 适用范围与局限

讨论强调当前主要在一个双臂平台验证；大幅改变机器人形态仍需研究。视频无法直接提供力和触觉，精细接触存在信息缺口。人—机器人配对与进度对齐质量、预测画面的正确性，都会影响后续动作；级联也可能把上游错误传给下游。

## 我的理解

HOST 与WAM方向的连接在于：预测自己的未来画面可以成为跨embodiment的中间接口。然而本文主要解决示范跟随，不能据此声称它已经是任意动作可控的通用世界模型。最有价值的设计，是让目标构造与模型推理顺序都服务“示范现在要求做哪一步”。

## 推测与问题

- 若示范跳步、回退或包含错误，单调DTW是否会错误重排目标？
- 未来画面生成精美但动作不可行时，有无约束保证两者一致？
- “技能获取”与“固定参数条件化”应在评测中同时报告，避免与训练时学习速度混比。

## 阅读可信度检查

- [x] 公式核对第4.4节，分清目标耦合与级联两组消融。
- [x] 数字注明任务集合与时间计费口径；未将507倍当成控制频率。
- [x] 未声称复现或任意机器人形态泛化。

## 关联笔记

- [机器人 ICL 系列](../../series/robot-in-context-learning.md)
- [WAM-TTT](wam-ttt.md)
- [BPP](behavior-prompting-policy.md)
