---
title: "警钟暗影微粒特效（pocketwatch_weapon_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "旺达警钟充能态的环绕暗影微粒：装备且有燃料时挂在持用者身上跟随武器符号，按移动与旺达年龄阶段连续发射，攻击时停发。"
aliases: ["pocketwatch_weapon_fx", "警钟特效", "警钟", "Alarming Clock", "旺达武器特效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/pocketwatch_weapon_fx.lua", "prefabs/pocketwatch_weapon.lua", "prefabs/wanda.lua", "prefabs/cane_victorian_fx.lua", "stategraphs/SGwilson.lua", "stategraph.lua", "fx.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 警钟暗影微粒特效（pocketwatch_weapon_fx）

## 快速判断

- 职责与适用：旺达武器警钟（pocketwatch_weapon，Alarming Clock）充能态（有燃料）的环绕粒子实体：联网 FX 壳加本地 VFXEffect 单发射器，宿主装备时挂到持用者身上、跟随 swap_object 符号；不是攻击爆发，而是按移动速度与旺达年龄阶段连续补粒，攻击时停发。FX 本体无组件、无光照、无声音，不参与武器数值。[工厂](../../../raw/current/scripts/prefabs/pocketwatch_weapon_fx.lua#L79-L96)
- 优先使用：查“警钟特效／旺达武器微粒”选本页；警钟宿主页未建，燃料门控与装备协议证据见 [[reference/components/fueled]] 与 [[reference/components/equippable]]；与 [[reference/prefabs/cane_victorian_fx]] 共用 envelope 名（数据不同）。
- 重要副作用：发射速率读取 owner.age_state——只有旺达玩家实体携带该字段（旧 3-6 粒/秒、普通 2-3、其余 1-1），移动越快越多；owner 带 "attack" 标签或播 pocketwatch_atk 系动画时整段停发。[速率](../../../raw/current/scripts/prefabs/pocketwatch_weapon_fx.lua#L135-L143)、[停发](../../../raw/current/scripts/prefabs/pocketwatch_weapon_fx.lua#L130)
- 修改边界：专用服务器提前返回不建粒子；客户端各自本地发射；FX 的生成与移除完全由宿主武器驱动（装备、入模型、燃料事件三条路径）。[端分界](../../../raw/current/scripts/prefabs/pocketwatch_weapon_fx.lua#L89-L96)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `pocketwatch_weapon_fx` | fn：Transform＋Network＋FX 标签 → SetPristine → 专服早退 → 注册 envelope 并配 VFXEffect（1 发射器）→ EmitterManager 回调 | 单一注册。sparkle 发射器：fx/pocketwatch.tex＋add 着色器＋bloom（BlendMode 为 AlphaBlended），5 帧 UV（0-3 表盘、4 齿轮，齿轮角速度为 ±11 再加 [-2,2] 扰动，其余 [-5,5]），上限 256、寿命 0.75s；发射按 PPS 累积而非爆发。[构造](../../../raw/current/scripts/prefabs/pocketwatch_weapon_fx.lua#L98-L113)、[粒子形态](../../../raw/current/scripts/prefabs/pocketwatch_weapon_fx.lua#L57-L77)、[注册](../../../raw/current/scripts/prefabs/pocketwatch_weapon_fx.lua#L158) |

`STRINGS.NAMES.POCKETWATCH_WEAPON` 官方译名“警钟”（msgid Alarming Clock）；未发现该 FX 注册名的 NAMES 翻译条目，标题中“暗影微粒特效”为已核验效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L346470-L346472)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 宿主武器 pocketwatch_weapon | prefabs 声明依赖；TryStartFx 在装备时 SpawnPrefab → AddFollower → SetParent(owner) → FollowSymbol(owner, "swap_object", 15, 70, 0)，且仅燃料非空时启动 | 监听与生成都在武器实体（主模拟组件路径）；FX 联网复制 | [依赖声明](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L7-L10)、[TryStartFx](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L12-L34) |
| 清理协议 | 脱下（onunequip）、装备到模型（onequiptomodel）、燃料耗尽（OnFuelChanged）都直接 Remove 实体；换持有者时先删旧再建新 | 无退场动画，立即消失 | [StopFx](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L36-L41)、[三条路径](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L52-L61)、[燃料事件](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L82-L96) |
| 燃料门控 | FX 存在与武器充能态同源：非空时 shadow_item 标签＋POCKETWATCH_SHADOW_DAMAGE；耗尽撤标签、改 POCKETWATCH_DEPLETED_DAMAGE 并 StopFx | 武器 fueled 组件 NIGHTMARE 燃料 | [[reference/components/fueled#关键接口]]；[充能切换](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L84-L94) |
| 攻击停发 | owner 带 "attack" 实体标签或播 pocketwatch_atk_pre／_pre_2／atk 动画时不发射；"attack" 来自玩家状态图攻击状态的 state tag（主模拟映射为实体标签） | 攻击表现另由 fx.lua 同名 bank 的动画 FX 承担 | [停发条件](../../../raw/current/scripts/prefabs/pocketwatch_weapon_fx.lua#L130)、[标签映射](../../../raw/current/scripts/stategraph.lua#L501-L503)、[状态挂标签](../../../raw/current/scripts/stategraph.lua#L553-L566)、[攻击状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10787-L10788) |
| 旺达年龄阶段 | owner.age_state 三档决定 PPS：old 3/6、normal 2/3、其他（含非旺达）1/1；再按移动距离 Lerp | age_state 由 wanda.lua 的 onhealthchange 按生命值阈值切换 | [速率](../../../raw/current/scripts/prefabs/pocketwatch_weapon_fx.lua#L135-L143)、[阶段切换](../../../raw/current/scripts/prefabs/wanda.lua#L166-L199)、[字段赋值](../../../raw/current/scripts/prefabs/wanda.lua#L90) |
| envelope 名复用 | 本厂注册 `cane_victorian_colourenvelope`／`cane_victorian_scaleenvelope`，与 [[reference/prefabs/cane_victorian_fx]] 同名但关键帧数据完全不同（暗色 vs 黄色闪烁） | 两厂 envelope 名冲突；同名重复注册的引擎侧行为未核验 | [本厂 envelope](../../../raw/current/scripts/prefabs/pocketwatch_weapon_fx.lua#L9-L10)、[cane 厂 envelope](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L5-L6) |
| 同名 bank 辨析 | fx.lua 另有一组以 pocketwatch_weapon_fx 为 bank／build 的动画 FX（wanda_attack_pocketwatch_*），是攻击动画表现，与本粒子 Prefab 无关 | 勿混用注册名与 bank 名 | [同名 bank](../../../raw/current/scripts/fx.lua#L2534-L2563) |

## 典型接入

### 复用环绕微粒或调整速率

原版链路：武器 prefabs 声明 → 装备时 TryStartFx 生成挂载 → 燃料与装备事件控制生死。模组必须在首个发射回调前 SetParent 到有 AnimState 的有效宿主，并按需添加 Follower＋FollowSymbol；否则回调读 owner.age_state 会报错。满足前提后可复用发射逻辑，但速率档位读的是 `owner.age_state` 字段——挂在非旺达实体上永远走最低档 1 粒/秒；想要三档表现需自己给实体设该字段。发射在攻击时会自动停，不需要调用方额外门禁。[速率字段](../../../raw/current/scripts/prefabs/pocketwatch_weapon_fx.lua#L135-L143)

## 常见陷阱与限制

- **误区：微粒常转不停**：安全前提是存在有效父级 owner；虽然外层条件允许 owner=nil，随后仍直接读 owner.age_state，无父级会报错。有效宿主“不在攻击”（attack 标签＋三个攻击动画判断）时才发射，攻击期间完全停发；平时速率也随移动从低档向高档插值，静止时最低。[条件与插值](../../../raw/current/scripts/prefabs/pocketwatch_weapon_fx.lua#L130-L147)
- **误区：FX 代表武器伤害状态**：FX 只是充能态的伴随视觉；伤害与 shadow_item 标签由武器 fueled 事件独立设置，两者同源（同一事件分支）但互不读写。[同源分支](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L84-L94)
- **边界：envelope 与 cane_victorian 厂重名**：两文件在同一次客户端会话都可能注册同名 envelope 且数据完全不同，后注册者行为属引擎侧，本页未核验；复用本厂配置时应整体改名。[重名](../../../raw/current/scripts/prefabs/pocketwatch_weapon_fx.lua#L9-L10)
- **边界：age_state 只在旺达实体上**：该字段由 wanda.lua 维护，其他角色或非玩家实体没有；照抄三档速率表对其他角色无效（恒 1/1）。[字段来源](../../../raw/current/scripts/prefabs/wanda.lua#L166-L199)
- **边界：晚加入与重载时序未核验**：装备状态下重载或晚加入时 TryStartFx 是否补挂 FX，本页未核验。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；完整 158 行工厂（文件末尾注册名核实为 pocketwatch_weapon_fx）、宿主 pocketwatch_weapon.lua 全文（装备／模型／燃料三路径与充能分支）、wanda.lua 年龄阶段、stategraph 标签映射与 SGwilson 攻击状态、cane_victorian envelope 名比对、fx.lua 同名 bank 辨析、译名已核验。未覆盖：贴图内容未解包、移动／攻击／加燃料实测、晚加入表现、重复注册 envelope 的引擎行为。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
