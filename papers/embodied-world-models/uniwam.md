---
type: paper
title: "UniWAM: Unified World-Action Model"
short_name: UniWAM
authors: [Wenxuan Song, Jiayi Chen, Jingbo Wang, Shuai Zhou, Xicheng Gong, Zehua Fan, Ziyang Zhou, Junwu E, Haodong Yan, Fuhao Li, Qize Yu, Xu Huang, Pengwei Wang, Wen Chen, Shunbo Zhou, Haoang Li]
year: 2026
venue: null
paper_url: "https://arxiv.org/abs/2610.02054v4"
code_url: "https://github.com/UniWAM/UniWAM"
project_url: "https://uniwam.github.io/"
source_urls:
  - "https://arxiv.org/pdf/2610.02054v4"
  - "https://github.com/UniWAM/UniWAM"
category: embodied-world-models
series: null
tags: [world-action-model, embodied-world-model, robotics, embodied-ai, video-prediction, transformer, generalization, evaluation]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "以MoT联合物理语言推理、未来视觉生成和动作预测，用多源监督对齐三者，并以历史动作初始化和未来噪声增强改善控制。"
---

# UniWAM：让语义理解、视觉未来与动作预测共同训练

> **证据边界：**本笔记对应 2610.02054v4（2026-10-14），已阅读 §3–5 与评测/实现附录；实验均为作者报告，未复现。它不是同名的 2609.39388《UniWAM Technical Report: Unified Mobile Manipulation…》，后者的 MAP-Data、MAP-Bench 与移动操作指标不属于本文。作者顺序按 arXiv v4 元数据，GitHub 引用和早期版本可能不同。

## 一句话总结

将 VLM 的语义先验、视频模型的时空先验和连续动作 expert 放入可交换信息的统一模型，再用不同来源的数据监督适合的模块；重点不只是拼接网络，而是避免单一动作训练损坏已有能力。

## 论文明确内容

### 任务与三个 expert

§4 / Figure 3 将模型写为条件联合分布：

$$
p_\theta(y,o_{t+1:t+h},a_{t+1:t+h}\mid o_t,s_t,I).
$$

$I$ 为任务指令，$o_t$ 为当前观测，$s_t$ 为机器人本体状态，$y$ 是物理语言描述或 VQA 答案。三个 expert 分别负责：

| 模块 | 初始化/功能 | 监督 |
| --- | --- | --- |
| Physical reasoner | Qwen3-VL-2B-Instruct；保留自回归语言输出能力 | VQA、物理动作语言描述 |
| World generator | Wan2.2-TI2V-5B；冻结的视频 autoencoder 提供 latent 空间 | 未来视觉 flow matching |
| Action predictor | 将状态 token 和带位置的动作 chunk 编码 | 连续动作 flow matching |

每层先做 expert 自己的归一化和投影，再把三种 modality 的 Q/K/V 在 token 维拼接，做带 mask 的联合 attention，最后分回 expert 的 residual/FFN。它不是 VLM 输出一个计划后完全独立运行 WAM 的串行系统。

Appendix G 的配置使用 14 维状态/动作、长度 16 的动作 chunk、8 个未来帧，视觉分辨率 384×320；这些是具体模型配置，不应泛化成架构定义。

### 物理语言与多源数据

低层动作被转换为有固定坐标约定的自然语言运动描述，使 human/robot 数据都有共同的局部运动语义。语言 head 的目标是答案 token 上的自回归交叉熵，而不是“把全部连续动作离散成普通词”：

$$
\mathcal L_{\rm lang}=-\sum_j\log p_\theta(y_j\mid o_t,I,y_{<j}).
$$

机器人示范、人类第一视角视频和 VQA 提供互补监督。人类视频不是未经处理直接当作机器人 action 标签；论文另有清理、动作/场景标注管线。VQA 用于保留已有语言理解能力，human 数据用于扩展具身视觉和物理语义覆盖。

**数据口径待核对：**v4 Table 1 的 robot subtotal 为约 4,958 h，human subtotal 约 5,072 h，但表中 total 写约 10,013 h，与两项相加的 10,030 h 不同；§3.1 又写 selected robot data 为 4,363 h。Figure 7 最后一点为 4,958 h robot + 5,072 h human。本笔记保留这些出处差异，不将任一数直接写成已核实的最终训练规模。

### Flow matching、未来视觉噪声与历史初始化

§4.2 令 $A_t^0$ 为干净未来动作，$A_t^1$ 为 source，则

$$
A_t^\tau=(1-\tau)A_t^0+\tau A_t^1,\qquad
u_t^a=A_t^1-A_t^0.
$$

视频分支使用同类插值和平方误差 flow matching。这里 $\tau=1$ 指 source、$\tau=0$ 指 data，生成方向是从 1 到 0。当前观测 latent 始终干净，不计入未来视觉损失；action/video noise level 独立采样。Appendix G Eq.11 的预训练权重为：

$$
\mathcal L_{\rm pretrain}
=\mathcal L_{\rm video}+\mathcal L_{\rm action}
+0.1\mathcal L_{\rm language}.
$$

