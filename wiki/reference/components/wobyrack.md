---
title: "wobyrack"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "给 Woby 挂载继承 dryingrack 的随身晾晒架，用独立容器实体承载食材，随 Walter 技能开关并在存档中按组件名恢复。"
aliases: ["wobyrack", "沃比晾晒架", "Woby 挂架", "随身晾肉架"]
tags: [component, item, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/wobyrack.lua", "components/dryingrack.lua", "prefabs/wobybig.lua", "prefabs/wobysmall.lua", "prefabs/woby_rack.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wobyrack

## 快速判断

- 职责与适用：dryingrack 的子类，专为 Woby（wobysmall／wobybig）把晾晒容器外置成 `woby_rack_container` 实体并挂到本体下；组件名是 `wobyrack`，Woby 上没有 `components.dryingrack`。[构造](../../../raw/current/scripts/components/wobyrack.lua#L1-L8)
- 优先使用：需要"移动单位上的晾晒架"时复用本组件加 dryingrack 全套方法（EnableDrying、GetContainer、SetShowItemFn 等），不要在 Woby 上直接挂 dryingrack。[基类入口](../../../raw/current/scripts/components/dryingrack.lua#L50-L62)
- 重要副作用：构造会 SpawnPrefab 一个容器实体并 SetParent 到 Woby；晾晒、雨天暂停、酸雨腐烂等行为全部由 dryingrack 基类基于该容器实例驱动。[基类构造](../../../raw/current/scripts/components/dryingrack.lua#L3-L10)
- 修改边界：开关由 Walter 技能 `walter_camp_wobyholder` 门控，EnableRack(false) 会整组件移除并回收 FX；存档恢复靠 prefab OnPreLoad 读 `data.wobyrack` 后重新启用。[门控](../../../raw/current/scripts/prefabs/wobybig.lua#L683-L684) [关闭](../../../raw/current/scripts/prefabs/wobybig.lua#L258-L263)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `Class(DryingRack, function(self, inst))` | SpawnPrefab(`woby_rack_container`)，把容器实体 SetParent 到 inst，再以容器调 `DryingRack._ctor`；容器 `isexposed` 被置 false。[实现](../../../raw/current/scripts/components/wobyrack.lua#L3-L8) |
| 基类晾晒方法 | 继承 dryingrack | `GetContainer`／`SetShowItemFn`／`SetHideItemFn`／`GetItemInSlot`／`EnableDrying`／`GetDryingInfoSnapshot` 等按 dryingrack 语义工作，容器参数来自构造传入。[基类](../../../raw/current/scripts/components/dryingrack.lua#L50-L96) |
| `DryingRack:OnSave/OnLoad` | 存档数据键为 `wobyrack` | EntityScript 按组件名收集 OnSave 数据；容器不在本体（`container.inst ~= self.inst`）时把容器持久数据写进 `data.contents`，载入时 SetPersistData 还原。[收集](../../../raw/current/scripts/entityscript.lua#L1928-L1935) [内容](../../../raw/current/scripts/components/dryingrack.lua#L422-L442) [分发](../../../raw/current/scripts/entityscript.lua#L1992-L2006) |
| `_dbg_print(...)` | 空实现 | print 已注释的调试桩，无行为。[实现](../../../raw/current/scripts/components/wobyrack.lua#L10-L12) |

## 典型接入

### 技能驱动的晾晒架开关与恢复（原版路径）

- 前提与选择：仅服务端；wobybig 在玩家技能事件里比较 `walter_camp_wobyholder` 决定开关，wobysmall 有对称实现。[wobybig 门控](../../../raw/current/scripts/prefabs/wobybig.lua#L681-L685)
- 生效与撤销：启用时 AddComponent 并立即 `EnableDrying()`、设置显示/隐藏回调、接管 `container.onanyopenfn/onanyclosefn`、生成两个 `woby_rack_swap_fx`；关闭时解除容器回调、RemoveComponent 并移除 FX 与覆盖贴图。[启用](../../../raw/current/scripts/prefabs/wobybig.lua#L227-L247) [关闭](../../../raw/current/scripts/prefabs/wobybig.lua#L258-L266)
- 原版依据：OnPreLoad 检查 `data.wobyrack`（即组件存档键存在）即重新启用，随后由组件 OnLoad 还原容器内容。[恢复](../../../raw/current/scripts/prefabs/wobybig.lua#L276-L280) [wobysmall](../../../raw/current/scripts/prefabs/wobysmall.lua#L204-L206)

## 常见陷阱与限制

- **用 `inst.components.dryingrack` 访问 Woby 的晾晒状态：** 组件注册名是 wobyrack，dryingrack 键不存在；基类内部注释明确要求用闭包而非 `inst.components.dryingrack`，因为"可能是 wobyrack"。通用晾晒知识见 [[reference/components/dryingrack]]。[基类注释](../../../raw/current/scripts/components/dryingrack.lua#L108-L112)
- **以为容器数据存在本体 container 组件里：** 内容物在独立 `woby_rack_container` 实体（woby_rack.lua 工厂提供），OnSave 特殊写入 `data.contents`；直接读 `inst.components.container` 拿不到晾晒架物品。[容器工厂](../../../raw/current/scripts/prefabs/woby_rack.lua#L268-L283) [contents 写入](../../../raw/current/scripts/components/dryingrack.lua#L438-L441)
- **Woby 消失时漏处理架上的物品：** 原版在回收/转移流程里先 GetContainer 逐槽 RemoveItemBySlot 并快照 dryinginfo，再移除实体；模组做类似转换需复用同一顺序。[收集](../../../raw/current/scripts/prefabs/wobybig.lua#L784-L793)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 wobyrack（14 行），核对 dryingrack 构造契约、wobybig/wobysmall 的启用-关闭-预载流程、容器工厂、EntityScript 按组件名收发存档。未审阅 dryingrack 全部晾晒数值与酸雨分支细节（见 dryingrack 页）。
- 结构检查：2026-10-28 validate_page.py 通过（wobyrack.md）；validate_source_links.py --strict 本页全部源码锚点通过；本页无完整 Lua 示例。
- 游戏运行：未运行；技能开关往返与跨存档恢复未实测。
