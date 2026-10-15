---
title: "deflectable"
created: "2026-10-26"
updated: "2026-10-28"
type: "component"
summary: "目标侧偏转端点：挂载即打 deflectable 标签，Deflect(deflector) 先同步回调再默认自删，供反制方（charlie_boss）弹回没有弹道组件的伪投射物攻击。"
aliases: ["deflectable", "可偏转目标", "偏转协议目标侧", "伪投射物反制"]
tags: ["component", "callback"]
dst_build: "steam-buildid:26230496"
sources: ["components/deflectable.lua", "components/projectile.lua", "components/complexprojectile.lua", "components/combat.lua", "components/damagetyperesist.lua", "componentutil.lua", "prefabs/charlie_boss.lua", "prefabs/willow_flame_fx.lua", "prefabs/slingshotammo.lua", "prefabs/slingshotammo_debuffs.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# deflectable

## 快速判断

- 职责与适用：给"实体自身就是攻击载体、但没有弹道组件"的目标（跟随 FX、伪投射物）提供偏转端点；反制方检测 `components.deflectable` 后调 `Deflect` 弹掉它。真投掷物的偏转走 [[reference/components/projectile]]／[[reference/components/complexprojectile]] 的同构 API。[charlie_boss.lua:226](../../../raw/current/scripts/prefabs/charlie_boss.lua#L226)、[deflectable.lua:31](../../../raw/current/scripts/components/deflectable.lua#L31
- 优先使用：组件构造自动加 `deflectable` 标签；按源码注释在 prefab pristine 段显式预加标签，主端 `AddComponent` 后按需 `SetOwner`＋`SetOnDeflectFn`＋`SetKeepOnDeflect`。[deflectable.lua:7-9](../../../raw/current/scripts/components/deflectable.lua#L7-L9)
- 重要副作用：`Deflect` 默认把宿主实体整个 `Remove()`；回调在移除判定前同步执行，可在回调内做标记转移。[deflectable.lua:31-38](../../../raw/current/scripts/components/deflectable.lua#L31-L38)
- 修改边界：服务端组件（原版宿主均主端挂载）；`deflectable` 标签还兼任 charlie_boss 反射态的伤害类型键——反射期攻击者或武器带该标签时伤害乘 0。[charlie_boss.lua:341](../../../raw/current/scripts/prefabs/charlie_boss.lua#L341)、[damagetyperesist.lua:33-42](../../../raw/current/scripts/components/damagetyperesist.lua#L33-L42)

## 关键接口

| 接口／字段 | 签名或已核验约定 | 条件、作用与证据 |
|---|---|---|
| 构造 | 无参；自动 `AddTag("deflectable")` | V2C 注释建议在 prefab pristine 段显式预加标签（网络端可见）再主端挂组件。[deflectable.lua:7-9](../../../raw/current/scripts/components/deflectable.lua#L7-L9) |
| `SetOnDeflectFn(fn)` | `fn(inst, owner, deflector)` | 偏转回调，Deflect 内同步执行；传 nil 可清（对象池复用）。[deflectable.lua:15-17](../../../raw/current/scripts/components/deflectable.lua#L15-L17)、[:33](../../../raw/current/scripts/components/deflectable.lua#L33) |
| `SetKeepOnDeflect(keep)` | truthy 保留 | 默认 falsy：Deflect 后 `inst:Remove()`；keep 才留实体交回调善后。[deflectable.lua:19-21](../../../raw/current/scripts/components/deflectable.lua#L19-L21)、[:35-37](../../../raw/current/scripts/components/deflectable.lua#L35-L37) |
| `SetOwner(owner)`／`GetOwner()` | 攻击发起者实体或 nil | owner 不参与组件自身逻辑，只供反制方取反弹目标／方向。[deflectable.lua:23-29](../../../raw/current/scripts/components/deflectable.lua#L23-L29)、[charlie_boss.lua:227](../../../raw/current/scripts/prefabs/charlie_boss.lua#L227 |
| `Deflect(deflector)` | deflector＝反制实体，无返回值 | 回调→按 keep 决定是否 Remove。[deflectable.lua:31-38](../../../raw/current/scripts/components/deflectable.lua#L31-L38) |
| `OnRemoveFromEntity()` | 组件移除钩子 | 仅摘标签，不清 owner／回调（池化宿主须自行清，见典型接入）。[deflectable.lua:11-13](../../../raw/current/scripts/components/deflectable.lua#L11-L13) |

三侧偏转协议同构（回调前两参＝载体、归属者，第三参＝deflector；默认移除、keep 保留）：projectile 用 `SetOnDeflectFn`／`SetKeepOnDeflect`／`Deflect`，归属字段 `owner`（[projectile.lua:132](../../../raw/current/scripts/components/projectile.lua#L132)、[:532](../../../raw/current/scripts/components/projectile.lua#L532)）；complexprojectile 用 `SetOnDeflect`，归属字段 `attacker`（[complexprojectile.lua:73](../../../raw/current/scripts/components/complexprojectile.lua#L73)、[:213](../../../raw/current/scripts/components/complexprojectile.lua#L213)）；deflectable 覆盖其余目标。

## 典型接入

### 原版宿主的三种用法

- **willow_shadow_flame（一次性自走火苗）**：pristine 预加标签（落实注释建议）[willow_flame_fx.lua:203-204](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L203-L204)，主端挂组件 [:233](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L233)；`settarget` 时 `SetOwner(source)` 定反弹归属 [:56](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L56)；不设回调、不 keep——被弹即火苗链终止，攻击后用 `inst:IsValid()` 区分正常爆炸与被弹掉 [:125-140](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L125-L140)。它带 weapon 组件＋`pseudorangedweapon` 标签，反制方按远程武器分支处理 [componentutil.lua:103-109](../../../raw/current/scripts/componentutil.lua#L103-L109)。
- **slingshotammo_horrorfuel_debuff_fx（池化复用 DoT 火苗）**：仅主端挂组件、pristine 未预加标签 [slingshotammo_debuffs.lua:471-497](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L471-L497)；每次复用前 `SetOwner(attacker)`＋`SetOnDeflectFn` 置 `data.deflected = true` 停掉后续 tick [slingshotammo.lua:487-488](../../../raw/current/scripts/prefabs/slingshotammo.lua#L487-L488)，回池时两者清 nil [:458-463](../../../raw/current/scripts/prefabs/slingshotammo.lua#L458-L463)。其攻击 `GetAttacked(inst, 0, nil, ...)` 不传武器 [slingshotammo_debuffs.lua:387](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L387)，反制方走"非远程→按 attacker 泛化偏转"分支；攻击后同样 `IsValid()` 判定 [:392](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L392)。
- **charlie_boss（反制方）**：`blocked` 事件（受击全格挡时 combat 推送 [combat.lua:730](../../../raw/current/scripts/components/combat.lua#L730）三路分派——远程武器监听 `weapononattack` 后：`data.projectile` 有效走 projectile:Deflect，否则对武器泛化偏转 [charlie_boss.lua:259-286](../../../raw/current/scripts/prefabs/charlie_boss.lua#L259-L286)；非远程对 attacker 泛化偏转 [:290](../../../raw/current/scripts/prefabs/charlie_boss.lua#L290)；complexprojectile 由逐帧 FindEntities 扫描命中后 Deflect，`_reflect_ignores` 防同弹重复 [:309-324](../../../raw/current/scripts/prefabs/charlie_boss.lua#L309-L324)。泛化偏转先 `GetOwner` 定反弹目标再 `Deflect`，沿攻击来向生成 charlie_boss_projectile 反射 [:225-251](../../../raw/current/scripts/prefabs/charlie_boss.lua#L225-L251)。

### 给自定义伪投射 FX 挂偏转端点

- 前提与选择：模组 prefab 文件；pristine 段 `inst:AddTag("deflectable")`，主端挂组件后按下述配置；owner 设为攻击发起者。
- 生效与撤销：被 charlie_boss 一类反制方命中即回调＋自删；反射归属与方向由 owner 决定。

```lua
local function SetupDeflectable(inst, caster)
    inst:AddComponent("deflectable")
    inst.components.deflectable:SetOwner(caster)
    inst.components.deflectable:SetOnDeflectFn(function(inst, owner, deflector)
        -- 回调后组件默认 Remove()；需要保留实体时改用 SetKeepOnDeflect(true)
    end)
end
```

- 调整与验证：回调参数顺序 `(inst, owner, deflector)` 与 projectile 一致，同一处理函数可跨侧复用 [deflectable.lua:33](../../../raw/current/scripts/components/deflectable.lua#L33)、[projectile.lua:533](../../../raw/current/scripts/components/projectile.lua#L533)。

## 常见陷阱与限制

- **Deflect 默认删实体，不走宿主回收路径**：池化宿主被弹时实体直接 Remove；slingshotammo 靠回调先置停更标记再接受移除 [slingshotammo.lua:488](../../../raw/current/scripts/prefabs/slingshotammo.lua#L488)。
- **带标签≠必被偏转**：反制方查的是 `components.deflectable` 组件而非标签搜索 [charlie_boss.lua:226](../../../raw/current/scripts/prefabs/charlie_boss.lua#L226)；但标签会被 damagetyperesist.GetResist 当伤害类型键命中，反射期带标签的攻击者/武器伤害乘 0 [charlie_boss.lua:341](../../../raw/current/scripts/prefabs/charlie_boss.lua#L341)、[damagetyperesist.lua:33-42](../../../raw/current/scripts/components/damagetyperesist.lua#L33-L42)。
- **不设 owner 也能偏转，反弹方向退化为反制者自身朝向**：charlie_boss 在实体同位或无有效 owner 时用自身 rotation 兜底定向 [charlie_boss.lua:234-243](../../../raw/current/scripts/prefabs/charlie_boss.lua#L234-L243)。

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖 deflectable.lua 全部 40 行、projectile／complexprojectile 偏转 API 对照、charlie_boss 三条偏转分派路径与 damagetyperesist 键语义、三个宿主的挂载与回调用法。未覆盖：游戏运行下的反射表现与网络同步。
- 游戏运行：未运行。
- 结构检查：2026-10-26 执行 validate_page.py（standard 档）与 validate_source_links.py（--strict）通过；页内 Lua 示例经 parser_probe.exe --syntax-only 语法检查通过。

相关：[[reference/components/projectile]]；[[reference/components/complexprojectile]]。
