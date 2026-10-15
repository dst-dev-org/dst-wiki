---
title: "进食灵魂特效（wortox_eat_soul_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "进食灵魂物品时挂在进食者身上的进食动画：纯动画实体，骑乘换 mounted_eat 六面向，动画播完或进食状态退出时删除。"
aliases: ["wortox_eat_soul_fx", "进食灵魂特效", "吃灵魂特效", "灵魂进食动画"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/wortox_eat_soul_fx.lua", "stategraphs/SGwilson.lua", "stategraphs/SGhermitcrab.lua", "prefabs/wortox_soul.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 进食灵魂特效（wortox_eat_soul_fx）

## 快速判断

- 职责与适用：进食灵魂（wortox_soul，官方译名"灵魂"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L351880-L351882)）时挂在进食者身上的纯动画实体：bank／build `wortox_eat_soul_fx` 播 `eat`，FinalOffset 1；无组件、无伤害／治疗等玩法作用。[工厂](../../../raw/current/scripts/prefabs/wortox_eat_soul_fx.lua#L11-L23)
- 优先使用：复用进食表现走 `SpawnPrefab → SetParent(进食者)`，骑乘再调 `MakeMounted()`；生成点与清理见下文导航表。
- 重要副作用：无玩法作用；用 `DECOR`＋`NOCLICK` 而非 `FX` 标签，注释说明是避免 FX 捕获 mouseover。[标签](../../../raw/current/scripts/prefabs/wortox_eat_soul_fx.lua#L25-L26)
- 修改边界：所有端都有 AnimState（联网复制动画显示）；主模拟侧监听 `animover` 自删并暴露 `MakeMounted`；客户端在 SetPristine 后早退。[构造](../../../raw/current/scripts/prefabs/wortox_eat_soul_fx.lua#L28-L36)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wortox_eat_soul_fx` | fn：Transform＋AnimState＋Network → 播 `eat` → SetPristine → 客户端早退 → 主模拟挂 `animover` 自删、`persists=false`、暴露 `MakeMounted` | 单一注册；`PlayAnimation("eat")` 未显式传循环参数，实际是否重复未核验。[注册](../../../raw/current/scripts/prefabs/wortox_eat_soul_fx.lua#L41)、[主模拟](../../../raw/current/scripts/prefabs/wortox_eat_soul_fx.lua#L34-L36) |

`MakeMounted` 把四面向改六面向并播 `mounted_eat`，供骑乘进食调用。[定义](../../../raw/current/scripts/prefabs/wortox_eat_soul_fx.lua#L6-L9)

标题中文为已核验效果描述，非官方译名（NAMES／SKIN_NAMES 未发现该 FX 注册名条目）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 宿主状态：SGwilson `eat` | `feed.components.soul ~= nil` 时生成并 `SetParent` 到玩家；骑乘分支再调 `MakeMounted()`；状态 onexit 删除残留 fx | 自食与被喂食都走该状态；28 帧经 `souleater:EatSoul` 消耗灵魂 | [生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6445-L6451)、[消耗](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6472-L6476)、[onexit 清理](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6526-L6527)、[[reference/components/souleater#关键接口]] |
| 宿主状态：SGwilson `float_eat` | 漂浮进食同样判 `feed.components.soul` 生成挂载，无骑乘分支；引用传入 float 状态并在中断／结束时删除 | 漂浮形态专用 | [生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26400-L26403)、[清理](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26459-L26460) |
| 宿主状态：SGhermitcrab `eat` | 被喂灵魂时生成，先对齐螃蟹朝向再挂载；状态退出时删除 | 隐居螃蟹吃灵魂路径 | [生成](../../../raw/current/scripts/stategraphs/SGhermitcrab.lua#L1826-L1830)、[清理](../../../raw/current/scripts/stategraphs/SGhermitcrab.lua#L1876-L1877) |
| soul 组件判定 | 生成条件 `feed.components.soul` 来自灵魂物品的 soul 组件 | 物品侧不限制进食者身份 | [组件挂载](../../../raw/current/scripts/prefabs/wortox_soul.lua#L318) |
| 动画资源 | bank／build `wortox_eat_soul_fx`，anim `eat`／`mounted_eat` | 资源包内容未解包 | [声明](../../../raw/current/scripts/prefabs/wortox_eat_soul_fx.lua#L1-L3)、[设置](../../../raw/current/scripts/prefabs/wortox_eat_soul_fx.lua#L20-L22) |

## 典型接入

### 复用进食灵魂表现

- 前提与选择：目标模组文件任意主模拟侧脚本；生成后父级对齐进食者。
- 生效与撤销：正常由 `animover` 自删；宿主侧另有 statemem 引用兜底删除，两条清理路径独立成立。

```lua
local fx = SpawnPrefab("wortox_eat_soul_fx")
fx.entity:SetParent(eater.entity)
if eater.components.rider ~= nil and eater.components.rider:IsRiding() then
    fx:MakeMounted() -- 六面向 + mounted_eat
end
```

- 调整与验证：换 anim 需另建工厂；`MakeMounted` 是主模拟侧暴露函数，客户端调用无效。[原版调用](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6446-L6450)

## 常见陷阱与限制

- **误区：只有沃拓克斯吃灵魂才有特效**：状态图只判 `feed.components.soul`，接入这些状态的玩家与隐居螃蟹吃灵魂时可生成本特效，不能扩展成所有自定义进食者都自动接入。[hermitcrab 判定](../../../raw/current/scripts/stategraphs/SGhermitcrab.lua#L1826-L1830)
- **边界：骑乘姿态由调用方选择**：fx 默认四面向 `eat`；SGwilson 骑乘分支显式调 `MakeMounted`，`float_eat` 与 hermitcrab 不调。[骑乘分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6448-L6450)
- **边界：双路径清理但不重复删除**：`animover` 自删与状态 onexit 的 `statemem.soulfx:Remove()` 都在主模拟执行，本页未验证两条路径的先后顺序，不保证总由 animover 先触发。[onexit](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6526-L6527)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；41 行工厂全文、SGwilson eat／float_eat、SGhermitcrab eat 生成与清理点、soul 组件判定、译名检索已核验；动画资源内容、eat 动画时长与清理竞态未核验。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
