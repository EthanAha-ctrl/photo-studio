# Benchmark

| 基准               | 游戏                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | 画面与节奏           | 任务数    | 主要考察         | 打分方式               |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- | ------ | ------------ | ------------------ |
| MineDojo-300<br> | Minecraft（单一游戏）<br><br>NVIDIA 等机构于 2022 年发布的 Minecraft 研究框架，原始任务集包含约 1,581 个可程序化判定的任务，分为生存、采集、科技树、战斗四类<br><br>战斗 200 个。 例：「夜晚平原，持铁剑、盾牌、钻石甲，猎杀一只蝙蝠」。击杀目标即成功。<br>- 采集 100 个。 例：「在平原获得一块原木」。背包中出现目标物品即成功。<br><br>每个任务跑 8 次，每局最多 1,000 次决策（每次决策推进 50 毫秒游戏时间，约合 50 秒）。任务短而具体，不涉及「从零挖到钻石」这类长程目标。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | 3D 第一人称，开放世界    | 300    | 采集资源、与怪物战斗   | 成功率：读取游戏内部状态判定     |
| ViZDoom-10       | Doom（1993 年的经典 FPS）<br><br>官方自带 12 张训练用小地图。ViZDoom-10 取其中 10 张<br><br>每张地图跑 50 个固定种子，每局 150 至 2,100 次决策。归一化回报的算法是把每局原始奖励除以该地图的专属系数（例如 Basic 除以 100），再对地图取平均。分数可以为负：开枪不中、死亡都会扣分.<br><br><br>\| 类别   \| 地图                                        \| 玩法                        \|<br>\| ---- \| ----------------------------------------- \| ------------------------- \|<br>\| 精确射击 \| Basic                                     \| 房间对面有一只怪，左右移动后开枪命中        \|<br>\| 精确射击 \| Defend the Center                         \| 站在圆形房间中央原地转身，射击四面涌来的敌人    \|<br>\| 精确射击 \| Defend the Line                           \| 站在房间一端，射击对面逼近的敌人          \|<br>\| 目标导航 \| Deadly Corridor                           \| 穿过两侧有敌人的走廊，拿到尽头的护甲        \|<br>\| 目标导航 \| My Way Home                               \| 在多房间迷宫中找到目标物品             \|<br>\| 移动战斗 \| Battle 1、Battle 2                         \| 在迷宫中边走边打，同时捡弹药与血包         \|<br>\| 移动战斗 \| Deathmatch                                \| 在竞技场中击杀多种敌人，不同敌人分值不同      \|<br>\| 生存   \| Health Gathering、Health Gathering Supreme \| 地面持续掉血，靠不断捡血包活得更久；后者地图更复杂 \|<br> | 3D 第一人称，快节奏射击   | 10 张地图 | 瞄准、走位、导航、生存  | 归一化回报：原始奖励除以地图专属系数 |
| GameWorld        | 34 款浏览器小游戏<br>横跨多款游戏                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | 以 2D 休闲游戏为主     | 170    | 跨游戏泛化，含反应与规划 | 成功率与进度：读取游戏状态判定    |
| OmniGameArena    | 12 款为评测新写的 UE5 游戏<br>横跨多款游戏                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | 3D 实时，含单人、对战、合作 | 12 款游戏 | 对从未公开过的游戏的适应 | 归一化分：每款游戏的进度比例     |

GameWorld 于 2026 年 4 月发布（ECCV 2026），收录 34 款在浏览器里运行的游戏，共 170 个任务，分五类：

| 类别      | 游戏                                                                                                             |
| ------- | -------------------------------------------------------------------------------------------------------------- |
| 跑酷（8）   | Boxel Rebound、Chrome Dino、Cubefield、Edge Surf、Flappy Bird、NS-Shaft、Run 3、Temple Run 2                          |
| 街机（7）   | Breakout、Core Ball、Google Snake、Pac-Man、Rocket League 2D、World's Hardest Game、World's Hardest Game 2           |
| 平台跳跃（8） | Another Gentleman's Adventure、Captain Callisto、Doodle Jump、GeoDash、Mario Game、OvO、Restless Wing Syndrome、Vex 3 |
| 益智（7）   | 2048、Astray、Hextris、Minesweeper、Stack、Tetris、Wordle                                                            |
| 模拟（4）   | Fireboy and Watergirl、Minecraft Clone、Monkey Mart、Wolfenstein 3D                                               |

