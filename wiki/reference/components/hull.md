---
title: "hull"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "船体附属数据与挂接：登记船半径，挂接船唇、跳板与燃烧定位器，并处理下水动画与跳板皮肤存档。"
aliases: ["船唇挂接", "跳板挂接", "船体半径", "hull"]
tags: [component, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/hull.lua", "prefabs/boat.lua", "prefabs/yotd_boats.lua", "entityscript.lua", "components/boatmagnet.lua", "brains/waveyjoneshandbrain.lua", "brains/crabkingclawbrain.lua", "components/shadowhandspawner.lua", "stategraphs/SGgnarwail.lua", "tuning.lua", "components/boatringdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hull

## 快速判断

- 职责与适用：每条船的船体附属数据容器：保存 `radius`、`boat_lip`、`plank`，提供把实体挂到船上的通用入口，并在下水时播放船唇／跳板动画；不含浮力、平台成员或碰撞体本身。[构造](../../../raw/current/scripts/components/hull.lua#L1-L3)、[挂接](../../../raw/current/scripts/components/hull.lua#L12-L26)
- 原版宿主与注册名：`prefabs/boat.lua` 的 `create_master_pst` 添加，覆盖 `boat`、`boat_pirate`、`boat_ancient`、`boat_otterden`、`boat_grass`、`boat_ice`、`boat_ice_crabking`；`prefabs/yotd_boats.lua` 的 `dragonboat_server` 覆盖 `dragonboat_body` 与 `dragonboat_shadowboat`。[木船加组件](../../../raw/current/scripts/prefabs/boat.lua#L519-L544)、[注册名](../../../raw/current/scripts/prefabs/boat.lua#L1639-L1660)、[龙舟加组件](../../../raw/current/scripts/prefabs/yotd_boats.lua#L386-L420)、[龙舟注册名](../../../raw/current/scripts/prefabs/yotd_boats.lua#L915-L922)
- 优先使用：自定义船在主模拟构造段（prefab 工厂里 `SetPristine` 之后）加 `hull`，先 `SetRadius`，再 `SetBoatLip` 与 `AttachEntityToBoat(..., true)`；跳板用 `SetPlank` 只登记引用，挂接要自己做；放置完成后调 `OnDeployed`。[半径](../../../raw/current/scripts/components/hull.lua#L44-L51)、[部署](../../../raw/current/scripts/components/hull.lua#L59-L73)
- 修改边界：只在服务端；`radius` 是普通字段，不联网；组件没有 `OnRemoveFromEntity`／`OnRemoveEntity`，挂接实体的清理靠各自注册的船 `onremove` 监听；原版船的半径还另存于物理圆柱、`walkableplatform.platform_radius` 与 `boatringdata` 三处。[宿主端](../../../raw/current/scripts/prefabs/boat.lua#L772-L777)、[物理圆柱](../../../raw/current/scripts/prefabs/boat.lua#L395)、[平台半径](../../../raw/current/scripts/prefabs/boat.lua#L412-L413)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `SetRadius(radius)`／`GetRadius(radius)` | 世界单位 → void／数值 | 只写／读本字段；`GetRadius` 的参数没有使用；消费者假设已设置，未设置时返回 `nil`。[L44-L51](../../../raw/current/scripts/components/hull.lua#L44-L51) |
| `SetBoatLip(obj, scale)` | 已生成实体，缩放可选 → void | 记为 `boat_lip`，`AddPlatformFollower` 让船唇跟随，再 `AttachEntityToBoat(obj, 0, 0)`、加 `ignoremouseover`，有 `scale` 时同时缩放 `AnimState`。[L33-L42](../../../raw/current/scripts/components/hull.lua#L33-L42) |
| `AttachEntityToBoat(obj, offset_x, offset_z, parent_to_boat)` | XZ 偏移；布尔 → void | 在 `obj` 上监听船的 `onremove`，船移除时 `Remove` 该对象；`parent_to_boat` 为真时 `SetParent` 并用本地坐标，否则 0 延迟后用世界坐标摆一次，之后不跟随。[L17-L26](../../../raw/current/scripts/components/hull.lua#L17-L26)、[偏移任务](../../../raw/current/scripts/components/hull.lua#L12-L15) |
| `SetPlank(obj)`／`plank` | 实体 → void | 只登记跳板引用供下水动画与存档皮肤使用，不挂接、不设父级；原版先 `AttachEntityToBoat` 再 `SetPlank`。[L29-L31](../../../raw/current/scripts/components/hull.lua#L29-L31) |
| `OnDeployed()` | 无参数 | 有 `walkableplatform` 时 `PushAwayItemsOnBoatPlace`；船唇播 `place_lip`→循环 `lip`；跳板先 `Hide`，1.25 秒后 `Show` 并播 `plank_place`。[L59-L73](../../../raw/current/scripts/components/hull.lua#L59-L73) |
| `OnSave()`／`LoadPostPass(ents, data)` | 存档表 | 只保存跳板的 `skinname`／`skin_id` 并在读档后重新 reskin；船唇、跳板实体与半径由宿主 prefab 重建。[L76-L88](../../../raw/current/scripts/components/hull.lua#L76-L88) |
| 字段 `boat_lip`／`plank`／`radius` | 实体或数值，构造时不初始化 | 由 `SetBoatLip`／`SetPlank`／`SetRadius` 写入；`OnDeployed` 与存档逻辑都按字段是否存在分支。[构造](../../../raw/current/scripts/components/hull.lua#L1-L3) |

## 典型接入

### 原版宿主：木船与龙舟的 hull 装配

- 前提与选择：目标文件 `prefabs/boat.lua` 的 `create_master_pst`（在所有变体的主模拟段调用），以及 `prefabs/yotd_boats.lua` 的 `dragonboat_server`；加组件必须在 `SetPristine` 与服务端门槛之后，此时 `radius` 已从船数据算出。[木船调用点](../../../raw/current/scripts/prefabs/boat.lua#L777)、[龙舟调用点](../../../raw/current/scripts/prefabs/yotd_boats.lua#L403-L404)
- 端与单位：仅服务端；`radius` 是世界单位（木船 4、草船 3、水獭巢 3.75、远古 4、冰船 1.6、龙舟 3），跳板偏移在其基础上加 `PLANK_EDGE_OFFSET`（-0.05）。[半径值](../../../raw/current/scripts/prefabs/boat.lua#L758)、[冰船半径](../../../raw/current/scripts/tuning.lua#L8136)、[龙舟半径](../../../raw/current/scripts/tuning.lua#L8142)、[跳板偏移](../../../raw/current/scripts/prefabs/boat.lua#L518-L543)
- 生效与撤销：这段装配在船生成时执行，船唇／跳板随之生成并跟随；从 kit 部署船时 `ondeploy` 调 `hull:OnDeployed()` 播下水动画。撤销即改回 prefab；已生成的船需重建，运行期不要 `RemoveComponent`。[部署调用](../../../raw/current/scripts/prefabs/boat.lua#L722-L744)、[龙舟部署](../../../raw/current/scripts/prefabs/yotd_boats.lua#L638-L656)
- 原版依据：龙舟的跳板放在 `0.75π` 方向、`radius + 0.05` 处，说明偏移与角度由宿主决定，`hull` 只负责挂接；船唇与跳板 prefab 由船数据传入（有的船只有船唇，如海盗船）。[龙舟跳板](../../../raw/current/scripts/prefabs/yotd_boats.lua#L411-L420)

```lua
-- 摘自 prefabs/boat.lua 的 create_master_pst（非完整 prefab，仅 hull 装配段）
local hull = inst:AddComponent("hull")
hull:SetRadius(radius)

if data.boatlip_prefab then
    hull:SetBoatLip(SpawnPrefab(data.boatlip_prefab), data.scale or 1.0)
end

if data.plank_prefab then
    local walking_plank = SpawnPrefab(data.plank_prefab)
    hull:AttachEntityToBoat(walking_plank, 0, radius + PLANK_EDGE_OFFSET, true)
    hull:SetPlank(walking_plank)
end
```

- 调整与验证：`parent_to_boat = true` 用本地坐标，船转向时跳板随船；换船唇／跳板 prefab 时要同时提供 `place_lip`／`lip`／`plank_place` 动画。示例只做 Lua 5.1 语法检查，未运行。

## 常见陷阱与限制

- 半径未设会晚失败：`GetRadius` 返回 `nil`，`GetSafePhysicsRadius`、`boatmagnet`、`SGgnarwail` 等会做 `nil + 1`／`nil - 1` 一类运算而报错；宿主在加组件后立刻 `SetRadius`。[消费方](../../../raw/current/scripts/prefabs/boat.lua#L482-L483)、[SG](../../../raw/current/scripts/stategraphs/SGgnarwail.lua#L246)、[boatmagnet](../../../raw/current/scripts/components/boatmagnet.lua#L211-L213)、[AI](../../../raw/current/scripts/brains/crabkingclawbrain.lua#L52)、[另一处 AI](../../../raw/current/scripts/brains/waveyjoneshandbrain.lua#L48)
- 半径不是单一来源：`hull.radius` 只供读取，物理圆柱、`walkableplatform.platform_radius`、`boatringdata.radius` 都是各自保存的副本；只改 hull 不会改碰撞、平台成员或旋转几何。[平台半径](../../../raw/current/scripts/prefabs/boat.lua#L412-L413)、[旋转数据](../../../raw/current/scripts/components/boatringdata.lua#L14-L20)
- 非父级挂接不跟随：`parent_to_boat = false` 只在 0 延迟任务里按船当时的世界坐标摆一次，船移动或转向后对象留在原地；需要跟随就用 `true` 或自行 `AddPlatformFollower`。[L12-L25](../../../raw/current/scripts/components/hull.lua#L12-L25)
- 清理绑定在船上，不在组件上：每个挂接对象监听的是船的 `onremove`；即使 `RemoveComponent("hull")`，闭包仍持有组件并会在船移除时删掉这些对象，而半径消费者此时已读不到 `hull`。不要在运行期移除组件。[L17-L18](../../../raw/current/scripts/components/hull.lua#L17-L18)
- 挂接对象会被一并删除：`FinishRemovingEntity` 对船移除时所有通过 `AttachEntityToBoat` 挂上的实体执行 `Remove()`；不要把有独立玩法生命周期的实体交给它。[L5-L10](../../../raw/current/scripts/components/hull.lua#L5-L10)
- 船唇没有失效跟踪：`SetBoatLip` 只存引用，船唇被单独移除后 `boat_lip` 仍非空，`OnDeployed` 会对已失效实体播动画，表现异常／可能报错；换船唇要同步清字段。[L33-L42](../../../raw/current/scripts/components/hull.lua#L33-L42)、[部署动画](../../../raw/current/scripts/components/hull.lua#L64-L67)
- 只存跳板皮肤：`OnSave` 只有 `plank_skinname`／`plank_skin_name`，船唇和跳板实体由 prefab 重新生成；自定义挂接物不在存档里，读档后不会自动回来。[L76-L88](../../../raw/current/scripts/components/hull.lua#L76-L88)、[组件存档](../../../raw/current/scripts/entityscript.lua#L1929-L1949)
- 客户端没有 `hull`：组件只在服务端主模拟段添加，客户端读 `components.hull` 得到 `nil`；客户端要半径请读 `boatringdata`（双端构造）。[服务端门槛](../../../raw/current/scripts/prefabs/boat.lua#L772-L777)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 2 处（tuning OCEAN_ICE_RADIUS／DRAGON_BOAT_RADIUS、entityscript GetPersistData 组件存档段）；无结论修订，仅锚点迁移——依据：全局枢纽 diff 审阅＋变更清单命中 tuning.lua、entityscript.lua、shadowhandspawner.lua（后者在 sources 但本页无正文引用，改动不涉结论）；hull.lua、boat.lua、yotd_boats.lua 及各消费方未变。
- 源码审阅：2026-10-21，steam-buildid:25643519。完整阅读 `components/hull.lua`（90 行），覆盖半径、船唇／跳板挂接、`onremove` 清理、下水动画、皮肤存档与读档 reskin；核对两处宿主 AddComponent 位置与九个注册名、部署调用点及半径消费方（`entityscript`、`boatmagnet`、`SGgnarwail`、`crabkingclawbrain`、`waveyjoneshandbrain`）。未覆盖实机船唇／跳板动画与 skinned 跳板。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py` 通过（0 issue；行数与字节低于 180 行／14 KiB 上限）；`validate_source_links.py --strict` 通过（43 处源码引用、36 个唯一目标、0 error／0 warning）。正文 Lua 示例另存临时文件 `output/group102-snippets.lua`（已清理），`parser_probe --syntax-only` 通过（只查 Lua 5.1 语法，未执行）。

相关：[[reference/components/boatring]]、[[reference/components/boatrotator]]、[[reference/components/boatringdata]]、[[reference/components/walkableplatform]]、[[reference/components/boatphysics]]。
