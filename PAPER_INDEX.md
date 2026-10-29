# Paper Index

> 此文件由 `python scripts/kb.py build-index` 根据论文 YAML 自动生成，请勿手工编辑。

当前共有 **27** 篇论文笔记。

## Model-based Reinforcement Learning

| Paper | Year | Venue | Main idea | Status | Confidence |
|---|---:|---|---|---|---|
| [DreamerV1](papers/model-based-rl/dreamer/dreamer-v1.md) | 2019 | ICLR 2020 | 在学习到的 latent dynamics 中展开想象轨迹，并直接训练 actor 与 critic。 | `finished` | medium |
| [DreamerV2](papers/model-based-rl/dreamer/dreamer-v2.md) | 2020 | ICLR 2021 | 用 categorical latent、KL balancing 与混合梯度估计把 Dreamer 推进到 Atari。 | `finished` | medium |
| [DreamerV3](papers/model-based-rl/dreamer/dreamer-v3.md) | 2023 | — | 以 symlog、twohot、free bits 和 return normalization 统一跨领域训练尺度。 | `finished` | medium |
| [SuperMimic](papers/model-based-rl/supermimic.md) | 2026 | CoRL 2026 | 用共享、可微的动力学替代模型跨动作库摊薄建模成本，再在短期想象中反向传播训练全身控制器。 | `skimmed` | medium |

## Generative & Interactive World Models

| Paper | Year | Venue | Main idea | Status | Confidence |
|---|---:|---|---|---|---|
| [Genie](papers/generative-world-models/genie.md) | 2024 | — | 从无动作标签视频中学习离散 latent action，并据此逐帧生成可交互环境。 | `finished` | medium |
| [LingBot-World 1.0](papers/generative-world-models/lingbot-world-v1.md) | 2026 | — | 把开放域视频生成器通过长视频训练、动作适配、因果改造和蒸馏变成实时可控世界模型。 | `finished` | medium |
| [LingBot-World 2.0](papers/generative-world-models/lingbot-world-v2.md) | 2026 | — | 以原生 causal teacher、MoBA、少步蒸馏和 Director–Pilot 接口提升长时生成与交互能力。 | `finished` | medium |

## Embodied World Models

| Paper | Year | Venue | Main idea | Status | Confidence |
|---|---:|---|---|---|---|
| [LingBot-VA 1.0](papers/embodied-world-models/lingbot-va-v1.md) | 2026 | — | 先预测任务期望的未来视觉 latent，再以 inverse dynamics 解出动作，并用真实观察持续校正。 | `finished` | medium |
| [LingBot-VA 2.0](papers/embodied-world-models/lingbot-va-v2.md) | 2026 | — | 围绕语义 visual-action 表示、原生 causal 预训练、MCP 与闭环部署重做机器人世界模型。 | `finished` | medium |
| [UniWAM](papers/embodied-world-models/uniwam.md) | 2026 | — | 以MoT联合物理语言推理、未来视觉生成和动作预测，用多源监督对齐三者，并以历史动作初始化和未来噪声增强改善控制。 | `skimmed` | medium |
| [VPP2](papers/embodied-world-models/vpp2.md) | 2026 | — | 先用完整事件与细粒度caption训练可泛化的操作视频预测器，再蒸馏固定时长视觉规划并以MoT学习隐式逆动力学。 | `skimmed` | medium |

## Spatial World Models

| Paper | Year | Venue | Main idea | Status | Confidence |
|---|---:|---|---|---|---|
| [LingBot-Map](papers/spatial-world-models/lingbot-map.md) | 2026 | — | 用 anchor、局部窗口和压缩 trajectory memory 分工保存流式三维重建所需的历史。 | `finished` | medium |

## Surveys

| Paper | Year | Venue | Main idea | Status | Confidence |
|---|---:|---|---|---|---|
| [TCA Survey](papers/surveys/temporal-credit-assignment-survey.md) | 2023 | — | 用深度、密度和广度刻画时序信用分配，并统一比较时间邻近与回报分解方法。 | `finished` | medium |

