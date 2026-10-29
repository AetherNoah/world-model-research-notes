---
type: paper
title: "In-Context Imitation Learning via Next-Token Prediction"
short_name: ICRT
authors: [Letian Fu, Huang Huang, Gaurav Datta, Lawrence Yunliang Chen, William Chung-Ho Panitch, Fangchen Liu, Hui Li, Ken Goldberg]
year: 2024
venue: null
paper_url: "https://arxiv.org/abs/2408.15980v2"
code_url: null
project_url: "https://icrt.dev/"
source_urls:
  - "https://arxiv.org/html/2408.15980v2"
  - "https://icrt.dev/"
category: related-methods
series: robot-in-context-learning
tags: [robotics, in-context-learning, transformer, generalization]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "把同任务多条机器人传感运动轨迹串联为因果序列，仅监督prompt之后的动作，实现冻结策略的新任务条件化。"
---

# ICRT：用 next-token prediction 做上下文模仿

> **名称核对：**用户清单中的“ICIL”给出的完整题名对应 arXiv:2408.15980；论文方法名实际是 **In-Context Robot Transformer（ICRT）**。ICIL 是 in-context imitation learning 的范式简称，不另建一篇同名方法。依据2024-09-27的v2正文IV–VII与补充材料；未复现，venue暂不猜测。

## 一句话总结

把完整机器人示范放在序列前缀，让因果Transformer从当前观察预测动作；任务信息来自示范，而不是额外语言指令或奖励。

## 论文明确内容

### 数据形式与避免“忽略prompt”的捷径

轨迹包含图像、本体状态和动作 $(i_t,s_t,a_t)$；状态采用绝对末端位姿，动作含末端位姿增量及连续夹爪动作。训练按任务分组，将同任务多条示范拼接，再抽512步子序列；前面至少一条完整轨迹作为prompt。

仅依赖“一种场景只能做一种任务”的数据，模型可以忽略prompt。作者因此收集1,098条ICRT-MT轨迹、29个任务，使相同观察允许至少两种任务。DROID先抽10k轨迹，再按语言CLIP相似度0.9与每组至少4条过滤，最终约2k条用于预训练；这里语言用于**整理任务组**，不作为部署策略输入。

### 模型与监督

视觉ViT通过ImageNet/Open X-Embodiment混合数据预训练后冻结。多视角特征与状态经attention pooling成为state token，动作经MLP成为action token，交替送入Llama2式因果Transformer；默认是随机初始化的12层、768维模型，不是直接冻结一个大语言模型。

核心训练约束是只对prompt之后的动作计算L1损失。用概括记号表示：

$$
\mathcal L_{\rm act}=\frac{1}{|Q|}\sum_{t\in Q}\|\hat A_t-A_t\|_1,
$$

其中 $Q$ 是非prompt时间步、$A_t$ 是监督动作chunk；这是对第IV-B节的简写，非论文新增编号公式。作者预测未来16步动作并用temporal ensembling执行。部署权重固定，KV cache和观察/动作历史增长；“不更新权重”不代表上下文内存不增长。

### 任务与结果

Franka Emika真机上，pick-and-place与poke各设计6个未见任务，每任务5个难度级别，共每原语30次、总60次。每任务提前收集3条prompt候选，测试随机抽取；prompt布局与执行布局不同。Pick-and-place正确抓取可得0.5分、完成放置1分，因此表中汇总包含部分信用，不能全部解释为二值完整成功率。

| 来源 | 作者报告 | 解读 |
|---|---|---|
| Table I | pick-and-place65.0、poke93.3、平均79.2 | 反映任务/干扰物条件化能力，且不同原语计分不同 |
| Table II | 仅DROID为0，ICRT-MT-only平均73.3 | 场景内多任务歧义对学会读prompt很关键 |
| Table II | 加prompt loss降到22.5 | prompt本身的无条件模仿监督可能诱导忽略上下文 |
| 补充系统表 | ICRT39.6Hz，Llama2-7B版本10.7Hz | 大模型训练损失较低不保证闭环更好，推理频率是混杂因素 |

### 作者局限

第VII节不保证完全未见动作原语的泛化；默认机器人形态和阻抗控制器固定。同色异形干扰物会导致识别错误，且有限上下文约束长时间执行。结果不能直接当作任意人类视频到任意机器人迁移。

## 我的理解

“Next-token”在这里描述传感运动序列的预测结构，不要求把每个连续动作离散成语言token。数据分组和监督掩码的重要性至少与Transformer规模相当；让prompt成为必要条件，是获得ICL的前提。

## 推测与问题

- 数据任务分组靠语言相似度，错误分组是否会损害跨轨迹对应？
- rollout历史遇到重置/擦除等扰动时，旧历史可能与当前状态冲突；与短历史BPP需做匹配对照。
- 新物体、新布局、新动作原语是三种不同外推，报告时应分别列出。

## 阅读可信度检查

- [x] 标题与方法简称核实；公式明确标为自己的等价简写。
- [x] 数据过滤、计分和训练/测试更新边界核对原文。
- [x] 未声称纯视觉人类示范或跨形态适配，未复现。

## 关联笔记

- [机器人 ICL 系列](../../series/robot-in-context-learning.md)
- [BPP](behavior-prompting-policy.md)
- [RoboTTT](robottt.md)
