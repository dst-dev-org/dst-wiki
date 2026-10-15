---
title: "carnivaldecor"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "嘉年华装饰评分标记：把装饰物登记到附近 carnivaldecorranker 计分器并提供装饰值，供嘉年华活动统计。"
aliases: ["嘉年华装饰", "carnivaldecor"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/carnivaldecor.lua", "components/carnivaldecorranker.lua", "prefabs/carnivaldecor_eggride.lua", "prefabs/carnival_decor.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# carnivaldecor

## 快速判断

- 职责与适用：挂在嘉年华装饰物实体上，携带装饰值并自动注册到附近带 `carnivaldecor_ranker` 标签的计分器实体（[组件全文](../../../raw/current/scripts/components/carnivaldecor.lua#L22-L31)）。
- 优先使用：给装饰物 prefab 加组件；注册在构造后延迟 0 拍自动完成，无需手动调用（[注册](../../../raw/current/scripts/components/carnivaldecor.lua#L4-L12)）。
- 重要副作用：构造加 `carnivaldecor` 标签、延迟注册；组件或实体移除时按位置反查计分器注销（[移除](../../../raw/current/scripts/components/carnivaldecor.lua#L33-L40)）。
- 修改边界：注册仅在非世界生成期（`POPULATING` 为假）且装饰物不在平台上（`GetCurrentPlatform() == nil`）时执行；注册半径 `TUNING.CARNIVAL_DECOR_RANK_RANGE = 8`（[调参](../../../raw/current/scripts/tuning.lua#L5943)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetDecorValue()` | `-> number` | 返回 `self.value`，默认 1；计分器在登记时读取该值。[实现](../../../raw/current/scripts/components/carnivaldecor.lua#L42-L44) |
| `value` | `number，默认 1` | 可在宿主内直接赋值；计分器只在 AddDecor 登记时取值（[登记](../../../raw/current/scripts/components/carnivaldecorranker.lua#L48-L51)），改值后需重新登记才反映到计分器。 |

## 典型接入

### 让自定义装饰物参与嘉年华评分

- 前提与选择：装饰物主机端加本组件；附近需有 carnivaldecorranker 宿主（原版为嘉年华广场系实体）才会被登记。
- 生效与撤销：登记/注销都按实体位置做范围搜索（[注销搜索](../../../raw/current/scripts/components/carnivaldecor.lua#L14-L20)）；组件 `OnRemoveFromEntity` 同时摘标签并注销，`OnRemoveEntity` 仅注销。

```lua
AddPrefabPostInit("my_carnival_decor", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    local carnivaldecor = inst:AddComponent("carnivaldecor")
    carnivaldecor.value = 3
end)
```

- 调整与验证：`value` 在同一拍内赋值即可赶在延迟注册（`DoTaskInTime(0)`）之前生效；原版装饰宿主见 [carnivaldecor_eggride](../../../raw/current/scripts/prefabs/carnivaldecor_eggride.lua#L183) 与 [carnival_decor](../../../raw/current/scripts/prefabs/carnival_decor.lua#L87)。

## 常见陷阱与限制

- **世界生成期不注册**：`POPULATING` 为真时跳过延迟注册（[构造](../../../raw/current/scripts/components/carnivaldecor.lua#L26-L28)），生成出来的装饰靠存档/计分器自身的其他途径处理，不要期待生成瞬间即被登记。
- **平台上的装饰不登记**：`add_to_counters` 检查 `GetCurrentPlatform() == nil`（[注册](../../../raw/current/scripts/components/carnivaldecor.lua#L5-L11)），但 `remove_from_counters` 不查平台——把已登记装饰搬上船再删除时，注销搜索仍按地面位置执行。
- **登记值是快照**：计分器保存的是登记时的 `GetDecorValue()` 结果（[登记](../../../raw/current/scripts/components/carnivaldecorranker.lua#L48-L51)），运行中改 `value` 不会自动更新已登记条目。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、carnivaldecorranker 的 AddDecor/RemoveDecor、四个原版装饰宿主挂接、`CARNIVAL_DECOR_RANK_RANGE` 调参值。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
