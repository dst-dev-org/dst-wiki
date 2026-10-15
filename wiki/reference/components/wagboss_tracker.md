---
title: "wagboss_tracker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "forest 世界组件：记录 Wagstaff 首领是否被击败，存读持久化并广播事件，供裂隙 loot、稀有刷怪与分片同步读取。"
aliases: ["wagboss_tracker", "wagboss击败记录", "瓦格斯首领状态"]
tags: [component, event, server-client]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/wagboss_tracker.lua", "prefabs/forest.lua", "components/shard_wagbossinfo.lua", "stategraphs/SGalterguardian_phase4_lunarrift.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wagboss_tracker

## 快速判断

- 职责与适用：挂在 TheWorld（forest）上的布尔状态组件，`wagboss_defeated` 默认 false；监听世界事件 `wagboss_defeated` 置真并广播 `master_wagbossinfoupdate`。[组件全文](../../../raw/current/scripts/components/wagboss_tracker.lua#L5-L14)
- 优先使用：任何"Wagboss 打死后才解锁"的逻辑都读 `IsWagbossDefeated()`，不要自己监听一次性事件重放状态——组件带存读，重载后状态仍在。[查询接口](../../../raw/current/scripts/components/wagboss_tracker.lua#L12-L14)
- 重要副作用：OnLoad 恢复状态后会再推一次 `master_wagbossinfoupdate`，让分片同步组件在载入时拿到初值。[载入广播](../../../raw/current/scripts/components/wagboss_tracker.lua#L22-L27)
- 修改边界：仅主机世界有意义；事件 `wagboss_defeated` 由月球裂隙守卫终局状态图在 `statemem.wagstaff` 分支推送，其他世界（非 forest）没有本组件，读取方都先判空。[事件源](../../../raw/current/scripts/stategraphs/SGalterguardian_phase4_lunarrift.lua#L1668-L1670)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsWagbossDefeated()` | `-> boolean` | 返回 `self.wagboss_defeated`；唯一状态查询口。[实现](../../../raw/current/scripts/components/wagboss_tracker.lua#L12-L14) |
| `wagboss_defeated` | `boolean`（默认 false） | 监听 `wagboss_defeated` 置真并广播；OnSave 输出该字段，OnLoad 恢复后再次广播。[监听](../../../raw/current/scripts/components/wagboss_tracker.lua#L29) [存读](../../../raw/current/scripts/components/wagboss_tracker.lua#L16-L27) |
| `master_wagbossinfoupdate`（推送） | `{isdefeated = boolean}` | 击败时与 OnLoad 时向 TheWorld 推送；shard_wagbossinfo 在主分片监听并把值写入网络变量同步到其他分片。[推送写入](../../../raw/current/scripts/components/shard_wagbossinfo.lua#L35-L37) |

## 典型接入

### 按击败状态解锁内容（原版消费模式）

- 前提与选择：forest 世界组件，读取方一律 `TheWorld.components.wagboss_tracker and ...:IsWagbossDefeated()` 判空后再用；裂隙熊獾／独眼鹿／战犬的额外战利品、月暴管理、围栏笼刷新都走这一模式。[挂载](../../../raw/current/scripts/prefabs/forest.lua#L736) [loot 判定](../../../raw/current/scripts/prefabs/bearger.lua#L755)
- 生效与撤销：状态只进不退（无复位路径）；依赖它的分支在重载后由 OnLoad 广播重新生效。

```lua
-- 服务端：读取 Wagboss 击败状态（forest 世界）
local tracker = TheWorld.components.wagboss_tracker
if tracker ~= nil and tracker:IsWagbossDefeated() then
    -- 已击败分支：解锁刷新、替换 loot 等
end
```

- 调整与验证：非 forest 世界或模组自定义世界没有该组件，判空是必须步骤；消费方示例还有 lunarriftmutationsmanager（突变门槛）与 gestalt_cage（笼内生成）。[突变门槛](../../../raw/current/scripts/components/lunarriftmutationsmanager.lua#L81) [围栏笼](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L421-L422)

## 常见陷阱与限制

- **直接监听 `wagboss_defeated` 事件代替读状态：** 事件只在击杀瞬间推一次；重载存档后事件不会重放，而组件 OnLoad 会广播 `master_wagbossinfoupdate`——用事件做持久逻辑会丢状态。[载入广播](../../../raw/current/scripts/components/wagboss_tracker.lua#L22-L27)
- **在非主分片找组件：** 组件只存在于主分片 forest 世界；其他分片经 shard_wagbossinfo 的网络变量间接读取，不要跨分片直接访问组件实例。[分片同步](../../../raw/current/scripts/components/shard_wagbossinfo.lua#L43-L50)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（30 行），核对 forest 挂载、事件源状态图、shard_wagbossinfo 同步与 loot／刷新类消费方（bearger、deerclops、warg、moonstormmanager、wagpunk_manager、lunarriftmutationsmanager、gestalt_cage）。未逐行审阅 wagboss 战斗流程本身。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行；击败链路未实测。
