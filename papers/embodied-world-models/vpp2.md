---
type: paper
title: "Video Prediction Policy 2: Predict Better, Act Better"
short_name: VPP2
authors: [Yanjiang Guo, Haodong Yan, Zhide Zhong, Zhongru Zhang, Qingyuan Yang, Qingzhou Lu, Xiaoyu Chen, Yen-Jen Wang, Shuying Deng, Chenghan Yang, Puzhen Yuan, Chenxin Liu, Tun Ban, Xiang Zhu, Yichen Liu, Kun Feng, Haoang Li, Jianyu Chen]
year: 2026
venue: null
paper_url: "https://arxiv.org/abs/2610.10270v4"
code_url: "https://github.com/roboterax/video-prediction-policy-2"
project_url: "https://robert-gyj.github.io/video-prediction-policy-2/"
source_urls:
  - "https://arxiv.org/pdf/2610.10270v4"
  - "https://github.com/roboterax/video-prediction-policy-2"
category: embodied-world-models
series: null
tags: [world-action-model, embodied-world-model, robotics, video-prediction, diffusion, transformer, inverse-dynamics, generalization, real-time, evaluation]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "先用完整事件与细粒度caption训练可泛化的操作视频预测器，再蒸馏固定时长视觉规划并以MoT学习隐式逆动力学。"
---

# VPP2：先把操作未来预测对，再从未来生成动作

> **证据边界：**本笔记依据 arXiv 2610.10270v4（2026-10-26）的 §2–4、Figure 4、Table 1–3 和 Appendix C。这里只整理作者方法与结果，未运行代码。“零样本”指所述任务/配置的零样本部署，不是完全没见过机器人 embodiment；benchmark-specific post-training 结果与零样本实机实验分开。

## 一句话总结

VPP2 的核心不是另造一种 action head，而是先把视频预训练任务变得更接近 manipulation：明确描述一个完整事件、训练其未来演化，然后将这种能力压缩到适合控制的固定时长预测器，并尽量避免动作训练破坏它。

## 论文明确内容

### 为什么 event-level 而非直接短 chunk？

作者认为开放域视频生成先验不天然等于机器人策略先验：基础视频模型可能追求视觉合理性而忽略指令；一条简单指令对应许多短片段，容易形成歧义映射。§2 的数据处理因此：

1. 删除损坏、相机异常或突变的轨迹，将视频切成约 1–12 s 的语义事件。
2. 先用末端速度低谷、开合等启发式找边界，再由 VLM 调整。
3. caption 明确事件过程、哪个末端、哪个目标物体以及视角变化。
4. 要求末端和目标在首帧可见；存在相似物体时用属性/空间位置消歧。

“完整事件”使语言描述与完成后的视觉轨迹更一致；但不同事件时长不同，不能直接作为固定时间控制模型，因此还需要第二阶段。

### 三阶段学习链

§3 / Figure 4 从 Wan2.1-I2V-14B 出发：

| 阶段 | 学什么 | 与动作的关系 |
| --- | --- | --- |
| Event-level continued pretraining | 从首帧和详细指令预测完整操作事件；49帧、416×240 | 先获得可泛化的视频先验，还未依赖短动作标签 |
| Fixed-horizon post-training + consistency distillation | 人手预测2s、机器人预测8s的固定时长chunk，再压成单步视频生成 | 建立动作可对齐的物理时间尺度；部署视频输入变为17帧 |
| Action pretraining | 加入0.9B action DiT，通过MoT读取视频模型的预测表征/KV | 学习隐式inverse dynamics，生成约2s的短动作chunk |

早期动作训练冻结 video DiT 的 base weights，仅用 LoRA 适配视频主干，以减小随机初始化 action expert 的梯度冲击；不能把“保留视频先验”简写成整个训练期间视频网络完全不变。

### 关键目标与单步视觉规划

§3.1 Eq.2–3 以 $x_1$ 表示视频 latent，$\epsilon$ 为高斯噪声：

$$
x_s=(1-s)\epsilon+s x_1,\qquad
\mathcal L_{\rm FM}
=\mathbb E\|v_\theta(x_s,s,c)-(x_1-\epsilon)\|_2^2.
$$

$s=0$ 是噪声，$s=1$ 是数据；$c$ 含首帧和子任务描述。注意这与 [UniWAM](uniwam.md) 笔记的 source/data 下标方向不同，不能只凭下标统一符号。

固定chunk蒸馏在相邻flow时间上约束终点预测一致：

