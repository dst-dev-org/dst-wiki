---
title: "mermkingmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级鱼人王管理器：登记王座、挑选并护送鱼人候选者，候选到位后变身鱼人王，并维护跨分片的王与任务装备查询。"
aliases: ["mermkingmanager", "鱼人王", "merm king", "王座候选", "鱼人王管理器"]
tags: [component, creature, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/mermkingmanager.lua", "prefabs/cave.lua", "prefabs/forest.lua", "prefabs/mermthrone.lua", "prefabs/merm.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# mermkingmanager

## 快速判断

- 职责与适用：世界组件（cave 与 forest 都安装），维护王座表、候选者表与当前鱼人王；监听王座建造/摧毁与候选者到达事件，驱动变身与王位更替。[安装](../../../raw/current/scripts/prefabs/cave.lua#L539-L541) [森林安装](../../../raw/current/scripts/prefabs/forest.lua#L676
- 优先使用：查王用 `GetKing()`／`HasKingLocal()`；跨分片（洞穴王座建在地面等）用 `HasKingAnywhere()`；查王座归属用 `GetThrone(merm)`／`GetMainThrone()`。[查询](../../../raw/current/scripts/components/mermkingmanager.lua#L231-L265)
- 重要副作用：候选者被登记即改名 MERM_PRINCE 并加 `mermprince` 标签；变身 `ReplacePrefab(candidate, "mermking")` 原地替换实体并丢弃其物品；王死亡只标记 `king_dying` 且 persists=false，移除后推世界事件 `onmermkingdestroyed`。[变身](../../../raw/current/scripts/components/mermkingmanager.lua#L123-L153) [王死亡](../../../raw/current/scripts/components/mermkingmanager.lua#L41-L45)
- 修改边界：王座有效性要求未燃烧、未烧毁且无 mermkingcorpse 尸体占用；组件无 OnRemoveFromEntity；主王座被拆（如解构杖）会杀死当前王。[王座校验](../../../raw/current/scripts/components/mermkingmanager.lua#L223-L229) [主王座摧毁](../../../raw/current/scripts/components/mermkingmanager.lua#L112-L120)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `candidates` | {throne=candidate} 表 | 每个有效王座最多一名候选；登记由 `ShouldGoToThrone(merm, throne)` 完成（非 shadowminion、王座有效、该王座无候选）。[候选登记](../../../raw/current/scripts/components/mermkingmanager.lua#L176-L193) |
| `FindMermCandidate(throne)` | — | 在王座 50 距离找活的、非 player/mermking/mermguard 的 merm，令其走向王座；已有王时直接返回。[寻找](../../../raw/current/scripts/components/mermkingmanager.lua#L157-L174) |
| `ShouldTransform(merm)` | → bool | 候选已在王座 0.5 内、mermcandidate 组件允许且无并发变身时为真；登记 candidate_transforming。变身事件 `oncandidatekingarrived` 由 merm 状态机在候选坐下后推送。[判定](../../../raw/current/scripts/components/mermkingmanager.lua#L199-L214) [事件接线](../../../raw/current/scripts/components/mermkingmanager.lua#L12-L25) |
| `CreateMermKing(candidate, throne)` | — | 候选丢物品、ReplacePrefab 成王、登记死亡/移除监听、把王座从列表挪到 main_throne、重置全部其他候选，推 `onmermkingcreated {king, throne}`。[变身](../../../raw/current/scripts/components/mermkingmanager.lua#L123-L153) |
| `HasKingLocal()`／`HasKingAnywhere()` | → bool | 本世界王存在且活着；AnyWhere 版本再问 `shard_mermkingwatcher`（分片同步）。`HasKing()` 是兼容旧模组的 stub（等同 AnyWhere）。[本地](../../../raw/current/scripts/components/mermkingmanager.lua#L282-L286) [跨分片](../../../raw/current/scripts/components/mermkingmanager.lua#L288-L299) |
| `HasTridentLocal/Anywhere()` 等 | → bool | 王的任务装备（三叉戟/王冠/护肩）同构的本地/跨分片查询对，共三组。[装备查询](../../../raw/current/scripts/components/mermkingmanager.lua#L302-L333) |
| `IsThroneValid(throne)` | → bool | throne 有效、未燃烧/烧毁、0.1 内无 mermkingcorpse；候选取消与再挑选都先过它。[校验](../../../raw/current/scripts/components/mermkingmanager.lua#L223-L229) |
| `IsThrone(throne)` | → bool | 是否本管理器登记过的王座（主王座或列表内）。[登记查询](../../../raw/current/scripts/components/mermkingmanager.lua#L267-L279) |
| `OnSave()`／`LoadPostPass(newents, savedata)` | GUID 表 | 存王、候选、变身中候选与王座的 GUID；king_dying 期间主王座并入 thrones 列表保存（王不存），重载后恢复监听并重推 `onmermkingcreated`。[存读](../../../raw/current/scripts/components/mermkingmanager.lua#L336-L417) |

### 王座与候选的实体协议

- 王座（mermthrone 系 prefab）建造后推世界事件 `onthronebuilt {throne}`，被烧/砸/拆推 `onthronedestroyed`；管理器据此登记与清候选。[王座事件](../../../raw/current/scripts/prefabs/mermthrone.lua#L27-L38)
- 候选者（merm）吃食物累计 mermcandidate 卡路里；达到阈值后由其状态机执行"走向王座→入座→推送 oncandidatekingarrived"。[候选组件安装](../../../raw/current/scripts/prefabs/merm.lua#L1099

## 典型接入

### 自定义王座或鱼人加入王位流程

- 前提与选择：世界需装有 mermkingmanager（森林/洞穴默认都有）；自定义"鱼人"宿主要有 mermcandidate 组件参与卡路里流程；王座只需在建造/摧毁时推 `onthronebuilt`／`onthronedestroyed` 即可被登记。[候选安装](../../../raw/current/scripts/prefabs/merm.lua#L1095-L1099) [王座事件](../../../raw/current/scripts/prefabs/mermthrone.lua#L27-L29)
- 生效与撤销：候选登记后立即改观感（MERM_PRINCE）；取消候选走 `onremove`/`death` 自动清理，或王座被毁时统一重置。变身不可逆（ReplacePrefab 换实体）。
- 原版依据：mermwatchtower 等消费方都用 `HasKingAnywhere()` 决定守卫生成与台词，跨分片语义由 shard_mermkingwatcher 提供。[哨塔消费](../../../raw/current/scripts/prefabs/mermwatchtower.lua#L16-L17)

```lua
-- 服务器端：查询当前世界（或任一分片）是否已有鱼人王，决定奖励/任务线
local function HasAnyMermKing()
    local manager = TheWorld.components.mermkingmanager
    if manager == nil then
        return false
    end
    return manager:HasKingAnywhere()
end
```

- 调整与验证：洞穴王座与地面王座分属两个世界的管理器实例，跨分片结果依赖 shard_mermkingwatcher 的同步组件；只装管理器不装分片监听时 AnyWhere 退化为本世界。[分片依赖](../../../raw/current/scripts/components/mermkingmanager.lua#L293-L295)

## 常见陷阱与限制

- **直接杀王不走 OnKingDeath 协议会卡死王位：** 只有经 `death` 事件才置 `king_dying` 并把王 persists=false（不存档）；用 Remove 之类绕过只触发 OnKingRemoval，王位照常清空——但王座归属、候选重置等依赖成对事件。[成对监听](../../../raw/current/scripts/components/mermkingmanager.lua#L131-L132)
- **候选表按王座实例为键：** 王座被烧毁重建后是不同实体，旧候选条目由 `onthronedestroyed` 清理；模组替换王座实体而不推事件会留下悬挂候选。[清理](../../../raw/current/scripts/components/mermkingmanager.lua#L86-L121)
- **`ShouldTransform` 依赖 mermcandidate：** 对没有 mermcandidate 组件的实体会直接报错（访问 `merm.components.mermcandidate:ShouldTransform()` 无判空）；自定义候选必须装该组件。[访问](../../../raw/current/scripts/components/mermkingmanager.lua#L199-L206)
- **主王座被解构杖拆除会杀王：** 这是代码注释明示的唯一预期路径；普通王座摧毁不触碰王。[注释与实现](../../../raw/current/scripts/components/mermkingmanager.lua#L112-L119)
- **同名 deprecated 接口别用：** `HasKing()` 只为旧模组保留（等同 HasKingAnywhere），新代码用 Local/Anywhere 显式版本。[stub](../../../raw/current/scripts/components/mermkingmanager.lua#L297-L299)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 mermkingmanager.lua，核验王座登记、候选挑选/护送、变身与王位更替、死亡与移除事件对、任务装备查询、跨分片依赖与存读；核对 cave/forest 安装、mermthrone 事件推送与 merm.lua 候选组件接线、mermwatchtower 消费。未审校 mermcandidate 内部卡路里阈值细节。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；变身流程、王死亡后候选重选与跨分片同步未实测。

相关：[[reference/components/mermcandidate]]、[[reference/components/shard_mermkingwatcher]]。

## 相关链接

- [[reference/components/mermcandidate]]：候选者的卡路里积累与变身判定。
- [[reference/components/shard_mermkingwatcher]]：跨分片王状态的同步组件。
