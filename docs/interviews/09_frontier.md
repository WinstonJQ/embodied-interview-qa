# 卷九 · 🔥 近期热门方向与前沿题目

> 中文具身智能面试题库 · **第九卷**
> 观察窗口：2026 年 8—10 月 · 资料核对：2026-10-19；早期研究用于背景补充。
> **题目统计**：普通问答 0 · 前沿拓展 8 · 手撕 0 · N 备选 0 · 合计 **8** 个条目。

**难度分布（全卷）**：L1 **0** · L2 **5** · L3 **3**

本卷集中整理这段时间关注的热门方向：Jev、通用模型控制机械臂、Astra＋VLA、具身 Agent／Harness 与 RSI。**🔥 表示近期关注方向，不是企业面试频次或量化热度排名。**

题目依据原论文、作者代码和官方文档整理，统一标为“前沿拓展·非面经”；没有可计的具身企业面试原问，不声称企业问过，也不计面经频次。作者实验结论仅适用于其公开设置。

[返回主册](../index.html)

---

## §1 决策模型与机器人应用（2 题）

<details class="qa qa-frontier" id="f01">
<summary><span class="lv lv-l2">L2</span> <span class="freq">前沿拓展·非面经</span> <b>F01</b> · Jev／System One／RLCD 在公开资料中分别指什么？与 JEPA 有何区别？</summary>

**答**：TypeSafe 将 Jev 定位为首个 System One 模型：对给定状态和预定义问题输出类型化、带概率的决策。RLCD 是其公布的训练方法名称，意为“面向校准决策的强化学习”。Meta 的 JEPA 是联合嵌入预测架构，研究目标与接口均不同。

**易错**：名称相近不代表同一模型；官方未公开足以复现的 RLCD 损失、网络细节与训练数据，不能自行补写算法。

