---
title: "migrationpetowner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "轻量透传组件：让一个物品声明并查到寄养在自己身上的宠物，供跨世界迁移与虚拟房间重置逻辑保留／放行。"
aliases: ["迁移宠物持有者", "随身宠物迁移", "migrationpetowner"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/migrationpetowner.lua", "components/inventory.lua", "components/worldmigrator.lua", "components/virtualroomset.lua", "prefabs/beef_bell.lua", "prefabs/hats.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# migrationpetowner

## 快速判断

- 职责与适用：源码注释自述"A lightweight component to easily pass through a prefab capable of migrating alongside an inventory item"（[L1-L3](../../../raw/current/scripts/components/migrationpetowner.lua#L1-L3)）：宿主（物品）通过一个回调声明身上的宠物实体，迁移／重置系统据此决定物品是否随行。本身不管理宠物，也没有状态需要存档。
- 优先使用：需要"宠物跟物品走"时 AddComponent 并 `SetPetFn(fn)`，`fn(inst)` 返回单个实体或实体数组；查询侧用 `GetPet()`（取第一个）或 `GetAllPets()`。
- 重要副作用：无事件、无 tag、无网络；副作用全部发生在消费方（迁移白名单、房间重置保留）。
- 修改边界：任何实体可挂；回调返回什么就报什么，组件不做有效性检查。`GetPet()` 被源码标注为"Partial deprecation"，新代码倾向 `GetAllPets()`（[L14](../../../raw/current/scripts/components/migrationpetowner.lua#L14)）。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetPetFn(petfn)` | `fn(inst)` → 实体或实体数组或 nil | 记录回调，不入存档。[L10-L12](../../../raw/current/scripts/components/migrationpetowner.lua#L10-L12) |
| `GetPet()` | 实体或 nil | 调回调取第一个：回调返回单实体（`EntityScript.is_instance` 兼容分支）直接返回，返回表则取 `[1]`。[L14-L24](../../../raw/current/scripts/components/migrationpetowner.lua#L14-L24) |
| `GetAllPets()` | 实体数组或 nil | 单实体返回会被包成 `{ pets }`（向后兼容），否则原样返回回调结果；无回调返回 nil。[L26-L35](../../../raw/current/scripts/components/migrationpetowner.lua#L26-L35) |

## 典型接入

### 物品携带宠物随玩家跨世界迁移

- 前提与选择：宿主是有 inventoryitem 的物品；回调按需从 spawner／petleash 等来源解析宠物实体。消费方按 tag 与组件查询，不需要额外注册。
- 生效与撤销：`components/inventory` 在迁移收集时把物品的全部宠物并入 `inst.migrationpets`（[L274-L281](../../../raw/current/scripts/components/inventory.lua#L274-L281)）；`worldmigrator` 用 `GetPet()` 禁止带宠物品单独迁移，并在放行前调 `item.OnStopUsing` 解除配对（牛铃场景，[L165-L183](../../../raw/current/scripts/components/worldmigrator.lua#L165-L183)）；`virtualroomset` 在虚拟房间重置时保留带宠物品（[L1008](../../../raw/current/scripts/components/virtualroomset.lua#L1008)、[L1017](../../../raw/current/scripts/components/virtualroomset.lua#L1017)）。
- 原版依据：牛铃 `beef_bell.lua` 回调 `GetAliveBeefalo`（[L359-L360](../../../raw/current/scripts/prefabs/beef_bell.lua#L359-L360)）； Polly 罗杰帽两处与 Princess 帽各挂一个，注释都是 "Needed for pets being on the hat and not the player"（[hats.lua#L2644-L2645](../../../raw/current/scripts/prefabs/hats.lua#L2644-L2645)、[L6425-L6426](../../../raw/current/scripts/prefabs/hats.lua#L6425-L6426)）。

```lua
AddPrefabPostInit("my_pet_carrier", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst:AddComponent("migrationpetowner")
    inst.components.migrationpetowner:SetPetFn(function(host)
        return host.components.spawner ~= nil and host.components.spawner.child or nil
    end)
end)
```

- 调整与验证：回调返回 nil 表示"没宠物"，物品按普通物品参与迁移；换宠物后无需更新组件，回调每次查询时重新解析。本例未单独运行验证。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/migrationpetowner.lua（37 行），核对 inventory／worldmigrator／virtualroomset 三处消费逻辑与 beef_bell、hats 两个原版宿主的回调设置。未覆盖跨 shard 迁移的完整网络流程。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；60 行、4876 字节，simple 档内）；validate_source_links.py --strict 通过（12 处源码引用，0 error／0 warning）；lua 示例经 parser_probe.exe --syntax-only 解析通过。

## 相关链接

- [[reference/components/worldmigrator]]：跨分片迁移门户与宠物保留协议。
