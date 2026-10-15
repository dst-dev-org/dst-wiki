---
title: "麦斯威尔烟（maxwell_smoke）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "麦斯威尔 talker 三分支的烟：台词收尾两种模式都立即生成、普通出现分支 Sleep 后生成、取消分支用延迟任务；音效由 talker 分支播放，fx 纯表现。"
aliases: ["maxwell_smoke", "麦斯威尔烟", "消散烟", "maxwell 特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "components/maxwelltalker.lua", "prefabs/chessjunk.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 麦斯威尔烟（maxwell_smoke）

## 快速判断

- 职责与适用：麦斯威尔 talker（maxwelltalker 组件，幻觉麦斯威尔／提线木偶台词系统）台词收尾、出现与取消三种分支的烟雾表现，单注册；纯表现，音效由 talker 分支另行播放。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L328-L332)
- 优先使用：查"麦斯威尔烟／消散烟"选本页；talker 台词系统读 [[reference/components/talker|talker]]（maxwelltalker 是其专用变体）。
- 重要副作用：无音效字段——消散声（`maxwell/disappear(_adventure)`）由 talker 分支播放。[条目](../../../raw/current/scripts/fx.lua#L328-L332)
- 修改边界：talker 实体位置一次性落点（不父挂）；动画是否重复未核验。[落点](../../../raw/current/scripts/components/maxwelltalker.lua#L46-L46)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `maxwell_smoke` | 通用表条目 → 加载器代理＋一帧后本地实体，无 fn | bank／build=`max_fx`，anim=`anim`；无音效、无 tint、无 transform。[条目](../../../raw/current/scripts/fx.lua#L328-L332) |

注册名在简中 .po 无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，无附加字段 | 单注册 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L328-L332) |
| 生成者：台词结束（两模式立即） | `DoTalk` 台词线程收尾：冒险模式播 disappear_adventure 声后**立即**落点；普通模式关影子、播 disappear 声后**立即**落点——两分支都无延迟任务 | 组件方法；宿主与端未核验 | [收尾](../../../raw/current/scripts/components/maxwelltalker.lua#L182-L190) |
| 生成者：出现分支（普通模式延迟） | `DoTalk` 开场带 appearanim 时：冒险模式只出声 Sleep(1.4) 不生成；**普通模式 Sleep(0.4)→disappear 声→落点** | 组件方法；同上未核验宿主 | [开场](../../../raw/current/scripts/components/maxwelltalker.lua#L133-L142) |
| 生成者：取消分支（延迟任务） | `OnCancel`：冒险模式立即落点；普通模式 `DoTaskInTime(0.4)` 延迟任务内落点——**只有取消分支使用延迟任务** | 组件方法 | [取消](../../../raw/current/scripts/components/maxwelltalker.lua#L39-L54) |
| 依赖声明 | chessjunk（发条残骸）声明本名 | 仅声明；快照内该文件无生成调用 | [声明](../../../raw/current/scripts/prefabs/chessjunk.lua#L23-L23) |
| 资源 | bank／build=`max_fx`（麦斯威尔专属资源） | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L328-L332) |

## 典型接入

### 复用消散烟

- 前提与选择：给自定义 talker 实体配表现时，先分清分支语义——台词收尾与冒险取消立即生成、普通出现与普通取消有 Sleep／延迟任务；fx 只管烟。
- 生效与撤销：落点一次性动画，animover 自删。
- 原版依据：[三分支结构](../../../raw/current/scripts/components/maxwelltalker.lua#L133-L142)。

```lua
local fx = SpawnPrefab("maxwell_smoke")
fx.Transform:SetPosition(inst.Transform:GetWorldPosition())
```

- 调整与验证：冒险模式判定走 `SaveGameIndex:GetCurrentMode`（单机冒险语义），该判定在联机的实际走向**未核验**，不要据此假设联机必走某分支；组件宿主（哪个实体挂 maxwelltalker）也未核验。

## 常见陷阱与限制

- **误区：烟自带消散声**：数据条目无 sound；各分支的消散／再现声（两种冒险／普通音效名）都在 talker 内播放。[条目](../../../raw/current/scripts/fx.lua#L328-L332)
- **边界：三分支时序不同**：台词收尾两模式都立即、普通出现分支 Sleep 后生成、取消分支用延迟任务——三者在源码中是不同代码块，不能合并成"消散即生成"。[收尾](../../../raw/current/scripts/components/maxwelltalker.lua#L182-L190)
- **边界：调用点集中在专用组件**：快照内生成点全部在 maxwelltalker（麦斯威尔专属 talker）；通用 talker 组件不生成此烟。[调用面](../../../raw/current/scripts/components/maxwelltalker.lua#L46-L46)
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。单注册配置、加载器协议、maxwelltalker 三分支结构（台词收尾两模式立即／普通出现 Sleep 后／取消延迟任务）、chessjunk 声明辨析核验；.po 名称键定向核验无结果。未覆盖：`max_fx` 动画资源内容、组件宿主实体与端、冒险模式判定在联机的实际走向、晚加入时序。
- 游戏运行：未运行；烟雾视觉未实测。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
