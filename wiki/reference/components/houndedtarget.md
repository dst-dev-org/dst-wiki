---
title: "houndedtarget"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "调节实体被猎犬选中的目标权重并声明偷猎犬者身份，hounded 按权重分配猎犬、把偷窃者的份额转移给小偷。"
aliases: ["houndedtarget", "猎犬目标权重", "偷猎犬", "猎犬小偷"]
tags: [component, ai, creature]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/houndedtarget.lua", "components/hounded.lua", "components/luckuser.lua", "prefabs/wes.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# houndedtarget

## 快速判断

- 职责与适用：玩家侧的两张修正表——`target_weight_mult`（目标权重乘数）与 `hound_thief_sources`（布尔来源表）；hounded 在生成猎犬时读取。[构造](../../../raw/current/scripts/components/houndedtarget.lua#L1-L10)
- 优先使用：想改变某玩家被猎犬盯上的程度，用 `target_weight_mult:SetModifier(source, mult, name)`；想当"偷猎犬者"往 `hound_thief_sources` 写 true。负幸运的写入由 [[reference/components/luckuser]] 自动完成，无需重复实现。[写入参照](../../../raw/current/scripts/prefabs/wes.lua#L38-L42)
- 重要副作用：权重参与 `sqrt(spawnsToRelease) * weight` 的分配计算（默认无组件时为 1）；偷窃者会从组内抢走猎犬份额，数量受 `max_thieved_spawn_per_thief` 上限约束。[权重消费](../../../raw/current/scripts/components/hounded.lua#L284) [份额转移](../../../raw/current/scripts/components/hounded.lua#L304-L325)
- 修改边界：无存读、无网络变量；组件只表达"目标侧倾向"，猎犬生成算法、波次与数量归 hounded。[读取方](../../../raw/current/scripts/components/hounded.lua#L284-L289)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `target_weight_mult` | `SourceModifierList`（乘数表） | `GetTargetWeight()` 返回总值；hounded 对无组件玩家按 1 处理。维斯写入 `TUNING.WES_HOUND_TARGET_MULT`（2.0）。[读取](../../../raw/current/scripts/components/houndedtarget.lua#L12-L14) [维斯](../../../raw/current/scripts/prefabs/wes.lua#L41) [常量](../../../raw/current/scripts/tuning.lua#L2721) |
| `hound_thief_sources` | `SourceModifierList(inst, false, SourceModifierList.boolean)` | 布尔来源表；`IsHoundThief()` 为 true 时该玩家加入 thieves 列表参与抢夺。[实现](../../../raw/current/scripts/components/houndedtarget.lua#L7) [消费](../../../raw/current/scripts/components/hounded.lua#L287-L289) |
| `hound_thief` | `boolean`（已弃用） | 构造写 false，仅作 `IsHoundThief()` 的回退 or 项；不要再写入，来源表才是现行路径。[弃用字段](../../../raw/current/scripts/components/houndedtarget.lua#L9) |

## 典型接入

### 给角色登记猎犬倾向与偷窃者身份（维斯参照）

- 前提与选择：服务端；Wes 在 master_postinit 里确保组件存在后用专属来源名写入两条修正，来源名可作撤销键。[维斯](../../../raw/current/scripts/prefabs/wes.lua#L32-L42)
- 生效与撤销：修正随 SourceModifierList 生命周期生效；撤销用同一 source 键 RemoveModifier 或让来源实体 onremove 自动清理（利用表构造时传入的 inst）。数值语义参考 luckuser 的负幸运映射 `1+abs(luck)`。[luckuser 写入](../../../raw/current/scripts/components/luckuser.lua#L27-L33)

```lua
inst:AddComponent("houndedtarget")
inst.components.houndedtarget.target_weight_mult:SetModifier(inst, 2.0, "my_mod_misfortune")
inst.components.houndedtarget.hound_thief_sources:SetModifier(inst, true, "my_mod_misfortune")
```

- 调整与验证：乘数为乘法权重（2.0 即维斯常量），不是概率；偷窃份额分配细节（thieved_spawns 与上限）见 hounded 生成流程。[份额计算](../../../raw/current/scripts/components/hounded.lua#L323-L326)

## 常见陷阱与限制

- **写弃用的 `hound_thief` 字段当开关：** 现行判定先读 `hound_thief_sources:Get()`，再 or 上弃用字段；只改布尔字段在来源表为空时仍有效，但与 luckuser/维斯路径不一致，属于遗留兼容。[判定](../../../raw/current/scripts/components/houndedtarget.lua#L16-L18)
- **以为权重影响猎犬总数：** 权重只改变玩家之间的分配比例，组内总量另有按人数的指数衰减公式；偷窃是从他人份额转移而非增量。[分配](../../../raw/current/scripts/components/hounded.lua#L284) [衰减注释](../../../raw/current/scripts/components/hounded.lua#L297-L302)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（21 行），核对 hounded 权重与偷窃消费、luckuser 负幸运写入、维斯直写与 WES_HOUND_TARGET_MULT 常量。未逐行审阅 hounded 波次生成与分组算法。
- 结构检查：2026-10-28 validate_page.py 通过（houndedtarget.md）；validate_source_links.py --strict 本页全部源码锚点通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；权重分配与偷猎犬表现未实测。
