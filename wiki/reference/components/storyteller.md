---
title: "storyteller"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "实体对故事道具（营火等）讲故事的驱动组件：按 provided 剧本走 talker 台词、周期 tick 回调，结束/中断有完整收尾。"
aliases: ["讲故事", "营火故事", "storyteller"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/storyteller.lua", "components/talker.lua", "prefabs/walter.lua", "actions.lua", "componentactions.lua", "prefabs/portablefirepit.lua", "prefabs/wobybig.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# storyteller

## 快速判断

- 职责与适用：让挂载实体成为"讲故事者"，对着带 `storytellingprop` tag 的道具播放一段剧本；构造加 `storyteller` tag（动作过滤靠它）。[构造](../../../raw/current/scripts/components/storyteller.lua#L16-L28)
- 优先使用：配置用 `SetStoryToTellFn`／`SetOnStoryBeginFn`／`SetOnStoryOverFn`，发起用 `TellStory(storyprop)`，中途取消用 `AbortStory(reason)`，查询用 `IsTellingStory()`。[TellStory](../../../raw/current/scripts/components/storyteller.lua#L82-L117)
- 重要副作用：台词经 `talker:Say` 播放（noanim）；`donetalking`（talker 结束事件）与道具 `onremove` 都会终止故事；有 `ontickfn` 时按 `ticktime`（默认 2.5 秒）周期回调。[donetalking 来源](../../../raw/current/scripts/components/talker.lua#L245)
- 修改边界：组件不检夜／不检燃料，这些条件由 `storytotellfn` 返回值决定；`storytelling_dist = 10` 为声明字段，原版无任何读者。[返回字符串即失败](../../../raw/current/scripts/components/storyteller.lua#L85-L87)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `storytotellfn(inst, prop)` 回调 | 返回 nil／string／story 表 | string 视为拒绝原因（`TellStory` 返回 false）；story 表至少含 `lines`，可选 `ontickfn`、`ticktime`。[L82-L90](../../../raw/current/scripts/components/storyteller.lua#L82-L90) |
| `TellStory(storyprop)` | 布尔 | 组装 `{message, noanim=true, duration?}` 后 `talker:Say(lines, nil, true, true)`；挂 `donetalking` 与道具 `onremove` 监听并回调 `onstorybeginfn`。[L92-L113](../../../raw/current/scripts/components/storyteller.lua#L92-L113) |
| `AbortStory(reason)` | 无返回 | `OnDone` 后经 talker 说 `reason` 或 `ShutUp()`；道具被移除时自动走此路径。[L51-L60](../../../raw/current/scripts/components/storyteller.lua#L51-L60)、[监听](../../../raw/current/scripts/components/storyteller.lua#L26-L28) |
| `OnDone`（内部收尾） | 无返回 | 摘监听、取消 tick 任务、回调 `onstoryoverfn(inst, story)`、清 `self.story`。[L62-L76](../../../raw/current/scripts/components/storyteller.lua#L62-L76) |
| `OnStoryTick` | 无返回 | 周期调 `story.ontickfn(inst, story)`，间隔 `story.ticktime or 2.5` 秒。[L78-L80](../../../raw/current/scripts/components/storyteller.lua#L78-L80)、[L104](../../../raw/current/scripts/components/storyteller.lua#L104) |
| `IsTellingStory()` | 布尔 | `self.story ~= nil`；Woby 据此在主人讲故事时跟随停步。[L47-L49](../../../raw/current/scripts/components/storyteller.lua#L47-L49)、[Woby](../../../raw/current/scripts/prefabs/wobybig.lua#L890) |

## 典型接入

### 原版宿主：Walter 的营火故事

- 前提与选择：walter.lua 挂组件并注入三个回调；`StoryToTellFn` 里判夜、判火、抽剧本，失败返回 `"NOT_NIGHT"`／`"NO_FIRE"` 字符串，成功返回 `{style, id, lines}` 并生成 `_story_proxy` 音源代理。[挂载](../../../raw/current/scripts/prefabs/walter.lua#L1194-L1196)、[剧本函数](../../../raw/current/scripts/prefabs/walter.lua#L515-L541)
- 外部中断：夜晚结束或营火熄灭时，walter 侧守卫对在讲故事的角色 `AbortStory(ANNOUNCE_...)`。[守卫](../../../raw/current/scripts/prefabs/walter.lua#L1236-L1251)
- 动作入口：`TELLSTORY` 动作要求 doer 有 `storyteller` tag、目标有 `storytellingprop` tag，执行即 `TellStory(act.target or act.invobject)`。[动作](../../../raw/current/scripts/actions.lua#L1586-L1590)、[过滤](../../../raw/current/scripts/componentactions.lua#L863-L869)
- 表现联动：便携营火在白天的提示语里排除带 `storyteller` 组件的观察者（避免误导）。[营火](../../../raw/current/scripts/prefabs/portablefirepit.lua#L94-L96)

### 模组：给角色加一个可讲故事的新道具

- 前提与选择：目标道具 prefab 加 `storytellingprop` tag；给角色挂组件并注入 `storytotellfn`；不需要新增动作，复用 `TELLSTORY`。
- 生效与撤销：`storytotellfn` 返回 nil/string 即不出故事；组件随角色存在，移除需注意 `OnRemoveFromEntity` 只摘 tag，不打断进行中的故事（道具移除监听会兜底）。[L31-L33](../../../raw/current/scripts/components/storyteller.lua#L31-L33)

```lua
-- modmain.lua：让任意角色对着新道具讲一段三句的故事（服务端）
local function StoryToTellFn(inst, prop)
    if not GLOBAL.TheWorld.state.isnight then
        return "NOT_NIGHT"
    end
    return { lines = { "第一句。", "第二句。", { line = "带时长的第三句。", duration = 4 } } }
end

AddPrefabPostInit("walter", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst:AddComponent("storyteller")
    inst.components.storyteller:SetStoryToTellFn(StoryToTellFn)
end)
```

- 调整与验证：示例仅语法检查未运行；`lines` 条目为字符串或 `{line, duration}` 表，`duration` 需可 `tonumber`；`ticktime` 单位秒。

## 常见陷阱与限制

- `OnDone` 里直接解引用 `self.story.prop`：若在 `story == nil` 时被外部触发会报错；正常路径均由 `TellStory` 先赋值保证。[L64](../../../raw/current/scripts/components/storyteller.lua#L64)
- `storytotellfn` 返回的表被组件原地补 `story.prop = storyprop`，跨角色复用同一张常量表会被污染，应每次构造新表。[L90](../../../raw/current/scripts/components/storyteller.lua#L90)
- `storytelling_dist = 10` 声明后全库无读者（stageactor 自带同名字段同样未用）；想限制听众距离需自行实现，勿以为改它生效。[声明](../../../raw/current/scripts/components/storyteller.lua#L21)
- 台词用 `noanim = true`，说话者不播说话动画；`duration` 缺省时用 talker 默认节奏。[L92-L99](../../../raw/current/scripts/components/storyteller.lua#L92-L99)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖 storyteller.lua 全文；核对 talker 的 `donetalking` 事件、walter 挂载/剧本/守卫、`TELLSTORY` 动作与 componentactions 过滤、Woby 与便携营火两处读取；未覆盖 talker 分段台词的底层播放时序。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 与示例 Lua 语法检查通过（见批记录）。

相关：[[reference/components/talker]]。

## 相关链接

- [[reference/components/stageactor]]：舞台演员组件（表演面，与讲故事组件区分）。
