---
title: "vasedecoration"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "花瓶装饰物标记：让物品可对 vase 标签目标生成 DECORATEVASE，装饰效果由目标 vase 组件接收。"
aliases: ["花瓶装饰物品", "装饰花瓶的材料", "vasedecoration"]
tags: [component, item, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/vasedecoration.lua", "componentactions.lua", "actions.lua", "prefabs/forgetmelots.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# vasedecoration

## 快速判断

- 职责与适用：空组件，无字段无方法，仅把宿主注册进动作组件列表；作用对象是可放入花瓶的装饰物品。客户端动作候选靠 `vasedecoration` 标签与目标 `vase` 标签判定，不读组件字段。[组件](../../../raw/current/scripts/components/vasedecoration.lua#L1-L4)、[候选](../../../raw/current/scripts/componentactions.lua#L1915-L1921)
- 优先使用：装饰物品在 pristine 前加 `vasedecoration` 标签、主机侧加组件；目标侧为带 `vase` 标签且挂 vase 组件的实体。[宿主形态](../../../raw/current/scripts/prefabs/forgetmelots.lua#L28-L43)
- 重要副作用：组件本体无逻辑；执行时要求目标 `components.vase.enabled` 并调 `vase:Decorate(doer, invobject)`。[执行](../../../raw/current/scripts/actions.lua#L2452-L2455)
- 修改边界：组件加在 `ismastersim` 判定之后，仅主机持有；客户端候选经 actionreplica 的组件 id 网络变量同步（[注册](../../../raw/current/scripts/componentactions.lua#L3182-L3190)、[客户端回读](../../../raw/current/scripts/entityscript.lua#L92-L93)）。

## 典型接入

### 让物品能装饰花瓶（原版同构）

- 前提与选择：装饰物品 prefab 内落位——标签在 `SetPristine` 前加（双端可见），组件在 mastersim 分支后加（主机标记）；与原版 forgetmelots、lightbulb、moontree_blossom、petals、wormlight 五种装饰物同一形状（[对照](../../../raw/current/scripts/prefabs/forgetmelots.lua#L28-L43)）。
- 生效与撤销：手持装饰物对 `vase` 目标右键出现 DECORATEVASE（骑乘且物品非玩家 GrandOwner 时不生成，[候选](../../../raw/current/scripts/componentactions.lua#L1916-L1919)）；执行是否成功由目标 vase 组件 `enabled` 与 `Decorate` 决定（[执行](../../../raw/current/scripts/actions.lua#L2453-L2455)）。

```lua
AddPrefabPostInit("my_flower", function(inst)
    inst:AddTag("vasedecoration")
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("vasedecoration")
end)
```

- 调整与验证：装饰的放置效果、占用槽位与移除都在目标 vase 组件侧（原版目标：endtable、vaultrelic、decor_flowervase 加 `vase` 标签）；本组件不写存读、无清理钩子。

## 常见陷阱与限制

- **只加组件不加标签没有候选**：候选判定用 `inst:HasTag("vasedecoration")`（[候选](../../../raw/current/scripts/componentactions.lua#L1919)），组件 id 列表只负责让客户端跑这段 handler；缺标签时主机侧也走不进插入分支。
- **只加标签不加组件不阻碍候选**：标签在双端 prefab 构造时都会加上，动作仍会生成，但主机侧缺组件时该物品没有"装饰物"身份，其他按组件检索的系统识别不到。
- **动作生成不等于可装饰**：目标 vase 组件 `enabled=false` 时动作仍出现，执行静默失败（[执行检查](../../../raw/current/scripts/actions.lua#L2453)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、USEITEM 候选、DECORATEVASE 执行、forgetmelots 宿主形态及 lightbulb／moontree_blossom／petals／wormlight 的标签与组件行。未覆盖：vase 组件内部、其余 mod 端行为、运行验证。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
