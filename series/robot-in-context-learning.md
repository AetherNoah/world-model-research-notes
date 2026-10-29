---
type: series
title: "机器人 ICL 与 TTT：示范、快权重和交互记忆的七种接口"
papers: [icrt, behavior-prompting-policy, robottt, wam-ttt, stellavla, zeva, host]
status: active
updated: 2026-10-29
source_urls:
  - "https://arxiv.org/html/2408.15980v2"
  - "https://arxiv.org/html/2606.30457v2"
  - "https://arxiv.org/html/2607.15275v1"
  - "https://arxiv.org/html/2607.06988v3"
  - "https://arxiv.org/html/2608.11671v1"
  - "https://arxiv.org/html/2608.30880v2"
  - "https://arxiv.org/html/2607.20033v4"
---

# 机器人 ICL 与 TTT：示范、快权重和交互记忆的七种接口

> **证据边界：**这是七篇相关工作的阅读比较，不是作者共同提出的统一benchmark，也不表示它们存在继承关系。各篇原文版本、任务、统计口径和未验证结论见对应单篇笔记；全部未复现。

## 先区分“什么在变”

这些工作都在部署时利用额外信息，但不是同一种适配：

1. **上下文条件化：**权重不变，示范token、窗口或检索前缀改变。ICRT、BPP、StellaVLA、HOST主要属于此类。
2. **Test-time training：**在部署时以梯度更新fast weights；slow backbone可保持固定。RoboTTT、WAM-TTT属于此类，不能笼统称“完全无参数更新”。
3. **交互记忆：**权重不变，但写入执行动作及实际效果；Zeva的BIT/PIM属于此类。它使用自己的失败经验，不只依赖预先给定的专家示范。

“Online adaptation”只说明适配发生在部署期间；它可能是以上任一种，不能单凭这个词判断是否训练权重、是否需要奖励或是否需要新机器人动作标签。

## 方法比较

| 方法 | 部署新增信息 | 写到哪里 | 是否更新权重 | 训练阶段必须具备的条件 |
|---|---|---|---|---|
| ICRT | 同一机器人空间的图像/状态/动作示范及当前历史 | 因果序列、KV cache | 否 | 同任务多轨迹与有歧义的多任务场景；仅监督prompt后动作 |
| BPP | sensorimotor示范，iPhUMI可在人演示时获取动作/状态 | 缓存分块prompt，当前观察cross-attend | 否 | 多任务BC数据；足够任务多样性；扩散动作监督 |
| RoboTTT | 长执行历史，可先读人类视频 | 动作DiT中的fast weights | **是，逐步更新fast weights** | 长序列训练、TBPTT、独立chunk噪声；视频/机器人配对与纠错数据 |
| WAM-TTT | 目标场景少量action-free人类视频 | WAM视频专家TTT分支 | **是，适配阶段更新fast weights；执行期间固定** | 配对human–robot元训练、视频预测和K/V重建接口 |
| StellaVLA | 检索到的一条结构化示范 | 固定多模态/语言前缀 | 否 | 离线子目标与2D/3D标注、leave-one-out检索、动作/语言并行训练 |
| Zeva | 自己的动作与观察到的效果，或人引导机器人warm-up | BIT/PIM与检索prompt | 否 | CTE表征学习及记忆条件化动作训练；匹配的执行/效果记录 |
| HOST | 一条普通人类视频及滚动进度window | 定位→自视角未来观察→动作级联 | 否 | 大量同embodiment机器人对，较小的人—机器人对，任务进度对齐 |

“人类示范”必须拆开看：ICRT的human-teleoperated demo实际含机器人动作；BPP的手持夹爪有运动测量；HOST/WAM-TTT测试输入可以不含机器人动作，但它们的离线训练并非无需动作。Zeva的一次人类warm-up是人引导机器人产生交互记录，不等于任意旁观视频。

## 三种值得分开的设计问题

### 1. 如何防止策略忽略上下文

ICRT让同一场景有多个可做任务，并掩掉prompt段的监督；BPP通过同任务不同轨迹配对与更高任务多样性形成读取压力；StellaVLA以错误示范替换实验检验任务条件。它们的共同思想不是“更长前缀自然有用”，而是训练分布必须让示范含有当前观察无法确定的信息。

