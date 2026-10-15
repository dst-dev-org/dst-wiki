---
title: "sandstorms"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "沙尘暴世界状态：按季节与湿度开关沙尘暴，结合区域拓扑与绿洲算出实体的沙尘暴强度。"
aliases: ["沙尘暴管理", "sandstorms", "沙暴等级"]
tags: [component, data]
dst_build: "steam-buildid:26230496"
sources: ["components/sandstorms.lua", "prefabs/forest.lua", "components/stormwatcher.lua", "components/sandstormwatcher.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sandstorms

## 快速判断

- 职责与适用：挂在 TheWorld（[安装](../../../raw/current/scripts/prefabs/forest.lua#L674)），仅 master sim（[断言](../../../raw/current/scripts/components/sandstorms.lua#L7)）；夏天且无降水时沙尘暴开启（[条件](../../../raw/current/scripts/components/sandstorms.lua#L26-L28)），等级查询全走 `areaaware` 区域拓扑。
- 优先使用：玩家风暴表现已由 [[reference/components/stormwatcher]] 消费；自定义 UI 或逻辑直接读 `GetSandstormLevel`/`IsInSandstorm`。
- 重要副作用：开关时推 `ms_stormchanged`（`stormtype=STORM_TYPES.SANDSTORM`，[推送](../../../raw/current/scripts/components/sandstorms.lua#L30-L35)）。
- 修改边界：查询端要求实体有 [[reference/components/areaaware]]，否则等级恒 0（[待办](../../../raw/current/scripts/components/sandstorms.lua#L141)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsSandstormActive()` | → `bool` | 沙尘暴是否进行中（[实现](../../../raw/current/scripts/components/sandstorms.lua#L145-L147)）。 |
| `IsInSandstorm(ent)` | 实体，需 areaaware | 是否正处带 `sandstorm` 标签的区域（[实现](../../../raw/current/scripts/components/sandstorms.lua#L126-L130)）。 |
| `GetSandstormLevel(ent)` | → `0..1` | 沙尘暴深度减绿洲缓和后的强度（[实现](../../../raw/current/scripts/components/sandstorms.lua#L132-L143)）。 |
| `CalcSandstormLevel(ent)` | → `0..1` | 纯拓扑深度：到无 `sandstorm` 标签邻区的边距离归一化（[计算](../../../raw/current/scripts/components/sandstorms.lua#L97-L102)）。 |
| `CalcOasisLevel(ent)` / `IsInOasis(ent)` | → `0..1` / `bool` | 绿洲缓和度，基于注册的绿洲实体的 oasis 组件（[计算](../../../raw/current/scripts/components/sandstorms.lua#L113-L124)）。 |
| `RetrofitCheckIfWorldContainsOasis()` | → `bool` | 世界改造检查（[实现](../../../raw/current/scripts/components/sandstorms.lua#L149-L151)）。 |
| `ms_registeroasis`（事件） | 监听 | 绿洲实体登记与移除跟踪（[登记](../../../raw/current/scripts/components/sandstorms.lua#L55-L60)）。 |

## 典型接入

### 读取实体的沙尘暴强度

- 前提与选择：服务器侧；玩家自带 areaaware，普通物品需自行装组件才有非零结果。
- 生效与撤销：纯查询无副作用；开关事件可监听 `ms_stormchanged`。
- 原版依据：stormwatcher 以此换算 7 档 stormlevel（[换算](../../../raw/current/scripts/components/stormwatcher.lua#L112)）；沙尘暴观测组件同样读开关（[观测](../../../raw/current/scripts/components/sandstormwatcher.lua#L6-L12)）。

```lua
-- 服务器：查询并监听
local mgr = TheWorld.components.sandstorms
if mgr ~= nil then
    local lvl = mgr:GetSandstormLevel(player)
end

TheWorld:ListenForEvent("ms_stormchanged", function(world, data)
    if data.stormtype == STORM_TYPES.SANDSTORM then
        -- data.setting: 开/关
    end
end)
```

- 调整与验证：归一化深度以 `TUNING.SANDSTORM_FULLY_ENTERED_DEPTH`（20）为单位（[常量](../../../raw/current/scripts/tuning.lua#L3330)）；深入区域中心约 1。

## 常见陷阱与限制

- 拓扑深度只对"当前处于带 `sandstorm` 标签节点"的实体有意义：`CalcTaggedNodeDepthSq` 的调用前提由调用方保证（[注释](../../../raw/current/scripts/components/sandstorms.lua#L75)），区域外实体直接走 0 分支。
- 绿洲缓和依赖原版绿洲实体（[[reference/components/oasis]] 组件）注册；自定义遮蔽物不注册 `ms_registeroasis` 就不参与缓和。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、forest 安装点与 stormwatcher/sandstormwatcher 消费链、相关 tuning；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
