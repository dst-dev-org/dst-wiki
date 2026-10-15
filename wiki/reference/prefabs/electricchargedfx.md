---
title: "电充闪光特效（electricchargedfx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "伏特羊肉冻的电攻击 buff 附加／续期时打出的充电标记：网络代理＋本地橙色光效与放电动画＋主模拟侧目标闪白；无玩法组件，带电攻击来自 buff 的 electricattacks 组件。"
aliases: ["electricchargedfx", "electric_charged_fx", "电充特效", "充电特效", "伏特羊肉冻", "electric attack buff", "buff_electricattack"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/electric_charged_fx.lua", "prefabs/foodbuffs.lua", "preparedfoods_warly.lua", "components/colouradder.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 电充闪光特效（electricchargedfx）

## 快速判断

- 职责与适用：吃[伏特羊肉冻](../../../raw/current/scripts/preparedfoods_warly.lua#L30-L48)（官方译名 [NAMES](../../../raw/current/scripts/languages/chinese_s.po#L350740-L350742)）获得的 `buff_electricattack` 附加与续期时打出的"充电"标记特效：代理＋各端本地放电动画与橙色光斑，主模拟侧另驱动目标闪白；攻击带电来自 buff 的 [[reference/components/electricattacks|electricattacks 组件]]（[注册](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L149)、[生成](../../../raw/current/scripts/prefabs/foodbuffs.lua#L89)）。
- 优先使用：复用"目标被充电"表现时 `SpawnPrefab("electricchargedfx"):SetTarget(target)`（仅主模拟暴露 [SetTarget](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L144)）；攻击带电则挂 electricattacks 源而非本特效。
- 重要副作用：主模拟向目标推白闪（colouradder 栈），退出时还原；本地视觉一次性播 `discharged`，Light 强度 0.9 约 0.45 秒衰减到 0。[光衰减](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L6-L22)、[闪白](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L83-L107)
- 修改边界：代理 1 秒后必然自毁（[DoTaskInTime](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L142)），持久表现需自行续期；本页未覆盖攻击火花工厂（见 [[reference/prefabs/sparks|sparks 族]]）与 electricattacks 细节。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `electricchargedfx` | fn：Transform＋Network、FX 标签 → `DoTaskInTime(0, StartFX)`（SetPristine 前，双端都会执行）→ SetPristine → 非主模拟返回 → 主模拟 `persists=false`、1 秒自毁、暴露 SetTarget；StartFX 另建**非联网**视觉实体：bank/build `elec_charged_fx`、播 `discharged`（未显式传循环参数，实际是否重复未核验）、bloom、Light(半径 2/强度 0.9/橙色 235,121,12)、非专服才加 SoundEmitter，以 1/20 秒周期任务衰减光强 | 单一注册，代理＋本地视觉两实体；视觉挂在代理的父实体（即目标）上。[工厂](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L124-L147)、[StartFX](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L33-L71)、[注册](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L149) |

注册名无 NAMES 条目；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：buff_electricattack | [electric_attach](../../../raw/current/scripts/prefabs/foodbuffs.lua#L61-L90) 给目标加 electricattacks 源、监听 `onattackother` 补打电击火花（electrichitsparks，见 [[reference/prefabs/sparks|sparks 族]]；武器自带电 stimuli 或远程弹道时跳过），随后生成本特效；续期走 [electric_extend](../../../raw/current/scripts/prefabs/foodbuffs.lua#L92-L94) 再打一次 | 仅主模拟（buff 实体主模拟专用）；buff 来自伏特羊肉冻 `AddDebuff`，时长 `BUFF_ELECTRICATTACK_DURATION = day_time`（白昼时段长度；不是完整昼夜的 total_day_time） | [食物入口](../../../raw/current/scripts/preparedfoods_warly.lua#L43-L46)、[buff 注册](../../../raw/current/scripts/prefabs/foodbuffs.lua#L223)、[时长](../../../raw/current/scripts/tuning.lua#L4627)、[[reference/components/debuff]] |
| 组件：colouradder | 闪白经目标 colouradder 栈 Push/Pop；目标无该组件时退回裸 `AnimState:SetAddColour`，退出时优先 freezable:UpdateTint 还原 | 主模拟组件；colouradder 会经 colouraddersync 或 AnimState:SetAddColour 更新；仅凭本文件无法判定客户端实际白闪表现 | [闪白](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L91-L105)、[还原](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L73-L81)、[[reference/components/colouradder]]、[[reference/components/colouraddersync]] |
| 生命周期 | 代理 1 秒自毁；视觉实体在动画播完且光衰减结束后 Remove（animover 时尚未衰减完则置 `killfx` 等衰减到 0） | 双端各自清理；`sound` 标志保证专服（无 SoundEmitter）不播放电音 | [OnAnimOver](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L24-L31)、[onupdate](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L6-L22) |
| 资源 | anim/elec_charged_fx.zip（bank/build 同名）；音效 `dontstarve/characters/wx78/spark` 播一次 | 资源存在性按声明核验 | [assets](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L1-L4)、[音效](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L7-L10) |

## 典型接入

### 复用"目标被充电"标记

- 前提与选择：主模拟生成后调用 `SetTarget(target)`；target 需为有效实体。SetTarget 会把代理父子化到目标并启动主模拟闪白（[SetTarget](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L109-L122)）。
- 生效与撤销：代理 1 秒后自动移除，闪白约 1 秒内自行 Pop 还原（flash 从 1 每帧减 0.07，低于 0.1 收尾）；无需手动清理，勿对同一目标反复叠加。
- 原版依据：buff 的 attach 与 extend 各调一次即"充电刷新"的官方节奏。

```lua
local fx = SpawnPrefab("electricchargedfx")
fx:SetTarget(victim)  -- 仅主模拟；客户端本地视觉自动生成
```

- 调整与验证：闪白步进与出色条件写死在 [OnUpdateFlash](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L87-L95)；改观感建议自建工厂复用本页结构。

## 常见陷阱与限制

- **误区：名字带 fx 就是纯客户端贴花**：主模拟侧真实修改目标颜色栈并依赖 freezable 还原；目标失效时 OnUpdateFlash 自行拆除回调（[防失效](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L84-L86)）。
- **边界：专服无音效**：SoundEmitter 仅非专服添加，`sound` 标志随之关闭（[条件](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L43-L45)、[L67](../../../raw/current/scripts/prefabs/electric_charged_fx.lua#L67)）。
- **边界：1 秒硬寿命**：代理 `DoTaskInTime(1, inst.Remove)` 与目标死亡、buff 提前结束无关；持续充电表现需续期生成。
- **边界：闪白客户端可见性**：主模拟更新颜色栈或 AnimState；目标具体同步配置与客户端可见效果需另行验证，不能从组件仅在主模拟执行推断客户端一定看不到白闪。

## 验证记录

- 源码审阅：2026-10-30，build 26230496；工厂全文（代理／视觉两实体、光衰减、闪白、还原）、foodbuffs 的 electricattack attach/extend/注册、伏特羊肉冻食谱、colouradder 全文、tuning 时长与译名已核验；`discharged` 动画帧数与是否重复、晚加入残影、electrichitsparks 工厂未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、链接 --strict、label-check 通过；本批依据见[[updates/build-26335394/index#wiki-沿用记录]]（批次-2）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
