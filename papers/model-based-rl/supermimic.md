---
type: paper
title: "SuperMimic: Scaling Humanoid Whole-Body Control with One Differentiable World Model"
short_name: SuperMimic
authors: [Qiayuan Liao, Yiyang Shao, Junfeng Long, Kevin Zakka, Roberto Horowitz, Pieter Abbeel, Koushil Sreenath]
year: 2026
venue: CoRL 2026
paper_url: "https://arxiv.org/abs/2610.28480v1"
code_url: null
project_url: "https://supermimic.github.io/"
source_urls:
  - "https://arxiv.org/pdf/2610.28480v1"
  - "https://supermimic.github.io/"
category: model-based-rl
series: null
tags: [world-model, model-based-rl, robotics, humanoid-control, differentiable-dynamics, latent-imagination, actor-critic, reinforcement-learning, evaluation]
status: skimmed
confidence: medium
read_date: 2026-10-29
updated: 2026-10-29
main_idea: "用共享、可微的动力学替代模型跨动作库摊薄建模成本，再在短期想象中反向传播训练全身控制器。"
---

# SuperMimic：一套动力学模型，许多全身动作

> **证据边界：**已阅读 arXiv v1 的方法、主实验及附录 A–F 中与训练和评测有关的内容；下文数字为作者报告，未独立复现。CoRL 2026 身份来自论文首页及 arXiv Comments，Oral 标注来自 arXiv。基准中的“零失败”指训练动作覆盖，不是新动作泛化，也不是实机零失败。

## 一句话总结

不同参考动作都作用于同一个机器人身体。与其让 model-free RL 为每种动作反复估计后果，不如学习一次共享身体与接触动力学，并通过它给策略提供低方差的解析梯度。

## 论文信息与阅读定位

- 版本：2610.28480v1，2026-10-20；本笔记按此版本记录。
- 任务：Unitree G1 的参考动作跟踪，附带初步速度指令接口；不是视觉生成模型或人类视频到机器人动作的端到端模型。
- 方法主体见 §3、Figure 2；计算账本见 §4.1；失败判定与训练集边界见 Appendix F。

## 论文明确内容

### 问题与训练循环

论文采用 Dyna 式闭环，而非先固定世界模型、再永久离线优化策略：

1. 当前策略在 MuJoCo Warp 中采样，将执行的动作、状态、奖励与随机化上下文放入有限 replay buffer。
2. 从 buffer 抽取短窗口，更新世界模型。
3. 固定该次更新中的模型参数，从 replay 状态出发想象，更新策略与 critic。
4. 返回真实模拟器，收集改进后策略产生的新状态分布。

采样时可加入动作噪声以扩宽模型训练覆盖；记录的是实际执行的含噪动作。模型条件包含每个 episode 的模拟器上下文 $\xi$，如摩擦、校准偏移和躯干质心；传感噪声在模型外采样。因此，随机化环境并不要求模型把所有未知性都混成一个随机状态。

### 世界模型与训练目标

模型包含状态转移、奖励、无噪观测和终止预测；可解析计算的 head 不另加学习损失。设学习参数为 $\omega$，以真实 buffer 状态初始化，在记录动作下自回归展开 $k=8$ 步。§3.2 的 Eq.1 为：

$$
\mathcal L_{\mathrm{WM}}(\omega)=
\mathbb E_{\mathcal B}\sum_{j=1}^{k}
\left[
\|\hat s_j-s_j\|^2+
(\hat r_j-r_j)^2+
\|\hat{\bar o}_j-\bar o_j\|^2+
\operatorname{BCE}(\hat d_j,d_j)
\right].
$$

这里的“可微”是**学习到的替代动力学可微**，并不要求对非光滑接触的原模拟器求导。模型的预测 Jacobian 是有误差的；解析梯度也不等于真实环境的无偏梯度。

### 短视界 actor–critic

策略训练用 $H=32$ 步想象，长于模型拟合窗口。终止概率通过软生存权重进入目标：

$$
\hat c_0=1,\qquad
\hat c_{t+1}=\hat c_t(1-\hat d_{t+1}),
$$
$$
\hat J(\theta)=\mathbb E\left[
\sum_{t=0}^{H-1}\gamma^t\hat c_t\hat r_{t+1}
+\gamma^H\hat c_H V_\psi(\hat s_H,g_H,\xi)
\right].
$$

