---
title: "道具拟态怪影尾粒子（itemmimic_revealed_shadow）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "拟态蠕虫（itemmimic_revealed）脚下的影子尾迹：无 AnimState 的纯 VFXEffect＋EmitterManager 粒子层，由宿主工厂在共享段 AddChild，非联网、persists=false，本体死亡与跳跃状态经 net_event 推 _disabled 开关；兼做本体脚影（宿主无 DynamicShadow）。"
aliases: ["itemmimic_revealed_shadow", "拟态蠕虫影子", "道具拟态怪影子", "拟态蠕虫尾迹", "itemmimic影子"]
tags: [prefab, fx, creature, server-client]
size_class: standard
dst_build: "steam-buildid:26474873"
sources: ["prefabs/itemmimic_revealed.lua", "stategraphs/SGitemmimic_revealed.lua", "brains/itemmimic_revealedbrain.lua", "components/itemmimic.lua", "components/shadowthrall_mimics.lua", "prefabs/cave.lua", "prefabs/forest.lua", "entityscript.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 道具拟态怪影尾粒子（itemmimic_revealed_shadow）

## 快速判断

- 职责与适用：`itemmimic_revealed_shadow` 是拟态蠕虫（itemmimic_revealed）的**贴身影子尾迹**——一个不带 AnimState 的纯 `VFXEffect` 粒子实体，每帧吐一枚"影子"粒子（`images/shadow.tex`＋粒子着色器）叠在虫脚下；出生即满 alpha，随后逐渐变淡并缩小，粒子 14 帧寿命；它同时**充当本体的脚影**（本体没挂 DynamicShadow 组件）（[注册](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L288)）。
- 本体辨析：`itemmimic_revealed`（官方名"拟态蠕虫"，英文原名 "Mimicreep"，[po](../../../raw/current/scripts/languages/chinese_s.po#L344145-L344147)）是"道具拟态"玩法的生物形态：影池系世界组件克隆地面物品埋进地牢，玩家拾取后物品在使用或到期时 `ReplacePrefab` 成这只虫（[替换点](../../../raw/current/scripts/components/itemmimic.lua#L214)）；本体带 health／locomotor／playerprox／sanityaura／timer 等完整组件，玩法详见[本体辨析](#itemmimicrevealed-本体辨析)。
- 重要副作用：纯表现层——无 combat／无伤害；仅有的状态开关是死亡与跳跃时关闭发射（`_disabled`），已出生粒子靠 14 帧寿命自然消亡。
- 修改边界：只覆盖该注册的工厂结构、生成点、端与清理；itemmimic 组件的完整到期计时（`socket_shadow_mimicry` 暂停源体系）、拟态虫 Brain 细节、`wx78_mimicspawner` 另一传染源不展开。
- 名称：`ITEMMIMIC_REVEALED_SHADOW` 在简中 `chinese_s.po` 无 `STRINGS.NAMES` 键（检索无命中），标题中文为已核验效果描述，非官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `itemmimic_revealed_shadow` | `shadow_fn`：仅 Transform（**无 AnimState、无 Network**）→ 标签 `FX`；客户端侧 `AddClientSleepable`；`persists=false` → `InitializeShadowEnvelopes` 注册颜色／缩放两条包络（自置 nil 只跑一次）→ `entity:AddVFXEffect` 单发射器配置（见下）→ `EmitterManager:AddEmitter` 挂每帧发射回调 | 粒子寿命 `(MAX_LIFETIME-1)*FRAMES`＝14 帧、最大 17 粒、发射半径 1.0、`SetSortOrder(0, -1)` 压在角色层下；[工厂](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L229-L285)、[常量](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L226-L227)、[发射器配置](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L248-L260) |

工厂内的粒子配置一览（[包络](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L204-L223)、[发射回调](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L263-L282)）：

| 配置 | 值 | 语义 |
|---|---|---|
| 颜色包络 `itemmimicshadowcolourenvelope` | alpha 1.0（出生）→ 0.9（50%）→ 0.5（100%） | 出生即满 alpha，没有淡入段；生命末期仍剩 0.5，不是完全透明 |
| 缩放包络 `itemmimicshadowscaleenvelope` | (1.5, 1.5) → (0, 0) | 粒子从 1.5 倍缩到 0 |
| 渲染资源 | `images/shadow.tex` ＋ `shaders/vfx_particle.ksh` | 通用影子贴图＋粒子着色器 |
| 生成域 | `SetSpawnVectors(-1,0,1, 1,0,1)`；AddParticle 取 `circle_emitter(0.1)` 的 (px, 0, pz)、零速度 | 以本体为中心的小片生成域，粒子不漂移 |
| 发射节奏 | 回调内"发 1 枚→计数减回 0→计数加回 1" | 每个 sim tick 恒定 1 粒，稳态约 14 粒存活（上限 17） |

## 关联机制导航

### 生成点与端

| 环节 | 行为与条件 | 证据 |
|---|---|---|
| 生成 | 宿主 `itemmimic_revealed` 工厂**共享段**（SetPristine 前）`SpawnPrefab` 并 `AddChild` 挂为子体，归零位置／旋转、`SetAABB(0.5, 2)` | [生成块](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L121-L129) |
| 端 | `if not TheNet:IsDedicated()` 门禁——**每台非专服机器各持一份本地实例**；专服不生成 | [门禁](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L121) |
| 事件通路 | 宿主以 `net_event`（`itemmimic_revealed.toggle_tail_event`）广播开关，两端实例各自监听并翻转自身 `_disabled` | [net_event](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L119)、[监听](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L128) |
| 随宿主生灭 | `AddChild` 把尾迹登记进宿主的 Lua `children` 表：宿主 `Remove` 时遍历该表删除尾迹（[entityscript Remove](../../../raw/current/scripts/entityscript.lua#L1779-L1796)），不是原生 `SetParent` 级联；尾迹自身 persists=false 不入存档 | [AddChild](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L123) |

### `_disabled` 开关协议（toggle_tail）

宿主死亡（[on_death](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L47-L49)）与 SG 两个跳跃状态的进出（[jump_pre](../../../raw/current/scripts/stategraphs/SGitemmimic_revealed.lua#L39)、[jump_pre onexit](../../../raw/current/scripts/stategraphs/SGitemmimic_revealed.lua#L77)、[try_mimic](../../../raw/current/scripts/stategraphs/SGitemmimic_revealed.lua#L105)、[try_mimic onexit](../../../raw/current/scripts/stategraphs/SGitemmimic_revealed.lua#L162)）各推一次 net_event；`toggle_tail` 只翻转 `_disabled`（[实现](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L41-L45)）。`_disabled = true` 时发射回调直接 return——**只是停发新粒子，不清理存量**；影子靠 14 帧寿命自然消隐，跳起时是"淡出"而不是瞬间消失。

### 清理路径

尾迹无自删逻辑：常驻发射直到宿主移除时由 `AddChild` 登记的 Lua `children` 表删除（宿主工厂无对应的显式 Remove，子体关系即清理通道）。宿主死亡只是关发射，虫尸实体 `Remove` 时尾迹随 `children` 表删除。

### itemmimic_revealed 本体辨析

- **传染链**：世界的 `shadowthrall_mimics` 组件（挂在 [cave.lua:572](../../../raw/current/scripts/prefabs/cave.lua#L572)、[forest.lua:726](../../../raw/current/scripts/prefabs/forest.lua#L726)）克隆一件地面物品（连存档数据与皮肤）并挂 `itemmimic` 组件（[克隆](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L57-L65)）；玩家拾取后，非装备物品 8＋0~4 秒必爆（[on_put_in_inventory](../../../raw/current/scripts/components/itemmimic.lua#L59-L69)）、装备品在使用（攻击／工作／佩戴者受击）时 5 帧后爆（[使用触发](../../../raw/current/scripts/components/itemmimic.lua#L123-L132)）、耐久变动也会触发（[percentusedchange](../../../raw/current/scripts/components/itemmimic.lua#L110)），`TurnEvil` 丢弃物品并 ReplacePrefab 成拟态蠕虫，落地即跳（`jump` 事件起 5 秒 `recently_spawned` 免近计时）。
- **本体组件**：health 100（`SHADOW_LEECH_HEALTH`）／locomotor 跑速 6／playerprox(1.0, 2.5)（延迟 35 帧后启用踩踏即死 `DisperseFromBeingSteppedOn`）／sanityaura 小幅降理智／loot nightmarefuel（[组件装配](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L141-L176)、[调值](../../../raw/current/scripts/tuning.lua#L6974-L6975)）；`SetNoLoot` 可关掉掉落并存档（WX78 拟态心脏玩法用）。
- **行为**：Brain 让它远离玩家、并在 15 格内寻找可模仿目标发起 `NUZZLE` 动作（`try_mimic` 状态跳扑，[Brain](../../../raw/current/scripts/brains/itemmimic_revealedbrain.lua#L59-L76)）；"眼睛"弹出／缩回由 Brain 推 `eye_up`／`eye_down` 驱动本体动画。这些玩法链不在本页展开。

## 典型接入

### 复刻"无动画实体的贴身粒子尾迹"

- 前提与选择：跟随体只需要一团柔和大颗粒（影子／雾／灵气）且不想带动画库时，用 `VFXEffect`＋`EmitterManager` 组合：Transform-only 实体、包络管出生与消亡、发射回调恒定吐粒。
- 生效与撤销：宿主工厂在共享段 AddChild 挂子体（每台机器各一份，天然随宿主删除）；需要"暂停发射"语义时用一个布尔 upvalue 守卫回调，经 `net_event` 远程翻转。

```lua
local function shadow_fn()
    local inst = CreateEntity()
    inst:AddTag("FX")
    inst.persists = false
    inst.entity:AddTransform()

    EnvelopeManager:AddColourEnvelope("mytail_colour",
        { { 0.00, { 1, 1, 1, 1.0 } }, { 1.00, { 1, 1, 1, 0.5 } } })
    EnvelopeManager:AddVector2Envelope("mytail_scale",
        { { 0, { 1.5, 1.5 } }, { 1, { 0, 0 } } })

    local lifetime = 14 * FRAMES
    local effect = inst.entity:AddVFXEffect()
    effect:InitEmitters(1)
    effect:SetRenderResources(0, "images/shadow.tex", "shaders/vfx_particle.ksh")
    effect:SetMaxNumParticles(0, 17)
    effect:SetMaxLifetime(0, lifetime)
    effect:SetSortOrder(0, -1)
    effect:SetColourEnvelope(0, "mytail_colour")
    effect:SetScaleEnvelope(0, "mytail_scale")
    effect:SetRadius(0, 1.0)

    local circle_emitter = CreateCircleEmitter(0.1)
    EmitterManager:AddEmitter(inst, nil, function()
        local px, pz = circle_emitter()
        inst.VFXEffect:AddParticle(0, lifetime, px, 0, pz, 0, 0, 0)
    end)
    return inst
end
```

- 调整与验证：替换点是贴图／包络名／寿命帧数；发射回调每 tick 执行，加随机量要写进回调内部。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：这个"影子"不是 DynamicShadow。** 本体工厂只加 Transform／AnimState／SoundEmitter／Network（[L89-L92](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L89-L92)），脚下阴影完全由粒子层承担——删掉尾迹（或专服视角讨论阴影）时注意本体没有任何影子组件。
- **边界：工厂生成点在共享段（SetPristine 前）。** 客户端工厂也会各建一份本地子体（与 shadowrift_portal_fx 同款结构）；它不是联网实体，别对它读 netvar 或指望主模拟控制画面。
- **边界：`_disabled` 是"停发"不是"清屏"。** 关闭后已出生粒子按 14 帧寿命继续可见（[守卫](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L268-L270)）；需要瞬间消失要另写清理。
- **边界：粒子节奏是硬编码的恒定 1 粒/tick。** 回调里"while 发射→减到 0→加回 1"的写法实际就是每帧 1 粒（[L265-L281](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L265-L281)）；改发射率要重写该 upvalue 逻辑，调 `SetMaxNumParticles` 不改变节奏。
- **边界：net_event 通路上专服不存在监听者。** 生成与监听都在 `not TheNet:IsDedicated()` 分支内（[L121-L129](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L121-L129)），专服只有宿主本体收发事件——纯表现开关放本地是本设计的取舍。
- **边界：本体死亡尾迹"关灯"而非删除。** death 只推 toggle（[on_death](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L47-L49)）；尾迹实体真正的删除时刻是宿主实体 `Remove` 时遍历 Lua `children` 表，与"死亡演出"无耦合。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/itemmimic_revealed.lua`、`stategraphs/SGitemmimic_revealed.lua`（jump_pre／try_mimic 的 toggle 推送点）、`components/itemmimic.lua`；按需读取 `brains/itemmimic_revealedbrain.lua`（eye_up/eye_down 与 NUZZLE 检索段）、`components/shadowthrall_mimics.lua`（克隆与 itemmimic 挂载）、`prefabs/cave.lua`／`prefabs/forest.lua`（世界组件挂载）、`tuning.lua`（SHADOW_LEECH_*）。对覆盖注册名全域反查：`itemmimic_revealed_shadow` 命中注册（itemmimic_revealed.lua:288）、依赖（:10）、生成（:122）——**生成点唯一**；`itemmimic_revealed` 命中注册（:287）、生成（components/itemmimic.lua:214 ReplacePrefab）；`itemmimic_revealed.toggle_tail_event` 推送点共 5 处（SGitemmimic_revealed.lua:39/:77/:105/:162、prefab on_death）、监听在生成块 L128。台账批 1 结论复核属实（纯 VFXEffect＋EmitterManager、非联网、persists=false、AddChild 于本体）。`itemmimic_revealed.lua` 内无清单外 FX 注册（shadow_fn 即本页唯一 FX 注册）。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.ITEMMIMIC_REVEALED`＝"拟态蠕虫"（msgid "Mimicreep"，[L344145-L344147](../../../raw/current/scripts/languages/chinese_s.po#L344145-L344147)）；`ITEMMIMIC_REVEALED_SHADOW` 无 NAMES 键，标题使用效果描述。
- 分档说明：`size_class: standard`——单注册但需承载 VFXEffect 粒子结构表、共享段生成与 net_event 开关协议、本体传染链辨析；未选 simple。
- 未覆盖：itemmimic 组件到期计时全貌（`socket_shadow_mimicry` 暂停源、`ShouldItemMimicBeRevealedFor` 判定）、拟态虫 Brain 其余节点与 `NUZZLE` 动作的完整表现链、`wx78_mimicspawner` 传染源、粒子的实际渲染尺寸与跨端表现、动画包内容；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录（0 error 0 warning 后入库）。
- 第 16 轮补正（2026-11-07）：粒子改为“出生即满 alpha、随后变淡缩小，末期 alpha 仍 0.5”；删除“淡入”表述；父子清理写明 `children` 表删除（补 `entityscript.lua`）；本体官方名改用简中 msgstr“拟态蠕虫”并注英文原名。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/wx78_big_spark|WX78 外围表现族]]（同为 AddChild 挂接的本地表现层，含"代理无 AnimState"辨析）
- [[reference/prefabs/shadow_bishop_fx|暗影主教羽群与暗影裂隙散件]]（共享段生成粒子层的同款结构对照）
- 返回 [[reference/prefabs/fx-shadow-index|暗影生物、裂隙生物与附着]]（道具拟态怪）
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
