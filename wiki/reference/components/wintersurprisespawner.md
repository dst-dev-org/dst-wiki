---
title: "wintersurprisespawner：冬日盛宴惊喜树生成"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "冬日盛宴活动专属世界组件：冬季内在鹿群刷新点附近放置装满装饰与礼物的 winter_tree 惊喜树，单季限量并受世界设置计时器约束。"
aliases: [惊喜树生成器, 冬日盛宴惊喜, wintersurprisespawner, 惊喜圣诞树]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/wintersurprisespawner.lua", "prefabs/forest.lua", "prefabs/deerspawningground.lua", "components/klaussackspawner.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wintersurprisespawner（冬日盛宴惊喜树生成）

## 快速判断

- 职责与适用：仅冬日盛宴（Winters Feast）活动开启时挂载（([forest.lua:684-687](../../../raw/current/scripts/prefabs/forest.lua#L684-L687)）的服务端世界组件。监听 `ms_registerdeerspawningground`（由 [deerspawningground.lua:12](../../../raw/current/scripts/prefabs/deerspawningground.lua#L12 推送）收集候选点，冬季内经 [[reference/components/worldsettingstimer]] 计时在远离玩家的候选点旁生成 `winter_tree` 并填满礼物（[放置](../../../raw/current/scripts/components/wintersurprisespawner.lua#L188-L214)）。
- 优先使用：调数量与间隔 → 调谐 `WINTERSURPRISE_SPAWN_DELAY`(2 天±1 天)、`WINTERSURPRISE_MAX_SPAWNS`(3)（([tuning.lua:9244-9247](../../../raw/current/scripts/tuning.lua#L9244-L9247)）；组件无公开强制生成接口。
- 重要副作用：与克劳斯袋子联动——候选点须与袋子同分区（topology task_id），且不在袋子 10 单位内（[克袋避让](../../../raw/current/scripts/components/wintersurprisespawner.lua#L50-L64)）；礼物内的惊喜怪由 hounded 逃逸等级决定（[礼物配置](../../../raw/current/scripts/components/wintersurprisespawner.lua#L101-L104)）。
- 修改边界：冬季结束停表并清空当季计数（[季节切换](../../../raw/current/scripts/components/wintersurprisespawner.lua#L266-L275)）；活动未开启则组件不存在，相关行为整体缺席。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnPostInit()` | 载入后调用 | `WatchWorldState("iswinter")` 并立即按当前季态执行一次；向 worldsettingstimer 注册 `wintersurprise_spawntimer`（[实现](../../../raw/current/scripts/components/wintersurprisespawner.lua#L288-L293)）。 |
| `OnSave/OnLoad` | 存读钩子 | 只存 `spawnsthiswinter` 计数；计时本体在 worldsettingstimer（[存读](../../../raw/current/scripts/components/wintersurprisespawner.lua#L299-L310)）。 |
| `GetDebugString()` | debug 文本 | 输出当季计数（[debug](../../../raw/current/scripts/components/wintersurprisespawner.lua#L316-L318)）。 |

选点与放置链（均核验自组件内部）：

1. 候选登记：`ms_registerdeerspawningground` 去重入 `_spawners`，监听移除（[登记](../../../raw/current/scripts/components/wintersurprisespawner.lua#L255-L264)）。
2. 计时：冬季开始且未达上限 → `StartRespawnTimer(SPAWN_DELAY ± VARIANCE)`；到点 `SpawnWinterSurprise` 并续表直到用尽配额（[计时](../../../raw/current/scripts/components/wintersurprisespawner.lua#L220-L240)）。
3. 选点：洗牌候选点 → 同克袋分区、非袋 10 单位内、12 单位内无既有惊喜树（`is_leif` 标记）、3×3 地块可通行 → 距玩家 ≥35 → 5~10 单位偏移处无 INLIMBO/阻挡物/5 单位内无 structure（[有效性](../../../raw/current/scripts/components/wintersurprisespawner.lua#L47-L82)、[偏移](../../../raw/current/scripts/components/wintersurprisespawner.lua#L174-L205)）。
4. 配置：`growable:SetStage(5)` 长成、容器填满装饰品（10% 华丽）、环状撒 2~8 份礼物：约 25% 真礼（`GetNiceWinterTreeGiftLoot`），其余 `giftsurprise` 假礼并挂 `scenariorunner` 脚本 `gift_surprise`（[配置](../../../raw/current/scripts/components/wintersurprisespawner.lua#L89-L172)）。

## 典型接入

### 模组在非活动期启用惊喜树

- 前提与选择：服务端；原版挂载受 `IsSpecialEventActive(WINTERS_FEAST)` 门控，活动期由 `specialeventsetup` 统一管理。模组可在自己的世界里手工挂载本组件复刻行为。
- 生效与撤销：挂载后需满足"冬季＋有 deerspawningground 候选点"才开始出树；卸载组件不回收已生成的树。

```lua
-- 服务端，AddPrefabPostInit("forest", ...) 内（仅主模拟端）
AddPrefabPostInit("forest", function(inst)
    if not TheWorld.ismastersim then return end
    if inst.components.wintersurprisespawner == nil then
        inst:AddComponent("wintersurprisespawner")
    end
end)
```

- 调整与验证：真实礼物/假礼比例 0.25 写死（[比例](../../../raw/current/scripts/components/wintersurprisespawner.lua#L104)）；单季上限 3 由 `CanSpawnWinterSurprise` 把关（[配额](../../../raw/current/scripts/components/wintersurprisespawner.lua#L43-L45)）。

## 常见陷阱与限制

- **依赖克劳斯袋子组件存在与否**：`klaussackspawner` 不在时跳过分区校验与袋子距离限制（[判空](../../../raw/current/scripts/components/wintersurprisespawner.lua#L50)），选点条件随之放宽。
- **候选点复用鹿刷新点**：没有独立配置项；`deerspawningground` 数量决定可放置范围（[事件来源](../../../raw/current/scripts/prefabs/deerspawningground.lua#L12)），deerherdspawner/klaussackspawner 共用同一事件。
- **每季计数不按日历重置以外的方式清零**：只有 `OnIsWinter(false)`（冬季结束）清零；活动中途关闭再开启不会重置（[重置点](../../../raw/current/scripts/components/wintersurprisespawner.lua#L273)）。
- **礼物内容在生成时即定**：真礼/假礼、惊喜怪 prefab、装饰品在放置时生成并打包（unwrappable），事后无法改内容（[打包](../../../raw/current/scripts/components/wintersurprisespawner.lua#L156)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/wintersurprisespawner.lua` 全部 324 行、forest.lua 挂载条件、deerspawningground 事件、klaussackspawner 共用事件确认、tuning WINTERSURPRISE 块。未覆盖：`winter_tree` prefab 与 `gift_surprise` 场景脚本内容、ornament 随机表。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
