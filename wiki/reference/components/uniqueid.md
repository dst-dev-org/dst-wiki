---
title: "uniqueid"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "给实体发 prefab 内唯一编号：从世界级 uniqueprefabids 计数器取号并随存档持久化；原版唯一宿主是皮弗陶牛。"
aliases: ["uniqueid", "实体唯一编号", "牛编号"]
tags: [component, data]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/uniqueid.lua", "components/uniqueprefabids.lua", "prefabs/beefalo.lua", "components/beefalometrics.lua", "prefabs/world.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# uniqueid

## 快速判断

- 职责与适用：挂在需要稳定个体编号的实体上；构造下一帧从 `TheWorld.components.uniqueprefabids:GetNextID(inst.prefab)` 取号，同一 prefab 递增计数。[取号](../../../raw/current/scripts/components/uniqueid.lua#L2-L9)
- 优先使用：需要"这只牛是第几只"类持久身份时挂本组件并读 `id` 字段；beefalometrics 把它写进统计值。[宿主](../../../raw/current/scripts/prefabs/beefalo.lua#L1263) [统计消费](../../../raw/current/scripts/components/beefalometrics.lua#L6)
- 重要副作用：`OnLoad` 恢复 id 后取消待执行的取号任务，保证读档不重复发号；OnSave 无条件输出 id（可为 nil）。[存读](../../../raw/current/scripts/components/uniqueid.lua#L19-L30)
- 修改边界：编号是世界级计数器（uniqueprefabids 挂在 TheWorld 并存档），不是 GUID；组件不校验 id 唯一性——计数器缺失时 id 保持 nil。[计数器](../../../raw/current/scripts/components/uniqueprefabids.lua#L9-L15)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `id` | `number|nil` | 延迟取号（`DoTaskInTime(0, TryGetNewId)`）；读档有值则跳过。读取方需判 nil。[构造](../../../raw/current/scripts/components/uniqueid.lua#L11-L17) |
| `OnSave`／`OnLoad` | 存读 `id` | OnLoad 即使 nil 也赋值（显式清号语义），随后有值才取消任务。[实现](../../../raw/current/scripts/components/uniqueid.lua#L19-L30) |
| `GetDebugString()` | `-> string` | 调试串输出 id；无其他运行时行为。[实现](../../../raw/current/scripts/components/uniqueid.lua#L32-L34) |

## 典型接入

### 给自定义生物发持久编号（beefalo 参照）

- 前提与选择：世界需有 uniqueprefabids（原版 world prefab 已挂载，自定义世界要补挂）；实体侧只需挂组件，取号自动完成。[世界计数器](../../../raw/current/scripts/prefabs/world.lua#L609)
- 生效与撤销：编号随实体存档、计数器随世界存档，两边一致才能不重号；删除实体不会回收编号（计数只增不减）。[计数器实现](../../../raw/current/scripts/components/uniqueprefabids.lua#L9-L15)

```lua
-- 服务端：发号与读取
inst:AddComponent("uniqueid")
-- 下一帧起可读（TryGetNewId 在 DoTaskInTime(0) 里取号）：
local id = inst.components.uniqueid.id
```

- 调整与验证：按 prefab 名分桶计数（GetNextID 以 inst.prefab 为键），不同 prefab 互不影响；读档路径已在组件内处理，宿主无须重放。[取号](../../../raw/current/scripts/components/uniqueid.lua#L2-L9)

## 常见陷阱与限制

- **同帧读 id：** 取号在构造后一帧执行，同帧读到 nil；急用可自行延后或改组件调用 TryGetNewId 同款逻辑，但不要提前手动赋值覆盖存档恢复。[时序](../../../raw/current/scripts/components/uniqueid.lua#L16)
- **把 id 当全局唯一：** 计数器按 prefab 分桶，"1 号牛"与"1 号自定义生物"编号相同；跨 prefab 需组合 prefab 名使用。[分桶](../../../raw/current/scripts/components/uniqueprefabids.lua#L9-L15)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（36 行）与 uniqueprefabids 计数器（37 行），核对 beefalo 挂载、beefalometrics 读取、world 挂载行；grep 全库确认唯一 AddComponent 宿主为 beefalo.lua。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
