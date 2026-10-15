---
title: "singable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "战斗歌谣演唱组件：让带 songdata 的歌谣物品经 ACTIONS.SING 进入歌者灵感系统，原版用于韦斯变的战斗歌单。"
aliases: ["可演唱", "singable", "战斗歌谣"]
tags: [component, item]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/singable.lua", "prefabs/battlesongs.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# singable

## 快速判断

- 职责与适用：挂在歌谣物品（battlesong prefab）上，把"演唱"动作转成对歌者 `singinginspiration` 组件的 `AddSong` 注册；本体只持有一个可选回调。[Sing](../../../raw/current/scripts/components/singable.lua#L10-L25)
- 优先使用：做可演唱的增益歌/歌纸时挂本组件并在实体上提供 `songdata` 表；原版唯一挂载点是 [battlesongs 的 song_fn 工厂](../../../raw/current/scripts/prefabs/battlesongs.lua#L147)。
- 重要副作用：`Sing` 硬性要求歌者有 `singinginspiration` 组件，缺失时只打印日志并返回，不消耗物品；演唱后歌谣仍留在物品栏（finiteuses 消耗逻辑已被注释停用）。[守卫](../../../raw/current/scripts/components/singable.lua#L11-L14)
- 修改边界：动作入口 `ACTIONS.SING` 由 componentactions 限定 `battlesinger` 标签的角色（韦斯变）；组件本身不做角色检查。[动作过滤](../../../raw/current/scripts/componentactions.lua#L2899-L2903)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetOnSing(onsingfn)` | `fn(inst, singer) -> nil` | 演唱时先于 AddSong 调用；原版 song_fn 未注册该回调。[实现](../../../raw/current/scripts/components/singable.lua#L6-L8) |
| `Sing(singer)` | `singer: entity -> nil` | 歌者无 `singinginspiration` 时打印并返回；成功则调 onsingfn 并 `AddSong(self.inst.songdata, nil, self.inst)`。[实现](../../../raw/current/scripts/components/singable.lua#L10-L25) |
| 宿主字段 `songdata` | `table`（battlesong 数据） | 组件不持有歌数据，直接读 `self.inst.songdata`；prefab 构造时赋值。[赋值](../../../raw/current/scripts/prefabs/battlesongs.lua#L122) |

## 典型接入

### 复刻战斗歌谣物品（原版参照）

- 前提与选择：服务端；物品侧用 battlesongs 的 `song_fn` 工厂模式（网络实体 + `songdata` + singable），歌效果写在 `songdata` 的 `ONAPPLY/ONDETACH/TICK_FN` 等字段里由 singinginspiration 消费。[工厂](../../../raw/current/scripts/prefabs/battlesongs.lua#L105-L147)
- 生效与撤销：玩家（`battlesinger`）经 ACTIONS.SING 触发：先查 `IsSongActive` 防重复、`CanAddSong` 查灵感容量，通过后才 `Sing`；歌的移除由灵感系统按 buff 生命周期处理。[动作实现](../../../raw/current/scripts/actions.lua#L5242-L5255)
- 原版依据：`songdata.INSTANT` 的歌走 rechargeable 冷却而非持续 buff（([battlesongs.lua:139-142](../../../raw/current/scripts/prefabs/battlesongs.lua#L139-L142)），组件本身两种模式无差别。

```lua
-- 服务端：自定义歌谣物品的组件部分（等价 song_fn）
inst.songdata = { ONAPPLY = function(inst, target) end, SONG_FX = nil }
inst:AddComponent("singable")
```

- 调整与验证：`SetOnSing` 可用于演唱时的即时副作用（原版未用）；歌名/图鉴显示与 `songdata` 其他键由灵感系统与 UI 消费，需对照 battlesongs 内现成歌目。

## 常见陷阱与限制

- **给非 battlesinger 角色挂歌：** 组件的 `Sing` 只挡"无 singinginspiration"一种情况；角色资格由 componentactions 的 `battlesinger` 标签决定，绕过 UI 直接 Push 动作仍可被 singinginspiration 拒绝。[过滤](../../../raw/current/scripts/componentactions.lua#L2901-L2903)
- **期待演唱消耗歌纸：** 组件内 finiteuses 调用被注释停用，演唱不消耗物品；耐久/冷却语义由 `songdata.INSTANT`（rechargeable）或灵感系统承担。[注释块](../../../raw/current/scripts/components/singable.lua#L16-L18)
- **songdata 缺失：** `AddSong(self.inst.songdata, ...)` 传入 nil 依赖灵感系统容错；组件不校验 songdata 存在，构造物品时必须先赋值。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（27 行），核对 battlesongs.lua 工厂与 songdata、actions.lua SING 实现、componentactions.lua 角色过滤；grep 确认 `components.singable` 消费者仅 actions.lua。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；演唱链路未实测。
