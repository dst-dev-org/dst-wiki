---
title: "沃特施法特效（purebrilliance_castfx／horrorfuel_castfx）"
created: "2026-10-29"
updated: "2026-11-08"
type: prefab
summary: "纯粹辉煌／纯粹恐惧物品施放（mermbuffcast）时挂在施放者身上的施法动画：普通与骑乘四注册，恐惧燃料变体在客户端再挂非联网顶部子实体；消耗与随从 buff 由物品 spellcaster 完成。"
aliases: ["wurt_casting_fx", "purebrilliance_castfx", "purebrilliance_castfx_mount", "horrorfuel_castfx", "horrorfuel_castfx_mount", "沃特施法特效", "施法特效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/wurt_casting_fx.lua", "stategraphs/SGwilson.lua", "prefabs/purebrilliance.lua", "prefabs/horrorfuel.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 沃特施法特效（purebrilliance_castfx／horrorfuel_castfx）

## 快速判断

- 职责与适用：沃特（官方译名"沃特"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L351890-L351892)）施放纯粹辉煌／纯粹恐惧物品（`mermbuffcast` 标签路由到 SGwilson `mermbuffcastspell` 状态）时挂在施放者身上的施法动画：bank／build `wurt_planar_casting_fx`，FinalOffset 1；无玩法作用，物品消耗与鱼人随从 buff 都由物品 spellcaster 施法函数完成。[工厂](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L8-L52)
- 优先使用：查"施法特效／纯粹辉煌施法"选本页；骑乘使用 `_mount` 变体（六面向）。
- 重要副作用：恐惧燃料变体在非专服客户端额外生成非联网"顶部"子实体（finaloffset 2，微发光）；施法者身上另有 staff_castinglight 照明（状态图生成，非本厂）。[子实体](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L56-L86)
- 修改边界：客户端专属初始化（clientpostinit）只在非专服执行；实体动画播完由主模拟 animover 自删，状态退出另有兜底删除。[双清理](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16987-L16996)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `purebrilliance_castfx` | MakeFx(data)：Transform＋AnimState＋SoundEmitter＋Network → FX 标签 → 四面向 → 播 `purebrilliance_cast` → 客户端／公共 postinit → SetPristine → 客户端早退 → 主模拟 `persists=false`、animover 自删 | PureBrilliancePostInit：bloom、pb_energy_loop／pb_ray／SparkleBit 符号微发光、LightOverride 0.1。[注册](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L106-L110)、[postinit](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L88-L94) |
| `purebrilliance_castfx_mount` | 同上，六面向，anim `purebrillance_cast_mount`（anim 名在源码中就是这个拼写） | 骑乘施放时由状态图拼接后缀选择。[注册](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L111-L116) |
| `horrorfuel_castfx` | 同工厂，anim `horrorfuel_bottom_cast`；HorrorFuelPostInit：乘法色 alpha 0.5、horror_fx 符号微发光、点过滤；clientpostinit 生成顶部子实体 | 子实体播 `horrorfuel_top_cast`（显式不循环）、LightOverride 0.1，SetParent 到本实体（非联网）。[注册](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L117-L122)、[postinit](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L96-L101)、[子实体](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L56-L86) |
| `horrorfuel_castfx_mount` | 同上，六面向，底部 anim `horrorfuel_bottom_cast_mount`、顶部 `horrorfuel_top_cast_mount` | 骑乘变体。[注册](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L123-L129) |

