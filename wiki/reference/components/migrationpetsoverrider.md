---
title: "migrationpetsoverrider"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "为随玩家跨 Shard 迁移的宠物提供出生落点偏移回调，playerspawner 落地时按回调把宠物摆到传送点周围的可走位置。"
aliases: ["migrationpetsoverrider", "迁移宠物出生偏移", "宠物迁移落点", "migrationpets 覆写"]
tags: [component, creature]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/migrationpetsoverrider.lua", "components/playerspawner.lua", "prefabs/gestalt_guard_evolved.lua", "prefabs/knight.lua", "components/petleash.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# migrationpetsoverrider

## 快速判断

- 职责与适用：专用于 playerspawner 对 `inst.migrationpets` 的处理（源码首行注释言明）：玩家跨 Shard 迁移落地时，给每只迁移宠物求一个相对落点的偏移。[注释](../../../raw/current/scripts/components/migrationpetsoverrider.lua#L1-L2)
- 优先使用：服务端在宠物 prefab 里 `SetOffsetFromFn(fn)` 注册回调；playerspawner 对有组件的宠物调 `GetOffsetFrom(x,y,z)` 取偏移并 Teleport/SetPosition，无组件则原地放。[消费](../../../raw/current/scripts/components/playerspawner.lua#L244-L258)
- 重要副作用：回调默认实现返回 `(0,0,0)`；未注册回调或返回 0 时宠物与玩家重叠出生。原版两个宿主都用 FindWalkableOffset 找可走点，找不到才回 0。[默认](../../../raw/current/scripts/components/migrationpetsoverrider.lua#L13-L19) [宿主回调](../../../raw/current/scripts/prefabs/gestalt_guard_evolved.lua#L324-L333)
- 修改边界：纯服务端回调，无存读、无网络变量；构造中注释的 `onsetpositionfn` 没有对应成员方法，当前唯一能力是偏移回调。[构造](../../../raw/current/scripts/components/migrationpetsoverrider.lua#L2-L7)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetOffsetFromFn(fn)` | `fn(inst, x, y, z) -> ox, oy, oz` | 记录回调；x,y,z 是玩家迁移落点。gestalt_guard 用 `GESTALT_EVOLVED_CLOSE_RANGE+1 .. MID_RANGE-1` 半径，knight 的 YOTH 变体仅在 `not inst.friendly` 时偏移 6～12。[注册](../../../raw/current/scripts/components/migrationpetsoverrider.lua#L9-L11) [knight 回调](../../../raw/current/scripts/prefabs/knight.lua#L312-L322) |
| `GetOffsetFrom(x, y, z)` | `-> ox, oy, oz`（默认 0,0,0） | playerspawner 唯一消费点：`Physics:Teleport` 或 `Transform:SetPosition` 到落点加偏移。[消费](../../../raw/current/scripts/components/playerspawner.lua#L248-L256) |

## 典型接入

### 让随行宠物迁移后不压在传送点（原版参照）

- 前提与选择：宠物需是"迁移宠物"——petleash 的宠物、绑定幽灵（ghostlybond）与库存宠物在迁移时被填进 `player.migrationpets`；随后本组件才被查询。[名单来源](../../../raw/current/scripts/components/petleash.lua#L301-L303)
- 生效与撤销：组件在玩家迁移校验通过（`migration.worldid ~= TheShard:GetShardId()`）的落地分支消费一次，随后 `player.migration` 清空；回调无撤销接口，覆写前先考虑确定性。[落地分支](../../../raw/current/scripts/components/playerspawner.lua#L243-L247)
- 原版依据：gestalt_guard_evolved 与 knight（YOTH 变体）都在 follower 配置后挂载并注册回调，配套 follower.neverexpire 表明它们是长期随行宠物。[宿主挂载](../../../raw/current/scripts/prefabs/gestalt_guard_evolved.lua#L469-L470) [knight 挂载](../../../raw/current/scripts/prefabs/knight.lua#L593-L594)

```lua
-- 服务端：为迁移宠物注册落点偏移（按 knight YOTH_OffsetFromFn 模式）
inst:AddComponent("migrationpetsoverrider")
inst.components.migrationpetsoverrider:SetOffsetFromFn(function(inst, x, y, z)
    local offset = FindWalkableOffset(Vector3(x, y, z), math.random() * TWOPI, 8, 12, false, false, NoHoles)
    if offset then
        return offset.x, offset.y, offset.z
    end
    return 0, 0, 0
end)
```

- 调整与验证：FindWalkableOffset 参数（半径、尝试次数、可走判定）决定落点质量；返回值必须三元数值，组件不校验。

## 常见陷阱与限制

- **以为组件负责迁移名单或跨 Shard 传送：** 宠物如何随玩家过 Shard 由 petleash/ghostlybond/inventory 与 migration 系统处理；本组件只在目标 Shard 落地时算偏移，不移动实体本身。[名单](../../../raw/current/scripts/components/petleash.lua#L301-L303) [传送](../../../raw/current/scripts/components/playerspawner.lua#L255-L256)
- **回调里做昂贵搜索不设兜底：** 原版回调都限定半径与尝试次数并返回 0 兜底；无界搜索会拖慢落地分支，返回非数值会直接破坏 Teleport 参数。[宿主参照](../../../raw/current/scripts/prefabs/gestalt_guard_evolved.lua#L324-L333)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（21 行），核对 playerspawner 落地消费、两个原版宿主的回调实现与挂载、migrationpets 名单填充。未审阅 Shard 迁移握手与存档恢复全流程。
- 结构检查：2026-10-28 validate_page.py 通过（migrationpetsoverrider.md）；validate_source_links.py --strict 本页全部源码锚点通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；跨 Shard 落地偏移未实测。
