---
title: "savedscale"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "把实体 Transform 缩放自动写入存档并在载入时恢复，适合尸体、鸟粪等生成时带随机缩放的可再生物件。"
aliases: ["savedscale", "缩放存档", "保存实体缩放", "Transform 缩放持久化"]
tags: [component, data]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/savedscale.lua", "prefabs/corpses.lua", "prefabs/guano.lua", "prefabs/rock_ice_temperature.lua", "prefabs/sharkboi_ice_hazard.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# savedscale

## 快速判断

- 职责与适用：唯一职责是实现 `OnSave/OnLoad`，与 EntityScript 的组件持久化协议对接；生成时调用过 `Transform:SetScale` 的实体挂上即可在重载后恢复缩放。[实现](../../../raw/current/scripts/components/savedscale.lua#L5-L19)
- 优先使用：生成侧给随机缩放（如 corpses 按数据表缩放）时挂本组件；无组件时缩放丢失、重载回默认 1。[生成侧缩放](../../../raw/current/scripts/prefabs/corpses.lua#L645-L647)
- 重要副作用：OnSave 只在缩放非默认时产出数据（等比缩放只存 `x`，各轴不等才存 y/z），空表返回 nil——不产生存档垃圾；OnLoad 以 `data.x or 1` 为基准补齐 y/z。[存档规则](../../../raw/current/scripts/components/savedscale.lua#L5-L14)
- 修改边界：组件不主动设缩放，运行中改 Scale 不会同步任何网络变量；恢复只在载入时发生一次。[载入](../../../raw/current/scripts/components/savedscale.lua#L16-L19)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnSave()` | `-> {x=?, y=?, z=?}|nil` | `x` 仅当 `sx ~= 1`；`y` 仅当 `sy ~= sx`；`z` 仅当 `sz ~= sx`。等比缩放(如 1.5,1.5,1.5)只写 x。全默认返回 nil，EntityScript 跳过该键。[实现](../../../raw/current/scripts/components/savedscale.lua#L5-L14) [收集协议](../../../raw/current/scripts/entityscript.lua#L1928-L1935) |
| `OnLoad(data)` | `data: {x?, y?, z?}` -> `nil` | `scale = data.x or 1`，再 `SetScale(scale, data.y or scale, data.z or scale)`；EntityScript 只在有存档数据时分发，data 不会为 nil。[实现](../../../raw/current/scripts/components/savedscale.lua#L16-L19) [分发](../../../raw/current/scripts/entityscript.lua#L1992-L2006) |

## 典型接入

### 让随机缩放的再生物件跨存档保持外观（原版参照）

- 前提与选择：服务端挂载；corpses 在构造时按数据表 `SetScale(scale, scale, scale)`，同批挂 savedscale；guano、rock_ice_temperature、sharkboi_ice_hazard 同构挂载。[corpses 挂载](../../../raw/current/scripts/prefabs/corpses.lua#L748) [guano](../../../raw/current/scripts/prefabs/guano.lua#L117)
- 生效与撤销：组件随实体存读自动工作，无注册回调、无撤销路径；移除组件后已有存档里的 savedscale 键会被 EntityScript 忽略（组件缺失时不分发）。[分发守卫](../../../raw/current/scripts/entityscript.lua#L2005-L2006)
- 原版依据：rocks 冰石等世界再生物件用同一组件维持视觉随机性；不涉及动画缩放或战斗碰撞体缩放。

```lua
-- 服务端：生成时缩放并持久化
inst:AddComponent("savedscale")
local scale = 0.9 + math.random() * 0.3
inst.Transform:SetScale(scale, scale, scale)
```

- 调整与验证：OnSave 语义下等比缩放恢复后各轴一致；若运行期动态改单轴缩放，需确认写入顺序在存档前，组件不做脏标记。

## 常见陷阱与限制

- **把"缩放为 1 但各轴不同"的数据想当然：** 规则是"与 x 比较"而非"与 1 比较"——`y` 只在 `sy ~= sx` 时存；`sx == 1` 且 sy/z 有差异时 x 不存，OnLoad 以 1 为基准恢复，行为正确但自定义解析存档时要按此规则读。[实现](../../../raw/current/scripts/components/savedscale.lua#L9-L12)
- **以为缩放参与客户端同步：** 组件只在存读链路工作；网络端缩放需另行处理（Transform 缩放本身不进组件协议），本组件不解决跨端显示。[职责](../../../raw/current/scripts/components/savedscale.lua#L1-L3)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（21 行），核对四个原版挂载点、corpses 生成缩放、EntityScript 组件存读协议。未审阅 corpse 各种类缩放取值表。
- 结构检查：2026-10-28 validate_page.py 通过（savedscale.md）；validate_source_links.py --strict 本页全部源码锚点通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；缩放跨存档恢复未实测。
