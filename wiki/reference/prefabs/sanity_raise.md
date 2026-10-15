---
title: "影岩升降标记动画（sanity_raise）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "通用表两注册的黑石升降标记：影柱计时分支、梦魇岩三状态、破损远古祭坛交互与场景脚本在实体位置落点的半透明波纹表现；名称不代表理智增减。"
aliases: ["sanity_raise", "sanity_lower", "理智升降动画", "影柱升起特效", "梦魇岩升降", "升降标记", "黑石波纹"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "prefabs/shadow_pillar.lua", "stategraphs/SGnightmarerock.lua", "prefabs/altar_prototyper.lua", "scenarios/chest_insanity.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 影岩升降标记动画（sanity_raise）

## 快速判断

- 职责与适用：黑石类实体升降／交互时的伴随波纹表现，两注册（`sanity_raise`／`sanity_lower`）；纯表现，**名称只是资源动画名，不代表理智增减方向**——理智结算由各消费者自身逻辑完成。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L42-L54)
- 优先使用：查"影柱升起／梦魇岩动画／黑石波纹"选本页；理智数值变化选 [[reference/components/sanity|sanity 组件]]。
- 重要副作用：无音效；`tintalpha 0.5` 半透明渲染。[条目](../../../raw/current/scripts/fx.lua#L42-L54)
- 修改边界：全部生成点均为一次性落点（`SetPosition`，不父挂）；动画是否重复未核验。[落点](../../../raw/current/scripts/stategraphs/SGnightmarerock.lua#L22-L22)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `sanity_raise` | 通用表条目 → 加载器代理＋一帧后本地实体，无 fn | bank／build=`blocker_sanity_fx`，anim=`raise`，tintalpha 0.5。[条目](../../../raw/current/scripts/fx.lua#L42-L47) |
| `sanity_lower` | 同上协议 | 同 bank，anim 换 `lower`；音效与 fn 均无。[条目](../../../raw/current/scripts/fx.lua#L49-L54) |

两注册在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，差异仅 anim 名 | 同族同协议 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L42-L54) |
| 生成者：影柱计时 | `shadow_pillar`（麦斯威尔重做影柱）"delay" 计时到期**立即**落 `sanity_raise` 与 pre 声，8 帧后才执行 PreRaise；"lifetime" 计时到期若仍处延迟升起阶段（`_delayraisetask` 未清）直接 Remove **不生成 lower**，否则落 `sanity_lower` 并 `DoLower` | 服务端；两分支互斥，伴随各自的音效 | [升起](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L130-L135)、[降下](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L153-L159) |
| 生成者：梦魇岩状态图 | `SGnightmarerock` 的 `raise` 态落 sanity_raise；`lower` 态与 `conceal`（从 raise 转入）态落 sanity_lower | 服务端；三处均在实体位置落点 | [raise](../../../raw/current/scripts/stategraphs/SGnightmarerock.lua#L18-L23)、[lower](../../../raw/current/scripts/stategraphs/SGnightmarerock.lua#L43-L48)、[conceal](../../../raw/current/scripts/stategraphs/SGnightmarerock.lua#L64-L70) |
| 生成者：破损祭坛交互 | `altar_prototyper` 损坏态交互完成（broken_doonact）播放 ding 声后落 `sanity_lower` | 服务端；伴随 `researchmachine_lvl3_ding` 音效 | [交互](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L381-L391) |
| 生成者：场景脚本 | `scenarios/chest_insanity.lua` 场景脚本（提供场景回调，OnLoad 接入宝箱陷阱逻辑；实际装配入口未核验）生成岩石并按玩家理智播升降动画，同时固定落 `sanity_raise` | 服务端脚本回调；装配路径未核验 | [场景](../../../raw/current/scripts/scenarios/chest_insanity.lua#L30-L40) |
| 资源 | bank／build=`blocker_sanity_fx`（理智阻挡条的动画资源） | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L42-L54) |

## 典型接入

### 借用升降标记

- 前提与选择：给自定义黑石类实体配升降表现时，按实体运动方向选 raise／lower 并在实体位置落点；不要按名称把它接到"理智数值变化"上。
- 生效与撤销：落点一次性动画，animover 自删；不跟随宿主。
- 原版依据：[梦魇岩状态落点](../../../raw/current/scripts/stategraphs/SGnightmarerock.lua#L18-L23)。

```lua
local fx = SpawnPrefab(direction == "up" and "sanity_raise" or "sanity_lower")
fx.Transform:SetPosition(ent.Transform:GetWorldPosition())
```

- 调整与验证：半透明程度由数据 `tintalpha 0.5` 固定，运行时改需自建条目；两注册无 fn，显示层级与普通条目一致。

## 常见陷阱与限制

- **误区：sanity_raise＝理智上升**：名称来自 `blocker_sanity_fx` 资源的 raise／lower 动画；影柱升起、梦魇岩出现等场景与玩家理智变化无必然关系，理智结算在消费者逻辑（sanity 组件调用）里。[条目](../../../raw/current/scripts/fx.lua#L42-L54)
- **边界：全部为落点式**：四个消费者的调用都是 `Transform:SetPosition` 一次性定位，无父挂用例；实体移动后表现不跟随。[落点](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L135-L135)
- **边界：chest_insanity 装配入口未核验**：该文件是场景脚本（提供场景回调、OnLoad 接入宝箱陷阱逻辑），实际何时被装配未核验；已核验的运行时生成路径以影柱／梦魇岩／祭坛三处为准。[场景](../../../raw/current/scripts/scenarios/chest_insanity.lua#L39-L39)
- **边界：半透明固定值**：`tintalpha 0.5` 由数据表决定，两名一致；需要不透明变体得新增条目。[染色](../../../raw/current/scripts/prefabs/fx.lua#L71-L72)
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。两条目配置、加载器协议、影柱两计时分支（含 delay 立即落 raise 与 lifetime 延迟升起期直接 Remove 的例外）、梦魇岩三状态、破损祭坛交互、场景脚本回调逐条核验（chest_insanity 装配入口未核验）；.po 名称键定向核验无结果。未覆盖：`blocker_sanity_fx` 动画资源内容、影柱／梦魇岩的完整理智结算（各自页面范围）、晚加入时序。
- 游戏运行：未运行；波纹视觉与半透明表现未实测。

返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