每个任务由一句自然语言指令加一个量化目标组成，例如 Temple Run 2 的「单局达到 4000 分，且不绊倒、不死亡、不复活」。评测器直接读取游戏状态（分数、坐标、生命数等）判定，不依赖截图或 LLM 裁判。

两个指标：

- 成功率（SR）：是否完全达成目标。
- 进度（PG）：从初始值到目标值完成了多少，取 0 到 1。

==OmniGameArena== 于 2026 年 6 月发布，包含 12 款用虚幻引擎 5 为该基准从零编写的实时 3D 游戏。它们由香港大学、香港中文大学、清华与 LIGHTSPEED（腾讯光子工作室）合作编写，打包在同一个 UE5 构建里，都是规则简单的小型游戏，规模上接近派对游戏的单个关卡。

| 游戏            | 模式  | 玩法                                            | 计分            |
| ------------- | --- | --------------------------------------------- | ------------- |
| ObstacleRun3D | 单人  | 3D 跑酷，躲开物理障碍跑到终点                              | 已跑距离 ÷ 全程     |
| ObstacleRun2D | 单人  | 2D 横版线性关卡                                     | 同上            |
| LastStand     | 单人  | 站在地砖平台上，地砖先变红预警再坠落，尽量不掉下去；WASD 移动、空格跳、镜头可自由旋转 | 存活时间 ÷ 最长时限   |
| MonsterShoot  | 单人  | 找到并射杀敌对单位，同时少挨打                               | 有效伤害 ÷ 敌人总血量  |
| SceneEscape   | 单人  | 完成 NPC 指派的若干任务后逃出场景                           | 完成任务数 ÷ 总数    |
| CueChase      | 单人  | 第三人称探索，找到并激活散布在地图里的隐藏机关                       | 激活数 ÷ 总数      |
| SoloCraft     | 单人  | 收集、加工、交付物品以完成订单                               | 交付价值 ÷ 目标价值   |
| SkyDuel       | 对战  | 一对一战斗                                         | 剩余血量 ÷ 满血     |
| CrystalGuard  | 对战  | 对称攻防：打掉对方水晶核心，守住己方                            | 己方核心血量 ÷ 满血   |
| MidlineClash  | 对战  | 两方在同一空间抢着完成资源订单                               | 己方得分 ÷ 目标分    |
| SharedFloor   | 合作  | 两个智能体共用工作台、订单柜台、垃圾桶完成订单，每交一单订单就轮换             | 团队交付价值 ÷ 团队目标 |
| HandoffRun    | 合作  | 两个角色各有禁入区，须跨区传递物品                             | 同上            |

任务都偏短。 四个基准的单局长度都在几十秒到几分钟量级，长达数小时的完整通关能力，这些基准测不出来。样本量小。 每个「模型 × 游戏」只跑 5 局，论文自己也说各游戏的榜首在不同模型间轮换。主结果在模型思考时冻结游戏。 反应速度不计入分数，这偏离了真实游玩；计入延迟的结果只在附录里报了 4 个模型。Script2Play 论文只用了 5 款单人游戏。 上文引用的 GameScaling 0.361、NitroGen 0.064 等数字都来自这个子集，对战与合作 7 款未覆盖。最好的商用 VLM 平均只有 0.47，有公开权重的最高为 GameScaling-9B 的 0.361，NitroGen 仅 0.064，而且这是在「思考时暂停游戏」的宽松条件下取得的。

---

在 open-weight 范围内，目前证据最强的首选是 Player-1（腾讯 ARC，2026 年 11 月），并列候选是 GameScaling（THUSI-Lab 与米哈游，2026 年 10 月）。闭源系统中，Game-TARS（字节跳动）好。Game-TARS 基于专有模型 Seed-VL-1.5 继续预训练。

