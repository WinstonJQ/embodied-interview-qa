# 卷五 · 工程落地 / 系统设计 / 开放题 · 高频面试题

> 中文具身智能秋招高频面试题库 · **第五卷**
> 题源：牛客 / 知乎 / CSDN / GitHub 公开面经 / NVIDIA 技术博客 / arXiv 工程论文
> **题目统计**：普通问答 54 · 手撕 8 · N 备选 3 · 合计 **65** 个条目。

**难度分布（全卷，含手撕与 N 备选）**：L1 **8** · L2 **52** · L3 **5**

> 2026-10 维护说明：新增题标“低频实录”，保留具体岗位与来源；旧 🔥 数字沿用历史版本，未重核全部原帖，不代表本季度频次。

**使用方式**：题目默认折叠，点开看答案。建议按 **L1 → L2 → L3** 顺序刷；同级内按频次从高到低。手机端原生支持。

---

## §1 感知-规划-控制系统架构（5 题）

> 机器人系统的骨架：三层架构 + 实时性约束 + 通信中间件 + 任务编排。

<details class="qa" id="q01">
<summary><span class="lv lv-l1">L1</span> <span class="freq">🔥×7</span> <b>Q01</b> · 感知-规划-控制三层架构各层的实时性要求是什么？为什么不能统一成一个频率？</summary>

**答**：频率由被控对象带宽、传感器更新、计算延迟和允许的状态陈旧度共同决定。电机驱动或关节伺服通常比视觉感知和任务规划更快，但具体数值依硬件与接口确定。常用异步分层：感知输出带时间戳的状态，策略输出目标或动作块，实时控制器在自己的周期跟踪，并处理超时和约束。

**易错**：动作执行频率、策略调用频率和底层伺服频率不能混为一谈；VLA 可以输出连续动作目标，不只输出任务指令。