后训练有两个关键改动：

1. **Future-frame noise augmentation。**以 0.5 概率进一步扰动未来 latent，保留系数 $s_{\rm aug}\sim U(0.5,1)$；保持当前观测、原 flow target 与 timestep embedding 不变。目标是减少 action expert 对“过于干净的未来”依赖，而不是禁止视频与动作交流。
2. **History-conditioned flow matching。**用最近已经执行的动作 chunk 加小噪声初始化 source：

$$
A_t^1=A_{t-1}+\epsilon_a,\qquad
\epsilon_a\sim\mathcal N(0,0.02^2I).
$$

历史 chunk 与目标 chunk 同维，视觉 source 仍为高斯噪声。它学习历史到未来的位移场，而非仅在 conditioning 中附加历史。Appendix C 展示 A2A 在 2–4 步下优于 N2A 的十步变体，作者默认用四步 A2A；这并不自动给出完整系统的墙钟延迟优势。

### 实验协议与主要结果

| 设置 | 作者报告 | 读数边界 |
| --- | --- | --- |
| 标准 LIBERO，Table 4 | 均值 99.2% | ID 成绩；不能单独证明开放域泛化 |
| RoboTwin 2.0，Table 5 | C2C 75.14%，C2R 68.32%，两者均值 71.73% | 50 个任务，各 50 条 clean demonstrations；C2C、C2R 各按 100 episodes 评估；C2R领先列出方法，但C2C并非最高，OpenWAM-α为89.4% |
| LIBERO-Plus，Table 6 | 总体 92.6%；覆盖 camera/robot/language/light/background/noise/layout 七类扰动 | 同表 camera 92.0%和layout84.6%并非各维最高，不应写“所有扰动维度第一” |
| 实机四任务，Figure 6 | 平均 SR 67.5%，IFR 82.5% | AgileX Piper双臂，每任务200条示范；所有比较方法按同一协议训练80K steps，8×H100 |
| 长任务 tidy-up，Figure 6(c) | Task Progress 5.0，$\pi_{0.5}$ 4.8，Motus 3.2 | 0–6 的六阶段部分进度分，绝不是83.3%完成率 |

IFR 衡量是否把指令关联到正确物体和交互，SR 衡量最终完整成功。选对物体后抓取失败可能 IFR 对、SR 错，这两个分数不能互换。

### 消融如何支持结论

§5.4 / Figure 9 的多源数据比较中，robot+human 的 C2C/C2R 为 75.12/67.56，加入 VQA 后为 75.14/68.32：VQA 在已有 human+robot 混合上的增益较小，不能将整个收益全归给 VQA。另一路逐步消融显示历史动作初始化将平均 SR 从 64.80 提到 69.73，未来视觉增强进一步到 71.34。

§5.3 从约 250 到 5,000 h robot-data 做 scaling sweep：小规模时 human 数据可能有害，超过约 1,000 h 后收益才显现。文中给出特定区间的 log-linear loss 拟合，但该拟合不是任意数据规模下的普适定律。

## 我的理解

UniWAM 在“生成未来”之外保留可训练的语言 expert，使动作可利用语义目标和视觉时序两种结构。另一方面，增强故意让未来信息变差，说明控制器需要的是**对预测误差鲁棒的表征**，而不只是最好看的视频。

历史初始化利用了控制的时间连续性；它的收益与 TTT/ICL 不同：这里没有在每个新任务上在线更新 fast weights，也不需要一条测试示范作为 prompt。是否能处理动作突变，应独立于平均平滑任务表现衡量。

## 局限与待核对问题

- 论文未单列完整 limitation 章节；以下是阅读边界：四项实机任务和一个进度任务不足以覆盖一般性具身能力，统计置信区间与跨随机种子的稳定性仍需检查。
- 模型学习联合相关结构不等于识别因果机制；t-SNE 中 human/robot 重叠也不能独立证明因果式跨 embodiment 迁移。
- 数据小时口径不一致需要作者澄清。官方代码 README 的训练耗时/硬件与 v4 Appendix G 也可能不同；本笔记以固定 PDF 版本为主，不混用配置。
- **待验证：**历史动作接近静止或上一动作错误时，A2A source 会不会产生惯性偏置？建议按转向、停止、重抓等子集拆解，而非仅平均成功率。

## 阅读可信度检查

- [x] 三 expert、训练目标和历史source公式已核对 §4；明确了flow方向。
- [x] ID/OOD、SR/IFR、Task Progress及C2C/C2R已分开。
- [x] 主要数字对照 Table4–6、Figure6/9；内部数据口径冲突已记录。
- [x] 未声称运行代码、重现实验或验证普适scaling law。

## 关联笔记

- [VPP2](vpp2.md)：先保护视频泛化，再引入动作模块的另一种训练路线。
- [LingBot-VA 1.0](lingbot-va-v1.md)：视频与动作联合建模。
- [动作条件](../../topics/action-conditioning.md)：未来预测与动作控制的关系。
