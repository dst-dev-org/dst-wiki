---
title: "sentientaxe"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "露西（会说话的斧子）人格组件：跟踪持有者并按场景（拾起/砍伐/雕刻/变形/掉落）经 lucy_classified 播放台词。"
aliases: ["会说话的斧子", "sentientaxe", "露西台词", "Lucy 人格"]
tags: [component, callback, item]
dst_build: "steam-buildid:26230496"
sources: ["components/sentientaxe.lua", "prefabs/lucy.lua", "components/possessedaxe.lua", "prefabs/woodie.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sentientaxe

## 快速判断

- 职责与适用：挂在 `lucy` 斧子 prefab 上的主模拟组件；维护 `owner`（当前持有者 Woodie）与两套发言任务（即时 Say、定时闲聊），台词全部来自 `STRINGS.LUCY` 表。[安装](../../../raw/current/scripts/prefabs/lucy.lua#L187)
- 优先使用：让露西说话用 `Say(list, sound_override, delay)`（内部走 `lucy_classified:Say` 实现客户端表现）；持有者变更由 `possessedaxe` 组件推 `axepossessedbyplayer` 驱动 `SetOwner`。[发言](../../../raw/current/scripts/components/sentientaxe.lua#L148-L164) [归属事件](../../../raw/current/scripts/components/possessedaxe.lua#L157)
- 重要副作用：SetOwner 建立后监听持有者的 builditem/finishedwork/wereeaterchanged/startwereplayer/stopwereplayer 与斧子的 ondropped/equipped；闲聊定时器 10~15 秒自续，发言后 60~120 秒重排。[监听组](../../../raw/current/scripts/components/sentientaxe.lua#L73-L85)
- 修改边界：依赖 `lucy_classified`（lucy 工厂生成并挂在 inst 上）与 `equippable`/`inventoryitem`；换成其他宿主没有 classified 就静默不出声。[classified](../../../raw/current/scripts/prefabs/lucy.lua#L190-L194)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetOwner(owner)` | `owner: 玩家或 nil；事件驱动` | 换主：取消旧任务/监听、重置 warnlevel/waslow/`_lastcarvingtalks`、建新监听；已装备时立即说拾起台词并排闲聊。[实现](../../../raw/current/scripts/components/sentientaxe.lua#L50-L87) |
| `Say(list, sound_override?, delay?)` | `list: 字符串表` | 带 delay 时转 `say_task` 延迟且到期还要过 `ShouldMakeConversation`；即时路径先经 `lucy_classified:Say`（随机一条），再排 60+rand*60 秒闲聊。[实现](../../../raw/current/scripts/components/sentientaxe.lua#L148-L164) [classified 生成](../../../raw/current/scripts/prefabs/lucy.lua#L190-L194) |
| `Say` 场景触发 | 事件回调组 | 掉落（2 帧延迟）、装备拾起、雕刻物品（`carve_<recipe>` 串，`TUNING.LUCY_CARVING_TALK_COOLDOWN` 限频）、CHOP 完工、海狸值上升警告、Woodie 变形往返。[雕刻](../../../raw/current/scripts/components/sentientaxe.lua#L89-L101) [变形](../../../raw/current/scripts/components/sentientaxe.lua#L122-L138) [变形事件源](../../../raw/current/scripts/prefabs/woodie.lua#L1072-L1076) |
| `ShouldMakeConversation()` | `-> boolean` | 持有者存活、非变身中/变形态/鬼魂才闲聊；不满足则定时器顺延重排。[实现](../../../raw/current/scripts/components/sentientaxe.lua#L171-L185) |
| `MakeConversation()` | 闲聊主体 | 按位置分支：地上（15 距离内才说）、已装备、在背包（容器持有者是 owner）。说完必然重排下一轮。[实现](../../../raw/current/scripts/components/sentientaxe.lua#L187-L210) |

## 典型接入

### 触发一句露西台词

- 前提与选择：主模拟端；list 是 `STRINGS.LUCY.*` 的台词数组形式（Say 内部 `math.random(#list)` 抽一条），自定义台词传同结构表即可，不必改原版表。[抽取](../../../raw/current/scripts/components/sentientaxe.lua#L158-L159)

```lua
local function LucySay(inst, list, sound)
    local axe = inst.components.sentientaxe
    if TheWorld.ismastersim and axe ~= nil then
        axe:Say(list, sound)
    end
end
```

- 调整与验证：delay 参数只走 `say_task`；两次 Say 会互相取消前一个任务（单槽），不要用它做并排多句播报。[单槽](../../../raw/current/scripts/components/sentientaxe.lua#L148-L156)

## 常见陷阱与限制

- **换主事件来自 possessedaxe：** `axepossessedbyplayer`/`axerejectedowner`/`axerejectedotheraxe` 由斧子附身组件推；直接 GiveItem 给别人不会触发拒绝台词链，除非走过 possessedaxe 的归属判定。[事件源](../../../raw/current/scripts/components/possessedaxe.lua#L149)
- **无 owner 时几乎全哑：** SetOwner(nil) 拆掉全部监听；掉落台词（toground）也挂在 owner 存在期间的 ondropped 上，未被认领过的斧子不会说话。[监听生命周期](../../../raw/current/scripts/components/sentientaxe.lua#L60-L72)
- **闲聊可能被静默吞掉：** 延迟任务到期时 `ShouldMakeConversation` 不通过就整句丢弃（不是顺延重说）；轮询该组件无法保证听到台词。[门禁](../../../raw/current/scripts/components/sentientaxe.lua#L140-L146)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（212 行），核验 lucy 工厂安装与 classified 生成、possessedaxe 归属事件源、woodie 变形事件推送。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。
