---
title: "worldmeteorshower"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "森林世界组件：决定流星矿石是否升级为月岩壳（rock_moon_shell）的累积概率与外部加成修正。"
aliases: ["流星雨世界组件", "月岩壳概率", "worldmeteorshower"]
tags: [component, loot]
dst_build: "steam-buildid:26230496"
sources: ["components/worldmeteorshower.lua", "prefabs/forest.lua", "prefabs/shadowmeteor.lua", "components/meteorshower.lua", "prefabs/moon_altar.lua", "prefabs/hermitcrab.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# worldmeteorshower

## 快速判断

- 职责与适用：挂在 forest 世界（TheWorld）上，控制陨石掉落 `rock_moon` 时向 `rock_moon_shell` 升级的概率；仅服务端存在（[断言](../../../raw/current/scripts/components/worldmeteorshower.lua#L9)）。
- 优先使用：想改月岩壳掉率就操作 `moonrockshell_chance_additionalodds` 修正表，不要直接改 `moonrockshell_chance`（它是持久化的累积值）。
- 重要副作用：`GetMeteorLootPrefab` 有状态——每颗 rock_moon 都会推进累积概率；重复调用同一掉落会连续推进（[实现](../../../raw/current/scripts/components/worldmeteorshower.lua#L22-L41)）。
- 修改边界：OnSave 只存累积概率，不存修正表；修正项需来源方自行维护增删（原版 celestial_altar／pearl_tasks 两处）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetMeteorLootPrefab(prefab)` | `prefab: string` → `string, force:boolean 或 nil` | rock_moon 累加 `TUNING.MOONROCKSHELL_CHANCE`（0.35，tuning.lua:3156）后按"修正加成满/世界≥60 天/随机"判定升级；命中返回 `("rock_moon_shell", true)`；场上有第二颗同掉落时降级回 rock_moon 去重。[实现](../../../raw/current/scripts/components/worldmeteorshower.lua#L22-L41) |
| `GetRockMoonShellWaveOdds()` | → `number` | 累积未满 1 时返回修正表当前加成，供 meteorshower 波次展示（[调用](../../../raw/current/scripts/components/meteorshower.lua#L219)）。[实现](../../../raw/current/scripts/components/worldmeteorshower.lua#L14-L20) |
| `moonrockshell_chance_additionalodds` | `SourceModifierList`（additive） | 外部加成修正表，键名限定来源（如 `"celestial_altar"`）。[构造](../../../raw/current/scripts/components/worldmeteorshower.lua#L12) |
| `SpawnMeteorLoot(prefab)` | `prefab: string` → `entity` | 已标注 deprecated、仅为模组保留的旧入口。[实现](../../../raw/current/scripts/components/worldmeteorshower.lua#L43-L45) |
| `OnSave`／`OnLoad(data)` | 存读钩子 | 只持久化 `moonrockshell_chance`。[实现](../../../raw/current/scripts/components/worldmeteorshower.lua#L51-L60) |

## 典型接入

### 为流星掉落加月岩壳概率

- 前提与选择：世界必须装了本组件（forest 默认装，[安装](../../../raw/current/scripts/prefabs/forest.lua#L675)）；用 SetModifier 带 key 写入，重复调用同 key 会覆盖而非叠加。
- 生效与撤销：修正即时影响下一次 GetMeteorLootPrefab 判定；撤销用同 key 移除修正（来源方自己管理时机）。
- 原版依据：天体祭坛阶段给 `"celestial_altar"` 加成（[moon_altar](../../../raw/current/scripts/prefabs/moon_altar.lua#L181-L184)）；隐士好感度给 `"pearl_tasks"` 加成（[hermitcrab](../../../raw/current/scripts/prefabs/hermitcrab.lua#L1404-L1407)）。

```lua
AddPrefabPostInit("forest", function(inst)
    if not TheWorld.ismastersim then return end
    inst:DoTaskInTime(0, function()
        if inst.components.worldmeteorshower ~= nil then
            inst.components.worldmeteorshower.moonrockshell_chance_additionalodds
                :SetModifier(inst, 0.2, "my_mod_bonus")
        end
    end)
end)
```

- 调整与验证：加成为加法修正，与累积值一起参与"≥1 必掉"判定；60 天后世界固定掉壳（cycles 判定），修正只影响早期。

## 常见陷阱与限制

- 消费端在 shadowmeteor 掉落分配时调用（[调用点](../../../raw/current/scripts/prefabs/shadowmeteor.lua#L140-L141)），直接 SpawnPrefab("rock_moon") 不走本组件；想强制改掉落请复用 GetMeteorLootPrefab 的返回值对。
- 修正表不持久化：读档后 celestial_altar/pearl_tasks 由来源方重新注册；模组修正也需在载入钩子里重设。

## 相关链接

- [[reference/components/moonaltarlinktarget]]：同为 moon_altar.lua 挂载的天体祭坛组件。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 forest／shadowmeteor／meteorshower／moon_altar／hermitcrab 消费链、tuning 取值；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
