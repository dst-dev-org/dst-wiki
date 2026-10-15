---
title: "scaler"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "等比缩放的统一入口：SetScale 立即应用到 Transform 并触发 OnApplyScale 钩子，数值随存档保存恢复；原版宿主是岩石龙虾。"
aliases: ["scaler", "缩放组件", "实体缩放"]
tags: [component, data]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/scaler.lua", "prefabs/rocky.lua", "prefabs/rockyherd.lua", "stategraphs/SGrocky.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# scaler

## 快速判断

- 职责与适用：为需要"缩放值 + 立即生效 + 存读恢复"的实体提供统一入口；`scale` 默认 1，`SetScale` 存值并调 `ApplyScale` 写 Transform。[组件全文](../../../raw/current/scripts/components/scaler.lua#L2-L31)
- 优先使用：需要缩放联动其他系统（碰撞体、动画）时设 `OnApplyScale` 钩子，在回调里同步；rocky 用它更新 Physics 胶囊半径。[钩子挂接](../../../raw/current/scripts/prefabs/rocky.lua#L535-L536)
- 重要副作用：OnSave 无条件输出 `scale`，OnLoad 有值才 `SetScale` 恢复；恢复会再次触发 ApplyScale 钩子。[存读](../../../raw/current/scripts/components/scaler.lua#L15-L26)
- 修改边界：只做等比缩放（三轴同值）；缩放不进网络变量，客户端表现依赖 Transform 同步；直接 `Transform:SetScale` 不会更新组件数值，两者不要混用。[实现](../../../raw/current/scripts/components/scaler.lua#L7-L13)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetScale(scale)` | `scale: number -> 无` | 存值后立即 `ApplyScale`；宿主无须再手动 SetScale。[实现](../../../raw/current/scripts/components/scaler.lua#L28-L31) |
| `ApplyScale()` | `-> 无` | `Transform:SetScale(scale, scale, scale)`；`OnApplyScale` 字段存在时以 `(inst, scale)` 调用。字段由宿主赋值，组件不初始化。[实现](../../../raw/current/scripts/components/scaler.lua#L7-L13) |
| `scale` | `number`（默认 1） | 读取方直接访问字段：rocky 的生长／酸雨萎缩按当前值递增递减，SG 动画用它重映射幅度。[生长](../../../raw/current/scripts/prefabs/rocky.lua#L57-L62) [动画](../../../raw/current/scripts/stategraphs/SGrocky.lua#L19) |
| `OnSave`／`OnLoad` | 存读 `scale` | 载入有值即 SetScale 恢复；无存档数据不动作。[实现](../../../raw/current/scripts/components/scaler.lua#L15-L26) |

## 典型接入

### 可生长实体的缩放（rocky 参照）

- 前提与选择：生成后挂组件、设 OnApplyScale 同步碰撞体；生长逻辑按 dt 把 `scale` 逼近上限后调 SetScale。[挂接](../../../raw/current/scripts/prefabs/rocky.lua#L535-L536) [生长](../../../raw/current/scripts/prefabs/rocky.lua#L57-L62)
- 生效与撤销：SetScale 即刻生效并随存档持久；rockyherd 生成新成员后立即 `SetScale(TUNING.ROCKY_MIN_SCALE)` 收敛初始体型，说明组件适合"生成时定值 + 后续生长"两类场景。[群体初值](../../../raw/current/scripts/prefabs/rockyherd.lua#L35)

```lua
-- 服务端：可生长实体的缩放（含存档恢复）
inst:AddComponent("scaler")
inst.components.scaler.OnApplyScale = function(inst, scale)
    inst.Physics:SetCapsule(BASE_RADIUS * scale, 1)  -- 参照 rocky 同步碰撞体
end
inst.components.scaler:SetScale(1.5)  -- 立即应用，存读自动
```

- 调整与验证：上限／下限判断留在宿主侧（rocky 的 `IsMaxSize` 比较 scale 与 TUNING），组件不做范围校验；酸雨萎缩是反向用例。[上限](../../../raw/current/scripts/prefabs/rocky.lua#L403-L404)

## 常见陷阱与限制

- **按文件头注释理解职责：** 组件第 1 行注释写着 "Tracks the herd that the object belongs to"，与实际实现无关（疑似复制残留）；组件只管缩放，群体逻辑在 herd。[失真注释](../../../raw/current/scripts/components/scaler.lua#L1)
- **绕过组件直接改 Transform：** 直接 `Transform:SetScale` 后组件 `scale` 字段不同步，后续 SetScale／存读会按旧值覆盖外观；改缩放一律走 `SetScale`。[实现](../../../raw/current/scripts/components/scaler.lua#L28-L31)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（33 行），核对 rocky 挂接／生长／酸雨／碰撞胶囊、rockyherd 初值、SGrocky 动画读取。grep 确认原版唯一 AddComponent 宿主为 rocky.lua。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
