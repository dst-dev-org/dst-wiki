---
title: "carnivalevent"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "嘉年华世界活动管理：在活动开启时生成并保活嘉年华主持人，登记广场供召唤与活动结算。"
aliases: ["嘉年华活动", "carnivalevent", "猪王嘉年华"]
tags: [component, event]
dst_build: "steam-buildid:26230496"
sources: ["components/carnivalevent.lua", "prefabs/forest.lua", "prefabs/carnival_plaza.lua", "prefabs/world.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# carnivalevent

## 快速判断

- 职责与适用：挂在地表世界 TheWorld（[安装](../../../raw/current/scripts/prefabs/forest.lua#L693)），仅 master sim（[断言](../../../raw/current/scripts/components/carnivalevent.lua#L10)）；负责嘉年华主持人实体的生成、复活与广场注册表。
- 优先使用：查询/召唤经 `GetRandomPlaza` + `SummonHost`；自定义广场在建造时 `RegisterPlaza`、移除时 `UnregisterPlaza`。
- 重要副作用：主持人被移除会立即重生（[复活](../../../raw/current/scripts/components/carnivalevent.lua#L50-L53)）；注册广场时推 `ms_carnivalplazabuilt`（[事件](../../../raw/current/scripts/components/carnivalevent.lua#L89)）。
- 修改边界：`SpawnCarnivalHost` 只在 `SPECIAL_EVENTS.CARNIVAL` 活动开启时执行（[门控](../../../raw/current/scripts/components/carnivalevent.lua#L56)），活动关闭时本组件空转。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `RegisterPlaza(plaza)` / `UnregisterPlaza(plaza)` | 广场实体 | 维护广场表并广播建造事件（[实现](../../../raw/current/scripts/components/carnivalevent.lua#L87-L96)）。 |
| `DoesAnyPlazaExist()` / `GetRandomPlaza()` | → `bool` / 广场+序号 | 查询与随机取样（[实现](../../../raw/current/scripts/components/carnivalevent.lua#L98-L104)）。 |
| `SummonHost(plaza)` | → `bool` | 召唤主持人到广场；无主持人返回 false（[实现](../../../raw/current/scripts/components/carnivalevent.lua#L106-L112)）。 |
| `OnPostInit()` | 世界 PostInit 钩子 | 读档后主持人丢失时补生成（[调用](../../../raw/current/scripts/prefabs/world.lua#L343-L344)）。 |
| `OnSave` / `LoadPostPass` | 存读钩子 | 存主持人 GUID 并在读档后重挂（[存读](../../../raw/current/scripts/components/carnivalevent.lua#L124-L146)）。 |

## 典型接入

### 自定义广场与召唤入口

- 前提与选择：服务器侧；广场实体需实现 `SummonedToPlaza` 被主持人调用（[转发](../../../raw/current/scripts/components/carnivalevent.lua#L111)）。
- 生效与撤销：注册表按实体键维护，主持人存活状态由组件自恢复。
- 原版依据：原版广场在建造/移除时注册注销（[注册](../../../raw/current/scripts/prefabs/carnival_plaza.lua#L407-L410)、[注销](../../../raw/current/scripts/prefabs/carnival_plaza.lua#L173-L174)）；主持人落点用出生点附近可行走位置加 [[reference/components/knownlocations]] 记 home（[落点](../../../raw/current/scripts/components/carnivalevent.lua#L64-L67)）。

```lua
-- 服务器：取一个广场并召唤主持人
local ce = TheWorld.components.carnivalevent
if ce ~= nil then
    local plaza = ce:GetRandomPlaza()
    if plaza ~= nil then
        ce:SummonHost(plaza)
    end
end
```

- 调整与验证：主持人生成位置在出生点 12（退 6）单位内找可行走点（[常量](../../../raw/current/scripts/components/carnivalevent.lua#L16-L17)）。

## 常见陷阱与限制

- `OnLoad` 为空壳，主持人完全靠 `LoadPostPass` 的 GUID 引用恢复（[实现](../../../raw/current/scripts/components/carnivalevent.lua#L138-L146)）；跨天存档引用失效时走 `OnPostInit` 重生成。
- 活动门控在 `SpawnCarnivalHost` 内部：即使组件存在，活动关闭时 `SummonHost` 之外的生成路径也不可用。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、forest 安装点、carnival_plaza 消费链与 world PostInit 钩子；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
