---
title: "oceanthrowable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "把物品标记为可掷入海洋，OCEAN_TOSS 动作命中时为其补挂 complexprojectile 并回调宿主配置弹道。"
aliases: ["oceanthrowable", "海洋投掷物", "投掷入海", "OCEAN_TOSS"]
tags: [component, item]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/oceanthrowable.lua", "prefabs/chum.lua", "actions.lua", "componentactions.lua", "prefabs/bootleg.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# oceanthrowable

## 快速判断

- 职责与适用：`AddProjectile()` 确保实体有 complexprojectile 组件后回调宿主配置；原版唯一挂载是 `chum`（鱼饵球），动作侧 OCEAN_TOSS 负责脱手与发射。[实现](../../../raw/current/scripts/components/oceanthrowable.lua#L11-L19) [挂载](../../../raw/current/scripts/prefabs/chum.lua#L122-L123)
- 优先使用：动作链是 `DropItem` 脱手 → `AddProjectile()` 补弹道组件 → `complexprojectile:Launch(pos, doer)`；宿主在 `SetOnAddProjectileFn` 里设速度、重力与命中回调。[动作](../../../raw/current/scripts/actions.lua#L5199-L5215) [宿主配置](../../../raw/current/scripts/prefabs/chum.lua#L76-L82)
- 重要副作用：AddProjectile 幂等——已有 complexprojectile 就不再添加，但回调每次都会执行；指定 target 时弹道 targetoffset 固定 `{x=0,y=1.5,z=0}`。[实现](../../../raw/current/scripts/components/oceanthrowable.lua#L11-L19) [偏移](../../../raw/current/scripts/actions.lua#L5206-L5208)
- 修改边界：无存读；原版另有"组件未完成"的源码注释（bootleg 弃用它改走 TOSS）， POINT 入口还要求渔网投掷判定，复用前先确认动作登记条件。[注释](../../../raw/current/scripts/prefabs/bootleg.lua#L151)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetOnAddProjectileFn(fn)` | `fn(inst) -> nil` | 记录回调；chum 用它设 HorizontalSpeed 15、Gravity -35、LaunchOffset 与 onthrown/OnHit。回调在 AddProjectile 内同步执行。[注册](../../../raw/current/scripts/components/oceanthrowable.lua#L7-L9) [chum 回调](../../../raw/current/scripts/prefabs/chum.lua#L76-L82) |
| `AddProjectile()` | `-> nil` | 无 complexprojectile 时 `AddComponent("complexprojectile")`，再调回调；不发射，发射由动作侧 Launch 完成。[实现](../../../raw/current/scripts/components/oceanthrowable.lua#L11-L19) |

## 典型接入

### 复刻 chum 的投掷弹道配置（原版参照）

- 前提与选择：物品需 inventoryitem（动作先 DropItem 脱手）；动作登记有两条路——POINT 入口（右键海面）要求 `CanCastFishingNetAtPoint`，TARGET 入口（右键海中目标）要求 `IsOceanAtPoint`，且排除手柄控制。[TARGET 登记](../../../raw/current/scripts/componentactions.lua#L2441-L2448) [POINT 登记](../../../raw/current/scripts/componentactions.lua#L2178-L2184)
- 生效与撤销：OCEAN_TOSS 成功后物品成为飞行实体，落水/命中由 complexprojectile 的 OnHit 接手；chum 的 OnAddProjectile 同时挂 onthrown（起抛表现）。弹道组件细节见 [[reference/components/complexprojectile]]。[chum 配置](../../../raw/current/scripts/prefabs/chum.lua#L76-L82)
- 原版依据：bootleg 注释表明该组件当时未完成、bootleg 转投 TOSS 动作——复用时以 chum 链路为准。[注释](../../../raw/current/scripts/prefabs/bootleg.lua#L151)

```lua
-- 服务端：为可入海物品配置弹道（按 chum 模式）
inst:AddComponent("oceanthrowable")
inst.components.oceanthrowable:SetOnAddProjectileFn(function(inst)
    inst.components.complexprojectile:SetHorizontalSpeed(15)
    inst.components.complexprojectile:SetGravity(-35)
    inst.components.complexprojectile:SetLaunchOffset(Vector3(.25, 1, 0))
    inst.components.complexprojectile:SetOnLaunch(onthrown)
    inst.components.complexprojectile:SetOnHit(OnHit)
end)
```

- 调整与验证：速度单位与重力符号沿 complexprojectile 约定；onthrown/OnHit 需为 prefab 内已定义函数。

## 常见陷阱与限制

- **以为登记动作只需要组件：** POINT 入口额外要求渔网落点判定（源码即如此），TARGET 入口要求海面坐标且非手柄；不满足时 OCEAN_TOSS 不出现，与组件是否存在无关。[POINT 条件](../../../raw/current/scripts/componentactions.lua#L2178-L2184) [TARGET 条件](../../../raw/current/scripts/componentactions.lua#L2441-L2448)
- **回调里重复 AddComponent 或重复设弹道：** AddProjectile 对已有 complexprojectile 跳过添加但仍执行回调；多次触发会覆盖前次弹道配置，回调应写成幂等配置。[实现](../../../raw/current/scripts/components/oceanthrowable.lua#L11-L19)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（21 行），核对 chum 挂载与弹道回调、OCEAN_TOSS 全流程、两条 componentactions 登记与 bootleg 注释。未审阅 complexprojectile 飞行与落水判定（见其组件页）。
- 结构检查：2026-10-28 validate_page.py 通过（oceanthrowable.md）；validate_source_links.py --strict 本页全部源码锚点通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；投掷落海与命中未实测。
