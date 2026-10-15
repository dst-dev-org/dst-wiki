---
title: "spectatorcorpse"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "尸体观战相机：玩家以可复活尸体形态倒地时，本地焦点相机平滑吸附到最近的存活玩家，客户端/服务端共跑一个组件。"
aliases: ["尸体观战", "观战镜头", "spectatorcorpse"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/spectatorcorpse.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "prefabs/world.lua", "gamemodes.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# spectatorcorpse

## 快速判断

- 职责与适用：游戏模式开启 `spectator_corpse` 时给玩家挂上；玩家变成"可复活尸体"后把本地相机焦点源 `CorpseCam` 指向范围内最近的存活玩家。[挂载](../../../raw/current/scripts/prefabs/player_common.lua#L2475-L2477)
- 优先使用：无公开调用方法；状态由 `ms_becameghost{corpse=true}`／`ms_respawnedfromghost{corpse=true}` 事件驱动，读取视角表现应关注 `focalpoint` 的焦点源。[构造](../../../raw/current/scripts/components/spectatorcorpse.lua#L81-L84)
- 重要副作用：网络变量 `spectatorcorpse._isspectating`（脏事件 `isspectatingdirty`）跨端同步；组件 `OnUpdate` 只在本地玩家（`TheFocalPoint` 的父实体是自己）时执行相机逻辑。[网络变量](../../../raw/current/scripts/components/spectatorcorpse.lua#L79)
- 修改边界：构造双端都会跑，但 master 端只负责置网络变量；`TheFocalPoint` 仅客户端存在；gamemode 关闭该属性时组件根本不挂载。[gamemode](../../../raw/current/scripts/gamemodes.lua#L71)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `_isspectating` | net_bool | 服务端在成尸/复活时 set；两端经脏事件 `isspectatingdirty` 触发 `OnIsSpectatingDirty`。[声明](../../../raw/current/scripts/components/spectatorcorpse.lua#L79)、[置位](../../../raw/current/scripts/components/spectatorcorpse.lua#L21-L29) |
| `maxrange`／`startspeed`／`priority` | 40／0.5 每秒／1 | 焦点吸附最大范围、力量增速与焦点源优先级，可按需求覆写。[字段](../../../raw/current/scripts/components/spectatorcorpse.lua#L70-L76) |
| `OnUpdate`（组件循环） | 无返回 | 力量从 0 增长到 maxrange；在 maxrange 内挑"非自己、未死亡成鬼、可见"的最近玩家（上一次目标半径减 4 的粘滞优先），调 `focalpoint:StartFocusSource/StopFocusSource("CorpseCam")`。[选目标](../../../raw/current/scripts/components/spectatorcorpse.lua#L98-L114) |
| `OnIsSpectatingDirty`（内部） | 无返回 | true 时启动组件更新，false 时停止并撤销焦点源；`lasttarget`/`str` 一并复位。[L3-L19](../../../raw/current/scripts/components/spectatorcorpse.lua#L3-L19) |

## 生命周期

- 服务端：监听 `ms_becameghost`／`ms_respawnedfromghost`，`data.corpse` 为真才置 `_isspectating`（普通死亡变鬼不触发）。[事件推送](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L793)
- 两端：`playeractivated` 后激活；客户端额外监听 `isspectatingdirty`，`playerdeactivated` 时停止更新并解绑。[激活](../../../raw/current/scripts/components/spectatorcorpse.lua#L41-L50)
- `TheFocalPoint` 是客户端世界实体（world.lua 生成），焦点源机制归 `focalpoint` 组件；本组件只是它的一个来源。[生成](../../../raw/current/scripts/prefabs/world.lua#L548)

## 典型接入

### 模组：调整观战镜头手感

- 前提与选择：仅自定义游戏模式把 `spectator_corpse` 置 true 才生效；调整手感覆写实例字段即可，无需重定义方法。
- 生效与撤销：对已挂组件的玩家实体改 `maxrange`／`startspeed` 立即影响下一次 `OnUpdate`；恢复原值即撤销。

```lua
-- modmain.lua：尸体观战镜头看得更远、吸得更快（双端）
AddPrefabPostInitAny(function(inst)
    if inst.components and inst.components.spectatorcorpse ~= nil then
        inst.components.spectatorcorpse.maxrange = 60
        inst.components.spectatorcorpse.startspeed = 1.5
    end
end)
```

- 调整与验证：示例仅语法检查未运行；`AddPrefabPostInitAny` 的回调在 prefab 构造（含组件挂载）之后执行，可以直接改到字段。

## 常见陷阱与限制

- `OnUpdate` 有 `TheFocalPoint.entity:GetParent() == self.inst` 前置：专服与远端玩家不执行相机逻辑，别在服务端调用它做游戏逻辑。[L91](../../../raw/current/scripts/components/spectatorcorpse.lua#L91)
- 目标筛选排除"死亡或成鬼"玩家但未排除其他形态的尸体玩家；多具尸体同场时会互相成为观战对象，最后由距离决定。[L98-L110](../../../raw/current/scripts/components/spectatorcorpse.lua#L98-L110)
- 成尸判定依赖事件携带 `corpse = true`：直接把玩家设成 ghost 不会进入观战；复活路径同理（`ms_respawnedfromghost{corpse=true}`）。[置位条件](../../../raw/current/scripts/components/spectatorcorpse.lua#L21-L29)
- 组件自移除钩子只有 `playerdeactivated` 分支的收尾，无 `OnRemoveFromEntity`；运行期移除组件需自己停更新并撤销 `CorpseCam` 焦点源。[L52-L66](../../../raw/current/scripts/components/spectatorcorpse.lua#L52-L66)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文（含双端激活分支与 OnUpdate 选目标算法）；核对 player_common 挂载条件、player_common_extensions 的成尸/复活事件、world.lua 的 TheFocalPoint 生成与 gamemodes 属性；未覆盖 focalpoint 组件内部的多焦点源合成算法。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 与示例 Lua 语法检查通过（见批记录）。

相关：[[reference/components/revivablecorpse]]。
