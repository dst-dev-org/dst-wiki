---
title: "winonateleportpadmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "温妮传送垫注册表：TheWorld 上收集已建好的传送垫实体，供温妮技能远程传送挑选落点。"
aliases: ["传送垫注册表", "winonateleportpadmanager", "温妮传送垫管理"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/winonateleportpadmanager.lua", "prefabs/world.lua", "prefabs/winona_teleport_pad.lua", "components/remoteteleporter.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# winonateleportpadmanager

## 快速判断

- 职责与适用：挂在 TheWorld 上、仅 mastersim（构造断言，[断言](../../../raw/current/scripts/components/winonateleportpadmanager.lua#L3)、[宿主](../../../raw/current/scripts/prefabs/world.lua#L592)）；维护 `{ [传送垫实体] = { onremove = fn } }` 注册表（[注册表](../../../raw/current/scripts/components/winonateleportpadmanager.lua#L6)）。
- 优先使用：传送垫 prefab 主动推 `ms_registerwinonateleportpad` 自报（[自报](../../../raw/current/scripts/prefabs/winona_teleport_pad.lua#L810)）；读取侧用 `GetAllWinonaTeleportPads()` 拿全部垫子（[读取](../../../raw/current/scripts/components/winonateleportpadmanager.lua#L18-L20)）。
- 重要副作用：注册延迟到垫子的 `onbuilt`/`entitywake`/`entitysleep` 任一触发才入表，且三监听一次性移除（[注册流程](../../../raw/current/scripts/components/winonateleportpadmanager.lua#L28-L49)）；垫子 `onremove` 自动出表（[清理](../../../raw/current/scripts/components/winonateleportpadmanager.lua#L38-L44)）。
- 修改边界：`OnRemoveFromEntity` 移除桥接监听并逐垫卸载 onremove 监听（[清理](../../../raw/current/scripts/components/winonateleportpadmanager.lua#L9-L14)）；注册表仅在主机端存在。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetAllWinonaTeleportPads()` | `-> table（键为传送垫实体）` | 返回注册表本体（非拷贝）；消费方遍历键即可。[实现](../../../raw/current/scripts/components/winonateleportpadmanager.lua#L18-L20) |
| `OnRegisterWinonaTeleportPad(pad)` | `pad: entity -> 无` | 收到注册事件后挂 onbuilt/entitywake/entitysleep 三监听，任一触发即入表；表项只带 onremove 清理函数。[实现](../../../raw/current/scripts/components/winonateleportpadmanager.lua#L28-L49) |
| `winonateleportpads` | `table` | 注册表字段；请勿直接增删，走事件与出表机制。[字段](../../../raw/current/scripts/components/winonateleportpadmanager.lua#L6) |

## 典型接入

### 挑选传送落点（原版参照）

- 前提与选择：消费方 [[reference/components/remoteteleporter]]（温妮技能传送器）在 Teleport 时读注册表，按镜头内最远/镜头外最近挑点并排除附近半径（[消费](../../../raw/current/scripts/components/remoteteleporter.lua#L91-L92)）。
- 生效与撤销：垫子建造完成且实体被唤醒过才可作落点；删除垫子即时出表，无滞后。

```lua
-- 服务端：遍历当前全部可用传送垫
local manager = TheWorld.components.winonateleportpadmanager
if manager ~= nil then
    for pad in pairs(manager:GetAllWinonaTeleportPads()) do
        local x, y, z = pad.Transform:GetWorldPosition()
    end
end
```

- 调整与验证：注册表键即实体，可直接读 Transform；不需要也不应该手工往表里塞实体。

## 常见陷阱与限制

- **注册不是即时的**：垫子从"生成"到"入表"要等 onbuilt/entitywake/entitysleep 之一（[注册流程](../../../raw/current/scripts/components/winonateleportpadmanager.lua#L46-L48)）；刚建造或从未唤醒的垫子不在落点候选里。
- **仅主机端**：客户端没有该组件与注册表，客户端 UI 的落点数据需走其他同步途径。
- **自定义垫子要主动自报**：组件只监听 `ms_registerwinonateleportpad` 事件，不改名就不识别其他 prefab；自定义落点需推同名事件。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、world.lua 宿主挂载、winona_teleport_pad 注册事件推送、remoteteleporter 消费链路。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。

相关：[[reference/components/remoteteleporter]]
