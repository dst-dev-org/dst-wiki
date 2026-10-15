---
title: "boatringdata"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "每条船的旋转与半径数据：客户端可读的 _isrotating 网络布尔，以及双端构造时写入的半径与分段数。"
aliases: ["船旋转数据", "船半径与分段", "客户端读取船是否在转", "boatringdata"]
tags: [component, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/boatringdata.lua", "components/boatring.lua", "components/walkableplatform.lua", "components/walkableplatformmanager.lua", "prefabs/boat.lua", "prefabs/yotd_boats.lua", "componentactions.lua", "components/locomotor.lua", "util.lua", "prefabs/boat_bumpers.lua", "entityscript.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatringdata

## 快速判断

- 职责与适用：每条船的旋转／半径数据组件，不做旋转计算；`boatring` 的几何查询都转调它，客户端用 `IsRotating()` 判断船是否在转。`radius`／`segments` 是普通 Lua 字段，只有 `_isrotating` 是网络布尔。[构造](../../../raw/current/scripts/components/boatringdata.lua#L1-L9)、[客户端读取](../../../raw/current/scripts/components/boatringdata.lua#L30-L32)
- 原版宿主与注册名：`prefabs/boat.lua` 的公共前置 `create_common_pre` 在 `SetPristine` 前添加，随 `boat`、`boat_pirate`、`boat_ancient`、`boat_otterden`、`boat_grass`、`boat_ice`、`boat_ice_crabking` 注册；`prefabs/yotd_boats.lua` 的 `dragonboat_common` 覆盖 `dragonboat_body` 与 `dragonboat_shadowboat`。[木船系](../../../raw/current/scripts/prefabs/boat.lua#L450-L452)、[注册名](../../../raw/current/scripts/prefabs/boat.lua#L1639-L1660)、[龙舟](../../../raw/current/scripts/prefabs/yotd_boats.lua#L198-L200)、[龙舟注册名](../../../raw/current/scripts/prefabs/yotd_boats.lua#L915-L922)
- 优先使用：判断“船是否在转”读 `IsRotating()`（双端可调）；取船缘几何用 `GetRadius()`／`GetNumSegments()`；服务端只通过 `boatring` 间接写 `SetIsRotating`，不要在外部另设。[消费点](../../../raw/current/scripts/componentactions.lua#L218-L222)、[船缘吸附](../../../raw/current/scripts/util.lua#L1501-L1503)
- 修改边界：`SetIsRotating` 断言 `TheWorld.ismastersim`；`SetRadius`／`SetNumSegments` 只改本端字段，不联网；组件没有 `OnSave`、`OnRemoveFromEntity` 或平台注册职责。[同步器](../../../raw/current/scripts/components/boatringdata.lua#L37-L40)、[平台注册](../../../raw/current/scripts/components/walkableplatform.lua#L53-L67)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `radius` | 世界单位，默认 `4` | 船缘吸附、bumper 分段与拾取距离都用它；木船／海盗／远古 4、水獭巢 3.75、草船 3、冰船 1.6、龙舟 3。[字段](../../../raw/current/scripts/components/boatringdata.lua#L4)、[船数据](../../../raw/current/scripts/prefabs/boat.lua#L758-L963)、[冰船](../../../raw/current/scripts/tuning.lua#L8136)、[bumper](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L104-L106) |
| `segments` | 整数，默认 `8` | 船缘圆周分段数；原版船 8 段，龙舟 4 段，影响 bumper 吸附角与 `boatring:GetBumperAtPoint` 的分段。[字段](../../../raw/current/scripts/components/boatringdata.lua#L5)、[龙舟数据](../../../raw/current/scripts/prefabs/yotd_boats.lua#L198-L200) |
| `GetRadius()`／`SetRadius(radius)` | 数值 → 数值／void | getter 双端可读；setter 只写本端字段，不广播、不触发更新。[L14-L20](../../../raw/current/scripts/components/boatringdata.lua#L14-L20) |
| `GetNumSegments()`／`SetNumSegments(segments)` | 整数 → 整数／void | 同上；`boatring:GetNumSegments` 直接转调。[L22-L28](../../../raw/current/scripts/components/boatringdata.lua#L22-L28) |
| `IsRotating()` | 返回 `boolean` | 读 `_isrotating:value()`；客户端可用，不要求主模拟。[L30-L32](../../../raw/current/scripts/components/boatringdata.lua#L30-L32) |
| `SetIsRotating(isrotating)` | 布尔 → void | `assert(TheWorld.ismastersim)` 后写网络布尔；由 `boatring:SetRotationDirection` 按 `dir ~= 0` 调用。[L37-L40](../../../raw/current/scripts/components/boatringdata.lua#L37-L40)、[调用方](../../../raw/current/scripts/components/boatring.lua#L46-L61) |
| `_isrotating` | `net_bool(inst.GUID, "boatringdata._isrotating")` | 构造时初始化为 `false`；没有 dirty 事件名，读取方轮询 `:value()`，不能靠事件监听。[L7-L8](../../../raw/current/scripts/components/boatringdata.lua#L7-L8) |
| `RegisterPlatform` | 不在本组件 | 平台 UID／manager 注册由 `walkableplatform:GetUID`／`OnLoad` 调 `walkableplatformmanager:RegisterPlatform`；`boatringdata` 只是被平台上的 `boatring` 读取。[平台注册](../../../raw/current/scripts/components/walkableplatform.lua#L53-L67)、[manager](../../../raw/current/scripts/components/walkableplatformmanager.lua#L26) |

## 典型接入

### 原版宿主：在公共预构造里写入半径与段数

- 前提与选择：目标文件 `prefabs/boat.lua` 的 `create_common_pre` 与 `prefabs/yotd_boats.lua` 的 `dragonboat_common`；`AddComponent("boatringdata")` 必须在 `entity:SetPristine()` 之前，客户端和服务端都会执行同一段构造，因此两端拿到同一组 `radius`／`segments`。[木船](../../../raw/current/scripts/prefabs/boat.lua#L450-L456)、[龙舟](../../../raw/current/scripts/prefabs/yotd_boats.lua#L198-L206)
- 端与单位：双端；半径是世界单位，段数是整数；写入时机是 prefab 构造，不是读档或 `postinit` 后的运行期。[木船半径](../../../raw/current/scripts/prefabs/boat.lua#L758)、[草船半径](../../../raw/current/scripts/prefabs/boat.lua#L812)
- 生效与撤销：影响用该 prefab 生成的船；`radius` 应与同一船的 `walkableplatform.platform_radius`、物理圆柱半径一致；改动 prefab 后只影响新生成的船，已生成实例需重建。[平台半径](../../../raw/current/scripts/prefabs/boat.lua#L412-L413)
- 原版依据：龙舟把段数改成 4，说明段数与船形绑定；`boatring` 构造时 `assert` 本组件存在，所以宿主必须先加它再加 `boatring`。[断言](../../../raw/current/scripts/components/boatring.lua#L12)、[龙舟段数](../../../raw/current/scripts/prefabs/yotd_boats.lua#L198-L200)

### 客户端只读：轮询所在船的旋转状态

- 前提与选择：目标文件模组 `modmain.lua`；客户端用 `GetCurrentPlatform()` 找当前船，再读 `components.boatringdata`；适合 HUD、提示或本地表现，不需要服务端 RPC。[平台获取](../../../raw/current/scripts/entityscript.lua#L1856-L1865)
- 端与单位：双端都会执行这段只读逻辑；客户端拿它做本地表现，服务端执行也没有写入；`IsRotating()` 返回布尔，`GetRadius()` 返回世界单位；`SetIsRotating`／`SetRadius` 不要在客户端调用。
- 生效与撤销：影响本地玩家表现，不改变服务端状态；移除该初始化并重载模组即撤销。

```lua
-- modmain.lua：客户端读取所在船的旋转状态（只读）
local function CheckBoatRotation(inst)
    inst:DoPeriodicTask(0.5, function()
        local boat = inst:GetCurrentPlatform()
        local data = boat and boat.components.boatringdata
        if data then
            print("rotating", data:IsRotating(), "radius", data:GetRadius())
        end
    end)
end
AddPlayerPostInit(CheckBoatRotation)
```

- 调整与验证：`0.5` 是秒；`IsRotating` 只表示“在转”，不含方向与速度；示例只做 Lua 5.1 语法检查，未运行。

## 常见陷阱与限制

- `radius`／`segments` 不是网络变量：服务端在运行期改 `SetRadius` 只改本端，客户端仍用旧值做船缘吸附／bumper 定位；原版只在双端都会执行的 prefab 构造里设置它们。[L4-L5](../../../raw/current/scripts/components/boatringdata.lua#L4-L5)、[设置点](../../../raw/current/scripts/prefabs/boat.lua#L450-L452)
- 默认值会掩盖漏设：组件默认 `4`／`8`；自定义船忘记 `SetRadius` 时不会报错，但 `boatring`、bumper、`SnapToBoatEdge` 与物理／平台半径会各说各话。[默认值](../../../raw/current/scripts/components/boatringdata.lua#L4-L5)、[吸附](../../../raw/current/scripts/util.lua#L1501-L1503)
- `SetIsRotating` 只能在服务端调：客户端调用会直接命中 `assert(TheWorld.ismastersim)`；客户端要改状态只能发事件／动作给服务端。[L37-L40](../../../raw/current/scripts/components/boatringdata.lua#L37-L40)
- 网络布尔没有 dirty 事件：`net_bool` 第三个参数为空，不能 `ListenForEvent` 某个 dirty 事件，只能在需要时读 `:value()`（原版在构造动作列表或判断跳船时读）。[L7-L8](../../../raw/current/scripts/components/boatringdata.lua#L7-L8)、[跳船判断](../../../raw/current/scripts/components/locomotor.lua#L1296-L1302)
- 只有“是否在转”，没有方向：`boatring` 的方向与旋转速度只在服务端；客户端的动作列表在旋转时只显示“停止”，未旋转时按左／右键分别给出逆／顺时针动作，不能从本组件推断当前方向。[动作列表](../../../raw/current/scripts/componentactions.lua#L215-L226)
- “在转”不等于“有控件”：`boatring` 在 0 个 rotator 时仍按 1 个继续转（失控船表现），`_isrotating` 仍为真；停止只由 `boatring:SetRotationDirection(0)` 写入。[转速说明](../../../raw/current/scripts/components/boatring.lua#L107-L110)
- 不要当平台注册器：本组件不调用 `RegisterPlatform`，也没有 UID；需要平台能力应加 `walkableplatform`，两者的半径要手动保持一致。[平台注册](../../../raw/current/scripts/components/walkableplatform.lua#L53-L67)
- 没有存档与清理：没有 `OnSave`，读档靠 prefab 重新构造出同样的半径／分段；运行期 `RemoveComponent("boatringdata")` 会让 `boatring` 的 `assert`／`GetRadius` 直接失败。[断言](../../../raw/current/scripts/components/boatring.lua#L12)、[转调](../../../raw/current/scripts/components/boatring.lua#L38-L44)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 2 处（tuning.lua OCEAN_ICE_RADIUS、entityscript.lua GetCurrentPlatform）；结论修订：无，仅锚点迁移。依据：枢纽 diff 审阅＋变更清单命中 componentactions.lua、util.lua、components/locomotor.lua、entityscript.lua、tuning.lua、prefabs/boat.lua（命中区域逐行核对均未漂移）。
- 源码审阅：2026-10-21，steam-buildid:25643519。完整阅读 `components/boatringdata.lua`（42 行），覆盖构造、四个 getter／setter、网络布尔与主模拟断言；核对两处宿主 AddComponent 位置与九个注册名、`boatring`／`componentactions`／`locomotor`／`util`／`boat_bumpers` 的消费点。未覆盖实机网络抖动与多人同时登船。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py` 通过（0 issue；行数与字节低于 180 行／14 KiB 上限）；`validate_source_links.py --strict` 通过（44 处源码引用、32 个唯一目标、0 error／0 warning）。正文 Lua 示例另存临时文件 `output/group102-snippets.lua`（已清理），`parser_probe --syntax-only` 通过（只查 Lua 5.1 语法，未执行）。

相关：[[reference/components/boatring]]、[[reference/components/boatrotator]]、[[reference/components/hull]]、[[reference/components/walkableplatform]]、[[reference/components/boatphysics]]。
