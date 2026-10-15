---
title: "highlight"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "给实体叠加高亮加色并驱动闪光动画：悬停选中、攻击前闪烁等对目标的一次性着色，随用随加、用完即移除。"
aliases: ["悬停高亮", "highlight", "目标高亮", "闪光高亮"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/highlight.lua", "components/playercontroller.lua", "prefabs/oceanwhirlportal.lua", "components/highlightchild.lua", "prefabskin.lua", "prefabs/wortox_soul_spawn.lua", "prefabs/hermitcrab_teashop.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# highlight

## 快速判断

- 职责与适用：把加法色经 `AnimState:SetHighlightColour` 应用到实体及其 `highlightchildren` 子实体，支持常亮高亮与缓入缓出的闪光；原版没有任何 prefab 静态注册，由 `playercontroller` 在悬停与攻击前动态 `AddComponent`（[悬停](../../../raw/current/scripts/components/playercontroller.lua#L2860)、[攻击](../../../raw/current/scripts/components/playercontroller.lua#L4612)）。
- 优先使用：需要"当前指向目标"式高亮时直接仿照 playercontroller 按需加组件、调 `Highlight`/`Flash`，`UnHighlight` 后组件自动移除（[移除](../../../raw/current/scripts/components/highlight.lua#L126)）。
- 重要副作用：纯 Lua 着色，无网络变量，每台机器各自维护；`UnHighlight`/闪光结束会 `RemoveComponent("highlight")`。
- 修改边界：着色只作用于 AnimState；不可高亮时（非玩家且 `CanEntitySeeTarget(ThePlayer, inst)` 失败）只清色不生效（[门槛](../../../raw/current/scripts/components/highlight.lua#L98)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Highlight(r, g, b)` | 参数可省略 | 常亮加色；默认 `.2,.2,.2`，实体带 `highlightoverride` 表时用其前三元（[覆盖](../../../raw/current/scripts/components/highlight.lua#L100-L104)）。 |
| `UnHighlight()` | 无参数 | 清常亮并在非闪光态移除组件（[实现](../../../raw/current/scripts/components/highlight.lua#L119-L128)）。 |
| `Flash(toadd, timein, timeout)` | 数值参数 | 启动缓入缓出闪光更新；结束时若未常亮则自移除（[更新](../../../raw/current/scripts/components/highlight.lua#L37-L79)）。 |
| `SetAddColour(col)` | `col` 取 `.x/.y/.z` | 设置基色叠加；与高亮色、闪光值求和后统一应用（[求和](../../../raw/current/scripts/components/highlight.lua#L82-L84)）。 |
| `inst.highlightoverride` | `{r, g, b}` 实体字段 | 修改默认常亮色；原版由个别 prefab 设置，如 `oceanwhirlportal.lua:222`。 |
| `inst.highlightflashaddoverride` | 数值实体字段 | 覆盖攻击闪光强度，playercontroller 读取（[读取](../../../raw/current/scripts/components/playercontroller.lua#L4615-L4616)）。 |
| `inst.highlightchildren` | 实体数组 | 同步着色的子实体；由皮肤系统与 `highlightchild` 组件维护（[设置](../../../raw/current/scripts/prefabskin.lua#L3078)）。 |

## 典型接入

### 临时闪光或常亮标记一个目标

- 前提与选择：任意端可用（客户端效果只影响本机显示）；组件是纯 Lua，按需 `AddComponent`，无需预制。
- 生效与撤销：`Flash` 自动走完并自拆；`Highlight` 需配对 `UnHighlight`，否则组件常驻。
- 原版依据：悬停高亮即"动态加组件 + Highlight"，攻击前闪光同理（[悬停链](../../../raw/current/scripts/components/playercontroller.lua#L2852-L2870)、[闪光链](../../../raw/current/scripts/components/playercontroller.lua#L4609-L4618)）。

```lua
-- 给目标做一次 0.3 强度闪光（本机生效；target 为任意合法实体）
if target.components.highlight == nil then
    target:AddComponent("highlight")
end
target.components.highlight:Flash(0.3, 0.125, 0.1)
```

- 调整与验证：闪光强度默认 `.2`、进/出 `0.125/0.1` 秒（[参数](../../../raw/current/scripts/components/playercontroller.lua#L4614)），可在实体上写 `highlightflashaddoverride` 调整。

## 常见陷阱与限制

- 在 wortox 灵魂 FX 上，`components.highlight` 存在与否决定着色路径分支（[分支](../../../raw/current/scripts/prefabs/wortox_soul_spawn.lua#L264)）：给这类实体常驻挂 highlight 会改变其特效逻辑。
- 子实体列表不是组件自管的：静态子实体要自己写 `inst.highlightchildren`（如 [茶铺](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L377)），FX 子实体建议用 [[reference/components/highlightchild]] 登记。
- 组件自移除发生在 `UnHighlight` 或闪光收尾，直接 `RemoveComponent` 也安全——`OnRemoveFromEntity` 会清空 AnimState 高亮色（[清理](../../../raw/current/scripts/components/highlight.lua#L130-L141)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、playercontroller 悬停/攻击两条消费链、wortox FX 与 `highlightoverride`/`highlightchildren` 设置者；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
