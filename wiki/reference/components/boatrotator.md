---
title: "boatrotator"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "船上的转向控件：把控件绑定到所在船的 boatring，把方向归一化为 -1／0／1，并用状态机表现旋转与停止。"
aliases: ["船上的转向控件", "船转向器", "boat_rotator 组件", "boatrotator"]
tags: [component, lifecycle, state-machine, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/boatrotator.lua", "components/boatring.lua", "components/boatringdata.lua", "stategraphs/SGboatrotator.lua", "prefabs/boat_rotator.lua", "prefabs/boat.lua", "actions.lua", "componentactions.lua", "brains/waveyjoneshandbrain.lua", "recipes.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatrotator

## 快速判断

- 职责与适用：`boat_rotator` 建筑上的服务端控件组件；构造后找出所在船并向船的 `boatring` 注册自己，之后把交互方向归一化、转交 `boatring` 并广播事件。它只维护 `boat` 引用与 `sg.mem.direction` 表现，不改船体变换；整船旋转属于 `boatring`，舵向与推进属于 `boatphysics`（见页尾相关链接）。[构造](../../../raw/current/scripts/components/boatrotator.lua#L1-L16)、[转发](../../../raw/current/scripts/components/boatrotator.lua#L31-L42)
- 优先使用：玩家操控走原版 `ROTATE_BOAT_*` 动作（右键顺时针、左键逆时针、旋转中再点停止）；服务端动作调用 `SetRotationDirection`，程序化控制也用同一入口；AI 侧（waveyjoneshand）则读 `sg.mem.direction` 判断控件是否停转。[动作](../../../raw/current/scripts/actions.lua#L4972-L4994)、[可用动作](../../../raw/current/scripts/componentactions.lua#L215-L226)、[AI](../../../raw/current/scripts/brains/waveyjoneshandbrain.lua#L10-L14)
- 重要副作用：每次设置都会 `boat:PushEvent("rotationdirchanged", dir)`，船上的 `boatring` 监听后把所有 rotator 的状态机切到 `on`／`off`；`SetBoat` 还会把本实体朝向对齐船体并写 `sg.mem.direction`。[事件](../../../raw/current/scripts/components/boatrotator.lua#L40-L41)、[监听](../../../raw/current/scripts/components/boatring.lua#L17-L35)
- 修改边界：只在服务端；宿主要求平台带 `boat` tag、已有 `boatring`，实体有带 `mem.direction` 的状态机；绑定只在构造后的 0 延迟任务里做一次，运行期换船不会自动重绑。[任务](../../../raw/current/scripts/components/boatrotator.lua#L8-L15)、[宿主](../../../raw/current/scripts/prefabs/boat_rotator.lua#L76-L98)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `SetBoat(boat)` | 船实体或 `nil` → void | 解绑旧船、`Transform:SetRotation` 对齐、`boatring:AddRotator(self)`、同步 `sg.mem.direction`；状态机带 `idle` 标签时重进 `idle` 刷新动画；同一船直接返回。[L44-L66](../../../raw/current/scripts/components/boatrotator.lua#L44-L66) |
| `SetRotationDirection(dir)` | 数值 → void | 每次用 `GetCurrentPlatform()` 现取所在船，要求 `boat` tag 且 `boatring` 存在；正数归一为 1、负数 -1、0 为 0；转交 `boatring` 后广播事件。不在船上时静默返回。[L31-L42](../../../raw/current/scripts/components/boatrotator.lua#L31-L42) |
| `boat` | 船实体或 `nil` | 注册所在的船；只在 `SetBoat` 与船事件回调里改，不随平台变化实时更新。[L3-L6](../../../raw/current/scripts/components/boatrotator.lua#L3-L6) |
| `OnRemoveFromEntity()` | 无返回 | 只取消构造时的 0 延迟任务，不解除 `boat`／`boatring` 注册；运行期 `RemoveComponent` 会留下脏表项。[L18-L23](../../../raw/current/scripts/components/boatrotator.lua#L18-L23) |
| `OnRemoveEntity()` | 无返回 | 实体移除时 `SetBoat(nil)`，从船的 rotator 表移除。[L25-L29](../../../raw/current/scripts/components/boatrotator.lua#L25-L29) |
| `OnDeath()` | 无返回 | 监听船的 `death`：本实体有效时 `SetBoat(nil)`；船被 `onremove` 时只把 `self.boat` 清空。[L68-L73](../../../raw/current/scripts/components/boatrotator.lua#L68-L73) |
| 状态机契约 | `sg.mem.direction`、`idle` 标签 | `idle` 按方向选顺／逆时针待机动画；`place` 结束按方向进 `on` 或 `idle`；`off` 结束时把方向清零；`on`／`off` 都是 `busy` 的过场。[SG](../../../raw/current/scripts/stategraphs/SGboatrotator.lua#L12-L110) |

## 典型接入

### 原版宿主：`boat_rotator` 的建造与绑定

- 前提与选择：目标文件 `prefabs/boat_rotator.lua`；唯一 `AddComponent("boatrotator")` 在 `SetPristine` 和服务端门槛之后，随后 `SetStateGraph("SGboatrotator")`；建筑由 `boat_rotator_kit` 部署，注册名 `boat_rotator`、`boat_rotator_kit`、`boat_rotator_kit_placer`。[加组件](../../../raw/current/scripts/prefabs/boat_rotator.lua#L70-L78)、[状态机](../../../raw/current/scripts/prefabs/boat_rotator.lua#L96-L98)、[注册](../../../raw/current/scripts/prefabs/boat_rotator.lua#L106-L108)
- 端与单位：仅服务端；方向是整数 -1／0／1，没有距离或时间单位；配方为木板 ×2、绳子 ×1、齿轮 ×1，科技 `SEAFARING_ONE`。[配方](../../../raw/current/scripts/recipes.lua#L626)
- 生效与撤销：建造到船上后 0 延迟任务用 `GetCurrentPlatform()` 找带 `boat` tag 的平台并 `SetBoat`；锤拆走 `Remove()`→`OnRemoveEntity`→`SetBoat(nil)`；烧毁只 `RemoveComponent("boatrotator")`，不会解绑。[建造](../../../raw/current/scripts/prefabs/boat_rotator.lua#L36-L39)、[锤拆](../../../raw/current/scripts/prefabs/boat_rotator.lua#L16-L24)、[烧毁](../../../raw/current/scripts/prefabs/boat_rotator.lua#L30-L34)
- 原版依据：船上 `boatring` 在 `boat` 系与 `dragonboat_*` 的公共／主机构造里存在；整船转速、旋转更新与 `boatringdata` 的同步细节见 [[reference/components/boatring]]，本页不重复。[船宿主](../../../raw/current/scripts/prefabs/boat.lua#L575-L578)、[数据](../../../raw/current/scripts/components/boatringdata.lua#L30-L40)

### 服务端监听转向变化

- 前提与选择：目标文件模组 `modmain.lua`；`AddPrefabPostInit("boat", ...)` 在船 prefab 完成后执行，监听 `rotationdirchanged` 可取到归一化方向，适合服务端音效、AI，或再转发给客户端做 HUD，不要直接改 `Transform`。[事件生产](../../../raw/current/scripts/components/boatrotator.lua#L40-L41)
- 端与单位：只在服务端接收；`dir` 是 -1、0、1 的整数，事件参数里没有船实体，`inst` 即船。
- 生效与撤销：影响之后生成的 `boat`；删除 postinit 并重载模组即撤销；不影响已生成实例，也不改原版控件。

```lua
-- modmain.lua：服务端监听整船转向变化（事件由 boatrotator 发出）
AddPrefabPostInit("boat", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end

    inst:ListenForEvent("rotationdirchanged", function(_, dir)
        -- dir 已归一化为 -1、0、1；每次设置都会推送，记到船实体供后续逻辑读取
        inst._rotationdir = dir
    end)
end)
```

- 调整与验证：每次 `SetRotationDirection` 都推送一次，同方向重复调用不是边沿事件；程序化控制可在服务端拿到船上的 `boat_rotator` 实体后调 `inst.components.boatrotator:SetRotationDirection(1)`。示例只做 Lua 5.1 语法检查，未运行。

## 常见陷阱与限制

- 绑定只尝试一次：0 延迟任务在下一帧用 `GetCurrentPlatform()` 取一次船，取不到就保持 `boat = nil`，之后没有平台变化重试；把控件搬到另一条船后 `SetRotationDirection` 虽会作用于新船，但新船的 `boatring.rotators` 没有它，旧船仍留着它。[L8-L15](../../../raw/current/scripts/components/boatrotator.lua#L8-L15)、[L44-L57](../../../raw/current/scripts/components/boatrotator.lua#L44-L57)
- 解绑回调传错来源：`RemoveEventCallback("onremove"…)`／`("death"…)` 传的是新船参数；`SetBoat(nil)` 时该值为 `nil`，`EntityScript` 会回退成 `self`，旧船上的监听不会被移除。手动 `SetBoat` 换船后，旧船死亡会把当前船的 rotator 移除，旧船被移除则 `OnBoatRemoved` 把 `self.boat` 清空。[L47-L64](../../../raw/current/scripts/components/boatrotator.lua#L47-L64)、[回退](../../../raw/current/scripts/entityscript.lua#L1249-L1255)
- 缺少空值检查：`SetBoat` 直接访问 `boat.components.boatring` 和 `self.inst.sg.mem`；带 `boat` tag 却没有 `boatring` 的平台、没有状态机或没有 `mem` 的自定义宿主都会在这里报错。[L55-L62](../../../raw/current/scripts/components/boatrotator.lua#L55-L62)、[SG mem](../../../raw/current/scripts/stategraphs/SGboatrotator.lua#L16-L23)
- 方向归一化只在这个入口：正数一律变 1、负数变 -1，`nil` 会在比较时直接出错；绕过它直接调 `boatring:SetRotationDirection(2)` 会得到两倍角速度（见 [[reference/components/boatring]]）。[L37-L38](../../../raw/current/scripts/components/boatrotator.lua#L37-L38)
- 状态机被间接驱动：`SetRotationDirection` 自己不改状态，真正切 `on`／`off` 的是 `boatring` 的 `rotationdirchanged` 监听；控件状态机若不在 `boatring.rotators` 里，就只会转发整船旋转而没有动画。[监听](../../../raw/current/scripts/components/boatring.lua#L17-L35)
- 运行期移除／烧毁：`OnRemoveFromEntity` 只取消任务，不 `SetBoat(nil)`；rotator 留在 `boatring.rotators` 中继续参与转速，完整后果见 [[reference/components/boatring]] 的陷阱。[L18-L23](../../../raw/current/scripts/components/boatrotator.lua#L18-L23)
- 客户端没有该组件：客户端实体上没有 `components.boatrotator`，动作可见性靠 `boatringdata:IsRotating()`，不要在前端读控件组件。[仅服务端](../../../raw/current/scripts/prefabs/boat_rotator.lua#L72-L78)、[客户端判断](../../../raw/current/scripts/componentactions.lua#L215-L226)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 2 处（actions.lua ROTATE_BOAT 三动作、entityscript.lua RemoveEventCallback 回退语义）；结论修订：无，仅锚点迁移。依据：枢纽 diff 审阅＋变更清单命中 actions.lua、componentactions.lua、recipes.lua、entityscript.lua、prefabs/boat.lua（命中区域逐行核对均未漂移）。
- 源码审阅：2026-10-21，steam-buildid:25643519。完整阅读 `components/boatrotator.lua`（75 行），覆盖构造任务、`SetBoat` 绑定／解绑与回调清理、方向归一化、`rotationdirchanged` 生产与消费、`SGboatrotator` 的 `mem.direction` 生命周期；核对宿主 `prefabs/boat_rotator.lua` 的 AddComponent 位置与三个注册名、动作与客户端消费点。未覆盖实机手感、多人并发与模组换船场景。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py` 通过（0 issue；行数与字节低于 180 行／14 KiB 上限）；`validate_source_links.py --strict` 通过（37 处源码引用、31 个唯一目标、0 error／0 warning）。正文 Lua 示例另存临时文件 `output/group102-snippets.lua`（已清理），`parser_probe --syntax-only` 通过（只查 Lua 5.1 语法，未执行）。

相关：[[reference/components/boatring]]、[[reference/components/boatringdata]]、[[reference/components/hull]]、[[reference/components/boatphysics]]、[[reference/components/walkableplatform]]。
