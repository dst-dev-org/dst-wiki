---
title: "desolationspawner：荒芜区树木再播种"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级再播种器：按世界生成密度记录各区域应产苗木，累计再长时控间后在该区域随机点补种树苗/蘑菇树，季节与月相调制速度。"
aliases: [荒芜再播种, 树木再生长, desolationspawner, 树木补种]
tags: [component, worldgen, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/desolationspawner.lua", "prefabs/forest.lua", "prefabs/cave.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# desolationspawner（荒芜区树木再播种）

## 快速判断

- 职责与适用：`forest` 与 `cave` 世界都挂载的服务端组件（[forest.lua:667](../../../raw/current/scripts/prefabs/forest.lua#L667)、[cave.lua:539](../../../raw/current/scripts/prefabs/cave.lua#L539）。按世界生成的 `densities` 数据把"某区域曾有哪些树"登记成区域表，每 11 秒累计 `REGROWTH_TIME_MULTIPLIER × timemult`，到点后在世界生成时的区域内随机点补种对应苗木（[再生长](../../../raw/current/scripts/components/desolationspawner.lua#L228-L245)）。
- 优先使用：要新增可再播种的类型 → 调用唯一的公开接口 `SetSpawningForType`；原版在构造时注册了 13 类（[注册](../../../raw/current/scripts/components/desolationspawner.lua#L170-L219)）。注：与 [[reference/components/plantregrowth]] 分工——本组件按"世界生成密度"补种，后者按存活的源实体再生。
- 重要副作用：选点检查禁止在基地（20 单位内 structure/wall）、已有同类（50 单位内按 searchtags）与不可种植点生成；每个更新周期最多补种一株（[性能](../../../raw/current/scripts/components/desolationspawner.lua#L240-L242)）。
- 修改边界：构造即断言 [mastersim](../../../raw/current/scripts/components/desolationspawner.lua#L7)；区域数据依赖 `TheWorld.generated`，世界未生成完时跳过（[就绪检查](../../../raw/current/scripts/components/desolationspawner.lua#L112-L115)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetSpawningForType(prefab, product, regrowtime, searchtags, timemult)` | prefab 源名、产物 prefab 名、秒数、标签表、`timemult(): number` | 登记替换数据并立即按 `densities` 填区域表；`timemult` 可传函数按季节/月相调制（0 即暂停该类）（[实现](../../../raw/current/scripts/components/desolationspawner.lua#L154-L158)）。 |
| `LongUpdate(dt)` | 秒 | 累计各 prefab 内部时间→找到到点区域→`DoRegrowth` 一次即返回；周期任务同样调它（[实现](../../../raw/current/scripts/components/desolationspawner.lua#L228-L245)、[周期](../../../raw/current/scripts/components/desolationspawner.lua#L168)）。 |
| `OnSave/OnLoad` | 存读钩子 | 存每区域每 prefab 的 `density` 与剩余 `regrowtime`；读回时重新加内部时间偏移（[存读](../../../raw/current/scripts/components/desolationspawner.lua#L251-L281)）。 |
| `GetDebugString()` | debug 文本 | 汇总每类最近的可再长时间（[debug](../../../raw/current/scripts/components/desolationspawner.lua#L287-L301)）。 |

选点验证 `TestForRegrow`（[实现](../../../raw/current/scripts/components/desolationspawner.lua#L47-L75)）：3 单位内无任何实体（过密）→ 20 单位内无 structure/wall（避基地）→ 50 单位内无 searchtags 命中（已播种）→ `CanPlantAtPoint`＋`CanPlacePrefabFilteredAtPoint` 且不在路上；`tree_rock_sapling` 跳过种植检查（[豁免表](../../../raw/current/scripts/components/desolationspawner.lua#L21-L23)）。

## 典型接入

### 登记自定义可再生类型

- 前提与选择：服务端世界实体、世界生成完成后；必须在 `TheWorld.generated.densities` 里存在同名源 prefab 的密度记录，否则没有区域可种（[密度读取](../../../raw/current/scripts/components/desolationspawner.lua#L117-L136)）。
- 生效与撤销：注册后内部时间从 0 起，首个周期即可能补种；无移除接口，只能清空组件级数据（不提供公开撤销）。

```lua
-- 服务端，AddPrefabPostInit("world", ...) 中
AddPrefabPostInit("world", function(inst)
    if not TheWorld.ismastersim then return end
    inst:DoTaskInTime(0, function()
        if inst.components.desolationspawner then
            inst.components.desolationspawner:SetSpawningForType(
                "evergreen",            -- 世界生成时的源 prefab（须有密度记录）
                "pinecone_sapling",     -- 补种产物
                TUNING.EVERGREEN_REGROWTH.DESOLATION_RESPAWN_TIME,
                { "evergreen" },        -- 播种搜索标签
                nil                     -- timemult，nil 恒为 1
            )
        end
    end)
end)
```

- 调整与验证：`regrowtime` 为秒；`timemult` 返回 0 时该类冻结（对照原版冬季常青树乘 0，[注册](../../../raw/current/scripts/components/desolationspawner.lua#L170-L172)）。

## 常见陷阱与限制

- **每周期只补一株**：`LongUpdate` 命中后立即 return，世界很大时荒区恢复是慢过程（[单次限制](../../../raw/current/scripts/components/desolationspawner.lua#L240-L242)）。
- **依赖世界生成数据**：`densities` 没记录的 prefab 不会生成任何区域条目；模组后加的树源世界里没有旧密度就不被补种。
- **月相只影响月树**：`moon_tree` 的 timemult 按月相 0~2 倍调制（[月树表](../../../raw/current/scripts/components/desolationspawner.lua#L192-L202)），其余类型按季节。
- **`tree_rock` 系为旧档保留**：三条 `tree_rock*` 注册注明 deprecated，只为旧存档与改造保留（[注释](../../../raw/current/scripts/components/desolationspawner.lua#L208-L219)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/desolationspawner.lua` 全部 308 行、forest/cave 两处挂载行与周边世界管理块。未覆盖：`CanPlacePrefabFilteredAtPoint` 过滤表、世界生成 densities 的写入端。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
