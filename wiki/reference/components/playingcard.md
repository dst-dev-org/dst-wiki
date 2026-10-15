---
title: "playingcard"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "扑克牌实体的花色与点数编码：单数字 id 存 suit*barrier+pips，供牌堆、balatro 机器与 RETURN 动作读写。"
aliases: ["扑克牌", "卡牌组件", "playingcard"]
tags: [component, item]
dst_build: "steam-buildid:26230496"
sources: ["components/playingcard.lua", "components/playingcardsmanager.lua", "prefabs/deck_of_cards.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# playingcard

## 快速判断

- 职责与适用：给 `playing_card` 实体记录一个整数 id（花色 × 位屏障 + 点数），提供读写与分解；id 语义由 playingcardsmanager 发放（[发放](../../../raw/current/scripts/components/playingcardsmanager.lua#L131-L134)）。
- 优先使用：模组要"发一张随机牌"直接走 `playingcardsmanager:MakePlayingCard`，别自己拼 id；已知 id 洗牌才用 SetID。
- 重要副作用：构造时加 `playingcard` tag，供动作系统与组件动作识别（[tag](../../../raw/current/scripts/components/playingcard.lua#L25)）；移除组件时摘除。
- 修改边界：id 只入存档不自动联网同步——客户端显示走宿主 `_card_id` netvar（[宿主](../../../raw/current/scripts/prefabs/deck_of_cards.lua#L314-L324)），组件本身无网络变量。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetID(id)` | `id: number` | 写入 `_card_id` 并触发宿主 `on_new_id(inst, id)` 回调（可选）。[实现](../../../raw/current/scripts/components/playingcard.lua#L37-L43) |
| `GetID()` | → `number`（初始 -1 表示未发牌） | 返回原始 id。[实现](../../../raw/current/scripts/components/playingcard.lua#L45-L47) |
| `GetSuit()`／`GetPips()` | → `number` | 分别取 `floor(id/barrier)` 与 `fmod(id, barrier)`；id ≤ 0 时返回 0。[实现](../../../raw/current/scripts/components/playingcard.lua#L49-L57) |
| `pip_count` | `number`，默认 `TUNING.PLAYINGCARDS_NUM_PIPS`（13，tuning.lua:8643） | watcher 按点数位数重算 `pip_digit_barrier`（两位数点数时为 100）。[watcher](../../../raw/current/scripts/components/playingcard.lua#L1-L14) |
| `OnSave`／`OnLoad(data)` | 存读钩子 | 保存 `card_id`；OnLoad 直接赋值，不校验缺省。[实现](../../../raw/current/scripts/components/playingcard.lua#L64-L70) |

## 典型接入

### 发一张随机扑克牌

- 前提与选择：`TheWorld` 需有 playingcardsmanager（管理可用 id 池）；balatro 机器就是这么拿牌的（[调用](../../../raw/current/scripts/prefabs/balatro_machine.lua#L97)）。
- 生效与撤销：卡牌移除时 playingcardsmanager 通过其 onremove 监听回收 id；SetID 重发不会自动回收旧 id。
- 原版依据：MakePlayingCard / RegisterPlayingCard（[manager](../../../raw/current/scripts/components/playingcardsmanager.lua#L128-L146)）；牌堆回收走 GetID（[deck_of_cards](../../../raw/current/scripts/prefabs/deck_of_cards.lua#L337)）。

```lua
-- 从牌管理器发一张随机牌（facedown 为背面朝上）
local card = TheWorld.components.playingcardsmanager:MakePlayingCard(nil, true)
local suit, pips = card.components.playingcard:GetSuitAndPip()
```

- 调整与验证：花色数 4、点数 13（tuning.lua:8642-8643）；`GetSuitAndPip` 只在 id > 0 时有意义，-1/0 表示未发牌或背面。

## 常见陷阱与限制

- OnLoad 直接 `self._card_id = data.card_id`（[实现](../../../raw/current/scripts/components/playingcard.lua#L68-L70)）：存档数据存在但缺 card_id 键时变 nil，后续 GetSuit 的 `> 0` 比较会报错；模组自建卡牌需保证 OnSave/OnLoad 成对。
- `pip_count` watcher 只在 `_card_id ~= -1` 时重算位屏障（[实现](../../../raw/current/scripts/components/playingcard.lua#L2-L13)），未发牌的卡不维护 barrier，属预期。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 playingcardsmanager／deck_of_cards／actions 消费链、tuning 取值；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
