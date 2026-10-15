---
title: "lightningblocker"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "给实体登记圆形避雷范围与落雷回调，让世界天气把落雷改为无伤害的雷声并通知宿主。"
aliases: ["避雷", "落雷阻挡", "避雷柱", "雷击回调", "lightningblocker"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/lightningblocker.lua", "components/weather.lua", "componentutil.lua", "prefabs/lightning.lua", "prefabs/oceantree_pillar.lua", "prefabs/watertree_pillar.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lightningblocker

## 快速判断

- 职责与适用：挂在**要挡住落雷的实体**上，只保存一个平方半径和可选回调；真正的命中判定、雷种切换与生成雷实体都在世界 weather 组件里完成。[构造](../../../raw/current/scripts/components/lightningblocker.lua#L10-L19) [判定入口](../../../raw/current/scripts/components/weather.lua#L630-L661)
- 优先使用：`SetBlockRange(range)` 传世界单位距离，`SetOnLightningStrike(fn)` 注册 `fn(inst, pos)`；宿主被阻挡时会收到一次回调。原版两棵海洋巨树柱即此模式。[小柱](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L600-L602) [大柱](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L495-L497)
- 重要副作用：被阻挡的落雷会换成 `thunder`，只播雷声与闪屏，不产生范围电击与点燃；落点仍是原本会落雷的位置，不是柱子中心。[阻挡分支](../../../raw/current/scripts/components/weather.lua#L678-L680) [雷种效果](../../../raw/current/scripts/componentutil.lua#L1065-L1089)
- 修改边界：仅服务端；组件无网络字段、无存档；客户端既没有组件也没有 `lightningblocker` 标签，不能用于客户端判定。[增删标签](../../../raw/current/scripts/components/lightningblocker.lua#L1-L8)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `block_rsq` | number（平方距离） | 真正参与比较的字段；由属性回调维护 `lightningblocker` 标签，`> EPSILON(0.001)` 才加标签，否则摘除。[属性](../../../raw/current/scripts/components/lightningblocker.lua#L1-L8) [初值](../../../raw/current/scripts/components/lightningblocker.lua#L10-L13) |
| `SetBlockRange(newrange)` | number?（世界单位） | 存 `newrange * newrange`；传 nil 存 nil 并因属性回调摘标签，等于关闭阻挡。[L25-L27](../../../raw/current/scripts/components/lightningblocker.lua#L25-L27) |
| `SetOnLightningStrike(fn)` | `function(inst, pos)` | 只保存函数，不校验、不包装；同一宿主后设置者覆盖前者。[L29-L31](../../../raw/current/scripts/components/lightningblocker.lua#L29-L31) |
| `DoLightningStrike(pos)` | Vector3 | 仅当回调存在时转调 `on_strike(self.inst, pos)`；不判距离、不扣血、不生成特效，可被外部直接调用。[L33-L37](../../../raw/current/scripts/components/lightningblocker.lua#L33-L37) |
| `OnRemoveFromEntity()` | 无参 | 摘 `lightningblocker` 标签，此后不再被落雷查询到。[L21-L23](../../../raw/current/scripts/components/lightningblocker.lua#L21-L23) |

### 落雷链路（`ms_sendlightningstrike`）

1. weather 组件监听 `ms_sendlightningstrike`，正常运行时的雷也由它自己 `PushEvent` 触发。[监听](../../../raw/current/scripts/components/weather.lua#L823) [天气触发](../../../raw/current/scripts/components/weather.lua#L1110-L1114)
2. 以落雷点为中心 40 单位查询，排除 `playerghost`／`INLIMBO`，只收 `lightningrod`／`lightningtarget`／`lightningblocker` 之一。[查询参数](../../../raw/current/scripts/components/weather.lua#L627-L635)
3. 所有带 blocker 的实体先被记入 `blockers`；若落雷点落在某 blocker 的 `block_rsq + 0.0001` 内，即判为被阻挡（取遍历顺序中第一个命中，不比较最近）。[L636-L651](../../../raw/current/scripts/components/weather.lua#L636-L651)
4. 被阻挡时：以**原落雷点**调用 `DoLightningStrike`，雷种改为 `thunder`，随后在原点生成该雷实体。[L676-L680](../../../raw/current/scripts/components/weather.lua#L676-L680) [生成](../../../raw/current/scripts/components/weather.lua#L729)
5. 先被避雷针或目标重定向、再落进某 blocker 范围时，同样改 `thunder` 并回调；这条检查在避雷针与普通目标两个分支各有一份。[避雷针分支](../../../raw/current/scripts/components/weather.lua#L684-L693) [目标分支](../../../raw/current/scripts/components/weather.lua#L705-L714)
6. `thunder` 只播雷声、闪屏与震屏，`StrikeLightningAtPoint` 只对 `lightning` 做电击与点燃，因此阻挡后没有范围伤害。[雷实体](../../../raw/current/scripts/prefabs/lightning.lua#L128-L129) [伤害条件](../../../raw/current/scripts/componentutil.lua#L1070-L1089)

阻挡分支排在避雷针之前：落雷点同时落在 blocker 范围与避雷针范围内时，blocker 先命中。[L678-L682](../../../raw/current/scripts/components/weather.lua#L678-L682)

## 典型接入

### 原版宿主：两棵海洋巨树柱

- 目标与位置：两者都在 `entity:SetPristine()` 与服务端守卫之后装配，客户端不会执行。[小柱守卫](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L566-L570) [大柱守卫](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L445-L449)
- 数值与单位：小柱 `TUNING.SHADE_CANOPY_RANGE_SMALL`＝22，大柱 `TUNING.SHADE_CANOPY_RANGE`＝28，均为世界单位；内部存平方值。[小柱](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L600-L602) [大柱](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L495-L497) [常量](../../../raw/current/scripts/tuning.lua#L6334-L6335)
- 回调写法：两者都只声明 `OnLightningStrike(inst)`，忽略第二个参数；用 `_lightning_drop_task` 做 20 帧延迟掉落，并在任务未完成时直接返回，形成天然冷却。[小柱回调](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L481-L494) [大柱回调](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L307-L320)
- 生效与撤销：无存档、无网络变量；读档由 prefab 工厂重新设置范围与回调，实体移除即摘标签失效。

### 模组最小宿主

- 目标文件：`scripts/prefabs/my_lightning_rod.lua`，在 `modinfo.lua` 的 `PrefabFiles` 中登记，服务端 `SpawnPrefab("my_lightning_rod")` 生成。
- 端与资源：仅服务端逻辑，无贴图／音频资源；本示例不含外观、可放置与存档，只演示组件接线。
- 生效与撤销：范围内落雷被换成雷声并触发回调；`RemoveComponent("lightningblocker")` 或删除实体后不再参与判定，已经排队的雷不会回收。

```lua
-- scripts/prefabs/my_lightning_rod.lua
local BLOCK_RANGE = 22 -- 世界单位，原版小柱同值

local function OnStrike(blocker, pos)
    -- pos 是本次本会落雷的位置（Vector3），不是柱子位置
    local nugget = SpawnPrefab("goldnugget")
    nugget.Transform:SetPosition(pos:Get())
end

local function fn()
    local inst = CreateEntity()

    inst.entity:AddTransform()
    inst.entity:AddNetwork()

    inst.entity:SetPristine()
    if not TheWorld.ismastersim then
        return inst
    end

    inst:AddComponent("lightningblocker")
    inst.components.lightningblocker:SetBlockRange(BLOCK_RANGE)
    inst.components.lightningblocker:SetOnLightningStrike(OnStrike)

    return inst
end

return Prefab("my_lightning_rod", fn)
```

替换点：`BLOCK_RANGE` 决定覆盖半径（世界单位，比较时用平方），`OnStrike` 换成宿主的真实反应；不需要自己查距离或手动加标签，weather 只在命中范围内回调。

## 常见陷阱与限制

- **回调签名有两个参数**：`on_strike(inst, pos)`；原版两柱只写 `OnLightningStrike(inst)` 所以忽略 `pos`。`pos` 是 weather 传入的 Vector3，被其它代码直接调用 `DoLightningStrike` 时可能为 nil，回调里应自判。[L33-L37](../../../raw/current/scripts/components/lightningblocker.lua#L33-L37)
- **阻挡不移动落点**：`thunder` 生成在原目标位置，柱子不会替目标挨这一下；只有先重定向到避雷针／目标，才会在重定向点再检查 blocker 范围。[L678-L680](../../../raw/current/scripts/components/weather.lua#L678-L680)
- **组件自己不做判定**：能否被查到取决于实体是否带 `lightningblocker` 标签，而标签由 `block_rsq` 属性回调维护；若手动 `AddTag("lightningblocker")` 而 `block_rsq` 为 nil，weather 的 `block_rsq + 0.0001` 会直接报错。[标签维护](../../../raw/current/scripts/components/lightningblocker.lua#L2-L8) [比较](../../../raw/current/scripts/components/weather.lua#L650)
- **半径是距离，比较是平方**：`SetBlockRange` 收距离再平方；距离小到平方值不超过 0.001 时视为无效并摘标签。用 `GetDistanceSqToPoint` 做三维距离比较，跨平台高度也会计入。[L25-L27](../../../raw/current/scripts/components/lightningblocker.lua#L25-L27) [L649-L651](../../../raw/current/scripts/components/weather.lua#L649-L651)
- **多个阻挡范围重叠时取遍历顺序第一个**：源码变量名是 `closest_blocker`，但只判断“是否已找到”，并不比较距离；重叠布局下触发的柱子不确定。[L649-L651](../../../raw/current/scripts/components/weather.lua#L649-L651)
- **无存档**：组件没有保存函数，读档只靠 prefab 工厂重建；在运行期改过半径或回调的宿主需要重设。[L10-L31](../../../raw/current/scripts/components/lightningblocker.lua#L10-L31)
- **只处理事件通道**：本页只核验 `ms_sendlightningstrike` 这条落雷入口（天气、法杖、书等都用它）；直接生成雷实体的路径不经过此判定。[触发点](../../../raw/current/scripts/components/weather.lua#L1110-L1114)
- **阻挡后仍有雷实体**：会生成 `thunder`（网络实体，0.5 秒后自删），不是“完全没有雷”；只是没有电击、点燃与伤害。[雷实体](../../../raw/current/scripts/prefabs/lightning.lua#L99-L126)
- **客户端没有判定入口**：原版两柱都在服务端守卫之后装配，落雷查询与回调只在服务端 weather 里进行；客户端即使能读到同步过去的标签，也拿不到不存在的组件字段与回调，不能自行判断“这一下会不会被挡”。[小柱守卫](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L566-L570)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 3 处（componentutil +40、tuning +14，区间文本逐段比对一致）；结论修订：无（SHADE_CANOPY_RANGE 系键值未变）。
- 依据：tuning 键级 hunk 地图＋变更清单命中 componentutil.lua、tuning.lua。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖构造与属性回调、范围设置与标签、回调转调、移除清理、weather 的查询参数与三个阻挡分支、雷种与伤害条件、两个原版宿主的装配位置、半径常量与回调冷却。
- Lua 示例：仅按 Lua 5.1 做语法检查，未执行、未进游戏；示例无外观／放置／存档。
- 未运行游戏；未覆盖：雷声实体在多人端的表现、重叠 blocker 的实际选择顺序、运行期改半径、其它直接生成雷的代码路径。
- 结构检查：`validate_page.py` 无 issue；`validate_source_links.py --strict` 为 0 error／0 warning（本页 42 处源码引用）；示例 753 字节经 `parser_probe --syntax-only` 通过。

相关：[[reference/components/sheltered]]；[[reference/components/rainimmunity]]。
