---
title: fencerotator：工具旋转围栏与可转向目标
created: 2026-10-23
updated: "2026-10-28"
type: component
summary: 为工具接入围栏旋转动作，优先沿用目标的 SetOrientation，并通过事件交给工具处理消耗。
aliases: [fencerotator, 旋转围栏, 围栏旋转工具, 修改旋转角度]
tags: [component, item, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/fencerotator.lua, componentactions.lua, actions.lua, tuning.lua, prefabs/fence_rotator.lua, prefabs/fence.lua, prefabs/yoth_lance.lua, stategraphs/SGwilson.lua, stategraphs/SGwilson_client.lua, modutil.lua]
reliability: verified
validation: source-reviewed
status: active
---

# fencerotator：工具旋转围栏与可转向目标

## 快速判断

- 职责与适用：组件挂在工具上；`Rotate(target, delta)` 旋转目标，不负责识别围栏、选择目标或装备逻辑。[完整实现](../../../raw/current/scripts/components/fencerotator.lua#L1-L22)
- 优先使用：从原版 `ROTATE_FENCE` 接入；旋转优先调用目标 `SetOrientation`，避免绕过围栏的动画库／门偏移更新。[动作](../../../raw/current/scripts/actions.lua#L5876-L5889) [围栏方向处理](../../../raw/current/scripts/prefabs/fence.lua#L164-L194)
- 重要副作用：先旋转，再向工具发 `fencerotated`，最后在目标位置生成 `fence_rotator_fx`；组件自身不扣耐久。[执行顺序](../../../raw/current/scripts/components/fencerotator.lua#L10-L19)
- 修改边界：原版工具仅服务端添加组件；动作菜单的资格检查不在 `Rotate` 内，直接调用需自行确认目标有效且有 Transform。[宿主端边界](../../../raw/current/scripts/prefabs/fence_rotator.lua#L59-L78) [菜单门禁](../../../raw/current/scripts/componentactions.lua#L2317-L2320)

## 关键接口

| 接口／入口 | 条件、作用与证据 |
|---|---|
| `Rotate(target, delta)` | `target == nil` 立即返回；否则读取当前角度，加上 `delta or TUNING.FENCE_DEFAULT_ROTATION`。没有成功返回值。[实现](../../../raw/current/scripts/components/fencerotator.lua#L5-L19) |
| `target.SetOrientation(target, angle)` | 存在时优先调用；否则直接 `Transform:SetRotation(angle)`。围栏实现还处理宽／窄动画库与门偏移，不应机械替换为裸 Transform。[分派](../../../raw/current/scripts/components/fencerotator.lua#L10-L15) [围栏实现](../../../raw/current/scripts/prefabs/fence.lua#L164-L194) |
| `TUNING.FENCE_DEFAULT_ROTATION` | 当前默认 45 度；原版动作显式传入此值，不仅是组件省略参数时的默认值。[常量](../../../raw/current/scripts/tuning.lua#L6845) [调用](../../../raw/current/scripts/actions.lua#L5882-L5885) |
| `fencerotated` | 事件发在工具，未携带目标数据。`fence_rotator` 宿主监听后 `finiteuses:Use(1)`，而组件不自带消耗配置。[事件](../../../raw/current/scripts/components/fencerotator.lua#L17-L19) [宿主监听与消耗](../../../raw/current/scripts/prefabs/fence_rotator.lua#L23-L27) [注册监听](../../../raw/current/scripts/prefabs/fence_rotator.lua#L65-L69) |

动作收集要求右键、目标带 `rotatableobject`，工具不带 `fire`／`burnt`；`faced_chair` 目标还须有 `cansit`。这里检查燃烧标签的是**工具**，不是目标。原版围栏在工厂添加 `rotatableobject`。[收集](../../../raw/current/scripts/componentactions.lua#L2317-L2320) [围栏标签](../../../raw/current/scripts/prefabs/fence.lua#L614-L622)

`ROTATE_FENCE` 执行时还检查工具拟态揭露；有组件即调用并返回 `true`，不复查上述全部菜单标签，也不使用 `Rotate` 的返回值。玩家服务端和客户端都映射到 `doswipeaction`。[动作](../../../raw/current/scripts/actions.lua#L5876-L5889) [服务端映射](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1674) [客户端映射](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L921)

## 典型接入

### 让原版围栏旋转工具每次旋转 90 度

目标模组 `modmain.lua`；只修改 `fence_rotator`，沿用原版资源与动作。包装组件入口，让显式传入 45 度的原版动作也生效；不修改全局 TUNING，也不影响 `yoth_lance`。[宿主注册](../../../raw/current/scripts/prefabs/fence_rotator.lua#L65-L87) [另一宿主](../../../raw/current/scripts/prefabs/yoth_lance.lua#L79-L92) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L595)

```lua
AddPrefabPostInit("fence_rotator", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local rotator = inst.components.fencerotator
    if rotator == nil then return end
    local rotate = rotator.Rotate
    rotator.Rotate = function(self, target, delta)
        return rotate(self, target, 90)
    end
end)
```

单位为度；本例对该工具的所有 `Rotate` 调用固定使用 90 度，包括其他模组的直接调用。原版事件仍只发一次，工具仍消耗一次耐久；不通过重复调用 45 度实现。新初始化工具不再应用此 PostInit 后恢复原行为，已经转动的目标不会自动转回。[旋转与事件](../../../raw/current/scripts/components/fencerotator.lua#L10-L19) [耐久消费](../../../raw/current/scripts/prefabs/fence_rotator.lua#L23-L27)

新增工具若也要消耗次数，按 [[reference/components/finiteuses]] 配置并监听工具事件；装备／物品栏组装分别复用 [[reference/components/equippable]]、[[reference/components/inventoryitem]]。只挂组件不等于已做完可拾取、网络与装备工厂。

## 常见陷阱与限制

- **只改默认 `delta`**：原版动作显式传入 TUNING 常量，省略参数分支不会命中；修改入口须覆盖实际调用，见上例。
- **把 `Rotate` 返回值当成功标记**：实现无成功返回；反过来动作返回 `true` 也不代表组件检查过全部目标资格。[实现](../../../raw/current/scripts/components/fencerotator.lua#L5-L19) [动作](../../../raw/current/scripts/actions.lua#L5882-L5885)
- **自行改 Transform 再调组件**：会额外旋转一次，还可能绕过目标方向维护。组件已处理 `SetOrientation` 分派，目标缺少此方法才用裸 Transform。[分派](../../../raw/current/scripts/components/fencerotator.lua#L10-L15)
- **从长矛挂载推断所有宿主都会扣耐久**：`yoth_lance` 有 `fencerotator`，但本次核验的工厂没有监听 `fencerotated`；旋转消耗由各宿主决定，不是组件保证。[长矛工厂](../../../raw/current/scripts/prefabs/yoth_lance.lua#L62-L92)

## 验证记录

- 版本沿用：2026-10-25，`steam-buildid:26148036`；原结论复用、锚点迁移 6 处（actions.lua ROTATE_FENCE 两区间、componentactions.lua 收集、tuning.lua 旋转常量、SGwilson.lua／SGwilson_client.lua doswipeaction 映射）；依据：actions.lua／componentactions.lua 枢纽 diff 审阅＋变更清单命中上述五文件，动作收集条件、执行顺序与 FENCE_DEFAULT_ROTATION＝45 同旧版。未运行游戏。
- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整组件、动作收集与执行、玩家双端动作映射、旋转常量、原版工具／长矛工厂及围栏方向实现；跨页物品栏、装备、耐久组装复用同版本页面。
- 结构检查：本页格式、链接、源码锚点与完整 modmain 示例 Lua 5.1 语法检查通过。
- 游戏运行：未运行；90 度旋转、门偏移、不同围栏样式、耐久耗尽与多人预测待实测；不声称支持任意自定义目标或 SG。
