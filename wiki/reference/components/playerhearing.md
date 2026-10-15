---
title: "playerhearing"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家音频滤波组件：按装备标签或外部开关向世界 DSP 系统推 push/popdsp 滤波表，实现闷音效果。"
aliases: ["玩家听觉", "闷音滤波", "DSP muffle", "playerhearing"]
tags: [component, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/playerhearing.lua", "components/dsp.lua", "prefabs/player_classified.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# playerhearing

## 快速判断

- 职责与适用：给活跃玩家安装（[安装](../../../raw/current/scripts/prefabs/player_common.lua#L722-L730)），维护两组 DSP 滤波预设：戴 `mufflehat` 标签装备的闷音、被蠕虫吞噬的 `wormdigestionmuffle`（[预设](../../../raw/current/scripts/components/playerhearing.lua#L3-L36)）。
- 优先使用：要给玩家加全局闷音效果时，新增预设表项并用 `OverrideDSPEnabled` 开关。
- 重要副作用：滤波表通过玩家上的 `pushdsp`/`popdsp` 事件交给世界 `dsp` 组件实时生效（[消费](../../../raw/current/scripts/components/dsp.lua#L242-L243)；dsp 挂于 cave/forest 世界实体，[例](../../../raw/current/scripts/prefabs/forest.lua#L629)）。
- 修改边界：装备监听 equip/unequip 双端注册、`inventoryclosed` 仅客户端补扫（[初始化](../../../raw/current/scripts/components/playerhearing.lua#L58-L68)）；`duration` 0.5 秒用于 dsp 平滑。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OverrideDSPEnabled(dsp, enabled)` | `(string, boolean)` | 外部强制开/关某预设（如 wormdigestionmuffle），立即重算（[实现](../../../raw/current/scripts/components/playerhearing.lua#L83-L86)）。 |
| `GetDSPTables()` | → 表 | 返回当前生效的滤波表集合，dsp 组件在玩家监听建立时回放（[消费](../../../raw/current/scripts/components/dsp.lua#L244-L247)）。 |
| `UpdateDSPTables()` | 无参数 | diff 后对新增/移除的预设推 `pushdsp`/`popdsp`（[实现](../../../raw/current/scripts/components/playerhearing.lua#L92-L104)）。 |
| `overridedsp` | 表 `{预设名=bool}` | 外部开关状态源；`self[预设名]` 为实际生效态（[构造](../../../raw/current/scripts/components/playerhearing.lua#L70-L81)）。 |

预设表项含 `lowdsp`（各声道低通频率，如 creature 声道 2500）、可选 `equiptag`（true 表示按装备标签触发）与 `duration`（[结构](../../../raw/current/scripts/components/playerhearing.lua#L3-L36)）。

## 典型接入

### 新增自定义闷音预设

- 前提与选择：预设键名即组件字段名，需同时进 `DSP` 表与 `overridedsp` 初始化；改组件文件或 PostInit 补表。
- 生效与撤销：装备含对应标签（键名即标签）的物品自动启用；卸下或关背包后自动回退。
- 原版依据：`mufflehat` 预设按 `inventory:EquipHasTag("mufflehat")` 判定（[判定](../../../raw/current/scripts/components/playerhearing.lua#L42-L48)）；蠕虫吞噬经 player_classified 的 `wormdigestionsound` netvar 调 `OverrideDSPEnabled`（[消费](../../../raw/current/scripts/prefabs/player_classified.lua#L1091-L1099)）。

- 调整与验证：`DSP` 预设表是组件文件内 local，组件类未暴露注册新预设的方法；干净扩展需 PostInit 阶段直接改写实例状态或维护组件补丁，无原版示例代码可抄。lowdsp 数值越小闷得越狠；`duration` 交给 dsp 组件做插值过渡，默认 0.5 秒。

## 常见陷阱与限制

- `UpdateDSP` 的翻转逻辑依赖"状态与目标不同才翻"（[实现](../../../raw/current/scripts/components/playerhearing.lua#L42-L48)）：直接改 `self[键]` 不经 `UpdateDSPTables` 不会推事件，DSP 不生效。
- 客户端关背包时不会再收 equip/unequip 事件，靠 `inventoryclosed` 强制重扫（[注释](../../../raw/current/scripts/components/playerhearing.lua#L61-L66)）；漏注册该事件会造成服务器/客户端滤波不一致。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、dsp 组件消费点与 player_classified 蠕虫吞噬开关链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
