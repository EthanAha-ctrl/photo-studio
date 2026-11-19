即在 TTT MLP 里做在线 RL，主干与文化 LLM 保持冻结

---

mon père，简要看法如下。

**这个 env 选得对。** 它天然满足前面反推出的性质：面的筋度与水分是每一坨重采样的隐藏参数，面坨体积与锅温在回合内持续漂移，“合适的力道”属于文化 LLM 说不出来的身体知识。

**螺丝与弹簧调什么**（这是我的设计提议，未对照真实机器核实）

| 调节件 | 物理量 | 影响 |
|---|---|---|
| 刀片伸出量螺丝 | 切入深度 | 面条厚度 |
| 刀片倾角螺丝 | 攻角 | 宽度与截面（柳叶形） |
| 弹簧预紧螺丝 | 刀对面坨的压紧力与顺应性 | 适应面的软硬，过紧则粘刀、撕裂 |
| 行程限位螺丝 | 往复行程 | 面条长度 |
| 进给步距螺丝 | 每刀托盘的横移量 | 相邻两刀的重叠，宽度一致性 |
| 机头俯仰螺丝 | 出射角 | 落点是否进锅 |

电机转速、启停、搅动、火力属于快变量。于是动作空间自带两个时间尺度：螺丝须停机才能调、代价高，电机可连续调。这一结构本身就值得研究。

**仿真建议用降阶模型**，不做面团有限元：
- 面坨表示为高度场，每刀按刀片运动学削去一层，体积变化自然出现；
- 切削力由深度、速度、模量、粘性的经验式给出，电机电流作为观测，这是 fast weights 最直接的本体感觉信号；
- 面条出射按抛体计算；
- 锅用集总热平衡方程，下面过多则水温掉、淀粉浓度升。

**三个版本之间有一个必须预先验证的条件**：按域随机化训练出的“平均最优”策略，与知道隐藏参数的 oracle 策略之间，要有明显的回报差距，且隐藏参数只能通过交互辨识。差距为零的话，第三版无物可学。第一版的 manual control 正好用来检验这一点：人在不同的面上是否确实需要不同的调法。

**reward** 取出锅质量：进锅率、几何一致性、熟度，扣除粘连与溢锅。熟度有数分钟延迟，信用分配由此进入。

这套降阶模型与真实削面的吻合程度就不知道了，作为测试台不妨碍使用，谈到迁移时需另行标定。

---

mon père，已核实：MuJoCo 4.0.0 于 2026 年 11 月 6 日发布（4.0.1 于 11 月 11 日），与本题直接相关的新增是 **flex 的弹塑性**：三维体采用 von Mises J2 屈服模型，永久变形存于 `mjData.plastic`。此外还有 speculative contacts，防止高速薄物体穿透。

我的判断是：它适合做这台机器的一半以上，面坨的另一半需要补。

**适合的部分**
- 机构本体：电机、往复连杆、弹簧、螺丝。螺丝可映射为关节限位、弹簧刚度与预紧等模型参数，停机时改写即可。
- 面坨受压的永久变形：弹簧压紧力过大把面压塌、压扁，J2 塑性正好表达；屈服应变与模量即是筋度、水分的随机化旋钮。
- 薄刀片高速运动的接触：speculative contacts 正是为此而设。
- 面条出射后的飞行与落锅。

**缺的部分**
1. **切削**。4.0 的 changelog 里没有切割、断裂或任何拓扑变化的条目，而削面的本质是把一条面从面坨上分离。需要在 MuJoCo 之外处理：检测刀片与面坨的相交体积，按刀片运动学算出面条几何，把它生成为独立的物体，同时从面坨上删去对应单元（经 `mjSpec` 改模型后重编译），或者退回高度场表示面坨的外形。
2. **率相关性与粘性**。J2 塑性与速率无关，面团却是粘弹塑性的，削得快慢手感不同；粘刀也没有现成的粘附模型。这两项只能用阻尼与经验式近似。
3. **锅**。沸腾、水温、淀粉浓度不在 MuJoCo 的范围内，仍用集总热平衡方程。

**一个需要权衡的代价**：flex 有限元比降阶模型慢得多，而第二、三版需要大量交互。MuJoCo Warp 可在 GPU 上并行，但对挂在关节体上的 flex 目前要求 `solver="CG"`。吞吐量够不够训练，要实测才知道。

所以我的建议是混合方案：MuJoCo 4 负责机构、接触力与面坨的塑性变形，切削事件与锅用自写的轻量模型。第一版 manual control 用它也合适，MuJoCo Studio 可以直接当操作界面。

