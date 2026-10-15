---
title: "miasmawatcher"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家侧瘴气检测与减速：进出瘴气源时同步 classified、挂 debuff，并按护目镜／鬼魂视觉与骑乘豁免给 locomotor 挂减速。"
aliases: ["瘴气检测", "瘴气减速", "miasmawatcher"]
tags: [component, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/miasmawatcher.lua", "prefabs/miasma_cloud_fx.lua", "prefabs/player_common.lua", "prefabs/player_classified.lua", "simutil.lua", "prefabs/woodie.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# miasmawatcher

## 快速判断

- 职责与适用：检测玩家是否处于瘴气云内并施加对应减速；只对挂了它的玩家实体生效，服务端组件。[构造](../../../raw/current/scripts/components/miasmawatcher.lua#L3-L7)
- 优先使用：加/移除瘴气源用 `AddMiasmaSource(src)`／`RemoveMiasmaSource(src)`（由瘴气云实体调用）；查询用 `IsInMiasma()`；改减速倍率用 `SetMiasmaSpeedMultiplier(mult)`。[UpdateMiasmaWalkSpeed](../../../raw/current/scripts/components/miasmawatcher.lua#L102-L113)
- 重要副作用：首次进入瘴气写 `player_classified.isinmiasma`（客户端经 `miasmalevel` 事件读）、`AddDebuff("miasmadebuff")`，并按倍率在 locomotor 上挂 `miasma` 外部减速；改变量名客户端可见。[AddMiasmaSource](../../../raw/current/scripts/components/miasmawatcher.lua#L19-L28)
- 修改边界：减速判定依赖 `playervision`、`rider`、`locomotor` 组件，仅适用于玩家；世界级开关事件 `miasmacloudexists` 由瘴气云 prefab 推送，组件不生成云。[监听](../../../raw/current/scripts/components/miasmawatcher.lua#L9-L16)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `hasmiasmasource` | SourceModifierList(boolean) | 多瘴气源聚合成一个布尔；任一源存在即在瘴气中。[构造](../../../raw/current/scripts/components/miasmawatcher.lua#L7) |
| `AddMiasmaSource(src)` | 无返回 | 记录源 `src`；从无到有时写 classified、调 `UpdateMiasmaWalkSpeed`、挂 `miasmadebuff`。瘴气云在 `MIASMA_RADIUS` 内对可见实体调用。[云侧调用](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L443-L453) |
| `RemoveMiasmaSource(src)` | 无返回 | 移除源；归零时反向写 classified、解除减速与 debuff。[L31-L40](../../../raw/current/scripts/components/miasmawatcher.lua#L31-L40) |
| `IsInMiasma()` | 布尔 | 返回 `hasmiasmasource:Get()`；simutil 在非玩家实体上作服务端判定回退。[IsInMiasma](../../../raw/current/scripts/components/miasmawatcher.lua#L42-L44)、[simutil](../../../raw/current/scripts/simutil.lua#L536-L538) |
| `SetMiasmaSpeedMultiplier(mult)` | 数值，clamp 到 0..1 | 默认 `TUNING.MIASMA_SPEED_MOD = .4`；设 ≥1 时卸掉监听等于免疫。[L85-L100](../../../raw/current/scripts/components/miasmawatcher.lua#L85-L100)、[tuning](../../../raw/current/scripts/tuning.lua#L7730) |
| `ToggleMiasma(active)` | 无返回 | 世界事件 `miasmacloudexists`（true/false）驱动；控制减速监听组的增删，不直接判"在瘴气里"。[L71-L83](../../../raw/current/scripts/components/miasmawatcher.lua#L71-L83)、[推送](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L651) |
| `UpdateMiasmaWalkSpeed()` | 无返回 | 无源、护目镜视觉、鬼魂视觉或骑乘时不减速；否则 `locomotor:SetExternalSpeedMultiplier(inst,"miasma",mult)`。[L102-L113](../../../raw/current/scripts/components/miasmawatcher.lua#L102-L113) |
| `enabled` | 布尔，默认 false | 世界里存在瘴气云才为 true；仅决定监听组是否挂载。[ToggleMiasma](../../../raw/current/scripts/components/miasmawatcher.lua#L74-L82) |

## 典型接入

### 原版宿主：谁在挂它、谁改倍率

- 唯一挂载点：`player_common` master 段 `AddComponent("miasmawatcher")`，与 stormwatcher 系并列。[挂载](../../../raw/current/scripts/prefabs/player_common.lua#L2742)
- 倍率调整：频道施法临时取 `MinMult(TUNING.MIASMA_SPEED_MOD, mult)` 并在结束时还原（[L312](../../../raw/current/scripts/prefabs/player_common.lua#L312)、[L322](../../../raw/current/scripts/prefabs/player_common.lua#L322)）；woodie 变回人形恢复默认、海狸／驼鹿／鹅形态设 1（形态免疫，[恢复](../../../raw/current/scripts/prefabs/woodie.lua#L1222)、[形态](../../../raw/current/scripts/prefabs/woodie.lua#L1278)）。
- 客户端读取：classified 变量 `miasmawatcher.isinmiasma`（[定义](../../../raw/current/scripts/prefabs/player_classified.lua#L1466)）脏事件转发 `miasmalevel`（[L732](../../../raw/current/scripts/prefabs/player_classified.lua#L732)）；`inst.IsInMiasma` 客户端读 classified，服务端走本组件（[绑定](../../../raw/current/scripts/prefabs/player_common.lua#L2283)、[读取](../../../raw/current/scripts/prefabs/player_common.lua#L272)）。

### 模组：让自定义云体也产生瘴气效果

- 前提与选择：目标玩家已自带组件（所有角色都有）；只需在服务端对范围内玩家调 `AddMiasmaSource(你的云实体)`，退出时 `RemoveMiasmaSource`。
- 生效与撤销：同一源重复 `Add` 幂等（SourceModifierList 覆盖同 key）；忘移除会让玩家永久带 debuff 与减速，云 `onremove` 时必须清理。
- 原版依据：瘴气云本体按周期 `FindEntities(MIASMA_RADIUS)` 后逐个调用（[add](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L443-L453)、[remove](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L406-L407)）。

```lua
-- modmain.lua：自定义瘴气圈，玩家进入即视为在瘴气中（服务端）
AddPrefabPostInit("world", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst:DoPeriodicTask(1, function()
        for _, v in ipairs(GLOBAL.AllPlayers) do
            local inw, miasma = false, v.components and v.components.miasmawatcher
            -- 自行按你的区域判定 inw；这里示意调用契约
            if miasma ~= nil then
                if inw and not miasma:IsInMiasma() then
                    miasma:AddMiasmaSource(inst)
                elseif not inw and miasma:IsInMiasma() then
                    miasma:RemoveMiasmaSource(inst)
                end
            end
        end
    end)
end)
```

- 调整与验证：示例只做 Lua 语法检查，未运行；`inw` 判定需替换为真实区域逻辑，`inst` 作源 key 保证成对增删。

## 常见陷阱与限制

- `miasmadebuff` 由 `AddDebuff` 挂载，本体在 prefab `miasmadebuff`；移除走 `RemoveDebuff("miasmadebuff")`，与源列表一一对应，直接清 debuff 不断源会立刻被下帧状态覆盖。[L27](../../../raw/current/scripts/components/miasmawatcher.lua#L27)、[L38](../../../raw/current/scripts/components/miasmawatcher.lua#L38)
- 减速监听组只在 `enabled && mult < 1` 时挂载：世界里没有瘴气云时改倍率不生效，等 `miasmacloudexists` true 才补挂；判定时序见 [ToggleMiasma](../../../raw/current/scripts/components/miasmawatcher.lua#L71-L83)。
- 骑乘与视觉豁免依赖事件刷新：`mounted`／`dismounted`／`gogglevision`／`ghostvision` 触发重算，不主动轮询；绕过事件直接改状态会滞后一拍。[L50-L55](../../../raw/current/scripts/components/miasmawatcher.lua#L50-L55)
- 组件移除时只清监听不清 debuff 与源列表；运行期 `RemoveComponent("miasmawatcher")` 需自行处理残留状态。[OnRemoveFromEntity](../../../raw/current/scripts/components/miasmawatcher.lua#L65-L69)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖构造与监听、`AddMiasmaSource`／`RemoveMiasmaSource`／`IsInMiasma`／`ToggleMiasma`／`SetMiasmaSpeedMultiplier`／`UpdateMiasmaWalkSpeed` 全部分支；核对瘴气云 add/remove 调用、`miasmacloudexists` 推送、player_common 挂载与倍率调整、woodie 形态倍率、classified 客户端链路；未覆盖 debuff prefab 内部实现与表现层。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 与示例 Lua 语法检查通过（见页尾批记录，未达标不交付）。

相关：[[reference/components/sandstormwatcher]]、[[reference/components/moonstormwatcher]]、[[reference/components/locomotor]]。