**资料**：[TypeSafe 原始公告](https://typesafe.ai/blog/introducing-system-one-models-and-jev)、[Meta I-JEPA 原始研究页](https://ai.meta.com/research/publications/self-supervised-learning-from-images-with-a-joint-embedding-predictive-architecture/)。

**性质**：依据技术资料整理的学习问答，非面经，不计频次。

</details>

<details class="qa qa-frontier" id="f02">
<summary><span class="lv lv-l2">L2</span> <span class="freq">前沿拓展·非面经</span> <b>F02</b> · RoboJEV 怎样让 Jev 控制机械臂？为什么不能只看 schema 和置信度验收？</summary>

**答**：RoboJEV 把 MuJoCo 的末端／物体位姿、夹爪与接触等特权结构化状态交给 TypeSafe Jev；先选意图，再选 XYZ 方向和夹爪指令，由笛卡尔控制器执行、物理判据验收。schema 只约束输出形状；`confidence` 是概率分布集中度，校准也不保证当前动作成功。

**易错**：该项目无相机输入，API 等待时仿真暂停，演示视频省略等待，未验证真机；Jev 另有实验性图像接口，不代表本项目采用视觉。

**资料**：[RoboJEV 作者仓库](https://github.com/lykycy123/RoboJEV)、[TypeSafe 入门文档](https://docs.typesafe.ai/introduction)、[概率校准说明](https://docs.typesafe.ai/introduction/machine-learning-primer)、[置信度说明](https://docs.typesafe.ai/confidence)、[实验性图像文档](https://docs.typesafe.ai/images)。

**性质**：依据技术资料整理的学习问答，非面经，不计频次。

</details>

---

## §2 通用模型控制与 VLA 协作（3 题）

<details class="qa qa-frontier" id="f03">
<summary><span class="lv lv-l2">L2</span> <span class="freq">前沿拓展·非面经</span> <b>F03</b> · Astranomy 的 Direct、Agentic、Orchestrated 三种 Astra 机器人接口如何分工？</summary>

**答**：Direct 每次由 Astra 直接给末端／关节目标或轨迹段，不保留跨调用会话；Agentic 用持久会话保留交互历史并生成动作；Orchestrated 审阅动作模型的候选，可接受、修改或接管。动作段执行后再送新图像、机器人状态和反馈。模型推理耗时与底层控制频率应分别测量。

**易错**：报告的 15 Hz 是控制步频率；成功回合中的 Direct 末端接口平均单次推理约 18.56 秒，不能写成 Astra 以 15 Hz 决策；其仿真在推理期间暂停。

**资料**：[NVIDIA Astranomy 作者报告](https://research.nvidia.com/labs/srl/projects/astranomy/)。

**性质**：依据技术资料整理的学习问答，非面经，不计频次。

</details>

<details class="qa qa-frontier" id="f04">
<summary><span class="lv lv-l2">L2</span> <span class="freq">前沿拓展·非面经</span> <b>F04</b> · RoboDojo 的 Astra 演示能证明什么？为什么不能据此认定动态真机控制可靠？</summary>

**答**：RoboDojo 主评测是 42 个任务、2100 次仿真试验，说明模型在该动作接口和基准下能完成部分任务。真机仅为选择性诊断；作者因不合理或不安全动作及设备损伤中止正式真机协议。因此需另测推理等待期间的环境变化、接触扰动、安全停止与实际成功率。

**易错**：选出的真机片段不是完整真机基准；仿真任务成功也不能推出连续动态环境或硬件上的实时闭环可靠性。

**资料**：[RoboDojo Astra 原论文](https://arxiv.org/html/2609.24170v1)。

**性质**：依据技术资料整理的学习问答，非面经，不计频次。

</details>

<details class="qa qa-frontier" id="f05">
<summary><span class="lv lv-l3">L3</span> <span class="freq">前沿拓展·非面经</span> <b>F05</b> · Astra＋VLA：候选轨迹审阅与 Python 原语编排有何不同？</summary>

**答**：银河通用研究让任务微调 π0.5 先给候选关节轨迹，Astra 选择沿用前段或用短末端动作局部覆盖。PyRUA-Lean 则由 Astra 写 Python cell，组合含冻结 VLA 的原语、检查中间结果并决定后续调用。前者改候选动作，后者编排可复用执行接口。

**易错**：银河通用结果是 RoboDojo 仿真子集；PyRUA 的无 VLA 扩展是另设实验，4B Coder 未在该设置测试。不能混成同一系统或泛化到真机。

**资料**：[银河通用候选轨迹研究](https://arxiv.org/html/2609.38537v1)、[PyRUA-Lean 原论文 v2](https://arxiv.org/html/2610.01939v2)。

**性质**：依据技术资料整理的学习问答，非面经，不计频次。

</details>

---

## §3 具身 Agent、Harness 与自我改进（3 题）

<details class="qa qa-frontier" id="f06">
<summary><span class="lv lv-l2">L2</span> <span class="freq">前沿拓展·非面经</span> <b>F06</b> · 机器人 harness 如何把观测、动作执行和验收接成物理闭环？</summary>

**答**：Thea 每轮汇入新鲜物理观测，模型选工具，执行后由 harness 触发操作评价；Show-Harness 让 VLM 输出离散语义动作，再由本体解释器转成有工作空间限制的控制目标。二者均让模型动作经过执行层，并据后续观测继续决策；Thea 另设强制操作评价。

**易错**：模型说“完成”、工具返回成功或评价器给分都不能单独充当物理真值；不同论文的工具调用与语义动作接口也不能混写。

**资料**：[Thea 作者系统说明](https://eit-hai.github.io/thea/system.html)、[Show-Harness 原论文](https://arxiv.org/html/2609.10522v1)。

**性质**：依据技术资料整理的学习问答，非面经，不计频次。

</details>

<details class="qa qa-frontier" id="f07">
<summary><span class="lv lv-l3">L3</span> <span class="freq">前沿拓展·非面经</span> <b>F07</b> · 具身 Agent 的“递归自我改进”（RSI）究竟在修改什么？</summary>

**答**：先标明改动对象与生效时点。RegenHarness 提议跨任务修订上下文、模板和路由等配置；COAP 由 coding agent 在开发期改共享代码，执行期冻结；EmbodiedEvo 继承代码技能和文字经验；ENPIRE 在人工协助建立任务接口后，还可改训练算法并更新策略权重。

**易错**：一次失败重试不等于跨任务改进；这里 RSI 是 Recursive Self-Improvement，运动模仿论文中的 RSI 也可指 Reference State Initialization。

**资料**：[RegenHarness 原论文](https://arxiv.org/html/2609.27612v1)、[COAP 原论文](https://arxiv.org/html/2610.12369v1)、[EmbodiedEvo 原论文](https://arxiv.org/html/2610.18867v1)、[ENPIRE 原论文](https://arxiv.org/html/2606.19980v1)、[Reference State Initialization 原论文](https://arxiv.org/abs/1907.11842)。

**性质**：依据技术资料整理的学习问答，非面经，不计频次。

</details>

<details class="qa qa-frontier" id="f08">
<summary><span class="lv lv-l3">L3</span> <span class="freq">前沿拓展·非面经</span> <b>F08</b> · 如何验证机器人 RSI 不是回归、评测泄漏，或只有改进协议没有增益？</summary>

**答**：保留旧版，比较时固定或记录模型／策略版本与预算，使用独立验收指标；开发、选优、留出测试按任务与布局隔离，记录每次改动、失败和回滚，在新场景重复比较并单列真机验证。RegenHarness 提出版本门禁，尚未证明持续量化增益；COAP 的结果限于使用已知资产的仿真；ENPIRE 与 EmbodiedRSI 的真机证据限于各自配置的任务。

**易错**：验证集单调上升不是未见任务或无限轮次自我进化；EmbodiedRSI 真机底层策略先按任务示教微调，再冻结，不能称本体零样本。

**资料**：[RegenHarness 原论文](https://arxiv.org/html/2609.27612v1)、[COAP 原论文](https://arxiv.org/html/2610.12369v1)、[ENPIRE 原论文](https://arxiv.org/html/2606.19980v1)、[EmbodiedRSI 原论文](https://arxiv.org/html/2610.10498v1)。

**性质**：依据技术资料整理的学习问答，非面经，不计频次。

</details>