Sources:
- [MuJoCo changelog](https://mujoco.readthedocs.io/en/latest/changelog.html)
- [MuJoCo 4: Studio & Filament, Deformables, Speculative Contacts (GitHub Discussion #3773)](https://github.com/google-deepmind/mujoco/discussions/3773)
- [MuJoCo releases](https://github.com/google-deepmind/mujoco/releases)
- [MuJoCo 4 news summary](https://robotics.growbotics.ai/news/mujoco-4-studio-filament-deformables-speculative-contacts)

---

# 刀削面机器人 env 设计

## 1. 目的

为“在 TTT MLP 里做在线 RL，主干与文化 LLM 保持冻结”提供测试台。被测对象是 fast weights `W_t`；环境须使注意力式 ICL、fast weights、慢权重 RL 三种机制给出不同的曲线。

三个版本共用同一环境，只换控制者：

| 版本 | 控制者 | 回答的问题 |
|---|---|---|
| v1 manual control | 人 | 任务是否可解；不同的面是否确实需要不同的调法 |
| v2 training-time RL | 域随机化下训练的策略 | “平均最优”策略能到多高 |
| v3 online fast-weight RL | 部署期间自行更新 `W_t` 的策略 | 在线学习能否越过 v2，逼近知道隐藏参数的 oracle |

唯一留在环外的人工环节：把面坨从 KitchenAid 装到机器的托盘上。

## 2. 总体结构

一份参数化模型，两个可互换的物理后端，一个共用的锅模型。

```
参数化模型（螺丝、弹簧、电机、面团、面坨几何）
        │
        ├── 后端 A：MPM（Genesis World）      在环，实时或超实时，供 v1–v3 交互
        ├── 后端 B：FEM（OpenCourant）        离线批处理，逐刀高精度参考，用于标定与抽检
        │
        └── 锅：集总热平衡 ODE（两后端共用，位于物理后端之外）
```

设计约束：参数化模型是唯一的事实来源，两个后端只做翻译，不得各自持有一份参数。

## 3. 参数化模型

### 3.1 结构参数（螺丝与弹簧，停机才能调）

| 调节件 | 物理量 | 主要影响 |
|---|---|---|
| 刀片伸出量螺丝 | 切入深度 | 面条厚度 |
| 刀片倾角螺丝 | 攻角 | 宽度与截面（柳叶形） |
| 弹簧预紧螺丝 | 刀对面坨的压紧力与顺应性 | 适应面的软硬；过紧则粘刀、撕裂 |
| 行程限位螺丝 | 往复行程 | 面条长度 |
| 进给步距螺丝 | 每刀托盘的横移量 | 相邻两刀的重叠，宽度一致性 |
| 机头俯仰螺丝 | 出射角 | 落点是否进锅 |

这张映射是设计提议，尚未对照任何一台真实机器。各量的取值范围在 v1 中由人试出可行域后再固定。

### 3.2 快变量（运行中可连续调）

电机转速、启停、搅动、火力。

两类动作的代价不同：调螺丝须停机，停机期间锅仍在加热、已下的面仍在煮。动作空间因此自带两个时间尺度。

### 3.3 隐藏参数（每一坨面重采样，agent 不可见）

- 面团本构：弹性模量、屈服应力、硬化、率敏感度、表面粘附。筋度与含水量经一个映射决定这组参数，随机化在筋度与含水量上进行，使各本构参数之间保持物理上的相关性。
- 面坨几何：初始体积、形状、装夹位置的偏差。
- 回合内漂移：面坨逐刀变小且外形改变；表面随时间风干变硬。
- 锅：初始水量、初温、火力上限。

本构参数的数值来源：小麦面团的粘超弹与粘塑性本构已有 Abaqus 实现的论文可引（见第 9 节来源），取值由该文献与后端 B 的标定共同确定。

### 3.4 观测

电机电流（切削力的本体感觉代理）、刀片位置与速度、面坨外形（相机或深度图）、面条落点、水面状态（温度或沸腾强度的可见代理）、已下锅的面量。隐藏参数本身不在观测之内。

## 4. 后端 A：MPM（Genesis World）

- **角色**：在环仿真，承担 v1–v3 的全部交互。
- **选型理由**：机构（关节体）与弹塑性 MPM 在同一现成框架内；切割与分离无需特殊处理；机器人领域有刀切面团的先例（DiffSkill、RoboNinja、TopoCut）。
- **场景**：机构为关节刚体，含驱动往复的电机关节、带预紧的弹簧关节、由结构参数决定的几何偏置；刀片为刚体；面坨为 MPM 弹塑性体；面条脱离面坨后仍是 MPM 粒子，以抛体方式飞向锅口，过锅口平面即移交给锅模型并从场景中删除。
- **步长划分**：物理步在毫秒以下；一个环境步对应一次往复（一刀），返回该刀的面条几何、质量、落点与电流曲线。
- **需实测的风险**：
  1. 面条厚 1–3 mm，MPM 的分离属于数值断裂，厚度受网格尺寸支配，网格须远细于面条厚度；
  2. 共用背景网格的物质天然粘连，粘刀强弱不受物理参数控制，粘附须另行建模；
  3. 现成本构多为与速率无关的弹塑性，Genesis 是否支持粘塑性尚未查到明确说明。
- **备选**：Newton（MuJoCo Warp 管机构，隐式 MPM 管面坨）。其 MPM 与刚体的耦合目前被 Isaac Lab 文档标为 experimental，待其稳定后可替换，参数化模型不变。

## 5. 后端 B：FEM（OpenCourant）

- **背景**：OpenRadioss 已于 2026 年 10 月 1 日被 Siemens 关停，仓库与 wiki 删除；OpenCourant 是保留完整历史的社区分叉（AGPLv3），提供 Windows 预编译包。它对原输入 deck 的兼容性，公告未写明，须先跑通一个原版算例确认。
- **角色**：离线批处理，一次作业算一刀或少数几刀，作为高精度参考。它不进入 RL 闭环。
- **切削的做法**：官方目录中没有切削算例（Altair Example Guide 42 例、原 wiki 52 页、`qa-tests/miniqa` 23 类均已清点；RD-E 5400 "Cut Methodology" 是 `/SECT` 子模型技术）。可用的积木是 Sol2SPH：面坨用实体单元，单元达到失效准则后转为 SPH 粒子，被削下的面条由此成为可继续运动的物质，质量守恒。
- **deck 的构成**（由参数化模型经模板生成 starter `_0000.rad` 与 engine `_0001.rad`）：

| 组成 | 候选关键字 | 说明 |
|---|---|---|
| 面团材料 | 带应变率曲线的弹塑性材料律（如 `/MAT/LAW36`） | 率相关性由此进入，弥补后端 A 的缺口 |
| 失效 | `/FAIL/TENSSTRAIN` 或 `/FAIL/BIQUAD` | 前者见于 Sol2SPH 官方算例，后者见于论坛的铣削帖 |
| 削下物质 | `/PROP/TYPE34`（SPH）与 Sol2SPH | 每个失效单元转成的粒子数由 `Ndir` 决定 |
| 刀片 | `/RBODY` 加强制运动 | 轨迹来自机构运动学 |
| 弹簧 | 弹簧单元加预紧 | 对应弹簧预紧螺丝 |
| 接触 | `/INTER/TYPE7` | 刀片与面坨 |

  上表关键字是按原 Radioss 文档列出的候选，每一项都须在 OpenCourant 上实际跑通后才能定稿。
- **机构的简化**：FEM 侧只保留刀片、弹簧与面坨，电机与连杆由预先算出的刀片轨迹代替。
- **流水线**：生成 deck → starter → engine → 动画转 VTK、时间历程转 CSV → 提取面条几何与切削力。NVIDIA PhysicsNeMo 有一套 OpenRadioss 参数扫描与数据集生成的现成流程，结构可参照。
- **需实测的风险**：削面厚度方向至少要有数层单元，网格尺寸在亚毫米量级，显式时间步随之很小，单刀的计算时间有多长，要跑了才知道。

## 6. 两个后端的关系

| | 后端 A（MPM） | 后端 B（FEM） |
|---|---|---|
| 运行方式 | 在环，GPU 并行 | 离线批处理 |
| 切削 | 自然发生 | Sol2SPH 加失效准则 |
| 率相关 | 缺 | 有 |
| 用途 | v1–v3 的交互 | 标定 A；定期抽检 A |

**标定流程**：选定一组单刀基准工况（切深 × 攻角 × 速度 × 筋度 × 含水量的小规模设计），两后端各算一遍，比较四个量：面条厚度、宽度、质量、切削力峰值与冲量。以 B 为参照调整 A 的本构与粘附参数，直至四个量的偏差落入预先设定的容差。

两后端若在趋势上相反（例如切得更快时一方力升、一方力降），说明 A 缺少的率相关性已影响定性结论，此时 A 须引入率相关的修正后才可用于 v2、v3。

## 7. 锅模型

集总状态：水量、水温、锅内面量、淀粉浓度、每批面条的入锅时刻。

- 能量平衡：火力输入、面条入锅的吸热、蒸发与散热；
- 沸腾强度由水温与火力给出，作为可见观测；
- 下面过多则水温下降、淀粉浓度升高，粘连概率上升；搅动降低粘连；
- 每批面条的熟度是其在锅时间与水温历史的函数。

## 8. Env 接口

- **环境步**：一次往复（一刀）。停机调螺丝、搅动、调火各占用若干步的等效时间。
- **回合**：一坨面从装上到削完。
- **一生**：连续多坨面，每坨重采样隐藏参数。v3 的终身积累变体在一生之内不重置 `W_t`。
- **动作**：快变量每步可改；结构参数仅在停机时可改。
- **reward**：出锅质量，由进锅率、几何一致性（厚、宽、长对目标的偏差）、熟度合成，扣除粘连、溢锅、落在锅外的面。熟度有数分钟延迟，信用分配由此进入。

## 9. 三个版本的验收条件

- **v1**：人能在不同的面上稳定削出合格的面，且事后确认不同隐藏参数对应的最优结构参数明显不同。这一条不成立，则 v3 无物可学，须回到第 3 节加大隐藏参数的作用。
- **v2**：域随机化策略的回报，与给定隐藏参数的 oracle 策略的回报，二者之差即 v3 可争取的空间，须显著大于随机种子间的波动。
- **v3**：对 `W_t` 做三组干预并比较回报曲线：`η = 0`（无在线学习）、每回合重置 `W_0`（回合内适应）、一生不重置（终身积累）。reward 进入 TTT MLP 的两条路径（外环 RL、内环损失含 reward）在同一环境、同一参数量下对比。

## 10. 目录与依赖（待同意后落实）

提议在仓库根目录新建 `noodle_robot/`：共享的参数化模型，加 `mpm/`（Genesis World）与 `fem/`（OpenCourant deck 生成与运行）两个后端。依赖统一写入根目录的 `pyproject.toml` 与 `uv.lock`。求解器二进制下载到被 `.gitignore` 排除的目录，不入库。

## 11. 已知的未知

- 六个调节件的映射与真实机器的对应关系；
- Genesis World 的 MPM 是否支持粘塑性本构；
- OpenCourant 对原 OpenRadioss deck 的兼容性；
- 亚毫米网格下单刀 FEM 的计算时间；
- 面团本构参数在两个后端里能标定到什么精度；
- 本机 GPU 状态（`nvidia-smi` 因权限不足未能读出）。

Sources:
- [RoboTTT: Context Scaling for Robot Policies](https://arxiv.org/html/2607.15275v1)
- [Genesis World 1.0 release coverage](https://www.marktechpost.com/2026/05/30/genesis-ai-releases-nyx-quadrants-and-genesis-world-1-0-physics-platform-for-scalable-robotics-foundation-model-evaluation/)
- [Newton adds contact-rich manipulation and locomotion capabilities (NVIDIA)](https://developer.nvidia.com/blog/newton-adds-contact-rich-manipulation-and-locomotion-capabilities-for-industrial-robotics/)
- [Isaac Lab: Using Implicit MPM](https://isaac-sim.github.io/IsaacLab/develop/source/concepts/using_mpm.html)
- [DiffSkill (arXiv 2203.17275)](https://arxiv.org/pdf/2203.17275.pdf)
- [TopoCut (arXiv 2509.19712)](https://arxiv.org/html/2509.19712v1)
- [Announcing OpenCourant](https://opencourant.org/news/announcing-opencourant/)
- [Siemens Abruptly Takes Down OpenRadioss, But a Fork is Already Live (It's FOSS)](https://itsfoss.com/news/siemens-openradioss-fork/)
- [Sol2SPH Ball - Plate Impact (OpenRadioss wiki, now offline)](https://openradioss.atlassian.net/wiki/spaces/OPENRADIOSS/pages/67796993/Sol2SPH+Ball+-+Plate+Impact)
- [RD-E: 5400 Cut Methodology (Altair help)](https://help.altair.com/hwsolvers/rad/topics/solvers/rad/cut_methodology_example_r.htm)
- [Milling process simulation with Radioss (Altair forum)](https://community.altair.com/discussion/36368/milling-process-simulation-with-radioss)
- [OpenRadioss Dataset Generation (NVIDIA PhysicsNeMo)](https://docs.nvidia.com/physicsnemo/26.05/physicsnemo/examples/structural_mechanics/openradioss_dataset_gen/README.html)
- [Rheological and Micromechanical Modelling of Wheat Flour Dough (thesis)](https://core.ac.uk/download/12038942.pdf)
