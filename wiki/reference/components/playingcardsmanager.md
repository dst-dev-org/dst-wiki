---
title: "playingcardsmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "森林世界的主模拟扑克牌 ID 池管理器：为卡组与散卡分配牌 ID 并在实体移除时回收，供抽牌、发牌与补登记使用。"
aliases: ["扑克牌管理器", "playingcardsmanager", "发牌", "牌 ID 回收"]
tags: [component, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/playingcardsmanager.lua", "prefabs/forest.lua", "prefabs/deck_of_cards.lua", "prefabs/balatro_machine.lua", "actions.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# playingcardsmanager

## 快速判断

- 职责与适用：挂在森林世界实体上的主模拟组件（构造即断言 `TheWorld.ismastersim`），维护全局可用牌 ID 池并跟踪卡组／散卡实体；发牌从池取 ID，实体移除时回收。[断言](../../../raw/current/scripts/components/playingcardsmanager.lua#L7) [安装](../../../raw/current/scripts/prefabs/forest.lua#L731-L732)
- 牌 ID 语义：`suit * 100 + pip`（`PIP_DIGIT_BARRIER = 100`），初始化按 `TUNING.PLAYINGCARDS_NUM_SUITS`（4）× `TUNING.PLAYINGCARDS_NUM_PIPS` 全量枚举；池空后退化为纯随机 ID 且不再入池。[构造](../../../raw/current/scripts/components/playingcardsmanager.lua#L16-L29) [回退](../../../raw/current/scripts/components/playingcardsmanager.lua#L109-L113)
- 优先使用：从卡组抽牌走 `ACTIONS.DRAW_FROM_DECK`（组件发牌＋回收）；新生成且未定 ID 的扑克牌由卡组工厂调用 `RegisterPlayingCard` 补登记。[抽牌](../../../raw/current/scripts/actions.lua#L6468-L6477) [登记](../../../raw/current/scripts/prefabs/deck_of_cards.lua#L340-L347)
- 修改边界：仅主模拟端存在；客户端消费方都自带无组件兜底分支。组件本身无网络字段，牌 ID 的表现由 `playingcard` 组件承担。[客户端兜底](../../../raw/current/scripts/prefabs/deck_of_cards.lua#L349-L351)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `MakeDeck(size)` | `size: number? -> deck 实体` | 生成 `deck_of_cards`，随机抽 `size`（默认且最小 1）个 ID 交给 `deckcontainer:AddCards`，监听 onremove 并加入跟踪表。[实现](../../../raw/current/scripts/components/playingcardsmanager.lua#L104-L122) |
| `MakePlayingCard(id, facedown)` | `id: number?、facedown: boolean? -> playing_card 实体` | `id` 缺省时从池随机取；`facedown` 置 `card._faceup = false`；同样登记 onremove 回收。[实现](../../../raw/current/scripts/components/playingcardsmanager.lua#L124-L141) |
| `RegisterPlayingCard(card)` | `card: 实体 -> card 实体` | 对已存在但 ID 未定的牌补随机 ID 并登记回收；`deck_of_cards` 初始化时对 ID=-1 的牌调用。[实现](../../../raw/current/scripts/components/playingcardsmanager.lua#L143-L151) [调用](../../../raw/current/scripts/prefabs/deck_of_cards.lua#L346-L347) |
| `OnSave`／`OnLoad`／`LoadPostPass` | 标准存读钩子 | 持久化 `available_ids`；有跟踪实体时返回 `living_decks` GUID 列表与 ents，载入后对每个实体恢复 onremove 监听。[实现](../../../raw/current/scripts/components/playingcardsmanager.lua#L65-L98) |
| `Debug_ResetAvailableIDs()` | `-> nil` | 清空重建 ID 池；当前实现引用未定义全局，调用即报错，见陷阱。[实现](../../../raw/current/scripts/components/playingcardsmanager.lua#L172-L179) |

## 典型接入

### 服务端给玩家发一张随机扑克牌

- 前提与选择：森林世界主模拟端的服务器代码（如世界后初始化）经 `TheWorld.components.playingcardsmanager` 访问；走组件保证 ID 池与回收一致，不走组件则失去全局唯一性。
- 生效与撤销：牌实体移除时 `card_onremove` 回收 ID；卡组移除时 `deck_onremove` 整组回收。[回收](../../../raw/current/scripts/components/playingcardsmanager.lua#L37-L59)

```lua
local function GiveRandomCard(player)
    if not TheWorld.ismastersim then
        return nil
    end
    local manager = TheWorld.components.playingcardsmanager
    if manager == nil then
        return nil
    end
    local card = manager:MakePlayingCard(nil, false)
    if card ~= nil then
        player.components.inventory:GiveItem(card)
    end
    return card
end
```

- 调整与验证：`facedown = true` 生成背面牌（原版 balatro_machine 奖励路径即如此）；完整卡组抽牌链路见 `ACTIONS.DRAW_FROM_DECK`。[背面奖励](../../../raw/current/scripts/prefabs/balatro_machine.lua#L91-L97)

## 常见陷阱与限制

- **Debug_ResetAvailableIDs 是坏的：** 它读全局 `NUM_SUITS`／`NUM_PIPS`（本快照从未定义），而初始化读 `TUNING.PLAYINGCARDS_*`；直接调用抛 `attempt to index a nil value (global 'NUM_SUITS')`。[对比](../../../raw/current/scripts/components/playingcardsmanager.lua#L25-L29) [常量位置](../../../raw/current/scripts/tuning.lua#L8642)
- **池空不补：** `available_ids` 耗尽后发牌退化为纯随机，且这种 ID 不入池、不回收；只有登记过 onremove 的实体才在移除时归还 ID。[回退](../../../raw/current/scripts/components/playingcardsmanager.lua#L109-L113)
- **跟踪表混装卡组与散卡：** `decks` 同时登记 deck_of_cards 与 playing_card 实体，OnSave 的 `living_decks` 两者都会出现；这是设计行为，LoadPostPass 按实体类型各自的 onremove 回调处理。[存档](../../../raw/current/scripts/components/playingcardsmanager.lua#L65-L82)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（181 行）、forest.lua 安装点、deck_of_cards／balatro_machine／actions.lua 消费路径与 tuning.lua 常量。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。

相关：[[reference/components/playingcard]]、[[reference/components/deckcontainer]]
