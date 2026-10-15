---
title: "hullhealth"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "服务端船体损伤结算：把碰撞换算成漏水与 HP 损失，按秒结算漏水，并支持自降解。"
aliases: ["船体损伤与漏水", "撞船掉血", "船体破洞", "hullhealth"]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/hullhealth.lua", "components/boatphysics.lua", "components/health.lua", "components/walkableplatform.lua", "components/boatleak.lua", "components/boatring.lua", "prefabs/boat.lua", "prefabs/boat_leak.lua", "prefabs/yotd_boats.lua", "tuning.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hullhealth

## 快速判断

- 职责与适用：服务端船体损伤结算：把 boatphysics 的碰撞换算成 HP 损失与固定漏点，按秒扣漏水，并支持自降解。对象是有 `health`、`boatphysics`、`walkableplatform` 的船 prefab；不负责碰撞检测、帆桨与沉没。[构造](../../../raw/current/scripts/components/hullhealth.lua#L17-L43)
- 优先使用：碰撞上限用 `max_health_damage`（HP），漏水灵敏度用 `small_leak_dmg`／`med_leak_dmg`（0..1 累计量），持续掉血用 `SetSelfDegrading`（秒/点），禁漏水用 `leakproof`；炮弹等外来源走宿主 prefab 的 `spawnnewboatleak` 事件。[tuning](../../../raw/current/scripts/tuning.lua#L4140-L4167)、[宿主监听](../../../raw/current/scripts/prefabs/boat.lua#L188-L213)
- 重要副作用：每 1 秒一次 `health:DoDelta`；维护 `is_leaking` tag；在 6 个固定漏点生成或升级 `boat_leak`；新漏点给甲板上的玩家推 `on_standing_on_new_leak`。[UpdateHealth](../../../raw/current/scripts/components/hullhealth.lua#L68-L95)、[OnCollide](../../../raw/current/scripts/components/hullhealth.lua#L187-L201)
- 修改边界：只在服务端（所有宿主都在 `ismastersim` 之后添加）；不设最大生命、不触发沉没；运行期移除会留下常驻任务与监听（见陷阱）。

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `leak_point_count` | 整数，默认 6 | 固定漏点数量；决定角度与 `leak_damage` 表长，但 `OnCollide` 选点循环硬编码 1..6，改小会越界。[构造](../../../raw/current/scripts/components/hullhealth.lua#L20-L38)、[选点](../../../raw/current/scripts/components/hullhealth.lua#L143-L156) |
| `leak_radius`／`leak_radius_variance`／`leak_angle_variance` | 世界单位；弧度 | 漏点相对船心的半径（默认 2.5±1）与角度抖动（PI/8），决定 `boat_leak` 落点。[GetLeakPosition](../../../raw/current/scripts/components/hullhealth.lua#L97-L104) |
| `leak_damage`／`leak_indicators`／`leak_indicators_dynamic` | `[1..6]=0..1`；实体表 | 每个固定漏点的累计破损，上限 1；达到 `small_leak_dmg`／`med_leak_dmg`（默认 0.1／0.75，远古船 0.2／0.85）才生成或升级漏点，也是修复后重开的依据。实体分固定／动态两表，只统计有效且状态为 small/med 的。[生成](../../../raw/current/scripts/components/hullhealth.lua#L110-L134)、[tuning](../../../raw/current/scripts/tuning.lua#L4166-L4183)、[UpdateHealth](../../../raw/current/scripts/components/hullhealth.lua#L71-L90) |
| `max_health_damage` | HP，默认 70（远古船 40） | 单次碰撞／单个 bumper 的伤害基数，乘速度比后取 `floor`；不是总血量。[tuning](../../../raw/current/scripts/tuning.lua#L4143)、[结算](../../../raw/current/scripts/components/hullhealth.lua#L203-L210) |
| `leakproof` | 布尔 | 为真时 `RefreshLeakIndicator` 直接返回 false；`boat_leak` 自己也会在落到 leakproof 船时移除，动态漏点同样留不住。[hullhealth](../../../raw/current/scripts/components/hullhealth.lua#L110-L117)、[boat_leak](../../../raw/current/scripts/prefabs/boat_leak.lua#L47-L54) |
| `SetSelfDegrading(stat)` | 秒／1 HP；0 停止 | 启动 `OnUpdate`，每累计 `stat × GetDamageMult("degradedamage")` 秒扣 1 HP；停止时只移除 tag，不清进度。[SetSelfDegrading](../../../raw/current/scripts/components/hullhealth.lua#L214-L222)、[OnUpdate](../../../raw/current/scripts/components/hullhealth.lua#L237-L254) |
| `GetDamageMult(cat)` | 倍率，默认 1 | 在船心 4 格内找带 `boat_health_buffer` 且站在本船上的实体，命中返回 `TUNING.BOAT_HEALTH_BUFFER_MULT[cat]`（collide 0.5／degradedamage 2）。[实现](../../../raw/current/scripts/components/hullhealth.lua#L48-L66)、[tuning](../../../raw/current/scripts/tuning.lua#L6759-L6763) |

### 碰撞到伤害的顺序

1. 命中点角度在 6 个固定漏点中选最近的一个，只看角度、不看距离。[选点](../../../raw/current/scripts/components/hullhealth.lua#L137-L156)
2. `|hit_dot_velocity| / (speed_damage_factor or 1)` 大于 0.258（约 75°）才算有效，否则按擦过忽略。[L158-L164](../../../raw/current/scripts/components/hullhealth.lua#L158-L164)
3. 靠近速度 `hit_adjacent_speed = boatphysics:GetVelocity() × |hit_dot_velocity|`（单位/秒）。[L165](../../../raw/current/scripts/components/hullhealth.lua#L165)
4. bumper：速度大于 0.8 且船有 `boatring` 时查命中点覆盖的 bumper；命中就推 `boatcollision`、按同一基数扣其 HP 后返回，不再漏水／扣船体 HP；撞世界且 bumper 带 `collision_world_safe`（蟹王）时连它也不掉血。[L167-L185](../../../raw/current/scripts/components/hullhealth.lua#L167-L185)
5. 漏水：速度大于 2 时累加 `min(速度 - 2, 1 - 已累计)`，达到阈值生成／升级漏点，给甲板玩家推事件。[L187-L201](../../../raw/current/scripts/components/hullhealth.lua#L187-L201)
6. 直伤：速度大于 0.8 时按 `70 × min(速度/10, 1) × GetDamageMult("collide")` 扣 HP，`floor` 后入账。[L203-L210](../../../raw/current/scripts/components/hullhealth.lua#L203-L210)

漏水与直伤是两条独立结算，同一撞可同时触发；bumper 分支会短路两者。

### 漏水、修理与自降解

- 每秒合计：small 漏点 0.5、med 1.0；动态漏点若没有 `boatleak` 组件按 med 计；合计为 0 时清 `is_leaking`。[UpdateHealth](../../../raw/current/scripts/components/hullhealth.lua#L68-L95)
- 常规补丁把漏点实体改成 `repaired...` 状态，`UpdateHealth` 对其计 0；水草补丁用 `SetRepairedTime` 定时，到期自动回到 med_leak。[boatleak 修复](../../../raw/current/scripts/components/boatleak.lua#L96-L133)、[水草补丁](../../../raw/current/scripts/components/boatleak.lua#L251-L260)、[到期](../../../raw/current/scripts/components/boatleak.lua#L48-L60)
- 修理不清 `leak_damage`：同一会话里再撞同一点，会按原累计值立即重开，`RefreshLeakIndicator` 复用已存在的补丁实体。[L187-L194](../../../raw/current/scripts/components/hullhealth.lua#L187-L194)
- 自降解每次固定 1 HP，与最大生命无关；`degradefx` 只影响碎屑特效概率（每帧 1/30 ÷ 倍率）。[OnUpdate](../../../raw/current/scripts/components/hullhealth.lua#L237-L254)

### 与 boatphysics／health／walkableplatform 的分工

| 组件 | 负责 | hullhealth 的边界 |
|---|---|---|
| boatphysics | 检测碰撞与反弹，派发 `on_collide`；不扣船体 HP。 | 只消费事件；缺 boatphysics 时 `GetVelocity` 报错。见 [[reference/components/boatphysics]]。 |
| health | HP、死亡与 `DoDelta`；沉没由 SGboat 处理。 | 只调 `DoDelta`，不设上限、不带攻击者，无击杀归属。 |
| walkableplatform | 平台半径、甲板实体与玩家集合。 | 只用玩家集合推事件；动态漏点由 prefab 挂上平台。[Prefab](../../../raw/current/scripts/prefabs/boat.lua#L188-L213) |
| boatring | 船缘 bumper 列表与命中查询。 | 仅 bumper 分支调用 `GetBumperAtPoint`。见 [[reference/components/boatring]]。 |
| hull | 船唇、跳板挂接与半径。 | 与 HP／漏水无关。 |

### 存读

- 固定漏点写进自己的存档：`leak_point`、`leak_damage`、`leak_state`、`repaired_timeout`；无有效漏点时返回 nil。动态漏点 `isdynamic` 时 `persists = true`，由实体自身存档，读档后挂回动态表。[OnSave](../../../raw/current/scripts/components/hullhealth.lua#L256-L270)、[动态漏点存读](../../../raw/current/scripts/components/boatleak.lua#L285-L321)
- `EntityScript:LoadPostPass` 会遍历组件调用 `LoadPostPass`，读档时自动执行，无需 prefab 转调。[entityscript](../../../raw/current/scripts/entityscript.lua#L1964-L1975)
- `selfdegradingtime` 不在 hullhealth 存档里：水獭巢船自己在 prefab 的 OnSave 记 `erosion_rate`，木板船／草船／冰船只靠构造时设定。[水獭巢船](../../../raw/current/scripts/prefabs/boat.lua#L1121-L1138)

## 典型接入

### 原版宿主与变体（不开新组件）

- 木船 `boat`：`create_master_pst` 依次加 `boatring`、`hullhealth`、`boatphysics`，后加 `health`；1 秒任务构造时挂上，首次结算时 health 已存在。[boat.lua](../../../raw/current/scripts/prefabs/boat.lua#L576-L584)
- 同工厂还有 `boat_pirate`、`boat_ancient`、`boat_otterden`、`boat_grass`、`boat_ice`、`boat_ice_crabking`；龙舟走 `dragonboat_server`，对应 `dragonboat_body`／`dragonboat_shadowboat`。[注册](../../../raw/current/scripts/prefabs/boat.lua#L1639-L1665)、[龙舟](../../../raw/current/scripts/prefabs/yotd_boats.lua#L915-L920)
- 变体：草船自降解 5 秒/点 + `leakproof`，`spawnnewboatleak` 改为瞬间 15／30 HP；水獭巢船自降解 + `leakproof` + 回血；冰船自降解 1 秒/点 + `leakproof`；远古船改 40 伤害与 0.2／0.85 阈值；龙舟漏点半径按 0.6×DRAGON_BOAT_RADIUS 收窄。[草船](../../../raw/current/scripts/prefabs/boat.lua#L842-L845)、[水獭巢船](../../../raw/current/scripts/prefabs/boat.lua#L1218-L1221)、[冰船](../../../raw/current/scripts/prefabs/boat.lua#L1355-L1356)、[龙舟](../../../raw/current/scripts/prefabs/yotd_boats.lua#L455-L457)
- 这些是原版行为，不存在“注册组件”的模组接入点；要改参数用下一节。

### 模组调参（服务端 postinit）

- 目标文件：模组 `modmain.lua`；`AddPrefabPostInit("boat", fn)` 在 prefab 工厂之后调用，能拿到已构造好的组件；客户端也会调用，先判 `ismastersim`。
- 端与资源：仅服务端；无需新资源；客户端血量由 `healthsyncer` 复制。
- 生效与撤销：只影响之后生成的 `boat`（其它变体分别注册）；删 postinit 并重载即撤销。
- 单位：`max_health_damage` 是 HP；两个阈值是 0..1 累计量，不是 HP。

```lua
-- modmain.lua：只改新生成木船的船体损伤参数
AddPrefabPostInit("boat", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local hullhealth = inst.components.hullhealth
    if hullhealth == nil then
        return
    end
    hullhealth.max_health_damage = 35  -- 单次碰撞 HP 上限，默认 70
    hullhealth.small_leak_dmg = 0.2    -- 0..1，达到才生成小漏
    hullhealth.med_leak_dmg = 0.7      -- 0..1，达到才升中漏
end)
```

- 验证：示例只做 Lua 5.1 语法检查，未在游戏内执行；字段读取点见上表。

## 常见陷阱与限制

- `leak_point_count` 不能改：选点循环固定 1..6，改成 4 会在 `leak_damage[5]` 做 nil 比较，改成 8 则 7、8 号点不会被碰撞命中。[构造](../../../raw/current/scripts/components/hullhealth.lua#L36-L38)、[循环](../../../raw/current/scripts/components/hullhealth.lua#L145)
- `leakproof` 与动态漏点互斥：冰船既 `leakproof` 又监听 `spawnnewboatleak`，漏点实体下一帧会被 `checkforleakimmune` 移除并打印警告，只有自降解生效。[boat_leak](../../../raw/current/scripts/prefabs/boat_leak.lua#L47-L54)、[冰船](../../../raw/current/scripts/prefabs/boat.lua#L1352-L1356)
- 移除组件不会清场：`RemoveComponent("hullhealth")` 后 1 秒任务与 `on_collide` 监听仍在（回调有 nil 检查，不报错）；再次添加会得到两份，结算翻倍。无 `OnRemoveFromEntity`。[构造](../../../raw/current/scripts/components/hullhealth.lua#L41-L42)
- `damage_alignment` 的除数 `speed_damage_factor` 在本 build 没有任何写入点，实际恒为 1；只有外部碰撞源自己塞这个字段才会变。[L158-L160](../../../raw/current/scripts/components/hullhealth.lua#L158-L160)
- 没有攻击者：所有扣血都是 `DoDelta(-x)`，死亡不记击杀，也不触发依赖 afflicter 的减伤；要加护盾挂 `health.deltamodifierfn` 或监听 `healthdelta`。[DoDelta](../../../raw/current/scripts/components/health.lua#L613-L649)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 5 处（tuning BOAT 参数两段、单发上限、缓冲倍率，entityscript LoadPostPass 段）；无结论修订，仅锚点迁移——依据：全局枢纽 diff 审阅＋变更清单命中 tuning.lua、entityscript.lua，本版 tuning 改动为插入新常量，BOAT 血量／漏点阈值／缓冲倍率数值未变；hullhealth.lua 及协作组件／prefab 未变。
- 源码审阅：2026-10-21，steam-buildid:25643519。覆盖构造、OnCollide 六步、UpdateHealth、RefreshLeakIndicator、SetSelfDegrading／OnUpdate、GetDamageMult、OnSave／LoadPostPass；核对全部宿主注册名及 boat_leak／boatphysics／health／walkableplatform 的边界；未覆盖表现层。
- 游戏运行：未运行；本页结论均来自源码静态核验，不含手感与数值实测。
- 结构检查：`validate_page.py` 通过（0 issue；119 行，低于 200 行／16 KiB 上限）；`validate_source_links.py --strict` 通过（50 处引用、0 error／0 warning）；正文 Lua 示例另存 492 字节，`parser_probe --syntax-only` 通过（只查 Lua 5.1 语法，未执行）。

相关：[[reference/components/boatphysics]]、[[reference/components/boatring]]、[[reference/components/health]]。
