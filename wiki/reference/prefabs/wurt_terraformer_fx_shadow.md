---
title: "沃特沼泽改造、水盾与暗影鱼人特效（wurt_terraformer_fx_shadow 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "沼泽瓦片改造与恢复、暗影／月亮地面粒光、沃特水盾及暗影鱼人离场的八种通用表表现。"
aliases: ["wurt_swamp_terraform_fx", "wurt_terraformer_fx_shadow", "wurt_terraformer_fx_lunar", "wurt_water_splash_1", "wurt_water_splash_2", "wurt_water_splash_3", "shadow_merm_spawn_poof_fx", "shadow_merm_smacked_poof_fx", "沃特沼泽改造特效", "沃特水盾", "暗影鱼人烟"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/wurt_swamp_terraformer.lua", "prefabs/wurt.lua", "stategraphs/SGmerm.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 沃特沼泽改造、水盾与暗影鱼人特效（wurt_terraformer_fx_shadow 等）

## 快速判断

- 八名为通用表动画表现；瓦片更换、湿度抵伤与鱼人移动由生成者负责。通用代理与本地动画生命周期见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 效果中文为按动画与调用语境核验的描述，非官方译名；注册名的 NAMES 键未命中，角色 Wurt 官方简中名为沃特。[角色名称](../../../raw/current/scripts/languages/chinese_s.po#L352520-L352522)

## 注册与构造

| 注册名 | bank／build；anim | 字段与证据 |
|---|---|---|
| `wurt_swamp_terraform_fx` | `pond_splash_fx`／同左；`swamp_splash` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L3439-L3442) |
| `shadow_merm_spawn_poof_fx` | `merm_shadow_fx`／同左；`spawn_poof` | `meta4/shadow_merm/spawn_poof`；fn 层偏移 1、alpha 0.5。[条目](../../../raw/current/scripts/fx.lua#L3445-L3453) |
| `shadow_merm_smacked_poof_fx` | 同上；`smacked_poof` | `meta4/shadow_merm/smacked_poof`；fn 层偏移 1、设第 14 帧、alpha 0.5。[条目](../../../raw/current/scripts/fx.lua#L3456-L3465) |
| `wurt_water_splash_1`／`2`／`3` | `splash_water_rot`／`wurt_splash_fx`；依次为 `watershield_small`／`medium`／`large` | 均带 `meta4/wurt/water_shield`，fn 为 `FinalOffset1`。[1](../../../raw/current/scripts/fx.lua#L3468-L3473)、[2](../../../raw/current/scripts/fx.lua#L3476-L3481)、[3](../../../raw/current/scripts/fx.lua#L3484-L3489) |
| `wurt_terraformer_fx_shadow` | `cane_shadow_fx`／同左；初始 `shad1` | 无 sound；`tintalpha=0.5`，fn 随机播放 `shad1`～`shad3`。[条目](../../../raw/current/scripts/fx.lua#L3492-L3499) |
| `wurt_terraformer_fx_lunar` | `moon_altar_link_fx`／同左；初始 `fx1` | 无 sound；`tintalpha=0.5`，fn 缩放 0.5 并以同一随机数选 `fx1`／`fx2`／`fx3`。[条目](../../../raw/current/scripts/fx.lua#L3502-L3517) |

## 关联机制导航

| 生成者 | 条件与证据 |
|---|---|
| 沼泽改造器 | `DoTerraform` 只对未被物体阻挡、陆地且非海洋／不可通行／临时瓦片排任务；回调改瓦片、保存下层瓦片并在中心生成 `wurt_swamp_terraform_fx`。撤销时仅处理仍等于改造目标的瓦片，恢复下层并同样生成 FX。[排程](../../../raw/current/scripts/prefabs/wurt_swamp_terraformer.lua#L107-L143)、[改造](../../../raw/current/scripts/prefabs/wurt_swamp_terraformer.lua#L90-L103)、[撤销](../../../raw/current/scripts/prefabs/wurt_swamp_terraformer.lua#L157-L179) |
| 改造器周期光点 | `SetType` 默认 SHADOW；周期按类型表选 shadow／lunar，对图案每个瓦片随机判定、只在瓦片仍为目标类型时落随机点，y=0；计时器随后重启。两个 FX 落点式，不父挂。[类型](../../../raw/current/scripts/prefabs/wurt_swamp_terraformer.lua#L7-L20)、[周期](../../../raw/current/scripts/prefabs/wurt_swamp_terraformer.lua#L53-L85) |
| 沃特湿度抵伤 | `ignore_absorb` 为假、负伤害、非持续伤害且有来源，并且湿度大于 0、技能档大于 0 时，按湿度及技能档计算吸收并消耗湿度；`ceil` 按吸收占上限比例选 1～3 名并将生成代理 `AddChild`。技能失效时取消 health 的该 modifier。[计算及生成](../../../raw/current/scripts/prefabs/wurt.lua#L481-L515)、[安装](../../../raw/current/scripts/prefabs/wurt.lua#L559-L563) |
| 暗影鱼人 | `shadow_spawn` 入状态在自身位置生 spawn，14 帧生 smacked；`shadow_loyaltyover` 14 帧也生 smacked，15 帧移除鱼人。两效果落点式。[状态](../../../raw/current/scripts/stategraphs/SGmerm.lua#L648-L683)、[离场](../../../raw/current/scripts/stategraphs/SGmerm.lua#L692-L716) |

## 常见陷阱与限制

- 水盾只是动画与声音，不能仅生成它来获得抵伤；湿度消耗和伤害变更由 `RedirectDamageToMoisture` 结算。
- shadow／lunar 光点依据改造器类型与当前瓦片选名；动画随机选择不决定瓦片转换。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。逐项核对表项字段；在 `raw/current/scripts` 全树检索完整注册名及 `wurt_water_splash_`、`wurt_terraformer_fx`、`shadow_merm` 前缀，追踪动态选名和依赖声明；未核验动画资源及游戏内表现。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-table-character-index|通用表 FX·角色与装备补充]]。
