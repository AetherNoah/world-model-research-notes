---
type: paper
title: "WAM-TTT: Steering World-Action Models by Watching Human Play at Test Time"
short_name: WAM-TTT
authors: [Yusen Feng, Bingchen Han, Jiangran Lyu, Kai Liu, Yixin Zheng, Yuxuan Wan, Weiheng Liu, Sun Han, Ruiqin Li, Yulong Zhang, Fangfu Liu, Xuesong Shi, Libin Liu, Yizhou Wang, Zhizheng Zhang, He Wang]
year: 2026
venue: null
paper_url: "https://arxiv.org/abs/2607.06988v3"
code_url: null
project_url: "https://wam-ttt.github.io/"
source_urls:
  - "https://arxiv.org/html/2607.06988v3"
  - "https://wam-ttt.github.io/"
category: related-methods
series: robot-in-context-learning
tags: [world-action-model, robotics, test-time-training, in-context-learning, video-prediction, generalization]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "以人类视频预测和键值重建更新WAM视频侧fast weights，在不改基础动作专家的情况下调整机器人行为。"
---

# WAM-TTT：把人类视频写入世界—动作模型的快权重

> **证据边界：**依据 2026-10-23 的 arXiv v3，第 3–5 节及附录 B/C/E；未复现。文中主指标是分阶段 **progress**，不是二值任务成功率。

## 一句话总结

先用配对 human–robot 数据学习一个可迁移的记忆写入接口，再在部署前用目标场景的人类 RGB 视频更新视频侧 fast weights，借助 WAM 内已有的视频—动作耦合改变控制。

## 论文明确内容

### 架构与两层适配

基础模型为 LDA，DiT 含视频和动作专家，通过 joint attention 交换信息。作者只给视频专家加 TTT 残差：视频 token 变为原输出加 $\Delta z_{\rm TTT}$；动作 token 没有直接的 TTT 残差，但会经后续联合注意力受到影响。

人类 token 生成 Key/Value，机器人 token 生成 Query。元训练通过同任务配对片段和归一化任务阶段建立人类写入、机器人读取的接口。fast weights 的初始化和 Q/K/V/O 投影属于 slow parameters；离线外层训练会同时优化 WAM 与这些参数。

### 测试时到底更新什么

式 (3)–(8) 定义自监督视频预测与键值重建的联合目标：

$$
\mathcal L_{\rm adapt}=\mathcal L_{\rm vg}^{\rm human}+\lambda\sum_\ell\mathcal L_{\rm KVM}^{(\ell)},\qquad
\mathcal L_{\rm KVM}^{(\ell)}=\frac{\|f_{W^{(\ell)}}(K_h^{(\ell)})-V_h^{(\ell)}\|_2^2}{BL_hd}.
$$

部署阶段只对 $W$ 做内层 SGD；WAM、动作专家、slow projections 和初始化参数全部冻结。输入是一小批 action-free 人类第一人称视频，不需要测试视频的机器人动作标签。适配结束后的 $W_N$ **在机器人 rollout 中保持固定**（式 (9)）；这与 RoboTTT 随执行每步继续更新记忆不同。附录 B 给出的内层步数为 1，不能想象为长时间在线微调整个网络。

### 数据、评估协议与结果

第 4.1 节：2,286 对人类—机器人 episodes；Unitree G1、Galbot 两指双臂、Galbot Sharpa 灵巧双臂共 9 个任务。每个任务分配给一种 embodiment，Orig. 在训练采集隔间评估，New 同时改变家庭环境、光照、桌高和物体。主表每个 task/setting 25 次，按预设子目标给予部分分数。

| v3 来源 | 结果 | 边界 |
|---|---|---|
| Table 1 New 平均 progress | WAM-TTT 46.2，LDA 32.5，WAM-Cotrain 25.3，WAM-ICL 7.1 | LDA 到 WAM-TTT 为 +13.7 个 progress 点，不能改写为成功率 |
| Appendix C Orig. | 61.1 | New 为 46.2，仍存在明显域移损失 |
| Table 3 Table Bussing | 完整方法 100，去元训练 9.0，去记忆重建 66.7 | 每格仅 10 次，支持该接口的重要性，不代表各成分独立可加 |
| Table 1 Stamp Paper | WAM-TTT 8.3，LDA 33.3 | 并非每项任务都提升，精确几何对齐仍会失败 |

作者在同一 WAM 内比较 LingBot-style、XSkill-style 等接口；它们是受控接口变体，不能当作对这些原始完整系统的直接排名。

### 作者局限

第 5 节指出配对阶段分布不一致会破坏元训练信号；fast network 与固定 slow projections 限制适配范围；RGB 接口没有利用手姿、接触和 3D 线索。文中尚未系统量化任务偏离元训练分布的失效边界。

## 我的理解

WAM 的视频损失提供了“没有动作标签也能更新”的入口，但这不自动保证视觉改善会带来正确控制。本文通过 human–robot 元训练把这个入口对齐成任务相关的控制接口；因此“无动作测试视频”不等于“训练阶段也不需要配对数据”。

## 推测与问题

- 对不同技能、不同相机或不完整示范，归一化阶段配对是否依然有效？
- 应分别测适配耗时、rollout 控制延迟和基础技能保留，避免把冻结 backbone 等同于无部署成本。
- WAM-ICL 的低分可能受基础模型未充分学会视频条件化影响，不能概括为“ICL 天然无效”。

## 阅读可信度检查

- [x] 核对 v3 的更新边界、适配后固定规则与 progress 协议。
- [x] 数字定位 Table 1/3、Appendix B/C。
- [x] 未声称复现或通用跨 embodiment 零样本迁移。

## 关联笔记

- [机器人 ICL 系列](../../series/robot-in-context-learning.md)
- [RoboTTT](robottt.md)
- [HOST](host.md)
