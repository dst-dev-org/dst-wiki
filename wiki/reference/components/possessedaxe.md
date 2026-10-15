---
title: "possessedaxe"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "露西斧（Lucy）的归属与复原组件：只认 woodcutter（伍迪）主人，与玩家 userid 建立持久链接，等待期超时或主人不合格即还原为普通斧（可带剩余耐久与特效），支持存档续等。"
aliases: ["possessedaxe", "露西斧组件", "Lucy 复原", "possessed axe"]
tags: [component, item, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/possessedaxe.lua", "prefabs/lucy.lua", "prefabs/woodie.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# possessedaxe

## 快速判断

- 职责与适用：挂在 lucy prefab 上的服务端组件（([lucy.lua:224-226](../../../raw/current/scripts/prefabs/lucy.lua#L224-L226)）；维护三层归属——`currentowner`（实体持有者链）、`player`／`userid`（绑定的伍迪，掉落也不断链）、等待期计时；主人不是 woodcutter、已有另一把 Lucy、等待超时或 `possessedaxe` 事件都会触发 `Revert()` 换回普通斧。[构造默认](../../../raw/current/scripts/components/possessedaxe.lua#L65-L99) [归属校验](../../../raw/current/scripts/components/possessedaxe.lua#L16-L33)
- 优先使用：改还原目标 → `revert_prefab`／`revert_uses`／`revert_fx`／`transform_fx` 字段（伍迪砍树变形即这么接）；手动触发还原 → `Revert()`。[还原实现](../../../raw/current/scripts/components/possessedaxe.lua#L167-L202) [伍迪接线](../../../raw/current/scripts/prefabs/woodie.lua#L691-L692)
- 重要副作用：`LinkToPlayer` 推 `possessedaxe`（玩家侧）与 `axepossessedbyplayer`（斧侧）事件，[[reference/components/sentientaxe]] 靠后者认主；拒绝主人时推 `axerejectedowner`／`axerejectedotheraxe` 并把斧丢在地上。[链接与事件](../../../raw/current/scripts/components/possessedaxe.lua#L136-L158) [拒绝](../../../raw/current/scripts/components/possessedaxe.lua#L21-L32)
- 修改边界：等待期默认 `TUNING.LUCY_REVERT_TIME`(90 秒)，玩家实体移除（死亡/掉线）即转入按 userid 的等待；存档只存 userid 与剩余等待时间，同名玩家重入触发 `ms_playerjoined` 续链，超时直接还原。[等待](../../../raw/current/scripts/components/possessedaxe.lua#L107-L121) [存档](../../../raw/current/scripts/components/possessedaxe.lua#L204-L213)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `(inst)` | 默认 `revert_prefab = "axe"`、`revert_time = LUCY_REVERT_TIME`；监听 `onputininventory`／`ondropped` 触发归属重检；`onplayerjoined` 桥接重连续链或还原。([possessedaxe.lua:65-99](../../../raw/current/scripts/components/possessedaxe.lua#L65-L99) |
| 归属重检 | `OnChangeOwner`／`OnCheckOwner` | 持有者变化后延迟 0 帧检查：非 `woodcutter` tag → `Drop`＋`axerejectedowner`；已有另一把 possessedaxe → `Drop`＋对另一把推 `axerejectedotheraxe`；玩家主人则 `LinkToPlayer`。([possessedaxe.lua:16-63](../../../raw/current/scripts/components/possessedaxe.lua#L16-L63) |
| `LinkToPlayer(player)` | 玩家或 nil | 停等待、换绑 `player`／`userid`，互推事件并监听玩家 `onremove` 与 `possessedaxe`；nil 时解除全部并推 `axepossessedbyplayer(nil)`。([possessedaxe.lua:136-158](../../../raw/current/scripts/components/possessedaxe.lua#L136-L158) |
| `WaitForPlayer(userid, delay)` | userid, 秒 | 解链进入等待：留 userid 监听 `ms_playerjoined`（等伍迪回来），nil 时清掉监听、不再按 userid 等待；超时 `Revert()`。([possessedaxe.lua:107-121](../../../raw/current/scripts/components/possessedaxe.lua#L107-L121) |
| `StopWaitingForPlayer()` | 无参 | 取消等待任务与 `ms_playerjoined` 监听；由 `LinkToPlayer` 内部调用。([possessedaxe.lua:123-134](../../../raw/current/scripts/components/possessedaxe.lua#L123-L134) |
| `Revert()` | 返回还原后的斧实体 | 按 `revert_prefab` 生成新斧并恢复 `revert_uses` 耐久；三种位置（地面/装备栏/背包槽）逐一搬移并清理本体，`revert_fx` 播变身特效。([possessedaxe.lua:167-202](../../../raw/current/scripts/components/possessedaxe.lua#L167-L202) |
| `Drop()` | 无参 | 从当前持有者背包（含容器）丢到地面；校验失败路径使用。([possessedaxe.lua:160-165](../../../raw/current/scripts/components/possessedaxe.lua#L160-L165) |
| `OnSave`／`OnLoad` | 存档钩子 | 存非默认 `revert_prefab`、`revert_uses`、`userid` 与剩余等待时间；载入无主时按剩余时间续等。([possessedaxe.lua:204-226](../../../raw/current/scripts/components/possessedaxe.lua#L204-L226) |

## 典型接入

### 让自定义"有灵性"工具复用归属与还原逻辑

- 前提与选择：宿主需 inventoryitem（owner 链与 Drop）、equippable（装备位还原分支）与 finiteuses（耐久还原可选）；"合法主人"判定硬编码为 `woodcutter` tag，自定义角色需另写组件或改 tag 语义。
- 生效与撤销：链接即生效并广播事件；`Revert` 是终点操作（本体被 Remove），不可撤销。
- 原版依据：伍迪侧变形接线 ([woodie.lua:681-702](../../../raw/current/scripts/prefabs/woodie.lua#L681-L702)（onworked 砍树时以当前斧生成 Lucy 并回填耐久）；[[reference/components/sentientaxe]] 经 `axepossessedbyplayer` 认主。

```lua
-- lucy prefab master_postinit 的原版接线（还原目标与特效）：
inst.components.possessedaxe.revert_fx = "lucy_ground_transform_fx"
inst.components.possessedaxe.transform_fx = "lucy_transform_fx"
-- 伍迪 onworked 中（变形其他斧为 Lucy 时）：
-- lucy.components.possessedaxe.revert_prefab = equipitem.prefab
-- lucy.components.possessedaxe.revert_uses = itemuses
```

- 调整与验证：`revert_time` 只影响"无主/死亡等待"时长，不影响正常持有；验证用 `GetDebugString()` 查持有链与超时剩余。

## 常见陷阱与限制

- 主人合法性只看 `woodcutter` tag（([possessedaxe.lua:1-3](../../../raw/current/scripts/components/possessedaxe.lua#L1-L3)）：给其他角色用 Lucy 会在下一次归属检查时被丢在地上并推 `axerejectedowner`。
- `onplayerdied` 回调已定义但源码内无任何监听点（([possessedaxe.lua:82](../../../raw/current/scripts/components/possessedaxe.lua#L82)，全库 grep 仅此一处）：死亡等待实际由 `onplayerremoved` 与 `ms_playerjoined` 路径覆盖，勿以为该回调会被触发。
- `OwnerAlreadyHasPossessedAxe` 扫描整个背包（([possessedaxe.lua:5-14](../../../raw/current/scripts/components/possessedaxe.lua#L5-L14)），箱子里的第二把 Lucy 不阻止绑定，但同背包两把会互斥。
- `Revert` 直接 `Remove` 本体（([possessedaxe.lua:178](../../../raw/current/scripts/components/possessedaxe.lua#L178）：在其事件回调里再操作 `self.inst` 会访问失效实体，需以返回的新斧为准。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全文；核验 lucy.lua 挂载与特效接线、woodie.lua 变形接线、sentientaxe 认主监听、`onplayerdied` 全库无监听点及 LUCY_REVERT_TIME 调谐。
- 游戏运行：未运行。
- 结构检查：未执行。

相关链接：[[reference/components/sentientaxe]]、[[reference/components/inventoryitem]]、[[reference/components/equippable]]、[[reference/components/finiteuses]]
