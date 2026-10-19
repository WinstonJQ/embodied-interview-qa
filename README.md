# 具身智能高频面试题库

> 中文具身智能（Embodied AI）面试题库 · 公开面经选题，另有明确标识的非面经前沿拓展
> *A Chinese-language interview question bank for Embodied-AI engineering roles (VLA / IL / RL / World Models / Engineering).*

[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Live-2ea44f?logo=github)](https://winstonjq.github.io/embodied-interview-qa/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/WinstonJQ/embodied-interview-qa/pulls)
[![Stars](https://img.shields.io/github/stars/WinstonJQ/embodied-interview-qa?style=social)](https://github.com/WinstonJQ/embodied-interview-qa/stargazers)

---

## 📖 在线阅读

**👉 <https://winstonjq.github.io/embodied-interview-qa/>**

打开网页直接看；默认全部折叠，点击题目展开答案；手机端原生支持（地铁公交都能刷）。

**🔥 前沿拓展**：[卷九](docs/interviews/09_frontier.html)收录 Jev／RoboJEV、Astra＋VLA、具身 Agent／Harness 与 RSI 等近期关注方向，明确标注为非面经学习题。

下方截图为历史界面示例，题目与统计以在线题库为准。

### 🖥️ 桌面端预览

<table>
  <tr>
    <td width="33%" align="center" valign="top">
      <img src="images/computer_page_overview_01.png" alt="桌面端主册上半"><br>
      <sub><em>主册 · 上半部（hero + 五卷目录）</em></sub>
    </td>
    <td width="33%" align="center" valign="top">
      <img src="images/computer_page_overview_02.png" alt="桌面端主册下半"><br>
      <sub><em>主册 · 下半部（标签速查 + 使用方式）</em></sub>
    </td>
    <td width="33%" align="center" valign="top">
      <img src="images/computer_page_overview_03.png" alt="桌面端卷子页"><br>
      <sub><em>卷子页 · 左侧 TOC + 「← 返回主册」</em></sub>
    </td>
  </tr>
</table>

### 📱 手机端预览 · 默认折叠 = 自测 + 复盘

看到题目先在脑子里答一遍，答不上来再点开对答案——折叠交互天然就是一种**自测玩法**。

<table>
  <tr>
    <td width="33%" align="center" valign="top">
      <img src="images/mobile_page_overview_01.jpeg" alt="手机端主册"><br>
      <sub><em>手机端主册 · 浏览器直接打开 · 无需 app</em></sub>
    </td>
    <td width="33%" align="center" valign="top">
      <img src="images/mobile_page_overview_02.jpeg" alt="题目折叠态"><br>
      <sub><em>默认折叠 · 只看题目 · 自己先想答案</em></sub>
    </td>
    <td width="33%" align="center" valign="top">
      <img src="images/mobile_page_overview_03.jpeg" alt="题目展开态"><br>
      <sub><em>点开看答案 · 含表格、易错点</em></sub>
    </td>
  </tr>
</table>

---

## 这是什么

具身智能面试涉及 VLA、模仿学习、控制、感知导航与工程系统，公开面经分散在多个平台。本项目按九卷组织复习范围；普通题提供精简答案和“易错”，手撕题保留实现。

面经题以可定位原问的公开经历为依据。论文和官方文档用于核对答案，不能替代面经证明某题被问过。卷九收录 F 编号的前沿学习问答，清楚标为非面经且不计频次。新增实录保留岗位限定：机器人企业的平台、前端或数据岗位，不能直接视作机器人策略算法岗。

---

## 九卷目录

| 卷 | 主题 | 题数 | 一句话 |
|---|---|---:|---|
| [一](docs/interviews/01_basics.html) | **通识基础**（含 §H 手撕） | 55 | DL 基本盘 + RL 入门 + 机器人学 + 手撕（Attention / LayerNorm / Bellman / VAE ELBO） |
| [二](docs/interviews/02_rl_algo.html) | **RL 算法**（含 §H 手撕） | 50 | PPO / SAC / TD3 / 离线 RL / RLHF + 手撕（PPO clipped / GAE / DPO / GRPO 核心公式） |
| [三](docs/interviews/03_vla_il.html) | **VLA / 模仿学习**（含 §H 手撕） | 77 | OpenVLA / π 系列 / Diffusion Policy + Flow Matching + 手撕（BC / DP 训练 step / ACT chunking / CLIP InfoNCE） |
| [四](docs/interviews/04_world_sim.html) | **世界模型 / Sim2Real** | 31 | Dreamer / V-JEPA / 域随机化 / Isaac Lab，人形和四足公司重点 |
| [五](docs/interviews/05_engineering.html) | **工程落地**（含 §H 手撕） | 62 | VLA 部署 / FSDP / teleop 数据飞轮 / Agent 与 Harness 平台实录 + 手撕（数值稳定 softmax / CE / KL / Adam） |
| [六](docs/interviews/06_legged_control.html) | **腿足机器人控制 / 遥操作**（含 §H 手撕） | 55 | 浮动基座、MPC / WBC、RL-locomotion、AVP teleop、HumanPlus、actuator network sim2real + 手撕（PID / KF / EKF / LQR / 四元数 / IK / 互补滤波 / DH / MPC QP） |
| [七](docs/interviews/07_perception_nav.html) | **3D 感知 / SLAM / VLN / ObjectNav / Embodied VLM**（含 §H 手撕） | 67 | 点云 / NeRF / SLAM / Nav2 / HAMT / NaVid / VLFM / SAM 2 + 手撕（IoU / NMS / BFS·DFS / A* / ViT patch） |
| [八](docs/interviews/08_coding_systemdesign.html) | **通用工程：LeetCode + 系统设计** | 40 | LeetCode 高频 30 题 + ML 系统设计 5 + 机器人系统设计 5（所有具身 / 自动驾驶岗通用） |
| [九](docs/interviews/09_frontier.html) | **🔥 近期热门方向与前沿题目** | 8 | Jev / RoboJEV / Astra 机械臂与 VLA / 具身 Agent 与 Harness / RSI；非面经拓展 |

目录口径为**非 N 备选 445 题**（其中 437 道原有/实录题、8 道非面经拓展），另有 **19 题 N 备选**，合计 **464 个问答条目**。其中手撕题共 **84 题**：卷一至七共 54 题，卷八 30 题；卷八另有 10 道系统设计题。目录数字不是“高频题总数”。

---

## 特点

- 🔥 **频次分层**：旧 🔥 沿用历史统计，本轮未全面重核；新高频须有至少三份独立面经，一至两份标“低频实录”。同一候选人同公司多轮、转载和镜像合并计数
- 🔥 **近期热门方向**：卷九标题的火苗表示近期关注主题，不代表企业面试频次或量化热度排名。
- 📚 **前沿拓展**：F 编号和“前沿拓展·非面经”标签表示依据技术资料整理的学习题，面经频次为 0，不称企业真题
- ✏️ **短答案**：普通题目标为 ≤350 字精简答案 + “易错”；手撕题单列，包含代码
- 📱 **折叠交互**：默认全部折起只看题目，点击展开答案；基于 HTML5 `<details>` 原生折叠，零 JS，手机端无需任何 app
- 🔍 **来源可查**：题目下方提供原始面经或论文、官方技术资料链接，便于核对适用范围
- 🏷️ **难度标签**：L1 必会 · L2 进阶 · L3 顶级 lab，方便按层次规划复习
- 🏢 **按主题组织**：新题的题源注记保留公司、岗位和日期确定性，避免把平台岗问法泛化到算法岗

---

## 使用建议

1. **首次浏览**：直接点开[在线题库](https://winstonjq.github.io/embodied-interview-qa/)，挑你最关心的方向（VLA / RL / 工程落地……）
2. **复习顺序**：按 L1 → L2 → L3，同等级内按频次（🔥×N）从高到低
3. **手机端**：地铁公交也能刷，HTML5 折叠原生支持，无需任何 app
4. **快速过场**：面试前 30 分钟，只看每题的"易错"一句，刷过的题快速复盘

---

## ⭐ 觉得有用？

**点个 Star 是对维护者最大的鼓励** —— 也能让算法群里其他在找具身岗的同学更容易发现这个题库。

如果你在面试中真的被考到了里面的题，欢迎来 [Issue](https://github.com/WinstonJQ/embodied-interview-qa/issues) 留言或私信——这是后续频次更新的一手依据。

---

## 贡献

发现答案错误、想补新题、想质疑某条结论——都欢迎 [开 Issue](https://github.com/WinstonJQ/embodied-interview-qa/issues) 或 [提 PR](https://github.com/WinstonJQ/embodied-interview-qa/pulls)。

**贡献新题的简单格式**（在对应卷的 Markdown 末尾追加即可）：

````markdown
<details class="qa">
<summary><span class="lv lv-l2">L2</span> <span class="freq">低频实录·1源</span> <b>Q??</b> · 你的题目？</summary>

**答**：精简答案（≤350 字）。

**易错**：一句话点关键陷阱。

**题源 / 核验**：[原始面经](原帖链接)，公司与岗位，面试日期；年份不明时明确注明。另列核答案所用论文或官方文档。

</details>
````

或者直接在 issue 里描述题目 + 来源（哪个平台 / 帖子链接），维护者负责合并到对应卷。

**贡献题源要求**：提供能读取正文、定位具体原问的原始公开面经。汇编或 GitHub 列表应追溯原帖；教程、招聘要求与论文不能计面经频次。不得提交个人猜测的“我觉得会考”题。

---

## 致谢

HTML 渲染脚本参考 [ARIS · Auto-claude-code-research-in-sleep](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep)。

---

## License

[MIT](LICENSE) — 自由使用、修改、再分发。

如果在你的项目里引用了本题库的内容，**给本仓库点个 ⭐ Star 是最好的引用方式**。
