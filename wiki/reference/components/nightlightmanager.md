---
title: "nightlightmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级夜灯注册表：登记建成的噩梦夜灯及其地图节点 tag，供查理复活点等服务按过滤条件找最近夜灯。"
aliases: ["夜灯管理器", "nightlightmanager", "查理复活点"]
tags: [component]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/nightlightmanager.lua", "prefabs/world.lua", "prefabs/nightlight.lua", "simutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# nightlightmanager

## 快速判断

- 职责与适用：挂在 `TheWorld` 上的服务端注册表，收集已建成的 `nightlight`（噩梦夜灯）实体并缓存其所在地图视觉节点的 tag，供"在夜灯旁复活"这类查询使用。[构造](../../../raw/current/scripts/components/nightlightmanager.lua#L2-L8)
- 优先使用：查询走 `GetNightLightsWithFilter(filterfn, ...)` + `FindClosestNightLightFromListToInst(list, inst, filterfn)`，过滤器直接用组件自带的三个 `Filter_*`。[查询](../../../raw/current/scripts/components/nightlightmanager.lua#L42-L66)
- 重要副作用：夜灯实体 `onremove` 时自动出册；注册延迟到 `onbuilt`／`entitywake`／`entitysleep` 首个事件。[注册](../../../raw/current/scripts/components/nightlightmanager.lua#L96-L120)
- 修改边界：构造断言 mastersim；在船（可走平台）上的夜灯永不登记；无存读钩子，依赖实体唤醒事件重建。[平台守卫](../../../raw/current/scripts/components/nightlightmanager.lua#L106-L115)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `ms_registernightlight`（输入事件） | 注册实体 | nightlight prefab 构建完成时对世界推送；组件桥接到 `OnRegisterNightLight`。[监听](../../../raw/current/scripts/components/nightlightmanager.lua#L7)、[推送](../../../raw/current/scripts/prefabs/nightlight.lua#L158) |
| `GetNightLightsWithFilter(filterfn, ...)` | 实体数组 | 对注册表逐项跑 `filterfn(self, nightlight, data, ...)` 收集命中项；`data` 含坐标与 `node_tags`（`table.invert(node.tags)`）。[L42-L50](../../../raw/current/scripts/components/nightlightmanager.lua#L42-L50)、[缓存](../../../raw/current/scripts/components/nightlightmanager.lua#L70-L90) |
| `Filter_OnlyInTags`／`Filter_OnlyOutTags`／`Filter_InTagsAndOutTags` | 过滤器函数 | 按 `nightlightdata.node_tags` 判"在任一 tag／不在任何 tag／进且出"。[L30-L40](../../../raw/current/scripts/components/nightlightmanager.lua#L30-L40) |
| `FindClosestNightLightFromListToInst(list, inst, filterfn)` | 实体或 nil | 列表内按 `GetDistanceSqToInst` 找最近；可再叠加自定义 filterfn。[L52-L66](../../../raw/current/scripts/components/nightlightmanager.lua#L52-L66) |
| `UpdateNightLightPosition(nightlight)` | 无返回 | 刷新缓存坐标与节点 tag；节点未变时短路，不重读拓扑。[L70-L90](../../../raw/current/scripts/components/nightlightmanager.lua#L70-L90) |

## 典型接入

### 原版消费者：查理复活点

- simutil 的 `FindCharlieRezSpotFor` 用 `Filter_OnlyOutTags` 排除 `lunacyarea` 节点上的夜灯，再取距玩家最近且允许传送的夜灯作为复活点；找不到才回退出生点。[消费](../../../raw/current/scripts/simutil.lua#L378-L387)
- 挂载：`world.lua` master 段统一 `AddComponent("nightlightmanager")`，全部世界共享一份注册表。[挂载](../../../raw/current/scripts/prefabs/world.lua#L591)
- 注册时机：夜灯在 prefab 末尾推 `ms_registernightlight`；实际入册要等 `onbuilt`（新造）或 `entitywake`/`entitysleep`（读档唤醒），且当前不在船上。[夜灯侧](../../../raw/current/scripts/prefabs/nightlight.lua#L156-L158)、[入册条件](../../../raw/current/scripts/components/nightlightmanager.lua#L96-L119)

### 模组：按区域筛选夜灯

- 前提与选择：服务端查询；节点 tag 来自世界拓扑 `node.tags`，不是实体 tag，过滤条件要按拓扑 tag（如 `lunacyarea`）来写。
- 生效与撤销：查询即时返回，不产生状态；自定义过滤器按 `filterfn(self, nightlight, data, ...)` 签名实现即可。

```lua
-- 服务端：找玩家附近、不在 lunacyarea 的最近夜灯
local manager = GLOBAL.TheWorld and GLOBAL.TheWorld.components.nightlightmanager
if manager ~= nil then
    local lights = manager:GetNightLightsWithFilter(manager.Filter_OnlyOutTags, { "lunacyarea" })
    local light = manager:FindClosestNightLightFromListToInst(lights, player)
end
```

- 调整与验证：示例仅语法检查未运行；返回的是实体引用，位置读取用其 Transform。

## 常见陷阱与限制

- 只登记 `GetCurrentPlatform() == nil` 的夜灯：船上的噩梦夜灯不参与查询，也不在后来"从船移到陆地"时补登（无重扫逻辑）。[L106](../../../raw/current/scripts/components/nightlightmanager.lua#L106)
- 注册只进不出（除 onremove）：夜灯被锤毁走 `onremove` 会出册，但若实体长期 limbo（不触发 wake/sleep 边界）缓存坐标可能过期；查询前无自动校验。[出册](../../../raw/current/scripts/components/nightlightmanager.lua#L107-L113)
- `node_tags` 是节点 tag 的 invert 表：判断用 `nightlightdata.node_tags[tag] ~= nil`，与实体 tag 系统完全独立，别混用。[L18-L28](../../../raw/current/scripts/components/nightlightmanager.lua#L18-L28)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖 nightlightmanager.lua 全文（过滤器、查询、注册与缓存）；核对 world.lua 挂载、nightlight.lua 注册推送、simutil.lua 消费段；未覆盖拓扑节点 tag 的生成细节。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 与示例 Lua 语法检查通过（见批记录）。
