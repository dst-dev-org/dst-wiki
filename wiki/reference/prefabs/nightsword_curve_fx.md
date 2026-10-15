---
title: "扭曲匕首攻击火花特效（nightsword_curve_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "暗夜剑扭曲匕首皮肤的挥击三路粒子爆发：攻击动画 0.13 秒处一次喷出 15 烟、1 闪点与 25 颗受重力下坠的火星。"
aliases: ["nightsword_curve_fx", "扭曲匕首特效", "扭曲匕首", "暗夜剑皮肤特效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/nightsword_curve_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "constants.lua", "emitters.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 扭曲匕首攻击火花特效（nightsword_curve_fx）

## 快速判断

- 职责与适用：暗夜剑皮肤“扭曲匕首”（nightsword_curve）的挥击粒子实体：联网 FX 壳加本地 VFXEffect，宿主装备时挂到持剑者身上跟随武器符号，攻击动画 0.13 秒处一次性爆发；无组件、无声音、无光照，不改变武器数值。[工厂](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L169-L186)
- 优先使用：查“扭曲匕首／暗夜剑皮肤攻击特效”选本页；同族 [[reference/prefabs/nightsword_cotl_fx]]／[[reference/prefabs/nightsword_lightsbane_fx]]／[[reference/prefabs/nightsword_sharp_fx]]／[[reference/prefabs/nightsword_wizard_fx]] 为同一模板改参，装备生成协议完全一致。
- 重要副作用：本厂是五厂中唯一给火星发射器配下坠加速度的（SetAcceleration y −0.3＋按朝向的初速度）；闪点走 additive＋bloom，烟走 reveal。[火星](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L218-L231)
- 修改边界：专用服务器提前返回不建粒子；客户端各自本地发射；骑乘抑制读取宿主 rider 组件、仅主模拟有效。[端分界](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L179-L186)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `nightsword_curve_fx` | fn：Transform＋Network＋FX 标签 → SetPristine → 专服早退 → 注册 envelope 并配 VFXEffect（3 发射器）→ EmitterManager 回调 | 单一注册。烟（0 号，上限 32，寿命 1.1s，reveal）、闪点（1 号，上限 6，寿命 0.2s，sparkle.tex＋add＋bloom＋4 帧 UV）、火星（2 号，上限 128，寿命 0.6s，spark.tex＋add＋bloom＋按速度旋转＋下坠）；envelope `nightsword_curve_*`。[构造](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L188-L231)、[注册](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L310) |

皮肤 `STRINGS.SKIN_NAMES.nightsword_curve` 官方译名“扭曲匕首”；未发现该 FX 注册名的 NAMES 翻译条目，标题中“攻击火花特效”为已核验效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L400735-L400737)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源皮肤 nightsword_curve | fx_prefab 单项存入 SKIN_FX_PREFAB；皮肤 init_fn 仅主模拟读第 1 项存 inst.vfx_fx 并挂 equipped／unequipped／onremove 监听 | 监听在武器实体上；联网实体复制到各端 | [皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8680-L8691)、[映射保存](../../../raw/current/scripts/prefabskin.lua#L4094-L4096)、[初始化](../../../raw/current/scripts/prefabskin.lua#L3238-L3254) |
| 宿主装备协议 | equipped 时 SpawnPrefab → SetParent(data.owner) → FollowSymbol(owner, "swap_object", 0, -100, 0)；unequipped／onremove 直接 Remove | FX 挂持剑者；再装备生成新实例 | [equipped](../../../raw/current/scripts/prefabskin.lua#L3220-L3229)、[清理](../../../raw/current/scripts/prefabskin.lua#L3231-L3236) |
| 发射驱动：EmitterManager | AddEmitter(inst, nil, fn) 无限寿命注册，每 tick 调回调；实体移除时自动反注册 | 非专服各端独立运行；无父级时空转 | [AddEmitter](../../../raw/current/scripts/emitters.lua#L16-L31)、[回调](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L243-L305) |
| 攻击门禁与骑乘 | 宿主 "atk" 动画时间 > 0.13 时 burst_state 0→1→2 发射一次；mount 非 nil 跳过整段 | 门禁读宿主动画；骑乘抑制仅主模拟有效 | [门禁](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L248-L258)、[骑乘](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L246) |
| 朝向与偏移 | 挥向右（facing 1）时发射点加 NIGHTSWORD_FX_OFFSETS 相机偏移；火星初速度按四朝向取相机右／下方向 ×0.35，异常朝向打印错误并跳过火星 | 常量 RIGHT=0.75、DOWN=2.9 | [偏移](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L266-L270)、[火星方向](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L282-L301)、[常量](../../../raw/current/scripts/constants.lua#L2944-L2947) |
| 资源 | fx/animsmoke.tex、fx/sparkle.tex、fx/spark.tex＋reveal／add 着色器；烟 15、闪点 1、火星 25 | 声明与使用已核验，贴图内容未解包 | [资源声明](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L1-L23)、[爆发数量](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L262-L264) |

## 典型接入

### 复用挥击爆发协议

原版仅经皮肤系统接入：皮肤 fx_prefab → nightsword_init_fn 装监听 → equipped 生成并挂载跟随。模组直接 SpawnPrefab 后自行 SetParent＋FollowSymbol 亦可，但父实体 AnimState 必须播 "atk" 动画，否则门禁永不满足；无父级则回调空转。火星方向依赖父实体 GetCurrentFacing 与相机向量，脱离角色上下文时四朝向都不匹配会整段跳过火星（仅烟与闪点照发）。[火星方向门禁](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L294-L301)

## 常见陷阱与限制

- **误区：火星向上飘**：火星发射器 SetAcceleration(0, −0.3, 0) 且初速度叠加朝向向量，是下坠四散；只有烟与闪点原地漂浮。[火星配置](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L230-L231)、[初速度](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L143-L148)
- **边界：骑乘抑制不分端一致**：组件表仅主模拟存在——主机模拟骑乘时无爆发，纯客户端缺 rider 时 mount 为 nil，此门禁不拦截；仍须满足 atk 动画与时间条件，实际骑乘表现未运行核验。[骑乘](../../../raw/current/scripts/prefabs/nightsword_curve_fx.lua#L246)
- **边界：晚加入与重载时序未核验**：装备状态下重载或晚加入时 equipped 事件是否补挂 FX，本页未核验。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；完整 310 行工厂（文件末尾注册名核实为 nightsword_curve_fx）、皮肤定义与 fx_prefab 映射、SKIN_FX_PREFAB 保存与 nightsword_init_fn／equipped／clear_fn、constants 偏移、emitters.lua 注册语义、官方皮肤译名已核验；五个 nightsword 工厂对照为同模板改参。未覆盖：贴图与 envelope 最终视觉、多次攻击／骑乘实测、晚加入表现。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