$$
\mathcal L_{\rm CD}=
\mathbb E\left[
\lambda(s,s')
\|F_\theta(x_s,s,c)-
\operatorname{stopgrad}(F_{\bar\theta}(\hat x_{s'},s',c))\|_2^2
\right].
$$

这里 $s<s'$，$\hat x_{s'}$ 由冻结teacher流积分得到，$\bar\theta$ 为EMA student。推理用 $F_\theta(\epsilon,0,c)$ 单次前向得到视觉latent。**单步说的是视觉分支**；动作分支仍可有多次去噪。

### 推理与高层规划

§3.2 报告在 bfloat16、torch.compile 条件下，视频约0.12s，五步动作去噪约0.1s，每chunk合计约0.22s。它不是完整 agent 的端到端延迟：VLM规划、通信和机器人执行并不都包含在这个数字里。

§3.3允许VLM根据观测与反馈把模糊/长指令分解成显式子任务，再交给VPP2执行。论文也有不使用子任务规划的结果；评测时必须标清是哪种配置。

### 视频预测评估

§4.1 随机收集人手、机器人各50组 image–instruction pairs，比较 Wan2.1、LVP、Cosmos3 两个规模、固定时长预训练消融与 VPP2。Table1 的 instruction-following rate 先由GPT式评测器判定；Figure6另有人类成对偏好判断，两者不是同一个指标。

| 模型 | Human | Robot |
| --- | --- | --- |
| Cosmos3-64B | 0.70 | 0.78 |
| VPP2-Fixed-Step | 0.48 | 0.42 |
| VPP2 | 0.80 | 0.90 |

两类均值从0.74到0.85对应摘要中11个百分点的差值。Fixed-Step消融支持event-level训练的价值，但数据、caption、训练日程共同影响结果，不能据此断言一种架构解决所有视觉泛化。

### 操作实验与比较口径

| 实验 | 设置 | 作者结果 |
| --- | --- | --- |
| 零样本实机 ALOHA，Figure8 | 在训练见过的embodiment上直接部署，无任务特定finetuning；10类任务；$\pi_{0.5}$和Fast-WAM按相同可见ALOHA数据训练 | 平均成功率58.5%，对照40.0%和20.5%；10类中9类最好 |
| LIBERO，Table2 / Appendix C | 仅四个标准suite训练；再测ID、Pro的position/task扰动、OOD的重组场景，不额外训练对应OOD任务 | ID98.8%，Pro45.0%，OOD63.9% |
| RoboDojo，Table3 | 42个双臂任务，按五个能力维度聚合；视频输入增加初帧及每秒采样的8个历史帧，单独后训练 | Score39.26、SR32.26%；GPT-6 Astra的榜单对照是28.97/22.48 |

RoboDojo 从**蒸馏前**Stage2 checkpoint开始适配，视频先训练10K steps，再联合训练video/action。不能直接把原始零样本checkpoint的效果与这个后训练分数画等号。Table3的baseline来自2026年9月官方榜单，不是作者在相同硬件上重跑全部系统。

Appendix C.2 的LIBERO-Pro每个suite–perturbation列100 episodes，共800；空间、物体、目标等细分结果见原表。这个45.0%只对应所选position/task协议，不能等同于另一些工作用的完整七维LIBERO-Plus成绩。

### 有无VLM规划的区别

Figure9针对五组选定RoboDojo任务、每任务50次：不加规划的均值27.6%，VLM子任务规划配置为57.6%。这是五任务子集均值，不是Table3的42任务整体成功率。为了训练子任务条件策略，示范还要切段并配子任务描述；它并非仅在原checkpoint上加一句提示词。

## 我的理解

它将“长程预测怎样学得好”和“短程动作怎样稳定输出”分成不同时间尺度：event-level扩大语义对齐，fixed-horizon提供一致时钟，短action chunk服务闭环控制。这比把视频模型直接当成更大的视觉encoder更有针对性。

MoT只是桥梁，视频先验是否被保留更取决于数据与训练顺序。该论文的启发是先建立受控、可泛化的中间任务，再谨慎地接上动作学习；但视觉预测质量与控制因果正确性仍不等价。

## 局限、推测与复现问题

- 论文没有单列完整局限章节；阅读边界是：zero-shot ALOHA仍属于见过的embodiment，不能推出零样本跨形态控制。
- 视频定量样本为人手/机器人各50组，且部分指标依赖模型裁判；宜核验评测prompt、评分稳定性和失败分类。
- **待验证：**caption详细程度、event切分和训练量分别贡献多少？Fixed-Step对照不能完全替代这三者的独立受控消融。
- **待验证：**给出视觉上合理但不可执行的未来时，动作expert会遵循错误未来还是纠正它？可与动作可控性评测连接。
- **复现优先级：**先锁定v4和checkpoint，区分视频单步、动作五步及RoboDojo历史适配；分别记录policy latency和外部VLM planner延迟。

## 阅读可信度检查

- [x] 三阶段、flow方向与视觉/动作去噪次数已核对 §3。
- [x] 数字已对照Table1–3/Figure8–9；区分了均值口径和任务子集。
- [x] 零样本实机、benchmark后训练及外部榜单对照已分开。
- [x] 本笔记未新增图片；方法图请见论文Figure4，第三方素材权利归原作者。
- [x] 未声称运行代码或独立验证作者结论。

## 关联笔记

- [UniWAM](uniwam.md)：统一三expert与异质数据监督，动作source初始化方式不同。
- [LingBot-VA 1.0](lingbot-va-v1.md)：视频/动作联合建模的邻近路线。
- [动作可控性](../../topics/action-controllability.md)：预测正确、指令服从和可执行性需分别检验。
