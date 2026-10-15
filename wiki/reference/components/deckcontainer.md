---
title: "deckcontainer"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "卡牌堆容器：维护 card id 列表，支持增删、按位插取、合并拆分、洗牌反转，空堆自动移除实体，供扑克牌玩法驱动。"
aliases: ["牌组容器", "deckcontainer"]
tags: [component, item, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/deckcontainer.lua", "prefabs/deck_of_cards.lua", "actions.lua", "components/playingcardsmanager.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# deckcontainer

## 快速判断

- 职责与适用：给牌组实体（原版 `deck_of_cards`）维护 card id 有序列表；负责增删、按位访问、两堆合并/拆分、洗牌与反转。原版在 [deck_of_cards.lua](../../../raw/current/scripts/prefabs/deck_of_cards.lua#L263-L268) 挂组件并接回调。
- 优先使用：牌局动作（摸/出/收/合堆）由 actions.lua 的 PLAYING_CARD 系列动作消费（[actions](../../../raw/current/scripts/actions.lua#L6469-L6571)）；批量发牌走 `AddCards`（playingcardsmanager，[L115](../../../raw/current/scripts/components/playingcardsmanager.lua#L115)）。
- 重要副作用：`RemoveCard` 把堆删到 0 张时直接 `inst:Remove()` 实体（[L84-L86](../../../raw/current/scripts/components/deckcontainer.lua#L84-L86)）；构造时加 `deckcontainer` 标签（[L10](../../../raw/current/scripts/components/deckcontainer.lua#L10)）。
- 修改边界：不校验 card id 内容，id 语义由 playingcard 组件与牌局动作定义；回调（on_card_added 等）全由宿主赋值，组件不设默认值。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddCard(id, position)` | `(number, 可选 position) -> void` | 无 position 追加到末尾，有则 `table.insert` 到该位；触发 `on_card_added`。[实现](../../../raw/current/scripts/components/deckcontainer.lua#L44-L54) |
| `AddCards(ids)` | `table -> void` | 批量追加，触发 `on_cards_added_bulk`。[实现](../../../raw/current/scripts/components/deckcontainer.lua#L61-L70) |
| `RemoveCard(position)` | `可选 position -> id` | 仅当 `position < #cards` 按位删，否则删末尾；触发 `on_card_removed`；删空移除实体。[实现](../../../raw/current/scripts/components/deckcontainer.lua#L72-L89) |
| `PeekCard(position)` | `可选 position -> id` | 同样的位置语义返回而不删除。[实现](../../../raw/current/scripts/components/deckcontainer.lua#L91-L98) |
| `MergeDecks(other_deck, moved_count)` | `(deckcontainer, 可选 n) -> void` | 把另一堆前 n 张（默认全堆）并入本堆并从对方移除。[实现](../../../raw/current/scripts/components/deckcontainer.lua#L19-L30) |
| `SplitDeck(source_deck, num_to_get)` | `(deckcontainer, number) -> void` | 从源堆取 num 张后 `Reverse()`，触发 `on_split_deck`。[实现](../../../raw/current/scripts/components/deckcontainer.lua#L32-L41) |
| `Shuffle/Reverse/Count` | `-> void/void/number` | 洗牌触发 `on_deck_shuffled`；反转原地；Count 返回张数。[实现](../../../raw/current/scripts/components/deckcontainer.lua#L100-L113) |
| `AddRandomCard` | `-> void` | `id = 100 * math.random(4) + math.random(13)`，即按花色×点数的编码惯例。[实现](../../../raw/current/scripts/components/deckcontainer.lua#L56-L59) |
| `OnSave/OnLoad` | 存读钩子 | 保存 cards 列表；OnLoad 用 `ConcatArrays` 追加而非覆盖。[实现](../../../raw/current/scripts/components/deckcontainer.lua#L116-L124) |

## 典型接入

### 自定义牌组实体

- 前提与选择：主机端物品 prefab 加组件并接回调；回调签名 `(inst)` 或 `(inst, id/ids)`，由 deck_of_cards 的接线方式可见（[L263-L268](../../../raw/current/scripts/prefabs/deck_of_cards.lua#L263-L268)）。

```lua
local deckcontainer = inst:AddComponent("deckcontainer")
deckcontainer.on_card_removed = function(inst_)
    print(inst_.prefab, "now has", inst_.components.deckcontainer:Count(), "cards")
end
deckcontainer:AddRandomCard()
```

- 调整与验证：卡片 id 编码需与 playingcard 组件的 `GetID` 一致（原版编码见 `AddRandomCard`）；deck_of_cards 的相关页面见 [[reference/components/playingcard]]。

## 常见陷阱与限制

- **position 语义是"越界即末尾"**：`RemoveCard(#cards)` 不会删该位而是删末尾（条件是严格小于，[L74](../../../raw/current/scripts/components/deckcontainer.lua#L74)）；`PeekCard` 同理（[L93](../../../raw/current/scripts/components/deckcontainer.lua#L93)）。
- **OnLoad 是追加**：加载会把存档 cards 拼到现有列表后（[L120-L124](../../../raw/current/scripts/components/deckcontainer.lua#L120-L124)），宿主若在构造时预填了初始牌会重复。
- **删空即移除实体**：需要"空堆留存"的玩法要自己拦截（在 on_card_removed 里判断后补充），组件不提供开关。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、deck_of_cards 挂接、actions.lua 牌局动作消费点、playingcardsmanager 批量发牌。
- 游戏运行：未运行。
