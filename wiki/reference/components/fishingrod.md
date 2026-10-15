---
title: fishingrod
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "池塘钓竿的服务端钓鱼状态机：取目标、咬钩计时、上钩与收杆、捕获及中断清理。"
aliases: ["钓竿", "鱼竿", "池塘钓鱼", "钓鱼", "上钩", "收杆"]
tags: [component, item, equipment, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/fishingrod.lua", "components/fishingrod_replica.lua", "components/fishable.lua", "prefabs/fishingrod.lua", "prefabs/pond.lua", "prefabs/oasislake.lua", "actions.lua", "componentactions.lua", "components/playercontroller.lua", "stategraphs/SGwilson.lua", "tuning.lua", "entityscript.lua", "entityreplica.lua", "class.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# fishingrod

## 快速判断

- 职责与适用：挂在可装备钓竿物品上的服务端组件，保存 `target`／`fisherman` 与咬钩、上钩、已捕获三个状态，驱动池塘钓鱼；动作选择与表现由 SGwilson 的 `fishing*` 状态和 `FISH`／`REEL` 动作完成。[fishingrod.lua:13](../../../raw/current/scripts/components/fishingrod.lua#L13)、[SGwilson.lua:6233](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6233
- 优先使用：`FISH` 动作调用 `StartFishing(target, fisherman)`；`fishing` 状态进入时 `WaitForFish()`；`REEL` 按 `HasHookedFish`／`FishIsBiting` 分派 `Reel()`／`Hook()`／`StopFishing()`；`catchfish` 状态在时间线调用 `Collect()`。[actions.lua:1888-1892](../../../raw/current/scripts/actions.lua#L1888-L1892)、[SGwilson.lua:6247](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6247)、[SGwilson.lua:6353](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6353
- 重要副作用：`target`／`hookedfish`／`caughtfish` 通过属性 setter 写客户端 replica 网络变量；`fishingnibble`、`fishingstrain`、`fishingcatch`、`fishingcollect`、`fishingcancel`、`fishingloserod` 分别推给钓竿和／或钓鱼者。[fishingrod.lua:1](../../../raw/current/scripts/components/fishingrod.lua#L1)、[:36](../../../raw/current/scripts/components/fishingrod.lua#L36)、[fishingrod_replica.lua:4](../../../raw/current/scripts/components/fishingrod_replica.lua#L4
- 修改边界：服务端组件；不实现 `OnSave`／`OnLoad`／`OnRemoveFromEntity`，钓鱼中存档或移除组件都不保证复位；客户端只读 replica，不应在客户端改写这些字段。[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `target` / `fisherman` | entity / entity；初始 `nil` | `IsFishing()` 只看二者非空；`StartFishing` 要求目标有 `components.fishable`，并先 `StopFishing`。[fishingrod.lua:87](../../../raw/current/scripts/components/fishingrod.lua#L87)、[:103](../../../raw/current/scripts/components/fishingrod.lua#L103) |
| `hookedfish` / `caughtfish` | entity / entity；初始 `nil` | setter 只同步 replica 布尔；查询条件见下两行。[fishingrod.lua:5](../../../raw/current/scripts/components/fishingrod.lua#L5)、[:9](../../../raw/current/scripts/components/fishingrod.lua#L9) |
| `minwaittime` / `maxwaittime` | number 秒；构造默认 `0`/`10`，原版 prefab 设 `4`/`40` | `WaitForFish` 用 `min + (1-fishleft/maxfish)*(max-min)`：鱼量满最短、鱼少更长。[fishingrod.lua:23](../../../raw/current/scripts/components/fishingrod.lua#L23)、[fishingrod.lua:112](../../../raw/current/scripts/components/fishingrod.lua#L112)、[prefabs/fishingrod.lua:74](../../../raw/current/scripts/prefabs/fishingrod.lua#L74 |
| `minstraintime` / `maxstraintime` | number 秒；构造默认 `0`/`6`，prefab 设 `0`/`5` | `Hook` 按钓竿耐久百分比插值脱杆超时。[fishingrod.lua:25](../../../raw/current/scripts/components/fishingrod.lua#L25)、[fishingrod.lua:149](../../../raw/current/scripts/components/fishingrod.lua#L149)、[prefabs/fishingrod.lua:75](../../../raw/current/scripts/prefabs/fishingrod.lua#L75 |
| `SetWaitTimes(min, max)` / `SetStrainTimes(min, max)` | number 秒；无返回值 | 只影响之后的 `WaitForFish`／`Hook` 计算，不重置进行中的任务。[fishingrod.lua:65](../../../raw/current/scripts/components/fishingrod.lua#L65)、[:70](../../../raw/current/scripts/components/fishingrod.lua#L70) |
| `IsFishing()` | boolean | 仅 `target` 与 `fisherman` 均非空；不校验目标有效性、装备状态或 SG。[fishingrod.lua:87](../../../raw/current/scripts/components/fishingrod.lua#L87 |
| `HasHookedFish()` | boolean | `target` 非空且 `hookedfish` 非空；客户端 replica 还要求 target 网络值非空。[fishingrod.lua:91](../../../raw/current/scripts/components/fishingrod.lua#L91)、[fishingrod_replica.lua:21](../../../raw/current/scripts/components/fishingrod_replica.lua#L21 |
| `HasCaughtFish()` | boolean | 仅 `caughtfish` 非空；`componentactions` 用它抑制新的 `FISH` 动作。[fishingrod.lua:95](../../../raw/current/scripts/components/fishingrod.lua#L95)、[componentactions.lua:1264](../../../raw/current/scripts/componentactions.lua#L1264 |
| `FishIsBiting()` | boolean | 读钓鱼者 SG 的 `nibble` 标签，不是计时器剩余量。[fishingrod.lua:99](../../../raw/current/scripts/components/fishingrod.lua#L99 |
| `StartFishing(target, fisherman)` | 目标需有 `components.fishable` | 先 `StopFishing`，设两者并 `StartUpdatingComponent`；不检查 `fishable` 标签（动作层检查）。[fishingrod.lua:103](../../../raw/current/scripts/components/fishingrod.lua#L103 |
| `WaitForFish()` | 无返回值 | `GetFishPercent() > 0` 才 `DoTaskInTime` 排 `DoNibble`；空塘不排任务。[fishingrod.lua:112](../../../raw/current/scripts/components/fishingrod.lua#L112 |
| `Hook()` | 无返回值 | `HookFish` 生成隐藏鱼并立即扣 `fishleft`；有 `finiteuses` 时排脱杆超时；推 `fishingstrain`。[fishingrod.lua:146](../../../raw/current/scripts/components/fishingrod.lua#L146)、[fishable.lua:58](../../../raw/current/scripts/components/fishable.lua#L58 |
| `Release()` | 无返回值 | 归还上钩鱼（`ReleaseFish` 删除实体并回补鱼量）后 `StopFishing`。[fishingrod.lua:159](../../../raw/current/scripts/components/fishingrod.lua#L159)、[fishable.lua:82](../../../raw/current/scripts/components/fishable.lua#L82 |
| `Reel()` | 无返回值 | `RemoveFish` 取鱼，放到钓鱼者外侧 1 单位，推 `fishingcatch`（带 `build`）；不结束钓鱼。[fishingrod.lua:166](../../../raw/current/scripts/components/fishingrod.lua#L166)、[fishable.lua:92](../../../raw/current/scripts/components/fishable.lua#L92 |
| `Collect()` | 无返回值 | 显示鱼、开物理与影子、`persists=true`，推 `fishingcollect`，再 `StopFishing`。[fishingrod.lua:187](../../../raw/current/scripts/components/fishingrod.lua#L187 |
| `StopFishing()` | 无返回值 | 只向钓鱼者推 `fishingcancel`，清空目标与鱼引用并停更新；不归还上钩鱼。[fishingrod.lua:133](../../../raw/current/scripts/components/fishingrod.lua#L133 |
| `OnUpdate(dt)` | 每帧 | 钓鱼者无效、SG 无 `fishing`／`catchfish` 标签、无 `equippable` 或未装备时调用 `StopFishing`。[fishingrod.lua:75](../../../raw/current/scripts/components/fishingrod.lua#L75 |

### 状态与事件顺序

1. `FISH` → `StartFishing` → 钓鱼者进入 `fishing`，`WaitForFish` 排 `DoNibble`。[actions.lua:1888-1892](../../../raw/current/scripts/actions.lua#L1888-L1892)、[SGwilson.lua:6247](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6247
2. 定时到 → `DoNibble` 向钓竿和钓鱼者推 `fishingnibble` → SG 进入 `fishing_nibble`（1–2 秒）；超时回 `fishing` 并再次 `WaitForFish`。[fishingrod.lua:36](../../../raw/current/scripts/components/fishingrod.lua#L36)、[SGwilson.lua:6276-6283](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6276-L6283)
3. 咬钩窗口内 `REEL` → `Hook` → 两侧 `fishingstrain` → SG `fishing_strain`；耐久超时 → `DoLoseRod` → 两侧 `fishingloserod` → SG `loserod` 先 `Release()` 再删竿。[fishingrod.lua:45](../../../raw/current/scripts/components/fishingrod.lua#L45)、[SGwilson.lua:6379](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6379
4. 应变期间 `REEL` → `Reel` → 两侧 `fishingcatch` → SG `catchfish` → 24 帧后 `Collect` → 两侧 `fishingcollect`，钓竿 prefab 监听该事件扣 1 次耐久，最后 `StopFishing`。[fishingrod.lua:166](../../../raw/current/scripts/components/fishingrod.lua#L166)、[SGwilson.lua:6353](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6353)、[prefabs/fishingrod.lua:32](../../../raw/current/scripts/prefabs/fishingrod.lua#L32

## 典型接入

### 缩短咬钩等待并收紧脱杆窗口（原版钓竿 postinit）

- 前提与选择：写入目标模组 `modmain.lua`，仅服务端执行；复用原版 `fishingrod` prefab、资源与 `finiteuses` 消耗，不新增 prefab。[prefabs/fishingrod.lua:38](../../../raw/current/scripts/prefabs/fishingrod.lua#L38
- 生效与撤销：新生成及读档重建的钓竿都会经过 `fn`，postinit 改组件计时；停用模组后恢复原版 `4`/`40` 与 `0`/`5`。只改计时，不改 `TUNING.FISHINGROD_USES`、伤害或 SG。[prefabs/fishingrod.lua:73](../../../raw/current/scripts/prefabs/fishingrod.lua#L73
- 原版依据：prefab 添加组件后立即设置等待与脱杆时间；`WaitForFish`、`Hook` 读取这些字段。[prefabs/fishingrod.lua:73](../../../raw/current/scripts/prefabs/fishingrod.lua#L73)、[fishingrod.lua:112](../../../raw/current/scripts/components/fishingrod.lua#L112

```lua
-- modmain.lua（服务端逻辑）
AddPrefabPostInit("fishingrod", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local rod = inst.components.fishingrod
    if rod ~= nil then
        rod:SetWaitTimes(2, 15)  -- 秒；鱼量满约 2s，最低鱼量约 15s
        rod:SetStrainTimes(1, 3) -- 秒；满耐久 3s，耗尽时趋近 1s
    end
end)
```

- 调整与验证：参数单位为秒，不是帧；`SetStrainTimes` 的 `min` 对应耐久 0，`max` 对应耐久 100%。示例仅做 Lua 语法检查，未运行游戏；不影响已在上钩状态的旧计时。

### 自建宿主的最低条件

- 服务端给物品 `AddComponent("fishingrod")` 并保证有 `equippable`（`OnUpdate` 检查）；物品还需 `fishingrod` 标签，客户端 `playercontroller` 用该标签放大钓鱼目标搜索半径，动作回调与 replica 由组件注册提供。[prefabs/fishingrod.lua:53](../../../raw/current/scripts/prefabs/fishingrod.lua#L53)、[playercontroller.lua:3434](../../../raw/current/scripts/components/playercontroller.lua#L3434
- 目标实体必须有 `fishable` 组件并带 `fishable` 标签；该标签由 `fishable` 构造时 `self.frozen = false` 触发属性 setter 添加，冻结时移除，不是 prefab 显式 `AddTag`。[fishable.lua:1](../../../raw/current/scripts/components/fishable.lua#L1)、[fishable.lua:17](../../../raw/current/scripts/components/fishable.lua#L17)、[class.lua:36](../../../raw/current/scripts/class.lua#L36
- 需要“脱杆超时”和“收鱼扣耐久”时再加 `finiteuses`，并监听 `fishingcollect`；否则 `Hook` 不排 `DoLoseRod`，钓竿也不会因捕获而消耗。[fishingrod.lua:149](../../../raw/current/scripts/components/fishingrod.lua#L149)、[prefabs/fishingrod.lua:88](../../../raw/current/scripts/prefabs/fishingrod.lua#L88

### 与海钓的分工

- 池塘钓以固定水域实体为目标，按塘内鱼量等待咬钩，靠 SG 的 `nibble`／`strain` 与耐久决定成败；海钓以水面浮标和鱼实体为目标，靠容器中的 bobber／lure、线长与张力，见 [[reference/components/oceanfishingrod]]。
- 选择：固定池塘、鱼量驱动、单竿单鱼 → 本页；开放水面、需要抛距与拟饵搭配 → 海钓竿。两者不共享状态机，不能把本页字段直接套到海钓组件。

## 常见陷阱与限制

- **钓鱼中存档会丢上钩鱼**：`HookFish` 上钩即扣 `fishleft`，隐藏鱼 `persists=false`；`fishable` 只保存 `fishleft`，本组件又不保存 `hookedfish`，存档后鱼从塘中消失且不会补回。[fishable.lua:58](../../../raw/current/scripts/components/fishable.lua#L58)、[fishable.lua:75](../../../raw/current/scripts/components/fishable.lua#L75)、[fishable.lua:135](../../../raw/current/scripts/components/fishable.lua#L135
- **`StopFishing` 不归还上钩鱼**：它只清 `hookedfish` 引用，已生成的隐藏子实体仍在 `fishable.hookedfish` 表里；主动放弃应先 `Release()`。原版 `loserod` 正是先 `Release()` 再删竿。[fishingrod.lua:133](../../../raw/current/scripts/components/fishingrod.lua#L133)、[fishingrod.lua:159](../../../raw/current/scripts/components/fishingrod.lua#L159
- **没有 `finiteuses` 就没有脱杆超时**：`DoLoseRod` 只在 `inst.components.finiteuses` 存在时创建；原版非 quagmire 分支会加 `finiteuses`，quagmire 分支是否由外部脚本补加未核验。[fishingrod.lua:149](../../../raw/current/scripts/components/fishingrod.lua#L149)、[prefabs/fishingrod.lua:77](../../../raw/current/scripts/prefabs/fishingrod.lua#L77
- **`IsFishing()` 不是强一致状态**：目标与钓鱼者非空即为真；卸下装备、玩家切状态或目标失效要等下一次 `OnUpdate` 才 `StopFishing`，动作判断不能把它当“正在有效钓鱼”。[fishingrod.lua:75](../../../raw/current/scripts/components/fishingrod.lua#L75)、[fishingrod.lua:87](../../../raw/current/scripts/components/fishingrod.lua#L87
- **取消只通知钓鱼者**：钓竿自身的 `fishingcancel` 推送被注释；监听钓竿实体的模组收不到取消事件。[fishingrod.lua:134](../../../raw/current/scripts/components/fishingrod.lua#L134
- **空塘永不咬钩**：`WaitForFish` 只在 `GetFishPercent() > 0` 时排任务；`fishleft == 0` 时没有定时器；重新进入 `fishing` 状态只是重新计算，鱼量仍为 0 时依旧不排。[fishingrod.lua:112](../../../raw/current/scripts/components/fishingrod.lua#L112
- **移除组件不清理**：`RemoveComponent` 只停更新并调用可选的 `OnRemoveFromEntity`；本组件未实现，隐藏鱼与 `target`／`fisherman` 不会经 `StopFishing` 复位；replica 随组件移除而反注册。[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670
- **绕过 setter 改字段会破坏客户端**：`target`／`hookedfish`／`caughtfish` 同步依赖 `Class` 属性 setter；用 `rawset` 或直接改内部表不会更新 replica。[fishingrod.lua:31](../../../raw/current/scripts/components/fishingrod.lua#L31)、[class.lua:36](../../../raw/current/scripts/class.lua#L36

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 6 处（actions.lua FISH fn、SGwilson fishing*／catchfish／loserod 状态）；结论修订：无，仅锚点迁移。依据：枢纽 diff 审阅＋变更清单命中 actions.lua、componentactions.lua、SGwilson.lua、components/playercontroller.lua、entityscript.lua（命中区域逐行核对均未漂移）。
- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖组件构造与宿主、`FISH`／`REEL` 动作、等待与脱杆计时、上钩／捕获事件、replica 字段、存档与移除路径。未运行游戏。
- 结构检查：2026-10-21，106 行通过 `validate_page.py`；源码链接 72 处、0 error／0 warning（`--strict`）；示例 `parser_probe --syntax-only` 通过（385 字节）。
- 游戏运行：未运行；多人同步、存档回归、quagmire 分支与 `finiteuses` 耗尽瞬间未覆盖。

相关：[[reference/components/finiteuses|耐久消耗]]；[[reference/components/equippable|装备回调]]；[[reference/components/oceanfishingrod|海钓竿]]。
