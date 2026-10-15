---
title: "暗影拍指（charlie_snap）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "玫瑰眼镜燃料呈现链的开场暗影拍指：黑剪影动画，音效由数据 fn 在本地实体延迟 21 帧自挂播放；升级或洞穴走实心变体，纯表现。"
aliases: ["charlie_snap", "charlie_snap_solid", "暗影拍指", "查理拍指", "夜手拍指", "拍指特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "componentutil.lua", "prefabs/charlieresidue.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗影拍指（charlie_snap）

## 快速判断

- 职责与适用：温娜玫瑰眼镜的燃料呈现链开场表现（暗影之手"拍指"剪影），两注册；音效在数据 fn 内延迟 21 帧由本地实体自挂播放。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L229-L250)
- 优先使用：查"暗影拍指／查理拍指"选本页；呈现链全貌与 shadow_puff 步骤见 [[reference/prefabs/sand_puff|sand_puff 族]]。
- 重要副作用：无直接音效字段——fn 在**本地实体**上临时加 SoundEmitter 并延迟播 `meta4/shadow_snap/snap`。[fn](../../../raw/current/scripts/fx.lua#L235-L238)
- 修改边界：落点式（fuel 呈现时 y=2 高度）；动画是否重复未核验。[呈现链](../../../raw/current/scripts/componentutil.lua#L611-L615)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `charlie_snap` | 通用表条目 → 加载器代理＋一帧后本地实体，fn 内联 | bank／build=`charliesnap`，anim=`snap`，黑 tint×0.7；fn 自挂 SoundEmitter 延迟 21 帧播拍指声。[条目](../../../raw/current/scripts/fx.lua#L229-L239) |
| `charlie_snap_solid` | 同上协议 | 同资源同 fn，**无 tintalpha**（全不透明黑）。[条目](../../../raw/current/scripts/fx.lua#L241-L250) |

两注册在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目；差异仅 tintalpha | 同族同协议 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L229-L250) |
| 生成者：燃料呈现链 | 玫瑰眼镜残余物激活产出燃料后，`OnFuelPresentation1` 在对象周围 (x,2,z) 落点生成——**升级（温娜 winona_charlie_2）或洞穴**选 `_solid`，否则普通版；25 帧后接 shadow_puff 步 | 服务端；呈现链完整语义见 sand_puff 族页 | [分派](../../../raw/current/scripts/componentutil.lua#L611-L615)、[[reference/prefabs/sand_puff|sand_puff 族]] |
| 依赖声明 | charlieresidue（查理残余物）声明两注册 | 仅声明；快照内该文件无对应生成调用 | [声明](../../../raw/current/scripts/prefabs/charlieresidue.lua#L7-L8) |
| 资源 | bank／build=`charliesnap` | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L229-L250) |

## 典型接入

### 复用拍指表现

- 前提与选择：拍指声在 fx 数据 fn 内延迟 21 帧自动播放（本地实体自挂 SoundEmitter），直接 Spawn 即有声；洞穴或升级语义由生成方选 `_solid`。
- 生效与撤销：落点一次性动画，animover 自删；临时 SoundEmitter 随实体回收。
- 原版依据：[呈现链](../../../raw/current/scripts/componentutil.lua#L611-L615)。

```lua
local solid = upgraded or TheWorld:HasTag("cave")
local fx = SpawnPrefab(solid and "charlie_snap_solid" or "charlie_snap")
fx.Transform:SetPosition(x, 2, z)   -- 呈现高度 y=2
```

- 调整与验证：透明度差异在数据 tintalpha（0.7 对 1.0）；想改拍指延迟需改数据 fn（21 帧），运行时不可调。

## 常见陷阱与限制

- **误区：拍指声是通用工厂音效字段**：数据条目无 `sound` 键，声音由 fn 延迟 21 帧在本地实体上播放——通用协议的 sounddelay 机制与此不同。[fn](../../../raw/current/scripts/fx.lua#L235-L238)
- **边界：solid 选择含洞穴条件**：升级**或**洞穴世界都走实心变体；两条件是"或"不是叠加，别把洞穴条件套给 shadow_puff 步（那边仅升级，见 sand_puff 族页）。[分派](../../../raw/current/scripts/componentutil.lua#L611-L615)
- **边界：charlieresidue 仅有声明**：依赖声明不等于生成；快照内该文件未生成两注册。[声明](../../../raw/current/scripts/prefabs/charlieresidue.lua#L7-L8)
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。两注册配置（含 fn 延迟音效）、加载器协议、燃料呈现链分派与升级／洞穴条件、charlieresidue 声明辨析核验；.po 名称键定向核验无结果。未覆盖：`charliesnap` 动画资源内容、呈现链之外的潜在调用、晚加入时序。
- 游戏运行：未运行；拍指视觉与延迟音效未实测。

返回 [[reference/prefabs/fx-transition-index|变身、召唤、传送与退场]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
