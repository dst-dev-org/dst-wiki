---
title: "drawable：接收和保存绘制图样"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "维护小木牌、相框的可绘制状态与图像数据，由宿主回调处理外观、名称和绘制后锁定。"
aliases: ["drawable", "可绘制目标", "小木牌重绘", "物品相框", "绘图存档"]
tags: [component, item, building, actions, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/drawable.lua", "components/drawingtool.lua", "components/burnable.lua", "prefabs/minisign.lua", "prefabs/decor_pictureframe.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# drawable：接收和保存绘制图样

## 快速判断

- 职责与适用：挂在被绘制的实体上，维护主图／背景及图集四字段、`candraw` 状态和变更回调；选择附近素材及工具消耗归 [[reference/components/drawingtool]]。[数据与回调](../../../raw/current/scripts/components/drawable.lua#L9-L25)、[工具协作](../../../raw/current/scripts/components/drawingtool.lua#L43-L50)
- 优先使用：`SetCanDraw` 控制正常交互可绘状态，`SetOnDrawnFn` 对接外观，`OnDrawn` 写入图样或用 nil 清空；原版小木牌回调同时负责画后锁定。[接口](../../../raw/current/scripts/components/drawable.lua#L31-L65)、[宿主回调](../../../raw/current/scripts/prefabs/minisign.lua#L67-L93)
- 重要副作用：仅四个图像字段有变化才调用回调；`imagesource` 不参与比较，也不保存。读档以 `src=nil` 重放图像变更，回调须能处理。[变化判断](../../../raw/current/scripts/components/drawable.lua#L43-L65)、[存读](../../../raw/current/scripts/components/drawable.lua#L83-L96)
- 修改边界：原版服务端添加组件，客户端动作依赖 pristine 前标签，显示与名字网络数据由宿主实现；仅添加 drawable 不会自动画出图像。[小木牌端边界](../../../raw/current/scripts/prefabs/minisign.lua#L159-L185)、[相框端边界](../../../raw/current/scripts/prefabs/decor_pictureframe.lua#L127-L147)

## 关键接口

| 接口／字段 | 签名或约束 | 条件、作用与证据 |
|---|---|---|
| `SetCanDraw`／`CanDraw` | `(boolean)`／`() → candraw` | 默认 true；赋值回调增删 `drawable` 标签，不清已有图像；正常 DRAW 执行才检查这一门禁。[状态](../../../raw/current/scripts/components/drawable.lua#L1-L37)、[动作](../../../raw/current/scripts/actions.lua#L4282-L4295) |
| `SetOnDrawnFn` | `(fn)`；回调 `(inst, image, src, atlas, bgimage, bgatlas)` | 替换单个回调；显示、名字、锁定、音效由宿主决定。[设置](../../../raw/current/scripts/components/drawable.lua#L39-L41)、[小木牌实现](../../../raw/current/scripts/prefabs/minisign.lua#L67-L93) |
| `OnDrawn` | `(image, src, atlas, bgimage, bgatlas)`；无成功返回 | 四种图像字段的空串归一为 nil，再比较字段决定回调；本方法不查 candraw。[实现](../../../raw/current/scripts/components/drawable.lua#L43-L65) |
| `GetImage`／`GetAtlas`／`GetBGImage`／`GetBGAtlas` | `() → 对应字段` | 读取组件中当前保存的图名和图集，不等于宿主显示名。[实现](../../../raw/current/scripts/components/drawable.lua#L67-L81) |
| `OnSave`／`OnLoad` | 有主图时返回四字段数据；加载时经 `OnDrawn` 恢复 | 不保存 src、candraw 或回调；主图为 nil 时无组件数据，读入有 image 的数据才重放。[实现](../../../raw/current/scripts/components/drawable.lua#L83-L96) |
| `OnRemoveFromEntity` | `()` | 只移除 `drawable` 标签，不负责撤销宿主 symbol 覆盖或名字字段。[实现](../../../raw/current/scripts/components/drawable.lua#L27-L29) |

## 典型接入

### 让放置的小木牌能够重新绘图

完整 `modmain.lua`，仅修改原版 `minisign`，复用其资源和显示回调。默认小木牌画后锁定，燃烧时也禁止绘制；因此同时包装绘制与熄灭回调，保留原逻辑后恢复非燃烧时可绘。[原锁定](../../../raw/current/scripts/prefabs/minisign.lua#L55-L93)、[回调设置](../../../raw/current/scripts/prefabs/minisign.lua#L184-L195)

```lua
AddPrefabPostInit("minisign", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local drawable = inst.components.drawable
    local burnable = inst.components.burnable
    if drawable == nil or burnable == nil then
        return
    end
    local function RefreshCanDraw(sign)
        sign.components.drawable:SetCanDraw(
            not sign.components.burnable:IsBurning())
    end
    local olddrawn = drawable.ondrawnfn
    drawable:SetOnDrawnFn(function(sign, image, src, atlas, bgimage, bgatlas)
        if olddrawn ~= nil then
            olddrawn(sign, image, src, atlas, bgimage, bgatlas)
        end
        RefreshCanDraw(sign)
    end)
    local oldextinguish = burnable.onextinguish
    burnable:SetOnExtinguishFn(function(sign)
        if oldextinguish ~= nil then
            oldextinguish(sign)
        end
        RefreshCanDraw(sign)
    end)
    RefreshCanDraw(inst)
end)
```

- 生效与撤销：新建和读档的小木牌可重画，原点燃回调仍关闭绘制；熄灭时恢复。组件读档会调用包装后的绘图回调，原宿主的 `OnLoad` 继续恢复名称。移除这段并重载恢复原版策略，不新增存档字段。[读图](../../../raw/current/scripts/components/drawable.lua#L92-L96)、[宿主名称读档](../../../raw/current/scripts/prefabs/minisign.lua#L132-L142)
- 边界与替换点：不修改 `minisign_drawn` 物品或相框；不恢复已烧毁实体。实际绘图仍需附近素材和工具，仍沿用羽毛笔消费。熄灭回调调用前 `burning` 已置 false。[熄灭顺序](../../../raw/current/scripts/components/burnable.lua#L484-L492)、[设置入口](../../../raw/current/scripts/components/burnable.lua#L106-L109)
- 验证重点：画不同物品、画同图、烧着时交互、熄灭后重画、保存重载和挖起再放置；这些场景尚未游戏运行验证。

### 选择原版显示／转移模式

| 需求 | 可参考的实际宿主及边界 |
|---|---|
| 放置目标显示物品图标并锁定 | `minisign` 注册到 `fn`，`OnDrawnFn` 覆盖 `SWAP_SIGN` 和背景，设置 `candraw=false`；名称另用 `_imagename` net_string。[回调](../../../raw/current/scripts/prefabs/minisign.lua#L67-L93)、[注册](../../../raw/current/scripts/prefabs/minisign.lua#L265-L269) |
| 挖起后保留图样，重新部署 | 原版挖掘生成 `minisign_drawn`，复制四字段与名称；该物品工厂不可堆叠且 `CanDraw=false`，部署再复制到新木牌。[挖起／部署](../../../raw/current/scripts/prefabs/minisign.lua#L28-L53)、[物品工厂](../../../raw/current/scripts/prefabs/minisign.lua#L232-L249) |
| 可拾取相框更新库存图标和显示名 | `decor_pictureframe` 注册自己的显示回调，更新 symbol、名称网络值与库存图标；`OnSave`／`OnLoad` 另维护显示名，不由 drawable 包办。[回调](../../../raw/current/scripts/prefabs/decor_pictureframe.lua#L35-L68)、[名称存读](../../../raw/current/scripts/prefabs/decor_pictureframe.lua#L89-L111)、[注册](../../../raw/current/scripts/prefabs/decor_pictureframe.lua#L178) |

## 常见陷阱与限制

- **只调用 SetCanDraw(false) 却仍被代码重绘：** 它控制标签及正常动作门禁；直接 `OnDrawn` 或工具 `Draw` 不检查，内部复制／读档仍能写入。[动作边界](../../../raw/current/scripts/actions.lua#L4282-L4295)、[直接写入](../../../raw/current/scripts/components/drawable.lua#L43-L65)
- **同图换素材没有回调：** 四图字段未变时，换 `src` 不算变化；不能依靠 src 更新触发名称或奖励。工具消费仍可能执行。[比较](../../../raw/current/scripts/components/drawable.lua#L56-L63)、[工具顺序](../../../raw/current/scripts/components/drawingtool.lua#L43-L50)
- **读档 src=nil 导致报错或名字丢失：** src 不持久化；宿主须像小木牌／相框一样处理 nil 并独立恢复名称，不保存实体引用冒充图样数据。[组件存读](../../../raw/current/scripts/components/drawable.lua#L83-L96)、[小木牌名称](../../../raw/current/scripts/prefabs/minisign.lua#L123-L142)
- **设置 nil 清图后永久不能画：** 清理外观和解锁是宿主回调责任；原版仅在未燃烧时重新启用。自定义回调不要漏掉背景清理和可绘状态。[小木牌清理分支](../../../raw/current/scripts/prefabs/minisign.lua#L84-L90)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件（actions.lua=R-01、componentactions.lua=R-04）对照更新报告逐条核验，锚点重定位 2 处、结论修订 0 处——DRAW.fn 原文未变（动作区整体 +3 行移），本页其余 sources 均未漂移；未覆盖：运行验证。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；覆盖状态标签、四字段去重、空串处理、回调与存读／移除，小木牌工厂及图样转移，相框名称／图标协作，重绘示例的点燃与熄灭顺序。未据此断言全部宿主统一同步图像。
- 游戏运行：未运行；示例列出的重画、燃烧、存读与多人交互场景待游戏实测。
- 结构检查：定向 `validate_page.py`、源码链接严格检查通过；本页完整 `modmain.lua` 经现有 Full Moon 解析器 `--syntax-only` 通过。仅验证格式、引用和 Lua 5.1 语法。

相关：[[reference/components/drawingtool]]、[[reference/components/burnable]]、[[reference/components/inventoryitem]]。
