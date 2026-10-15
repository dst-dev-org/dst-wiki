---
title: "farmtiller"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "锄地生成 farm_soil 土堆，区分农田地皮资格、动作执行和锄头耐久消耗。"
aliases: ["锄地", "锄头耐久减半", "生成种植土堆", "farmtiller"]
tags: [component, item, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/farmtiller.lua", "components/map.lua", "prefabs/farm_hoe.lua", "actions.lua", "componentactions.lua", "bufferedaction.lua", "components/finiteuses.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# farmtiller

## 快速判断

- 职责与适用：锄地能力挂在工具／执行者上；合法地点先处理附近土堆，再生成 `farm_soil`，不是把普通地皮变成农田。[完整实现](../../../raw/current/scripts/components/farmtiller.lua#L1-L18) [地皮检查](../../../raw/current/scripts/components/map.lua#L209-L212)
- 优先使用：人物交互沿 `ACTIONS.TILL → Till(pt,doer)`；改耗损走 [[reference/components/finiteuses]] 的动作消费配置，不重写锄地函数。[动作](../../../raw/current/scripts/actions.lua#L4712-L4724) [锄头配置](../../../raw/current/scripts/prefabs/farm_hoe.lua#L112-L122)
- 重要副作用：成功时向 doer 发 `tilling`；附近土堆会收到 collapsesoil／breaksoil。组件自身不扣耐久。[Till](../../../raw/current/scripts/components/farmtiller.lua#L5-L16) [土堆处理](../../../raw/current/scripts/components/map.lua#L178-L183)
- 修改边界：本页服务端；直接 Till 绕过动作／SG 和成功后的物品使用链，不等同玩家挥锄一次。[动作结算](../../../raw/current/scripts/bufferedaction.lua#L22-L36)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `Till(pt,doer?)` | pt 需 x／z 及 `Get()`；成功 true，失败 false | 检查 `CanTillSoilAtPoint(pt.x,0,pt.z,false)`；允许后生成土堆，位置使用 pt:Get()。通常传地面 Vector3；没有自定义半径或批量参数。[实现](../../../raw/current/scripts/components/farmtiller.lua#L5-L16) |
| `Map:CanTillSoilAtPoint(x,y,z,ignore_tile_type)` | 资格判定 | 本组件传 false，要求农田地皮和部署净空；不要为方便跳过后者。[实现](../../../raw/current/scripts/components/map.lua#L185-L211) |
| `tilling` | 事件发给 doer，无 data | doer 可 nil；失败不发。不发给锄头或新土堆。[实现](../../../raw/current/scripts/components/farmtiller.lua#L10-L15) |
| `ACTIONS.TILL` | invobject 优先，否则 doer | 先检查物品拟态揭露，再调 farmtiller；无该组件才尝试 quagmire_tiller。[动作](../../../raw/current/scripts/actions.lua#L4712-L4724) |

玩家右键地面候选还检查地图资格；服务端执行时组件再次检查。原锄头同时配置手持／武器、inherent TILL、finiteuses 动作消费与 farmtiller，不能把只加组件当作完整新工具骨架。[候选](../../../raw/current/scripts/componentactions.lua#L2109-L2112) [工厂](../../../raw/current/scripts/prefabs/farm_hoe.lua#L96-L122)；新手持工厂见 [[cookbook/equipment/hand]]。

## 典型接入

### 普通园艺锄每次成功锄地只耗半点耐久

放目标模组 `modmain.lua`；只修改 `farm_hoe`，保留动画、地皮门禁和实际锄地。原普通锄头消费 1，金锄另有自己的倍率，本例不改金锄或热泉锄。[普通配置](../../../raw/current/scripts/prefabs/farm_hoe.lua#L114-L122) [金锄](../../../raw/current/scripts/prefabs/farm_hoe.lua#L152-L156) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("farm_hoe", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.farmtiller ~= nil
        and inst.components.finiteuses ~= nil then
        inst.components.finiteuses:SetConsumption(GLOBAL.ACTIONS.TILL, 0.5)
    end
end)
```

只设每次动作基础消耗，不增加现有剩余次数。成功 BufferedAction 通知物品，EntityScript 分派至组件，finiteuses 再应用 efficientuser／自定义消费；最终扣值可能不是 0.5。停用后重新初始化恢复配置，不补回已经消耗的耐久。[成功通知](../../../raw/current/scripts/bufferedaction.lua#L26-L32) [分派](../../../raw/current/scripts/entityscript.lua#L1811-L1814) [最终消费](../../../raw/current/scripts/components/finiteuses.lua#L101-L114)；倍率详见 [[reference/components/efficientuser]]。

## 常见陷阱与限制

- **Till 成功就应该扣耐久**：直接调用仅锄地／发事件，耐久依赖外层成功物品动作；手动脚本应明确选完整动作或自行一次结算，避免重复 Use。[组件](../../../raw/current/scripts/components/farmtiller.lua#L5-L16) [消费链](../../../raw/current/scripts/bufferedaction.lua#L26-L32)
- **任意点 SpawnPrefab 土堆代替 Till**：会绕过农田／净空和旧土堆处理；保留地图检查与原方法，不将演示扩成无条件批量改地。[地图](../../../raw/current/scripts/components/map.lua#L178-L211)
- **工具附上 farmtiller 就有完整交互**：原工具还有工厂、手持、资源和动作接入；组件没有自动添加这些能力。[原工厂](../../../raw/current/scripts/prefabs/farm_hoe.lua#L96-L122)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对本页 sources 变更文件 actions.lua／componentactions.lua（R-01/R-04）、components/map.lua（C-23）、entityscript.lua（R-08）：TILL 候选／执行、CanTillSoilAtPoint 与事件分派均未被本次 hunk 触及（C-23/R-08 变化在本页锚点之后），结论无修订；锚点重定位 4 处；未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整组件、地图资格／土堆事件、动作候选／执行、原普通与金锄配置、成功通知及 finiteuses 消费。未审校 farm_soil／种子和 SG 动画完整流程。
- 结构检查：格式、来源、锚点、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明动画、预测或实际耐久。
- 游戏运行：未运行；多人／手柄、障碍物／土堆重叠、连续操作和其他模组耗损倍率待实测。

- 2026-10-28 build 26230496：actions.lua TILL、componentactions.lua 候选段因拟态揭示三参化／sweeper handler 移位；TILL 语义未变。