---
title: "firefx"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "火焰特效组件：按等级表驱动火焰动画、灯光半径／强度／颜色与燃火音效，支持受控燃烧变体与熄灭动画返回值，供火焰 FX 实体与燃烧系统。"
aliases: ["火焰特效", "火焰灯光等级", "firefx"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/firefx.lua", "prefabs/character_fire.lua", "components/burnable.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# firefx

## 快速判断

- 职责与适用：只做"火的表现"：动画、灯光、音效随 `levels` 等级表切换（components/firefx.lua:88-135）。宿主通常是独立的火焰 FX 实体（character_fire、campfirefire、fire 等十余个 prefab）。
- 优先使用：配合 `burnable` 使用——燃烧组件按火势调 `SetLevel`／`SetPercentInLevel`，并写入 `radius_levels` 供火势蔓延查询（components/burnable.lua:179-181、163-165）。
- 重要副作用：构造即生成 `firefx_light` 子实体并 `StartUpdatingComponent` 做灯光闪烁（lines 18-24、51-57）；点火／熄灭音效与 `daytime` 声音参数随等级与昼夜更新（lines 92-94、59-62）。
- 修改边界：`OnRemoveEntity`／`OnRemoveFromEntity` 移除灯光实体（lines 27-31）；实体休眠停音、唤醒续播（lines 161-172）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `levels` | 数组，每级 `{anim, pre, pst, pst_fast, anim_controlled_burn..., sound, soundintensity, radius, intensity, falloff, colour}` | 等级表驱动全部表现；实例见 character_fire.lua:18-70（3 级） |
| `radius_levels` | 数组（可选） | burnable 写入的蔓延半径表，优先于 `levels[.].radius`（lines 65-67；burnable.lua:163-165） |
| `SetLevel(lev, immediate, controlled_burn)` | 升降级 | 首次／升级播点火音（`bigignitesoundthresh=3` 分大小，line 10、92-94）；有 `pre` 时先播前摇再循环（lines 96-108）；同步灯光四参数（lines 111-115） |
| `SetPercentInLevel(percent)` | `(0..1)` | 级内插值半径与强度（lines 78-86），火势渐强渐弱用 |
| `Extinguish(fast)` | `-> bool` | 停音、播 `pst`／`pst_fast`／受控燃烧熄灭动画；返回 true 表示还有熄灭动画，宿主不应立即移除（lines 137-155；burnable.lua:563） |
| `AttachLightTo(target)` | 灯光换宿主 | 自动跟随目标 onremove 回挂本体（lines 20-22、33-49） |
| `SetFxLightOffsetPosition(off)` | `(Vector3)` | 灯光相对偏移（lines 157-159） |
| `light` | `firefx_light` 实体 | 灯光操作入口（line 18） |
| `usedayparamforsound` | bool | 开启后按昼夜设 `daytime` 声音参数（lines 14、59-62） |

## 典型接入

### 自定义火焰 FX 实体（服务端）

- 前提与选择：普通 FX 实体（Transform／AnimState／SoundEmitter／Network）；等级表的 `anim` 必须是宿主 build 里的循环动画，`pre`／`pst` 可省略。
- 生效与撤销：`SetLevel(1)` 点火；`Extinguish()` 返回 true 时等动画播完再 `Remove()`，false 可直接移除。交给 burnable 驱动时无需手动调（burnable.lua:535-538 自动 SetLevel＋AttachLightTo）。
- 原版依据：prefabs/character_fire.lua:121-123（挂载＋赋 levels）；角色加热走 heater 按等级取热（character_fire.lua:9-10、123-125）。

```lua
local firelevels =
{
    { anim = "level1", radius = 2, intensity = 0.6, falloff = 0.7, colour = { 1, 1, 0.7 }, sound = "dontstarve/common/campfire" },
    { anim = "level2", radius = 3, intensity = 0.75, falloff = 0.5, colour = { 1, 1, 0.8 }, sound = "dontstarve/common/campfire" },
}

local function fn()
    local inst = CreateEntity()
    inst.entity:AddTransform()
    inst.entity:AddAnimState()
    inst.entity:AddSoundEmitter()
    inst.entity:AddNetwork()
    inst.entity:SetPristine()
    if not TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("firefx")
    inst.components.firefx.levels = firelevels
    inst.components.firefx:SetLevel(1)
    return inst
end
```

- 调整与验证：`radius` 单位同 Light 半径；`colour` 为 0~1 数组；受控燃烧（`controlled_burn`）只在表内提供 `*_controlled_burn` 动画时生效（lines 89-90、104-107）。

## 常见陷阱与限制

- **levels 未设即调用 SetLevel：** `params = self.levels[self.level]` 直接索引（line 101），等级数超过表长会被 `math.min(lev, #self.levels)` 压回（line 100），但空表仍会崩。
- **灯光不随目标删除残留：** 依赖 `AttachLightTo` 的 onremove 回挂（lines 20-22）；直接改 `light` 宿主而不走该接口会失去保护。
- **音效循环依赖唤醒补播：** 实体休眠时强停 `fire` 音（lines 161-163），若自定义流程在休眠中改了 `playingsound` 字段，唤醒补播可能与实际动画不符（lines 165-172）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。通读 components/firefx.lua 全文（174 行）；核验 character_fire.lua:9-10、18-70、121-139，burnable.lua:163-165、179-181、535-538、563。未运行游戏。
- 结构检查：validate_page.py / validate_source_links.py 通过（见批级自验）。

相关：[[reference/components/burnable]]、[[reference/components/heater]]
