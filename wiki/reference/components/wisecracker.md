---
title: "wisecracker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家自动台词组件：批量监听状态事件经 talker 播报 ANNOUNCE 台词，含吃食评价、明暗切换与大量节流的一次性提示。"
aliases: ["自动播报", "ANNOUNCE 台词", "wisecracker 组件"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/wisecracker.lua", "prefabs/player_common.lua", "components/charlie_tracker.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wisecracker

## 快速判断

- 职责与适用：玩家组件，纯"事件→`talker:Say(GetString(inst, ...))`"桥接层；台词条目按角色 GetString 取本地化串。
- 优先使用：想让玩家对新状态出声 → 挂同款事件监听或补挂到本组件模式；屏蔽某类播报 → 移除组件（台词层无开关）。
- 重要副作用：每帧 `OnUpdate` 监视明暗切换（进光/进暗各需 LightWatcher 停留 0.5s，:545-563）；foodbuff/fishbuff 台词按优先级缓存一次（:307-319、:341-352）。
- 修改边界：台词内容在 `GetString`/角色 speech 文件内，本组件只决定触发；Wortox 专属台词按 `soulstealer` 标签挂载（:242-297）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnUpdate` | `(dt) -> nil` | 明暗播报（ANNOUNCE_ENTER_LIGHT/DARK）与积压的 food/fish buff 台词出队（:545-578）。 |
| oneat 处理（局部） | `(data) -> nil` | 优先级：变质→痛苦（回血负+理智≤0 且非强胃/零吸收/无食材亲和）→新鲜时按 masterfood/prepared/raw/preserved/cooked 细分（masterchef）→ SAME_OLD_1..5（foodmemory 计数）→ stale/spoiled（:10-58）。 |
| 节流播报组（局部） | 各事件闭包 | repelled 5s、soultoofew/soultoomany 30s、shadowthrall 咬 10s+全局共享、woby 台词按 msgid 区分冷却、stompshrouden 3s 全员共享+10s 个人（:90-98、:253-281、:366-437、:510-521）。 |
| 延迟播报组（局部） | DoTaskInTime 桥 | 椅子着火 0.5s、gelblob 1.6s、shadow assist 2-4s、dreadstone 0.5-0.8s 等（:331-333、:359-364、:481-490、:529-538）。 |
| quagmire 挂载 | `event_server_data("quagmire", "components/wisecracker").AddQuagmireEventListeners(inst)` | 仅该游戏模式加载额外监听（:540-542）。 |

## 关键流程：台词触发的三类模式

1. 直触发：hungerdelta 越过 `HUNGRY_THRESH`、startfreezing/startoverheating、burnt、snared、wormholespit 等即 Say（:127-168）。
2. 节流触发：带 GetTime 戳的闭包，同事件在冷却内静默；`ms_stompshrouden` 用文件级 `shared_laststompshrouden` 做全员共享冷却（:1、:510-521）。
3. 延迟触发：DoTaskInTime 后校验再播（shadow assist 还要求在 vault 房间内，:473-490）。

## 典型接入

### 给玩家新增一条事件播报（复用 talker，不依赖本组件）

- 前提与选择：本组件监听集是写死的；mod 新台词直接监听事件并调 `inst.components.talker:Say(GetString(inst, "..."))`，与组件播报共存（talker 自身排队）。
- 生效与撤销：挂载点在 player_common.lua:2779（组件随玩家创建）；移除组件即静默全部默认播报。
- 原版依据：components/wisecracker.lua:10-58（oneat 全链路）。

```lua
-- 在角色或全局后初始化里给玩家加自定义播报（模式同 wisecracker）
inst:ListenForEvent("my_custom_event", function(inst, data)
    if inst.components.talker ~= nil then
        inst.components.talker:Say(GetString(inst, "ANNOUNCE_MY_EVENT"))
    end
end)
```

- 调整与验证：`GetString` 需角色 speech 表含该键，缺键返回 nil 会导致 Say 报错；节流参照 `repelled`（:90-98）。

## 常见陷阱与限制

- 本组件不存档：全部监听在构造时挂载，无 OnSave/OnLoad；台词状态（冷却戳）也不跨存档。
- `charlie_tracker` 硬依赖：grue 三事件的 `IsCharlieDefeated` 直接访问 `TheWorld.components.charlie_tracker`（:104-106），缺该组件的世界在这些事件触发时会报错（charlie_tracker.md 已有页面）。
- oneat 判定链顺序敏感：strongstomach 分支的 `and/or` 优先级（:17-21）决定了怪物肉豁免与零吸收豁免的组合方式，改写时需整段照抄再调。
- 大量监听无移除路径：组件无 OnRemoveFromEntity，事件回调与玩家同生命周期；动态装卸需自行 RemoveEventCallback。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 580 行；确认 charlie_tracker.lua 消费（:104-106）、quagmire 分支（:540-542）；挂载点与 talker 组件行为按 player_common 与 talker 页核对。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/talker]]
- [[reference/components/foodmemory]]
