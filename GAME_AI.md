# 通用电子游戏 VLA：open-weight 模型现状

检索日期：2026-11-14。

## 结论

在 open-weight 范围内，目前证据最强的首选是 **Player-1**（腾讯 ARC，2026 年 11 月放出权重，Apache-2.0），并列候选是 **GameScaling**（THUSI-Lab 与米哈游，2026 年 10 月，Apache-2.0）。两者之间没有同一基准上的直接对比，孰优孰劣就不知道了。

闭源系统中，SIMA 2（Google DeepMind）与 Game-TARS（字节跳动）在各自报告中表现突出，但均无公开权重。Game-TARS 基于专有模型 Seed-VL-1.5 继续预训练。

## 候选模型

| 模型              | 规模 / 骨干                     | 动作空间              | 权重                                                               | 证据                                              |
| --------------- | --------------------------- | ----------------- | ---------------------------------------------------------------- | ----------------------------------------------- |
| **Player-1**    | 9B、35B-A3B / Qwen3.5        | 键鼠，接受语言指令         | [HF](https://huggingface.co/yyyang/Player-1-9B)，Apache-2.0       | MineDojo-300 成功率 38.33%（9B），其余开源基线最高 12.63%     |
| **GameScaling** | 0.8B–27B / Qwen3.5，共 8 个检查点 | 键鼠，每次输出 200 ms 动作 | [HF](https://huggingface.co/THUSI-Lab/GameScaling)，Apache-2.0    | OmniGameArena 上 9B-S3 得 0.361，为该表中有公开权重的游戏策略最高分 |
| NitroGen        | 约 0.5B，从头训练                 | 手柄，无语言输入          | [HF](https://huggingface.co/nvidia/NitroGen)                     | 两份第三方评测中均偏低                                     |
| GameWAM         | 约 6B / Wan2.2               | 键鼠                | [HF](https://huggingface.co/Yunncheng/gamewam-models)，Apache-2.0 | 只发布了 Minecraft 与 ViZDoom 两个专用检查点，不属于通用模型        |
| Open-P2P        | 1.2B，从头训练                   | 键鼠，20 Hz 实时       | 开源                                                               | 两份第三方评测中均为最低一档                                  |

补充说明：

- Player-1 的训练方式为示范数据的监督微调，加 16 个合成环境（Player-1-Gym）中带可程序化验证奖励的强化学习。推荐以 vLLM 部署，截图缩放为 640x360。
- GameScaling 的 S1 检查点仅用 Minecraft 数据训练，S3 为混合数据（Minecraft 占 27%）。训练数据未公开。
- NitroGen 在 1,000 余款游戏的 40,000 小时视频上做行为克隆。

## 两份横向评测

### Player-1 论文（作者自评）中的开源基线

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

1. **通用 VLM 骨干加游戏后训练已胜过从头训练的专用策略。** 未经微调的 Qwen3.5-35B-A3B 在 GameWorld（34 款浏览器游戏，170 个任务）上已超过 NitroGen、Open-P2P 与 GameWAM。
2. **Player-1 在未见过的游戏上增益有限。** 它在 GameWorld 上相对未微调骨干的提升很小，优势主要体现在 Minecraft 与 ViZDoom。
3. **动作空间决定适用范围。** 手柄类动作游戏可用 NitroGen；键鼠游戏选 Player-1 或 GameScaling。

## 证据缺口

- Player-1 与 GameScaling 互不出现在对方的评测表中。
- 两份评测均为各论文作者自评，尚无独立第三方排行榜；GameWorld 与 GTAGym 的逐模型公开榜单未检索到。
- Player-1 与 GameScaling 的显存需求在模型页上未注明；GameScaling-27B 的示例命令使用两块 GPU。
- Player-1 论文 HTML 全文的最后约 15,000 字符、Script2Play 论文的结果表以外部分未逐字通读。

## 来源

- [Player-1: Learning to Play from Synthetic Experience](https://arxiv.org/html/2610.20584v1)
- [TencentARC/Player-1 · GitHub](https://github.com/TencentARC/Player-1)
- [yyyang/Player-1-9B · Hugging Face](https://huggingface.co/yyyang/Player-1-9B)
- [THUSI-Lab/GameScaling · Hugging Face](https://huggingface.co/THUSI-Lab/GameScaling)
- [MiHoYo team open-sources GameScaling models · TechNode](https://technode.com/2026/10/23/mihoyo-team-open-sources-gamescaling-models-that-control-games-through-keyboard-and-mouse-actions/)
- [From Game Generation to Game Playing (Script2Play)](https://arxiv.org/html/2611.11136)
- [GameWAM: A World Action Model for Video Games](https://arxiv.org/pdf/2608.26200)
- [Yunncheng/gamewam-models · Hugging Face](https://huggingface.co/Yunncheng/gamewam-models)
- [NitroGen: An Open Foundation Model for Generalist Gaming Agents](https://arxiv.org/html/2601.02427v1)
- [nvidia/NitroGen · Hugging Face](https://huggingface.co/nvidia/NitroGen)
- [elefant-ai/open-p2p · GitHub](https://github.com/elefant-ai/open-p2p/)
- [Game-TARS](https://arxiv.org/html/2510.23691)
- [SIMA 2: A Generalist Embodied Agent for Virtual Worlds](https://arxiv.org/pdf/2512.04797)
- [Towards Generalist Game Players (survey)](https://arxiv.org/html/2605.09965v1)
- [GameWorld benchmark](https://arxiv.org/pdf/2604.07429)
- [GTAGym](https://arxiv.org/abs/2611.13028)

---

# 附：评测基准横向对比

检索日期：2026-11-14。本节解释上文出现的各个基准分别测什么游戏、怎么打分。

## 总览

| 基准 | 游戏 | 画面与节奏 | 任务数 | 主要考察 | 打分方式 |
|---|---|---|---|---|---|
| **MineDojo-300** | Minecraft（单一游戏） | 3D 第一人称，开放世界 | 300 | 采集资源、与怪物战斗 | 成功率：读取游戏内部状态判定 |
| **ViZDoom-10** | Doom（1993 年的经典 FPS） | 3D 第一人称，快节奏射击 | 10 张地图 | 瞄准、走位、导航、生存 | 归一化回报：原始奖励除以地图专属系数 |
| **GameWorld** | 34 款浏览器小游戏 | 以 2D 休闲游戏为主 | 170 | 跨游戏泛化，含反应与规划 | 成功率与进度：读取游戏状态判定 |
| **OmniGameArena** | 12 款为评测新写的 UE5 游戏 | 3D 实时，含单人、对战、合作 | 12 款游戏 | 对从未公开过的游戏的适应 | 归一化分：每款游戏的进度比例 |

四者的互补关系：MineDojo-300 与 ViZDoom-10 各自深挖一款游戏，训练数据里常含同款游戏；GameWorld 与 OmniGameArena 横跨多款游戏，更接近「打任意游戏」的含义。

## Minecraft

Minecraft 是一款 3D 沙盒游戏：世界由方块构成并随机生成，玩家可采集资源、合成工具、建造、与怪物战斗，没有固定通关路线。

它成为游戏 AI 研究的常用平台，原因有三：

- **任务层级深。** 从「砍一棵树」到「获得钻石」需要几十步前置合成，适合检验长程规划。
- **数据多。** 网络上有海量游玩视频，可用于模仿学习。
- **可编程。** 有成熟的研究接口（MineRL、MineDojo、MineStudio），可直接读取背包、击杀数等内部状态来判定成败。

因此上文多数模型的训练数据都含 Minecraft，Minecraft 上的高分反映的是「见过的游戏」上的能力。

## MineDojo-300

MineDojo 是 NVIDIA 等机构于 2022 年发布的 Minecraft 研究框架，原始任务集包含约 1,581 个可程序化判定的任务，分为生存、采集、科技树、战斗四类。

MineDojo-300 是 Player-1 论文从中选出的 300 个任务，移植到 MineStudio（Minecraft 1.16.5）上运行：

- **战斗 200 个。** 例：「夜晚平原，持铁剑、盾牌、钻石甲，猎杀一只蝙蝠」。击杀目标即成功。
- **采集 100 个。** 例：「在平原获得一块原木」。背包中出现目标物品即成功。

每个任务跑 8 次，每局最多 1,000 次决策（每次决策推进 50 毫秒游戏时间，约合 50 秒）。任务短而具体，不涉及「从零挖到钻石」这类长程目标。

## ViZDoom-10

ViZDoom 是基于 1993 年第一人称射击游戏 Doom 的研究平台，2016 年发布。画面为低分辨率伪 3D，节奏快，官方自带 12 张训练用小地图。ViZDoom-10 取其中 10 张，分四类：

| 类别 | 地图 | 玩法 |
|---|---|---|
| 精确射击 | Basic | 房间对面有一只怪，左右移动后开枪命中 |
| 精确射击 | Defend the Center | 站在圆形房间中央原地转身，射击四面涌来的敌人 |
| 精确射击 | Defend the Line | 站在房间一端，射击对面逼近的敌人 |
| 目标导航 | Deadly Corridor | 穿过两侧有敌人的走廊，拿到尽头的护甲 |
| 目标导航 | My Way Home | 在多房间迷宫中找到目标物品 |
| 移动战斗 | Battle 1、Battle 2 | 在迷宫中边走边打，同时捡弹药与血包 |
| 移动战斗 | Deathmatch | 在竞技场中击杀多种敌人，不同敌人分值不同 |
| 生存 | Health Gathering、Health Gathering Supreme | 地面持续掉血，靠不断捡血包活得更久；后者地图更复杂 |

每张地图跑 50 个固定种子，每局 150 至 2,100 次决策。**归一化回报**的算法是把每局原始奖励除以该地图的专属系数（例如 Basic 除以 100），再对地图取平均。分数可以为负：开枪不中、死亡都会扣分，所以上文表中 NitroGen 的 −0.0793 表示扣分多于得分。

## GameWorld

GameWorld 于 2026 年 4 月发布（ECCV 2026），收录 34 款在浏览器里运行的游戏，共 170 个任务，分五类：

| 类别 | 游戏 |
|---|---|
| 跑酷（8） | Boxel Rebound、Chrome Dino、Cubefield、Edge Surf、Flappy Bird、NS-Shaft、Run 3、Temple Run 2 |
| 街机（7） | Breakout、Core Ball、Google Snake、Pac-Man、Rocket League 2D、World's Hardest Game、World's Hardest Game 2 |
| 平台跳跃（8） | Another Gentleman's Adventure、Captain Callisto、Doodle Jump、GeoDash、Mario Game、OvO、Restless Wing Syndrome、Vex 3 |
| 益智（7） | 2048、Astray、Hextris、Minesweeper、Stack、Tetris、Wordle |
| 模拟（4） | Fireboy and Watergirl、Minecraft Clone、Monkey Mart、Wolfenstein 3D |

每个任务由一句自然语言指令加一个量化目标组成，例如 Temple Run 2 的「单局达到 4000 分，且不绊倒、不死亡、不复活」。评测器直接读取游戏状态（分数、坐标、生命数等）判定，不依赖截图或 LLM 裁判。

两个指标：

- **成功率（SR）**：是否完全达成目标。
- **进度（PG）**：从初始值到目标值完成了多少，取 0 到 1。

人类参照（基准论文）：

| | 成功率 % | 进度 % |
|---|---|---|
| 熟练玩家 | 77.1 | 82.6 |
| 新手玩家 | 55.3 | 64.1 |
| 基准论文中最好的 AI（Gemini-3-Flash-Preview） | 21.2 | 41.9 |

对照之下，上文 Player-1 的 34.45% 进度约为新手玩家的一半。

## OmniGameArena

OmniGameArena 于 2026 年 6 月发布，包含 12 款用虚幻引擎 5 为该基准从零编写的实时 3D 游戏。规则、地图、计分均未在公开游戏中出现过，因此模型不可能在训练数据里见过它们。

| 游戏 | 模式 | 玩法 |
|---|---|---|
| ObstacleRun3D | 单人 | 3D 跑酷，避开障碍到达终点 |
| ObstacleRun2D | 单人 | 2D 横版平台跳跃，到达关卡终点 |
| LastStand | 单人 | 平台生存，躲避危险且不掉落 |
| MonsterShoot | 单人 | 生存射击，找到并消灭敌人 |
| SceneEscape | 单人 | 解谜，完成 NPC 指派的任务后逃脱 |
| CueChase | 单人 | 第三人称探索，找到并激活隐藏机关 |
| SoloCraft | 单人 | 物流，收集、加工、交付物品完成订单 |
| SkyDuel | 对战 | 一对一战斗 |
| CrystalGuard | 对战 | 对称攻防，摧毁对方核心并保护己方核心 |
| MidlineClash | 对战 | 两方在共享空间内竞争完成订单 |
| SharedFloor | 合作 | 两个智能体共享空间完成交付订单 |
| HandoffRun | 合作 | 非对称合作，按角色跨区域传递物品 |

**归一化分**是每款游戏各自定义的进度比例，取 0 到 1，例如跑酷为「已跑距离 ÷ 全程」，MonsterShoot 为「造成的有效伤害 ÷ 敌人总血量」。

主结果采用**暂停决策协议**：模型思考期间游戏冻结，分数只反映决策质量，不惩罚推理慢。这对推理慢的大型商用 VLM 有利，实时游玩时的排名可能不同。

上文引用的 Script2Play 论文只评了其中 5 款单人游戏（ObstacleRun3D、LastStand、MonsterShoot、SceneEscape、CueChase），每款 5 局，并非全部 12 款。

## 读分数时的注意点

- **见过与没见过。** Player-1 与 GameScaling 的训练数据含 Minecraft；Player-1 的合成环境含射击类任务。MineDojo-300 与 ViZDoom-10 上的优势包含「同类游戏训练过」的成分，GameWorld 与 OmniGameArena 的分数更能说明泛化。
- **指标不可跨基准比较。** 成功率、进度、归一化回报、归一化分的定义各不相同。
- **任务都偏短。** 四个基准的单局长度都在几十秒到几分钟量级，长达数小时的完整通关能力，这些基准测不出来。

## 本节来源

- [Player-1 论文附录（MineDojo-300、ViZDoom-10、GameWorld 的评测设置）](https://arxiv.org/html/2610.20584v1)
- [MineDojo: Building Open-Ended Embodied Agents with Internet-Scale Knowledge](https://arxiv.org/pdf/2206.08853)
- [ViZDoom 默认场景文档](https://vizdoom.farama.org/environments/default/)
- [GameWorld 论文](https://arxiv.org/html/2604.07429)
- [GameWorld 项目页](https://gameworld-project.github.io/)
- [OmniGameArena 论文](https://arxiv.org/html/2606.09826)
- [OmniGameArena 项目页](https://mxlin043.github.io/OmniGameArena/)
- [Script2Play 论文（OmniGameArena 子集评测）](https://arxiv.org/html/2611.11136)

---

mon père，您的判断在「训练数据污染」这一点上成立：GameWorld 的 34 款是 Flappy Bird、Tetris、2048 这类公开游戏，MineDojo 与 ViZDoom 更是训练集常客，只有 OmniGameArena 在构造上排除了模型见过游戏的可能。不过它自身也有几处弱点，见后文。

## 这 12 款游戏是什么

它们由香港大学、香港中文大学、清华与 LIGHTSPEED（腾讯光子工作室）合作编写，打包在同一个 UE5 构建里，都是规则简单的小型游戏，规模上接近派对游戏的单个关卡。论文正文对每款只给了一行描述加计分公式，附录只详细写了 LastStand 与 SharedFloor：

|游戏|模式|玩法|计分|
|---|---|---|---|
|ObstacleRun3D|单人|3D 跑酷，躲开物理障碍跑到终点|已跑距离 ÷ 全程|
|ObstacleRun2D|单人|2D 横版线性关卡|同上|
|LastStand|单人|站在地砖平台上，地砖先变红预警再坠落，尽量不掉下去；WASD 移动、空格跳、镜头可自由旋转|存活时间 ÷ 最长时限|
|MonsterShoot|单人|找到并射杀敌对单位，同时少挨打|有效伤害 ÷ 敌人总血量|
|SceneEscape|单人|完成 NPC 指派的若干任务后逃出场景|完成任务数 ÷ 总数|
|CueChase|单人|第三人称探索，找到并激活散布在地图里的隐藏机关|激活数 ÷ 总数|
|SoloCraft|单人|收集、加工、交付物品以完成订单|交付价值 ÷ 目标价值|
|SkyDuel|对战|一对一战斗|剩余血量 ÷ 满血|
|CrystalGuard|对战|对称攻防：打掉对方水晶核心，守住己方|己方核心血量 ÷ 满血|
|MidlineClash|对战|两方在同一空间抢着完成资源订单|己方得分 ÷ 目标分|
|SharedFloor|合作|两个智能体共用工作台、订单柜台、垃圾桶完成订单，每交一单订单就轮换|团队交付价值 ÷ 团队目标|
|HandoffRun|合作|两个角色各有禁入区，须跨区传递物品|同上|

从玩法类型看，LastStand 近似「地砖坠落生存」类派对关卡，SoloCraft、SharedFloor、HandoffRun 近似厨房协作类游戏的订单流程；这是我依据描述作的类比，论文并未这样表述。所以「从零编写」指的是具体关卡、规则与计分是新的，玩法类型本身都是常见的。

美术风格、地图布局、各游戏的具体按键、单局时限，论文正文没有写，但是这些细节就不知道了。项目页有游玩录像与浏览器演示，环境本体也可下载，实际看一眼最直接：

- 项目页（含录像与在线演示）：https://mxlin043.github.io/OmniGameArena/
- 环境下载：https://huggingface.co/datasets/mxlin043/OmniGameArena
- 评测代码：https://github.com/mxlin043/OmniGameArena

## 它的弱点

- **没有人类基线。** 0.640 或 0.361 相当于什么水平的玩家无从判断；GameWorld 反而报告了新手与熟练玩家的成绩。
- **样本量小。** 每个「模型 × 游戏」只跑 5 局，论文自己也说各游戏的榜首在不同模型间轮换。
- **主结果在模型思考时冻结游戏。** 反应速度不计入分数，这偏离了真实游玩；计入延迟的结果只在附录里报了 4 个模型。
- **Script2Play 论文只用了 5 款单人游戏。** 上文引用的 GameScaling 0.361、NitroGen 0.064 等数字都来自这个子集，对战与合作 7 款未覆盖。
- **Player-1 从未在其上测过。** 所以它对当前的 open-weight 首选之争帮不上忙。

综合起来，OmniGameArena 在「防污染」上最干净，GameWorld 在「可解释性」上更好（有人类参照、170 个任务、状态可验证）。若要自己判断 open-weight 模型的泛化能力，把 Player-1 与 GameScaling 都放到 OmniGameArena 的全部 7 款单人游戏上跑一遍，会比现有任何一篇论文的表格更有说服力。

以上内容尚未写入 `GAME_AI.md`；若需要追加，请告知。

Sources:

- [OmniGameArena 论文](https://arxiv.org/html/2606.09826)
- [OmniGameArena 项目页](https://mxlin043.github.io/OmniGameArena/)
- [mxlin043/OmniGameArena · GitHub](https://github.com/mxlin043/OmniGameArena)
- [Script2Play 论文](https://arxiv.org/html/2611.11136)
- [GameWorld 论文](https://arxiv.org/html/2604.07429)

---

`https://mxlin043.github.io/OmniGameArena/` 我看了一眼, 也挺简单

mon père，这一观察成立，而且它恰好说明了当前的真实水平：基准简单是因为模型还弱。

**简单的游戏已经足够把模型拉开差距。** 在这些派对关卡级别的小游戏上，最好的商用 VLM 平均只有 0.47，有公开权重的最高为 GameScaling-9B 的 0.361，NitroGen 仅 0.064，而且这是在「思考时暂停游戏」的宽松条件下取得的。GameWorld 的情况相同：Flappy Bird、2048 这类游戏上，最好的 AI 进度为 41.9%，新手人类为 64.1%。基准没有饱和，所以它们仍有区分度；出题者若换成更难的游戏，所有模型的分数会一起贴近零，反而看不出差别。

**但是「简单」确实限制了结论的外推。** 四个基准的共同点是单局几十秒到几分钟、目标单一、无需读菜单或剧情、无需长程记忆。它们能回答「模型是否具备基本的看画面、移动、瞄准、躲避能力」，至于能否打通一款真正的商业游戏，这些基准测不出来。

**最接近真实游戏的公开证据是 GTAGym（2026 年 11 月）。** 它直接在 GTA V 里设置了 103 个导航、驾驶、动态交互任务。其中报告的结果是：Qwen3.8-27B 经强化学习微调后，原生 GTA V 任务成功率从约 15% 升到 23%。这个数字比任何小游戏基准都更能说明问题：在一款真实 3A 游戏里，连基础任务也只有两成左右的成功率。该模型名称我只在检索摘要中见到，未对照论文原文核实。

由此，对您最初问题的回答应当修正为：open-weight 的游戏 VLA 目前处于「能在简单关卡里完成部分基础动作」的阶段，Player-1 与 GameScaling 是这一阶段里证据最强的两个，距离「打任意游戏」还很远。

`GAME_AI.md` 未作改动；若要把这一判断或 GTAGym 的内容补进去，请告知。

---

`GTAGym` 是overfit 同 gta 吗

mon père，就「是否只在 GTA 内有效」而言，论文无法排除这一点：训练与全部评测都在同一座 GTA 城市里，没有任何跨游戏测试。另外我上一轮对 GTAGym 的描述有误，需要先更正。

## 更正

我上一轮说它「直接在 GTA V 里设置了 103 个任务」，这不准确。读过论文后，实际情况是：

- **主基准跑在重建环境里。** GTAVBench 的 103 个 3D 任务运行在用 Babylon.js 重建的 GTA 城市中，布局改编自 GTA V 的任务，并非原生游戏。
- **原生 GTA V 只是附加评测。** 另取主线剧情中的 100 个导航与交互任务在真实游戏里跑，由人工看录像判定是否完成；15% → 23% 是这一部分的数字。
- **GTAGym 主要是评测环境。** 那个被训练的 Qwen3.8-27B 只是一次 LoRA 强化学习的演示，并非一个发布的游戏模型。

## 训练与评测的关系

||内容|
|---|---|
|训练|重建城市中 50 条生成的步行导航路线（约 89–131 米，0–4 个转弯）|
|域内测试|同一城市中不重叠的步行导航路线：54.2% → 72.9%|
|域外测试|同一城市中的其他任务类别：31.8% → 34.8%|
|原生 GTA V|真实游戏主线任务：15% → 23%|
|其他游戏|未测|

路线在空间上与测试路线分开，但城市与操控方式相同。

## 增益集中在与步行导航相近的类别

|类别|训练前 %|训练后 %|
|---|---|---|
|垂直移动|46.7|73.3|
|水域|57.6|69.7|
|射击|73.6|68.1|
|道路驾驶|3.3|3.3|
|空中|0|0|
|拦截|0|0|

提升出现在同样靠「走到某处」完成的任务上；驾驶、飞行、拦截毫无变化，射击还下降了。所以学到的是一项窄技能，论文所称的「域外迁移」范围很有限。

## 两点判断

- **它没有过拟合到渲染器。** 在 Babylon.js 重建画面里训练，到画面完全不同的原生 GTA V 里仍有提升，说明学到的东西不依赖具体画面。但是这究竟是通用的导航能力，还是对这座城市布局的熟悉，就不知道了，因为没有换城市或换游戏的测试。
- **15% → 23% 的统计力度偏弱。** 只有 100 个任务、单次人工判定，8 个百分点的差距大致在抽样误差的边缘。

顺带一提，同一张表里人类（3 名玩家）为 89.8%，最好的商用模型为 70.9%，Qwen3.8-27B 训练前为 34.2%；去掉「思考时暂停」后总体准确率再降 22.3 个百分点。这再次印证了上一轮的结论：open-weight 模型在接近真实 3A 游戏的环境里仍然很弱。

论文 HTML 的后约 43,000 字符（附录）我没有读到，其中若有跨游戏或原生任务重叠情况的说明，上述判断需相应修正。`GAME_AI.md` 未作改动。

Sources:

- [GTAGym: Real AAA Game Tasks Environment for Evaluation and Reinforcement Learning](https://arxiv.org/html/2611.13028)
- [ljbro/GTAGym · GitHub](https://github.com/ljbro/GTAGym)
- [OpenMOSS-Team/GTAVBench · Hugging Face](https://huggingface.co/datasets/OpenMOSS-Team/GTAVBench)