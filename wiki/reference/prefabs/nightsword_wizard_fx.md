---
title: "诅咒之剑攻击暗影微粒特效（nightsword_wizard_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "暗夜剑诅咒之剑皮肤的挥击暗色微粒爆发：攻击动画 0.13 秒处一次喷出 5 烟与 3 颗海绵纹理暗色微粒，两路都走 reveal 着色器。"
aliases: ["nightsword_wizard_fx", "诅咒之剑特效", "诅咒之剑", "Cursed Sabre", "暗夜剑皮肤特效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/nightsword_wizard_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "constants.lua", "emitters.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 诅咒之剑攻击暗影微粒特效（nightsword_wizard_fx）

## 快速判断

- 职责与适用：暗夜剑皮肤“诅咒之剑”（nightsword_wizard）的挥击粒子实体：联网 FX 壳加本地 VFXEffect，宿主装备时挂到持剑者身上跟随武器符号，攻击动画 0.13 秒处一次性爆发；无组件、无声音、无光照，不改变武器数值。[工厂](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L115-L132)
- 优先使用：查“诅咒之剑／Cursed Sabre／暗夜剑皮肤攻击特效”选本页；同族 [[reference/prefabs/nightsword_cotl_fx]]／[[reference/prefabs/nightsword_curve_fx]]／[[reference/prefabs/nightsword_lightsbane_fx]]／[[reference/prefabs/nightsword_sharp_fx]] 为同一模板改参，装备生成协议完全一致。
- 重要副作用：1 号发射器纹理是 fx/animsponge.tex 且与烟同用 reveal 着色器、AlphaBlended——两路均未启用 additive／bloom（cotl 也未启用），与亮闪点族不同；微粒寿命 1.5 秒、阻尼 1，比烟更久。[微粒配置](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L151-L161)
- 修改边界：专用服务器提前返回不建粒子；客户端各自本地发射；骑乘抑制读取宿主 rider 组件、仅主模拟有效。[端分界](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L125-L132)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `nightsword_wizard_fx` | fn：Transform＋Network＋FX 标签 → SetPristine → 专服早退 → 注册 envelope 并配 VFXEffect（2 发射器）→ EmitterManager 回调 | 单一注册。烟（0 号，上限 32，寿命 0.75s，animsmoke.tex＋reveal）、微粒（1 号，上限 32，寿命 1.5s，animsponge.tex＋reveal＋UV(1,1)＋旋转）；envelope `nightsword_wizard_*`，微粒色标深灰（100,100,100,0）→（50,50,50,200）→（0,0,0,100），缩放恒 0.6。[构造](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L134-L161)、[注册](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L217) |

皮肤 `STRINGS.SKIN_NAMES.nightsword_wizard` 官方译名“诅咒之剑”（msgid Cursed Sabre）；未发现该 FX 注册名的 NAMES 翻译条目，标题中“攻击暗影微粒特效”为已核验效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L400750-L400752)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源皮肤 nightsword_wizard | fx_prefab 单项存入 SKIN_FX_PREFAB；皮肤 init_fn 仅主模拟读第 1 项存 inst.vfx_fx 并挂 equipped／unequipped／onremove 监听 | 监听在武器实体上；联网实体复制到各端 | [皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8719-L8730)、[映射保存](../../../raw/current/scripts/prefabskin.lua#L4094-L4096)、[初始化](../../../raw/current/scripts/prefabskin.lua#L3238-L3254) |
| 宿主装备协议 | equipped 时 SpawnPrefab → SetParent(data.owner) → FollowSymbol(owner, "swap_object", 0, -100, 0)；unequipped／onremove 直接 Remove | FX 挂持剑者；再装备生成新实例 | [equipped](../../../raw/current/scripts/prefabskin.lua#L3220-L3229)、[清理](../../../raw/current/scripts/prefabskin.lua#L3231-L3236) |
| 发射驱动：EmitterManager | AddEmitter(inst, nil, fn) 无限寿命注册，每 tick 调回调；实体移除时自动反注册 | 非专服各端独立运行；无父级时空转 | [AddEmitter](../../../raw/current/scripts/emitters.lua#L16-L31)、[回调](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L172-L212) |
| 攻击门禁与骑乘 | 宿主 "atk" 动画时间 > 0.13 时 burst_state 0→1→2 发射一次；mount 非 nil 跳过整段 | 门禁读宿主动画；骑乘抑制仅主模拟有效 | [门禁](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L177-L186)、[骑乘](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L175) |
| 朝向与偏移 | 仅挥向右（facing 1）时发射点加 NIGHTSWORD_FX_OFFSETS 相机偏移，其余朝向无特判、无错误兜底 | 常量 RIGHT=0.75、DOWN=2.9 | [偏移](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L194-L198)、[常量](../../../raw/current/scripts/constants.lua#L2944-L2947) |
| 资源 | fx/animsmoke.tex、fx/animsponge.tex＋reveal 着色器；烟 5、微粒 3 | 声明与使用已核验，贴图内容未解包 | [资源声明](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L13-L18)、[爆发数量](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L191-L192) |

## 典型接入

### 复用挥击爆发协议

原版仅经皮肤系统接入：皮肤 fx_prefab → nightsword_init_fn 装监听 → equipped 生成并挂载跟随。模组直接 SpawnPrefab 后自行 SetParent＋FollowSymbol 亦可，但父实体 AnimState 必须播 "atk" 动画；无父级则回调空转。本厂没有朝向分支与错误兜底，复用时不需要处理朝向。[门禁与偏移](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L177-L198)

## 常见陷阱与限制

- **误区：SPARK 即亮闪光点**：本厂 SPARK 用 animsponge.tex＋reveal＋AlphaBlended、深灰色标，视觉是暗色雾粒，与 curve／sharp 厂走 additive＋bloom 的亮闪点完全不同；判断表现要看着色器与色标而非变量名。[微粒配置](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L48-L64)、[发射器](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L151-L161)
- **边界：骑乘抑制不分端一致**：组件表仅主模拟存在——主机模拟骑乘时无爆发，纯客户端缺 rider 时 mount 为 nil，此门禁不拦截；仍须满足 atk 动画与时间条件，实际骑乘表现未运行核验。[骑乘](../../../raw/current/scripts/prefabs/nightsword_wizard_fx.lua#L175)
- **边界：晚加入与重载时序未核验**：装备状态下重载或晚加入时 equipped 事件是否补挂 FX，本页未核验。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；完整 217 行工厂（文件末尾注册名核实为 nightsword_wizard_fx）、皮肤定义与 fx_prefab 映射、SKIN_FX_PREFAB 保存与 nightsword_init_fn／equipped／clear_fn、constants 偏移、emitters.lua 注册语义、官方皮肤译名已核验；五个 nightsword 工厂对照为同模板改参。未覆盖：贴图与 envelope 最终视觉、多次攻击／骑乘实测、晚加入表现。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
