---
title: "clientpickupsoundsuppressor"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "短暂把物品的 pickupsound 置为 NONE 并经 net_bool 同步到客户端，抑制复制／转移时的拾取音效。"
aliases: ["拾取音效抑制", "静音拾取", "clientpickupsoundsuppressor"]
tags: [component, server-client]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/clientpickupsoundsuppressor.lua", "actions.lua", "prefabs/player_common.lua", "prefabs/slingshot.lua", "prefabs/slingshotparts.lua", "prefabs/willow_ember.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# clientpickupsoundsuppressor

## 快速判断

- 职责与适用：抑制一次拾取音效：服务器调 `IgnoreNextPickupSound()` 后，组件把 `inst.pickupsound` 改成 `"NONE"` 两帧再还原；被拾取物品上的该字段由客户端 `OnGotNewItem` 播放（[播放点](../../../raw/current/scripts/prefabs/player_common.lua#L479-L483)），所以静音必须发生在客户端。
- 优先使用：在服务端把物品装进／卸出容器、即将触发 GiveItem 复制到客户端前调用 `IgnoreNextPickupSound()`。原版用在 `ACTIONS.CONTAINER_INSTALL_ITEM` 的卸下与槽位交换分支（[调用1](../../../raw/current/scripts/actions.lua#L6381-L6382)、[调用2](../../../raw/current/scripts/actions.lua#L6401-L6402)）。
- 重要副作用：用 `net_bool`（不是 net_event）同步 `_ignorenext`，true／false 都会用；只有客户端监听该事件并执行静音（[构造](../../../raw/current/scripts/components/clientpickupsoundsuppressor.lua#L20-L29)）。静音持续 `2 * FRAMES` 后由 `DoStaticTaskInTime` 还原（[L14-L16](../../../raw/current/scripts/components/clientpickupsoundsuppressor.lua#L14-L16)）。
- 修改边界：组件要在服务器与客户端同时存在（原版三处 AddComponent 都在 `SetPristine` 之前的公共段：[slingshot.lua#L505](../../../raw/current/scripts/prefabs/slingshot.lua#L505)、[slingshotparts.lua#L70](../../../raw/current/scripts/prefabs/slingshotparts.lua#L70)、[willow_ember.lua#L960](../../../raw/current/scripts/prefabs/willow_ember.lua#L960)）；只装在服务端时客户端不会静音。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IgnoreNextPickupSound()` | 无返回 | 服务端调用；`_ignorenext` 置为 `GetTimeAlive() <= 0` 的结果并经 net_bool 下发（`set_local`＋`set`）。[L31-L35](../../../raw/current/scripts/components/clientpickupsoundsuppressor.lua#L31-L35) |
| `_ignorenext` | net_bool，名 `clientpickupsoundsuppressor._ignorenext` | 构造创建；客户端注册同名事件监听，仅在非 mastersim 时挂。[L24-L28](../../../raw/current/scripts/components/clientpickupsoundsuppressor.lua#L24-L28) |
| `inst.pickupsound` | string 或 nil（宿主字段，非组件字段） | 组件读写该字段：置 `"NONE"` 两帧后还原原值；玩家拾取音效消费它。[消费](../../../raw/current/scripts/prefabs/player_common.lua#L480) |

## 常见陷阱与限制

- 客户端刚复制出来、且 `_ignorenext` 为 false 的事件会被直接忽略（服务端旧物品、客户端新实体 replication 的场景），这是源码注释解释的设计：避免覆盖"真的在生成瞬间推送"的事件（[L7-L13](../../../raw/current/scripts/components/clientpickupsoundsuppressor.lua#L7-L13)）。因此对已存在较久的物品调用本组件，静音能否生效取决于客户端实体年龄，不能保证 100% 抑制。
- 只静音 `pickupsound ~= "NONE"` 的物品；本来就没有 pickupsound 的宿主调用无效果（[L6](../../../raw/current/scripts/components/clientpickupsoundsuppressor.lua#L6)）。
- 两帧窗口很短：如果物品在 2 帧内被连续转移，第二次拾取不会被抑制。

### 原版参照

- 原版唯一调用方是 `ACTIONS.CONTAINER_INSTALL_ITEM`：在卸下（RemoveItem 后 GiveItem 前）与容器内换槽两个分支，对被移动物品调 `IgnoreNextPickupSound()`，并把 `inventory.ignoresound` 置位挡服务端侧音效（[actions.lua#L6376-L6406](../../../raw/current/scripts/actions.lua#L6376-L6406)）。模组复用时保持同一顺序：先调抑制、再做物品转移；宿主 prefab 必须在公共段 AddComponent 让客户端也持有组件。未单独运行验证。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/clientpickupsoundsuppressor.lua（37 行），核对三个原版宿主的公共段 AddComponent、actions.lua 两处服务端调用、player_common.lua 的 pickupsound 播放链。未覆盖实际网络时序表现。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；48 行、4956 字节，simple 档内）；validate_source_links.py --strict 通过（14 处源码引用，0 error／0 warning）；本页无完整 Lua 代码块。
