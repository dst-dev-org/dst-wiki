---
title: "施法与怀表回溯表现（staffcastfx）"
created: "2026-11-02"
updated: "2026-11-04"
type: prefab
summary: "法杖、抛币和怀表施法及回溯的八个联网动画注册，由玩家状态机选择、染色并管理。"
aliases: ["staffcastfx", "staffcastfx_mount", "cointosscastfx", "cointosscastfx_mount", "pocketwatch_cast_fx", "pocketwatch_cast_fx_mount", "pocketwatch_warpback_fx", "pocketwatch_warpbackout_fx", "施法特效", "怀表回溯特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/staffcastfx.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "fx.lua", "prefabs/player_common.lua", "prefabs/pig_coin.lua", "prefabs/pocketwatch.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 施法与怀表回溯表现（staffcastfx）

## 快速判断

- 本页覆盖独立工厂 `staffcastfx.lua` 的八个注册；它们负责联网动画和颜色，施法结果、传送、无敌及声音由调用方等其他逻辑处理。[工厂](../../../raw/current/scripts/prefabs/staffcastfx.lua#L23-L72)、[状态机](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20594-L20625)
- 标题和中文别名是依据用途核验的效果描述，非官方译名；简中 `languages/chinese_s.po` 中定向检索八个注册名的 `STRINGS.NAMES` 键均未命中。
- 不要把下列 `pocketwatch_cast_fx` 注册与通用表其他怀表动画混为一谈：`fx.lua` 的 `pocketwatch_heal_fx`、`pocketwatch_ground_fx` 把 `pocketwatch_cast_fx` 当 **bank** 名借用，与本页同名字注册不是同一对象；怀表攻击与治疗表现另见 [[reference/prefabs/wanda_attack_pocketwatch_old_fx|怀表攻击与治疗特效]]。[同名 bank](../../../raw/current/scripts/fx.lua#L2570-L2596)

## 注册与构造

工厂创建 Transform、AnimState、Network 并加 `FX` 标签；骑乘版六方向，其余四方向，动画层偏移 1。资源、动画与注册对应如下；资源包内容和帧数未解包核对。[资源](../../../raw/current/scripts/prefabs/staffcastfx.lua#L1-L21)、[工厂和注册](../../../raw/current/scripts/prefabs/staffcastfx.lua#L27-L72)

| 注册名 | bank | build | 动画 | 声明动画资源 |
|---|---|---|---|---|
| `staffcastfx` | `staff_fx` | `staff` | `staff` | `anim/staff.zip` |
| `staffcastfx_mount` | `staff_fx` | `staff` | `staff_mount` | `anim/staff.zip` |
| `cointosscastfx` | `cointosscast_fx` | `cointosscast_fx` | `cointoss` | `anim/cointosscast_fx.zip`、`anim/mount_cointosscast_fx.zip` |
| `cointosscastfx_mount` | `mount_cointosscast_fx` | `cointosscast_fx` | `cointoss` | 同抛币资源 |
| `pocketwatch_cast_fx` | `pocketwatch_cast_fx` | `pocketwatch_casting_fx` | `pocketwatch_cast` | `anim/pocketwatch_casting_fx.zip`、`anim/pocketwatch_casting_fx_mount.zip` |
| `pocketwatch_cast_fx_mount` | `pocketwatch_casting_fx_mount` | `pocketwatch_casting_fx_mount` | `pocketwatch_cast` | 同怀表施法资源 |
| `pocketwatch_warpback_fx` | `pocketwatch_warp_casting_fx` | `pocketwatch_warp_casting_fx` | `warpfx` | `anim/pocketwatch_warp_casting_fx.zip` |
| `pocketwatch_warpbackout_fx` | `pocketwatch_warp_casting_fx` | `pocketwatch_warp_casting_fx` | `warpfx_pst` | `anim/pocketwatch_warp_casting_fx.zip` |

`SetPristine` 后非主模拟端返回；主模拟端设置 `persists = false`，将 `SetUp(colour)` 暴露给调用方，并在 `animover` 移除实体。`SetUp` 接收前三项 RGB 并设 alpha 为 1；它不是客户端可直接调用的工厂接口。八个注册的 `SpawnPrefab` 都只在服务端玩家状态机 `SGwilson` 中出现，客户端 `SGwilson_client` 只以 `server_states` 镜像同名施法状态并做预测，不生成这些实体。[颜色与端分界](../../../raw/current/scripts/prefabs/staffcastfx.lua#L23-L62)、[客户端镜像](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L6484-L6489)

## 关联机制导航

| 状态／生成 | 条件、落点与清理 | 证据 |
|---|---|---|
| `castspell` | 进入时取手持物 `fxcolour`，缺省白色；若物品有 `DoGemShine` 则走该方法，否则按 rider 骑乘状态选择法杖两版、父挂玩家并 `SetUp`。第 53 帧清除保存的引用并执行动作；此之前退出时若特效仍有效则移除。另有独立施法光源。 | [进入](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16170-L16196)、[执行和退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16240-L16277) |
| `cointosscastspell` | 从缓冲动作物品取颜色、声音；第 7 帧按 rider 选择抛币版、父挂并染色，第 15 帧生成独立光源。第 53 帧放弃特效引用并执行动作；此前退出会移除仍有效的特效。 | [时间线](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16363-L16401)、[退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16419-L16428) |
| `pocketwatch_cast` | 缓冲动作记录怀表、目标和 `castfxcolour`；第 8 帧怀表仍有效且 `CanCast` 才按骑乘状态生成、父挂并染色，第 16 帧若特效引用存在则生成光源。动作失败分支移除特效和光源；退出也移除仍有效的引用。 | [施法](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20458-L20520)、[退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20539-L20546) |
| `pocketwatch_warpback` | 回溯动作成功进入此状态后，生成 `pocketwatch_warpback_fx`、父挂玩家并染色。玩家动画完成时将其解父并放到玩家当时世界位置，放弃引用后继续迁移或回溯；提前退出则移除仍持有的有效特效。 | [前置状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20566-L20584)、[进入](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20594-L20612)、[解父和迁移](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20632-L20673) |
| `pocketwatch_warpback_pst` | 玩家传送至目标位置并发 `onwarpback` 后，在目标坐标生成 `pocketwatch_warpbackout_fx` 并染色；此状态没有保存该特效用于退出清理，它由工厂动画完成事件移除。 | [落点](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20682-L20706)、[工厂](../../../raw/current/scripts/prefabs/staffcastfx.lua#L54-L62) |

`player_common`、`pig_coin` 和 `pocketwatch` 中同名列表是依赖声明，不能当作生成点。[玩家依赖](../../../raw/current/scripts/prefabs/player_common.lua#L2208-L2209)、[抛币依赖](../../../raw/current/scripts/prefabs/pig_coin.lua#L8-L9)、[怀表依赖](../../../raw/current/scripts/prefabs/pocketwatch.lua#L15-L22)

## 常见陷阱与限制

- 骑乘版本由调用方选择，工厂不会根据父实体自动切换；`castspell` 直接使用 rider，抛币和怀表选择则先检查该组件是否存在。[选名](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16183-L16191)、[抛币选名](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16380-L16385)、[怀表选名](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20479-L20484)
- 工厂声明不自带声音或玩法组件。法杖／抛币状态中的声音和独立光源，以及怀表回溯状态的传送和无敌均不可归因于特效实体本身。[抛币声音](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16387-L16400)、[回溯效果](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20614-L20625)、[目标传送](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20699-L20706)
- 动画完成、状态中断和解父是不同的清理路径；放弃状态引用不等于立即移除已生成实体。未运行游戏验证视觉、网络时序及中断表现。

## 验证记录

- 2026-11-02 源码审阅，Steam buildid 26335394；逐项核对工厂字段与注册，全文检索 `raw/current/scripts` 中八个注册名及其前缀、状态机条件选名，区分真实生成与依赖声明；不声称覆盖外部模组的动态生成。
- 2026-11-04 批 C 独立复核：重读 `staffcastfx.lua` 与 `SGwilson` 四个施法／回溯状态，补核客户端 `SGwilson_client` 的同名镜像状态（不生成 FX）及 `fx.lua` 中借用同名 bank 的怀表注册；其余锚点逐条对读。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-character-index|角色技能、施法与读书]]。
