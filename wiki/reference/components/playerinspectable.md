---
title: "playerinspectable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家外观检查组件：把玩家装备皮肤与技能选择写进引擎网络变量，供头像弹窗等远端界面读取玩家形象。"
aliases: ["玩家检查", "playerinspectable", "头像弹窗数据"]
tags: [component, network]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/playerinspectable.lua", "prefabs/player_common.lua", "prefabs/slingshot.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# playerinspectable

## 快速判断

- 职责与适用：挂在玩家实体上，监听 `equip/unequip` 把每个装备栏的皮肤名写入 `Network:SetPlayerEquip`，监听技能选择三事件把技能选择写入 `Network:SetPlayerSkillSelection`；用途注释为"玩家头像弹窗检查"（Player avatar popup inspection）。[监听集合](../../../raw/current/scripts/components/playerinspectable.lua#L20-L28)
- 优先使用：不要手动挂载。原版在 ([player_common.lua:2728-2729](../../../raw/current/scripts/prefabs/player_common.lua#L2728-L2729) 随玩家构造加入，所有玩家实体自动具备。
- 重要副作用：写入的是引擎级网络变量（SetPlayerEquip/SetPlayerSkillSelection），客户端无需 classified 即可读；这就是头像弹窗能在远端显示装备皮肤的基础。
- 修改边界：组件只做"装备/技能 → 网络变量"的单向同步；读取侧逻辑不归本组件，也不监听穿戴栏之外的物品变化。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `equip` / `unequip`（监听） | `data: {eslot, item}` | 装备时 `SetPlayerEquip(EquipSlot.ToID(data.eslot), 皮肤名 or item.playerinspectable_override or item.prefab)`；卸下时写空串。[OnEquip](../../../raw/current/scripts/components/playerinspectable.lua#L3-L9) |
| `onactivateskill_server` / `ondeactivateskill_server` / `onsetskillselection_server`（监听） | `data`（未使用） | 有 `skilltreeupdater` 时取 `GetPlayerSkillSelection()` 全量写入网络变量；无 updater 直接返回。[OnSkillSelectionUpdated](../../../raw/current/scripts/components/playerinspectable.lua#L11-L18) |
| `item.playerinspectable_override` | `string \| nil`（物品侧字段） | 物品可声明覆盖名替代 prefab 名上报；原版使用者是 slingshot 的三个变体。[覆盖示例](../../../raw/current/scripts/prefabs/slingshot.lua#L647) |

## 典型接入

### 让自定义装备在头像弹窗显示自定义名（原版参照）

- 前提与选择：服务端；装备物品侧设置 `playerinspectable_override` 字段即可，无需改玩家侧代码；同源模型的不同皮肤/变体用它归并显示名。[优先级](../../../raw/current/scripts/components/playerinspectable.lua#L4)
- 生效与撤销：字段在每次 equip 事件时读取，改值后需重新穿脱装备才生效；卸下装备该槽位上报空串。
- 原版依据：slingshot 在多个构造分支统一设 `inst.playerinspectable_override = "slingshot"`（([slingshot.lua:645-647](../../../raw/current/scripts/prefabs/slingshot.lua#L645-L647)，另见 743/767 行），保证三变体在弹窗里同名。

```lua
-- 服务端：物品构造时声明覆盖名
inst.playerinspectable_override = "my_custom_armor"
```

- 调整与验证：覆盖名是任意字符串，引擎侧不校验 prefab 存在性；显示效果取决于读取界面的字符串/皮肤解析，需在头像弹窗实测确认。

## 常见陷阱与限制

- **期待组件上报非装备数据：** 只同步装备栏与技能选择；玩家名牌、表情等由其他网络通道处理。新增同步项需改组件本体。[单一职责](../../../raw/current/scripts/components/playerinspectable.lua#L20-L28)
- **绕过 equip 事件改穿戴：** 直接改 `components.inventory` 的装备表而不经 equippable 的 equip/unequip 事件时，网络变量不刷新，弹窗显示旧值；组件依赖事件驱动。[事件依赖](../../../raw/current/scripts/components/playerinspectable.lua#L23-L24)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（30 行），核对 player_common.lua 挂载位置与注释、slingshot.lua 三处 override 用法；grep 确认 `playerinspectable_override` 全部使用者。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；头像弹窗显示未实测。

相关：[[reference/components/inspectable]]
