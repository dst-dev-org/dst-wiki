---
title: "itemmimic：拟态物品现形与 TurnEvil"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "把物品登记为拟态：按装备槽/持有状态监听使用事件，到点或自动到期时现形替换为 itemmimic_revealed 并惊吓掉落者。"
aliases: [物品拟态, 拟态物品现形, itemmimic, 拟怪物品]
tags: [component, callback, item, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/itemmimic.lua", "components/itemmimicconfig.lua", "componentutil.lua", "components/shadowthrall_mimics.lua", "components/builder.lua", "actions.lua", "components/trader.lua", "components/hauntable.lua", "components/pocketwatch_dismantler.lua", "prefabs/staff_common.lua", "prefabs/wx78_mimicspawner.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# itemmimic（拟态物品现形与 TurnEvil）

## 快速判断

- 职责与适用：挂在拟态物品上的服务端行为组件。自动现形倒计时（默认一天＋随机一天）内按"怎么被用"决定是否提前现形：手持使用/攻击、头身装备挨打、容器内非白名单动作、机器交互都会触发判定；现形即 `TurnEvil`——弹出持有者背包、替换为 `itemmimic_revealed` 并惊吓掉精神（[TurnEvil](../../../raw/current/scripts/components/itemmimic.lua#L201-L229)）。
- 优先使用：原版影子暗影拟怪来源在 `shadowthrall_mimics` 组件（[挂载](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L59)）；自制拟态物品 `AddComponent("itemmimic")`，要动作级强制揭示再加 [[reference/components/itemmimicconfig]]。
- 重要副作用：非装备类拟态物入包后 8~12 秒必现形（[入包](../../../raw/current/scripts/components/itemmimic.lua#L59-L71)）；`pausesources` 非空时暂停自动现形并记录剩余时间（[暂停](../../../raw/current/scripts/components/itemmimic.lua#L178-L185)）。
- 修改边界：组件处理"何时现形"，动作级白名单由 itemmimicconfig＋全局 `ShouldItemMimicBeRevealedFor` 判定链决定（[门禁](../../../raw/current/scripts/componentutil.lua#L2339)）；单独挂本组件不产生强制揭示。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `TurnEvil(target)` | target 为使用者实体或 nil | 从所有者容器 `DropItem`，`ReplacePrefab("itemmimic_revealed")`，对新实体 `PushEvent("jump", target)`；target 有效则 `startled`，判定通过再扣 `SANITY_SMALL`(10)（[实现](../../../raw/current/scripts/components/itemmimic.lua#L201-L229)）。 |
| `AddPauseSource(source, reason)` | 源实体＋理由串 | 记入 `pausesources`（SourceModifierList 布尔表），并取消自动现形任务、把剩余时间存 `reveal_time_remaining`（[实现](../../../raw/current/scripts/components/itemmimic.lua#L178-L185)）。 |
| `RemovePauseSource(source, reason)` | 同上 | 全部暂停源解除后按剩余时间重启自动现形（[实现](../../../raw/current/scripts/components/itemmimic.lua#L187-L199)）。 |
| `SetNoLoot(noloot)` | 布尔 | 存档字段；TurnEvil 后转给 revealed 实体（[SetNoLoot](../../../raw/current/scripts/components/itemmimic.lua#L231-L233)）。 |
| `OnSave/OnLoad` | 存读钩子 | 存 `noloot`、`reveal_time_remaining`、`add_component_if_missing=true`（读档自动补挂）（[存读](../../../raw/current/scripts/components/itemmimic.lua#L254-L285)）。 |
| `LongUpdate(dt)` | 跳时间补偿 | 暂停中跳过；否则按剩余时间推进或立即现形（[LongUpdate](../../../raw/current/scripts/components/itemmimic.lua#L236-L251)）。 |

事件到现形的映射（均在组件内注册）：

- 入包：给持有者监听 `performaction`；非装备类另起 8+4×random 秒现形任务（[on_put_in_inventory](../../../raw/current/scripts/components/itemmimic.lua#L59-L71)）。
- 装备：按槽位挂使用方事件——HANDS `working`/`onattackother`，HEAD/BODY `attacked`/`blocked`（[on_equipped](../../../raw/current/scripts/components/itemmimic.lua#L3-L21)）。
- performaction 白名单：`EQUIP/UNEQUIP/DROP/PICKUP` 不触发（[ACCEPTABLE_ACTIONS](../../../raw/current/scripts/components/itemmimic.lua#L91-L97)），其余动作 5 帧后判定（[监听器](../../../raw/current/scripts/components/itemmimic.lua#L139-L144)）。
- 机器交互：`machineturnedon/off`、`percentusedchange` 10 帧后判定（[机器反应](../../../raw/current/scripts/components/itemmimic.lua#L105-L110)）。
- 暂停刷新：世界事件 `itemmimic_refreshpausestates` 重算 `socket_shadow_mimicry` 暂停源（WX-78 影子拟态插座按住拟态）（[刷新](../../../raw/current/scripts/components/itemmimic.lua#L112-L120)）。

## 典型接入

### 自制拟态物品（对照原版来源）

- 前提与选择：物品 Prefab 服务端构造段。原版由 `shadowthrall_mimics` 生成拟态实体时挂载（[来源](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L59)），场景与调试命令也可直接挂（[场景](../../../raw/current/scripts/scenarios/chest_labyrinth_mimic.lua#L19)、[debug](../../../raw/current/scripts/debugcommands.lua#L4274)）。
- 生效与撤销：挂上即开始自动现形倒计时；玩家正常拾取/装备/丢弃不触发现形，只有白名单外的使用动作或到期才现形。

```lua
-- 服务端，物品 prefab 的 master_postinit 内
inst:AddComponent("inventoryitem")
inst:AddComponent("itemmimic")
inst.components.itemmimic:SetNoLoot(true)  -- 现形怪不掉该物品承载的战利品

-- 撤销（通常不需要；读档时 OnSave 的 add_component_if_missing 会自动补挂）
inst:RemoveComponent("itemmimic")
```

- 调整与验证：自动现形周期为 `ITEMMIMIC_AUTO_REVEAL_BASE + rand×RAND`（各一天，[调谐](../../../raw/current/scripts/tuning.lua#L8518-L8519)）；现形实体与惊吓表现由 `itemmimic_revealed` prefab 决定，本页未核验。

## 常见陷阱与限制

- **多处系统绕过判定直接现形**：GIVE 动作（([actions.lua:2290-2291](../../../raw/current/scripts/actions.lua#L2290-L2291)）、制造消耗与拟态折扣（([builder.lua:522-539](../../../raw/current/scripts/components/builder.lua#L522-L539)）、作祟（([hauntable.lua:84-85](../../../raw/current/scripts/components/hauntable.lua#L84-L85)）、拆解（([pocketwatch_dismantler.lua:21-26](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L21-L26)）、绿星杖（([staff_common.lua:652-657](../../../raw/current/scripts/prefabs/staff_common.lua#L652-L657)）都直接调 `TurnEvil`，不走 itemmimicconfig 白名单。
- **trader 默认拒收拟态物**：`acceptsmimics` 未开启时交易入参即被拒（[trader.lua:109](../../../raw/current/scripts/components/trader.lua#L109）。
- **暂停源只管自动倒计时**：`pausesources` 不拦截事件驱动的判定（WX-78 插座按住是靠判定链对普通使用者返回 false＋强制刷新暂停源实现，见 [刷新暂停](../../../raw/current/scripts/components/itemmimic.lua#L112-L119)）。
- **无 `AddComponent("itemmimic")` 的 prefab 原版挂载**：原版挂载发生在 `shadowthrall_mimics` 组件运行时与场景/调试脚本，prefab 文件内无构造期挂载（全库 grep 核验）。
- **拟态不复拟态**：来源组件对已带 itemmimic 的实体不再生成新拟态（[防套娃](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L87)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/itemmimic.lua` 全部 298 行、判定链入口（componentutil.lua:2339）、直接 TurnEvil 的六个系统调用点、shadowthrall_mimics 挂载与防套娃、trader 拒收、debugcommands 与场景挂载、tuning 自动现形常量。未覆盖：`itemmimic_revealed` prefab 表现、WX-78 影子拟态插座完整行为。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
