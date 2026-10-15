---
title: "boatring"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "服务端整船旋转与船缘 bumper 登记：驱动船体朝向，并给 hullhealth 提供命中点上的 bumper。"
aliases: ["整船旋转", "船体转向", "船缘保险杠", "boatring"]
tags: [component, lifecycle, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/boatring.lua", "components/boatringdata.lua", "components/boatrotator.lua", "components/hullhealth.lua", "prefabs/boat.lua", "prefabs/yotd_boats.lua", "prefabs/boat_rotator.lua", "prefabs/boat_bumpers.lua", "componentactions.lua", "actions.lua", "util.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatring

## 快速判断

- 职责与适用：不是独立物品，而是每条船上的服务端组件：维护整船旋转方向与速度、登记船缘 bumper 与驾驶用 rotator。`SetRotationDirection` 直接改船的 `Transform` 朝向（整船和甲板一起转），不是 boatphysics 的舵向或速度。宿主是 `boat` 系与 `dragonboat_body`／`dragonboat_shadowboat`。[构造](../../../raw/current/scripts/components/boatring.lua#L5-L36)、[宿主](../../../raw/current/scripts/prefabs/boat.lua#L576)
- 优先使用：玩家操控走 `boat_rotator`（`boatrotator` 组件 → `boatring`）；判断船是否在转读 `boatringdata:IsRotating()`（客户端有）；碰撞吸收用 `GetBumperAtPoint`。[控件](../../../raw/current/scripts/components/boatrotator.lua#L31-L42)、[数据](../../../raw/current/scripts/components/boatringdata.lua#L30-L32)
- 重要副作用：`OnUpdate` 每更新帧 `Transform:SetRotation`；`SetRotationDirection` 写网络布尔 `boatringdata._isrotating`；死亡时只 KillSound("boat_movement")。[OnUpdate](../../../raw/current/scripts/components/boatring.lua#L105-L112)、[OnDeath](../../../raw/current/scripts/components/boatring.lua#L101-L103)
- 修改边界：仅服务端（客户端没有 boatring，只有 boatringdata）；构造强依赖 boatringdata；没有 `OnRemoveFromEntity`，不要运行期移除。

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `rotationdirection` | 数值；约定 -1／0／1 | 旋转方向。`boatring` 本身不归一化，`boatrotator` 会先归一化；直接传 2 会按两倍速转。[设置](../../../raw/current/scripts/components/boatring.lua#L46-L61)、[归一化](../../../raw/current/scripts/components/boatrotator.lua#L37-L38) |
| `rotate_speed`／`max_rotate_speed` | 度／更新帧；默认 0.5／2 | 每次 `OnUpdate` 转角 = `min(max(1, #rotators) × rotate_speed, max_rotate_speed) × direction`，不使用 `dt`。[L8-L9](../../../raw/current/scripts/components/boatring.lua#L8-L9)、[L105-L112](../../../raw/current/scripts/components/boatring.lua#L105-L112) |
| `rotators` | rotator 组件表 | 每个 `boatrotator` 在 `SetBoat` 时 `AddRotator`；数量决定转速，0 个时按 1 个算（失控船）。[AddRotator](../../../raw/current/scripts/components/boatring.lua#L75-L81)、[SetBoat](../../../raw/current/scripts/components/boatrotator.lua#L44-L66) |
| `SetRotationDirection(dir)` | 方向值 | 写 `rotationdirection`、同步 `boatringdata:SetIsRotating(dir ~= 0)`；非 0 启动更新，0 停止。不会主动推 `rotationdirchanged`。[实现](../../../raw/current/scripts/components/boatring.lua#L46-L61) |
| `GetRotationDirection()` | 当前方向值 | 读最近一次设置值；`boatrotator:SetBoat` 用它初始化 `sg.mem.direction`。[L63-L65](../../../raw/current/scripts/components/boatring.lua#L63-L65) |
| `AddRotator`／`RemoveRotator` | 组件引用 | 只增删表项，不启停旋转；是否继续转由 `rotationdirection` 与更新开关决定。[L75-L81](../../../raw/current/scripts/components/boatring.lua#L75-L81) |
| `AddBumper`／`RemoveBumper` | 实体引用 | 船缘 bumper 的登记表，供 `GetBumperAtPoint` 与 `hullhealth` 使用；不负责生成或存档 bumper。[L67-L73](../../../raw/current/scripts/components/boatring.lua#L67-L73) |
| `GetBumperAtPoint(x, z)` | 世界坐标 → bumper 或 nil | 以船心到 bumper 的方向为锥心，按分段角 `(360/segments)/RADIANS` 作为总张角（±一半）判断；命中返回第一个有效 bumper。分段见 `boatringdata`。[L83-L99](../../../raw/current/scripts/components/boatring.lua#L83-L99)、[IsWithinAngle](../../../raw/current/scripts/util.lua#L1456-L1472) |
| `GetRadius()`／`GetNumSegments()` | 世界单位；整数 | 直接转调 `boatringdata`；默认 4／8，龙舟 3／4。[L38-L44](../../../raw/current/scripts/components/boatring.lua#L38-L44)、[龙舟数据](../../../raw/current/scripts/prefabs/yotd_boats.lua#L198-L200) |
| `OnSave`／`OnLoad` | `{ rotationdirection }` | 只存方向，不存 bumper／rotator／实际角度；读档立刻恢复旋转。实际朝向由 `savedrotation` 另存。[L114-L125](../../../raw/current/scripts/components/boatring.lua#L114-L125) |

### 旋转速度与几何

- 转角是角度制，`ReduceAngle` 归一化到 0..360；速度按“每个更新帧”叠加，60 fps 下 `0.5` 约 30°/秒，30 fps 下约 15°/秒，掉帧会变慢。[OnUpdate](../../../raw/current/scripts/components/boatring.lua#L105-L112)
- 0 个 rotator 且方向非 0 时仍会转：`numrotators = math.max(1, #self.rotators)`，源码注释把这当作“失控的船”表现。[L107-L109](../../../raw/current/scripts/components/boatring.lua#L107-L109)
- bumper 覆盖是半个分段角：8 段时每个 bumper 覆盖 ±22.5°，4 段时 ±45°；bumper 由 prefab 吸附到分段中心，少了 bumper 就会留缺口。已被移除的 bumper 跳过，但不会从表里剔除。[GetBumperAtPoint](../../../raw/current/scripts/components/boatring.lua#L83-L99)、[吸附](../../../raw/current/scripts/util.lua#L1492-L1513)

## 典型接入

### 原版链路：boat_rotator → boatrotator → boatring

- 控件实体是 `boat_rotator`（可部署建筑，另带 `_kit` 与 placer）；服务端加 `boatrotator` 组件，不直接碰 `boatring`。[注册](../../../raw/current/scripts/prefabs/boat_rotator.lua#L78)、[L106-L108](../../../raw/current/scripts/prefabs/boat_rotator.lua#L106-L108)
- `boatrotator` 构造后用 0 延迟任务取 `GetCurrentPlatform()`，要求平台带 `boat` tag 且已有 `boatring`，然后 `AddRotator` 并同步 `sg.mem.direction`；船被移除或死亡时 `RemoveRotator`。[SetBoat](../../../raw/current/scripts/components/boatrotator.lua#L44-L66)
- 玩家动作 `ROTATE_BOAT_CLOCKWISE`／`COUNTERCLOCKWISE`／`STOP` 调 `boatrotator:SetRotationDirection`，后者改 `boatring` 并 `PushEvent("rotationdirchanged")`；`boatring` 监听该事件，把所有 rotator 的状态机切到 `on`／`off`。[动作](../../../raw/current/scripts/actions.lua#L4972-L4994)、[转发](../../../raw/current/scripts/components/boatrotator.lua#L31-L42)、[监听](../../../raw/current/scripts/components/boatring.lua#L17-L35)
- bumper 链：`boat_bumper_*` 建造时吸附并 `AddBumper`，死亡时 `RemoveBumper`；读档由船的 `OnLoadPostPass` 按半径重新登记并修正朝向。撞击时 `hullhealth` 用 `GetBumperAtPoint` 让 bumper 吃伤害。[建造](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L160-L179)、[读档](../../../raw/current/scripts/prefabs/boat.lua#L165-L186)、[死亡](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L210-L217)、[hullhealth](../../../raw/current/scripts/components/hullhealth.lua#L167-L185)
- 客户端没有 `boatring`：`COMPONENT_ACTIONS` 用 `boatringdata:IsRotating()` 决定显示“停止”还是顺／逆时针；`locomotor` 在船间跳跃时也用它判断平台是否静止。[actions](../../../raw/current/scripts/componentactions.lua#L215-L226)、[跳跃](../../../raw/current/scripts/components/locomotor.lua#L1296-L1302)
- 宿主与注册名：`boat`、`boat_pirate`、`boat_ancient`、`boat_otterden`、`boat_grass`、`boat_ice`、`boat_ice_crabking`，以及 `dragonboat_body`／`dragonboat_shadowboat`；`boatringdata` 在这些 prefab 的公共预构造里先加，`boatring` 在 master postinit 里后加。[boat.lua](../../../raw/current/scripts/prefabs/boat.lua#L450-L452)、[L576](../../../raw/current/scripts/prefabs/boat.lua#L576)、[注册](../../../raw/current/scripts/prefabs/boat.lua#L1639-L1665)、[龙舟](../../../raw/current/scripts/prefabs/yotd_boats.lua#L915-L920)

### 模组调参（服务端 postinit）

- 目标文件：模组 `modmain.lua`；`AddPrefabPostInit("boat", fn)` 在 prefab 工厂之后调用，组件已存在；客户端也会调用，先判 `ismastersim`。
- 端与资源：仅服务端字段；无需新资源；客户端只通过 `boatringdata` 的网络布尔感知“是否在转”。
- 生效与撤销：只影响之后生成的 `boat`（其它宿主分别注册）；删 postinit 并重载即撤销。已生成的船不受影响。
- 单位：度／更新帧；`max_rotate_speed` 只是上限，实际速度还乘 rotator 数量。

```lua
-- modmain.lua：放慢整船旋转（服务端字段）
AddPrefabPostInit("boat", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local boatring = inst.components.boatring
    if boatring == nil then
        return
    end
    boatring.rotate_speed = 0.25       -- 度/更新帧，默认 0.5
    boatring.max_rotate_speed = 1.0    -- 度/更新帧上限，默认 2
end)
```

- 验证：示例只做 Lua 5.1 语法检查，未在游戏内执行。

## 常见陷阱与限制

- 构造顺序不能反：`BoatRing` 构造第一句就 `assert(inst.components.boatringdata ~= nil)`，先加 `boatring` 的 prefab 会直接报错；原版都是在公共预构造里先加 `boatringdata`。[assert](../../../raw/current/scripts/components/boatring.lua#L12)
- 移除组件会留下坏回调：没有 `OnRemoveFromEntity`，构造时挂的 `death` 监听仍在；死亡时 `on_death` 执行 `inst.components.boatring:OnDeath()`（无 nil 检查），移除后再死会报错。[监听](../../../raw/current/scripts/components/boatring.lua#L1-L3)、[L34-L35](../../../raw/current/scripts/components/boatring.lua#L34-L35)
- 正在转时拆掉最后一个 rotator 不会停：`BoatRotator:SetBoat(nil)` 只 `RemoveRotator`，不清 `rotationdirection`；`OnUpdate` 又把 0 个 rotator 当 1 个，于是船继续以 `rotate_speed` 自转，且没有控件能再发停止。[SetBoat](../../../raw/current/scripts/components/boatrotator.lua#L44-L51)、[L107-L109](../../../raw/current/scripts/components/boatring.lua#L107-L109)
- 烧毁 rotator 会留下脏表项：`onburnt` 只 `RemoveComponent("boatrotator")`，而组件的 `OnRemoveFromEntity` 只取消初始化任务、不调 `SetBoat(nil)`；这条 rotator 仍留在 `boatring.rotators` 里继续计速（若在转）。[onburnt](../../../raw/current/scripts/prefabs/boat_rotator.lua#L30-L34)、[清理](../../../raw/current/scripts/components/boatrotator.lua#L18-L23)
- 帧率依赖：`OnUpdate(dt)` 完全不用 `dt`，转速随帧率变化；改 `rotate_speed` 时按“每更新帧”而非“每秒”估算。[L105-L112](../../../raw/current/scripts/components/boatring.lua#L105-L112)
- 死亡不停止旋转：`OnDeath` 只 `KillSound("boat_movement")`，不清方向、不 `StopUpdatingComponent`、不清 rotator／bumper；若在转，沉没动画期间仍会转。[OnDeath](../../../raw/current/scripts/components/boatring.lua#L101-L103)
- 存读只恢复方向：`OnSave` 只有 `rotationdirection`，读档 `OnLoad` 立即 `SetRotationDirection`；bumper／rotator 靠各自的 prefab 重建，自定义旋转源要自己重新注册。[L114-L125](../../../raw/current/scripts/components/boatring.lua#L114-L125)
- `dir` 不归一化：`boatring:SetRotationDirection(2)` 会得到两倍角速度，`GetRotationDirection()` 也返回 2；只有 `boatrotator` 会夹到 -1／0／1。[L37](../../../raw/current/scripts/components/boatrotator.lua#L37)、[L46-L50](../../../raw/current/scripts/components/boatring.lua#L46-L50)
- `boatringdata` 的同步范围：`_isrotating` 是 `net_bool`，但 `SetIsRotating` 断言 `TheWorld.ismastersim`；`radius`／`segments` 是普通字段，不在网络上同步，两端都必须设置。[数据](../../../raw/current/scripts/components/boatringdata.lua#L4-L8)、[同步器](../../../raw/current/scripts/components/boatringdata.lua#L37-L40)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 1 处（actions.lua ROTATE_BOAT 三动作）；结论修订：无，仅锚点迁移。依据：枢纽 diff 审阅＋变更清单命中 actions.lua、componentactions.lua、util.lua、components/locomotor.lua、prefabs/boat.lua（命中区域逐行核对均未漂移，boat.lua 仅行尾差异）。
- 源码审阅：2026-10-21，steam-buildid:25643519。覆盖构造与 assert、SetRotationDirection／OnUpdate／OnDeath、rotator 与 bumper 增删、GetBumperAtPoint 与 `IsWithinAngle` 几何、OnSave／OnLoad；核对全部宿主注册名，以及 `boatrotator`、`boat_bumpers`、`hullhealth`、`componentactions`、`locomotor` 的调用点；未覆盖表现层。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py` 通过（0 issue；99 行，低于 200 行／16 KiB 上限）；`validate_source_links.py --strict` 通过（55 处引用、0 error／0 warning）；正文 Lua 示例另存 409 字节，`parser_probe --syntax-only` 通过（只查 Lua 5.1 语法，未执行）。

相关：[[reference/components/boatphysics]]、[[reference/components/hullhealth]]、[[reference/components/health]]。
