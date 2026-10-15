---
title: "uniqueprefabids"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界实体上的每 prefab 自增计数器：给 uniqueid 等消费者分配跨存档稳定的序号。"
aliases: ["prefab计数器", "唯一序号分配", "uniqueprefabids"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/uniqueprefabids.lua", "components/uniqueid.lua", "prefabs/world.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# uniqueprefabids

## 快速判断

- 职责与适用：挂在 TheWorld 上的服务端计数器：按 prefab 名维护"已发到的最大序号"，`GetNextID(prefabname)` 自增并返回。原版唯一用途是给 `uniqueid` 组件分配实体序号（世界实体 Metrics 段添加，[world.lua#L609](../../../raw/current/scripts/prefabs/world.lua#L609)）。
- 优先使用：`TheWorld.components.uniqueprefabids:GetNextID(inst.prefab)`；一般通过给实体加 `uniqueid` 组件间接使用，见 [[reference/components/uniqueid]]。
- 重要副作用：`topprefabids` 表随 OnSave／OnLoad 持久化（[存](../../../raw/current/scripts/components/uniqueprefabids.lua#L18-L22)、[读](../../../raw/current/scripts/components/uniqueprefabids.lua#L24-L26)），读档后新序号从上次最大值继续，不与旧实体冲突。
- 修改边界：只在服务端世界实体上存在；计数器全局唯一、按 prefab 名分桶；不校验 prefab 名，任意字符串键都会新建桶。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetNextID(prefabname)` | number ≥ 1 | 首次返回 1，此后每次 +1；单线程自增，无并发保护。[L9-L16](../../../raw/current/scripts/components/uniqueprefabids.lua#L9-L16) |
| `topprefabids` | 表：prefab 名 → 最大序号 | 构造为空表；OnSave 原样保存，OnLoad 用 `data.topprefabids or {}` 恢复。[L3-L7](../../../raw/current/scripts/components/uniqueprefabids.lua#L3-L7) |
| `GetDebugString()` | string | 拼接所有桶的 "名: 序号"。[L28-L34](../../../raw/current/scripts/components/uniqueprefabids.lua#L28-L34) |

## 常见陷阱与限制

- 序号稳定性依赖"先读档、后取号"：`uniqueid` 组件存档自带 id，读档直接沿用；只有新实体才在 0 秒任务里取号（[uniqueid.lua#L4-L11](../../../raw/current/scripts/components/uniqueid.lua#L4-L11)）。跳过 uniqueid 直接取号的实现要自己处理读档实体的旧号。
- 计数器不回收：销毁实体不会让序号回退，长存档里各桶计数只增不减（当前仅用于唯一性，无上限问题）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/uniqueprefabids.lua（37 行）与 components/uniqueid.lua（37 行），核对 world.lua 添加位置（master_postinit 之后的 Metrics 段）与唯一消费者。未覆盖引擎存档装配。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；43 行、3207 字节，simple 档内）；validate_source_links.py --strict 通过（7 处源码引用，0 error／0 warning）；本页无完整 Lua 代码块。
