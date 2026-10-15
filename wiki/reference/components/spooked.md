---
title: spooked
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "复用万圣夜的工作惊吓积累，区分个人蝙蝠幻象、通用惊吓特效、角色动作和精神损失。"
aliases: ["spooked", "砍树惊吓", "采矿惊吓", "挖坟惊吓", "万圣夜蝙蝠幻象", "TryCustomSpook"]
tags: [component, callback, event, state-machine, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/spooked.lua", "components/age.lua", "actions.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "stategraphs/SGwilson.lua", "prefabs/battreefx.lua", "prefabs/spooked_fx.lua", "prefabs/rocks.lua", "prefabs/mound.lua", "componentutil.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# spooked

## 快速判断

- 职责与适用：为工作者积累惊吓量，抽中时生成特效并延迟发送 `spooked`；原版在万圣夜活动期间给玩家服务端挂载并注册精神响应。不是持续精神光环，也不是树精或敌人的恐惧 AI。[定义:37](../../../raw/current/scripts/components/spooked.lua#L37)、[玩家挂载:2746](../../../raw/current/scripts/prefabs/player_common.lua#L2746)、[服务端分界:2613](../../../raw/current/scripts/prefabs/player_common.lua#L2613)
- 优先使用：原版树木工作走 `Spook(source)`；自定义工作场景参考 `TryCustomSpook(source, fxprefab, mult)` 的采矿／挖掘调用。两者积累公式不同，不可照搬倍率。[Spook:69](../../../raw/current/scripts/components/spooked.lua#L69)、[自定义:102](../../../raw/current/scripts/components/spooked.lua#L102)
- 重要副作用：触发后清空积累，给自身排入带 `{source = source}` 的事件；SG 惊吓动作和精神损失是两个独立消费者，特效已生成不代表动作一定执行。[事件:65](../../../raw/current/scripts/components/spooked.lua#L65)、[SG 门禁:2413](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2413)、[精神监听:895](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L895)
- 修改边界：复用已有玩家组件；组件缺失时不擅自补挂并假定活动行为齐全。普通 `Spook` 依赖源码内固定树种／生长阶段映射，自定义 Prefab 不会自动匹配。[映射:1](../../../raw/current/scripts/components/spooked.lua#L1)、[匹配:79](../../../raw/current/scripts/components/spooked.lua#L79)

## 关键接口

| 接口／字段 | 约定 | 条件、作用与证据 |
|---|---|---|
| `spookedlevel`、`spookedthreshold`、`maxspookedlevel` | 初始 0、70、100 | 只有严格高于阈值才抽取；调整两者需同时复查门禁与概率公式，阈值不低于上限时，正常封顶积累不能通过门禁。[默认值:39](../../../raw/current/scripts/components/spooked.lua#L39)、[公式:56](../../../raw/current/scripts/components/spooked.lua#L56) |
| `maxspookdelta`、`maxspookage` | 3.5；默认严酷季节长度 × 每天秒数 | 前者只用于普通 Spook；年龄来自 `age:GetAge()` 的秒数，缺 age 按完整年龄因子计算。[默认值:42](../../../raw/current/scripts/components/spooked.lua#L42)、[年龄使用:71](../../../raw/current/scripts/components/spooked.lua#L71)、[GetAge:51](../../../raw/current/scripts/components/age.lua#L51) |
| `CalcSpookedLevelDecay(t)` | 可选当前时间，返回衰减值 | `max(0, spookedlevel - (t-lastspooktime)^2)`；只计算，不回写字段，不是每秒减固定值。[实现:60](../../../raw/current/scripts/components/spooked.lua#L60) |
| `IsSpookImmune()` | 无参，返回布尔（26230496 新增） | 勇气药水 buff 或 `wereplayer` 标签即免疫；从 ShouldSpook 抽出的独立判定，可单独查询不改积累。[实现:47](../../../raw/current/scripts/components/spooked.lua#L47) |
| `ShouldSpook()` | 返回抽取结果 | 低于或等于阈值、或 `IsSpookImmune()` 命中时 false；否则以归一化积累的平方传给幸运公式。只检查当前字段，不主动衰减。[实现:52](../../../raw/current/scripts/components/spooked.lua#L52) |
| `Spook(source)` | source 为有效树木实体，无成功返回约定 | 先更新积累／时间，再做怪树、工作状态和动画映射筛选；成功生成个人 `battreefx`，8 帧（woodcutter）或 10 帧后发事件并归零。[实现:69](../../../raw/current/scripts/components/spooked.lua#L69)、[生成与排队:89](../../../raw/current/scripts/components/spooked.lua#L89) |
| `TryCustomSpook(source, fxprefab, mult)` | mult 默认 1；source 需 Transform | 不走树种映射和 CHOP 门禁；触发时生成指定 FX 并定位，10 帧后发事件并归零。[实现:102](../../../raw/current/scripts/components/spooked.lua#L102) |
| `TryForcingCustomSpook(source, fxprefab)` | source 需 Transform（26230496 新增） | 强制惊吓：免疫检查通过后在 source 位置生成 FX、10 帧后发事件并归零，**无视当前 spookedlevel 门槛**（不走 ShouldSpook／幸运抽取），返回 true；免疫时返回 false 且无副作用。leafpile trick 掉落用 `spooked_worms_fx` 调用。[实现:119-129](../../../raw/current/scripts/components/spooked.lua#L119-L129) |
| `GetDebugString()` | 返回衰减后的显示文本 | 不改变积累，不触发抽取或事件。[实现:131](../../../raw/current/scripts/components/spooked.lua#L131) |

### 积累与概率如何计算

令 `a = min(1, age秒数 / maxspookage)`（缺 age 时为 1）。每次调用先取平方时间衰减值，普通 Spook 加 `a² × 3.5`，自定义默认加 `a² × 100 × mult`，再限制上限；均更新 `lastspooktime`。因此新生玩家不等于立即满概率，间隔调用的衰减也不等于连续高频调用。[普通积累:70](../../../raw/current/scripts/components/spooked.lua#L70)、[自定义积累及源码 NOTE:103](../../../raw/current/scripts/components/spooked.lua#L103)

`k = (level-threshold)/(max-threshold)` 后使用 `TryLuckRoll(inst, k², LuckFormulas.SpookedChance)`；最终概率还经过角色幸运度变换，不能把 `k²` 宣称为所有角色的最终概率。[调用:56](../../../raw/current/scripts/components/spooked.lua#L56)、[幸运公式:2175](../../../raw/current/scripts/componentutil.lua#L2175)、[抽取:2237](../../../raw/current/scripts/componentutil.lua#L2237)

## 典型接入

### 在已有工作流程中选择惊吓触发点

不提供脱离原工作逻辑的独立 demo：下表是已核验的服务端接入位置，新增模组工作物应在自己的成功工作／状态变化回调中复用，先确认 `worker.components.spooked` 存在、source 仍有效、特效资源可加载，再决定调用频率；不能每帧调用自定义接口。

| 需求模式 | 原版接入 | 选择与生效边界 |
|---|---|---|
| 每次成功砍树后尝试 | `ACTIONS.CHOP` 在成功工作且目标仍有效后 `Spook(target)`。[actions.lua:1748](../../../raw/current/scripts/actions.lua#L1748 | 普通接口还要求目标是旧生长阶段 4，或仍可 CHOP；怪树先返回。树种必须有映射；失败筛选之前积累已增加。[筛选:75](../../../raw/current/scripts/components/spooked.lua#L75) |
| 采矿破坏外观变档才尝试 | 岩石变为 med／low 且动画有变化时，自定义蜘蛛 FX，分别用 HIGH／LOW 倍率。[rocks.lua:183](../../../raw/current/scripts/prefabs/rocks.lua#L183 | 不是每次敲击；当前 HIGH=0.315、LOW=0.1，基准是自定义的 100。[tuning.lua:9034](../../../raw/current/scripts/tuning.lua#L9034 |
| 挖坟工作完成后尝试 | 坟丘回调检查工作者组件，传 `spooked_worms_fx` 与 DIG 倍率。[mound.lua:123](../../../raw/current/scripts/prefabs/mound.lua#L123 | DIG=0.456；FX 是预制物资源，不是动画名称。[常量:9036](../../../raw/current/scripts/tuning.lua#L9036)、[FX 注册:53](../../../raw/current/scripts/prefabs/spooked_fx.lua#L53) |

### 视觉、动作与精神分别由谁负责

- **个人蝙蝠幻象**：普通 Spook 调 `SetViewerAndAnim(player, anim)`，`battreefx` 用 classified target 指向该玩家；客户端排 `batspooked` 以触发蝙蝠覆盖层。这个客户端事件不同于服务端 `spooked`。[配置:41](../../../raw/current/scripts/prefabs/battreefx.lua#L41)、[覆盖层事件:33](../../../raw/current/scripts/prefabs/battreefx.lua#L33)、[客户端工厂:73](../../../raw/current/scripts/prefabs/battreefx.lua#L73)
- **通用蜘蛛／虫子特效**：自定义接口只 SpawnPrefab／定位，不设置个人 viewer；已核验 FX 是带 Network 的普通实体，播放动画后移除，不承诺只让被吓者看见。[生成:110](../../../raw/current/scripts/components/spooked.lua#L110)、[FX 工厂:20](../../../raw/current/scripts/prefabs/spooked_fx.lua#L20)、[移除:44](../../../raw/current/scripts/prefabs/spooked_fx.lua#L44)
- **动作响应**：SGwilson 收到 `spooked` 时检查非 busy、未死、未骑乘、未装备 `spook_protection`；通过才进入状态，停止移动、清动作、播放动画并暂停客户端预测。[门禁:2413](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2413)、[状态:15130](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15130)
- **精神损失**：玩家另外监听同一事件，若非 no_sanity 模式且当时没装备 `spook_protection`，1.35 秒后扣 `TUNING.SANITY_SMALL`；这条监听不检查 SG 的 busy／骑乘条件。仅有视觉效果、或仅拒绝状态切换，都不能等同免扣精神。[精神回调:893-903](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L893-L903)、[挂载监听:2746](../../../raw/current/scripts/prefabs/player_common.lua#L2746)

## 常见陷阱与限制

- **把保护装备当成禁生成 FX**：`ShouldSpook` 不检查 `spook_protection`，保护在动作与精神消费者分别检查。视觉可能仍生成，积累也可能归零；若修改免疫需求，明确阻止的是抽取、视觉、动作还是精神；`TryForcingCustomSpook` 同样尊重 IsSpookImmune 但绕过抽取。[抽取:52](../../../raw/current/scripts/components/spooked.lua#L52)、[触发归零:89](../../../raw/current/scripts/components/spooked.lua#L89)
- **勇气／变身期间完全不积累**：两接口先增加积累，再调用 ShouldSpook；buff 与 wereplayer 阻止抽取，但不阻止积累更新。普通接口也可能在未匹配树种时留下积累。[普通顺序:69](../../../raw/current/scripts/components/spooked.lua#L69)、[自定义顺序:102](../../../raw/current/scripts/components/spooked.lua#L102)
- **把自定义返回值当 FX 实体**：实现把 `SpawnPrefab(fxprefab).Transform:SetPosition(...)` 的返回值赋给 `fx` 后返回，不是单独保存生成实体；本页未核验引擎 SetPosition 返回约定，不据此调用 `fx:Remove()`。[赋值:113](../../../raw/current/scripts/components/spooked.lua#L113)
- **任意 FX／source 都安全**：自定义路径没有 SpawnPrefab 失败或 source 缺失检查；普通路径还读取 source.monster／components。调用方负责有效对象与注册特效，不把接口视为资源探测器。[普通读取:75](../../../raw/current/scripts/components/spooked.lua#L75)、[自定义读取:111](../../../raw/current/scripts/components/spooked.lua#L111)
- **存档恢复／移组件会自动撤销惊吓**：完整定义没有 OnSave／OnLoad 或移组件清理；已排的事件任务挂在玩家实体上，移除组件本身不在该实现中取消它。[定义:37](../../../raw/current/scripts/components/spooked.lua#L37)、[排队:97](../../../raw/current/scripts/components/spooked.lua#L97)、[自定义排队:113](../../../raw/current/scripts/components/spooked.lua#L113)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 中漂移文件 actions.lua、player_common.lua、SGwilson.lua、componentutil.lua、tuning.lua（rocks.lua／mound.lua 本轮 diff 为空；更新报告 R-01/P-83/SB-17/R-05）——CHOP 接入、SG 门禁与 spooked 状态、幸运公式、惊吓倍率常量均未变；锚点重定位 11 处，其中修正既有锚点 1 处（tuning.lua 高／低倍率原指段注释行，改指 MINE_SPOOKED_MULT_HIGH）；未覆盖：运行验证。
- 版本增量复核：2026-10-28，Steam buildid 26230496；spooked.lua 变化对照 build-26230496 报告 C-6 逐条核验：新增 `IsSpookImmune`（从 ShouldSpook 抽出，条件不变）与 `TryForcingCustomSpook`（绕过门槛强制惊吓）；Spook／TryCustomSpook／积累公式未变，锚点重定位并补两接口行；其余 sources 未在本轮变化（actions.lua／SGwilson.lua 相关锚点另行批量迁移）；未覆盖：运行验证。

- 源码审阅：2026-10-23，Steam buildid 25643519；完整审阅组件 120 行、年龄秒数来源及普通／自定义积累，核对 CHOP、岩石变档、挖坟消费者、玩家活动挂载、SG 与精神响应、两类 FX 和幸运抽取。正文结论覆盖上述路径；未核验其他角色 SG、所有工作 Prefab、引擎 Transform 返回值或外部模组。
- 结构检查：本页元数据、大小、链接与源码锚点检查通过；无新增完整 Lua 示例，导航登记由本批整合检查。
- 游戏运行：未运行；多人可见性、实际随机频率、事件延迟及保护装备动态切换未实测。

相关：[[reference/components/sanity]]、[[reference/components/workable]]。