这是 §3.3 Eq.2 的写法。BPTT 沿着状态、动作与模型的 Jacobian 把奖励梯度传回策略；critic 用想象轨迹上的 GAE/$\lambda$-return 拟合视界以外的回报。每次 actor 更新中模型权重冻结，但状态与动作上的计算图保留。目标中的 $g_t$ 是参考动作或其他目标接口。跟踪奖励沿用 BeyondMimic，并移除了其 self-collision penalty；奖励差异也是复现时需要锁定的设置。

critic 的作用不只是提高静态分数。去掉它仍可能得到接近的正常跟踪回报，但策略会更短视；被推扰后，恢复收益落在短想象窗以外时，bootstrap 的差异更明显。

### 实验设置与结果

| 证据 | 协议与结果 | 解释边界 |
| --- | --- | --- |
| 与 SONIC 比较，§4.1 / Appendix F | 129,514 条过滤后的 BONES-SEED 训练动作；1 张 H100 训练 3 h 后失败 25 条，SONIC released checkpoint 失败 179 条；2 张 H100 训练 9.8 h 后为 0 | 同一模拟环境、确定性物理、无观测噪声或 domain randomization；按 SuperMimic 终止规则判失败，不能当作未见动作测试 |
| 计算预算 | SONIC 报告 128 张 L40、168 h；SuperMimic 的 3 h 包含完整训练循环，不含训练后的 checkpoint 评测 | 数据分别约 611 h 与 264.56 h，GPU 型号也不同；GPU-hours 不是统一硬件 FLOPs |
| 受控 PPO-Sim 对照，Figure 4 | 相同 LAFAN1 40 段、2.45 h 动作、观测、奖励、随机化和网络规模；SuperMimic 1 GPU 在 1.2 h 达到评测归一化回报 0.82；PPO-Sim 在 18 h 预算内未达到 | 比跨论文预算更接近方法层面的受控证据；训练回报与整段 checkpoint 评测回报仍需区分 |
| 动作库 scaling，Figure 3 | 数据从 1 h 增至约 264 h 时，8 步归一化状态预测误差约保持在 0.007–0.011 | 在线策略访问分布中的短期误差，不是任意初始状态、长时 rollout 的全面保证 |

Figure 5 还将“快的模型 rollout”和“更高效的梯度估计”分开：模型 rollout 在所测 batch 范围内快于模拟器，解析梯度在匹配回报下比世界模型中的 PPO 少用约 64 倍想象步数。不能把所有收益仅归结为模型推理快。

实机展示覆盖多种 G1 动作，但大规模失败表来自模拟器。附录的 benchmark 明确写明这些是双方训练动作，因此更合适的指标名是**训练动作覆盖率**。

### 作者明确列出的局限

§6 列出三项：多接口训练尚缺充分研究；动力学仅来自模拟器，未用实机数据修正；当前训练限于平地，不包含复杂地形或物体交互。速度 encoder 实验也只是初步接口研究，其速度任务奖励不更新共享 decoder，不能据此推断完全联合多任务训练已经解决。

## 我的理解

它与像素级 world model 最不同之处是建模对象：这里有可从模拟器读取的状态、可设计的密集奖励和已知随机化参数，因此模型只需给控制提供足够好的局部预测与导数。把这个结论直接外推到开放世界视频模型会漏掉可观测性和奖励定义的难度。

另一个关键取舍是“用短视界限制模型偏差，再用 critic 连接长期回报”。这与 Dreamer 共享“模型内优化控制”的思想，但状态表示、动力学监督和策略梯度路径都不相同。

## 推测与问题

- **待验证：**模型的值预测误差不大时，Jacobian 仍可能错得足以误导 actor；能否直接衡量梯度方向与真实模拟器中的策略改善一致性？
- **待验证：**平地训练中共享动力学足够有效，加入物体接触后，数据量、模型容量及局部视界会如何共同变化？
- **复现优先级：**先在 LAFAN1 的受控对照检查 $k$、$H$、critic 和 action noise，再讨论大动作库的计算比例；不能只复刻摘要里的极端算力比。

## 阅读可信度检查

- [x] 关键失败数、计算协议与训练集边界已核对 §4.1 和 Appendix F。
- [x] 上述两组公式已核对 §3.2–3.3；省略解析 head 的损失规则已注明。
- [x] 本笔记未新增第三方图片，原图可在论文 Figure 2–5 查看。
- [x] 个人理解与待验证问题已分开。
- [x] 未声称运行代码或复现实验。

## 关联笔记

- [DreamerV1](dreamer/dreamer-v1.md)：想象中的控制学习，但不应把两者动力学表示混为一谈。
- [RSSM](../../topics/rssm.md)：部分可观测状态的递归建模。
- [强化学习基础](../../topics/reinforcement-learning-foundations.md)：GAE、bootstrap 与策略梯度。
