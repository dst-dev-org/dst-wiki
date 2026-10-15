---
title: "pocketwatch_dismantler"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "精密工具拆怀表的资格与执行分工、完整配方返还和噩梦燃料排除；不提供通用物品拆解协议。"
aliases: ["禁止拆解回溯表", "精密工具拆表", "怀表拆解返材料", "pocketwatch_dismantler"]
tags: [component, item, loot, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/pocketwatch_dismantler.lua", "components/lootdropper.lua", "prefabs/pocketwatch_dismantler.lua", "prefabs/pocketwatch_common.lua", "prefabs/pocketwatch.lua", "componentactions.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# pocketwatch_dismantler

## 快速判断

- 职责与适用：精密工具执行原怀表拆解，按配方全额材料表排除 nightmarefuel；有 itemmimic 时改为使拟态生物变恶，不走返材料／删除路径。[执行](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L16-L40)
- 优先使用：改资格包 CanDismantle，保留原冷却／clockmaker；合法执行由 DISMANTLE_POCKETWATCH 动作先查再调 Dismantle。不要直接调执行方法来测试“能否拆”。[资格](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L5-L13) [动作](../../../raw/current/scripts/actions.lua#L5687-L5693)
- 重要副作用：Dismantle 生成返还实体，正常路径删除目标；自身不复查资格、不返回成功。工具没有在该执行链扣耐久或消费自身。[完整方法](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L16-L40)
- 修改边界：服务端；候选要求 clockmaker 与目标 pocketwatch 标签，执行又假定 inventoryitem／lootdropper／配方等存在。挂此组件不等于支持拆所有物品。[候选](../../../raw/current/scripts/componentactions.lua#L1568-L1571) [原目标工厂](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L66-L85)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `CanDismantle(target,doer)` | 目标有 rechargeable 且未充满时先 false,ONCOOLDOWN；之后检查 doer 的 clockmaker，无该标签 false，否则 true。缺 rechargeable 本身不拒绝，也不在此检查 pocketwatch 标签。[方法](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L5-L13) |
| `Dismantle(target,doer)` | 从末端 owner 的 inventory／container 选接收方，pocketdimension_container 排除；有接收方用工具位置作发放回退点，否则用 doer 位置散落。[接收方](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L16-L19) [发放](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L28-L40) |
| 正常配方返还 | GetFullRecipeLoot(AllRecipes[target.prefab])，按 ingredients 与 deconstruct 递归展开，不套普通 HAMMER 比例或耐久返还；随后过滤 nightmarefuel。目标无配方等情况未作容错。[调用](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L28-L40) [展开](../../../raw/current/scripts/components/lootdropper.lua#L114-L131) |
| 拟态分支 | 若有 itemmimic，播放声音并 TurnEvil(doer)，不生成正常配方返还，也不走末尾 target:Remove。[分支](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L21-L28) |

正常路径源码注释说“先删除以腾空间”，但实际循环发放在前、target:Remove 在后；应按代码顺序判断满库存行为，不以注释反推。本页不保证材料仍回到原格位或全部进入库存。[顺序](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L28-L40)

## 典型接入

### 精密工具禁止拆解原回溯表，保留其他怀表原规则

目标模组 `modmain.lua`；无新资源，工具工厂已服务端挂载组件。只改执行资格，不改配方、返还比例或客户端候选；原 pocketwatch_recall 注册名由实际工厂确认。[工具](../../../raw/current/scripts/prefabs/pocketwatch_dismantler.lua#L31-L45) [目标注册](../../../raw/current/scripts/prefabs/pocketwatch.lua#L423-L429)

```lua
AddPrefabPostInit("pocketwatch_dismantler", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local dismantler = inst.components.pocketwatch_dismantler
    if dismantler == nil then return end
    local old = dismantler.CanDismantle
    dismantler.CanDismantle = function(self, target, doer)
        local allowed, reason = old(self, target, doer)
        if not allowed then return allowed, reason end
        if target.prefab == "pocketwatch_recall" then return false end
        return allowed, reason
    end
end)
```

先保留原冷却原因／角色门禁，再拒绝回溯表；仅返回 false，不编造本地化失败字符串。原动作会因此不执行拆解，原候选仍可能显示；若需隐藏按钮须另审双端 ComponentAction，不能据服务端门禁声称 UI 已同步。[动作](../../../raw/current/scripts/actions.lua#L5687-L5693) [候选](../../../raw/current/scripts/componentactions.lua#L1568-L1571)

停用并重载后恢复原方法，不改保存数据或回溯标记；已经拆掉的目标不恢复。其他代码直接调 Dismantle 会绕过资格包装。本例只保护原 Prefab 名，模组新增怀表须明确加入自己的保护名单；回溯坐标机制另见 [[reference/components/recallmark]]。

## 常见陷阱与限制

- **CanDismantle 为 true 证明任意目标可拆**：它不检目标所有必要组件／配方；原候选与怀表工厂共同构成前提，不能绕过后拿岩石或自定义空壳调用。[资格](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L5-L13) [目标工厂](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L49-L85)
- **按普通锤拆比例返还，或含噩梦燃料**：此处选 GetFullRecipeLoot，再明确排除 nightmarefuel；改普通 lootdropper 掉落名单不等于改拆表材料。[执行](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L28-L40)
- **先删表再给材料／保证库存有空格**：实际相反；满库存去向按原 GiveItem／散落链处理，不能只信注释。[实际循环](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L28-L40)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中的漂移文件 actions.lua（R-01）、componentactions.lua（R-04）、components/lootdropper.lua（C-22）逐 hunk 对照，锚点重定位 2 处（DISMANTLE_POCKETWATCH 5668→5677），结论修订 0 处——拆解动作、clockmaker／pocketwatch 候选与 GetFullRecipeLoot 展开（lootdropper.lua:114-131 未变）均未变；lootdropper 本轮只改 FlingItem（nofling／Teleport 回退，C-22），与本页无配方返还路径无交集；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整拆解组件、动作／候选、工具与共用怀表工厂、回溯表注册、完整配方展开。未全面审校拟态转化、所有怀表配方／SG、SpawnPrefab 失败或库存溢出实际位置。
- 结构检查：本批格式、来源、链接／锚点、大小与登记检查；完整保护包装经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；冷却／角色／保护目标、候选提示、拟态、满库存与口袋维度、材料数量和其他模组包装待实测。
