---
type: series
title: "Agentic Policy：从通用模型直接控制到可恢复的机器人执行系统"
papers: [astranomy, unexpected-robot-policy, astra-embodied-policies, harness-vla, robobridge, robot-use-agents]
status: active
updated: 2026-10-29
---

# Agentic Policy：从通用模型直接控制到可恢复的机器人执行系统

> **证据边界：**这是用户给出的七个链接的阅读路线与跨工作比较，不是同一研究团队的模型代际。匿名网页与2609.38537的重叠结果合并记录，因此对应六篇独立笔记。所有数字为原作者报告，未进行统一环境复现。

## 阅读顺序与原始输入映射

1. [Robot-Use Agents](../papers/related-methods/robot-use-agents.md)：先理解通用agent调用机器人的观点框架。它是评论文章，没有自己的实验。
2. [An Unexpected Robot Policy](../papers/related-methods/unexpected-robot-policy.md)：不接学习型motor policy的直接控制；完整RoboDojo协议与示范ICL负结果。
3. [Astra as Embodied Policies](../papers/related-methods/astra-embodied-policies.md)：从双臂扩展到六类任务，检查低层控制器究竟提供了什么。
4. [Astranomy](../papers/related-methods/astranomy.md)：控制模型不变，系统消融动作表示、历史、几何提示、持久harness和WAM审核。
5. [Harness VLA](../papers/related-methods/harness-vla.md)：将冻结VLA封装成接触primitive，靠记忆和空间重绑定实现新组合。
6. [RoboBRIDGE](../papers/related-methods/robobridge.md)：用五模块、异步感知、层级恢复和primitive adapter形成执行系统。

用户原始第3项[匿名交互报告](https://anonymous-report-421.github.io/public-website/?view=1)标题为*GPT 6 Astra as an Embodied Policy*。它重点报告10-task RoboDojo/RoboLab，和原始第4项[arXiv:2609.38537](https://arxiv.org/abs/2609.38537)中对应实验一致；完整论文扩展到六个域。它仍是可独立阅读的网页来源，但不在本库重复制造一篇相同结果的论文笔记。版本区别和署名见对应笔记。

## 三种“agentic”，不是同一个动作模型

| 路线 | 模型输出 | 运动能力来自哪里 | 学习/适应发生在哪里 |
|---|---|---|---|
| Native/direct policy | EEF/joint目标或短轨迹 | 模型决策 + 解析IK/PD/接口 | 固定权重；上下文与反馈 |
| Hybrid / policy review | 接受策略候选或局部修正 | 预训练policy/whole-body controller + 模型审核 | 策略已有机器人训练；审核时history |
| Primitive harness | primitive调用、参数和恢复决策 | 解析技能 + VLA接触技能 | 任务记忆、全局规则；部分方法另训LoRA |

“LLM权重冻结”只描述某个组件，不能推出零训练、零示范或没有外部技能。Astra的导航primitive、全身tracker、Harness的task memory、RoboBRIDGE的LoRA训练都是结果的一部分。

## 七个比较轴

### 1. 谁提出动作，谁保证动作可执行？

必须记录坐标系、EEF/关节表示、action chunk、IK或路径规划、控制频率、碰撞/到达反馈。能输出合理坐标不等于具备运动规划或接触稳定性；模块之间的hidden competence不能归给LLM。

### 2. 观测与反馈是否同条件？

RGB、RGB-D、metric world map、机器人state、官方task-success signal与物体真值是不同的信息预算。即使没有真值位姿，也可能有额外几何提示或wiki recipe。不能把它们和仅靠RGB+自然语言的策略直接作因果归因。

### 3. 历史、记忆与参数更新分开

持久对话、最近两张图、跨episode成功trace、规则文件、fast weights和梯度微调不属于同一个“online learning”。本组主要研究harness/上下文；与ICL/TTT系列比较时，要说明更新对象和测试数据边界。

### 4. Benchmark数字不能拼成排行榜

- Unexpected Robot Policy用42任务×50 episodes，Average为五个能力轴均值；其22.48%不能和特选10-task的48%直接比较。
- 匿名报告/RoboDojo对低成功任务分层选择；广域论文中48%来自50个配对实例。
- Astranomy是RoboLab-120每任务5次，Agentic74.8%；匿名页RoboLab98%只是10-task子集。
- Harness VLA的LIBERO-Pro、RoboCasa365和RoboTwin不共享同一任务分布；RoboBRIDGE的RoboCasa atomic24 tasks也不是同一协议。

### 5. 成功率、过程Score、路径效率各自报告

部分完成Score不是SR，HumanoidBench某任务return领先也不是成功任务比例。导航高SR可能伴随低SPL和绕行。完整评估应包含局部失败、不可逆失败、误恢复和停止条件。

### 6. 模拟暂停和walltime决定部署边界

Astranomy推理时暂停physics，Direct平均18.56s/调用；广域Astra的30s locomotion最终尝试包含250次、平均39.86s/次的同步调用。动作视频的模拟时间与真实等待时间必须分开。部署时世界不会等待模型完成推理。

同样，模型延迟均值、不同硬件上的专用策略中位数、源轨迹工具时长和planner时长不能直接相除成系统加速比。Harness Flash表尤其需要保留选择protocol与计时定义。

### 7. 恢复何时真正有用？

重试只对可恢复状态有价值。接触把对象推落、遮挡导致错误grounding、monitor误报打断成功动作，都可能使“更会反思”反而变差。应测量诊断正确率、恢复引起的额外风险/成本，以及是否保留已经成功的行为，而非只展示最漂亮的修正视频。

## 可以形成的研究问题

**个人理解，尚未由这些论文统一证明：**世界模型可以为agent提供动作后果和反馈预期，harness可以给world-action model提供错误分类、retry边界和验证接口。值得研究的不是再加一层自由文本planner，而是把调用决策、预测不确定性和可验证执行条件连接起来。

一个可检验协议应固定低层policy、观测、示范和时间预算，分别比较逐chunk审核、事件触发审核与无审核；记录SR、失败恢复率、不可逆错误、模型调用数、tokens、包含推理的walltime。然后再测试移除暂停physics或加入真实通信延迟时的退化。这个协议是后续研究建议，不是上述论文已完成的实验。