这也提示一个捷径风险：若任务可由背景、物体身份或语言完全猜出，成功不证明模型学会了示范推理。相反，错误示范降低成功率只证明模型在使用条件，不能证明检索或解释永远正确。

### 2. 时间对应与记忆容量

- ICRT保存rollout历史，便于复用KV cache，但上下文仍受长度上限和旧历史干扰。
- BPP由当前观察查找示范片段，短历史在画布重置等情况下不保留整段冲突轨迹。
- HOST显式学习任务进度对应，避免人和机器人速度不同导致目标错配；其单调假设也可能在回退/跳步视频中失效。
- RoboTTT把历史压入可学习更新规则的记忆，推理状态大小固定，但记忆压缩和长序列训练不免费。
- Zeva按当前phase查找动作效果，BIT与PIM处理不同时间尺度；它的PIM默认只跨同一episode内的attempt。

### 3. 视觉预测如何影响控制

WAM-TTT把人类视频损失转成视频侧快权重，通过既有joint attention影响动作；HOST将机器人未来画面直接作为动作的前置生成目标。二者都利用视觉—动作耦合，但前者是**适配接口**，后者是**条件生成因子分解**。视频预测改善不自动保证接触和动作正确，需要对动作误差、完整成功及闭环反馈单独评估。

## 这些结果不能直接排成排行榜

| 论文里的常见数字 | 实际口径 | 容易产生的误读 |
|---|---|---|
| RoboTTT 79% | 三项装配的平均部分完成分数 | 当作所有任务79%完整成功 |
| WAM-TTT 46.2 | 9项New环境任务的progress | 当作46.2%任务成功率 |
| ICRT 79.2 | 含pick-and-place部分信用的汇总 | 与纯二值SR横比 |
| BPP 3.43 | DrawAnything绘图误差，越小越好 | 与LIBERO成功率混比 |
| StellaVLA 0.63 | VLA-Arena聚合分数 | 当成任意真机OOD成功率 |
| Zeva CSR增长 | 固定episode内多次尝试累计成功 | 当作一次尝试能力或永久跨episode学习 |
| HOST 29秒/507倍 | 新技能示范获取与SFT流程耗时对比 | 当作每步推理延迟或完整任务用时 |

还必须控制测试信息预算：有完整目标任务示范的策略，不能与只给一句语言的策略直接解释为相同条件下的通用能力差。预训练数据、骨干、机器人形态、动作频率与重试预算也需要对齐。

## 一个更严谨的评测记录表

未来自己复现时，每项实验至少记录：

- 测试任务是否真正未见，还是已见原语的组合；对象、背景和布局的划分是否独立。
- prompt来自哪条轨迹，是否泄漏待评估轨迹、未来观察或测试动作；构建retrieval pool是否排除了目标轨迹。
- prompt是否含动作、本体状态、力、标定、语言、人工子目标；人类和机器人是否同embodiment。
- 更新的是token缓存、外部记忆、fast weights还是完整策略；何时重置、是否跨episode保留。
- 示范数量、适配时间、峰值显存、每chunk延迟、完整执行时间与机器人控制频率。
- 失败后的重试次数、环境是否重置，SR与部分进度及CSR是否分别报告。

## 对世界模型研究的启发与待验证问题

我的理解是，这组工作把“模型能否预测世界”推进为“新上下文如何改变控制”。可能值得研究的方向是：把动作可控性误差作为示范检索或快记忆更新的可靠性信号；在记忆污染时主动弃用上下文；将任务进度与局部物理效果分开存储。它们属于后续假设，以上七篇并未共同证明这些方案有效。

## 阅读入口

- [ICRT：In-Context Imitation Learning via Next-Token Prediction](../papers/related-methods/icrt.md)；用户简称ICIL对应这篇，实际方法名ICRT。
- [BPP：What Enables In-Context Behavior Prompting for Manipulation?](../papers/related-methods/behavior-prompting-policy.md)；v1题名为Behavior Prompting Policy: Demonstrations as Prompts for Manipulation。
- [RoboTTT](../papers/related-methods/robottt.md)
- [WAM-TTT](../papers/related-methods/wam-ttt.md)
- [StellaVLA](../papers/related-methods/stellavla.md)
- [Zeva](../papers/related-methods/zeva.md)
- [HOST](../papers/related-methods/host.md)
