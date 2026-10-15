---
title: "pumpkinhatcarvable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "南瓜帽雕刻组件：让可装备帽类物品按五官部位（reye/leye/mouth）记录雕刻变体，走弹窗 RPC 校验与雕刻台状态。"
aliases: ["南瓜帽雕刻", "pumpkinhatcarvable", "雕刻南瓜帽", "脸数据"]
tags: [component, server-client, callback, state-machine]
dst_build: "steam-buildid:26230496"
sources: ["components/pumpkinhatcarvable.lua", "prefabs/hats.lua", "componentactions.lua", "actions.lua", "popupmanager.lua", "screens/redux/pumpkinhatcarvingscreen.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# pumpkinhatcarvable

## 快速判断

- 职责与适用：可装备帽类（原版 pumpkin_hat 等）的雕刻协议：服务端管理唯一雕刻者（carver）、弹窗生命周期与脸数据落库；客户端有同名组件供建构行动与界面读取。原版安装于 hats.lua 的南瓜帽。[安装](../../../raw/current/scripts/prefabs/hats.lua#L6111-L6112)
- 优先使用：读当前脸数据用 `GetFaceData()`（走 `collectfacedatafn`）；雕刻入口走 `CARVEPUMPKIN` 行动，不要直接调 BeginCarving 绕过弹窗 RPC。[行动构造](../../../raw/current/scripts/componentactions.lua#L1585-L1596) [行动执行](../../../raw/current/scripts/actions.lua#L6062)
- 重要副作用：雕刻期间 `inventoryitem.canbepickedup = false`、玩家进 `pumpkincarving` 状态并开 `POPUPS.PUMPKINHATCARVING` 弹窗；拾起／漂浮会中断，离开 3 距离或失去视野自动收尾。[开始](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L134-L157) [自动收尾](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L186-L192)
- 修改边界：脸数据规则（部位×工具档×全脸或全空）硬编码在本组件并被弹窗 RPC 校验复用；宿主要自己提供外观切换（onchangefacedatafn）与当前数据回读（collectfacedatafn）。[校验复用](../../../raw/current/scripts/popupmanager.lua#L232-L246)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `PARTS`／`PART_IDS`／`VARS_PER_TOOL` | 类常量 | `{"reye","leye","mouth"}`、反查表、每工具档 9 个变体；弹窗 RPC 校验与雕刻界面直接读这三个类字段。[定义](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L1-L8) [类字段](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L98-L100) |
| `GetFaceData()` | `-> {reye=?, leye=?, mouth=?}` | 经 `collectfacedatafn` 收集当前脸数据；雕刻界面打开时也走它读初始值。[实现](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L111-L117) [界面](../../../raw/current/scripts/screens/redux/pumpkinhatcarvingscreen.lua#L199-L200) |
| `CanBeginCarving(doer)` | `-> ok, 原因?` | false 情形：雕刻者 busy／已是自己、燃烧中（`BURNING`）、被占用（`INUSE`）、已装备、漂浮中。[实现](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L119-L132) |
| `BeginCarving(doer)`／`EndCarving(doer)` | `仅主模拟` | 开：锁拾取、监听 doer 的 onremove／ms_closepopup、进状态、StartUpdating、onopenfn。收：解锁、推 `ms_endpumpkincarving`、onclosefn。[开](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L134-L157) [收](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L159-L180) |
| `onclosepopup` | 内部回调 | 监听 doer `ms_closepopup`，把弹窗参数按 PARTS 组装成 facedata 交给 `onclosepumpkin`。[实现](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L69-L80) |
| `onchangefacedatafn` | `function(inst, facedata)` | 校验通过后的落库钩子；原版接 hats 的 `pumpkin_onload` 持久化与换装。[原版接线](../../../raw/current/scripts/prefabs/hats.lua#L6152) |
| `collectfacedatafn` | `function(inst, data)` | 当前脸数据回读；原版接 `pumpkin_onsave`。[原版接线](../../../raw/current/scripts/prefabs/hats.lua#L6112) |

## 典型接入

### 校验规则（模组自定宿主必须遵守）

- 工具档位：`variation 1..27` 映射 `math.ceil(variation/9)` 号 `pumpkincarver1..3`，雕刻者背包（含容器）没有对应刀则该部位改动被丢弃；0 表示无。[校验](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L10-L46)
- 全脸或全空：三个部位要么全 0 要么全非 0，混合直接拒绝——RPC 校验同规则（`popupmanager` 的 validaterpcfn）。[规则](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L32-L43)

```lua
-- 服务端，自定义可雕刻帽（需自带换装/持久化逻辑）
inst:AddComponent("pumpkinhatcarvable")
inst.components.pumpkinhatcarvable.collectfacedatafn = function(inst, data)
    data.reye, data.leye, data.mouth = inst._face and inst._face.reye or 0,
        inst._face and inst._face.leye or 0, inst._face and inst._face.mouth or 0
end
inst.components.pumpkinhatcarvable.onchangefacedatafn = function(inst, facedata)
    inst._face = facedata
end
```

- 调整与验证：原版宿主不要覆盖 `onchangefacedatafn`（会断开 hats 的存档链）；提交路径是弹窗 RPC → `ms_closepopup` → 校验 → 落库 → `EndCarving`。[弹窗注册](../../../raw/current/scripts/prefabs/hats.lua#L6111-L6112)

## 常见陷阱与限制

- **客户端构造提前 return：** 非主模拟分支直接 `return inst`，服务端字段（carver/range/回调）在客户端全是 nil；客户端代码只能用 `GetFaceData` 与类常量。[分支](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L59-L61)
- **雕刻被打断的隐式路径：** 实体被拾起或开始漂浮立即 EndCarving（中断时不问 doer）；燃烧态只挡"开始"，不中断进行中的雕刻。[中断](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L48-L51)
- **自动收尾靠组件更新：** EndCarving 后 StartUpdating 仍会跑一轮；`carver` 为 nil 的更新会自我停止，不会泄漏任务。[更新体](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L186-L192)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（194 行），核验 hats.lua 安装与钩子接线、componentactions/actions/popupmanager/雕刻界面消费点。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。

相关：[[reference/components/equippable]]
