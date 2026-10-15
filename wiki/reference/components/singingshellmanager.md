---
title: "singingshellmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "动态挂在 TheWorld 上的海螺活动管理器：有海螺可被触发时开启全体玩家扫描，没海螺时自移除。"
aliases: ["海螺管理器", "海螺扫描开关", "singingshellmanager"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/singingshellmanager.lua", "prefabs/singingshell.lua", "components/singingshelltrigger.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# singingshellmanager

## 快速判断

- 职责与适用：服务器（构造断言 mastersim，[断言](../../../raw/current/scripts/components/singingshellmanager.lua#L11)）侧开关：任一海螺进入可触发状态时由海螺 prefab `TheWorld:AddComponent` 挂上（[注册](../../../raw/current/scripts/prefabs/singingshell.lua#L88-L94)），全部失活时自移除（[自移除](../../../raw/current/scripts/components/singingshellmanager.lua#L49-L54)）。
- 优先使用：想查询"当前是否有海螺可唱"直接判 `TheWorld.components.singingshellmanager ~= nil`。
- 重要副作用：挂上即对 `AllPlayers` 调 `singingshelltrigger:StartUpdating()`，并监听 `ms_playerjoined` 补开新玩家扫描（[开启](../../../raw/current/scripts/components/singingshellmanager.lua#L27-L39)、[补开](../../../raw/current/scripts/components/singingshellmanager.lua#L56-L58)）。
- 修改边界：无存档无网络变量，纯会话内开关；`OnRemoveFromEntity` 统一 `StopUpdatingPlayers()`（[清理](../../../raw/current/scripts/components/singingshellmanager.lua#L73-L75)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `active_shells` | 表（键为 shell 实体） | 当前活动海螺集合；空即触发自移除。 |
| `players_should_run_update` | `boolean`，恒为 `false` | 字段存在但从未置真（[声明](../../../raw/current/scripts/components/singingshellmanager.lua#L25)），扫描开关实际由 trigger 组件的 updating 状态承担。 |
| `RememberActiveShell(shell)` | 参数实体 | 登记活动海螺，海螺掉落/`OnEntityWake` 时调用（[调用点](../../../raw/current/scripts/prefabs/singingshell.lua#L189-L221)）。 |
| `ForgetActiveShell(shell)` | 参数实体 | 注销海螺；最后一个被注销时 `RemoveComponent("singingshellmanager")`。 |

## 典型接入

### 查询/联动海螺可唱状态

- 前提与选择：组件由原版海螺自动增删，模组不要手工 AddComponent；只在需要时读取存在性。
- 生效与撤销：海螺入包（`SetOnPutInInventoryFn`）、`onremove`、`OnEntitySleep` 都会注销（[钩子](../../../raw/current/scripts/prefabs/singingshell.lua#L189-L221)）。
- 原版依据：每玩家 `singingshelltrigger` 构造时也检查本组件决定是否立即开扫（[消费](../../../raw/current/scripts/components/singingshelltrigger.lua#L26-L28)）。

```lua
-- 服务器侧查询当前是否有可触发海螺
if TheWorld.components.singingshellmanager ~= nil then
    local count = 0
    for _ in pairs(TheWorld.components.singingshellmanager.active_shells) do
        count = count + 1
    end
end
```

- 调整与验证：`active_shells` 只在服务器有效；客户端判定请用海螺实体的相关 tag。

## 常见陷阱与限制

- 组件监听 `singingshell_wake`/`singingshell_sleep` 两个事件（[监听](../../../raw/current/scripts/components/singingshellmanager.lua#L65-L66)），但全库 grep 无任何推送者：实际登记全部走 `RememberActiveShell`/`ForgetActiveShell` 直调，这两个监听目前是死钩子，仿写时别指望推事件。
- 自移除发生在事件/直调路径内，若在遍历 `active_shells` 过程中注销最后一个海螺会立即失效，读侧注意判空。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、singingshell 注册/注销钩子与 singingshelltrigger 消费点；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。

相关：[singingshelltrigger](singingshelltrigger.md)
