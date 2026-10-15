---
title: "荆棘尖刺反伤与陷阱穿刺（bramblefx_armor 等 3 注册）"
created: "2026-11-05"
updated: "2026-11-07"
type: prefab
summary: "bramblefx.lua 共享工厂的 3 个混合 FX：穿在身上的荆棘外壳受击反伤尖刺环、亮茄系荆棘茄甲的升级版尖刺环（+5 平面伤害），以及 Wormwood 荆棘陷阱爆炸的原地穿刺环；三者都经 updatelooper 逐帧扩环并直接对 _combat 实体结算 GetAttacked——伤害归属分'穿戴者'与'FX 实体'两路，技能门禁与荆棘外壳抗性另有免疫链。"
aliases: ["bramblefx_armor", "bramblefx_armor_upgrade", "bramblefx_trap", "尖刺", "荆棘尖刺", "荆棘反伤", "荆棘陷阱穿刺", "沃姆伍德荆棘", "bramble thorns"]
tags: [prefab, fx, equipment, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/bramblefx.lua", "prefabs/armor_bramble.lua", "prefabs/armor_lunarplant.lua", "prefabs/trap_bramble.lua", "prefabs/wormwood.lua", "prefabs/merm_fx.lua", "components/playerspawner.lua", "components/combat.lua", "prefabs/skilltree_wormwood.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 荆棘尖刺反伤与陷阱穿刺（bramblefx_armor 等 3 注册）

## 快速判断

- 职责与适用：`bramblefx.lua` 共享工厂 `MakeFX(name, anim, damage, planardamage)` 的 **3 个"直接结算伤害"的混合 FX**（[工厂](../../../raw/current/scripts/prefabs/bramblefx.lua#L71-L121)、[注册](../../../raw/current/scripts/prefabs/bramblefx.lua#L123-L125)）：`bramblefx_armor`（荆棘外壳受击反伤尖刺环）、`bramblefx_armor_upgrade`（荆棘茄甲升级尖刺环，追加 5 点平面伤害与泛光）、`bramblefx_trap`（荆棘陷阱穿刺环）。实体不是纯视觉：主模拟用 `updatelooper` 逐帧扩环并对带 `_combat` 的实体调 `combat:GetAttacked`（[OnUpdateThorns](../../../raw/current/scripts/prefabs/bramblefx.lua#L12-L62)）。
- 伤害归属：armor 两档的攻击者记为**穿戴者**（`inst.owner`）， victims 若是穿戴者的 follower 则记 FX 实体（[L28-L37](../../../raw/current/scripts/prefabs/bramblefx.lua#L28-L37)）；trap 无 owner，攻击者记 **FX 实体自身**（[L38-L54](../../../raw/current/scripts/prefabs/bramblefx.lua#L38-L54)）——"FX 直接结算伤害"指实体作为 attacker 进战斗结算，非挂事件。
- 重要副作用：三个注册都挂 `thorny` 标签（不触发危险音乐），`bramblefx_trap` 另挂 `trapdamage`（出生保护判危险）；穿戴 `bramble_resistant` 装备（荆棘外壳／荆棘茄甲）对尖刺**完全免疫**（`EquipHasTag` 检查，[L21-L22](../../../raw/current/scripts/prefabs/bramblefx.lua#L21-L22)）。
- 修改边界：只覆盖 3 注册的工厂、生成点、扩环结算、门禁与清理；两件护甲的耐久／吸收率、陷阱的 mine 组件玩法、沃姆伍德技能树全貌不展开。
- 名称：家族显示名来自 `SetPrefabNameOverride("bramblefx")` → `STRINGS.NAMES.BRAMBLEFX`＝"尖刺"（[po](../../../raw/current/scripts/languages/chinese_s.po#L339959-L339962)）；宿主官方名 `ARMOR_BRAMBLE`＝"荆棘外壳"、`ARMOR_LUNARPLANT_HUSK`＝"荆棘茄甲"、`TRAP_BRAMBLE`＝"荆棘陷阱"（锚点见[验证记录](#验证记录)）；`BRAMBLEFX_ARMOR` 等三个注册名自身无 NAMES 键。

## 注册与构造

| 注册名 | 动画 | 伤害（tuning） | 差异与证据 |
|---|---|---|---|
| `bramblefx_armor` | `idle` | `TUNING.ARMORBRAMBLE_DMG`＝wilson_attack/1.5≈22.7 | 荆棘外壳反伤：无平面伤害、无泛光；`SetFXOwner(穿戴者)`。[注册](../../../raw/current/scripts/prefabs/bramblefx.lua#L123)、[调值](../../../raw/current/scripts/tuning.lua#L4562) |
| `bramblefx_armor_upgrade` | `idle` | `ARMORBRAMBLE_DMG`＋`spdamage { planar = ARMORBRAMBLE_DMG_PLANAR_UPGRADE = 5 }` | 荆棘茄甲升级反伤：构造期加泛光 `shaders/anim.ksh`（[L79-L81](../../../raw/current/scripts/prefabs/bramblefx.lua#L79-L81)），伤害表走 `GetAttacked` 第 5 参 `spdamage`（[combat.lua](../../../raw/current/scripts/components/combat.lua#L570)）。[注册](../../../raw/current/scripts/prefabs/bramblefx.lua#L124)、[调值](../../../raw/current/scripts/tuning.lua#L4566) |
| `bramblefx_trap` | `trap` | `TUNING.TRAP_BRAMBLE_DAMAGE`＝40 | 陷阱穿刺：额外 `trapdamage` 标签（[L85-L87](../../../raw/current/scripts/prefabs/bramblefx.lua#L85-L87)）、`canhitplayers` 保持工厂默认 true（由唯一生成点按 PVP 覆写）。[注册](../../../raw/current/scripts/prefabs/bramblefx.lua#L125)、[调值](../../../raw/current/scripts/tuning.lua#L4568) |

工厂共性：Transform／AnimState／Network＋`FourFaced`＋bank=build `bramblefx`＋`SetPrefabNameOverride("bramblefx")`＋标签 `FX`／`thorny`；`SetPristine` 后客户端早退；主模拟装 `updatelooper` 并注册 `OnUpdateThorns`、监听 `animover` → `Remove`、`persists=false`、`range = 0.75`、`ignore = {}`、`canhitplayers = true`、暴露 `SetFXOwner`（[主模拟段](../../../raw/current/scripts/prefabs/bramblefx.lua#L103-L115)）。**无 SoundEmitter**——陷阱触发声由 `trap_bramble` 播、护甲反伤声由穿戴者播（见生成点表）。

## 关联机制导航

### 扩环结算与免疫

`OnUpdateThorns` 每帧执行：`range += 0.75` → 以 `range + 3` 搜索带 `_combat` 的实体 → 有效目标（非 `ignore`、可见、有 combat、装备栏无 `bramble_resistant`）与中心距离小于 `range + 目标物理半径` 即结算一次并加入 `ignore`（每人最多一次）；有 owner 时还要过 owner 的 `combat:CanTarget(v)` 且非 `IsAlly(v)`，才会以穿戴者（follower 例外时以 FX）为攻击者结算。`range >= MAXRANGE(3)` 时注销更新函数（[OnUpdateThorns](../../../raw/current/scripts/prefabs/bramblefx.lua#L12-L62)）。可命中的标签集合按 `canhitplayers` 二选一：真→`NO_TAGS`（只排除 `playerghost`，玩家可被打），假→`NO_TAGS_NO_PLAYERS`（另排除 `player` 与 `companion`）（[标签表](../../../raw/current/scripts/prefabs/bramblefx.lua#L7-L10)）。`SetFXOwner(owner)` 把环心挪到 owner 位置、`canhitplayers = 非玩家 or PVP`、`ignore[owner] = true`（[L64-L69](../../../raw/current/scripts/prefabs/bramblefx.lua#L64-L69)）。

### 生成点与宿主调用

| 注册 | 生成语句（file:line） | 触发与门禁 |
|---|---|---|
| `bramblefx_armor` | [armor_bramble.lua#L23](../../../raw/current/scripts/prefabs/armor_bramble.lua#L23) `SpawnPrefab("bramblefx_armor"):SetFXOwner(owner)` | 荆棘外壳（宿主 `armor_bramble`）两条 proc 路径：①被击中/格挡（`attacked`/`blocked` 事件＋0.3 秒冷却＋`ShouldProcOnAttackedOrBlocked`）恒触发（[DoThorns](../../../raw/current/scripts/prefabs/armor_bramble.lua#L20-L28)）；②主动攻击累计 3 次未被击中触发——监听 `onattackother` 仅对 `plantkin`（沃姆伍德）安装，且需技能 `wormwood_armor_bramble` 已激活、命中数达 `WORMWOOD_ARMOR_BRAMBLE_RELEASE_SPIKES_HITCOUNT`(3)（[OnAttackOther](../../../raw/current/scripts/prefabs/armor_bramble.lua#L31-L45)）。反伤声 `armor/cactus` 由穿戴者播 |
| `bramblefx_armor_upgrade` | [armor_lunarplant.lua#L263](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L263) 同型 `SetFXOwner(owner)` | **仅荆棘茄甲（`armor_lunarplant_husk`）会生成**：husk 的 `OnHuskBlocked`（受击恒触发）与技能门禁的 `OnAttackOther` 都接到同一 `DoThorns`（[husk 接线](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L295-L301)）；本体 `armor_lunarplant`（官方名"亮茄盔甲"）的 `_onblocked` 是藤蔓 debuff、`_onattackother` 为空，**不生成**该 FX（[本体接线](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L69-L80)、[master_postinit](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L239-L243)） |
| `bramblefx_trap` | [trap_bramble.lua#L25](../../../raw/current/scripts/prefabs/trap_bramble.lua#L25) → 见下 | 荆棘陷阱（`mine` 组件）触发：`OnExplode` 后延迟 11 帧在陷阱位置 `SpawnPrefab("bramblefx_trap")` 并覆写 `canhitplayers = TheNet:GetPVPEnabled()`（[DoThorns](../../../raw/current/scripts/prefabs/trap_bramble.lua#L21-L27)、[OnExplode](../../../raw/current/scripts/prefabs/trap_bramble.lua#L31-L39)）；`SetFXOwner` 不调用——环心即陷阱位置、无 ignore 白名单 |

三处生成点都是**字面量 `SpawnPrefab`**，全 `raw/current/scripts` 反查无其它生成（含动态拼名）。另：`merm_fx.lua` 的月影鱼人尖刺 FX 复用 `SetPrefabNameOverride("bramblefx")` 只为死亡公告文案（[merm_fx.lua#L96](../../../raw/current/scripts/prefabs/merm_fx.lua#L96)），不是本页注册。

### 伤害归属与技能门禁

- **armor 两档：攻击者是穿戴者。** `GetAttacked(leader==owner and inst or owner, ...)` 的常规分支把伤害记到 `owner` 名下（仇恨、受击事件归穿戴者）；例外分支——受害者是穿戴者的 follower 且 leader 恰为 owner——攻击者改记 FX 实体（[L34](../../../raw/current/scripts/prefabs/bramblefx.lua#L34)）。第 5 参 `spdamage` 让升级版尖刺附 5 点平面伤害（无视常规护甲减伤的位面伤害通道）。
- **trap：攻击者是 FX 实体自身。** owner 为 nil，走 `CanBeAttacked` 分支 `GetAttacked(inst, ...)`（[L51](../../../raw/current/scripts/prefabs/bramblefx.lua#L51)）；非 PVP 下玩家 follower 免打（leader 为玩家且未主动攻击玩家，[L41-L49](../../../raw/current/scripts/prefabs/bramblefx.lua#L41-L49)）。工厂默认 `canhitplayers=true`，实际被唯一生成点覆写为 PVP 判定。
- **技能门禁**（[[reference/components/skilltreeupdater|skilltreeupdater 组件]]）：`wormwood_armor_bramble`（官方描述"荆棘外壳专家"，[po](../../../raw/current/scripts/languages/chinese_s.po#L368754-L368757)）只解锁"3 连击主动触发"，受击反伤不需要技能；`wormwood_blooming_trapbramble`（"荆棘陷阱专家"）在盛开时自动重置附近荆棘陷阱（[wormwood.lua#L328](../../../raw/current/scripts/prefabs/wormwood.lua#L328)、[po](../../../raw/current/scripts/languages/chinese_s.po#L368829-L368837)）。配方门禁：荆棘外壳／荆棘陷阱 `builder_tag="plantkin"`（[recipes.lua#L339-L340](../../../raw/current/scripts/recipes.lua#L339-L340)）；荆棘茄甲＝亮茄盔甲＋荆棘外壳合成、`builder_skill="wormwood_allegiance_lunar_plant_gear_1"`（[recipes.lua#L352](../../../raw/current/scripts/recipes.lua#L352)）。

### 清理路径

| 注册 | 壳 | 附属 | 证据 |
|---|---|---|---|
| 3 注册通用 | `animover` → `Remove`；`persists=false` 不入存档 | 无本地子实体 | [主模拟段](../../../raw/current/scripts/prefabs/bramblefx.lua#L106-L107) |
| 结算窗口 | `range` 从 0.75 起每帧 +0.75，达 3.0 那一帧结算后注销更新函数——约 3 帧扩环窗口；动画播完才删实体 | `ignore` 表保证每目标最多结算一次 | [L59-L61](../../../raw/current/scripts/prefabs/bramblefx.lua#L59-L61) |

## 典型接入

### 给装备做"受击反伤尖刺环"

- 前提与选择：宿主护甲监听 `attacked`/`blocked`，冷却去抖后生成 FX 并 `SetFXOwner(持有者)`；环心、免疫与 canhitplayers 都在 `SetFXOwner` 一步完成。想做"攻击 N 次触发"再加技能检查。
- 生效与撤销：FX 自删；结算对象由 `GetAttacked` 走正规战斗管线（护甲吸收、仇恨、受击事件齐全），不要再手工扣血。

```lua
local function DoThorns(inst, owner)
    inst._cdtask = inst:DoTaskInTime(0.3, function() inst._cdtask = nil end)
    SpawnPrefab("bramblefx_armor"):SetFXOwner(owner)
    if owner.SoundEmitter ~= nil then
        owner.SoundEmitter:PlaySound("dontstarve/common/together/armor/cactus")
    end
end
```

- 调整与验证：替换点是注册名（选 upgrade 需宿主自己负责平面伤害档位的差异，FX 已按注册内建）；`SetFXOwner` 是 bramblefx 三注册壳上的实例方法，别的 FX 没有。原版依据 [armor_bramble DoThorns](../../../raw/current/scripts/prefabs/armor_bramble.lua#L20-L28)。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：尖刺环是纯视觉。** 三个注册都经 `updatelooper` 直接调 `combat:GetAttacked`——trap 的 40 点伤害就是 FX 实体作为 attacker 结算的；armor 反伤约 22.7（升级版另加 5 平面）记在穿戴者名下。[结算段](../../../raw/current/scripts/prefabs/bramblefx.lua#L12-L62)
- **边界：armor 与 trap 的伤害归属不同。** armor→穿戴者（follower 例外出 FX）；trap→FX 实体（无归属玩家）。需要"谁打的"语义时按生成路径区分。[归属段](#伤害归属与技能门禁)
- **边界：`bramblefx_armor_upgrade` 的唯一宿主是荆棘茄甲。** 亮茄盔甲本体不装 thorns proc（它的受击路径是藤蔓 debuff）；穿亮茄盔甲打人不会出升级尖刺。[接线辨析](#生成点与宿主调用)
- **边界：`bramble_resistant` 装备免疫所有尖刺。** 判的是"装备栏里任一装备带该标签"（`EquipHasTag`），荆棘外壳与荆棘茄甲自身都带；穿戴者不会被自己的反伤打中（另在 `ignore` 表）。[免疫检查](../../../raw/current/scripts/prefabs/bramblefx.lua#L21-L22)
- **边界：`canhitplayers` 语义按生成路径不同。** armor：玩家穿戴时非 PVP 不打其他玩家；非玩家 owner 不受该 PVP 开关限制，另受 owner 的 `CanTarget`／非 `IsAlly` 战斗门禁。trap：工厂默认 true，但唯一生成点立即覆写为 PVP——两路实际都受 PVP 影响，但不能把 armor 的规则推广到所有 owner。[标签表](../../../raw/current/scripts/prefabs/bramblefx.lua#L7-L10)、[覆写](../../../raw/current/scripts/prefabs/trap_bramble.lua#L25-L27)
- **边界：`trapdamage` 标签参与出生保护的危险计数。** `SPAWN_PROTECTION_DANGER_TAGS` 含 `trapdamage`；但各分支条件不同：`spawnprotection=always` 在非 loading 时直接返回 true，`never` 直接 false；两个覆盖分支只在非 loading 时生效，loading 或 overrides 缺失时整段跳过并返回 false。只有默认分支才要求世界天数 >1，并在“16 格内危险实体数 >1”或“四档结构／阻挡物门槛”任一成立时启用（[playerspawner](../../../raw/current/scripts/components/playerspawner.lua#L106-L127)）。单枚荆棘陷阱只是危险计数之一，不能写成“放陷阱就阻止／必然启用重生保护”。`thorns` 事件在工厂里已注释掉（不触发 wisecracker 台词，[L35-L36](../../../raw/current/scripts/prefabs/bramblefx.lua#L35-L36)）。
- **边界：结算窗口只有约 3 帧。** 扩环到 3.0 即注销更新函数，之后动画继续播但不再结算；`ignore` 每目标一次。想复用为持续区域伤害需要自己另装循环。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/bramblefx.lua`、`prefabs/armor_bramble.lua`、`prefabs/trap_bramble.lua`；按需读取 `armor_lunarplant.lua` 的公共工厂／master_postinit／husk 段与 onequip，`components/combat.lua` GetAttacked 签名，`prefabs/skilltree_wormwood.lua` 两节点，`wormwood.lua` 重置荆棘陷阱行，`recipes.lua` 三条配方，`tuning.lua` 调值（wilson_attack=34×多人修正、ARMORBRAMBLE_DMG、PLANAR_UPGRADE、TRAP_BRAMBLE 三项、HITCOUNT）。全 `raw/current/scripts` 反查：`bramblefx_armor` 命中 armor_bramble.lua 依赖（L8）＋生成（L23）＋注册（bramblefx.lua:123）；`bramblefx_armor_upgrade` 命中 armor_lunarplant.lua 生成（L263）＋注册（L124）；`bramblefx_trap` 命中 trap_bramble.lua 依赖（L11）＋生成（L25）＋注册（L125）——三注册生成点唯一、无动态拼名；`SetPrefabNameOverride("bramblefx")` 另命中 merm_fx.lua:96（死亡公告借用，已辨析）；`trapdamage` 消费方 playerspawner.lua:106。未运行游戏。
- 名称核验：`languages/chinese_s.po` 定向检索——`BRAMBLEFX`＝"尖刺"（[L339959-L339962](../../../raw/current/scripts/languages/chinese_s.po#L339959-L339962)）、`ARMOR_BRAMBLE`＝"荆棘外壳"（[L338829-L338832](../../../raw/current/scripts/languages/chinese_s.po#L338829-L338832)）、`ARMOR_LUNARPLANT`＝"亮茄盔甲"（[L338839-L338842](../../../raw/current/scripts/languages/chinese_s.po#L338839-L338842)）、`ARMOR_LUNARPLANT_HUSK`＝"荆棘茄甲"（[L338844-L338847](../../../raw/current/scripts/languages/chinese_s.po#L338844-L338847)）、`TRAP_BRAMBLE`＝"荆棘陷阱"（[L350534-L350537](../../../raw/current/scripts/languages/chinese_s.po#L350534-L350537)）；`BRAMBLEFX_` 前缀下无其它 NAMES 键（三注册显示名统一走 override），正文使用已核验效果描述。
- 分档说明：`size_class: standard`——3 注册共享一工厂一清理模式，扩环结算一张表；复杂度集中在伤害归属与门禁辨析，两小节承载，无多状态机。
- 未覆盖：两件护甲的耐久／吸收率／修复链、陷阱 mine 组件的重置与作废流程、沃姆伍德技能树全貌（对照锁与 allegiance 分支）、`spdamage` 平面伤害在 combat 的完整减伤计算、动画包帧长与观感；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

- 第 15 轮补正（2026-11-07）：出生保护 always／never／默认分支拆开，owner 分支补 `CanTarget`／非 `IsAlly`，PVP 规则限定玩家 owner；同步维护日期。

## 相关链接

- [[reference/components/skilltreeupdater|skilltreeupdater 组件]]（`wormwood_armor_bramble` 技能门禁的判定载体）
- [[reference/prefabs/armor_lunarplant_glow_fx|装备与帽饰跟随光效]]（荆棘茄甲／亮茄护甲的 glow 附属 FX）
- [[reference/prefabs/wormwood_plant_fx|沃姆伍德开花植物特效]]（同角色 bloom 期的另一表现族）
- 返回 [[reference/prefabs/fx-status-index|增益、受击、护盾与状态反馈]]
