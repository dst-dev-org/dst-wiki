---
title: "boatcannon"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "船载炮的弹药与开火组件：装填炮弹 prefab、被玩家瞄准占用、按状态图时序发射 complexprojectile 并给船体后坐力。"
aliases: ["船炮", "船载加农炮", "boatcannon"]
tags: [component, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/boatcannon.lua", "components/boatcannonuser.lua", "prefabs/boat_cannon.lua", "stategraphs/SGboatcannon.lua", "actions.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatcannon

## 快速判断

- 职责与适用：挂在船炮建筑（`boat_cannon` prefab）上，管理"装了什么弹、谁在瞄、怎么发射"；发射逻辑与弹道由 `complexprojectile` 承担。[构造](../../../raw/current/scripts/components/boatcannon.lua#L9-L23)
- 优先使用：装填 `LoadAmmo(projectileprefab)`、查询 `IsAmmoLoaded()`、发射 `Shoot()`；瞄准占用由 `boatcannonuser:SetCannon` 间接触发 `StartAiming`／`StopAiming`，模组一般不直接调。[LoadAmmo](../../../raw/current/scripts/components/boatcannon.lua#L78-L81)
- 重要副作用：`loadedammo` 赋值联动 `ammoloaded` tag（客户端动作过滤可见）；装/卸弹推 `ammoloaded`／`ammounloaded` 事件；`StartAiming` 加 `occupied` tag。[tag 联动](../../../raw/current/scripts/components/boatcannon.lua#L1-L7)
- 修改边界：服务端组件（瞄准方 `boatcannonuser:SetCannon` 内有 mastersim 断言）；不存操作者，只存 `loadedammo`；弹药槽只有一个。[OnSave](../../../raw/current/scripts/components/boatcannon.lua#L25-L31)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `loadedammo` | 字符串（弹体 prefab 名）或 nil | 赋值监听切 `ammoloaded` tag；捡起船炮套件时 prefab 读它把弹药还原成物品。[L12](../../../raw/current/scripts/components/boatcannon.lua#L12)、[prefab 读取](../../../raw/current/scripts/prefabs/boat_cannon.lua#L29-L31) |
| `LoadAmmo(projectileprefab)` | 无返回 | 设值并推 `ammoloaded`／`ammounloaded` 事件；装填动作校验物品带 `boatcannon_ammo` tag 后传入其 `projectileprefab`。[L78-L81](../../../raw/current/scripts/components/boatcannon.lua#L78-L81)、[动作](../../../raw/current/scripts/actions.lua#L5043-L5055) |
| `IsAmmoLoaded()` | 布尔 | 状态图刷新与装填动作的前置判断。[L74-L76](../../../raw/current/scripts/components/boatcannon.lua#L74-L76)、[SG](../../../raw/current/scripts/stategraphs/SGboatcannon.lua#L21-L25) |
| `StartAiming(operator)`／`StopAiming()` | 无返回 | 记录操作者、加/摘 `occupied` tag、回调 `onstartfn`／`onstopfn`；操作者被移除时自动停止。[L41-L62](../../../raw/current/scripts/components/boatcannon.lua#L41-L62) |
| `Shoot()` | 无返回 | 无弹直接返回；按朝向 0.5 单位偏移、`PROJECTILE_INITIAL_HEIGHT=1.1` 生成弹体，以 `-rotation` 为水平角、`RANGE=20` 为落点距离 `complexprojectile:Launch`，随后清弹并给所在船 `boatphysics:ApplyForce(1)` 后坐。[L83-L120](../../../raw/current/scripts/components/boatcannon.lua#L83-L120)、[tuning](../../../raw/current/scripts/tuning.lua#L4284-L4288) |
| `onstartfn`／`onstopfn` | 回调字段 | `SetOnStartAimingFn`／`SetOnStopAimingFn` 注入，参数 `(inst, operator)`。[L33-L39](../../../raw/current/scripts/components/boatcannon.lua#L33-L39) |

## 典型接入

### 原版宿主：从装填到开火的完整链路

- 挂载：`boat_cannon` prefab master 段 `AddComponent("boatcannon")` 并设状态图 `SGboatcannon`。[挂载](../../../raw/current/scripts/prefabs/boat_cannon.lua#L337)、[状态图](../../../raw/current/scripts/prefabs/boat_cannon.lua#L343)
- 装填：`BOAT_CANNON_LOAD_AMMO` 动作从背包找/取带 `boatcannon_ammo` tag 的物品，`LoadAmmo(ammo.projectileprefab)` 并删除弹药物品；右键空手也可装填。[动作](../../../raw/current/scripts/actions.lua#L5043-L5055)
- 瞄准：`BOAT_CANNON_START_AIMING` 要求目标无操作者且未燃烧，经 `boatcannonuser:SetCannon` 回调本组件 `StartAiming` 并设基线朝向。[动作](../../../raw/current/scripts/actions.lua#L5074-L5082)、[user 侧](../../../raw/current/scripts/components/boatcannonuser.lua#L157-L170)
- 发射：`BOAT_CANNON_SHOOT` 让炮 `FacePoint` 后进 `shoot` 状态，第 7 帧调 `Shoot()`，第 10 帧按卸弹刷新动画。[SG 时序](../../../raw/current/scripts/stategraphs/SGboatcannon.lua#L70-L88)

### 模组：自定义炮弹与开火回调

- 前提与选择：新弹丸 prefab 需带 `boatcannon_ammo` tag、`projectileprefab` 字段与 `complexprojectile` 组件；发射后的行为全部由弹丸自己负责。
- 生效与撤销：`onstartfn`／`onstopfn` 建议包一层保留原回调；`Shoot` 内置后坐力无法参数化，改力度需 postinit 包 `Shoot`。

```lua
-- modmain.lua：船炮每次开火时广播事件（服务端）
AddPrefabPostInit("boat_cannon", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local cannon = inst.components.boatcannon
    if cannon == nil then
        return
    end
    local shoot = cannon.Shoot
    cannon.Shoot = function(self, ...)
        local ammo = self.loadedammo
        local ret = shoot(self, ...)
        if ammo ~= nil and self.loadedammo == nil then
            self.inst:PushEvent("my_cannon_fired", { ammo = ammo })
        end
        return ret
    end
end)
```

- 调整与验证：示例仅语法检查未运行；后坐力方向为射击反方向、力值固定 1，作用对象是 `GetCurrentPlatform()` 的船实体，岸上开火无后坐。[L110-L119](../../../raw/current/scripts/components/boatcannon.lua#L110-L119)

## 常见陷阱与限制

- `StopAiming` 里对操作者注册的是 `ListenForEvent("onremove", ...)` 而非移除监听（疑为笔误）：反复瞄准/停止会累积同名回调；因回调先判 `operator == self.operator` 且之后 operator 已清空，实际表现为无害空转，但依赖监听计数的行为会失真。[L52-L56](../../../raw/current/scripts/components/boatcannon.lua#L52-L56)
- `Shoot` 生成的弹体位置固定在炮口偏移 0.5 单位、高度 1.1，落点固定为朝向 × `RANGE=20`：修改射程只能覆盖 `TUNING.BOAT.BOATCANNON.RANGE` 或包装 `Shoot`。[L99-L108](../../../raw/current/scripts/components/boatcannon.lua#L99-L108)
- 弹药槽单发且只存 prefab 名：`SpawnPrefab` 失败时自动清弹返回，装填动作并不校验 prefab 有效性。[L88-L93](../../../raw/current/scripts/components/boatcannon.lua#L88-L93)
- 组件被移除时若有人在瞄，优先走操作者的 `boatcannonuser:SetCannon(nil)`，否则本地 `StopAiming`；模组强拆组件要先确认无人占用。[OnRemoveFromEntity](../../../raw/current/scripts/components/boatcannon.lua#L64-L72)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖 boatcannon.lua 全文（含 OnSave/OnLoad 与全部回调）；核对 boat_cannon 挂载/弹药还原、SGboatcannon 装填态刷新与 shoot 时序、三个动作、boatcannonuser 的 SetCannon 交互与 tuning 数值；未覆盖弹丸落点伤害结算与客户端准星 UI。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 与示例 Lua 语法检查通过（见批记录）。

相关：[[reference/components/complexprojectile]]、[[reference/components/boatphysics]]。

## 相关链接

- [[reference/components/boatcannonuser]]：玩家船炮瞄准端：服务端权威与客户端准星。
