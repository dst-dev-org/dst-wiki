---
title: "aoespell"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "地点施法的服务端门禁、回调返回与施法事件；实际效果和消耗由工厂实现。"
aliases: ["aoespell","地点施法","范围技能","施法返回值","CASTAOE"]
tags: ["component","callback","actions","server-client"]
dst_build: "steam-buildid:26230496"
sources: ["components/aoespell.lua","prefabs/slingshot.lua","actions.lua","componentactions.lua","components/map.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# aoespell

## 快速判断

- 职责与适用：地点施法执行器，CanCast 检查可用条件，CastSpell 调已注册效果；不自动生成范围伤害。选点表现由 [[reference/components/aoetargeting]]／[[reference/components/reticule]] 协作。[components/aoespell.lua:11](../../../raw/current/scripts/components/aoespell.lua#L11)、[components/aoespell.lua:26](../../../raw/current/scripts/components/aoespell.lua#L26
- 优先使用：SetSpellFn 注册效果，保留 CASTAOE 的 CanCast→CastSpell 顺序；直接调 CastSpell 不重做门禁。[actions.lua:4535](../../../raw/current/scripts/actions.lua#L4535
- 重要副作用：回调无返回被视为成功，并给有效 doer 发 oncastaoespell；成功不证明掉血／扣料。[components/aoespell.lua:11](../../../raw/current/scripts/components/aoespell.lua#L11
- 修改边界：服务端挂 aoespell，双端瞄准组件另按原版工厂；技能特定冷却、距离和伤害须明确实现，不靠组件名推定。[prefabs/slingshot.lua:765](../../../raw/current/scripts/prefabs/slingshot.lua#L765)、[prefabs/slingshot.lua:858](../../../raw/current/scripts/prefabs/slingshot.lua#L858

## 关键接口

| 接口 | 参数／返回 | 条件、作用与证据 |
|---|---|---|
| `SetSpellFn(fn)` | `(item, doer, pos) → success, reason` | 替换当前效果，nil 可撤销；不是与旧函数自动叠加。[components/aoespell.lua:7](../../../raw/current/scripts/components/aoespell.lua#L7 |
| `CanCast(doer, pos)` | bool | 无 spellfn 拒绝；spellbook 场景还验使用权限、物品 GrandOwner／fuel 或人物自身；再验 targeting enabled、骑乘与地图。需有效 doer／pos。[components/aoespell.lua:26](../../../raw/current/scripts/components/aoespell.lua#L26 |
| `CastSpell(doer, pos)` | success, reason | 不调用 CanCast；两返回均 nil 时改 true，仅 reason 非 nil 时保留 nil。即使无 spellfn 也默认 true。[components/aoespell.lua:11](../../../raw/current/scripts/components/aoespell.lua#L11 |
| `oncastaoespell` | doer 事件 | payload 为 item／pos／success，无 reason；失败也发送给有效 doer，不是消耗指令。[components/aoespell.lua:20](../../../raw/current/scripts/components/aoespell.lua#L20 |

地图检查由 Map:CanCastAtPoint 执行：alwayspassable 可绕过地面／阻挡分支，但 deployradius 仍参与净空检查。CanCast 不读 targeting.range，也不自动读 rechargeable 或 spellbookcooldowns。动作候选只是客户端预筛，执行入口仍须审阅。[components/map.lua:1101](../../../raw/current/scripts/components/map.lua#L1101)、[componentactions.lua:2027](../../../raw/current/scripts/componentactions.lua#L2027

（26148036 核验：R-01/R-04 新增物品 `nomagiccast` 门控只拦 CASTSPELL／BLINK／CORRUPTNIGHTMARE 及 spellcaster／blinkstaff／corruption 动作筛选，CASTAOE 执行 fn 与 POINT.aoespell 候选均无该检查——给物品加此标签不冻结本组件的地点施法；标签当前唯一动态添加／移除方是腐化王杖 king_cane。[prefabs/king_cane.lua:119](../../../raw/current/scripts/prefabs/king_cane.lua#L119)、[:127](../../../raw/current/scripts/prefabs/king_cane.lua#L127)）

## 典型接入

### 双弹弓地点施法增加服务端二十单位距离门禁

目标模组 `modmain.lua`，只适用原版 slingshot2ex；工厂已经注册地点技能、资源、弹药切换／发射与客户端瞄准。本例保留旧效果全部返回，在其前检查水平距离；不改实际弹体最大距离或客户端指示器。[prefabs/slingshot.lua:785](../../../raw/current/scripts/prefabs/slingshot.lua#L785)、[prefabs/slingshot.lua:915](../../../raw/current/scripts/prefabs/slingshot.lua#L915

```lua
AddPrefabPostInit("slingshot2ex", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local spell = inst.components.aoespell
    if spell == nil or spell.spellfn == nil then return end
    local original = spell.spellfn
    spell:SetSpellFn(function(item, doer, pos)
        if doer == nil or not doer:IsValid() or pos == nil then
            return false
        end
        local x, y, z = doer.Transform:GetWorldPosition()
        local dx, dz = pos.x - x, pos.z - z
        if dx * dx + dz * dz > 20 * 20 then return false end
        return original(item, doer, pos)
    end)
end)
```

二十是世界水平距离，边界包含二十；拒绝时不运行原效果。原版效果只利用所选点确定方向再发射，不能据本例承诺弹体仅飞二十单位。停用后新建／载入物品恢复工厂函数；无新增资源／任务。[prefabs/slingshot.lua:795](../../../raw/current/scripts/prefabs/slingshot.lua#L795

## 常见陷阱与限制

- **只调 CastSpell 即完成授权**：它不查 enabled／地图／所有权／冷却，直接调用者须先查门禁和技能条件。[components/aoespell.lua:11](../../../raw/current/scripts/components/aoespell.lua#L11)、[components/aoespell.lua:26](../../../raw/current/scripts/components/aoespell.lua#L26
- **无返回表示失败**：nil,nil 被规范为 true；包装旧回调时保留 success／reason 语义，不写 `if not original(...) then` 误拒原版无返回效果。[components/aoespell.lua:14](../../../raw/current/scripts/components/aoespell.lua#L14)、[prefabs/slingshot.lua:785](../../../raw/current/scripts/prefabs/slingshot.lua#L785
- **所有物品都会自动验证持有者**：GrandOwner／fuel 检查位于 spellbook 分支，非 spellbook 不能借用其保证。[components/aoespell.lua:31](../../../raw/current/scripts/components/aoespell.lua#L31
- **CanCast 的 true 代表射程合法**：这里只核对地图等条件，距离和实际效果由动作／技能消费者另管。[components/aoespell.lua:52](../../../raw/current/scripts/components/aoespell.lua#L52

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中变更文件 actions.lua、componentactions.lua、components/map.lua（aoespell.lua／slingshot.lua 本版未变）对照 build-26148036 更新报告核验，条目 R-01、R-04：nomagiccast 门控不含 CASTAOE／POINT.aoespell（结论修订 1 处，见正文标注），map.lua 仅新增 CanPointHaveRain 等雨量检查、CanCastAtPoint 本体未变；锚点重定位 3 处；未覆盖：运行验证。
- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整 aoespell、CASTAOE 候选／执行、地图检查与 slingshot2ex 工厂／效果。未审校全部施法 SG、角色权限或弹药内部。
- 结构检查：发布时检查格式、链接、源码行号和唯一登记；完整 modmain 示例仅作 Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；客户端超程提示、预测、技能树／弹药、多人和其他模组包装顺序待测。

相关：[[reference/components/spellcaster]]；[[reference/components/spellbook]]；[[reference/components/spellbookcooldowns]]。
