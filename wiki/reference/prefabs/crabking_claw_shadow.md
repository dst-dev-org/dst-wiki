---
title: "地面贴片与投影阴影 FX 族（crabking_claw_shadow）"
created: "2026-11-04"
updated: "2026-11-08"
type: prefab
summary: "4 文件 6 注册的地面贴片／投影阴影：巨钳水下阴影与挥击弧、冰眼结晶器冰圈（真实冻结／降温／grog）、绝望螨焦痕淡出、柱基座与裂口；宿主玩法不在本页。"
aliases: ["crabking_claw_shadow", "crabking_claw_swipe_fx", "deerclopseyeball_sentryward_fx", "fused_shadeling_bomb_scorch", "daywalker_pillar_base_fx", "daywalker_pillar_hole", "巨钳阴影", "巨钳挥击弧", "爆炸焦痕", "冰圈贴片", "柱子基座", "地面裂口", "地面贴片", "投影阴影"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/crabking_claw.lua", "stategraphs/SGcrabkingclaw.lua", "prefabs/deerclopseyeball_sentryward.lua", "prefabs/fused_shadeling_bomb.lua", "prefabs/shadowthrall_projectile_fx.lua", "prefabs/daywalker_pillar.lua", "tuning.lua", "entityscript.lua", "simutil.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 地面贴片与投影阴影 FX 族（crabking_claw_shadow）

## 快速判断

- 职责与适用：本页收 4 个文件 6 个注册——`crabking_claw_shadow`（巨钳水下投影）、`crabking_claw_swipe_fx`（巨钳挥击贴地弧）、`deerclopseyeball_sentryward_fx`（冰眼结晶器冰圈，带真实冻结／降温／grog）、`fused_shadeling_bomb_scorch`（绝望螨爆炸焦痕）、`daywalker_pillar_base_fx`／`daywalker_pillar_hole`（柱基座与地面裂口）。[巨钳侧注册](../../../raw/current/scripts/prefabs/crabking_claw.lua#L281-L282)、[冰圈注册](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L648)、[焦痕注册](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L453)、[柱体侧注册](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L839-L840)
- 优先使用：按对象选"贴地弧／阴影／焦痕／裂口"；要冻结／降温／grog 才用冰圈，它是主模拟每帧结算，不是纯视觉。宿主技能、爆炸伤害、囚禁等玩法不写在本页（宿主 Wiki 待建）。
- 重要副作用：冰圈会灭火、缓冷、把温度压到 -2、给 grog ＋0.1；其余 5 个无 combat／temperature／freezable 结算。宿主 `temperatureoverrider` 是另一套区块温度（[L498-L499](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L498-L499)），本页只记 FX 的 `OnUpdateIceCircle`。
- 修改边界：6 个工厂都 AddNetwork＋SetPristine，客户端通常早退；只有焦痕用 netvar。层位／朝向各工厂硬编码，不是通用 FX 协议。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `crabking_claw_shadow` | shadowfn：Transform＋AnimState＋SoundEmitter＋Network → 六面朝 → `ignorewalkableplatforms`／`notarget`／`NOCLICK`／`NOBLOCK` → `persists=false` → `LAYER_BELOW_GROUND`＋sort `ANIM_SORT_ORDER_BELOW_GROUND.UNDERWATER` → bank `crab_claw`／build `crab_king_claw_shadow_build`／`idle` 循环 → 客户端早退，主模拟无附加 | 水下投影；自身 0.7 缩放被注释，实际随宿主 0.7 变换。[工厂](../../../raw/current/scripts/prefabs/crabking_claw.lua#L210-L244)、[层位](../../../raw/current/scripts/prefabs/crabking_claw.lua#L218-L228)、[缩放注释](../../../raw/current/scripts/prefabs/crabking_claw.lua#L230-L231) |
| `crabking_claw_swipe_fx` | swipefn：Transform＋AnimState＋Network → FX／NOCLICK → bank/build `crabking_claw_swipe_fx`／`atk1` → OnGround＋`LAYER_BACKGROUND`＋sort 3 → 缩放 1/0.7 → SetPristine → 客户端壳早退 → 主模拟 `persists=false`＋animover Remove | 单动画挥击弧；八朝向调用被注释。[工厂](../../../raw/current/scripts/prefabs/crabking_claw.lua#L246-L278)、[层位与缩放](../../../raw/current/scripts/prefabs/crabking_claw.lua#L258-L266) |
| `deerclopseyeball_sentryward_fx` | fxfn：Transform＋AnimState＋SoundEmitter＋Network → 缩放 1.1 → bank/build `deer_ice_circle`／`pre` → LightOverride 0.1＋FinalOffset 1 → OnGround＋`LAYER_BACKGROUND`＋sort 3 → FX 标签 → SetPristine → 客户端早退 → 主模拟 `_radius` 初始化、KillFX／OnAnimOver、0 延时任务 OnInitIceCircle、循环音、`persists=false` | 玩法型地面圈；半径初始 0.25（加载时 3.5），每帧向 3.5 收敛，播 `pst` 收场。[工厂](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L588-L636)、[初始化](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L617-L633) |
| `fused_shadeling_bomb_scorch` | scorchfn：Transform＋AnimState＋Network → bank/build `burntground`／`idle` → OnGround＋`LAYER_BACKGROUND`＋sort 3 → NOCLICK／FX → `net_byte _fade` 初值 40（30 帧延迟＋10 帧淡出）＋每帧更新 → 缩放 0.7 → SetPristine → 客户端监听 `fadedirty` → 主模拟随机旋转、`persists=false` | 焦痕按帧自淡出自删；唯一使用 netvar 的注册。[工厂](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L412-L448)、[淡出常量](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L379-L380) |
| `daywalker_pillar_base_fx` | fn_base：Transform＋AnimState＋Network → FX／NOCLICK → bank/build `daywalker_pillar`／`idle` 循环 → FinalOffset 1 → `pillar_full` 换 `pillar_full_base`、隐藏 `fx_vibration`、LightOverride 1 → SetPristine → 客户端早退 → 主模拟 `persists=false` | 柱体基座，未设 OnGround（默认朝向）；用 `pillar_full_base` 符号替代柱身。[工厂](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L775-L802)、[符号与层](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L787-L791) |
| `daywalker_pillar_hole` | fn_hole：Transform＋AnimState＋Network → decor／NOCLICK（FX 标签被注释）→ bank/build `daywalker_hole`／`ground_1` → OnGround＋`LAYER_BACKGROUND`＋sort 3 → `pillar_crack_red` LightOverride 1 → SetPristine → 客户端早退 → 主模拟 `persists=false` | 贴地裂口；`decor` 而非 FX；宿主淡出时改播 `fadeout`。[工厂](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L806-L834)、[标签](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L813-L815) |

按 6 个注册名（含大小写变体）检索 `languages/chinese_s.po` 无 `STRINGS.NAMES.*` 命中；命中仅为宿主名：巨钳 `CRABKING_CLAW`（[L341792](../../../raw/current/scripts/languages/chinese_s.po#L341792)）、开裂的柱子 `DAYWALKER_PILLAR`（[L341992](../../../raw/current/scripts/languages/chinese_s.po#L341992)）、冰眼结晶器 `DEERCLOPSEYEBALL_SENTRYWARD`（[L342082](../../../raw/current/scripts/languages/chinese_s.po#L342082)）、绝望螨 `FUSED_SHADELING_BOMB`（[L343097](../../../raw/current/scripts/languages/chinese_s.po#L343097)）。标题中文为已核验效果描述，非官方译名；文件名注册名只作族入口。

## 关联机制导航

netvar 仅焦痕 `_fade`（`net_byte`，脏事件 `fadedirty`）；其余 5 个无 netvar，靠实体网络同步与动画。生成与清理由宿主或自身回调控制，宿主玩法不展开。

### 生成、父挂与清理

| 注册 | 生成点／父挂 | 清理 | 端与证据 |
|---|---|---|---|
| `crabking_claw_shadow` | 宿主 `fn` 末尾 `SpawnChild`→`AddChild`→`SetParent` | 宿主 `death`／`onremove` 两路 Remove | 主模拟；`persists=false`。[生成](../../../raw/current/scripts/prefabs/crabking_claw.lua#L205)、[回收](../../../raw/current/scripts/prefabs/crabking_claw.lua#L38-L48)、[SpawnChild](../../../raw/current/scripts/entityscript.lua#L896-L909)、[AddChild](../../../raw/current/scripts/entityscript.lua#L921-L935) |
| `crabking_claw_swipe_fx` | SG `attack` 第 18 帧 SpawnPrefab＋SetParent（提前 3 帧） | 自身 animover Remove | 主模拟（SG 在 master 分支内）；`persists=false`。[生成](../../../raw/current/scripts/stategraphs/SGcrabkingclaw.lua#L227-L231)、[自删](../../../raw/current/scripts/prefabs/crabking_claw.lua#L274-L275) |
| `deerclopseyeball_sentryward_fx` | 宿主插入眼球时在宿主坐标生成，不父挂 | 取出眼球／宿主移除各 KillFX；播 `pst`、取消任务、动画长＋0.25 秒后 Remove | 主模拟；`persists=false`。[生成](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L177-L180)、[取出](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L215-L218)、[移除](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L235-L239)、[KillFX](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L528-L542) |
| `fused_shadeling_bomb_scorch` | 本体爆炸后；`shadowthrall_projectile_fx` 的 AOE 后 | 自身按 `_fade` 自删：主模拟 Remove，客户端置 0 隐藏 | 主模拟生成；`net_byte` 同步。[爆炸](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L95-L106)、[复用](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L53-L55)、[自删](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L400-L410) |
| `daywalker_pillar_base_fx` | 宿主构造时 SpawnPrefab＋SetParent | 挖毁 `base:Remove()`；囚徒移除分支 `ErodeAway(base)` 后 Remove | 主模拟；`persists=false`。[生成](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L699-L703)、[挖毁](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L559)、[囚徒](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L734-L735)、[ErodeAway](../../../raw/current/scripts/simutil.lua#L575-L597) |
| `daywalker_pillar_hole` | 宿主构造时 SpawnPrefab＋SetParent | 挖毁 OnDestroyed→FadeOutHole（解父、`fadeout`、animover Remove）；囚徒移除同 | 主模拟；`persists=false`。[生成](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L699-L700)、[FadeOutHole](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L523-L528)、[摧毁](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L541-L543) |

### 冰圈结算（deerclopseyeball_sentryward_fx）

| 项 | 规则 | 证据 |
|---|---|---|
| 作用域 | `FindEntities(x, 0, z, _radius)`，oneof `{heatrock, freezable, fire, smolder, inventoryitemtemperature}`；排除 `playerghost`／`INLIMBO`／`flight`／`invisible` 与全部 `*_fueled`；跳过已死实体 | [标签表](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L544-L553) |
| 半径 | 每帧 `_radius × 0.98 + 3.5 × 0.02`；新生成 0.25，加载时 3.5 | [扩张](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L550-L551)、[初值](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L617)、[数值](../../../raw/current/scripts/tuning.lua#L7971) |
| 灭火 | `burnable` 且无 `fueled` → `Extinguish()` | [灭火](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L555-L557) |
| 冻结 | `freezable` 未冻结且 `coldness < ResolveResistance() × 0.7` → `AddColdness(0.1, 1, true)` | [冻结](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L558-L563) |
| 降温 | 实体温度高于 -2 → 设为 -2 | [降温](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L564-L567)、[数值](../../../raw/current/scripts/tuning.lua#L698) |
| grog／催眠 | 有 `grogginess`、未昏倒且 `grog_amount < 0.1` → `AddGrogginess(0.1)` | [grog](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L568-L573)、[数值](../../../raw/current/scripts/tuning.lua#L699) |
| 收场 | `KillFX` 置 `_radius=0`、播 `pst`、取消任务、动画长＋0.25 秒后 Remove；`OnAnimOver` 停循环音 | [KillFX](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L528-L542)、[停音](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L584-L586) |

## 典型接入

### 借用落地焦痕

- 前提与选择：主模拟生成；焦痕自带 30 帧停留＋10 帧淡出后自删，不需宿主计时清理；落点用 Transform 设位置。
- 生效与撤销：默认 0.7 缩放，可在生成后 `SetScale` 覆盖（原版 `shadowthrall_projectile_fx` 用 0.9）；不要在客户端单独 Spawn——客户端只隐藏，Remove 只在主模拟。
- 原版依据：[do_explosion_effect](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L95-L106)。

```lua
local function SpawnScorchDecal(x, z)
    if not TheWorld.ismastersim then
        return
    end
    local fx = SpawnPrefab("fused_shadeling_bomb_scorch")
    fx.Transform:SetPosition(x, 0, z)
    return fx
end
```

- 调整与验证：注册名需经宿主 `prefabs` 表声明或世界预载保证可生成（焦痕已在炸弹依赖表 [L8-L13](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L8-L13)）；`persists=false` 不写存档。

## 常见陷阱与限制

- **误区：冰圈只是视觉**：灭火、冻结、降温与 grog 全写在 FX 的 `OnUpdateIceCircle`，停掉 FX 就没有这些效果；宿主 `temperatureoverrider` 只做区块环境温度。[结算](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L550-L576)、[宿主覆盖](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L498-L499)
- **边界：巨钳阴影的 0.7 缩放是注释掉的**：它靠宿主 0.7 变换，单独 Spawn 是 1.0；层位在地面之下并用水下排序，陆地场景未核验。[注释](../../../raw/current/scripts/prefabs/crabking_claw.lua#L230-L231)、[宿主缩放](../../../raw/current/scripts/prefabs/crabking_claw.lua#L126-L127)
- **边界：挥击弧的 1/0.7 是给 0.7 宿主的补偿**：换到未缩放宿主会大约放大 43%；八朝向调用被注释。[缩放](../../../raw/current/scripts/prefabs/crabking_claw.lua#L265-L266)、[注释](../../../raw/current/scripts/prefabs/crabking_claw.lua#L256)
- **边界：焦痕客户端不自删**：客户端只 `_fade:set_local(0)` 并置 alpha 0 隐藏，Remove 只在主模拟。[客户端分支](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L401-L409)
- **边界：柱基座与裂口没有自身清理回调**：依赖宿主挖毁／囚徒移除分支；`daywalker_pillar_hole` 标签是 `decor` 不是 `FX`，按 FX 标签筛选会漏。[标签](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L813-L815)、[清理](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L541-L543)
- **边界：`SpawnChild` 依赖宿主 prefab 表**：宿主带 `self.prefabs` 而缺名会断言失败；巨钳在依赖表声明 shadow 与 swipe。[断言](../../../raw/current/scripts/entityscript.lua#L896-L904)、[依赖表](../../../raw/current/scripts/prefabs/crabking_claw.lua#L10-L14)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394；4 个文件中的 6 个工厂全文（6 注册）、宿主生成／清理锚点、SG 第 18 帧生成、tuning 半径／温度／grog 数值、`SpawnChild`／`AddChild`、`ErodeAway`、简中 `.po` 名称检索已核验。未覆盖：动画帧数／时长、贴地视觉对齐、水面／陆地实际渲染层、二进制资源；宿主玩法（巨钳 AI、结晶器产冰、炸弹伤害、囚禁）未展开。
- 游戏运行：未运行。
- 结构检查：validate_page.py、validate_source_links.py --strict 通过；Lua 示例经 parser_probe --syntax-only 通过。

相关：[[reference/prefabs/deer_ice_circle|无眼鹿冰火法阵]]（使用同名 `deer_ice_circle` 动画资源的地面圈）、[[reference/prefabs/crab_king_bubble1|蟹王伴随表现特效族]]（同 Boss 表现族的分层与生成协议）。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-10）。
