---
title: "drawingtool：把附近物品图标绘到目标上"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "用羽毛笔一类工具选取绘制目标附近的物品图标，并调用 drawable 和工具消费回调。"
aliases: ["drawingtool", "羽毛笔", "绘制物品图标", "画小木牌", "无限羽毛笔"]
tags: [component, item, actions, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/drawingtool.lua", "components/drawable.lua", "prefabs/featherpencil.lua", "prefabs/minisign.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# drawingtool：把附近物品图标绘到目标上

## 快速判断

- 职责与适用：挂在绘图工具上，选取目标旁的物品图标，交给 [[reference/components/drawable]] 保存和显示；不绘制自由线条，也不寻找工具持有者背包里的素材。[选源](../../../raw/current/scripts/components/drawingtool.lua#L2-L13)
- 优先使用：正常交互沿用 `DRAW` 动作；自定义工具配置 `SetOnDrawFn`，自定义图样优先配置素材的 `drawimageoverride`／`drawatlasoverride`。[动作门禁](../../../raw/current/scripts/actions.lua#L4282-L4295)、[图像优先序](../../../raw/current/scripts/components/drawingtool.lua#L25-L41)
- 重要副作用：先调用目标 `OnDrawn`，再调用工具回调；原版羽毛笔在后者消耗堆叠中的一个物品。[调用顺序](../../../raw/current/scripts/components/drawingtool.lua#L43-L50)、[消费](../../../raw/current/scripts/prefabs/featherpencil.lua#L6-L8)
- 修改边界：原版工具仅服务端添加组件，目标须自行实现绘制显示及网络字段；本组件没有存读、更新任务或专用移除钩子。[工具工厂](../../../raw/current/scripts/prefabs/featherpencil.lua#L27-L51)、[目标接入](../../../raw/current/scripts/prefabs/minisign.lua#L159-L185)

## 关键接口

| 接口／字段 | 签名或约束 | 条件、作用与证据 |
|---|---|---|
| `FindEntityToDraw` | 全局函数 `(target, tool) → entity或nil` | 以目标为中心半径 1.5，要求 `_inventoryitem`，排除 `INLIMBO`／`notdrawable`，跳过目标、工具及不可见实体；返回迭代中首个符合项，没有自行按距离排序。[实现](../../../raw/current/scripts/components/drawingtool.lua#L2-L13) |
| `GetImageToDraw` | `(target, doer) → image, src, atlas, bgimage, bgatlas`；无素材直接返回 | 图名按 `FunctionOrValue(src.drawimageoverride, src, target, doer)`、非空库存图名、prefab 顺序取值；图集按覆盖值、非空库存 atlas 取值。[实现](../../../raw/current/scripts/components/drawingtool.lua#L25-L41) |
| 背景图 | `src.inv_image_bg` | `image` 须为带 `.tex` 后缀且长度大于 4 的字符串，返回值去后缀；背景 atlas 与默认库存图集相同时省略。[实现](../../../raw/current/scripts/components/drawingtool.lua#L35-L40) |
| `SetOnDrawFn` | `(fn)`；回调 `(tool, target, image, src, atlas, bgimage, bgatlas)` | 替换单个工具回调；组件不自动消费耐久或堆叠。[设置](../../../raw/current/scripts/components/drawingtool.lua#L21-L23)、[调用](../../../raw/current/scripts/components/drawingtool.lua#L43-L50) |
| `Draw` | `(target, image, src, atlas, bgimage, bgatlas)`；无成功返回值 | 仅检查目标及其 drawable 存在；不复查 `CanDraw`、图名或最终图像是否发生变化。[实现](../../../raw/current/scripts/components/drawingtool.lua#L43-L50) |

交互发现只看目标 `drawable` 标签；执行时 `DRAW.fn` 再查 `CanDraw`、获取素材，图名为 nil 返回 `false, "NOIMAGE"`，调用 `Draw` 后返回 true。[动作收集](../../../raw/current/scripts/componentactions.lua#L1134-L1138)、[动作执行](../../../raw/current/scripts/actions.lua#L4282-L4295)

玩家两端都将 `DRAW` 映射到 `dolongaction`；服务端长动作超时调用 `PerformBufferedAction`，不要把提示出现当作已经绘制／消费。[服务端映射](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1447)、[客户端映射](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L729)、[执行点](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8384-L8393)

## 典型接入

### 让原版羽毛笔可重复使用

完整 `modmain.lua`，无额外资源。原版 `featherpencil` 工厂已经注册工具组件、库存物品和堆叠；这里仅替换绘制后的消费策略，不替换选源和目标处理。[注册及组件](../../../raw/current/scripts/prefabs/featherpencil.lua#L34-L58)

```lua
AddPrefabPostInit("featherpencil", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local drawingtool = inst.components.drawingtool
    if drawingtool ~= nil then
        drawingtool:SetOnDrawFn(function(tool, target, image, src, atlas, bgimage, bgatlas)
            -- 保留工具；有意替换原版堆叠消费回调。
        end)
    end
end)
```

- 生效与撤销：覆盖本模组加载后创建／读档的羽毛笔，原版绘图成功路径不再消费；删去此 PostInit 并重新载入会恢复工厂默认回调。不阻止燃烧或其他移除来源。
- 调整与验证：若需增加效果并保留原版消耗，保存 `drawingtool.ondrawfn` 后包装调用；消费可能移除当前工具，应在调用前完成依赖工具实体的处理。[原版消费](../../../raw/current/scripts/prefabs/featherpencil.lua#L6-L8)
- 操作前提：把想画的物品放在可绘小木牌旁，再用工具交互；`FindEntityToDraw` 不是按玩家光标指定素材，多个候选不保证选最近。[查找依据](../../../raw/current/scripts/components/drawingtool.lua#L4-L13)

## 常见陷阱与限制

- **绘制按钮存在却报 NOIMAGE：** 发现阶段只认目标标签；真正选源在执行阶段。确认素材位于目标半径 1.5 内、可见且没有排除标签。[发现](../../../raw/current/scripts/componentactions.lua#L1134-L1138)、[失败分支](../../../raw/current/scripts/actions.lua#L4282-L4293)
- **直接 Draw 绕过禁止重画：** `CanDraw` 是动作层门禁，组件调用者须自行检查；不要直接调用后把无返回值当作失败。[组件](../../../raw/current/scripts/components/drawingtool.lua#L43-L50)、[动作](../../../raw/current/scripts/actions.lua#L4282-L4295)
- **重复绘同图仍消耗工具：** drawable 四个图像字段无变化时不触发目标回调，但工具回调照常调用；如要按实际变化收费，须在调用目标前比较四字段，不能只看工具回调触发。[目标去重](../../../raw/current/scripts/components/drawable.lua#L43-L65)、[工具回调](../../../raw/current/scripts/components/drawingtool.lua#L43-L50)
- **图名带错后缀或 atlas 缺失：** 主图传给目标的是不带 `.tex` 的名字，小木牌宿主再拼后缀并使用 atlas；背景值则由本组件剥除 `.tex`。自定义资源需另行提供，不由工具注册。[取图](../../../raw/current/scripts/components/drawingtool.lua#L31-L40)、[目标消费](../../../raw/current/scripts/prefabs/minisign.lua#L67-L74)
- **覆盖函数返回空串仍会消费：** Lua 的空串为真，图像覆盖不会回退；DRAW 仅拒 nil，目标再把空串归 nil，可能变成清图并执行工具回调。覆盖返回 nil 则继续回退库存图名／prefab；要取消绘图应在动作／工具入口明确拒绝，不能依靠空串或 nil 覆盖值取消。[覆盖值](../../../raw/current/scripts/components/drawingtool.lua#L32-L40)、[动作检查](../../../raw/current/scripts/actions.lua#L4288-L4293)、[归一](../../../raw/current/scripts/components/drawable.lua#L43-L65)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件（actions.lua=R-01、componentactions.lua=R-04、SGwilson.lua=SB-17、SGwilson_client.lua 无专条目）逐条核验，锚点重定位 7 处、结论修订 0 处——DRAW 收集块与 DRAW.fn 原文未变，两端 `DRAW→dolongaction` 映射及 dolongaction 超时 PerformBufferedAction 原文未变（SB-17 改动不触及）；未覆盖：运行验证。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；覆盖选源、图像覆盖／背景、动作门禁与两端 SG 映射、原版羽毛笔工厂及消费、小木牌 drawable 协作。示例基于上述原版组件与消费回调，未审校全部绘图宿主。
- 游戏运行：未运行；无限羽毛笔、多人显示及与其他改写同一回调模组的兼容性未实测。
- 结构检查：定向 `validate_page.py`、源码链接严格检查通过；本页完整 `modmain.lua` 经现有 Full Moon 解析器 `--syntax-only` 通过。仅验证格式、引用和 Lua 5.1 语法。

相关：[[reference/components/drawable]]、[[reference/components/inventoryitem]]、[[reference/components/stackable]]。
