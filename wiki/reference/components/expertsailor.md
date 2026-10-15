---
title: "expertsailor"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "角色侧航海加成数值容器：划桨力／额外限速／起锚速度／降帆加速四组 getter/setter，供航海系统读取。"
aliases: ["资深水手", "航海加成", "expertsailor"]
tags: [component]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/expertsailor.lua", "components/oar.lua", "components/anchor.lua", "components/mightiness.lua", "actions.lua", "prefabs/wolfgang.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# expertsailor

## 快速判断

- 职责与适用：纯数值容器组件，只存四个航海加成并暴露 getter／setter；不监听事件、不施力。原版只有沃尔夫冈持有（与 mightiness 一起添加，[wolfgang.lua#L670](../../../raw/current/scripts/prefabs/wolfgang.lua#L670)），由 mighty 形态数据驱动赋值（[mightiness.lua#L336-L339](../../../raw/current/scripts/components/mightiness.lua#L336-L339)）。
- 优先使用：想给角色航海加成，AddComponent 后用 `SetRowForceMultiplier` 等四个 setter 赋值；消费方（划桨、起锚、降帆）在角色带组件时自动取值。
- 重要副作用：无。组件不写存档（无 OnSave／OnLoad），数值每次形态切换由 mightiness 重新灌入。
- 修改边界：不强制持有者是谁；但字段无默认值，未赋值时 getter 返回 nil，消费方各自回退默认。

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `GetRowForceMultiplier()`／`SetRowForceMultiplier(force)` | number 倍率或 nil | 划桨推进力倍率。[L5-L11](../../../raw/current/scripts/components/expertsailor.lua#L5-L11)、[消费](../../../raw/current/scripts/components/oar.lua#L26-L27) |
| `GetRowExtraMaxVelocity()`／`SetRowExtraMaxVelocity(vel)` | number 速度增量或 nil | 划桨额外最大速度。[L13-L19](../../../raw/current/scripts/components/expertsailor.lua#L13-L19)、[消费](../../../raw/current/scripts/components/oar.lua#L26-L27) |
| `GetAnchorRaisingSpeed()`／`SetAnchorRaisingSpeed(speed)` | number 倍率或 nil | 起锚速度倍率（单位"起锚秒／秒"）。[L21-L27](../../../raw/current/scripts/components/expertsailor.lua#L21-L27)、[消费](../../../raw/current/scripts/components/anchor.lua#L196) |
| `GetLowerSailStrength()`／`SetLowerSailStrength(strength)` | number 强度或 nil | 降帆加速强度，`ACTIONS.LOWER_SAIL_BOOST` 用它给 mast 加 furler。[L29-L35](../../../raw/current/scripts/components/expertsailor.lua#L29-L35)、[消费](../../../raw/current/scripts/actions.lua#L4851) |

## 典型接入

### 给自定义角色航海加成

- 前提与选择：在角色 master postinit AddComponent 并赋值；消费方对"组件不存在"与"字段为 nil"都回退默认，因此只需要为想强化的维度赋值。
- 生效与撤销：数值在划桨（oar）、起锚（anchor）、降帆（LOWER_SAIL_BOOST）时即时读取；移除组件即恢复默认手感。
- 原版依据：沃尔夫冈的数值由 mightiness 形态状态表在形态切换时批量灌入（[mightiness.lua#L336-L339](../../../raw/current/scripts/components/mightiness.lua#L336-L339)）。

```lua
AddPrefabPostInit("wilson", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst:AddComponent("expertsailor")
    inst.components.expertsailor:SetRowForceMultiplier(1.25)
    inst.components.expertsailor:SetAnchorRaisingSpeed(1.5)
end)
```

- 调整与验证：单位对齐消费方——row_force_mult 是倍率（默认回退 1）、extra_max_velocity 是速度增量（回退 0）、anchor_raise_speed 是倍率（回退 1）、lower_sail_strength 直接替换 `TUNING.DEFAULT_SAIL_BOOST_STRENGTH` 的位置（[回退点](../../../raw/current/scripts/actions.lua#L4851)）。未单独运行验证。

## 常见陷阱与限制

- 未赋值时 getter 返回 nil 而不是 0／1：`oar.lua`／`anchor.lua`／`actions.lua` 都用 `and ... or 默认值` 兜底；自己写消费逻辑时要处理 nil，否则乘法会炸。
- `master_crewman` tag 的额外乘区（`TUNING.MASTER_CREWMAN_MULT`）在消费方叠加，与本组件无关（[anchor.lua#L196](../../../raw/current/scripts/components/anchor.lua#L196)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/expertsailor.lua（39 行），核对沃尔夫冈添加点、mightiness 赋值链与 oar／anchor／LOWER_SAIL_BOOST 三个消费点及默认回退值。未覆盖数值的实际手感与多人同步（组件仅服务端，无同步需求）。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；65 行、4948 字节，simple 档内）；validate_source_links.py --strict 通过（13 处源码引用，0 error／0 warning）；lua 示例经 parser_probe.exe --syntax-only 解析通过。