## Related Methods

| Paper | Year | Venue | Main idea | Status | Confidence |
|---|---:|---|---|---|---|
| [ICRT](papers/related-methods/icrt.md) | 2024 | — | 把同任务多条机器人传感运动轨迹串联为因果序列，仅监督prompt之后的动作，实现冻结策略的新任务条件化。 | `skimmed` | medium |
| [CoCA](papers/related-methods/step-level-reward-for-free.md) | 2025 | — | 用相邻去噪状态对最终图像的相似度变化，为 diffusion trajectory 分配逐步训练权重。 | `revisit` | medium |
| [An Unexpected Robot Policy](papers/related-methods/unexpected-robot-policy.md) | 2026 | — | 在不接入学习型运动策略的条件下评估通用LLM直接输出EEF动作，并发现语义泛化、精细控制和示范ICL之间的明显能力差异。 | `skimmed` | medium |
| [Astra as Embodied Policies](papers/related-methods/astra-embodied-policies.md) | 2026 | — | 跨六类具身任务分解Astra的直接控制、策略协作和反馈适应能力，强调完整控制路径、试验预算和推理延迟决定结论边界。 | `skimmed` | medium |
| [Astranomy](papers/related-methods/astranomy.md) | 2026 | NVIDIA technical report | 固定通用模型，通过动作接口、历史和执行harness的系统消融，区分native policy、agentic policy与WAM协作的收益和成本。 | `skimmed` | medium |
| [BPP](papers/related-methods/behavior-prompting-policy.md) | 2026 | — | 以含观察、状态和动作的示范作prompt，用分块注意力与扩散动作头研究任务多样性如何形成推理时行为条件化。 | `skimmed` | medium |
| [Harness VLA](papers/related-methods/harness-vla.md) | 2026 | — | 将冻结VLA封装成可重试的局部接触技能，用固定解析primitive、任务记忆和全局失败知识完成语义重绑定、搬运和恢复。 | `skimmed` | medium |
| [HOST](papers/related-methods/host.md) | 2026 | — | 以共享任务进度对齐人类视频与机器人轨迹，再级联预测进度、机器人未来观察和动作，实现无任务专属更新的单视频技能条件化。 | `skimmed` | medium |
| [RoboBRIDGE](papers/related-methods/robobridge.md) | 2026 | IROS 2026 | 用监测、感知、规划、控制和机器人接口五模块给预训练策略补上失败诊断与分层恢复，并通过primitive-specific LoRA缓解域变化。 | `skimmed` | medium |
| [Robot-Use Agents](papers/related-methods/robot-use-agents.md) | 2026 | Personal essay | 从工具使用视角讨论通用云端agent控制机器人的可能传播路径，同时承认时延、可靠性和物理风险尚未解决。 | `skimmed` | medium |
| [RoboTTT](papers/related-methods/robottt.md) | 2026 | — | 把长时序机器人经验压入可在线梯度更新的fast weights，以sequence action forcing和TBPTT支持8K步上下文。 | `skimmed` | medium |
| [StellaVLA](papers/related-methods/stellavla.md) | 2026 | — | 将检索示范转为子目标及2D/3D运动语言，用并行动作/语言训练和缓存前缀实现不更新权重的任务适配。 | `skimmed` | medium |
| [WAM-TTT](papers/related-methods/wam-ttt.md) | 2026 | — | 以人类视频预测和键值重建更新WAM视频侧fast weights，在不改基础动作专家的情况下调整机器人行为。 | `skimmed` | medium |
| [Zeva](papers/related-methods/zeva.md) | 2026 | — | 将机器人自己的动作及其视觉效果编码为双时间尺度记忆，用上下文改变冻结策略的后续行为。 | `skimmed` | medium |
