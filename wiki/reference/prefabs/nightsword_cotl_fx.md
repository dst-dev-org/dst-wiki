---
title: "咩咩之刃攻击烟尘特效（nightsword_cotl_fx）"
created: "2026-10-29"
updated: "2026-11-08"
type: prefab
summary: "暗夜剑咩咩之刃皮肤的挥击烟尘爆发：装备时挂在持剑者身上跟随武器符号，攻击动画 0.13 秒处一次性喷出 15 粒烟与 3 粒余烬。"
aliases: ["nightsword_cotl_fx", "咩咩之刃特效", "咩咩之刃", "暗夜剑皮肤特效", "CotL暗夜剑"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/nightsword_cotl_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "constants.lua", "emitters.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 咩咩之刃攻击烟尘特效（nightsword_cotl_fx）

## 快速判断

- 职责与适用：暗夜剑（Dark Sword）皮肤“咩咩之刃”（nightsword_cotl）的挥击粒子实体：联网 FX 壳加本地 VFXEffect，宿主装备时生成并挂到持剑者身上、跟随其武器符号；宿主攻击动画播到 0.13 秒时一次性爆发，之后静默至下次攻击。本厂无 AddComponent、无 Light／SoundEmitter，不改变武器数值。[工厂](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L113-L130)
- 优先使用：查“咩咩之刃／暗夜剑皮肤攻击特效”选本页；同族 [[reference/prefabs/nightsword_curve_fx]]／[[reference/prefabs/nightsword_lightsbane_fx]]／[[reference/prefabs/nightsword_sharp_fx]]／[[reference/prefabs/nightsword_wizard_fx]] 为同一模板改参（发射器数、纹理、数量、朝向细节不同），装备生成协议完全一致。
- 重要副作用：爆发由宿主动画门禁驱动每次攻击一次；骑乘时整段跳过发射，但判断读取宿主 rider 组件，仅主模拟有效。[门禁](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L173-L182)、[骑乘](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L171)
- 修改边界：专用服务器在 SetPristine 后提前返回，不建 VFXEffect 与发射回调；客户端各自本地发射；实体无组件、persists=false 不入存档。[端分界](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L123-L130)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `nightsword_cotl_fx` | fn：Transform＋Network＋FX 标签 → SetPristine → 专服早退 → 非专服注册 envelope 并配 VFXEffect（2 发射器）→ EmitterManager 回调 | 单一注册。烟（0 号，上限 45，寿命 0.8s）与余烬（1 号，上限 5，寿命 0.6s）都走 fx/animsmoke.tex＋reveal 着色器、AlphaBlended；envelope `nightsword_cotl_*` 仅首次创建实体时注册一次。[构造](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L113-L158)、[注册](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L238) |

皮肤 `STRINGS.SKIN_NAMES.nightsword_cotl` 官方译名“咩咩之刃”；未发现该 FX 注册名的 NAMES 翻译条目，标题中“攻击烟尘特效”为已核验效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L400730-L400732)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源皮肤 nightsword_cotl | fx_prefab 单项存入 SKIN_FX_PREFAB；皮肤 init_fn 仅主模拟读第 1 项存 inst.vfx_fx 并挂 equipped／unequipped／onremove 监听 | 监听在武器实体上；SpawnPrefab 出的联网实体复制到各端 | [皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8667-L8678)、[映射保存](../../../raw/current/scripts/prefabskin.lua#L4094-L4096)、[初始化](../../../raw/current/scripts/prefabskin.lua#L3238-L3254) |
| 宿主装备协议 | equipped 时 SpawnPrefab → SetParent(data.owner) → FollowSymbol(owner, "swap_object", 0, -100, 0)；unequipped／onremove 直接 Remove | FX 挂持剑者而非武器；再装备生成新实例 | [equipped](../../../raw/current/scripts/prefabskin.lua#L3220-L3229)、[清理](../../../raw/current/scripts/prefabskin.lua#L3231-L3236) |
| 发射驱动：EmitterManager | AddEmitter(inst, nil, fn) 以无限寿命注册，每 tick 调回调；实体移除时自动反注册 | 非专服各端独立运行；无父级时回调空转 | [AddEmitter](../../../raw/current/scripts/emitters.lua#L16-L31)、[回调](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L168-L233) |
| 攻击门禁 | 宿主 AnimState 当前动画为 "atk" 且时间 > 0.13 时 burst_state 0→1，随后 1→2 发射一次；动画不在播则复位 0 | 读宿主动画，需要父实体有 AnimState | [门禁](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L173-L182) |
| 骑乘抑制 | parent.components.rider:GetMount() 非 nil 时跳过整段发射逻辑 | rider 组件仅主模拟存在；纯客户端恒为 nil，抑制不生效 | [骑乘](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L171) |
| 朝向与偏移 | 按 GetCurrentFacing 选烟扇角度：0→270°，1→0°加 RIGHT/DOWN 相机偏移，2→90°并垂直翻转，3→180°加 DOWN 偏移，再叠加相机 heading | 常量 NIGHTSWORD_FX_OFFSETS.RIGHT=0.75、DOWN=2.9 | [朝向](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L189-L205)、[常量](../../../raw/current/scripts/constants.lua#L2944-L2947) |
| 资源 | fx/animsmoke.tex＋shaders/vfx_particle_reveal.ksh；黑烟（RGB 10,10,10）烟扇半角 70°，15 烟＋3 烬 | 声明与使用已核验，贴图内容未解包 | [资源声明](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L1-L16)、[发射器配置](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L135-L158) |

## 典型接入

### 复用挥击爆发协议

原版仅经皮肤系统接入：皮肤 fx_prefab → nightsword_init_fn 装监听 → equipped 生成并挂载跟随。模组复用两条路：做皮肤沿用该协议；或直接 SpawnPrefab 后自行 SetParent＋FollowSymbol 到持武器角色。后者必须保证父实体 AnimState 会播 "atk" 动画，否则门禁永不满足；把 FX 放地上（无父级）则回调空转、永不发射。[回调与门禁](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L169-L182)

## 常见陷阱与限制

- **误区：烟尘持续环绕武器**：回调只在宿主 "atk" 动画超过 0.13 秒的那一帧爆发一次（burst_state 1→2 后不再发），烟粒寿命最长 0.8 秒、烬 0.6 秒，其余时间完全不发射。[门禁](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L173-L185)
- **边界：骑乘抑制不分端一致**：mount 判断读 parent.components.rider，组件表仅主模拟存在——专用服务器本来不建粒子，主机模拟上骑乘时无爆发，纯客户端缺 rider 时 mount 为 nil，此门禁不拦截；仍须满足 atk 动画与时间条件，实际骑乘表现未运行核验。[骑乘](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L171)
- **边界：朝向异常只跳过不崩溃**：fdir 不在 0-3 时 smoke_angle 为 nil，整次爆发跳过并 print 错误到控制台，无保护性报错。[兜底](../../../raw/current/scripts/prefabs/nightsword_cotl_fx.lua#L228-L229)
- **边界：晚加入与重载时序未核验**：装备状态下重载或晚加入时 equipped 事件是否补挂 FX，本页未核验。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；完整 238 行工厂（文件末尾注册名核实为 nightsword_cotl_fx）、皮肤定义与 fx_prefab 映射、SKIN_FX_PREFAB 保存与 nightsword_init_fn／equipped／clear_fn、constants 偏移、emitters.lua 注册语义、官方皮肤译名已核验；五个 nightsword 工厂对照为同模板改参。未覆盖：贴图与 envelope 最终视觉、多次攻击／骑乘实测、晚加入表现。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-03）。
