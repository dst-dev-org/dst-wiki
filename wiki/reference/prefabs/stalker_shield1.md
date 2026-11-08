---
title: "复活的骨架骨盾格挡（stalker_shield1～4 与随机基名）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "远古织影者受击时展开的大骨盾：4 个定号变体＋1 个随机基名共用 MakeShield 工厂；有真实玩法作用——2 帧后对半径 3 内生物结算 10 点伤害并减速击退，玩家走 repelled 事件由各自 SG 击退。"
aliases: ["stalker_shield", "stalker_shield1", "stalker_shield2", "stalker_shield3", "stalker_shield4", "骨盾格挡", "骨架盾牌特效", "复活的骨架护盾"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26474873"
sources: ["prefabs/stalker_shield.lua", "stategraphs/SGstalker.lua", "prefabs/stalker.lua", "components/health.lua", "components/combat.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwx78_possessedbody.lua", "prefabs/armor_skeleton.lua", "prefablist.lua", "fx.lua", "entityscript.lua", "mainfunctions.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 复活的骨架骨盾格挡（stalker_shield1～4 与随机基名）

## 快速判断

- 职责与适用：`prefabs/stalker_shield.lua` 用一个 `MakeShield` 工厂注册 5 个联网 FX：定号变体 `stalker_shield1`～`stalker_shield4` 与随机基名 `stalker_shield`（工厂内随机 4 选 1）。这是复活的骨架（stalker_atrium，官方名"远古织影者"）受击时贴身展开的大骨盾演出，**不是纯表现**：生成 2 帧后对半径 3 内的非玩家生物真实结算 10 点伤害并施加减速击退。[工厂与注册](../../../raw/current/scripts/prefabs/stalker_shield.lua#L123-L130)
- 优先使用：给自定义 Boss 复刻"受击格挡＋范围击退"时，`SpawnPrefab` 任一定号变体并引擎父挂到宿主；伤害与击退由 FX 自带的 `StartRepel` 自动执行，调用方只管生成时机。[典型接入](#给自定义宿主复刻受击骨盾)
- 重要副作用：伤害归属 FX 实体自身（`GetAttacked(inst, 10)`，FX 无 combat 组件也可作为攻击者参数）；玩家与 `possessedbody` 不吃伤害，改推 `repelled` 事件由其 SG 执行 9 帧硬直＋同款速度公式的击退。
- 修改边界：只覆盖 5 个注册的工厂、SGstalker 的动态生成链（`PickShield` 选号）、`StartRepel` 击退子系统与宿主 `hasshield` 条件；远古织影者本体战（引导者、骨刺、小兵）不展开；`fx.lua` 里复用同动画库的 `shadow_shield1`～`6`（骨架骑士护甲）不属本页，见[清单外注记](#常见陷阱与限制)。
- 名称：`STALKER`＝"复活的骨架"（[po](../../../raw/current/scripts/languages/chinese_s.po#L349995-L349997)）、`STALKER_ATRIUM`＝"远古织影者"（[po](../../../raw/current/scripts/languages/chinese_s.po#L350000-L350002)）；5 个覆盖注册名在简中 `chinese_s.po` 无 `STRINGS.NAMES` 键，标题与正文的"骨盾格挡"等为已核验效果描述，非官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `stalker_shield1` | `MakeShield("stalker_shield1", 1)` | `idle1` 动画、正向缩放 |
| `stalker_shield2` | `MakeShield("stalker_shield2", 2)` | `idle2` 动画、正向缩放 |
| `stalker_shield3` | `MakeShield("stalker_shield3", 3)` | `idle3` 动画、正向缩放 |
| `stalker_shield4` | `MakeShield("stalker_shield4", 4)` | **`idle3` 动画（`min(3, n)`）＋初始 x 镜像**（scale x=-2.36） |
| `stalker_shield` | `MakeShield("stalker_shield", nil, prefs)`：`n = math.random(4)` 后按上述规则取外观；`SetPrefabName("stalker_shield"..n)` 改实体名；第四参 `prefs` 是 4 个变体名（加载依赖，不是生成） | 随机基名；**`math.random` 在 SetPristine 前各端各自执行**，见[陷阱](#常见陷阱与限制) |

共同工厂（[定义](../../../raw/current/scripts/prefabs/stalker_shield.lua#L80-L121)）：Transform／AnimState／SoundEmitter／Network → 标签 `FX`（**无 NOCLICK**）→ bank／build `stalker_shield` → 播 `idle<min(3,n)>`、FinalOffset 2、缩放 2.36（n=4 时 x=-2.36）→ SetPristine → 客户端早退 → 主模拟：播 `dontstarve/creatures/together/stalker/shield`、`persists=false`、`animover` 自删＋动画长度＋1 帧定时兜底、`2*FRAMES` 后 `StartRepel`。

## 关联机制导航

### 生成点与选号（PickShield）

| 注册 | 生成语句 | 消费文件与条件 |
|---|---|---|
| `stalker_shield1`～`4` | `SpawnPrefab("stalker_shield"..tostring(shieldtype))`：[SGstalker.lua#L171](../../../raw/current/scripts/stategraphs/SGstalker.lua#L166-L187) | **唯一生成点**：宿主 SG 的 `attacked` 事件，且 `inst.hasshield` 为真；shieldtype 来自 `PickShield`（值域 1–4）；生成后引擎父挂 `fx.entity:SetParent(inst.entity)`；shieldtype < 3 且随机 < .5 时主模拟侧 `SetScale(-2.36, ...)` 补镜像 |
| `stalker_shield` | **无**（诚实缺口） | 全 `raw/current/scripts` 反查 `stalker_shield`：除本文件的资源串（assets [L3](../../../raw/current/scripts/prefabs/stalker_shield.lua#L1-L4)、bank／build L93-L94）、变体循环基名 [L126-L128](../../../raw/current/scripts/prefabs/stalker_shield.lua#L123-L130) 与 [SGstalker.lua#L171](../../../raw/current/scripts/stategraphs/SGstalker.lua#L166-L187) 的动态拼名前缀（`shieldtype` 恒为 1–4，产不出基名）外，基名字面命中：注册 [L130](../../../raw/current/scripts/prefabs/stalker_shield.lua#L123-L130)、检索注释 [L133](../../../raw/current/scripts/prefabs/stalker_shield.lua#L133)、[stalker.lua#L61](../../../raw/current/scripts/prefabs/stalker.lua#L55-L72)（`prefabs_atrium` 依赖声明）、[prefablist.lua#L1232](../../../raw/current/scripts/prefablist.lua#L1232)（文件清单）、[fx.lua#L4071-L4072](../../../raw/current/scripts/fx.lua#L4067-L4079)（同名 bank／build 字符串，属另一注册族）。无任何 `SpawnPrefab("stalker_shield")` 或指向基名的拼名 |

`PickShield`（[实现](../../../raw/current/scripts/stategraphs/SGstalker.lua#L57-L80)）——注释写明"variation 3 or 4 is the main shield"：

| 分支 | 条件 | 结果 |
|---|---|---|
| 总门 | 距上次任何盾 < 0.2 秒 | 返回 nil（不生成） |
| 主盾保底 | 距上次主盾 ≥ `MAIN_SHIELD_CD`＝1.2 秒 | `math.random(3, 4)` |
| 主盾插值 | 随机 < (距上次主盾)/1.2 | `math.random(3, 4)` |
| 小盾填充 | 否则 | 随机 < (dt/2.4 + .5) 则 2，否则 1 |

### 生成条件链（hasshield 与免伤）

`hasshield` 只在 stalker_atrium 置位（[atrium postinit](../../../raw/current/scripts/prefabs/stalker.lua#L1628-L1665)：commander 组件、`health.redirect`、`soldierschanged` 监听）：`OnSoldiersChanged` 以 `commander:GetNumSoldiers() > 0` 翻转 `hasshield` 并同步 `net_bool`（[实现](../../../raw/current/scripts/prefabs/stalker.lua#L730-L739)）——士兵即 `SpawnChannelers` 在半径 8.7 生成的 `shadowchanneler` 引导者（[生成](../../../raw/current/scripts/prefabs/stalker.lua#L674-L680)）。有盾的意义在免伤：`health.redirect = nodmgshielded`（[挂接](../../../raw/current/scripts/prefabs/stalker.lua#L726-L728)），`Health:DoDelta` 把 `ignore_absorb` 一并传给 redirect，redirect 返回真即返回 0（[health.lua#L623-L624](../../../raw/current/scripts/components/health.lua#L613-L624)）；`nodmgshielded` 返回 `hasshield and amount <= 0 and not ignore_absorb`——普通未标记忽略吸收的扣血路径全拦，**有盾时本体免伤，玩家打不动，转而触发 attacked 事件出盾 FX**；带 `ignore_absorb` 的伤害仍可绕过该 redirect。`attacked` 事件在伤害被 redirect 吞掉时依然推送（[combat.lua#L716](../../../raw/current/scripts/components/combat.lua#L716)，`blocked` 只看攻击入口条件，redirect 结果不影响）。stalker 家族 4 个注册（[stalker](../../../raw/current/scripts/prefabs/stalker.lua#L1796)、[stalker_forest](../../../raw/current/scripts/prefabs/stalker.lua#L1797)、[stalker_atrium](../../../raw/current/scripts/prefabs/stalker.lua#L1798)、[stalker_npc](../../../raw/current/scripts/prefabs/stalker.lua#L1799)）共用 [SGstalker](../../../raw/current/scripts/prefabs/stalker.lua#L1502)，但 commander 只加在 atrium postinit——原版只有远古织影者会出这套盾。

### StartRepel 击退子系统

`StartRepel`（[实现](../../../raw/current/scripts/prefabs/stalker_shield.lua#L57-L78)）在生成 2 帧后执行一次：`FindEntities` 半径 3（`REPEL_RADIUS`），必须带 `locomotor`、排除 `fossil`／`shadow`／`playerghost`／`INLIMBO`，且存活可见。分两路：

- **玩家／附身躯体**（`HasAnyTag("player", "possessedbody")`）：推 `repelled` 事件（数据 `{repeller=FX, radius=3}`），不结算伤害。消费端：[SGwilson "repelled" 状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L14273-L14310)——9 帧硬直、播 distress 动画，再用与 FX 侧同款公式 `speed = 25 * (.5*distsq/radius² - 1)` `SetMotorVel` 向外推；[SGwx78_possessedbody](../../../raw/current/scripts/stategraphs/SGwx78_possessedbody.lua#L641-L643) 同样转 `repelled` 状态。
- **其余生物**：`v.components.combat:GetAttacked(inst, 10)`——**10 点伤害以 FX 实体为攻击者记入**，结算在受击方的 combat／armor／health 侧；`GetAttacked` 的 attacker 参数只要求是实体，FX 自身没有 combat 组件也能充当攻击者（攻击者侧修正如 bonusdamagefn 因无 combat 自然跳过）（[代码](../../../raw/current/scripts/prefabs/stalker_shield.lua#L62-L70)）；有 Physics 的再进击退列表。

击退运动（[UpdateRepel](../../../raw/current/scripts/prefabs/stalker_shield.lua#L9-L41)）：每 tick 把生物面向盾心后给 `MotorVelOverride(25*k)`（k∈[-1,-0.5]，越近越快），随后 `speed` 每帧 +2、步长再 +0.25 递增，速度转正即清 override 停止；[TimeoutRepel](../../../raw/current/scripts/prefabs/stalker_shield.lua#L43-L52) 在 10 帧后强制清剩余 override——正常回调跑完时推离最多约 0.33 秒；若 FX 提前被删，任务随实体取消，已写在受击者身上的 `MotorVelOverride` 不会由此恢复（工厂没有 OnRemoveEntity 兜底）。近盾初速约 25、远盾约 12.5（单位：/s，MotorVel 尺度）。

### 清理路径

FX 自身：`persists=false` 不入存档；`animover` 与动画长度＋1 帧双兜底自删（[L112-L113](../../../raw/current/scripts/prefabs/stalker_shield.lua#L112-L113)）。击退任务（周期任务＋超时任务）都注册在 FX 实体上，FX 移除即随实体取消；但“取消”只停止后续更新，不等于恢复受击者的 `MotorVelOverride`——正常跑完才由 `UpdateRepel`／`TimeoutRepel` 清，提前删 FX 的这一收尾不完整。父子关系是 `entity:SetParent`（不登记 Lua `children`），宿主 Lua 层 Remove 不保证带走盾；盾以自身双兜底在约 2 秒内退场。

## 典型接入

### 给自定义宿主复刻受击骨盾

- 前提与选择：宿主需要一台联网实体（FX 经引擎父挂复制位置）；伤害与击退在主模拟自动执行（`StartRepel` 在 2 帧后跑），调用方无法经参数改伤害或半径——要改就复制工厂改 `REPEL_RADIUS`／`GetAttacked` 数值。
- 生效与撤销：正常演出无需手动清理（音效、动画、击退、双兜底自删都自动）；若调用方提前 `Remove` FX，击退任务虽随实体取消，受击者身上已写的 `MotorVelOverride` 不保证恢复（见[清理路径](#清理路径)）。玩家侧的硬直与位移由玩家 SG 的 `repelled` 状态接管，自定义玩家形态若要响应需自己监听该事件。
- 原版依据：[SGstalker 生成块](../../../raw/current/scripts/stategraphs/SGstalker.lua#L166-L177)、[StartRepel](../../../raw/current/scripts/prefabs/stalker_shield.lua#L57-L78)。

```lua
local function SpawnBoneShield(host)
    local fx = SpawnPrefab("stalker_shield3") -- 主盾；1/2 为小盾、4 为镜像主盾
    fx.entity:SetParent(host.entity)
    -- 2 帧后自动对半径 3 内生物结算 10 点伤害并击退；动画结束自删
end
```

- 调整与验证：替换点是变体号（3/4 为主盾档）；镜像由 SGstalker 的随机逻辑或变体 4 的工厂缩放决定，调用方无需设置。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：盾 FX 是纯表现。** `StartRepel` 对半径 3 内生物真实结算 10 点伤害（攻击者记 FX 实体）并施加击退；把它当"装饰动画"删掉会连带失去这段玩法。[StartRepel](../../../raw/current/scripts/prefabs/stalker_shield.lua#L62-L70)
- **边界：玩家不吃这 10 点伤害，也不被 FX 直接推。** 玩家与 `possessedbody` 走 `repelled` 事件，位移与硬直在各自 SG 里执行（[SGwilson](../../../raw/current/scripts/stategraphs/SGwilson.lua#L14273-L14310)）；若自定义角色 SG 没有 `repelled` 状态，事件推送后无表现。
- **边界：`fossil`／`shadow` 标签被排除。** 骨刺堆与暗影生物不受伤害也不被推——同族保护写在过滤标签里（[L55](../../../raw/current/scripts/prefabs/stalker_shield.lua#L54-L55)）。
- **边界：随机基名的跨端外观不能仅凭“无显式 netvar”断定不同步。** 基名工厂在 SetPristine 前 `n = math.random(4)` 并 `SetPrefabName("stalker_shield"..n)`；`EntityScript:SetPrefabName` 会写原生实体名，`SpawnPrefabFromSim` 又使用工厂最终 prefab（[entityscript L840-L843](../../../raw/current/scripts/entityscript.lua#L840-L843)、[mainfunctions L388-L399](../../../raw/current/scripts/mainfunctions.lua#L388-L399)）。但 `MakeShield` 的随机分支只在 `num == nil` 时执行：基名工厂被调用时随机选型并 `SetPrefabName` 改为定号名，`SpawnPrefabFromSim` 使用工厂最终 prefab。客户端收到的是基名还是定号构造、是否重跑该随机分支，均未核验——不能据此断言各端各自随机选号，也不能断言最终外观必然不同。原版从不生成基名（见缺口）；模组直连时建议用定号变体，避开这条未核验路径。[L91-L101](../../../raw/current/scripts/prefabs/stalker_shield.lua#L91-L101)
- **边界：SG 侧补镜像的 `SetScale` 在主模拟写入。** shieldtype 1/2 的随机镜像发生在生成后的主模拟实体上（[L173-L175](../../../raw/current/scripts/stategraphs/SGstalker.lua#L173-L175)）；工厂侧基准缩放（含变体 4 的镜像）在 SetPristine 前各端一致，SG 侧写入的客户端可见性未核验。
- **边界：无 NOCLICK。** 5 个注册只有 `FX` 标签、无碰撞体——可被点击命中（仅标签层面），复用时注意不要挡交互。[标签](../../../raw/current/scripts/prefabs/stalker_shield.lua#L89)
- **边界：清单外同名动画复用。** `fx.lua` 的通用 FX 数据表注册了 `shadow_shield1`～`shadow_shield6`（[条目](../../../raw/current/scripts/fx.lua#L4067-L4079)）——bank/build 与音效同 `stalker_shield`，但生成点是骨架骑士护甲 `armor_skeleton`（[生成](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L59)）。它们与骨盾 5 注册无注册或生成关系，本页不展开。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/stalker_shield.lua`；按需读取 `stategraphs/SGstalker.lua`（事件区 L115-L238 与 PickShield L57-L80）、`prefabs/stalker.lua`（依赖表、MakeStalker、atrium postinit、OnSoldiersChanged、SpawnChannelers、注册行）、`components/health.lua`（redirect 分支）、`components/combat.lua`（GetAttacked 尾段事件推送）、`stategraphs/SGwilson.lua`（repelled 处理器与状态）、`stategraphs/SGwx78_possessedbody.lua`（repelled 处理器）、`fx.lua`（shadow_shield 条目）、`armor_skeleton.lua`（shadow_shield 生成点，仅定位）。全域反查（`grep -rn "stalker_shield" --include=*.lua`）：命中 stalker_shield.lua（assets、bank/build、注册循环 L126、基名注册 L130、检索注释 L133）、SGstalker.lua:171（唯一生成语句，动态拼名）、stalker.lua:61（依赖声明）、fx.lua:4071-4072（同名资源串）、prefablist.lua:1232（文件清单）——定号变体只能由 SGstalker 动态拼名产生，基名无生成点（诚实缺口，格式参照 goosplash 页）；`stalker_shield1..4` 字面无独立生成语句。台账批 21 分诊结论复核属实（L128 循环、L130 基名、stalker.lua:61 唯一依赖）。
- 名称核验：`languages/chinese_s.po` 定向检索——`STALKER`＝"复活的骨架"（[L349995-L349997](../../../raw/current/scripts/languages/chinese_s.po#L349995-L349997)）、`STALKER_ATRIUM`＝"远古织影者"（msgid "Ancient Fuelweaver"，[L350000-L350002](../../../raw/current/scripts/languages/chinese_s.po#L350000-L350002)）；`STALKER_SHIELD` 及 4 个变体名无 `STRINGS.NAMES` 键。
- 分档说明：`size_class: standard`——单工厂 5 注册，但需承载 PickShield 选号、hasshield/免伤生成链、StartRepel 伤害与击退双子系统、诚实缺口与清单外注记；无多状态生命周期，未升 complex。
- 未覆盖：`shadow_shield1`～`6` 与 armor_skeleton 细节、引导者（shadowchanneler）与远古织影者战斗全貌、`stalker_minion`、盾动画的实际帧长（10 帧超时与动画长度的先后未运行核验）、SG 侧镜像缩放的跨端可见性；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录（0 error 0 warning 后交付）。
- 第 16 轮补正（2026-11-07）：免伤补 `not ignore_absorb` 门禁；随机基名改为“跨端网络构造与动画复制未核验”，撤回必然不同步；击退收尾区分正常跑完与提前删 FX，明确任务取消不恢复受击者 `MotorVelOverride`（补 `entityscript.lua`／`mainfunctions.lua` 来源）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/fossilspike_base|化石骨刺地面基座层]]（同为 Stalker 两代的地面附属表现）
- 返回 [[reference/prefabs/fx-status-index|增益、受击、护盾与状态反馈]]（复活的骨架）
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-04）。