| 模型          | 规模 / 骨干                     | 动作空间              | 权重                                                               | 证据                                              | Training                                                   |
| ----------- | --------------------------- | ----------------- | ---------------------------------------------------------------- | ----------------------------------------------- | ---------------------------------------------------------- |
| Player-1    | 9B、35B-A3B / Qwen3.5        | 键鼠，接受语言指令         | [HF](https://huggingface.co/yyyang/Player-1-9B)，Apache-2.0       | MineDojo-300 成功率 38.33%（9B），其余开源基线最高 12.63%     | 示范数据的监督微调，加 16 个合成环境（Player-1-Gym）中带可程序化验证奖励的强化学习          |
| GameScaling | 0.8B–27B / Qwen3.5，共 8 个检查点 | 键鼠，每次输出 200 ms 动作 | [HF](https://huggingface.co/THUSI-Lab/GameScaling)，Apache-2.0    | OmniGameArena 上 9B-S3 得 0.361，为该表中有公开权重的游戏策略最高分 | S1 检查点仅用 Minecraft 数据训练，S3 为混合数据（Minecraft 占 27%）。训练数据未公开。 |
| NitroGen    | 约 0.5B，从头训练                 | 手柄，无语言输入          | [HF](https://huggingface.co/nvidia/NitroGen)                     | 两份第三方评测中均偏低                                     | 在 1,000 余款游戏的 40,000 小时视频上做行为克隆                            |
| GameWAM     | 约 6B / Wan2.2               | 键鼠                | [HF](https://huggingface.co/Yunncheng/gamewam-models)，Apache-2.0 | 只发布了 Minecraft 与 ViZDoom 两个专用检查点，不属于通用模型        |                                                            |
| Open-P2P    | 1.2B，从头训练                   | 键鼠，20 Hz 实时       | 开源                                                               | 两份第三方评测中均为最低一档                                  |                                                            |

Player-1 基线

| 基线 | MineDojo-300 成功率 % | ViZDoom-10 归一化回报 | GameWorld 成功率 / 进度 % |
|---|---|---|---|
| GameWAM | 12.63 | 0.3938 | 5.88 / 16.91 |
| JARVIS-VLA | 6.71 | −0.1458 | 6.47 / 17.61 |
| NitroGen | 0.00 | −0.0793 | 8.82 / 23.41 |
| Open-P2P-1.2B | 0.29 | 0.0602 | 2.35 / 10.37 |
| UI-TARS-1.5-7B | 0.38 | −0.1743 | 5.49 / 19.71 |
| Qwen3.5-9B（未微调） | 9.21 | 0.2975 | 10.39 / 29.22 |
| Qwen3.5-35B-A3B（未微调） | 7.79 | 0.3912 | 13.14 / 33.49 |

Player-1-9B 在 MineDojo-300 上成功率为 38.33%。Player-1 在 GameWorld 上的进度为 34.45%，其 SFT 版本为 31.43%；34.45% 对应哪个尺寸未核实。

### Script2Play 论文的 OmniGameArena（平均归一化分）

| 模型 | 类型 | 分数 |
|---|---|---|
| Script2Play (4B) | 游戏策略（合成数据） | 0.640 |
| Gemini 3.1 Pro Preview | 商用 VLM | 0.473 |
| GPT-5.5 | 商用 VLM | 0.463 |
| Claude Opus 4.6 | 商用 VLM | 0.412 |
| GameScaling-9B-S3 | 游戏策略（人类数据） | 0.361 |
| GameScaling-0.8B-S3 | 游戏策略（人类数据） | 0.168 |
| GameScaling-2B-S3 | 游戏策略（人类数据） | 0.135 |
| Qwen3.5-397B-A17B | 开源 VLM | 0.106 |
| NitroGen | 游戏策略（人类数据） | 0.064 |
| Open-P2P | 游戏策略（人类数据） | 0.037 |

Script2Play 由 Qwen3.5-4B 在 150.3 小时合成 Minecraft 与 Roblox 数据上做行为克隆得到。它的权重是否公开就不知道了，论文正文与检索均未见发布链接。

## 观察

1. 通用 VLM 骨干加游戏后训练已胜过从头训练的专用策略。 未经微调的 Qwen3.5-35B-A3B 在 GameWorld（34 款浏览器游戏，170 个任务）上已超过 NitroGen、Open-P2P 与 GameWAM。
2. Player-1 在未见过的游戏上增益有限。 它在 GameWorld 上相对未微调骨干的提升很小，优势主要体现在 Minecraft 与 ViZDoom。
3. 动作空间决定适用范围。 手柄类动作游戏可用 NitroGen；键鼠游戏选 Player-1 或 GameScaling。

## 证据缺口

- Player-1 与 GameScaling 互不出现在对方的评测表中。
- 两份评测均为各论文作者自评，尚无独立第三方排行榜；GameWorld 与 GTAGym 的逐模型公开榜单未检索到。
- Player-1 与 GameScaling 的显存需求在模型页上未注明；GameScaling-27B 的示例命令使用两块 GPU。
- Player-1 论文 HTML 全文的最后约 15,000 字符、Script2Play 论文的结果表以外部分未逐字通读。

---

mon père, General Intuition (通用直觉) 是一家总部在纽约的 AI 实验室, 从游戏片段平台 Medal 拆分出来, CEO 是 Pim de Witte. 它的目标是在虚拟和物理环境里做"会感知, 会预测, 会行动"的模型, 自称 "the frontier lab for acting in space and time".

**公司**
- 2025 年 10 月完成 1.337 亿美元种子轮, 由 Khosla Ventures 和 General Catalyst 领投. 官网说过去一年累计融资超过 6.5 亿美元. 媒体还报道过估值 62 亿美元的一轮, 但我没在一手来源里核实到.
- 它最大的优势是数据. Medal 每年有数十亿条玩家上传的片段, 而且每条片段都带着玩家当时的操作 (键盘鼠标输入), 也就是"画面 + 动作"成对的数据. 互联网视频一般拿不到这种数据.

**研究方向**
1. 动作模型 (action model): 看到画面后决定下一步做什么. 据 Latent Space 的访谈, 他们现在主要做模仿学习, 也就是直接学玩家的操作.
2. 世界模型 (world model): 给定当前画面和一个动作, 预测下一帧会是什么样. 长期设想是让智能体 (agent) 在这个模拟出来的世界里做强化学习 (reinforcement learning, RL), 通过反复试错来学习.
3. 核心观点: 光有语言学不到对现实的"直觉", 还得经历"意图, 动作, 后果"这个过程, 而游戏是规模最大的这类数据来源.

**代表成果**
- MIRA: 一个可以多人一起玩的世界模型. 它用约 1 万小时的四人 Rocket League 数据训练, 能以每秒 20 帧实时运行, 是和 Kyutai 合作, 并与 Epic Games 一起做的. 训练数据来自公开的游戏机器人 (bot) 对局, 不是 Medal 的玩家数据. 目前通过商用 API 只开放给少数合作伙伴.
- 团队成员之前的工作: IRIS (ICLR 2023 口头报告), Δ-IRIS (ICML 2024), DIAMOND (NeurIPS 2024 spotlight), 都是"用世界模型在想象中训练智能体"这一路线. 还有 Wayve 的自动驾驶世界模型 GAIA-2.

**和你的方向的关系**
它和你的"生存游戏测试环境"思路很接近: 都用游戏来训练动作和世界模型. 不过它走的是数据规模路线, 没有用到类脑的记忆结构.

Sources:
- [General Intuition 官网](https://www.generalintuition.com/)
- [GamesBeat: $133.7M seed](https://gamesbeat.com/general-intuition-raises-133-7m-to-build-an-ai-frontier-research-lab-for-the-game-industry/)
- [InvestGame: $220M at $6.2B](https://investgame.net/news/general-intuition-raises-220m-at-6-2b-as-game-clips-become-training-data-for-world-model-ai/)
- [Latent Space 访谈](https://latent.space/reading/world-models-general-intuition/)
- [Sources News](https://sources.news/p/a-new-bet-on-what-comes-after-llms)

---

Let me check the paper directly for size and any real-world evaluation.

mon père, MIRA 是一个 50 亿参数 (5B) 的潜空间扩散模型 (latent diffusion model), 在单张 Nvidia B200 显卡上能以每秒 20 帧实时生成四人对局. 他们完全没有测过真实世界的表现, 论文只在 Rocket League 这一个游戏里做了评估.

**规模**
- 参数量约 50 亿. 二手报道说主体是扩散 Transformer (diffusion transformer), 时间和空间方向的注意力分开计算, 用流匹配 (flow matching) 训练, 编码器是 DINOv3-L. 这些架构细节我只在新闻里看到, 还没在论文正文里核对.
- 训练数据约 1 万小时 2v2 对局, 全部由开源机器人 Nexto 生成, 没有人类玩家的数据.

**测了什么**
- 都是游戏内的指标. 生成画面的质量分布一直到 5 分钟 (测到的最长时长) 都保持稳定, 作者还说连续生成几个小时也不会崩, 不过这只是观察, 没有量化结果.
- 论文有专门测"是否理解物理规律, 而不只是画面像"的实验, 但摘要里没给具体数字.

**真实世界**
没有机器人实验, 也没有从游戏迁移到现实的实验. 官网只说他们在和机器人方向的伙伴合作, 并把 MIRA 定位成走向物理 AI 的"垫脚石", 没有公布任何结果. 另外它只在一个游戏, 三张地图上训练, 换到别的游戏能否泛化也没测.

Sources:
- [arXiv 2607.05352 摘要](https://arxiv.org/abs/2607.05352)
- [GitHub mira-wm/mira](https://github.com/mira-wm/mira)
- [Neurohive 报道](https://neurohive.io/en/state-of-the-art/mira-a-world-model-fully-simulates-rocket-league-without-requiring-you-to-install-the-game-itself/)
- [KuCoin 报道 (5B 参数)](https://kucoin.com/news/flash/general-intuition-launches-open-source-5b-parameter-mira-model-for-real-time-rocket-league-2v2-simulation)

---


mon père, 同一个模型 (同一套权重) 既在真正的电子游戏里测过, 又在真实世界里测过的, 我只确认到一个, 就是 DeepMind 的 Gato. 如果放宽到"同一种算法"或"游戏式的仿真引擎", 例子就多一些.

**1. 同一套权重: Gato (DeepMind, 2022)**
- 这是一个 12 亿参数的通用策略模型. 同一套权重既能玩 Atari 游戏, 也能控制真实的 Sawyer 机械臂叠积木.
- 它不是世界模型, 只负责决定"下一步怎么做", 不预测"做了之后会发生什么".

**2. 同一种算法, 游戏和现实分别训练: Dreamer 系列 (Hafner 等人)**
- DreamerV3 在 Atari 和 Minecraft 里测过, 而且是第一个不靠人类数据就在 Minecraft 里挖到钻石的算法.
- DayDreamer (2022) 把 Dreamer 用到了 4 台真实机器人上, 比如四足机器人用大约 1 小时学会走路. 不过它用的是 DreamerV2, 而且在每台机器人上都是从零开始训练.
- 这条路线和 MIRA 团队以前做的 IRIS, DIAMOND 最接近, 都是"在世界模型的想象里训练智能体".

**3. 在游戏引擎类的仿真器里训练, 再到真实世界测试 (sim-to-real)**
- OpenAI Dactyl (2018, 2019): 用 MuJoCo 物理引擎加 Unity 渲染训练, 再控制真实的机械手转魔方.
- NVIDIA GR00T 和 Isaac 系列: 用基于 Omniverse 的 Isaac Sim 生成数据, 在真实的人形机器人上测试.
- 这些用的是专门的机器人仿真器, 不是像 Rocket League 这样的商业游戏.

**4. 我没找到的**
目前还没有哪个可交互的视频世界模型 (像 MIRA, Genie 3, GameNGen 这类) 公开报告过, 同一个模型在游戏和真实机器人上都测过. 2026 年这方面的论文, 比如 [Interactive World Simulator](https://arxiv.org/html/2603.08546) 和 [WorldArena 2.0](https://arxiv.org/html/2605.17912v1), 都只比较了"机器人仿真"和"真实机器人", 没有涉及游戏. "在游戏里学到的直觉能直接用到现实"这一点, 目前还没有人证明过, General Intuition 赌的正是这一点.

第 1 到第 3 部分是我凭记忆写的, 这次没有重新查原文, 具体数字只是大概.

Sources:
- [Interactive World Simulator (arXiv 2603.08546)](https://arxiv.org/html/2603.08546)
- [WorldArena 2.0 (arXiv 2605.17912)](https://arxiv.org/html/2605.17912v1)
- [GigaWorld-1 / WMBench (arXiv 2607.02642)](https://huggingface.co/papers/2607.02642)

---


mon père, 这五篇都只在游戏里测过, 没有一篇做了真实世界的实验. 其中四篇是"看画面, 直接输出键盘鼠标或手柄操作"的策略模型, 和 General Intuition 的动作模型是同一条路线. 只有 GameWAM 把世界模型 (预测下一帧画面) 和动作模型合在了一起.

| 工作 | 谁做的 | 类型 | 规模和数据 | 我的看法 |
|---|---|---|---|---|
| **NitroGen** (2026 年 1 月, CVPR 2026) | NVIDIA, Stanford, Caltech 等 | 行为克隆 (behavior cloning, 直接模仿玩家操作) 训练的视觉到动作模型 | 4 万小时游戏视频, 覆盖 1000 多款游戏. 手柄操作是从直播画面里的手柄显示叠层识别出来的 | 最有意思的是数据来源: 不用读游戏代码, 也不用玩家专门录操作, 直接利用公开视频. 这其实削弱了 Medal "自带操作标注"的独家优势. 有报道说它是在 GR00T N1.5 机器人模型基础上做的, 但没有做机器人测试 |
| **Open P2P** (2026 年 1 月) | Player2 / Elefant AI | 行为克隆训练的实时游戏策略模型, 能在消费级显卡上跑 | 8300 多小时人类录制的数据, 40 多款 3D 游戏, 每帧都有键鼠标注, 全部开源 | 标题的论点是"扩大行为克隆的规模能提升因果推理". 这个说法我会打个问号, 很可能只是模型更会利用画面里的时序线索. 它的价值主要在于数据和代码完全公开 |
| **GameScaling** (2026) | 米哈游, 清华, 港大, NUS | 在 Qwen3.5 上继续预训练的视觉语言动作模型, 参数 0.8B 到 27B | 主要是 Minecraft 和 Doom, 数据量从 50 到 1600 小时, 一共 120 组对照实验 | 五篇里在科学上最扎实. 它发现学习分两个阶段: 先学会"怎么操作界面", 这部分能迁移到别的游戏; 后学"这个世界的规律", 这部分只对见过的游戏有效. 越大的模型在单一游戏上训练久了, 换到差别大的游戏反而退步更多. 这直接说明"游戏直觉能泛化"并不是自动成立的. 我只读到了二手转载, 没看到原论文 |
| **GameWAM** (2026 年 8 月) | Guo 等人 | 世界动作模型 (world action model), 同时生成未来画面和键鼠操作 | 在 ViZDoom 和 Minecraft 上测试 | 结构上和你的方向最接近: 先在脑子里预演未来, 只执行一小段动作, 然后根据新画面重新规划. 它还报告了一个有意思的问题 (LASI): 动作输入里变化慢的成分会带偏生成画面中的镜头运动, 说明画面生成和动作生成会互相干扰. 规模很小, 只测了两个简单游戏 |
| **Player-1** (2026 年 10 月) | 腾讯 ARC | 视觉语言模型 (vision-language model, VLM) 的后训练框架: 先监督微调 (supervised fine-tuning, SFT), 再用强化学习 (reinforcement learning, RL) 训练 | 172 款游戏的 6000 多条带思维链的轨迹, 加上 16 个奖励可以自动判定的合成小环境. 底座是 Qwen3.5-9B 和 35B-A3B | 核心想法是"合成经验": 自己造小游戏来做强化学习, 不靠更多人类演示. 数据量很小, 这条路成不成, 要看这 16 个小环境里学到的技能能不能迁移到真实游戏. 我没找到独立评测 |

**整体判断**
1. 这个领域在 2026 年已经从"一个游戏一个模型"转向"一个模型玩很多游戏", 而且数据, 模型权重和评测基准大多开源了. 这对 General Intuition 不利, 它的数据壁垒在变薄.
2. 五篇里没有一篇回答"游戏里学到的东西能不能用到现实", 和上一个问题的结论一样, 这个问题还没人公开验证过.
3. 如果只挑一篇细读, 我推荐 GameScaling. 它的负面结论最有信息量, 正好可以拿来检验你那个生存游戏加课程学习 (curriculum) 测试环境的设计.

Sources:
- [NitroGen (arXiv 2601.02427)](https://arxiv.org/html/2601.02427v1)
- [Open P2P (arXiv 2601.04575)](https://arxiv.org/html/2601.04575v2), [project page](https://elefant-ai.github.io/open-p2p)
- [GameScaling (BestHub 转载)](https://www.besthub.dev/articles/gamescaling-open-source-models-reveal-scaling-laws-for-cross-game-generalization-5c1107d1af7c)
- [GameWAM (arXiv 2608.26200)](https://arxiv.org/pdf/2608.26200)
- [Player-1 (arXiv 2610.20584)](https://arxiv.org/html/2610.20584v1), [GitHub](https://github.com/TencentARC/Player-1)


