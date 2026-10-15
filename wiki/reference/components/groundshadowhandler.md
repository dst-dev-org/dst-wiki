---
title: "groundshadowhandler"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "为抛射物等实体生成贴地动态影子 FX，随飞行高度自动缩小，让空中实体有落地阴影。"
aliases: ["groundshadowhandler", "贴地影子", "动态影子缩放", "抛射物阴影"]
tags: [component]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/groundshadowhandler.lua", "prefabs/cannonballs.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# groundshadowhandler

## 快速判断

- 职责与适用：构造时生成一个非联网影子实体（Transform+DynamicShadow），逐帧跟随宿主 x/z 并按宿主 y 高度缩放，供有画面的端使用。([groundshadowhandler.lua:1-24](../../../raw/current/scripts/components/groundshadowhandler.lua#L1-L24)
- 优先使用：`AddComponent` 后调 `SetSize(width, height)` 设影子基础尺寸；不需要手动开更。[groundshadowhandler.lua:21](../../../raw/current/scripts/components/groundshadowhandler.lua#L21 ([groundshadowhandler.lua:35-39](../../../raw/current/scripts/components/groundshadowhandler.lua#L35-L39)
- 重要副作用：影子实体 `persists=false`、`SetCanSleep(false)`，带 FX／CLASSIFIED／NOCLICK／NOBLOCK 标签；宿主移除时随之移除。([groundshadowhandler.lua:4-16](../../../raw/current/scripts/components/groundshadowhandler.lua#L4-L16) ([groundshadowhandler.lua:26-33](../../../raw/current/scripts/components/groundshadowhandler.lua#L26-L33)
- 修改边界：缩放规则固定（y≤2 全尺寸，y≥6 缩到 0.3）；无位置接口，原版消费者直接访问内部字段 `ground_shadow` 调位置。([groundshadowhandler.lua:41-53](../../../raw/current/scripts/components/groundshadowhandler.lua#L41-L53) ([cannonballs.lua:202-205](../../../raw/current/scripts/prefabs/cannonballs.lua#L202-L205)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetSize(width, height)` | 数值对 | 记录原始尺寸并写入影子 `DynamicShadow`；OnUpdate 按比例乘此尺寸。([groundshadowhandler.lua:35-39](../../../raw/current/scripts/components/groundshadowhandler.lua#L35-L39) [groundshadowhandler.lua:51](../../../raw/current/scripts/components/groundshadowhandler.lua#L51 |
| `ground_shadow` | 影子实体（内部字段） | 构造即创建；原版炮弹延迟一帧直接 `SetPosition` 防止闪现在原点。[groundshadowhandler.lua:23](../../../raw/current/scripts/components/groundshadowhandler.lua#L23 ([cannonballs.lua:199-205](../../../raw/current/scripts/prefabs/cannonballs.lua#L199-L205) |
| `OnUpdate(dt)` | 缩放：`Lerp(1.0, 0.3, clamp((y-2)/4))` | 常量 MAX_LERP_HEIGHT=4、MIN_SCALE=0.3、MAX_SCALE=1.0；仅在宿主 valid 时跟随。([groundshadowhandler.lua:41-53](../../../raw/current/scripts/components/groundshadowhandler.lua#L41-L53) |

## 典型接入

### 抛射物贴地阴影

- 前提与选择：纯视觉组件，原版在 `not TheNet:IsDedicated()` 分支内添加，并延迟一帧避免影子闪现在 (0,0,0)。([cannonballs.lua:199-205](../../../raw/current/scripts/prefabs/cannonballs.lua#L199-L205)
- 生效与撤销：构造即开更；宿主 Remove 时 OnRemoveEntity／OnRemoveFromEntity 同实现清理影子。([groundshadowhandler.lua:26-33](../../../raw/current/scripts/components/groundshadowhandler.lua#L26-L33)

```lua
if not TheNet:IsDedicated() then
    inst:DoTaskInTime(0, function(inst)
        inst:AddComponent("groundshadowhandler")
        inst.components.groundshadowhandler:SetSize(1, 0.5)
    end)
end
```

- 调整与验证：尺寸单位与世界单位一致；需要初始位置时仿原版直接设 `ground_shadow.Transform`。

## 常见陷阱与限制

- 加在 dedicated 服务器上无意义（影子为本地视觉），且每帧 OnUpdate 白耗；照抄原版守卫。
- `SetSize` 必须在组件生效后调用一次，否则 OnUpdate 用 nil 尺寸乘法报错（original_width/height 仅由 SetSize 写入）。([groundshadowhandler.lua:35-38](../../../raw/current/scripts/components/groundshadowhandler.lua#L35-L38) [groundshadowhandler.lua:51](../../../raw/current/scripts/components/groundshadowhandler.lua#L51
- 高度取 `Transform:GetWorldPosition()` 的 y：贴地实体 y≈0 时 `(y-2)` 为负被 clamp 到 1，永远全尺寸。

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文；核验炮弹延迟添加与直接调位、其余消费者（mortarball、fishingnetvisualizer、carnivalgame_shooting 等）添加＋SetSize 模式。
- 游戏运行：未运行；未逐帧核对缩放手感。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。

相关：[[reference/components/complexprojectile]]
