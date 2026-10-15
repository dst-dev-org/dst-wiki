---
title: "vaulttorchgrid"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "地窖 lightsout 试炼的火炬网格：按列/行/间距批量布置 vault_torch，提供坐标取炬与邻接遍历，并把每只火炬的开关/卡死/破损状态位打包成十六进制串存档。"
aliases: ["vaulttorchgrid", "火炬网格", "lightsout 火炬阵", "vault torch grid"]
tags: [component, callback, data]
dst_build: "steam-buildid:26230496"
sources: ["components/vaulttorchgrid.lua", "prefabs/lightsout_trial.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# vaulttorchgrid

## 快速判断

- 职责与适用：挂在 lightsout_trial 房间实体上的服务端组件（[lightsout_trial.lua:509](../../../raw/current/scripts/prefabs/lightsout_trial.lua#L509）；`Initialize(cols, rows, spacing)` 以宿主为中心按 `col*spacing` 网格生成 `vault_torch`（`persists = false`），解谜逻辑经 `GetTorch`／`ForEach`／`ForEachAdjacent` 驱动。[布置](../../../raw/current/scripts/components/vaulttorchgrid.lua#L45-L67)
- 优先使用：布置完自己管交互 → 组件只管网格与存档；点亮/熄灭走火炬自身 machine 组件，组件不直接操作状态（存读档除外）。([lightsout.lua:495-500](../../../raw/current/scripts/prefabs/lightsout_trial.lua#L495-L500)
- 重要副作用：`OnRemoveEntity` 静默移除全部火炬（不走 ontorchremovedfn）；`Clear` 移除时回调 `ontorchremovedfn` 并清零行列计数。[清理](../../../raw/current/scripts/components/vaulttorchgrid.lua#L23-L43)
- 修改边界：火炬不持久化，网格状态靠组件 OnSave 的位串重建；行列为 0 时 OnSave 返回 nil。[存档](../../../raw/current/scripts/components/vaulttorchgrid.lua#L196-L211)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `(inst)` | `spacing = 4`（默认）、`numcols/numrows = 0`、空 `grid` 二维表；`ontorchaddedfn`／`ontorchremovedfn` 经 setter 注入。([vaulttorchgrid.lua:1-21](../../../raw/current/scripts/components/vaulttorchgrid.lua#L1-L21) |
| `Initialize(numcols, numrows, spacing)` | 数量×2, 间距(可选) | 先 `Clear`，`coloffs/rowoffs` 使网格以宿主居中；每格生成 `vault_torch`、写 `torch.col/row`（居中偏移坐标）、定位并调 `ontorchaddedfn`。([vaulttorchgrid.lua:45-67](../../../raw/current/scripts/components/vaulttorchgrid.lua#L45-L67) |
| `GetTorch(col, row)` | 偏移坐标 | 把居中偏移坐标换回 1 基内部索引取火炬，越界返回 nil；lightsout 用它定位卡死/损坏炬。([vaulttorchgrid.lua:69-72](../../../raw/current/scripts/components/vaulttorchgrid.lua#L69-L72) ([lightsout.lua:73-77](../../../raw/current/scripts/prefabs/lightsout_trial.lua#L73-L77) |
| `ForEach(cb)` | `(inst, torch) -> boolean` | 按列序遍历，回调返回 true 短路；lightsout 用它做全亮检查与救助提示。([vaulttorchgrid.lua:74-84](../../../raw/current/scripts/components/vaulttorchgrid.lua#L74-L84) ([lightsout.lua:495-500](../../../raw/current/scripts/prefabs/lightsout_trial.lua#L495-L500) |
| `ForEachAdjacent(torch, cb)` | 火炬实体, 回调 | 仅当火炬的 col/row 与网格一致时触发上下左右四邻回调（`_try_adjacent` 越界静默）；lightsout 的"翻转邻炬"依赖它。([vaulttorchgrid.lua:86-110](../../../raw/current/scripts/components/vaulttorchgrid.lua#L86-L110) |
| 状态位序列化 | 私有类 Serializer／Deserializer | 每炬 3 位：machine `IsOn`、`IsStuck`、`IsBroken`，按 32 位一组成十六进制串；读取侧按位还原（`MakeStuckOn/Off`、`machine:TurnOn/Off`、`MakeBroken/Normal`）。[序列化](../../../raw/current/scripts/components/vaulttorchgrid.lua#L114-L140) [还原](../../../raw/current/scripts/components/vaulttorchgrid.lua#L144-L192) |
| `OnSave`／`OnLoad` | 存档钩子 | 存 `cols/rows/bits`（非默认间距才存 `spacing`）；尺寸或间距变化时重新 `Initialize` 再逐炬还原状态。([vaulttorchgrid.lua:196-232](../../../raw/current/scripts/components/vaulttorchgrid.lua#L196-L232) |

## 典型接入

### 复用"批量布置＋邻接联动"的机关网格

- 前提与选择：宿主有 Transform（以自身为网格中心）；火炬需支持 `col`/`row` 字段与 `MakeStuckOn` 等状态方法，状态位序列化假定 machine 组件存在。
- 生效与撤销：`Initialize` 全量重建（先 Clear）；运行时增删单元格不被支持，改尺寸需重新 Initialize（OnLoad 也走这条路）。
- 原版依据：lightsout 试炼的布阵（3×3、间距 6）与翻转联动见 ([lightsout_trial.lua:32-35](../../../raw/current/scripts/prefabs/lightsout_trial.lua#L32-L35)、[495-500](../../../raw/current/scripts/prefabs/lightsout_trial.lua#L495-L500)。

```lua
-- 宿主 master_postinit：
inst:AddComponent("vaulttorchgrid")
inst.components.vaulttorchgrid:SetOnTorchAddedFn(function(owner, torch)
    -- 每炬生成后回调（接线/初次点亮等）
end)
-- 布置 3×3、间距 6 的火炬阵：
inst.components.vaulttorchgrid:Initialize(3, 3, 6)
-- 遍历邻炬做联动（如 lightsout 翻转规则）：
-- inst.components.vaulttorchgrid:ForEachAdjacent(torch, function(owner, adj) ... end)
```

- 调整与验证：`spacing` 单位是单位长度（原版房间用 6）；组件本身不判解谜，胜利条件由宿主经 `ForEach` 自查。

## 常见陷阱与限制

- `ForEachAdjacent` 防伪校验：若传入火炬的 `gridcol[row] ~= torch`（比如已被 Clear 替换）整个遍历静默跳过（([vaulttorchgrid.lua:96-109](../../../raw/current/scripts/components/vaulttorchgrid.lua#L96-L109)），回调不执行的排查应先核对火炬登记一致性。
- 位串按遍历序还原：OnLoad 前若网格尺寸/间距与存档不一致会先重建，再逐炬 `PopTorch`；自定义火炬缺 `IsStuck`／`MakeBroken` 等方法会在载入时报错（([vaulttorchgrid.lua:166-192](../../../raw/current/scripts/components/vaulttorchgrid.lua#L166-L192)）。
- `OnRemoveEntity` 与 `Clear` 行为不同：前者不发 `ontorchremovedfn`（([vaulttorchgrid.lua:23-29](../../../raw/current/scripts/components/vaulttorchgrid.lua#L23-L29)），依赖该回调做清理的逻辑在实体整体移除时会漏跑。
- 火炬 `persists = false`：不要单独给火炬写存档；组件只在 `numcols/numrows > 0` 时存档（([vaulttorchgrid.lua:196-197](../../../raw/current/scripts/components/vaulttorchgrid.lua#L196-L197)），空网格载入无需处理。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全文；核验 lightsout_trial.lua 挂载与全部消费点（布阵、取炬、邻接翻转、状态救助与全亮检查）。
- 游戏运行：未运行。
- 结构检查：未执行。
