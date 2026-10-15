---
title: "corpsepersistmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级尸体留驻管理器：注册来源函数决定哪些尸体不腐烂消失，并执行全图 200 具尸体上限。"
aliases: ["尸体留驻", "corpsepersistmanager", "尸体持久化"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/corpsepersistmanager.lua", "prefabs/world.lua", "componentutil.lua", "components/mutatedbuzzardmanager.lua", "prefabs/corpses.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# corpsepersistmanager

## 快速判断

- 职责与适用：挂在 TheWorld（仅 master sim，[断言](../../../raw/current/scripts/prefabs/world.lua#L594)所在段）上，收来源函数按周期给尸体打/撤留驻标记，控制哪些尸体可跨存档保留。
- 优先使用：想让某类生物死后尸体保留（供变异、复活等），注册 `AddPersistSourceFn` 即可，无需改尸体 prefab。
- 重要副作用：全图尸体硬上限 200 具（[常量](../../../raw/current/scripts/components/corpsepersistmanager.lua#L16)），超限时溢出部分被撤标记强制可清理。
- 修改边界：不保存不读档（无 OnSave/OnLoad）；`LongUpdate` 明确未支持（[注释](../../../raw/current/scripts/components/corpsepersistmanager.lua#L133-L134)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddPersistSourceFn(key, fn)` | `fn(corpse) → bool`，仅服务器 | 注册留驻来源；注册即启动周期更新（[实现](../../../raw/current/scripts/components/corpsepersistmanager.lua#L60-L66)）。 |
| `RemovePersistSourceFn(key)` | 无返回值 | 注销来源并撤掉所有尸体上该 key 的标记（[实现](../../../raw/current/scripts/components/corpsepersistmanager.lua#L68-L78)）。 |
| `ShouldRetainCreatureAsCorpse(creature)` | → `bool` | 上限未满且任一来源函数认可时为真；死亡判定入口调用（[消费](../../../raw/current/scripts/componentutil.lua#L1554)）。 |
| `IsMaxReached()` / `AnyCorpseExists()` | → `bool` | 读当前尸体计数状态（[实现](../../../raw/current/scripts/components/corpsepersistmanager.lua#L95-L101)）。 |
| `OnUpdate(dt)` | 周期回调 | 每 10–15 秒轮询一次：给认可 corpse `SetPersistSource(key, true)`、否则 `RemovePersistSource`（[轮询](../../../raw/current/scripts/components/corpsepersistmanager.lua#L103-L132)）。 |

## 典型接入

### 让特定尸体保留不清理

- 前提与选择：服务器侧在 world prefab 就绪后注册；原版使用者是变异秃鹫管理器（[注册](../../../raw/current/scripts/components/mutatedbuzzardmanager.lua#L406-L408)）。
- 生效与撤销：来源函数按周期重复求值，可随世界状态翻转结论；注销时该来源的所有标记立即撤销。
- 原版依据：尸体实体在 master postinit 时推送 `ms_registercorpse` 登记进列表（[登记](../../../raw/current/scripts/prefabs/corpses.lua#L809)）；死亡侧 `CanEntityBecomeCorpse` 先问本组件（[判定](../../../raw/current/scripts/componentutil.lua#L1547-L1554)）。

```lua
-- modmain.lua：让 boss 尸体不进入常规清理（服务器）
AddPrefabPostInit("world", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst:DoTaskInTime(0, function()
        local mgr = inst.components.corpsepersistmanager
        if mgr ~= nil then
            mgr:AddPersistSourceFn("mymod_keepboss", function(corpse)
                return corpse:HasTag("epic")
            end)
        end
    end)
end)
```

- 调整与验证：`key` 决定撤销粒度，不同系统用不同 key；周期常数 `CHECK_CORPSE_TIME_BASE/VAR` 为 10/5 秒（[常量](../../../raw/current/scripts/components/corpsepersistmanager.lua#L13-L14)）。

## 常见陷阱与限制

- 200 具上限是世界硬上限：`IsMaxReached()` 时 `ShouldRetainCreatureAsCorpse` 直接返回 false，且 `OnUpdate` 会主动撤掉溢出尸体的全部留驻标记（[溢出处理](../../../raw/current/scripts/components/corpsepersistmanager.lua#L119-L126)）。
- 只对走 `corpses` prefab 流程的尸体生效：标记实现在尸体侧的 `SetPersistSource`（[实现](../../../raw/current/scripts/prefabs/corpses.lua#L424)），自定义尸体需自己接同一事件链。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、world 安装点、componentutil 死亡判定、mutatedbuzzardmanager 注册链与 corpses 登记实现；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
