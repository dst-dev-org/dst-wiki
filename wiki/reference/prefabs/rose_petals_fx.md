---
title: "玫瑰花瓣飘落（rose_petals_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "藤蔓救起死亡玩家时种下玫瑰的花瓣飘落动画：联网 FX 在落点播 fall，播完不走立即移除而是 ErodeAway 一秒腐蚀淡出；生成方把死亡音效挂在本实体上播放。"
aliases: ["rose_petals_fx", "玫瑰花瓣", "花瓣飘落", "rose petals"]
tags: [prefab, fx]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/rose_petals_fx.lua", "stategraphs/SGwilson.lua", "simutil.lua", "prefabs/winona.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 玫瑰花瓣飘落（rose_petals_fx）

## 快速判断

- 职责与适用：**花瓣飘落动画实体**：联网 FX（AddNetwork），bank/build `rose_petals_fx`、播 `fall`（单参、未显式传循环参数）、FinalOffset -1；工厂自带 SoundEmitter 供生成方挂音效（源码注释明示"有些地方需要一个实体来播音"）。[工厂](../../../raw/current/scripts/prefabs/rose_petals_fx.lua#L6-L32)
- 优先使用：查"种玫瑰的花瓣/带音效宿主的一次性 FX"选本页；原版生成入口只有 SGwilson `death_vinesave_pst` 状态。
- 重要副作用：移除路径是 animover → **ErodeAway**（非 Remove）：默认 1 秒逐帧 SetErosionParams 腐蚀淡出后才 Remove，期间关闭 DynamicShadow。[移除](../../../raw/current/scripts/prefabs/rose_petals_fx.lua#L27)
- 修改边界：无 NOCLICK 标签但也无交互组件；persists=false 不存档；实体淡出期间仍占位约 1 秒。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `rose_petals_fx` | fn：CreateEntity → Transform＋AnimState＋SoundEmitter＋Network → FX 标签 → bank/build/anim → SetFinalOffset(-1) → SetPristine → 主模拟侧 animover ErodeAway＋persists=false | 单一注册；工厂无主模拟专属组件，只有移除路径差异。[工厂](../../../raw/current/scripts/prefabs/rose_petals_fx.lua#L6-L32)、[注册](../../../raw/current/scripts/prefabs/rose_petals_fx.lua#L34) |

注册名无 NAMES 条目；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：SGwilson `death_vinesave_pst` | `charlie_vinesave` 死亡救起流程的收尾状态：onenter 在玩家位置种 flower_rose（planted、NOCLICK、persists=false）并生成本 fx，把 `dontstarve/common/deathpoof` 播在本 fx 的 SoundEmitter 上 | 主模拟生成、联网复制；流程进入条件（藤蔓救起）见 SGwilson 死亡分支 | [状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L4065-L4113)、[救起分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3875-L3881) |
| API：ErodeAway | animover 触发：默认 1 秒、逐帧 `SetErosionParams(erode_amount, 0.1, 1.0)`，结束后 Remove；若实体带 floater 组件会先回调其 Erode | simutil 全局函数；第二参可自定义时长 | [实现](../../../raw/current/scripts/simutil.lua#L575-L597) |
| 关联声明：winona | Winona 的 prefabs 依赖表声明本 fx 与 flower_rose（其 roseinspectableuser 玫瑰机制相关），但生成点仍只在 SGwilson | 声明≠生成 | [依赖声明](../../../raw/current/scripts/prefabs/winona.lua#L17-L22) |
| 资源 | anim/rose_petals_fx.zip（bank/build 同名） | 声明与使用已核验，包内容未解包 | [资产声明](../../../raw/current/scripts/prefabs/rose_petals_fx.lua#L1-L4) |

## 典型接入

### 复用"播完腐蚀淡出"的一次性落地 FX

- 前提与选择：适合需要柔和消失（而非突然消失）的落地表现；实体带 SoundEmitter，可直接当音效宿主。
- 生效与撤销：SpawnPrefab＋定位即可；播完自动 ErodeAway（约 1 秒淡出）后移除，无需手工清理。想立即消失就自己 `fx:Remove()`，想更快淡出就自行调 `ErodeAway(fx, 0.3)`。
- 原版依据：SGwilson 把死亡音效挂在本实体上播（[生成段](../../../raw/current/scripts/stategraphs/SGwilson.lua#L4105-L4109)），验证了"借实体播音"的用法。

```lua
local fx = SpawnPrefab("rose_petals_fx")
fx.Transform:SetPosition(x, 0, z)
fx.SoundEmitter:PlaySound("dontstarve/common/deathpoof") -- 工厂自带 SoundEmitter
-- 播完 animover 自动 ErodeAway（默认 1 秒腐蚀淡出后移除）
```

- 调整与验证：淡出时长只有"默认 1 秒 / 自调"两档，工厂路径本身不传时长（animover 回调无参调用 ErodeAway）。

## 常见陷阱与限制

- **误区：animover 一定立即移除**：本 fx 挂的是 ErodeAway，有约 1 秒腐蚀淡出过程（[L27](../../../raw/current/scripts/prefabs/rose_petals_fx.lua#L27)＋[实现](../../../raw/current/scripts/simutil.lua#L575-L597)）；依赖"播完即消失"时序的逻辑（如立刻同一位置生成新物）会与残影重叠。
- **边界：声音挂在短命实体上**：脚本没有显式 KillSound；实体移除后一次性声音是否继续由引擎处理，本页未运行核验，不承诺长音效完整播放。
- **边界：ErodeAway 是全局函数不是方法**：调用形式 `ErodeAway(inst, 时长)`；它还会对带 floater 的实体触发 floater:Erode，复用时注意副作用（[实现](../../../raw/current/scripts/simutil.lua#L582-L584)）。
- **边界：原版仅一处生成**：Winona 依赖表里的声明不构成第二生成点；查找生成逻辑时以 SGwilson 状态为准。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；35 行工厂全文、SGwilson death_vinesave_pst 与藤蔓救起分支的 fx 相关段、ErodeAway 实现、winona 依赖声明、NAMES 检索（无命中）已核验；charlie_vinesave 完整触发链、动画包内容、淡出的实际渲染表现未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
