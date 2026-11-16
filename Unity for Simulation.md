
- [Prowl](https://github.com/ProwlEngine/Prowl)

| 需求                | Unity 内的现成机制                                                                                                                                  | 状况               |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| 接入 MuJoCo         | [官方 Unity 插件](https://mujoco.readthedocs.io/en/3.6.0/unity.html)，随 MuJoCo 发行，3.11 文档仍在维护                                                      | 可用，有下述限制         |
| 物理步长脱离 wall clock | 把 simulation mode 设为 Script 后手动调用 [`Physics.Simulate`](https://docs.unity3d.com/ScriptReference/Physics.Simulate.html)，每次传固定步长                | 可用               |
| 由外部模型驱动步进         | ML-Agents 的 [`DisableAutomaticStepping()` 与 `EnvironmentStep()`](https://docs.unity3d.com/Packages/com.unity.ml-agents@3.0/manual/index.html) | 可用，包在 2026 年仍有发布 |

机器人本体和可操作物体归 MuJoCo，静态场景几何导出成碰撞网格给 MuJoCo，其余装饰性物理留在 PhysX。两个世界之间的耦合（比如机器人推开一个由 PhysX 管的门）需要case by case 看。

### 游戏逻辑里的时间

`Physics.Simulate` 只管物理。动画、粒子、协程里的 `WaitForSeconds`、shader 里的 `_Time`（水面动画就靠它）都跟着 Unity 的帧时间走。凭记忆，`Time.captureDeltaTime` 能让每帧推进固定的仿真时间，从而把这些一并锁定，但本轮检索没有覆盖到它，需要实测。Asset Store 库里若有直接读系统时钟的代码，要逐个找出来。



- 大规模并行：Unity 是一个进程一个环境。大规模强化学习，这是结构性差距，需要换成 MuJoCo MJX 那一类架构。
- 集群上的headless rendering：AI2-THOR 的 headless 版本当年[依赖 Unity Simulation Pro](https://blog.allenai.org/ai2-thor-partners-with-unity-to-enable-headless-multi-node-training-in-the-cloud-571802be5ad3)。旧版 Unity Simulation 已于 [2023 年 12 月弃用](https://docs.unity3d.com/Simulation/manual/index.html)；新版 Simulation Pro 在 [2026 年 9 月以 Early Access 发布](https://unity.com/blog/unity-industry-product-releases-2026)，带 headless Linux 构建目标，但要求 Unity Industry 许可。

先在 Unity 内做一个最小闭环：一个场景、一个 MuJoCo 机器人、手动步进、第一视角图像送出、动作送回。这一步能直接暴露上面第 1、2 项的真实成本，并且完全不涉及移植。
Unity [官方文档](https://docs.unity.com/en-us/engine/current/script-reference/unityengine/time/capturedeltatime)对 `Time.captureDeltaTime` 的描述是：设为非零后，`Time.time` 每帧按该间隔推进，与真实时间和帧耗时无关。同一页紧接着写明：它对 `Time.unscaledTime` 没有任何影响，依赖 unscaled time 的动画和效果不会被锁住。所以它只锁住一条clock。Unity 里实际并存着好几条：

| 时钟                                                                                                        | 典型使用者                                | `captureDeltaTime` 能否锁住                                                                          |
| --------------------------------------------------------------------------------------------------------- | ------------------------------------ | ------------------------------------------------------------------------------------------------ |
| 缩放游戏时间（`Time.time`、`deltaTime`）                                                                           | 大多数游戏逻辑、Animator、粒子、`WaitForSeconds` | 能，文档明确                                                                                           |
| 非缩放时间（`unscaledTime`、`unscaledDeltaTime`）                                                                 | UI 动效、暂停菜单、tween 库的"忽略 timeScale"模式  | 不能，文档明确                                                                                          |
| 墙钟（`realtimeSinceStartup`、`Stopwatch`、`DateTime.Now`）                                                     | 限流、超时、冷却、性能自适应逻辑                     | 不能，它就是真实时间                                                                                       |
| 音频 DSP 时钟（[`AudioSettings.dspTime`](https://docs.unity3d.com/ScriptReference/AudioSettings-dspTime.html)） | 音乐同步、按节拍触发的事件                        | 不能，它由音频系统处理的采样数驱动                                                                                |
| 视频时钟                                                                                                      | Video Player                         | [可设为与 captureFramerate 同步](https://docs.unity.com/en-us/engine/7000.0/manual/video/player/clock) |
| 异步完成时刻                                                                                                    | 资源异步加载、`Task.Delay`、后台线程回调           | 不能，结果落在哪一帧取决于真实耗时                                                                                |

Shader 的 `_Time` 属于哪一条，我搜了两轮都没有找到文档依据, 要拿您的水面 shader 实测。

1. Asset Store 库里读 `Stopwatch` 的代码搬到任何引擎上还是读墙钟。自研引擎唯一的好处是能从底层保证只有一条时钟，但前提是所有上层代码都重写。
2. 严重程度可以分级。 训练需要的是因果一致，即模型思考期间世界不动，未必需要逐位确定性。一条泄漏的时钟只有在影响第一视角画面或影响物理状态时才要紧。音频时钟在 VLA 不吃音频输入时可以忽略；UI 动效不进入机器人视野时也可以忽略。

静态审计：上表每条时钟对应一组有限的 API 名称，可以在全部 C# 源码里搜出来；Asset Store 库若只有编译好的 DLL，就扫 IL 里的方法引用。产出是一张"谁在读哪条时钟"的表，逐项判断是否可见、是否影响物理。

动态检测：同一段动作序列跑两遍，第二遍在每步之间插入随机长度的真实等待，然后逐帧比对画面和物理状态。两遍出现差异的地方就是墙钟泄漏点。这个方法不需要读懂任何库的源码，漏网之鱼会自己浮上来。

修补手段按侵入程度排序：tween 类库通常自带"使用缩放时间"的开关；自己的代码把时间读取统一收口到一个仿真时钟接口；闭源库里顽固的调用可以在 IL 层替换；实在改不动且又可见的效果，从训练场景里拿掉。