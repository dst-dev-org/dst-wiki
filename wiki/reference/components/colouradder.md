---
title: "colouradder"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "按来源堆叠加法色（AddColour）：多来源 RGBA 求和后应用到 AnimState，支持子实体联动，是命中闪白、月色染色等特效的着色基座。"
aliases: ["附加色堆叠", "colouradder", "染色组件"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/colouradder.lua", "components/colouraddersync.lua", "prefabs/electrocute_fx.lua", "prefabs/player_common.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# colouradder

## 快速判断

- 职责与适用：给实体维护"来源 → RGBA"加法色堆栈，求和（逐通道 clamp 0..1）后应用到 AnimState；支持挂子实体联动传播。主机端组件（原版在 master postinit 挂，如[玩家](../../../raw/current/scripts/prefabs/player_common.lua#L2669)）。
- 优先使用：特效需要给目标临时叠加颜色时 `PushColour(source, r, g, b, a)` / `PopColour(source)`；原版电击特效对目标动态加组件后驱动（[electrocute_fx](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L160)）。
- 重要副作用：来源是实体时监听其 `onremove` 自动弹出；推入全零颜色等价 `PopColour`（[L86-L89](../../../raw/current/scripts/components/colouradder.lua#L86-L89)）。
- 修改边界：无存读钩子，颜色不随存档恢复；跨端显示依赖 colouraddersync（见 [[reference/components/colouraddersync]]），无同步组件时 `SetAddColour` 只在主机可见。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `PushColour(source, r, g, b, a)` | `(entity 或任意键, 0..1 x4) -> void` | 按来源记入堆栈（同源重复推入更新值），重新求和且结果变化才应用。[实现](../../../raw/current/scripts/components/colouradder.lua#L84-L107) |
| `PopColour(source)` | `source -> void` | 弹出来源并重算；无此来源时无操作。[实现](../../../raw/current/scripts/components/colouradder.lua#L109-L120) |
| `OnSetColour(r, g, b, a)` | `0..1 x4 -> void` | 最终应用入口：优先 `colouraddersync:SyncColour`，否则直接 `AnimState:SetAddColour`；并传播到全部子实体。[实现](../../../raw/current/scripts/components/colouradder.lua#L66-L82) |
| `AttachChild/DetachChild(child)` | `entity -> void` | 子实体联动：child 按其组件优先走 colouradder / colouraddersync / 裸 AnimState 三档。[实现](../../../raw/current/scripts/components/colouradder.lua#L24-L49) |
| `GetCurrentColour/CalculateCurrentColour` | `-> 0..1 x4` | 前者返回当前应用值；后者对堆栈求和并逐通道 clamp。[实现](../../../raw/current/scripts/components/colouradder.lua#L55-L64) |
| `OnRemoveFromEntity` | 组件移除钩子 | 清理来源与子实体的 `onremove` 监听，并让子实体 `PopColour(self.inst)`。[实现](../../../raw/current/scripts/components/colouradder.lua#L10-L22) |

## 典型接入

### 给目标加可叠加的临时闪光

- 前提与选择：主机端为目标加组件后 `PushColour`；`source` 可用字符串键，多特效互不干扰；撤销时 `PopColour` 同键即可，堆栈空后回到无色。

```lua
-- 主机端：目标可能是玩家或任意实体
if target.components.colouradder == nil then
    target:AddComponent("colouradder")
end
target.components.colouradder:PushColour("my_mod_flash", 0.3, 0.3, 0.3, 0)
target:DoTaskInTime(0.5, function()
    if target.components.colouradder ~= nil then
        target.components.colouradder:PopColour("my_mod_flash")
    end
end)
```

- 调整与验证：想要客户端同步显示时给目标配 colouraddersync，见 [[reference/components/colouraddersync]]；`PushColour` 任一通道为 nil 直接忽略（[L85](../../../raw/current/scripts/components/colouradder.lua#L85)）。

## 常见陷阱与限制

- **无持久化**：没有 OnSave/OnLoad，存档后颜色不恢复，宿主需在加载流程重建。
- **求和会饱和**：多来源叠加后逐通道 clamp 到 0..1（[L63](../../../raw/current/scripts/components/colouradder.lua#L63)），设计叠加色时按饱和结果调参。
- **zero 推入即弹出**：全零 PushColour 等价 PopColour（[L86-L89](../../../raw/current/scripts/components/colouradder.lua#L86-L89)），想"归零但不移除来源"需保留极小值。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、colouraddersync 驱动链、electrocute_fx 与 player_common 挂载点。
- 游戏运行：未运行。