**核验**：[ROS 2 实时编程说明](https://design.ros2.org/articles/realtime_background.html)、[RTC 作者说明](https://www.pi.website/research/real_time_chunking)。本题频次沿用历史记录，本轮只修订答案。

</details>

<details class="qa" id="q02">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×5</span> <b>Q02</b> · 机器人生产系统如何评估 ROS 2 和自研中间件？DDS 与实时性有哪些边界？</summary>

**答**：ROS 2 提供通信、工具与组件生态；常见 DDS 实现支持发现和 QoS，其他通信实现也须按发行版核对。生产是否采用取决于最坏延迟、抖动、可靠性、部署约束和可维护性。实时表现是内核、调度、执行器、内存、驱动与通信的系统属性，可将关键控制路径与非实时感知/服务隔离，再测目标周期下的时限违约率。

**易错**：不能断言“大公司不用 ROS 2”或给 DDS 固定微秒延迟；PREEMPT_RT 不是单独的保证，EtherCAT 也不是替代操作系统的实时调度器。

**核验**：[ROS 2 实时编程说明](https://design.ros2.org/articles/realtime_background.html)。本题频次沿用历史记录，本轮只修订答案。

</details>

<details class="qa" id="q03">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×4</span> <b>Q03</b> · 状态机（FSM）和行为树（BT）在机器人任务编排中的区别？各有什么适用场景？</summary>

**答**：FSM 用状态与转移表达流程，少量明确状态时容易实现与验证；行为树用条件、动作和控制节点组合任务，便于复用、响应条件变化和组织回退。选型取决于流程复杂度、中断语义与可观测性。机器人 Agent 也可在更高层选择工具或技能，由 FSM/BT/控制器落实执行；这些层可以共存。

**易错**：行为树不天然解决所有中断，也不能断言 VLA 指令通常都会先转成 BT；工具返回成功仍须结合物理状态判断任务是否完成。

**核验**：[Nav2 行为树说明](https://docs.nav2.org/rolling/getting_started/navigation_concepts/behavior_trees/)、[Thea 技术报告](https://eit-hai.github.io/thea/index.html)。本题频次沿用历史记录，本轮只修订答案。

</details>

<details class="qa" id="q04">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×4</span> <b>Q04</b> · 共享内存 vs 消息队列 vs ROS2 Topic，在低延迟控制回路中应该选哪种？</summary>

**答**：先按最坏延迟、抖动、负载和故障语义选通信路径。共享内存减少复制但要设计所有权、同步与进程故障处理；消息队列提供解耦和背压；ROS 2 Topic 提供类型、QoS 与生态集成。关键控制链路还应检查调度优先级、内存分配、锁竞争与数据时效，并在目标硬件压力下测量。

**易错**：通信方式不能对应固定微秒数字。直接禁用抢占或无界自旋可能拖住实时任务；双缓冲也必须有正确的发布与读取同步协议。

**核验**：[ROS 2 实时编程说明](https://design.ros2.org/articles/realtime_background.html)。本题频次沿用历史记录，本轮只修订答案。

</details>

<details class="qa">
<summary><span class="lv lv-l1">L1</span> <span class="freq">🔥×5</span> <b>Q05</b> · 什么是阻抗控制（Impedance Control）？与位置控制和力控制的区别？机器人操作任务里怎么选？</summary>

**答**：**阻抗控制**：让机器人末端呈现可调"弹簧-阻尼-质量"特性：$F = K \Delta x + D \dot{x} + M \ddot{x}$，通过调节虚拟刚度 K、阻尼 D，控制机器人与环境交互时的柔顺性。

- **位置控制**：精确跟踪轨迹，刚性高；接触未知环境时力过大 → 损坏零件或机器人
- **力控制**：维持期望接触力；需要力矩传感器 + 稳定的力估计，对刚性接触不稳定
- **阻抗控制**：两者折中；环境已知时刚、接触时柔，适合大多数操作

**选型**：无接触精密移动 → 位置控制；稳定接触力（如打磨/拧螺丝） → 力控制；插销/组装/接触不确定 → 阻抗控制。

**易错**：阻抗控制不是"只要有力矩传感器就能用"——需要精确的动力学模型（质量矩阵 M）；低成本机器人通常用导纳控制（Admittance Control，在位置控制外环）替代。

</details>

---

## §2 推理加速与边缘部署（8 题）

> VLA 推理慢是当前最高频工程面试话题；量化/TensorRT/Jetson/蒸馏覆盖 80% 考点。

<details class="qa" id="q06">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×6</span> <b>Q06</b> · VLA 推理有哪些瓶颈？自回归动作与扩散／Flow 动作头如何分别提速？</summary>

**答**：先做分段 profiling：视觉编码、主干条件编码、动作 token 解码或扩散/flow 迭代、传输和控制等待。AR 策略可考虑动作压缩或并行回归；迭代动作头可考虑减少采样步数或蒸馏；再评估量化、算子融合和异步动作块执行。方案应以目标硬件上的任务成功率、尾延迟和闭环稳定性验收。

**易错**：VLA 不全是自回归模型，GR00T/π0 类迭代动作头也不等于“一次 forward”；FAST 压缩动作不保证比 flow 解码更快。

**核验**：[OpenVLA-OFT 作者页](https://openvla-oft.github.io/)、[FAST 作者说明](https://www.pi.website/research/fast)。本题频次沿用历史记录，本轮只修订答案。

</details>

<details class="qa" id="q07">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×6</span> <b>Q07</b> · INT8 / FP16 / AWQ / GPTQ 量化的区别？机器人推理部署推荐哪种？</summary>

**答**：FP16/BF16 是浮点精度，不是量化算法；INT8 是表示精度，可配 PTQ 或 QAT。AWQ 利用校准激活寻找敏感通道，通过等价缩放降低低比特权重量化误差；GPTQ 用近似二阶信息逐步补偿权重量化误差。选择须同时考虑目标芯片内核、权重/激活精度、运行时内存及机器人闭环精度。

**易错**：AWQ 并非简单把显著通道保留为不量化；AWQ/GPTQ 不等于只能 INT4，也不能预先保证谁更稳或给出通用 Hz。

**核验**：[AWQ 原论文](https://arxiv.org/abs/2306.00978)、[GPTQ 原论文](https://arxiv.org/abs/2210.17323)。本题频次沿用历史记录，本轮只修订答案。

</details>

<details class="qa" id="q08">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×5</span> <b>Q08</b> · TensorRT、ONNX Runtime 与 PyTorch 导出／编译路径如何选？旧 TorchScript 工程怎么处理？</summary>

**答**：TensorRT 面向 NVIDIA 平台，需核对算子、动态形状和构建配置；ONNX Runtime 通过不同执行后端适配多平台；PyTorch 的 torch.export 捕获可供后端处理的图，torch.compile 用于编译优化。既有 TorchScript 工程可维护兼容，新项目应优先核对当前导出及部署路径，端侧可进一步评估 ExecuTorch。

**易错**：TorchScript 已被官方弃用，不宜继续作为新项目默认推荐；torch.export、ONNX 是图表达/导出路径，与具体推理运行时承担的职责不同。

**核验**：[PyTorch 官方弃用说明](https://pytorch.org/blog/pytorch-2-12-release-blog/)、[Torch-TensorRT legacy 文档](https://docs.pytorch.org/TensorRT/legacy.html)。本题频次沿用历史记录，本轮只修订答案。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×4</span> <b>Q09</b> · Jetson Orin AGX 跑 7B VLA 的典型配置和瓶颈在哪？如何估算推理吞吐？</summary>

**答**：**Jetson Orin AGX 64GB**（2023，275 TOPS INT8，64 GB LPDDR5 统一内存）：

**典型配置**：
- 7B VLA INT8 量化 → ~8-10 GB 显存
- 推理吞吐：**8-12 Hz**（取决于视觉 encoder 大小和 chunk 长度）
- 功耗：~40-60W（vs A100 ~400W）

**估算方法**：FP16 模型 = 参数量 × 2 bytes；INT8 = 参数量 × 1 byte；7B × 1 byte = 7 GB，加 KV cache + 激活 ≈ 9-11 GB。推理延迟 ≈ forward FLOPs / 峰值算力（考虑 memory-bound 效率 40-60%）。

**瓶颈**：① 统一内存带宽（204 GB/s）是主瓶颈——LLM 推理 memory-bound；② 视觉 encoder（ViT-L 约 300M）单帧约 20 ms；③ 散热限制功耗 throttling。

**易错**：TOPS 是 INT8 峰值，实际 LLM 推理用不满（带宽受限）；不要用 TOPS 直接估算真实 token/s。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×5</span> <b>Q10</b> · 知识蒸馏（Knowledge Distillation）的流程是什么？机器人场景有哪些适配挑战？</summary>

**答**：**KD 流程**：Teacher（大模型）软化输出（temperature T > 1 的 softmax 分布）→ Student 模仿 teacher 的"暗知识"（soft label + 中间层特征对齐），在小模型上实现接近大模型的性能。

**常规 KD 步骤**：① 确定 teacher（如 7B VLA）→ ② 设计 student（如 1B）→ ③ 在数据集上运行 teacher 生成 soft label → ④ student 用 KL 散度 + 任务 loss 联合训练。

**机器人场景挑战**：
- 动作输出是**多模态分布**（Diffusion/Flow Matching），KL 散度对多模态效果差；改用 **feature-level 蒸馏** 或 **score distillation**
- 训练数据量小（1K-50K demos），student 易过拟合 teacher
- 真机评估成本高，无法像 NLP 那样快速迭代 student 质量

**实际路线**：π0 系列用 PaliGemma 3B（SigLIP + Gemma 组合）作预训练 VLM backbone；LoRA 量化微调比全量 KD 更常用，成本更低。

**易错**：KD 不是"直接 copy teacher 权重 + 减小网络"，是软标签驱动的训练过程。

</details>

<details class="qa" id="q11">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×4</span> <b>Q11</b> · Flash Attention 在 VLA 推理中的收益是什么？KV cache 在 VLA 长 context 场景的必要性与局限？</summary>

**答**：FlashAttention 分块计算精确注意力，减少显存读写与中间注意力矩阵存储，标准 dense attention 的二次算术量并未消失。KV cache 保存已处理前缀的 K/V，避免自回归解码重复投影；每个新 token 仍需关注历史，生成整段序列的注意力计算通常仍是二次量级。缓存内存按实际缓存 token 数增长。

**易错**：动作块覆盖 50 个控制步，不等于缓存里新增 50 帧视觉 token；跨观测复用还取决于位置、掩码及条件是否改变。激活重计算也不是 KV cache 的替代品。

**核验**：[FlashAttention 原论文](https://arxiv.org/abs/2205.14135)、[Hugging Face 缓存说明](https://huggingface.co/docs/transformers/cache_explanation)。本题频次沿用历史记录，本轮只修订答案。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×3</span> <b>Q12</b> · batched inference 与机器人实时控制的矛盾如何解决？</summary>

**答**：**矛盾**：batched inference 通过聚合多个请求摊销固定 overhead，吞吐提升 N×；但单个请求必须等 batch 凑满才发，**延迟增加** = 等待时间 + 计算时间，与机器人实时控制的低延迟需求（< 100 ms）冲突。

**解决思路**：
1. **单路独占**：单机器人 → batch_size=1，不凑批，延迟最低但 GPU 利用率低
2. **时间窗口 mini-batch**：固定最大等待时间（如 10 ms），无论 batch 凑没凑满都发；在多机器人集群（Fleet）中平衡延迟和吞吐
3. **持续批处理（Continuous Batching）**：vLLM 实现，请求随到随入 batch，不固定等待窗口；适合云端推理服务
4. **Action Chunking 预计算**：一次 VLA forward 生成 H=50 步动作，低层控制器跑完再请求下次推理；实际推理频率 1-2 Hz，对 batch 要求低

**易错**：Action Chunking 不是"减少 VLA 调用频率"的妥协，而是 ACT 的核心设计思想（降低复合误差）；推理频率低是副产品。

</details>

<details class="qa">
<summary><span class="lv lv-l3">L3</span> <span class="freq">🔥×3</span> <b>Q13</b> · 投机解码（Speculative Decoding）的原理是什么？机器人 VLA 能用吗？有哪些挑战？</summary>

**答**：**原理**：用小 draft 模型（如 1B）串行快速生成 k 个候选 token，再用大 target 模型（如 7B）**并行验证**所有 k 个 token（一次 forward）；若验证通过则接受，否则截断重生成。理论加速比 = k / (1 + k × rejection_rate)，实测 NLP 场景 2-3×。

**机器人 VLA 的可行性**：
- **可行**：动作 token 分布相对简单（7 维连续动作离散化），draft 模型命中率可能较高
- **挑战 1**：VLA 动作 token 只有 7-350 个（chunk），speculation benefit 小（NLP 句子上千 token）
- **挑战 2**：需要维护两个模型（draft + target）的显存，Jetson 上极为紧张
- **挑战 3**：VLA 带视觉 token，draft 模型也需要处理图像 → 不是纯 LLM 投机

**实际应用**：主要见于云端 VLA 服务（多请求并发），边缘部署更常用量化 + 并行动作头（如 GR00T N1 的扩散式并行动作生成）；两者原理不同，但都减少了解码步数。

**易错**：投机解码不改变模型精度（验证步保证与 target 分布一致），是纯推理加速技巧。

</details>

---

## §3 训练工程（7 题）

> 分布式训练 + 内存优化 + 混合精度 + VLA 训练的特殊考量。

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×7</span> <b>Q14</b> · DDP / FSDP / DeepSpeed ZeRO 三者区别？VLA（7B）训练用哪个？</summary>

**答**：
| 方案 | 切分内容 | 每卡所需显存 | 通信量 | 适用 |
|---|---|---|---|---|
| **DDP** | 不切分（每卡完整模型） | 完整参数 + 梯度 + 优化器 | all-reduce 梯度 | < 3B，单机多卡 |
| **FSDP（ZeRO-3）** | 参数/梯度/优化器全切分 | 1/N（+临时通信开销） | all-gather + reduce-scatter | ≥ 7B，多机 |
| **DeepSpeed ZeRO-1/2/3** | 逐级切分优化器/梯度/参数 | 递减 | 递增 | ≥ 7B，灵活配置 |

**7B VLA 推荐**：
- **4×A100（40GB）**：FSDP or ZeRO-2（切梯度+优化器，不切参数）→ 稳定快
- **8×A100（80GB）**：ZeRO-1（只切优化器）足够，通信少
- **多机（≥4 节点）**：ZeRO-3 / FSDP + NVLink 节点内、InfiniBand 跨节点

**实测规律**：ZeRO-3 显存最省但通信最多，跨机 InfiniBand 慢时吞吐下降明显；VLA 首选 **ZeRO-2 + activation offload** 平衡显存和速度。

**易错**：FSDP 是 PyTorch 原生的 ZeRO-3 等价实现，不是比 ZeRO 更先进，是平台选择（PyTorch vs DeepSpeed）。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×5</span> <b>Q15</b> · VLA 训练需要多少显存？如何估算 7B 模型的训练显存？</summary>

**答**：**估算公式（混合精度训练，Adam optimizer）**：

显存 ≈ 参数 × (2 B FP16 参数 + 2 B FP16 梯度 + 4 B FP32 主参数 + 4 B Adam m + 4 B Adam v) + 激活值

= 参数量 × **16 bytes** + 激活值

**7B VLA 示例**：7 × 10⁹ × 16 = **112 GB**（仅模型状态），+ 激活值（batch 依赖，约 20-60 GB）→ 总计 **130-170 GB**。

**优化手段**：① gradient checkpointing：激活值只存 checkpoint 节点，重算其余，激活显存 → ~10 GB（代价：增加约 30% 计算量）；② ZeRO-3 切分到 8 卡：每卡 ~20 GB；③ INT8 量化 + LoRA：7B LoRA 约 20-30 GB（仅训练 adapter 参数，不更新全量权重）。

**典型配置**：7B VLA 全参数微调 → **8 × A100 80GB**（ZeRO-2）；LoRA 微调 → **4 × A100 40GB**。

**易错**："7B 模型 = 7 GB" 是 INT8 推理权重估算（1 byte/param）；FP16 推理权重约 14 GB；训练还需 Adam 优化器状态（额外 8 bytes/param），总量完全不同数量级。

</details>

<details class="qa">
<summary><span class="lv lv-l1">L1</span> <span class="freq">🔥×5</span> <b>Q16</b> · 混合精度训练（AMP）原理？FP16 vs BF16，VLA 训练建议用哪个？</summary>

**答**：**AMP 原理**：前向/反向用低精度（FP16/BF16）计算，维护 FP32 主参数副本用于更新；梯度放大（GradScaler）防 FP16 下溢（underflow）。节省显存约 50%，训练加速 2-3×（Tensor Core 专为低精度优化）。

**FP16 vs BF16**：
| | FP16（E5M10） | BF16（E8M7） |
|---|---|---|
| 指数位 | 5 位（范围小） | 8 位（与 FP32 同范围） |
| 精度位 | 10 位（精度高） | 7 位（精度略低） |
| 溢出风险 | 高（需 GradScaler） | 极低（范围与 FP32 一致） |
| 硬件支持 | A100/V100 均支持 | A100/H100 强支持；V100 不支持 |

**VLA 建议**：A100/H100 → **BF16**（无需 GradScaler，更稳定，VLM 大梯度场景不溢出）；V100 → FP16 + GradScaler。

**易错**：AMP 不是"全程 FP16"——优化器更新和权重维护仍是 FP32，只是前反向低精度。BF16 不需要 GradScaler。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×5</span> <b>Q17</b> · Flash Attention 的原理是什么？VLA 训练中为何几乎是标配？</summary>

**答**：**标准 Attention 瓶颈**：$QK^T$（$L \times L$）需 $O(L^2)$ HBM 读写；VLA context 达 512-2048 token 时带宽成瓶颈。

**核心思路**：① 分块（Tiling）：Q, K, V 小块在 SRAM 上完成 softmax + 加权，不写中间矩阵回 HBM；② Online Softmax 修正：维护 running max/sum 保证数值等价；③ 反向时重算 attention（不缓存 $L^2$ 矩阵）。

**收益**：HBM 读写 $O(L^2) → O(L)$，速度 2-4×，显存大幅降低。

**VLA 几乎标配原因**：多帧图像 token（每帧 256）+ 文本 + 历史动作，context 长度大，标准 attention 显存紧张甚至 OOM。

**易错**：Flash Attention 只改变内存访问模式，不改变计算结果（数学等价）；与 KV cache 优化目标不同——Flash Attention 训练/推理通用，KV cache 仅推理。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×4</span> <b>Q18</b> · gradient checkpointing 原理？VLA 训练中怎么用？与 ZeRO 如何配合？</summary>

**答**：**原理**：标准反向传播缓存所有前向激活（显存 $O(L \times depth)$）。Gradient Checkpointing 只保留每 N 层一个 checkpoint 节点，反向时从最近节点重算中间激活，以约 **30% 额外计算** 换取 **60-80% 激活显存节省**，且梯度数值不变（数学等价）。

**VLA 配置**：对 LLM backbone 每层开 `gradient_checkpointing=True`；视觉 ViT 层数少，可不开。

**与 ZeRO 配合**：ZeRO-2/3 切分参数/梯度显存；gradient checkpointing 切分激活显存，两者正交互补。典型：7B + ZeRO-2 + checkpointing → 4 × A100 40GB 可跑。

**易错**：不是"保存 ckpt 文件"，是激活重计算；开启后训练速度降低约 30%，不影响梯度准确性。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×4</span> <b>Q19</b> · 单机多卡 vs 多机训练：VLA 常见配置与跨机通信的瓶颈在哪？</summary>

**答**：
| | 单机多卡（8 × A100） | 多机（如 4 × 8 卡） |
|---|---|---|
| 卡间带宽 | NVLink 600 GB/s（A100 SXM） | InfiniBand 200-400 Gb/s（~25-50 GB/s）|
| 通信延迟 | 极低（µs） | ms 级 |
| 扩展性 | 受机器限制 | 水平扩展 |
| 常见瓶颈 | 单机内存（8 × 80GB = 640GB） | 跨机 all-reduce 等待 |

**VLA 常见配置**：
- 7B 全参数：8 × A100 80GB（ZeRO-2），单机足够
- 13B-70B 或大批量：多机，用 InfiniBand + NCCL，ZeRO-3
- 通信优化：gradient compression（梯度稀疏化/1-bit）or Async SGD（有精度损失）

**跨机瓶颈**：7B 模型 FP16 梯度约 14 GB，InfiniBand 200 Gb/s（实际有效带宽 ~60%）传输需 ~1 s；gradient accumulation 增大 batch 可分摊通信开销。

**易错**：多机不是"卡越多越快"——强扩展效率（Strong Scaling Efficiency）在 8+ 节点后常见 < 80%；通信开销随节点数增加。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×3</span> <b>Q20</b> · torch.compile 在 VLA 训练中的加速收益与常见坑？</summary>

**答**：**torch.compile**（PyTorch 2.0+）：基于 TorchDynamo（Python 字节码捕获计算图）+ TorchInductor（生成 Triton GPU kernel），将 eager 模式编译为融合 kernel，减少 kernel launch 次数和 HBM 读写。

**典型收益**：纯 Transformer forward/backward 约 **10-30% 吞吐提升**（无数据 IO 瓶颈时）；VLA 因含视觉 encoder + 动作头，实测 5-20%（因图像预处理、动态 shape 等限制编译效率）。

**常见坑**：① **动态 shape**：VLA 输入长度变化 → 每种 shape 重编译（首次 1-3 min），加 `dynamic=True` 缓解；② **自定义 CUDA op**（如 Flash Attention）fallback eager，需显式标注；③ **silent fallback**：编译失败不报错，需加日志确认是否生效。

**VLA 建议**：主要对 LLM backbone 开，视觉 encoder 保持 eager 避免动态 shape 坑。

**易错**：torch.compile 不能替代 Flash Attention（解决不同层面），两者可叠加。

</details>

---

## §4 数据飞轮与采集（6 题）

> 遥操设备选型 + 数据集特征 + 采集流程 + 互联网视频预训练。

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×6</span> <b>Q21</b> · teleop 遥操设备如何选型？VR / leader-follower / exoskeleton / Aloha-style 各适合什么场景？</summary>

**答**：
| 设备 | 示例 | 优势 | 劣势 | 适用场景 |
|---|---|---|---|---|
| **Leader-follower arm** | ALOHA, Lerobot SO-ARM100 | 力反馈自然，学习曲线低 | 需配套夹爪，双臂协调难 | 桌面操作、精细抓取 |
| **VR 手柄** | Quest 3 + ROS bridge | 低成本，6-DoF 追踪 | 无力觉，远端操作抖动大 | 大范围移动任务，数据量级 |
| **动作捕捉套装** | Rokoko / Vicon | 全身 DoF，丰富表达 | 昂贵，标注后处理复杂 | 人形全身操控 |
| **外骨骼** | Inspire Dexterous Hand + arm | 手指级灵巧度 | 穿戴复杂，延迟高 | 5 指灵巧操作（如拧瓶盖） |
| **数据手套** | SenseGlove / HaptX | 触觉反馈 | 贵，续航 | 精密触感任务 |

**2024-2026 主流**：ALOHA-style leader-follower（双臂 14-16 DoF）是最多公开数据集的选择；宇树 G1 用 VR + 全身运动捕捉。选型核心：**操作的 DoF 要求**（精细 → exo，范围 → VR，双臂 → leader-follower）。

**易错**：VR 手柄抖动会污染数据（需平滑滤波 / 仅用关键帧），不能直接当 GT 动作。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×5</span> <b>Q22</b> · Open-X-Embodiment / DROID / Ego4D 三大数据集的区别与各自定位？</summary>

**答**：
| 数据集 | 规模 | 载体 | 特点 | 适用 |
|---|---|---|---|---|
| **Open-X-Embodiment** | 1M+ 轨迹，22 种机器人 | 多 embodiment | 最大多体系开放数据集；任务短/简单，多样性强 | VLA 预训练，跨机器人泛化 |
| **DROID** | 76K 轨迹，~350h，Franka Panda | 单 embodiment | 564 场景、86 个任务、50 名采集者；自然语言指令 | 真实场景操作，policy 微调 |
| **Ego4D** | 3500+ 小时视频，74 个地点 | 纯视频，非机器人 | 以人为主视角，活动多样；无动作标签 | 视觉表征预训练（视觉理解），不直接训策略 |

**关键区别**：OXE 多体系宽泛，DROID 单体系高质量，Ego4D 无动作标签（只能做视觉预训练）。VLA 预训练通常：Ego4D + OXE 做视觉/语言预训练 → 任务数据微调 → DROID 验证泛化。

**易错**：Ego4D 不能直接训练机器人策略（缺动作标签），只用于视觉表征学习；不要说"用 Ego4D 训策略"。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×5</span> <b>Q23</b> · 如何设计高质量机器人数据采集流程？数据清洗与标注的关键点是什么？</summary>

**答**：**采集流程设计**：
1. **环境标准化**：固定照明/背景/相机位姿（减少无关变量），但 DROID-style 也需多场景覆盖泛化
2. **Operator 培训**：操作员统一培训（避免动作风格差异），多 operator 增加多样性（Mobile ALOHA 策略）
3. **同步录制**：多相机（wrist + overhead）+ 关节状态 + 时间戳对齐；< 1 ms 同步需**硬件触发**（外部触发信号）或 PTP（IEEE 1588，µs 级）；NTP 只能做毫秒级粗同步
4. **在线质量筛**：任务完成检测（如物体位置传感器）自动过滤失败轨迹

**数据清洗关键**：
- **抖动过滤**：VR 手柄输入需低通滤波（截止频率 5-10 Hz）
- **动作平滑**：去除突变（速度超阈值的帧）
- **失败轨迹剔除**：夹爪未闭合 / 末端超出工作空间 → 自动剔除
- **标注**：自然语言任务描述（多人标注取一致）；成功/失败标签

**易错**：不要只收集"成功轨迹"——失败轨迹（负样本）对 RLHF / offline RL 有价值；完全清洗掉失败数据使策略无法学习恢复。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×4</span> <b>Q24</b> · 互联网视频预训练（Ego4D/Something-Something/YouTube）对机器人策略有什么用？有哪些根本局限？</summary>

**答**：**用处**：① 学习视觉表征（物体形状、纹理、手部运动先验）→ 下游机器人任务 fine-tune 数据需求减少 5-10×；② 学习物理常识（物体如何掉落、液体如何流动）→ 减少仿真的 domain gap；③ 语言-视觉对齐（理解任务指令）。

**典型使用**：Ego4D 预训练 → EgoVLP/MAE 视觉 encoder → 迁移到 VLA（如 Octo 用 ImageNet 预训练 ViT）；**R3M**（Nair et al., 2022）用 **Ego4D** 人类活动视频预训练，对桌面抓取任务有明显收益。

**根本局限**：
- **无动作标签**：视频只有像素，不知道机器人该输出什么关节角 / 力矩
- **视角不匹配**：人手视角 vs 机器人腕部相机视角有巨大 domain gap
- **物理差异**：人手柔顺，机器人刚性，力的分布完全不同
- **长尾分布**：互联网视频大量是日常活动，机器人精密操作数据极少

**易错**：视频预训练的收益主要在**视觉感知**层，不会"自动学会如何操控"——策略学习仍需机器人专属数据。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×5</span> <b>Q25</b> · 数据飞轮（Data Flywheel）的核心闭环是什么？为什么 scale 起来很难？</summary>

**答**：**数据飞轮闭环**：部署机器人 → 采集交互数据（成功+失败）→ 过滤/标注 → 训练改进策略 → 更好的策略 → 更多/更难任务 → 继续采集。理论上闭环越快越强。

**为什么难 scale**：
1. **数据采集速度瓶颈**：teleop 需要人工操作，单台机器人采集约 1-5 demos/小时，1000 demos = 200-1000 人时
2. **数据多样性 vs 质量矛盾**：多场景保证泛化，但每场景样本少 → long-tail 问题
3. **标注成本**：自然语言任务描述 + 成功/失败标签 + 关键帧标注，人工成本非线性增长
4. **分布 shift**：旧数据与新策略的状态分布不一致 → 需 DAgger-style 在线数据收集
5. **Reset 成本**：真机每次 demo 后需 reset 场景，人工耗时大

**行业现状（2025）**：自主数据采集（机器人在夜晚自己采集）+ VR 远程采集集群是主要突破方向；Physical Intelligence（π）、Figure 都有大规模 teleop 采集基础设施。

**易错**：数据飞轮不是"有了数据就飞轮"，闭环的关键是**数据质量筛选 + 增量训练策略**，垃圾数据会反向拖累策略。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×3</span> <b>Q26</b> · 自主数据采集（Autonomous Data Collection）有哪些思路？与 teleop 相比的优势和局限？</summary>

**答**：**主要思路**：
1. **Reset-free Learning**：策略自主探索，无需人工 reset（如 Forward-Backward RL，在"做任务"和"恢复初始态"间循环）
2. **Curiosity-Driven Exploration**：自动选择"信心低"状态收集数据（DAGGER 风格在线查询）
3. **Scripted / Heuristic Collection**：写简单程序控制机器人随机/系统地扫描工作空间，采集带 annotation 的场景数据
4. **Human-in-the-Loop**：策略自主运行，失败时触发远程 teleoperation 补全

**vs teleop 优势**：24/7 无人工，单机器人日采集量可达 teleop 的 10-50×；边际成本极低。

**局限**：
- 初始策略不够强时，自主探索只采到低价值数据（随机碰撞）
- 缺乏任务目标驱动（需要 reward 定义或任务成功检测）
- 安全性：无人监督下需硬件安全限位

**易错**：自主数据采集不是"让机器人自己学"——仍需要明确的成功/失败检测和数据标注机制。

</details>

---

## §5 失败排查与调试（7 题）

> 真机部署必考：sim2real 定位 → reward hacking → 灾难性遗忘 → 控制调试。

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×6</span> <b>Q27</b> · 训练好的策略真机表现差，第一步怎么排查 sim2real gap？系统化排查流程是什么？</summary>

**答**：**系统化排查流程（由浅入深）**：

**Step 1 — 输入分布检查**：记录真机传感器数据（图像/关节状态）与仿真数据的分布差异（直方图对比）；常见：光照/背景/物体颜色偏差，关节摩擦力估计不准。

**Step 2 — 策略输出检查**：在真机上 rollout 时 log 动作分布；若策略输出与仿真动作差异大，说明观测 domain gap 大；若动作分布类似但执行失败，是执行器/物理 gap。

**Step 3 — 逐模块 ablation**：
- 关掉感知，直接输入仿真参考状态 → 排查控制 gap
- 用仿真相机 → 排查视觉 gap
- 用真实图像 + ground truth 状态 → 排查策略泛化

**Step 4 — 常见 gap 原因**：
- 物体质量/摩擦系数不准 → 加 domain randomization 重训
- 相机内参/外参偏差 → 重新标定
- 控制频率 / 延迟不匹配 → 统一仿真和真机延迟设置

**易错**：不要第一步就调超参数，先定位 gap 来源再针对性修；盲目加 DR 有时反而降低真机策略质量（过度泛化导致精度下降）。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×5</span> <b>Q28</b> · 什么是 reward hacking？机器人训练中的常见表现和解决思路？</summary>

**答**：**Reward hacking**：策略找到了"高 reward 但违背任务意图"的捷径行为，即 reward 函数没有完整表达人的意图。

**机器人常见表现**：
- 抓取任务：夹爪直接压住物体（不抬起）触发"接触 = 抓到"reward
- 导航任务：机器人原地抖动触发"速度大 = 探索积极"reward
- 灵巧操作：过度用力挤压物体达成"接触力 > 阈值"的假成功

**解决思路**：
1. **Reward Shaping 精细化**：加约束（如同时检测抬起高度 + 接触力 + 位移）
2. **RLHF / Preference Learning**：人类偏好标注替代 reward 函数
3. **Demo-conditioned reward**：只有动作接近 demo 时才给 reward（如 GAIL/AIRL）
4. **多目标约束**：主 reward + 行为约束 penalty（如异常力矩惩罚）
5. **真机验证频率提高**：仿真训练 + 每 N 步真机评估，早发现 hacking 行为

**易错**：Reward hacking 不是"策略 bug"，是 reward 设计问题；加更多训练数据解决不了，必须修 reward 定义。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×5</span> <b>Q29</b> · 灾难性遗忘（Catastrophic Forgetting）在 VLA 微调中怎么出现？如何缓解？</summary>

**答**：**出现场景**：将预训练 VLA（在大规模数据上训练的通用能力）在特定任务数据上**全参数微调**时，模型快速过拟合新任务 → 遗忘预训练时的语言理解/视觉泛化能力，在新任务上成功，其他任务失败。

**常见表现**：微调后在原任务上成功率骤降（如 OpenVLA 微调新桌面抓取 → 忘记开门）；语言指令理解变差（只能响应微调数据的指令风格）。

**缓解策略**：
1. **LoRA / PEFT**：只更新 < 1% 参数，冻结主干 → 最常用
2. **EWC**：Fisher 信息矩阵指导，惩罚重要参数大幅改变
3. **Replay / Co-training**：混入预训练数据（Mobile ALOHA co-train 策略）
4. **KI（Knowledge Insulation）**：π0.5 思路，stop-gradient 隔离 VLM 和 Action Expert
5. **差异 lr**：VLM backbone 用极小 lr（1e-5），action head 用较大 lr（1e-4）

**易错**：LoRA 不能完全防遗忘；彻底防遗忘仍需 co-training 混入原始数据。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×4</span> <b>Q30</b> · 关节控制振荡的常见原因与调参思路？PID 和阻抗控制各怎么调？</summary>

**答**：**振荡原因**：
- PID P 增益过大 → 超调 + 振荡（欠阻尼）
- 控制频率低（> 10 ms 周期）→ 相位滞后 → 条件不稳定
- 关节静摩擦（stiction）→ 低速黏滑（stick-slip）现象
- VLA 动作 chunk 末端不连续 → 关节速度跳变

**PID 调参**：① Ziegler-Nichols 法（逐渐增 P 至临界振荡，读取 $K_u$ 和 $T_u$，代入公式计算 P/I/D）；② 人工逐步调：先清零 I/D，调 P 至刚好振荡，再加 D（微分减振荡），最后加 I（消除稳态误差）；③ 加前馈（Feedforward）：预计算重力补偿，减轻 P 的负担。

**阻抗控制调参**：虚拟刚度 K 太大 → 刚性振荡（减 K 或加 D）；阻尼 D 太小 → 欠阻尼（增 D 至临界阻尼 $D = 2\sqrt{KM}$）；质量 M 影响响应速度（通常设为实际惯量的 1-5×）。

**VLA chunk 振荡**：Chunk 边界加速度不连续 → 用三次样条插值平滑相邻 chunk 的连接；或减小 chunk 长度 H。

**易错**：不要把"振荡"和"稳态误差"混淆，两者病根不同（前者是 P/D 问题，后者是 I 问题）。

</details>

<details class="qa">
<summary><span class="lv lv-l3">L3</span> <span class="freq">🔥×3</span> <b>Q31</b> · Diffusion Policy 训练时的 Mode Collapse（模式坍塌）如何排查？</summary>

**答**：**Diffusion Policy 的 Mode Collapse**：多模态动作分布下，策略只学会一种模式（如总是选同一抓取方向），丢失其他合法方案。

**排查方法**：① 同一观测下多次采样（N=100）动作，绘 2D 投影；若全部聚成一团 → collapse；② log 各 diffusion step 的 loss，若小 t（精细步）loss 极低而大 t 偏高 → 过拟合单一模式；③ 检查 score 网络输出方差是否异常小。

**根本原因**：① 训练数据单模态（operator 风格单一）→ 增加操作员多样性；② Beta schedule 过早收紧 → 调整为 cosine schedule；③ CFG Guidance scale 过高 → 降低；④ lr 过大 → 降至 1e-4 以下。

**易错**：Diffusion Policy 本设计防 mode collapse，若仍 collapse 先查数据；与 GAN collapse 机制不同，无需改判别器。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×3</span> <b>Q32</b> · 传感器漂移（Sensor Drift）怎么检测和补偿？以 IMU 和力矩传感器为例？</summary>

**答**：**IMU 漂移**：陀螺仪积分误差随时间累积（Gyro bias），表现为姿态估计随时间漂移（静止时角速度输出非零）。
- 检测：静置机器人采集 N 秒数据，计算静态时 bias 均值和方差
- 补偿：① 开机标定（校准 bias offset）；② **互补滤波 / EKF**：融合加速度计（无漂移但噪声大）和陀螺仪（短期精准但长期漂），EKF 状态量含 bias 项自动估计

**力矩传感器漂移**：温度变化导致零点漂移（Thermal Drift），表现为无负载时力/力矩读数随温度升高缓慢变化。
- 检测：预热机器人 5-10 分钟，监测空载读数变化（> 0.5 N 为需补偿）
- 补偿：① 运行前**重新清零（tare）**；② 建立温度-零点补偿模型（查表或线性拟合）；③ 选用内置温度补偿的传感器（ATI Axia / Rokbi）

**通用原则**：定期标定（每月或每 N 小时作业）；记录环境温度、湿度与漂移量，建补偿表。

**易错**：不要把"漂移"和"噪声"混淆——漂移是系统性偏差（低频），噪声是随机（高频）；滤波器设计不同。

</details>

<details class="qa" id="q33">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×3</span> <b>Q33</b> · VLA 推理时序抖动（Inference Jitter）如何影响控制？有哪些工程解决方案？</summary>

**答**：先测观测年龄、排队、预处理、推理、通信及控制执行的延迟分布，而非只看均值。动作块异步生成时维护绝对时间与执行位置，检测过期、缺块和超时；RTC 等方法让新块与已承诺动作对齐。再检查控制器的速度/加速度限制、缓冲下溢与停止/保持策略，并用实际负载压力测试。

**易错**：双缓冲或平滑并不自动消除新旧块冲突；较长块也可能增加观测陈旧。抖动可能来自调度、驱动、网络或设备，不能一概排除硬件。

**核验**：[RTC 作者说明](https://www.pi.website/research/real_time_chunking)、[ROS 2 实时系统设计](https://design.ros2.org/articles/realtime_background.html)。本题频次沿用历史记录，本轮只修订答案。

</details>

---

## §6 开放题与系统设计（6 题）

> 无标准答案，考察工程判断力和技术视野；答题骨架 + 必谈点 + 加分点。

<details class="qa" id="q34">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×5</span> <b>Q34</b> · 遥操示教 + 模仿学习有哪些工程优势和局限？</summary>

**答**：遥操作提供与机器人观测对齐的动作示教，可先用 BC、ACT、diffusion/flow 等方法建立任务基线，省去早期大量探索与奖励设计。数据采集、质量检查和真机评测也较易形成可控流程。但示教覆盖、重置成本、跨本体迁移和分布偏移仍是限制；可结合仿真、离线数据及 RL 后训练逐步补足。

**易错**：不存在统一最优范式或固定“几个月胜过几年”的周期；自主执行数据采集与人工遥操要分开说明，RL 也不只是对 BC 模型简单套 PPO。

**核验**：[ACT 原论文](https://arxiv.org/html/2304.13705)、[RECAP 原论文](https://arxiv.org/abs/2511.14759)。本题频次沿用历史记录，本轮只修订答案。

</details>

<details class="qa" id="q35">
<summary><span class="lv lv-l3">L3</span> <span class="freq">🔥×6</span> <b>Q35</b> · VLA 端到端方案 vs 传统模块化方案：2026 年你会怎么选？</summary>

**答**：从任务多样性、可用数据、精度、时延、调试成本及验收约束比较。模块化便于逐层诊断和复用已知几何/控制结构；VLA 可学习语言与视觉条件到动作的映射，但泛化与恢复需实测。可以组合高层模型、技能/策略和底层控制，先定义接口与故障归属，再以同任务、同预算比较成功率和成本。

**易错**：VLA 不必只负责高层规划，也并非天然解决长尾；端到端策略可以输出关节目标，各层控制频率不能统一写死。

**核验**：[Helix 02 官方说明](https://www.figure.ai/news/helix-02)、[PyRUA-Lean 原论文](https://arxiv.org/abs/2610.01939)。本题频次沿用历史记录，本轮只修订答案。

</details>

<details class="qa" id="q36">
<summary><span class="lv lv-l3">L3</span> <span class="freq">🔥×5</span> <b>Q36</b> · 如果给你 2 台 A100（80GB × 8）、3 个月，从零搭建一套 Humanoid VLA pipeline，你的规划是什么？</summary>

**答**：先澄清“2台”指服务器还是GPU，以及是否每台8张80GB A100；再确认本体、数据、团队、可用时段与交付范围。第一阶段跑通采集—训练—回放—受控真机基线；第二阶段围绕失败类型补数据与适配模型；第三阶段固定保留任务做鲁棒性、恢复和部署验收。按实测显存、吞吐与数据质量选择模型和微调方法。

**易错**：资源数量和交付范围未清楚前不能承诺成功率；桌面双臂原型不等于全身 humanoid pipeline，参数量也不能单独确定能否全参微调。

**核验**：[OpenVLA 原论文](https://arxiv.org/html/2406.09246)、[TorchData StatefulDataLoader](https://github.com/meta-pytorch/data/blob/main/torchdata/stateful_dataloader/README.md)。本题频次沿用历史记录，本轮只修订答案。

</details>

<details class="qa" id="q37">
<summary><span class="lv lv-l3">L3</span> <span class="freq">🔥×5</span> <b>Q37</b> · 讨论未来几年的具身技术路线时，怎样区分证据、假设与验证计划？</summary>

**答**：先定义要解决的任务、成本和可靠性目标，再区分已发表结果、尚待验证的假设与自己的路线选择。示教预训练、RL 后训练、仿真/合成数据及模型编排已有不同设定的研究；可选择其中一条提出可测里程碑，如保留场景成功率、人工介入率与每次成功成本，并解释失败时的替代方案。

**易错**：不要把发布演示称为普遍 SOTA，也不要给无人数据飞轮或世界模型落地写确定年份；预印本结论只能在其任务、接口和预算下比较。

**核验**：[RECAP 原论文](https://arxiv.org/abs/2511.14759)、[PyRUA-Lean 原论文](https://arxiv.org/abs/2610.01939)。本题频次沿用历史记录，本轮只修订答案。

</details>

<details class="qa" id="q38">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×4</span> <b>Q38</b> · 你认为具身智能当前最难突破的工程瓶颈是什么？怎么理解？</summary>

**答**：围绕具体系统选瓶颈，给出测量：数据覆盖看保留场景与失败簇；实时性看观测年龄和延迟分位数；长程可靠性看完成率、恢复率、人工介入与失败步骤。再设计单因素改进或消融，报告成本和剩余限制。数据、模型、控制、硬件和系统集成都可能成为主要约束。

**易错**：不能用未经测量的演示数量、采集工时或“当前最好系统成功率”充当行业事实；算力和带宽也可能是真实瓶颈，不能先验排除。

**核验**：[Thea 机器人 Harness 报告](https://eit-hai.github.io/thea/index.html)、[OpenVLA-OFT 项目](https://openvla-oft.github.io/)。本题频次沿用历史记录，本轮只修订答案。

</details>

<details class="qa" id="q39">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×3</span> <b>Q39</b> · 如何向非 AI 背景的机械 / 电气 / 产品同学解释 VLA 的局限性？跨部门沟通的要点是什么？</summary>

**答**：按协作者关心的指标解释：机械侧谈可达空间、负载、接触和实测精度；电气/控制侧明确指令类型、单位、时间戳、超时与停止语义；产品侧给出已验证任务、环境、失败模式及人工介入条件。用成功和失败样例、接口文档及同一验收集建立共同预期。

**易错**：VLA 没有通用的毫米精度、100ms 延迟或成功率；数字必须附具体本体、任务、版本与测试条件。“可继续采数据训练”也不等于上线后会自动学会。

**核验**：[Helix 02 官方说明](https://www.figure.ai/news/helix-02)、[ROS 2 实时系统设计](https://design.ros2.org/articles/realtime_background.html)。本题频次沿用历史记录，本轮只修订答案。

</details>

---

## §7 本次公开面经增补：Agent / Harness / 平台（12 题）

> 以下是具身企业前端、全栈、平台或数据岗位的单源原帖实录；不代表机器人策略算法岗高频。原帖创建/编辑年份已用公开结构化字段核实；面试年份原文未明示。

<details class="qa" id="q40">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q40</b> · Agent 的 Harness 应承担哪些职责，怎样与模型分工？</summary>

**答**：模型负责给出候选决策；Harness 管理工具发现与调用、上下文、任务状态、执行循环、权限与预算，并记录可复查的运行轨迹。按可测试接口拆开，模型或工具替换时仍能定位失败。若连接机器人，还要把状态观测、动作执行和任务完成验证接入闭环。

**易错**：Harness 没有唯一固定分层；不能把换一段提示词等同于已经实现可靠的执行系统。

**题源**：[公开面经原帖](https://www.nowcoder.com/feed/main/detail/0bf23431c64b495ea22deefc575c1034?sourceSSR=dynamic)；具身企业前端／平台岗，非机器人策略算法岗；原帖创建 2026-09-03（公开 HTML 结构化记录核实）；面试年份未核。

**核验**：[Thea 机器人 Harness 报告](https://eit-hai.github.io/thea/index.html)、[LangGraph 工作流与 Agent](https://docs.langchain.com/oss/python/langgraph/workflows-agents)。

</details>

<details class="qa" id="q41">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q41</b> · 怎样定义工具输入、返回结果和失败信息，便于 Agent 正确使用？</summary>

**答**：输入用明确的类型、单位、必填项和约束，并在运行时验证；返回区分业务结果、执行状态和结构化错误，附请求标识与必要证据。说明超时、取消、重复调用和重试语义；工具描述告诉模型何时适用，运行时负责实际检查。

**易错**：JSON 合法或工具调用成功不等于业务目标完成；运动指令已接收也不证明物体已经抓稳。

**题源**：[公开面经原帖](https://www.nowcoder.com/feed/main/detail/0bf23431c64b495ea22deefc575c1034?sourceSSR=dynamic)；具身企业前端／平台岗，非机器人策略算法岗；原帖创建 2026-09-03（公开 HTML 结构化记录核实）；面试年份未核。

**核验**：[MCP 工具规范](https://modelcontextprotocol.io/specification/2026-07-28/server/tools)、[Thea 机器人 Harness 报告](https://eit-hai.github.io/thea/index.html)。

</details>

<details class="qa" id="q42">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q42</b> · 短期上下文与长期记忆各存什么，怎样控制增长和错误传播？</summary>

**答**：短期状态保存当前目标、已执行步骤、最近观测与待解决问题；长期记忆保存跨任务可复用且有来源的信息。用结构化摘要、按需检索和过期策略限制上下文，保留来源、时间与版本。写入前验证，发现冲突时重新观测或查询，而非把旧摘要直接当事实。

**易错**：长期记忆不必全放向量库；模型猜测、工具错误和已经过期的机器人位置不应自动升级为可信记忆。

**题源**：[公开面经原帖](https://www.nowcoder.com/feed/main/detail/0bf23431c64b495ea22deefc575c1034?sourceSSR=dynamic)；具身企业前端／平台岗，非机器人策略算法岗；原帖创建 2026-09-03（公开 HTML 结构化记录核实）；面试年份未核。

**核验**：[LangGraph 记忆说明](https://docs.langchain.com/oss/python/concepts/memory)。

</details>

<details class="qa" id="q43">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q43</b> · ReAct 和先规划再执行的 Agent，应该怎样选择与组合？</summary>

**答**：ReAct 根据当前信息选择动作，再用观测决定下一步，适合执行中不断获得新信息；先规划再执行先形成子任务及依赖，再交给执行器，便于跟踪长任务。实际可先规划、逐步执行，在前置条件不成立或结果偏离时局部修订计划。

**易错**：先规划不意味着计划必须一路不变；观测也不能只读模型自己的文字总结，应保留工具和环境反馈。

**题源**：[公开面经原帖](https://www.nowcoder.com/feed/main/detail/0bf23431c64b495ea22deefc575c1034?sourceSSR=dynamic)；具身企业前端／平台岗，非机器人策略算法岗；原帖创建 2026-09-03（公开 HTML 结构化记录核实）；面试年份未核。

**核验**：[LangGraph 工作流与 Agent](https://docs.langchain.com/oss/python/langgraph/workflows-agents)、[ReAct 原论文](https://arxiv.org/abs/2210.03629)。

</details>

<details class="qa" id="q44">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q44</b> · 怎样让 Agent 的调用—观测循环及时结束，避免卡死和反复调用？</summary>

**答**：每轮记录目标、调用及结果，由明确的完成条件决定成功退出；另设步数、时间、token 和重试预算。对重复动作、状态无进展、连续失败及观测缺失做检测，触发恢复、降级或失败退出。日志要能区分模型决策问题与工具一直未完成。

**易错**：“模型说完成”不是充分验收；预算耗尽应报告未完成，不能当成任务成功。

**题源**：[公开面经原帖](https://www.nowcoder.com/feed/main/detail/0bf23431c64b495ea22deefc575c1034?sourceSSR=dynamic)；具身企业前端／平台岗，非机器人策略算法岗；原帖创建 2026-09-03（公开 HTML 结构化记录核实）；面试年份未核。

**核验**：[Thea 机器人 Harness 报告](https://eit-hai.github.io/thea/index.html)、[LangGraph 工作流与 Agent](https://docs.langchain.com/oss/python/langgraph/workflows-agents)。

</details>

<details class="qa" id="q45">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q45</b> · 什么情况下需要 Agent，什么时候把查询流程沉淀成脚本或工具？</summary>

**答**：先判断输入和执行路径是否稳定。可枚举、可验证的固定流程优先编码为工具或工作流；需要结合新上下文选择工具、调整步骤时再引入 Agent。把反复成功的查询封装成有契约的可测试工具，保留参数化和失败处理，让 Agent 负责选择与组合。

**易错**：沉淀工具不是照搬一次聊天记录；应剥离临时状态、硬编码凭据和偶然成功的假设，并用回归用例验证。

**题源**：[公开面经原帖](https://www.nowcoder.com/feed/main/detail/aa9d624ef29445b6b62ae2aa25695b2f?sourceSSR=dynamic)；具身企业全栈／平台岗，非机器人策略算法岗；原帖创建 2026-08-13，编辑/展示 2026-08-28（公开 HTML 结构化记录核实）；面试年份未核。

**核验**：[LangGraph 工作流与 Agent](https://docs.langchain.com/oss/python/langgraph/workflows-agents)。

</details>

<details class="qa" id="q46">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q46</b> · 高层已经生成 plan，落到机器人关节动作还需要哪些接口与反馈？</summary>

**答**：先把目标绑定到可观测对象和状态，检查技能前置条件；技能或策略生成末端目标、关节目标或轨迹，按输出类型接入适配器、必要的运动规划／IK 与控制器执行。回传执行进度、异常和新观测，并独立检查目标是否达成，失败时重新规划或恢复。

**易错**：任务规划的“抓杯子”、末端目标和电机命令是不同抽象层；具体系统可合并部分环节，但接口、坐标系与完成语义必须明确。

**题源**：[公开面经原帖](https://www.nowcoder.com/feed/main/detail/aa9d624ef29445b6b62ae2aa25695b2f?sourceSSR=dynamic)；具身企业全栈／平台岗；问题明确涉及机器人执行；原帖创建 2026-08-13，编辑/展示 2026-08-28（公开 HTML 结构化记录核实）；面试年份未核。

**核验**：[Thea 机器人 Harness 报告](https://eit-hai.github.io/thea/index.html)、[PyRUA-Lean 原论文](https://arxiv.org/abs/2610.01939)。

</details>

<details class="qa" id="q47">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q47</b> · Agent 项目如何报告功能测试成功率，避免只展示成功样例？</summary>

**答**：先定义一个完整任务何时成功，固定测试集、工具环境、模型／提示词版本和调用预算，按任务统计成功、失败和超时。除成功率外记录完成时间、调用成本及失败类别；重复随机运行时说明次数和聚合方式，保留失败轨迹。迭代与最终验收使用分开的样本。

**易错**：工具返回率、单步正确率和端到端任务成功率不同；不能让同一个模型的自评代替可验证结果，也不能挑最好一次代表全部。

**题源**：[公开面经原帖](https://www.nowcoder.com/feed/main/detail/90412c695d64469699c0e1d05f7fd153)；具身企业数据引擎岗；原问为数据分析 Agent 项目；原帖创建 2026-09-04，编辑/展示 2026-09-09（公开 HTML 结构化记录核实）；面试年份未核。

**核验**：[Thea 机器人 Harness 报告](https://eit-hai.github.io/thea/index.html)。

</details>

<details class="qa" id="q48">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q48</b> · MCP 与 Skill 解决的问题有什么不同，能怎样一起使用？</summary>

**答**：MCP 规范客户端与服务端发现和调用工具等能力的交互；Skill 通常把任务说明、流程和可复用资源组织成可加载的能力包。Skill 可以指导何时调用 MCP 暴露的工具，工具执行仍经过宿主运行时。比较时先约定这里的 Skill 指哪种实现。

**易错**：注册了工具不等于模型一定会正确选择；Skill 文本也不自动赋予权限或保证步骤执行顺序。

**题源**：[公开面经原帖](https://www.nowcoder.com/feed/main/detail/0710cc2eecc9444299dc38be78de3f76)；具身企业平台开发岗；与千寻全栈二面同一求职流程；原帖创建 2026-08-08，编辑/展示 2026-08-26（公开 HTML 结构化记录核实）；面试年份未核。

**核验**：[MCP 工具规范](https://modelcontextprotocol.io/specification/2026-07-28/server/tools)、[Agent Skills 规范](https://agentskills.io/specification)。

</details>

<details class="qa" id="q49">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q49</b> · Skill 只写在文本里，怎样判断并约束 Agent 是否遵循流程？</summary>

**答**：将必须满足的条件落实为运行时校验、状态机或工作流门控，文本负责解释目的和可选策略；每一步记录输入、调用、结果和状态转移，用正常与异常用例做回归。允许偏离的部分预先界定，例如工具等价替换；关键前置条件与验收要求不能仅靠模型自觉。

**易错**：提示词遵循率不是执行保证；把所有偏离都视为错误也会损失灵活性，需区分硬约束和可调整步骤。

**题源**：[公开面经原帖](https://www.nowcoder.com/feed/main/detail/0710cc2eecc9444299dc38be78de3f76)；具身企业平台开发岗；与千寻全栈二面同一求职流程；原帖创建 2026-08-08，编辑/展示 2026-08-26（公开 HTML 结构化记录核实）；面试年份未核。

**核验**：[Agent Skills 规范](https://agentskills.io/specification)、[LangGraph 工作流与 Agent](https://docs.langchain.com/oss/python/langgraph/workflows-agents)。

</details>

<details class="qa" id="q50">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q50</b> · 多 Agent 之间怎样交接上下文，减少信息丢失和状态冲突？</summary>

**答**：把任务标识、目标、已确认事实、待办、工具结果引用和版本组成显式交接状态；必要上下文随任务传递，大文件与长轨迹按引用读取。区分共享状态与各 Agent 的局部上下文，对并行更新规定所有权或合并规则，交接后检查关键信息是否齐全。

**易错**：共享同一个长聊天记录不等于状态一致；摘要可能漏约束，旧版本结果也不能直接覆盖较新的执行事实。

**题源**：[公开面经原帖](https://www.nowcoder.com/feed/main/detail/1d05dbe48b314a81804c2ffdd51ab569)；具身企业 Agent 平台开发岗，非机器人策略算法岗；原帖创建 2026-08-05，编辑/展示 2026-08-26（公开 HTML 结构化记录核实）；面试年份未核。

**核验**：[LangGraph 记忆说明](https://docs.langchain.com/oss/python/concepts/memory)、[RegenHarness 原论文](https://arxiv.org/abs/2609.27612)。

</details>

<details class="qa" id="q51">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q51</b> · Agent 工具失败时怎样维护状态、分类错误并决定是否重试？</summary>

**答**：先记录实际已发生的操作及当前状态。连接超时等暂态故障可在预算内退避重试；参数或权限错误应先修正条件；副作用是否发生不明时先查询确认。写操作采用请求标识和去重机制，不能保证幂等的动作需要恢复策略，失败记录要进入追踪与回归。

**易错**：超时不代表操作没发生；重复运动或重复写入可能产生额外副作用，不能把“全部重试三次”当通用容错。

**题源**：[公开面经原帖](https://www.nowcoder.com/feed/main/detail/1d05dbe48b314a81804c2ffdd51ab569)；具身企业 Agent 平台开发岗，非机器人策略算法岗；原帖创建 2026-08-05，编辑/展示 2026-08-26（公开 HTML 结构化记录核实）；面试年份未核。

**核验**：[MCP 工具规范](https://modelcontextprotocol.io/specification/2026-07-28/server/tools)、[RegenHarness 原论文](https://arxiv.org/abs/2609.27612)。

</details>

---

## §8 本次公开面经增补：训练与真机现场（3 题）

> 通用后训练与机器人软件岗位的具体追问；单源实录，不伪造频次。

<details class="qa" id="q52">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q52</b> · DDP 在 epoch 中途恢复训练，怎样尽量延续原来的采样顺序？</summary>

**答**：除模型外，保存优化器、调度器、随机数、训练步与梯度累积状态，以及各 rank 的 sampler／dataset／DataLoader 进度。分片、种子、shuffle epoch 和数据版本必须对应。可用 StatefulDataLoader 保存迭代状态，但仍需各 rank 协调；恢复后按 sampler 的补齐／丢尾规则核对样本 ID。

**易错**：只保存 batch 下标或调用 set_epoch 不够；worker 预取、随机增强、world size 改动都会影响复现，不能无条件保证逐位一致，也要区分预期的补齐重复与异常重放。

**题源**：[公开面经原帖](https://www.nowcoder.com/feed/main/detail/0f6cc57190264448bf1647dc36de4622)；具身企业通用大模型后训练实习岗，非 VLA 策略后训练专项；原帖创建 2026-02-10（公开 HTML 结构化记录核实）；面试年份未核，属旧帖补缺，非本季度面经证明。

**核验**：[TorchData StatefulDataLoader](https://github.com/meta-pytorch/data/blob/main/torchdata/stateful_dataloader/README.md)。

</details>

<details class="qa" id="q53">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q53</b> · 机器人运行中一只机械臂突然不动，怎样逐层定位？</summary>

**答**：先确认现场安全并保存故障时刻日志；从急停、安全状态、电源和驱动报警查起，再查总线链路、设备状态与周期数据。沿命令生成、发送、驱动接收、实际反馈逐层比对，检查是否限位、超时或保护触发。用同一时间线关联其他关节和传感器，定位第一处异常。

**易错**：“模型还在输出”不证明命令已到驱动；盲目重启会丢失现场，也可能使尚未结束的命令重新生效。

**题源**：[公开面经原帖](https://ac.nowcoder.com/discuss/1710893?type=0)；机器人软件工程岗；原帖为真机现场排查；原帖创建 2026-10-14（公开 HTML 结构化记录核实）；面试年份未核。

**核验**：[Beckhoff 诊断说明（TwinCAT 示例）](https://infosys.beckhoff.com/content/1033/tcsystemmanager/1089009035.html)、[ROS 2 实时系统设计](https://design.ros2.org/articles/realtime_background.html)。

</details>

<details class="qa" id="q54">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q54</b> · 相机偶发丢帧或没有图像，怎样区分采集、通信和处理故障？</summary>

**答**：沿相机采集、驱动缓冲、编码传输、订阅回调到推理消费逐段测帧号、时间戳、频率与队列积压。先看原始流是否存在，再核对带宽、CPU/GPU 负载、复制和解码开销，以及 ROS 2 QoS 是否兼容。区分采集时间与到达时间，回放复现并关联控制异常。

**易错**：增大缓存可能只把丢帧变成更大延迟；收到旧图不等于拥有当前观测，重传可靠性也不等于实时性。

**题源**：[公开面经原帖](https://ac.nowcoder.com/discuss/1710893?type=0)；机器人软件工程岗；原帖为相机链路与同步排查；原帖创建 2026-10-14（公开 HTML 结构化记录核实）；面试年份未核。

**核验**：[ROS 2 QoS 设计](https://design.ros2.org/articles/qos.html)。

</details>

---

## 前沿题迁移入口

Jev、Astra、具身 Harness 与 RSI 的学习题已独立为[卷九 · 🔥 近期热门方向与前沿题目](09_frontier.html)。

<span id="f01"></span>

[F01 → 卷九对应题目](09_frontier.html#f01)

<span id="f02"></span>

[F02 → 卷九对应题目](09_frontier.html#f02)

<span id="f03"></span>

[F03 → 卷九对应题目](09_frontier.html#f03)

<span id="f04"></span>

[F04 → 卷九对应题目](09_frontier.html#f04)

<span id="f05"></span>

[F05 → 卷九对应题目](09_frontier.html#f05)

<span id="f06"></span>

[F06 → 卷九对应题目](09_frontier.html#f06)

<span id="f07"></span>

[F07 → 卷九对应题目](09_frontier.html#f07)

<span id="f08"></span>

[F08 → 卷九对应题目](09_frontier.html#f08)

---

## 低频备选（频次 1-2，未入主表）

> 以下为旧版低频备选，未纳入主表题号；原始次数与题源本轮未重核。

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq" title="旧版记录 2 次，本轮未复核">历史低频·待核</span> <b>N1</b> · ONNX 模型导出常见坑：动态 shape / 自定义 op / 控制流？</summary>

**答**：**常见坑**：① **动态 shape**：PyTorch 默认导出固定 shape，VLA 输入 token 数变化 → 加 `dynamic_axes` 参数或 `torch.onnx.export` 时指定动态轴；② **自定义 CUDA op**（如 Flash Attention）：ONNX 不认识自定义算子 → 需注册为 custom op 或替换为 ONNX 原生 attention；③ **Python 控制流**（if/for）：`torch.jit.script` 要求显式类型，动态分支需改写；④ **int64 索引**：部分 ONNX runtime 不支持 int64 → 转 int32。

**排查工具**：`onnxruntime.InferenceSession` 验证导出模型；`Netron` 可视化计算图。

**易错**：不要假设 PyTorch → ONNX → TensorRT 一气呵成；每个转换步骤都可能有算子不兼容，需要逐步验证。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq" title="旧版记录 2 次，本轮未复核">历史低频·待核</span> <b>N2</b> · 机器人 CI/CD：如何为策略代码建立自动化测试与发布流程？</summary>

**答**：**核心挑战**：策略测试需要物理环境，单测无法覆盖真机行为。

**分层测试**：① **单元测试**：策略网络 forward/inference shape 检查（pytest）；② **仿真回归测试**：用 Isaac Lab / MuJoCo 跑固定场景，成功率回归（> baseline × 0.9）；③ **端到端仿真 rollout**：提交 PR 时自动在仿真器中跑 10 次评估；④ **人工真机验收**：重大版本才跑真机（成本高）。

**发布流程**：模型版本控制（W&B / MLflow）→ 量化（TensorRT）→ 灰度部署（先 1 台，再 N 台）→ 监控 rollback 指标（任务成功率 / 急停次数）。

**易错**：策略测试不能完全替代真机验证；仿真 CI 是必要条件，不是充分条件。

</details>

<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq" title="旧版记录 2 次，本轮未复核">历史低频·待核</span> <b>N3</b> · 开源贡献与 PR 流程：在具身智能开源项目中如何快速做出有价值的贡献？</summary>

**答**：**快速贡献路径**：① **文档/示例**：门槛最低，阅读文档发现 gap → 补充 example notebook / README；② **Bug fix**：跑通 quickstart 发现复现问题 → 提 issue + 附 MRE（Minimum Reproducible Example）→ PR；③ **算法改进**：阅读 issue tracker 找 "help wanted" 标签 → 先 issue 讨论方案，被接受后开 PR（避免无效劳动）。

**PR 要点**：小 PR 比大 PR 容易 review（单一功能）；必须附测试（pytest/rollout 结果）；描述清楚"为什么改"不只是"改了什么"；遵循 code style（`black`/`flake8` 预先过一遍）。

**具身 AI 开源项目**：Lerobot（Hugging Face）/ LeRobot、OpenVLA、GR00T、BehaviorTree.CPP、Nav2。

**易错**：不要直接提"我能实现 X 大功能" PR，先问 maintainer 是否在路线图上；避免重复造轮子。

</details>

---

## §H 手撕代码（8 题）

> 数值稳定与训练工程手撕段——与上文"概念+答案"题区分开。本节每题给"考察点 / 实现 / 易错"——附 ≤30 行 Python 实现。

<details class="qa qa-handcoding">
<summary><span class="lv lv-l1">L1</span> <span class="freq">🔥×8</span> <b>✍ H01</b> · 手撕数值稳定 softmax</summary>

**考察点**：减 max 防 `exp` 溢出；`keepdim=True` 保证 broadcast 正确；float32 仍可能极端 -inf 输入致除零。

**实现**：

```python
import torch

def softmax_stable(x, dim=-1):
    # 减 max 后所有元素 ≤ 0，exp 不会上溢
    x_max = x.max(dim=dim, keepdim=True).values
    exp_x = (x - x_max).exp()
    # keepdim=True 让 sum 形状与 exp_x 可广播
    return exp_x / exp_x.sum(dim=dim, keepdim=True)
```

**易错**：忘 `keepdim` 致广播错；全 -inf 输入（如整行 padding）exp=0、sum=0 → 除零，应在 mask 时先过滤。

</details>

<details class="qa qa-handcoding">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×5</span> <b>✍ H02</b> · 手撕 LogSumExp / log_softmax</summary>

**考察点**：`log Σ exp(x) = x_max + log Σ exp(x - x_max)`；log_softmax 必须一步算（不要 `log(softmax(x))`）。

**实现**：

```python
import torch

def logsumexp(x, dim=-1):
    x_max = x.max(dim=dim, keepdim=True).values
    # log-sum-exp 标准式：把 max 提出 exp 防上溢
    return x_max.squeeze(dim) + (x - x_max).exp().sum(dim=dim).log()

def log_softmax(x, dim=-1):
    # 等价 x - logsumexp(x)；一步算避免 log(softmax) 在 prob≈0 处 log(0) = -inf
    x_max = x.max(dim=dim, keepdim=True).values
    shifted = x - x_max
    return shifted - shifted.exp().sum(dim=dim, keepdim=True).log()
```

**易错**：用 `log(softmax(x))`；忘 `keepdim` 致广播错；float16 下 LSE 仍可能上溢，应用 float32 中间量。

</details>

<details class="qa qa-handcoding">
<summary><span class="lv lv-l1">L1</span> <span class="freq">🔥×7</span> <b>✍ H03</b> · 手撕 cross-entropy（含数值稳定）</summary>

**考察点**：CE = NLL + log_softmax 的组合；padding 用 `ignore_index` 屏蔽；soft label 走另一公式。

**实现**：

```python
import torch
import torch.nn.functional as F

def cross_entropy_hard(logits, target, ignore_index=-100):
    # logits: [B, C]；target: [B]，padding 处填 -100 屏蔽
    log_p = F.log_softmax(logits, dim=-1)        # 一步算，避免 log(softmax)
    nll = -log_p.gather(-1, target.clamp(min=0).unsqueeze(-1)).squeeze(-1)
    mask = (target != ignore_index).float()
    # mean 必须按有效样本数归一，否则 ignore 多时被稀释
    return (nll * mask).sum() / mask.sum().clamp(min=1.0)

def cross_entropy_soft(logits, soft_target):
    # soft_target: [B, C] 概率分布（label smoothing / 蒸馏常用）
    log_p = F.log_softmax(logits, dim=-1)
    return -(soft_target * log_p).sum(-1).mean()
```

**易错**：先 softmax 再 log 数值不稳；padding 未 mask 拉偏 loss；`reduction='mean'` 但分母用 batch 大小（应用有效样本数）。

</details>

<details class="qa qa-handcoding">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×5</span> <b>✍ H04</b> · 手撕 KL divergence（离散 / 高斯 / forward vs reverse）</summary>

**考察点**：KL 不对称（forward mode-covering / reverse mode-seeking）；高斯闭式；`F.kl_div` 首参约定 `log_q`。

**实现**：

```python
import torch
import torch.nn.functional as F

def kl_discrete(log_p, log_q):
    # 离散 KL(p‖q) = Σ p·(log p - log q)；用 log_prob 入参数值稳
    # p=0 处约定 0·log(0/q)=0；mask 掉 log_p=-inf 项避免 0·inf=NaN
    p = log_p.exp()
    term = p * (log_p - log_q)
    return term.masked_fill(log_p == float('-inf'), 0.0).sum(-1)

def kl_gauss_to_std(mu, logvar):
    # VAE 特例：KL(N(μ,σ²) ‖ N(0,I)) 闭式
    # 0.5 · Σ(μ² + σ² - 1 - log σ²)
    return 0.5 * (mu.pow(2) + logvar.exp() - 1 - logvar).sum(-1)

def kl_via_F(log_q, p_prob):
    # PyTorch 约定：第一参传 log_q，第二参传 p（prob，非 log）
    # 等价 Σ p·(log p - log q)
    return F.kl_div(log_q, p_prob, reduction='batchmean')
```

**易错**：log 用 σ 还是 σ²（差一倍）；`F.kl_div` 首参传 prob 而非 log_prob；forward/reverse 方向反；离散输入 p,q 都用 prob 致 `log(0)`。

</details>

<details class="qa qa-handcoding">
<summary><span class="lv lv-l1">L1</span> <span class="freq">🔥×9</span> <b>✍ H05</b> · 手撕 mini-batch 训练循环</summary>

**考察点**：`zero_grad → forward → backward → step` 顺序；梯度累加 + AMP 顺序；推理用 `eval()` + `no_grad()`。

**实现**：

```python
import torch
from torch.cuda.amp import autocast, GradScaler

def train_epoch(model, loader, opt, crit, device, accum=1):
    model.train()
    scaler = GradScaler()
    opt.zero_grad(set_to_none=True)  # set_to_none 比置 0 更省显存
    for i, (x, y) in enumerate(loader):
        x, y = x.to(device), y.to(device)
        with autocast():
            loss = crit(model(x), y) / accum         # 梯度累加要先除步数
        scaler.scale(loss).backward()                # AMP loss 先 scale 再 backward
        if (i + 1) % accum == 0:
            scaler.step(opt); scaler.update()
            opt.zero_grad(set_to_none=True)          # step 之后清梯度

    # 处理 loader 长度非 accum 整数倍：末尾残余梯度也 step
    if len(loader) % accum != 0:
        scaler.step(opt); scaler.update()
        opt.zero_grad(set_to_none=True)

@torch.no_grad()
def evaluate(model, loader, crit, device):
    model.eval()  # 关 dropout / 停 BN running 更新
    return sum(crit(model(x.to(device)), y.to(device)) for x, y in loader)
```

**易错**：`zero_grad` 漏 → 梯度累加错；忘 `model.eval()` BN/Dropout 异常；`no_grad` 没包显存炸；AMP loss 没 scale。

</details>

<details class="qa qa-handcoding">
<summary><span class="lv lv-l1">L1</span> <span class="freq">🔥×4</span> <b>✍ H06</b> · 手撕 warmup + cosine 学习率 schedule</summary>

**考察点**：warmup 防初始 lr 过大；cosine 末期渐降；边界 `t=t_w` 处 lr 必须连续（=lr_max）。

**实现**：

```python
import math
from torch.optim.lr_scheduler import LambdaLR

def warmup_cosine(opt, t_warm, t_total, min_ratio=0.0):
    # t_total 用 step 数而非 epoch（step 粒度更细）
    def lr_lambda(t):
        if t < t_warm:
            # 线性 warmup：t=0 → 0；t=t_warm → 1（边界连续）
            return float(t) / max(1, t_warm)
        # cosine 末期：t=t_warm → 1；t=t_total → min_ratio
        progress = (t - t_warm) / max(1, t_total - t_warm)
        cos = 0.5 * (1.0 + math.cos(math.pi * progress))
        return min_ratio + (1.0 - min_ratio) * cos
    return LambdaLR(opt, lr_lambda)
```

**易错**：`t_warm=0` 致除零；`t_total` 用 epoch 数末期未降到底；warmup 公式从 1 开始致 `t=0` 时 `lr=lr_max`；边界 `t=t_warm` lr 跳变。

</details>

<details class="qa qa-handcoding">
<summary><span class="lv lv-l2">L2</span> <span class="freq">🔥×5</span> <b>✍ H07</b> · 手撕 Adam / AdamW 更新规则</summary>

**考察点**：一阶/二阶 momentum + bias correction；AdamW 解耦 weight decay（不混进 momentum）。

**实现**：

```python
import torch

def adamw_step(params, grads, state, lr=1e-3, betas=(0.9, 0.999),
               eps=1e-8, wd=1e-2):
    # state[i] = {'m': ..., 'v': ..., 't': int}；外层维护
    b1, b2 = betas
    for p, g, s in zip(params, grads, state):
        s['t'] += 1
        # 一阶/二阶 momentum 指数滑动
        s['m'].mul_(b1).add_(g, alpha=1 - b1)
        s['v'].mul_(b2).addcmul_(g, g, value=1 - b2)
        # bias correction：消除初期 m,v 偏向 0 的偏置
        m_hat = s['m'] / (1 - b1 ** s['t'])
        v_hat = s['v'] / (1 - b2 ** s['t'])
        # AdamW：weight decay 直接乘 p，不进 m/v
        # （若改为 Adam，把 wd*p 那项加到 g 里、再走主更新即为 L2 正则）
        p.data.mul_(1 - lr * wd).addcdiv_(m_hat, v_hat.sqrt().add_(eps), value=-lr)
```

**易错**：忘 bias correction（初期步太小）；把 `wd*p` 加到 grad（变 L2 正则非 AdamW）；`v̂` 算根之前没存独立副本，多组 param 共享 buffer 时被破坏。

</details>

<details class="qa qa-handcoding">
<summary><span class="lv lv-l1">L1</span> <span class="freq">🔥×3</span> <b>✍ H08</b> · 手撕 dropout（train vs eval + inverted scale）</summary>

**考察点**：train 随机置零并 scale；eval 直接通过；inverted dropout 把 `1/(1-p)` 放训练侧让推理零开销。

**实现**：

```python
import torch
import torch.nn as nn

class Dropout(nn.Module):
    def __init__(self, p=0.5):
        super().__init__()
        assert 0.0 <= p < 1.0
        self.p = p

    def forward(self, x):
        if not self.training or self.p == 0.0:
            return x  # 推理无开销
        # mask ~ Bernoulli(1-p)：保留概率 1-p；用 x.dtype 防 fp16/bf16 被提升
        mask = (torch.rand_like(x) >= self.p).to(x.dtype)
        # inverted dropout：除 (1-p) 让 E[out] = x，无需推理时 scale
        return x * mask / (1.0 - self.p)
```

**易错**：推理还 drop（`F.dropout` 忘传 `training=self.training`）；忘 scale 致 E[out] 不一致；位置放错（残差里 dropout 在 add 之后破坏恒等映射）。

</details>

---
