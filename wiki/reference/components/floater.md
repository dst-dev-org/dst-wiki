---
title: "floater"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "物品/物件水面漂浮表现：落水时切换漂浮动画、生成前后水面特效、推漂浮事件，服务端另附湿度和降温。"
aliases: ["漂浮", "floater", "物品漂浮", "水面特效"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/floater.lua", "standardcomponents.lua", "simutil.lua", "prefabskin.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# floater

## 快速判断

- 职责与适用：可落实体（多为 inventoryitem）接触海面时的表现层组件：服务端检测落水并加湿/降温/推事件，客户端生成 `float_fx_front`/`float_fx_back` 与浮动参数（[监听](../../../raw/current/scripts/components/floater.lua#L4-L28)）。
- 优先使用：物品类 prefab 用 `MakeInventoryFloatable(inst, size, offset, scale, swap_bank, float_index, swap_data)` 一站式挂载（[包装](../../../raw/current/scripts/standardcomponents.lua#L1772-L1788)），全库 447 个 prefab 文件在用。
- 重要副作用：落水非手持物品会被 `inventoryitem:MakeMoistureAtLeast(OCEAN_WETNESS)` 与降温上限（[加湿](../../../raw/current/scripts/components/floater.lua#L152-L157)），并推 `floater_startfloating`/`floater_stopfloating` 事件（[事件](../../../raw/current/scripts/components/floater.lua#L170)）。
- 修改边界：是否显示由 `IsOceanAtPoint` 且非 `IsVisualGroundAtPoint` 决定（[判定](../../../raw/current/scripts/components/floater.lua#L99-L105)）；`is_obstable` 经 `SetIsObstacle` 传入查询。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetSize(size)` | "small"/"med"/"large" | 选择特效动画档位（[实现](../../../raw/current/scripts/components/floater.lua#L51-L54)）。 |
| `SetVerticalOffset(offset)` / `SetScale(scale)` | offset: number；scale: number 或 {x,y,z} | 特效高度与三轴缩放，已生成特效即时应用（[实现](../../../raw/current/scripts/components/floater.lua#L56-L87)）。 |
| `SetBankSwapOnFloat(should_bank_swap, float_index, swap_data)` | float_index 负值=左 | 漂浮时切 `floating_item` bank 定格并按 swap_data 覆盖 `swap_spear` 符号（[实现](../../../raw/current/scripts/components/floater.lua#L89-L143)）。 |
| `SetSwapData(swap_data)` | `{bank, anim, sym_name/sym_build}` | 仅换覆盖数据不启用 bank 切换（[实现](../../../raw/current/scripts/components/floater.lua#L95-L97)）。 |
| `IsFloating()` | → `bool` | 当前是否处于漂浮表现（[实现](../../../raw/current/scripts/components/floater.lua#L119-L121)）。 |
| `Erode(erode_time)` | 秒 | 服务端置 `_erode_time` netvar，客户端对前后特效 `ErodeAway`（[实现](../../../raw/current/scripts/components/floater.lua#L208-L212)、[消费](../../../raw/current/scripts/simutil.lua#L582-L583)）。 |

## 典型接入

### 让手持物品漂浮在水面

- 前提与选择：在 prefab 的 master postinit（或 common 内）调用 `MakeInventoryFloatable`；组件双端行为由自身 `ismastersim` 分支处理。
- 生效与撤销：`on_landed`/`on_no_longer_landed` 物理事件驱动；`onremove` 时强制走离水清理（[驱动](../../../raw/current/scripts/components/floater.lua#L6-L9)、[离水](../../../raw/current/scripts/components/floater.lua#L214-L222)）。
- 原版依据：皮肤应用后需重放漂浮状态与动画（[prefabskin](../../../raw/current/scripts/prefabskin.lua#L123-L139)）；`componentactions` 以 `IsFloating` 判定"地上机器"动作（[消费](../../../raw/current/scripts/componentactions.lua#L533)）；高温物品落水换 `hot_splash`（[分支](../../../raw/current/scripts/components/floater.lua#L159-L162)）。

```lua
-- prefab 内（master postinit）
MakeInventoryFloatable(inst, "small", 0.2, {1.0, 0.6, 1.0})
-- 需要漂浮时换动画才用：
inst.components.floater:SetBankSwapOnFloat(true, -11, {sym_name = "axe01", sym_build = "swap_axe"})
```

- 调整与验证：offset 是特效相对宿主的高度（默认 0）；`bob_percent` 控制浮动幅度参数（默认 1，[字段](../../../raw/current/scripts/components/floater.lua#L40)、[应用](../../../raw/current/scripts/components/floater.lua#L192)）。

## 常见陷阱与限制

- `MakeInventoryFloatable` 的 `swap_bank` 参数名虽含 bank，实际传的是"是否启用 bank 切换"布尔（[包装](../../../raw/current/scripts/standardcomponents.lua#L1784-L1787)），与组件接口名不同。
- 漂浮表现依赖物理落地事件：把实体直接 `SetPosition` 到海面而不触发 `on_landed` 不会出现特效；客户端特效由 `landeddirty` netvar 驱动，dedicated 服务器不生成（[netvar](../../../raw/current/scripts/components/floater.lua#L43-L44)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、standardcomponents 包装、simutil 腐蚀链、prefabskin/componentactions 消费点；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