四个注册的 `PlayAnimation(data.anim)` 均未显式传循环参数（顶部子实体显式传 false），实际是否重复未核验。`wortox`／`wurt` 一类清单名 `wurt_casting_fx` 是族名（沿用文件名），不是注册名。物品官方译名：PUREBRILLIANCE"纯粹辉煌"（[译名](../../../raw/current/scripts/languages/chinese_s.po#L346760-L346762)）、HORRORFUEL"纯粹恐惧"（[译名](../../../raw/current/scripts/languages/chinese_s.po#L343345-L343347)）；FX 注册名本身无 NAMES 条目。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 路由：ACTIONS.CASTSPELL | 物品带 `mermbuffcast` 标签（purebrilliance／horrorfuel 都有）时 ActionHandler 路由到 `mermbuffcastspell` 状态 | 与角色无关，只看物品标签 | [标签](../../../raw/current/scripts/prefabs/purebrilliance.lua#L62)、[标签](../../../raw/current/scripts/prefabs/horrorfuel.lua#L196)、[路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1307-L1314) |
| 宿主状态：mermbuffcastspell | 读 `item.fxprefab`（缺省 purebrilliance_castfx）、`item.castsound`、`item.lightcolour`；7 帧生成 `fxprefab..(mounted and "_mount" or "")` 并 SetParent 到施放者 | 主模拟状态；15 帧生成 staff_castinglight（lightcolour 上色）；51 帧执行动作 | [参数读取](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16942-L16950)、[生成与挂载](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16955-L16961)、[照明](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16962-L16966) |
| 物品侧 spellcaster | `PerformBufferedAction` 触发 spellcaster 施法：给 leader 的 lunarminion（纯粹辉煌）／shadowminion（纯粹恐惧）随从加 `wurt_merm_planar` debuff、推 cheer，然后消耗物品 | 法术效果与 FX 无读写关系 | [purebrilliance 施法](../../../raw/current/scripts/prefabs/purebrilliance.lua#L36-L49)、[物品配置](../../../raw/current/scripts/prefabs/purebrilliance.lua#L84-L93)、[horrorfuel 施法](../../../raw/current/scripts/prefabs/horrorfuel.lua#L134-L147)、[[reference/components/spellcaster#关键接口]] |
| 清理协议 | 主模拟 animover 自删；状态 onexit 主动 Remove spellfx 与 spelllight（提前中断时） | 双路径独立；子实体随父删除 | [onexit](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16987-L16996)、[animover](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L46) |
| 端分界 | clientpostinit 仅非专服执行（恐惧燃料顶部子实体只存在于有渲染的端）；专服只有底部动画实体 | 专服照样注册完整联网实体 | [clientpostinit](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L30-L32) |
| 声音 | 本厂给实体加了 SoundEmitter 组件但不播音；施法声由状态图用 `item.castsound` 播在施放者身上（lunar／shadow 两种） | 物品字段决定 | [组件声明](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L14)、[物品 castsound](../../../raw/current/scripts/prefabs/purebrilliance.lua#L93) |
| 动画资源 | bank／build `wurt_planar_casting_fx`，四个底部 anim＋两个顶部 anim | 资源包内容未解包 | [设置](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L25-L28) |

## 典型接入

### 复用为新物品的施法动画

- 前提与选择：给物品设 `inst.fxprefab`（填不含 `_mount` 的基础注册名，骑乘时状态再拼后缀）、`inst.castsound`、`inst.lightcolour` 并加 `mermbuffcast` 标签，即可走原版状态；或直接生成后 SetParent 到施放者。
- 生效与撤销：FX 按动画播完自删；走状态时中断由 onexit 兜底。

```lua
-- 物品侧（主模拟与客户端共用字段）
inst:AddTag("mermbuffcast")
inst.fxprefab = "purebrilliance_castfx"
inst.castsound = "meta4/casting/lunar"
inst.lightcolour = { 53/255, 132/255, 148/255 }
```

- 调整与验证：后缀拼接只认 `_mount`，自定义变体名必须以它结尾；fxprefab 缺省值是 purebrilliance_castfx，不带标签的物品走普通 castspell 状态。[缺省](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16944)

## 常见陷阱与限制

- **误区：只有沃特能触发本特效**：路由只看物品 `mermbuffcast` 标签，任何角色施放都会出特效；物品的实际获取途径（技能限制等）本页未穷举。[路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1307-L1314)
- **误区：mount 变体 anim 名拼写一致**：`purebrillance_cast_mount` 在源码中缺一个 i（注册名是 purebrilliance），复用动画时按源码字符串对齐。[注册](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L111-L116)
- **边界：恐惧燃料是"上下两截"**：底部联网实体＋顶部客户端子实体，只在一端删顶部不会同步给其他端；整组退场由父实体生命周期管理；两端可见内容不同（专服无顶部）。[结构](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L30-L32)
- **边界：SoundEmitter 挂而未用**：工厂加了组件但全文件无 PlaySound；不要以为本实体自带施法音。[组件声明](../../../raw/current/scripts/prefabs/wurt_casting_fx.lua#L14)
- **边界：晚加入时序未核验**：施放中途晚加入的客户端是否补顶部子实体未核验。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；129 行工厂全文与四注册、mermbuffcastspell 状态全程（参数读取／生成／照明／消耗／双清理）、两个物品的标签与 spellcaster 配置、译名检索已核验；动画资源内容与时长、物品获取途径、晚加入表现未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-character-index|角色技能、施法与读书]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-04）。
