
mon père，检索已完成，现在据此重新作答。

## 结论

有搞头，但只在范围收窄到"让您这一个游戏跑起来"时成立。检索结果修正了我上一轮的一个说法：直接以"Unity 游戏源码不改即可编译运行"为目标的开源项目是存在的，只是都处于极早期。

## 有人做过的

**以 `UnityEngine` API 兼容为明确目标的项目**

|项目|做法|现状|
|---|---|---|
|[Real Disunity](https://github.com/acoliver/real-disunity)|Rust 引擎加 wgpu 渲染，运行按 `UnityEngine` API 写的 C# 脚本；可读取 Unity 文本场景、prefab 和 `.meta` GUID|MIT，0.1 预发布，约 2600 次提交，6 星。目前只跑通 Snake、Pong、Breakout 三个固定游戏；3D 物理、音频、动画、粒子、大部分 `UnityEngine.UI` 均未实现|
|[OpenUnity](https://github.com/nanashili/OpenUnity)|自称 clean-room 的 Rust 重实现，带 C# 脚本兼容层|搜索摘要显示其 README 自述"Unity parity 未经验证"，缺碰撞事件、关节等。我直接抓取该仓库时返回 404，所以它现在是否还公开就不知道了|
|[UnEngine](https://github.com/jbruening/UnEngine)|克隆 `UnityEngine.dll`，Release 构建使用相同的命名空间和程序集名|MIT，51 星，53 次提交，无发行版；用途是在 Unity 之外测试脚本，不用于发布游戏|
|[united-godot](https://github.com/teadrinker/united-godot)|把 Unity API 的一个子集映射到 Godot 的 C# 类型上|MIT，10 次提交的概念验证；作者明确把"完整 Unity API 适配器"列为非目标|

Real Disunity 的长期目标写的是"Unity 游戏无需改源码即可编译运行"，README 同时承认尚未达到。

**"相似"或"资产转换"层面的项目**

- [Prowl](https://github.com/ProwlEngine/Prowl)：MIT 的 C# 引擎，1.3k 星，约 6100 次提交，处于 1.0 Preview 5。README 说保持与 Unity 相似的 API，"理想情况下 Unity 项目能以尽量小的阻力移植"。这是相似，不承诺兼容。
- [Unidot Importer](https://github.com/V-Sekai/unidot_importer)：MIT，982 星，把 `.unity`、`.prefab`、材质、动画等转成 Godot 4 格式。脚本和 shader 明确需要手工移植，Canvas/UI 不支持；README 写的支持范围是 Godot 4.0 到 4.2。
- 另有若干脚本转换器（[Zylann 的转换器](https://github.com/Zylann/unity_to_godot_converter)只生成 stub 脚本，[a2937 的工具](https://gh.hhj520.top/a2937/unity-to-godot-converter)基于 Roslyn 改写），都是实验性质。

**其他领域的成功先例**：FNA/MonoGame 对 XNA 的重实现、Wine 对 Win32 的重实现（这两项来自我的记忆，本轮未检索）。

## 难点

所有项目的进度都卡在同一处：API 签名容易抄，背后的行为和数据难复现。

1. **序列化资产**：场景、prefab、GUID 引用。Real Disunity 和 Unidot 都已经在啃这一块，说明可做。
2. **Shader**：没有任何项目能自动转换，全部要求手工移植。
3. **行为保真**：物理数值、脚本执行顺序、动画状态机、UI 布局。
4. **第三方插件**：每一个都是额外的兼容面。

## 法律

- UnityCsReference 仓库的 README 写明[仅供参考用途](https://awesome.ecosyste.ms/projects/github.com%2FUnity-Technologies%2FUnityCsReference)，读过再写有污染风险。
- Google v. Oracle 判定重实现 API 属于 fair use，但法院[只是假设 API 可受版权保护而未作裁决](https://www.rstreet.org/commentary/google-llc-v-oracle-america-inc-2/)，适用范围比通常理解的窄。
- clean-room 流程对 Unity 的 Reference Only License 是否构成足够保护，检索中没有找到直接讨论，这一点需要律师判断。

## 建议

两条可行路径，取决于您的游戏规模：

- **游戏较小、以 2D 为主**：可以直接试 Real Disunity，看缺哪些 API，缺口本身就是工作量的度量。
- **游戏较大或为 3D**：在 Godot .NET 或 Stride 上写一层只覆盖您游戏所用 API 的 shim，资产用 Unidot 转换，shader 手工重写。

如果只移植这一个游戏，直接改写成宿主引擎的原生写法通常比维护兼容层便宜；兼容层在有多个游戏要移植时才划算。

要进一步判断，我需要知道游戏是 2D 还是 3D、用哪条渲染管线、自定义 shader 数量、依赖的第三方插件，以及代码规模。

Sources:

- [Real Disunity](https://github.com/acoliver/real-disunity)
- [OpenUnity](https://github.com/nanashili/OpenUnity)
- [UnEngine](https://github.com/jbruening/UnEngine)
- [united-godot](https://github.com/teadrinker/united-godot)
- [Prowl](https://github.com/ProwlEngine/Prowl)
- [Unidot Importer](https://github.com/V-Sekai/unidot_importer)
- [Zylann/unity_to_godot_converter](https://github.com/Zylann/unity_to_godot_converter)
- [a2937/unity-to-godot-converter](https://gh.hhj520.top/a2937/unity-to-godot-converter)
- [UnityCsReference 许可信息](https://awesome.ecosyste.ms/projects/github.com%2FUnity-Technologies%2FUnityCsReference)
- [R Street: Google v. Oracle 评述](https://www.rstreet.org/commentary/google-llc-v-oracle-america-inc-2/)
- [Unity Editor Source Code Terms (Legacy)](https://unity.com/legal/unity-editor-source-code-terms-legacy)
- 
---


mon père，当前工作目录 `E:\Workspace\photo-studio` 里只有 README 和两份规则文件，没有 Unity 工程，所以这个游戏我还看不到。

有了具体的游戏，"有没有搞头"就可以从估计变成测量。请给我工程所在路径，我只读不写，统计以下几项：

1. **Unity 版本与渲染管线**：读 `ProjectSettings/ProjectVersion.txt` 和 `Packages/manifest.json`。
2. **API 表面**：扫描全部 C# 脚本，列出实际引用的 `UnityEngine.*` 类型和成员及其出现次数。这份清单就是兼容层必须实现的全部范围。
3. **Shader**：自定义 `.shader`、Shader Graph 的数量。这一项没有任何现成工具能自动转换。
4. **第三方依赖**：`Assets/Plugins`、Asset Store 包、native DLL。
5. **资产构成**：场景、prefab、动画控制器、UI Canvas 的数量，对照 Unidot 的支持列表看哪些能自动转换。

有了这五项，就能把 API 清单与 Real Disunity 的 `docs/unity-api.md` 和 Godot/Stride 的能力逐项对比，得出缺口大小，再判断兼容层与直接改写哪个便宜。

如果工程不在这台机器上，请告诉我：2D 还是 3D、Unity 版本、大致代码行数、用了哪些插件。

mon père，"睡眠效果"我理解为"水面效果"的笔误，以下按此作答。

## 判断

这种构成下，为这个游戏做 Unity 兼容层的搞头很小。成本集中在三处，而这三处恰好都是兼容层帮不上忙的地方。

**1. Shader 和水面效果**

上一轮检索到的所有项目都要求 shader 手工移植。水面效果通常依赖深度纹理、屏幕空间折射和反射、法线动画，有些还带曲面细分或浮力查询。这些绑定在 Unity 渲染管线的具体内部约定上，换引擎等于重写，并且要靠肉眼逐项调到观感一致。

**2. Asset Store 库**

每个库都把兼容面扩大一圈。库的作者会用到冷门 API、编辑器扩展、反射和 native plugin，您自己的游戏代码可能只用 `UnityEngine` 的 5%，库会把这个比例拉高很多。

还有许可问题。凭记忆，Asset Store EULA 允许把第三方资产用在自己的产品里，但不允许以源码形式再分发，这一条我本轮没有检索核对。如果它成立，移植后的工程即使跑在开源引擎上，也无法整体开源。

**3. 各种资产**

这一项相对最好办。模型、贴图、动画、音频是通用格式，Unidot 这类工具能转掉场景和 prefab 的大部分。材质只支持标准 shader，自定义材质要重新连。

## 所以先问动机

您想离开 Unity 的原因决定了哪条路合理：

- **担心收费或许可条款**：这个游戏留在 Unity 上发布最便宜。新项目再用开源引擎，不背历史包袱。
- **想让游戏本身开源或可被社区长期维护**：Asset Store 库是硬障碍，需要先逐个替换成开源等价物。这一步在 Unity 内部就可以做，做完后移植难度也随之下降。
- **想要某个 Unity 不支持的平台或需要改引擎源码**：直接改写到 Godot 或 Stride，按模块逐个迁移，不经过兼容层。
- **对"做一个 Unity 兼容引擎"这件事本身感兴趣**：那应当从小游戏起步，像 Real Disunity 从 Snake、Pong 开始那样。拿这个复杂游戏当第一个目标，会在看到任何画面之前耗掉很长时间。

您是哪一种？


---

mon père，这个目标改变了整个分析。检索后的结论：您列出的三项需求都不要求离开 Unity，"Unity 兼容引擎"与这个目标基本无关。

## 这三家是什么

- **Isaac Sim**：NVIDIA 基于 Omniverse 的仿真框架，USD 场景图加 PhysX 和 RTX 渲染。
- **Genesis**：现名 [Genesis World](https://genesis-world.readthedocs.io/)，2024 年 12 月起步的学术项目，现由 Genesis AI 支持，源码 [Apache 2.0](https://github.com/Genesis-Embodied-AI/genesis-world)，卖点是 GPU 并行物理的吞吐量。
- **Lightwheel**：[建在 Isaac Sim 和 Isaac Lab 之上](https://www.nvidia.com/en-us/case-studies/lightwheel/)的仿真数据公司，产品是物理属性经过校验的 SimReady 资产（USD 和 MJCF 格式）和任务基准。

三家的价值各不相同：引擎、吞吐量、内容。您手上的游戏对应的是内容这一项。

## 三项需求在 Unity 内的可行性

|需求|Unity 内的现成机制|状况|
|---|---|---|
|接入 MuJoCo|[官方 Unity 插件](https://mujoco.readthedocs.io/en/3.6.0/unity.html)，随 MuJoCo 发行，3.11 文档仍在维护|可用，有下述限制|
|物理步长脱离 wall clock|把 simulation mode 设为 Script 后手动调用 [`Physics.Simulate`](https://docs.unity3d.com/ScriptReference/Physics.Simulate.html)，每次传固定步长|可用|
|由外部模型驱动步进|ML-Agents 的 [`DisableAutomaticStepping()` 与 `EnvironmentStep()`](https://docs.unity3d.com/Packages/com.unity.ml-agents@3.0/manual/index.html)|可用，包在 2026 年仍有发布|

把 Unity 游戏做成具身智能环境也有先例：AI2-THOR 和 ThreeDWorld 都建在 Unity 上，并且都留在 Unity 上。

## 真正的工作量在哪里

**1. MuJoCo 与游戏现有物理的边界**

MuJoCo 插件要求每个 game object 至多挂一个 MuJoCo 组件，并且[增删组件会触发整个场景重建](https://mujoco.readthedocs.io/en/3.6.0/unity.html)，文档承认这对大模型很昂贵。游戏里的水面浮力、角色控制器和 Asset Store 库都是按 PhysX 的 `Rigidbody` 和碰撞回调写的，它们看不见 MuJoCo 里的物体。

可行的切法是：机器人本体和可操作物体归 MuJoCo，静态场景几何导出成碰撞网格给 MuJoCo，其余装饰性物理留在 PhysX。两个世界之间的耦合（比如机器人推开一个由 PhysX 管的门）需要逐项决定归属。

**2. 游戏逻辑里的时间**

`Physics.Simulate` 只管物理。动画、粒子、协程里的 `WaitForSeconds`、shader 里的 `_Time`（水面动画就靠它）都跟着 Unity 的帧时间走。凭记忆，`Time.captureDeltaTime` 能让每帧推进固定的仿真时间，从而把这些一并锁定，但本轮检索没有覆盖到它，需要实测。Asset Store 库里若有直接读系统时钟的代码，要逐个找出来。

**3. VLA 的观测与动作接口**

第一视角相机渲染到纹理、读回、送给模型，模型输出动作再映射到 MuJoCo 的 actuator。VLA 推理一次通常是几十到几百毫秒，仿真必须停下来等它，这正是您要脱离 wall clock 的原因。这部分工作量不大。

## 什么情况下才需要离开 Unity

- **大规模并行**：Isaac 和 Genesis 靠 GPU 物理在单机上并行数千个环境。Unity 是一个进程一个环境。如果用途是 VLA 的评测和数据采集，吞吐瓶颈在模型推理，这一点无关紧要；如果要做大规模强化学习，这是结构性差距，换开源引擎也解决不了，需要换成 MuJoCo MJX 那一类架构。
- **集群上的无显示器渲染**：AI2-THOR 的 headless 版本当年[依赖 Unity Simulation Pro](https://blog.allenai.org/ai2-thor-partners-with-unity-to-enable-headless-multi-node-training-in-the-cloud-571802be5ad3)。旧版 Unity Simulation 已于 [2023 年 12 月弃用](https://docs.unity3d.com/Simulation/manual/index.html)；新版 Simulation Pro 在 [2026 年 9 月以 Early Access 发布](https://unity.com/blog/unity-industry-product-releases-2026)，带 headless Linux 构建目标，但要求 Unity Industry 许可。普通 Unity Linux 构建能否在没有 X server 的机器上做 GPU 离屏渲染，检索中没有找到答案，需要在目标机器上实测。
- **对外分发**：如果要把这个 toolset 交给别人用，Unity 运行时条款和 Asset Store 库的再分发限制都会成为约束。Unity 对"用构建好的 player 做大规模 ML 训练"有没有专门条款，我查了但没有找到相关文本，这一点就不知道了。

## 建议

先在 Unity 内做一个最小闭环：一个场景、一个 MuJoCo 机器人、手动步进、第一视角图像送出、动作送回。这一步能直接暴露上面第 1、2 项的真实成本，并且完全不涉及移植。

闭环跑通之后，是否离开 Unity 取决于两个问题：这个 toolset 是自用还是要分发，以及目标规模是单机评测还是集群训练。

Sources:

- [MuJoCo Unity Plug-in 文档](https://mujoco.readthedocs.io/en/3.6.0/unity.html)
- [Unity Physics.Simulate 文档](https://docs.unity3d.com/ScriptReference/Physics.Simulate.html)
- [ML-Agents 3.0 文档](https://docs.unity3d.com/Packages/com.unity.ml-agents@3.0/manual/index.html)
- [Is ML-Agents being deprecated?](https://discussions.unity.com/t/is-ml-agents-being-deprecated/1643681)
- [Genesis World 文档](https://genesis-world.readthedocs.io/)
- [Genesis World 仓库](https://github.com/Genesis-Embodied-AI/genesis-world)
- [NVIDIA Lightwheel 案例](https://www.nvidia.com/en-us/case-studies/lightwheel/)
- [Lightwheel Hugging Face 页面](https://huggingface.co/LightwheelAI)
- [AI2-THOR 文档](https://ai2thor.allenai.org/ithor/documentation/)
- [AI2-THOR 与 Unity 的 headless 合作](https://blog.allenai.org/ai2-thor-partners-with-unity-to-enable-headless-multi-node-training-in-the-cloud-571802be5ad3)
- [Unity Simulation 文档（已弃用）](https://docs.unity3d.com/Simulation/manual/index.html)
- [Unity Industry 2026 产品发布](https://unity.com/blog/unity-industry-product-releases-2026)

---

mon père，同意，这一项是整个方案里最难缠的部分。上一轮留下的"凭记忆"我已经核对，结果比我当时说的更糟一些。

## 核对结果

Unity [官方文档](https://docs.unity.com/en-us/engine/current/script-reference/unityengine/time/capturedeltatime)对 `Time.captureDeltaTime` 的描述是：设为非零后，`Time.time` 每帧按该间隔推进，与真实时间和帧耗时无关。同一页紧接着写明：它对 `Time.unscaledTime` 没有任何影响，依赖 unscaled time 的动画和效果不会被锁住。

所以它只锁住一条时钟。Unity 里实际并存着好几条：

|时钟|典型使用者|`captureDeltaTime` 能否锁住|
|---|---|---|
|缩放游戏时间（`Time.time`、`deltaTime`）|大多数游戏逻辑、Animator、粒子、`WaitForSeconds`|能，文档明确|
|非缩放时间（`unscaledTime`、`unscaledDeltaTime`）|UI 动效、暂停菜单、tween 库的"忽略 timeScale"模式|不能，文档明确|
|墙钟（`realtimeSinceStartup`、`Stopwatch`、`DateTime.Now`）|限流、超时、冷却、性能自适应逻辑|不能，它就是真实时间|
|音频 DSP 时钟（[`AudioSettings.dspTime`](https://docs.unity3d.com/ScriptReference/AudioSettings-dspTime.html)）|音乐同步、按节拍触发的事件|不能，它由音频系统处理的采样数驱动|
|视频时钟|Video Player|[可设为与 captureFramerate 同步](https://docs.unity.com/en-us/engine/7000.0/manual/video/player/clock)|
|异步完成时刻|资源异步加载、`Task.Delay`、后台线程回调|不能，结果落在哪一帧取决于真实耗时|

Shader 的 `_Time` 属于哪一条，我搜了两轮都没有找到文档依据。凭记忆它跟随缩放游戏时间，但是这一点就不知道了，要拿您的水面 shader 实测。

## 这件事的性质

痛点的根源在于：游戏是按"世界与玩家共享一条墙钟"的假设写的，而训练环境要求世界只在被显式步进时前进。VLA 推理的那几百毫秒里，凡是挂在后四条时钟上的东西都会偷偷往前走。

由此有两个推论：

1. **换引擎不解决这个问题。** Asset Store 库里读 `Stopwatch` 的代码搬到任何引擎上还是读墙钟。自研引擎唯一的好处是能从底层保证只有一条时钟，但前提是所有上层代码都重写。
2. **严重程度可以分级。** 训练需要的是因果一致，即模型思考期间世界不动，未必需要逐位确定性。一条泄漏的时钟只有在影响第一视角画面或影响物理状态时才要紧。音频时钟在 VLA 不吃音频输入时可以忽略；UI 动效不进入机器人视野时也可以忽略。

## 把它从"痛"变成"有限清单"

两种手段，互相补充：

**静态审计**：上表每条时钟对应一组有限的 API 名称，可以在全部 C# 源码里搜出来；Asset Store 库若只有编译好的 DLL，就扫 IL 里的方法引用。产出是一张"谁在读哪条时钟"的表，逐项判断是否可见、是否影响物理。

**动态检测**：同一段动作序列跑两遍，第二遍在每步之间插入随机长度的真实等待，然后逐帧比对画面和物理状态。两遍出现差异的地方就是墙钟泄漏点。这个方法不需要读懂任何库的源码，漏网之鱼会自己浮上来。

修补手段按侵入程度排序：tween 类库通常自带"使用缩放时间"的开关；自己的代码把时间读取统一收口到一个仿真时钟接口；闭源库里顽固的调用可以在 IL 层替换；实在改不动且又可见的效果，从训练场景里拿掉。

工程路径给我之后，静态审计这一步我可以只读地做出来，您就能看到这张清单实际有多长。

Sources:

- [Unity: Time.captureDeltaTime](https://docs.unity.com/en-us/engine/current/script-reference/unityengine/time/capturedeltatime)
- [Unity: AudioSettings.dspTime](https://docs.unity3d.com/ScriptReference/AudioSettings-dspTime.html)
- [Unity: Video Player 时钟管理](https://docs.unity.com/en-us/engine/7000.0/manual/video/player/clock)
- [Unity Discussions: Recorder 与 unscaled time](https://discussions.unity.com/t/unity-recorder-vs-systems-with-unscaled-time/846491)
- [Unity Discussions: 离线逐帧渲染](https://discussions.unity.com/t/offline-rendering-of-video-frames-with-exactness-possible/908314)
- 