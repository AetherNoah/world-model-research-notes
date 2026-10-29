---
type: paper
title: "StellaVLA: In-Context Structured Demonstration for Generalizable Vision-Language-Action Models"
short_name: StellaVLA
authors: [Siyu Xu, Yunke Wang, Zijian Wang, Dihao Zhu, Chenghao Xia, Chengbin Du, Daochang Liu, Tao Huang, Chang Xu]
year: 2026
venue: null
paper_url: "https://arxiv.org/abs/2608.11671v1"
code_url: null
project_url: "https://stelledge.com/blog/stellavla"
source_urls:
  - "https://arxiv.org/html/2608.11671v1"
  - "https://arxiv.org/abs/2608.11671v1"
category: related-methods
series: robot-in-context-learning
tags: [robotics, in-context-learning, transformer, generalization, evaluation]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "将检索示范转为子目标及2D/3D运动语言，用并行动作/语言训练和缓存前缀实现不更新权重的任务适配。"
---

# StellaVLA：把示范变成可检索的结构化上下文

> **证据边界：**依据 arXiv v1 第 3–4 节与附录 A–D；作者列表采用 arXiv 元数据，正文另有团队署名。实验与消融未复现。“rationale”是自动产生的解释性标签，不证明找回了专家真实思维。

## 一句话总结

StellaVLA 把长而具身相关的轨迹压成任务计划、子目标、关键帧、2D 路径和 3D 运动描述，检索其中一条作为固定前缀，指导冻结部署策略。

## 论文明确内容

### 离线结构化与检索

第 3.1 节用 VLM 从轨迹的视觉变化划分语义片段；轨迹位移通过确定性 verbaliser 转成 3D 文本，再利用相机标定投影为 2D 运动描述。当前帧还带有对应子任务标签。训练时 leave-one-out，按任务指令的语言 embedding 余弦相似度，从排除目标轨迹的池里检索 top-1 示范。

这不是只把一段自然人视频输入模型：机器人示范有可用动作，跨具身的人手与 XR 记录也必须经过相同结构化接口。正文通过人手/XR 演示说明上下文可跨来源，但可执行动作监督仍来自真实机器人。

### 并行监督、非自回归控制

第 3.2 节共享 Qwen3-VL-4B backbone，动作 expert 是 MLP，预测连续 action chunk；语言 expert 并行预测当前子任务及相同动作 chunk 的 2D/3D 语言表示：

$$
\mathcal L=\mathcal L_{\rm act}+\lambda\mathcal L_{\rm lang},\quad\lambda=0.3.
$$

动作头不以当步生成的文字为条件。部署移除语言解码路径，模型权重不更新；示范前缀一条 rollout 内不变，KV cache 只计算一次。这消除语言逐 token 解码开销，但不是所有额外计算都消失。

### 实验设置与证据

| 来源 | 结果/协议 | 需要保留的含义 |
|---|---|---|
| Table 1 LIBERO | 每 suite 500 rollout；平均 98.8% | 多数外部 baseline 来自原论文，只有 StarVLA-OFT 是同 backbone/data 的受控对照 |
| Table 2 VLA-Arena | L0/L1/L2 平均 0.84/0.62/0.43；overall 0.63 | L1/L2 Long Horizon 几乎仍为零，固定前缀不负责重新规划 |
| Table 3 LIBERO-Plus | 扰动任务加权平均 85.1% | 排除 Orig.，不是所有列的算术平均 |
| Table 5 同 checkpoint 换示范 | 正确 98.8、无示范 62.4、错误 44.9 | 支持模型在使用任务条件，也说明错误检索会伤害控制 |
| Table 8 延迟 | 无示范64ms；未缓存183ms；缓存91ms | 实机含观察/控制开销约205ms/chunk，不能称严格“零延迟” |

真机为 AgileX Piper，125 条 teleoperation episodes；检索池还含 26 段人手 XR 记录和 26 条重定向轨迹。四个 ID 任务、两个 OOD-L1 扰动和一个 OOD-L2 新抽屉任务，每格 10 次。ID 85%、L1 75%，但所有方法都未完整完成 OOD-L2；部分进度不等于新任务成功。

### 消融和局限

固定 checkpoint 的 text-only 示范在 LIBERO/Plus 达到 98.8/84.4，接近图文的 98.8/85.1；说明相当多收益来自结构化程序信息，而非复制像素轨迹。附录与实验同时限定：标注/标定质量、检索池覆盖、固定示范无法纠正持续漂移、真实跨来源结果不能被单步 action-disagreement 指标替代。

## 我的理解

它把推理成本主要前移到离线标注和训练，部署读取一个已有的程序描述。应区分“模型通过语言辅助监督学到了更好的动作表示”和“测试时实时运行 CoT”——这里主要是前者。

## 推测与问题

- 只按指令语义检索会否忽略障碍、接触和对象状态？
- 结构化文本有运动单位、参考坐标和标定依赖，跨机器人是否需要显式重映射？
- 应在不含测试轨迹的示范池内评估，并单列“同任务有示范”和“完全未知原语”。

## 阅读可信度检查

- [x] 核对完整方法、训练/部署路径和四张关键表。
- [x] 区分模型延迟、系统延迟、单步一致性与闭环成功。
- [x] 保留真机小样本与 OOD-L2 失败边界，未声称复现。

## 关联笔记

- [机器人 ICL 系列](../../series/robot-in-context-learning.md)
- [BPP](behavior-prompting-policy.md)
- [ICRT](icrt.md)
