---
title: "savedrotation"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "朝向持久化组件：把实体旋转角写入存档并在载入后处理阶段恢复，解决船旋转甲板上物件随船转后的朝向复原。"
aliases: ["朝向存档", "savedrotation", "旋转持久化"]
tags: [component, data]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/savedrotation.lua", "prefabs/arrowsign.lua", "prefabs/sewing_mannequin.lua", "map/object_layout.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# savedrotation

## 快速判断

- 职责与适用：实现 `OnSave`/`LoadPostPass`，把 `Transform:GetRotation()` 存档并在载入后处理阶段（post pass，实体间关系恢复之后）用 `Transform:LoadRotation` 复原；专为"放在旋转船上、自身也随船旋转"的物件设计。[载入注释](../../../raw/current/scripts/components/savedrotation.lua#L12-L19)
- 优先使用：可摆放在船甲板上且朝向有意义的物件（指示牌、船用保险杠、人偶架、传送基座、嘉年华广场等）挂本组件。[arrowsign 挂载](../../../raw/current/scripts/prefabs/arrowsign.lua#L110)
- 重要副作用：旋转为 0 时 OnSave 返回 nil 不产生存档数据；恢复分两段——LoadPostPass 先 `LoadRotation`，`dodelayedpostpassapply` 为真时再延迟一帧 `SetRotation` 兜底。[两段恢复](../../../raw/current/scripts/components/savedrotation.lua#L12-L26)
- 修改边界：只管旋转不管位移；注释明确"只影响 platform follower 的旋转"；不设标签、无网络变量。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnSave()` | `-> {rotation: number} \| nil` | 旋转非 0 才返回表，否则 nil（不留空键）。[实现](../../../raw/current/scripts/components/savedrotation.lua#L7-L10) |
| `LoadPostPass(newents, data)` | `data: {rotation?}` | `Transform:LoadRotation(data.rotation or 0)`；可选触发延迟应用。[实现](../../../raw/current/scripts/components/savedrotation.lua#L12-L19) |
| `dodelayedpostpassapply` | `boolean \| nil` | 挂载方在构造后显式置 true（原版仅 sewing_mannequin），走 `ApplyPostPassRotation` 兜底。[原版用法](../../../raw/current/scripts/prefabs/sewing_mannequin.lua#L247-L248) |
| `ApplyPostPassRotation(angle)` | `angle: number` | `DoTaskInTime(0, SetRotation(angle))`；注释要求"放在旋转船上的物件"在自身 `OnLoadPostPass` 调用。[实现](../../../raw/current/scripts/components/savedrotation.lua#L21-L26) |

## 典型接入

### 让船甲板物件的朝向跨存档复原（原版参照）

- 前提与选择：服务端；物件可被玩家带上旋转的船。基础恢复靠组件自身 OnSave/LoadPostPass；若物件 OnLoad 顺序晚于船体装配，需按 arrowsign 模式在自己的 `OnLoadPostPass` 里读存档数据再调 `ApplyPostPassRotation`。[arrowsign 模式](../../../raw/current/scripts/prefabs/arrowsign.lua#L48-L50)
- 生效与撤销：组件随实体存读自动工作；`RemoveComponent` 后旧存档里的 rotation 键被忽略，实体回默认朝向。
- 原版依据：布局系统（object_layout）也读取静态布局数据里的 `data.savedrotation.rotation` 直接设置初始旋转，说明"savedrotation.rotation"是布局与存档共用的数据键。[布局消费](../../../raw/current/scripts/map/object_layout.lua#L460-L472)

```lua
-- 服务端：基础挂载（旋转为 0 不写档；默认恢复路径）
inst:AddComponent("savedrotation")

-- 服务端：船载物件按 arrowsign 模式补一次后处理应用
local function onloadpostpass(inst, data)
    if inst.components.savedrotation then
        local rot = data ~= nil and data.savedrotation ~= nil and data.savedrotation.rotation or 0
        inst.components.savedrotation:ApplyPostPassRotation(rot)
    end
end
inst.OnLoadPostPass = onloadpostpass
```

- 调整与验证：角度单位与 `Transform:GetRotation/SetRotation` 一致（度）；`LoadRotation` 与 `SetRotation` 的差异由引擎层处理，组件不做换算。

## 常见陷阱与限制

- **漏掉 OnLoadPostPass 兜底：** 组件内注释明确：放在旋转船上的物件必须在自身 `OnLoadPostPass` 调用 `ApplyPostPassRotation`，否则船先转完、物件朝向可能错位；仅靠 LoadPostPass 的 LoadRotation 对部分平台物件不够。[注释要求](../../../raw/current/scripts/components/savedrotation.lua#L21-L26)
- **误设 dodelayedpostpassapply：** 该标志会额外延迟一帧覆写朝向；原版只有 sewing_mannequin 使用，普通地面物件无需，多一次无意义的延迟写入。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（28 行），核对 arrowsign.lua、sewing_mannequin.lua 用法与 object_layout.lua 布局读取；grep 确认原版挂载点共 9 处（arrowsign 两个变体、atrium_ritual_marking、carnival_plaza、boat_bumpers、charliearena_lightrays、sewing_mannequin、telebase）。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；船载物件朝向复原未实测。

相关：[[reference/components/savedscale]]
