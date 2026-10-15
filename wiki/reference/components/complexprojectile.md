---
title: "complexprojectile"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "抛物线落点投射物的初速、重力与落地回调；伤害和碰撞由 Prefab 另行实现。"
aliases: ["complexprojectile", "抛物线投射物", "抛掷落点", "投掷重力", "高抛弹道"]
tags: [component, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/complexprojectile.lua", "components/weapon.lua", "components/combat.lua", "prefabs/eyeofterror_mini_projectile.lua", "stategraphs/SGeyeofterror.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# complexprojectile

## 快速判断

- 职责与适用：对目标坐标计算初速，逐帧应用重力和马达速度；默认下降到近地面时 Hit，无目标追踪／自动攻击。实体追踪与武器结算选 [[reference/components/projectile]]。[complexprojectile.lua:106](../../../raw/current/scripts/components/complexprojectile.lua#L106)、[:198](../../../raw/current/scripts/components/complexprojectile.lua#L198)
- 优先使用：设置速度、负重力和 SetOnHit，再 `Launch(targetPos, attacker, owningweapon?)`；独立发射链可由 [[reference/components/weapon#关键接口|SetProjectile]] 接入。[complexprojectile.lua:44](../../../raw/current/scripts/components/complexprojectile.lua#L44)、[:134](../../../raw/current/scripts/components/complexprojectile.lua#L134)、[weapon.lua:146](../../../raw/current/scripts/components/weapon.lua#L146
- 重要副作用：Launch 改传入 Vector3 的 y；Hit 先 Cancel 再回调，不自动删除、生成效果或造成伤害。落点应取本次独立值。[complexprojectile.lua:157](../../../raw/current/scripts/components/complexprojectile.lua#L157)、[:191](../../../raw/current/scripts/components/complexprojectile.lua#L191)
- 修改边界：服务端组件且需 Physics；新工厂在 SetPristine 前声明 projectile／complexprojectile 标签，不与 projectile 同挂。原版工厂可复用物理、网络和落地替换。[complexprojectile.lua:26](../../../raw/current/scripts/components/complexprojectile.lua#L26)、[eyeofterror_mini_projectile.lua:43](../../../raw/current/scripts/prefabs/eyeofterror_mini_projectile.lua#L43

## 关键接口

| 接口／字段 | 参数／默认 | 条件、作用与证据 |
|---|---|---|
| `SetHorizontalSpeed(speed)` | number；默认 4 | 名称虽叫 horizontalSpeed，实际作为轨迹总初速，水平分量为 cos(angle)×speed，不保证水平恒速。[complexprojectile.lua:44](../../../raw/current/scripts/components/complexprojectile.lua#L44)、[:106](../../../raw/current/scripts/components/complexprojectile.lua#L106) |
| `SetGravity(g)` | 默认 -9.81 | 每帧加 g×dt 到 velocity.y；普通落体用负值，不是 Physics:SetGravity。[complexprojectile.lua:53](../../../raw/current/scripts/components/complexprojectile.lua#L53)、[:198](../../../raw/current/scripts/components/complexprojectile.lua#L198) |
| `usehigharc` | 默认 true | 可解时 true 选高角、false 选低角；不可解退用 30 度，Launch 不返回失败。[complexprojectile.lua:106](../../../raw/current/scripts/components/complexprojectile.lua#L106 |
| `SetLaunchOffset(offset)`／`SetTargetOffset(offset)` | Vector3 | 发射偏移仅 x／y，且 attacker 非 nil 才应用；目标偏移只用 y，覆盖输入 targetPos.y。[complexprojectile.lua:57](../../../raw/current/scripts/components/complexprojectile.lua#L57)、[:141](../../../raw/current/scripts/components/complexprojectile.lua#L141) |
| `SetHorizontalSpeedForDistance(distance, fallback)` | 计算初速后赋值 | 考虑两端高度与前偏移；fallback 仅计算返回 nil 时用，不表示实际轨迹已核验。[complexprojectile.lua:48](../../../raw/current/scripts/components/complexprojectile.lua#L48)、[:85](../../../raw/current/scripts/components/complexprojectile.lua#L85) |
| `Launch(targetPos, attacker, owningweapon?)` | 无显式成功返回 | 保存 attacker／owningweapon（缺省自身）、朝落点并计算轨迹，叠加攻击者所在移动平台速度，之后 onlaunch／activeprojectile／更新。[complexprojectile.lua:134](../../../raw/current/scripts/components/complexprojectile.lua#L134 |
| `SetOnLaunch(fn)`／`SetOnHit(fn)` | `(inst, attacker, targetPos)`／`(inst, attacker, target)` | 默认落地 Hit 不传 target，第三参 nil；owningweapon 读字段，不是回调参数。[complexprojectile.lua:174](../../../raw/current/scripts/components/complexprojectile.lua#L174)、[:191](../../../raw/current/scripts/components/complexprojectile.lua#L191) |
| `SetOnUpdate(fn)` | `(inst)`，无 dt | 返回 truthy 跳过默认移动／重力／落地检测，不是默认更新后的附加回调。[complexprojectile.lua:198](../../../raw/current/scripts/components/complexprojectile.lua#L198 |
| `Hit(target?)`／`Cancel()` | 无显式返回 | Hit 停更新／速度后回调；Cancel 清 velocity，但不删实体、不清 attacker／owningweapon。[complexprojectile.lua:183](../../../raw/current/scripts/components/complexprojectile.lua#L183 |

（26148036 核验：新增偏转 API `SetOnDeflect(fn)`／`SetKeepOnDeflect(keep)`／`Deflect(deflector)`——偏转时回调 `ondeflectfn(inst, attacker, deflector)`，默认移除弹体、设 keep 才保留；纯新增，既有落地路径不变——[complexprojectile.lua:73](../../../raw/current/scripts/components/complexprojectile.lua#L73)、[:213](../../../raw/current/scripts/components/complexprojectile.lua#L213)；非弹道目标的偏转端点见 [[reference/components/deflectable]]）

目标偏移参与求解，但默认落地仍要求下降且 y≤0.05，不是抵达 targetoffset.y 就 Hit；提前碰撞由工厂另行实现。[complexprojectile.lua:157](../../../raw/current/scripts/components/complexprojectile.lua#L157)、[:207](../../../raw/current/scripts/components/complexprojectile.lua#L207)

## 典型接入

### 恐怖之眼小眼球抛射改用低弧线

放在目标模组 `modmain.lua`；复用 eyeofterror_mini_projectile 的物理／资源与落地替换，不重写 onhit。原版初速 15、重力 -35、发射高度 4.9，SG 创建实体并设置位置后 Launch；本例只切换求解分支。[eyeofterror_mini_projectile.lua:42](../../../raw/current/scripts/prefabs/eyeofterror_mini_projectile.lua#L42)、[:88](../../../raw/current/scripts/prefabs/eyeofterror_mini_projectile.lua#L88)、[SGeyeofterror.lua:612](../../../raw/current/scripts/stategraphs/SGeyeofterror.lua#L612

```lua
AddPrefabPostInit("eyeofterror_mini_projectile", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local projectile = inst.components.complexprojectile
    if projectile ~= nil then
        projectile.usehigharc = false
    end
end)
```

只在存在两条实数轨迹时改变角度选择，不保证所有落点可达。保留原版落地创建 eyeofterror_mini_grounded、传 commander、删除空中实体；停用模组后新发射恢复高弧线，原版空中实体不存档，无本例自建任务／资源。[eyeofterror_mini_projectile.lua:18](../../../raw/current/scripts/prefabs/eyeofterror_mini_projectile.lua#L18)、[:80](../../../raw/current/scripts/prefabs/eyeofterror_mini_projectile.lua#L80)、[:198](../../../raw/current/scripts/components/complexprojectile.lua#L198)

### 原版模式与伤害扩展

小眼球展示“飞行对象→落地替换”，不要求 weapon／combat。远程武器也可生成 complexprojectile 后 Launch，但组件自身不调用 combat；伤害、范围筛选、FX 和删除须在工厂回调落实，仅加 SetProjectile 不构成完整远程攻击 demo。[weapon.lua:146](../../../raw/current/scripts/components/weapon.lua#L146)、[complexprojectile.lua:191](../../../raw/current/scripts/components/complexprojectile.lua#L191

## 常见陷阱与限制

- **horizontalSpeed 等于实际水平速度**：求解用其作总初速，高低弧线改变水平分量；不直接用距离÷字段值算飞行秒数。[complexprojectile.lua:128](../../../raw/current/scripts/components/complexprojectile.lua#L128
- **onmissfn 字段证明支持落空回调**：虽初始化字段，本文件未调用它；不可套 projectile 的 SetOnMissFn。不可达也只退用角度，不发失败事件。[complexprojectile.lua:16](../../../raw/current/scripts/components/complexprojectile.lua#L16)、[:125](../../../raw/current/scripts/components/complexprojectile.lua#L125)
- **移组件等于 Cancel**：OnRemoveFromEntity 仅移标签，不主动停止 Physics；留实体改用途时，移组件前明确 Cancel 和后续物理策略。[complexprojectile.lua:33](../../../raw/current/scripts/components/complexprojectile.lua#L33)、[:183](../../../raw/current/scripts/components/complexprojectile.lua#L183)
- **任意高度／零距离都能套公式**：求解含距离和 gravity 除法；本页未交付垂直同点／零重力保障，也不把计算初速当完整场景验证。[complexprojectile.lua:106](../../../raw/current/scripts/components/complexprojectile.lua#L106

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 24 处、结论修订 1 处（偏转 API 与锚点重定位，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整 complexprojectile、weapon／combat 发射入口、小眼球工厂／落地与 SG 发射点。未审校全部投掷动作、碰撞回调和平台内部。
- 结构检查：发布时核对元数据、来源、大小、链接／锚点与唯一登记；完整 modmain 示例仅作 Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；高低弧线视觉、不同高度／距离、平台速度继承、离屏和多人同步待实测，不承诺低弧线必达落点。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/projectile]]；[[reference/components/weapon]]；[[reference/components/explosive]]。
