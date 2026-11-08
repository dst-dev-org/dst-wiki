---
title: 龙蝇火系与暗影斩击特效（firesplash_fx 等）
created: 2026-10-31
updated: 2026-11-08
type: prefab
summary: 熔炉（The Forge）资产出身、本体战斗复用的 6 个通用表 FX：龙蝇重击火环、三状态火焰（吐息代理由状态 onexit 移除）、麦斯威尔暗影突袭斩击二名随机选一；表现与伤害判定分属不同实体。
aliases: ["firesplash_fx", "tauntfire_fx", "attackfire_fx", "vomitfire_fx", "shadowstrike_slash_fx", "shadowstrike_slash2_fx", "龙蝇", "龙蝇火焰特效", "蜻蜓龙", "蜻蜓龙火焰特效", "重击火环", "暗影斩击", "影袭斩击", "暗影突袭特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["fx.lua", "stategraphs/SGdragonfly.lua", "components/groundpounder.lua", "prefabs/dragonfly.lua", "stategraphs/SGshadowwaxwell.lua", "prefabs/shadowwaxwell.lua", "prefabs/waxwell_shadowstriker.lua", "prefabs/willow_ember.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 龙蝇火系与暗影斩击特效（firesplash_fx 等）

## 快速判断

- 职责与适用：`fx.lua` 通用注册表中 6 个战斗表现特效，按宿主分两组：龙蝇（Dragonfly，检索别名蜻蜓龙）火系 4 名（firesplash／tauntfire／attackfire／vomitfire）与麦斯威尔暗影召唤物斩击 2 名（shadowstrike_slash／slash2）。生成链均在树内核验。公共工厂协议（代理＋本地实体、回收边界）只在 [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂·关键流程]] 维护。
- 优先使用：查"龙蝇火焰表现／地面重击火环／暗影突袭斩击"选本页。注意与任务直觉相反：**本组树内生成点全部是本体战斗内容**（沙漠首领龙蝇＋麦斯威尔召唤物），不是熔炉模式敌人；熔炉事件数据包内是否复用不可见。
- 重要副作用：全部为纯表现——无伤害、点燃或照明字段；攻击结算、伤害倍率与点燃都在生成者的 combat／SG 逻辑内，FX 只贴位播放。[如 shadowstrike 伤害链](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L1315-L1322)
- 修改边界：FX 条目是通用表数据，改表现走 fx.lua 注册字段；改触发时机走对应 SG 状态或组件配置（见关联导航）；两组的音效均由生成方播放，注册条目无 sound 字段。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `firesplash_fx` | 通用表条目 → prefabs/fx.lua `MakeFx`，无 fn | bank/build `dragonfly_ground_fx`，anim `idle`，bloom；地面重击的环状火溅。[注册](../../../raw/current/scripts/fx.lua#L944-L948) |
| `tauntfire_fx` | 同上 | bank/build `dragonfly_fx`，anim `taunt`，bloom。[注册](../../../raw/current/scripts/fx.lua#L951-L955) |
| `attackfire_fx` | 同上 | bank/build `dragonfly_fx`，anim `atk`，bloom。[注册](../../../raw/current/scripts/fx.lua#L958-L962) |
| `vomitfire_fx` | 同上 | bank/build `dragonfly_fx`，anim `vomit`，twofaced，bloom。[注册](../../../raw/current/scripts/fx.lua#L965-L970) |
| `shadowstrike_slash_fx` | 同上；fn=`FinalOffset1`（`SetFinalOffset(1)`，[定义](../../../raw/current/scripts/fx.lua#L1-L3)） | bank/build `lavaarena_shadow_lunge_fx`，anim `line`（直线斩），`transform` 字段做 1.25 倍三轴缩放，eightfaced。[注册](../../../raw/current/scripts/fx.lua#L926-L932) |
| `shadowstrike_slash2_fx` | 同上，与 `_fx` 仅 anim 不同 | 同 bank/build 与缩放，anim `curve`（弧线斩）。[注册](../../../raw/current/scripts/fx.lua#L935-L941) |

六个注册名均无 `nameoverride`／NAMES 条目，`languages/chinese_s.po` 定向检索无对应显示名键；宿主 `DRAGONFLY` 的官方译名为「龙蝇」（[po](../../../raw/current/scripts/languages/chinese_s.po#L342199-L342202)），`WAXWELL_SHADOWSTRIKER` 键译作「暗影」（[po](../../../raw/current/scripts/languages/chinese_s.po#L351914-L351917)）。标题用官方宿主名「龙蝇」，「蜻蜓龙」仅保留为检索别名；六个 FX 本身仍是已核验效果描述，非官方译名。`tauntfire`／`attackfire` 等命名沿用龙蝇动画段名，不自动代表嘲讽／攻击玩法归属。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| SG：SGdragonfly attack | `attack` 状态 onenter 生成 `attackfire_fx` 于龙当前位置并复制旋转 | 仅 `inst.enraged`（激怒期）时生成；落点式 | [生成点](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L462-L467) |
| SG：SGdragonfly pound | `pound` 状态（重击循环的 taunt 动画段）onenter 生成 `tauntfire_fx` 同位同转 | 进入 pound 前摇即出，非伤害帧；重击伤害由后续 `groundpounder:GroundPound()` 结算 | [生成点](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L598-L603)、[GroundPound 调用](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L621) |
| SG：SGdragonfly lavae | `lavae` 状态（吐熔岩虫）onenter 生成 `vomitfire_fx` 同位同转并留存代理 `inst.vomitfx` | 状态 onexit 移除代理并清引用；已创建的本地动画按自身生命周期结束；退出时恢复六面体 | [生成](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L669-L671)、[onexit](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L699-L706) |
| 组件：groundpounder | 龙蝇配置 `groundpoundfx = "firesplash_fx"`；环模式下 `GroundPound` 逐环逐点生成，落点 y=0 | `fxRings=2`，半径 3／5、密度 0.25；首环算出 4 点但被收为中心 1 点，次环 7 点，单次最多 8 枚；陆地非码头门禁可减少数量 | [配置](../../../raw/current/scripts/prefabs/dragonfly.lua#L537-L549)、[点数与首环](../../../raw/current/scripts/components/groundpounder.lua#L37-L67)、[生成门禁](../../../raw/current/scripts/components/groundpounder.lua#L161-L163) |
| SG：SGshadowwaxwell lunge | `doattack` 且 `combat.attackrange == 5` 时进 `lunge_pre`→`lunge_loop`；onupdate 近身（IsNear 1）时随机二选一生成斩击于目标位置上方 1.5，复制施放者旋转，随后 `DoAttack` 结算（带 1.5 倍伤害修正并 DropAggro） | 每次突袭一枚（命中分支置 `attackdone` 后不重复）；`attackrange=5` 由 SG `idle`／`ready_pre` 在存在 timer 且无 `shadowstrike_cd` 时 `SetRange(5)`；`shadowprotector`（protectorfn）与 `shadowworker`（workerfn）都经 `MakeOblivionSeeker` 安装 timer，故都满足该前提，但 timer 只是范围门禁，不能据此断言它必然突袭 | [进入条件](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L250-L256)、[范围赋值](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L339-L362)、[生成与结算](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L1315-L1322)、[protectorfn](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L322-L340)、[workerfn](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L357-L367) |
| 关联 Prefab：shadowwaxwell 系列／waxwell_shadowstriker | `MakeMinion` 系列（shadowworker、shadowprotector、shadowduelist、shadowlumber、shadowminer）挂 SGshadowwaxwell；shadowprotector 额外注入 `lavaarena_shadow_lunge` 资产与两斩击 FX 依赖；`waxwell_shadowstriker` 依赖声明两 FX，master_postinit 在事件数据包 | 树内未见 `waxwell_shadowstriker` 的生成调用（依赖声明不等于生成）；`shadowprotector` 与 `shadowworker` 的专属初始化都安装 timer，因而具备 lunge 范围门禁；timer 只说明前提，不等于必然进入 lunge，其他变体不据此推断 | [MakeMinion](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L439-L461)、[SG 挂载](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L557)、[protectorfn](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L322-L340)、[workerfn](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L357-L367)、[返回表](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L668-L675)、[shadowstriker 接线](../../../raw/current/scripts/prefabs/waxwell_shadowstriker.lua#L53) |
| 关联 Prefab：willow_ember | `spawnfirefx(pos)` 内落点生成 `firesplash_fx`＋`firering_fx` 四点火环 | **树内无调用点**：`spawnfirefx` 为 local 函数，全 scripts 树 rg 仅命中定义行，属未接线的死代码路径，不能当活跃生成点引用 | [定义](../../../raw/current/scripts/prefabs/willow_ember.lua#L260-L274) |
| 资源 | anim `dragonfly_ground_fx.zip`、`dragonfly_fx.zip`、`lavaarena_shadow_lunge_fx.zip`（bank/build 引用即用） | 注册字段已核验引用关系，anim 包内容未解包 | [注册行](../../../raw/current/scripts/fx.lua#L926-L970) |

## 典型接入

### 借用 groundpounder 组件复用"重击火环"

- 前提与选择：在自建生物的主模拟初始化或服务端回调执行；宿主须已有 `combat`，默认伤害环会访问它。以下片段还会结算伤害和破坏；FX 名可换成任意已注册落点特效。
- 生效与撤销：`GroundPound()` 调用时逐环点生成并结算伤害／破坏；FX 无自持玩法作用，寿命走通用工厂 animover 自删。
- 原版依据：龙蝇同款配置见 [dragonfly.lua](../../../raw/current/scripts/prefabs/dragonfly.lua#L537-L549)；组件字段消费见 [groundpounder.lua](../../../raw/current/scripts/components/groundpounder.lua#L161-L163)。

```lua
-- 服务端生物初始化／回调片段；inst 已有 combat。
assert(TheWorld.ismastersim and inst.components.combat ~= nil)
inst:AddComponent("groundpounder")
inst.components.groundpounder:UseRingMode()
inst.components.groundpounder.numRings = 3
inst.components.groundpounder.fxRings = 2
inst.components.groundpounder.groundpoundfx = "firesplash_fx"
inst.components.groundpounder:GroundPound()
```

- 调整与验证：`groundpoundfx` 缺省为 `groundpound_fx`（组件 L20），漏配会得到默认尘土环；环点仅在陆地非码头格出 FX。

## 常见陷阱与限制

- **误区：注册名代表玩法归属**：`tauntfire_fx`／`attackfire_fx`／`vomitfire_fx` 是龙蝇 taunt/atk/vomit 动画段的伴生火焰，嘲讽、仇恨与攻击判定都不在 FX 内；FX 无 combat、无 burner，点燃玩家由龙蝇攻击逻辑另行结算。[生成点](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L462-L467)
- **边界：attackfire 仅激怒期**：`enraged` 为假时普攻不出火焰；把 FX 当普攻恒定表现会误判普通阶段画面。[条件](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L463-L467)
- **边界：代理与本地动画分别清理**：`inst.vomitfx` 是网络代理，状态 onexit 移除该代理并清引用；已经创建的本地动画按自身 animover 清理，不能把二者当同一实体。代理若在延迟创建任务前移除，本地表现可能未创建。[onexit](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L699-L706)、[[reference/mechanisms/generic-fx-factory#关键流程|工厂生命周期]]
- **边界：firesplash 的地面门禁**：环生成前查 `IsLandTileAtPoint` 且排除码头，落点固定 y=0；海上重击不出火环（伤害结算仍可另行处理）。[门禁](../../../raw/current/scripts/components/groundpounder.lua#L161-L163)
- **边界：shadowstrike 斩击的宿主与接线缺口**：树内生成点只有 SGshadowwaxwell lunge 链；`attackrange=5` 的赋值本身在 SG `idle`／`ready_pre` 内（存在 timer 且无 `shadowstrike_cd` 时 `combat:SetRange(5)`），不是事件包缺口；`shadowprotector` 与 `shadowworker` 都经 `protectorfn`／`workerfn`→`MakeOblivionSeeker` 安装 timer，是可见的 timer 宿主，但 timer 只是范围门禁，不能据此断言 worker 必然突袭。仅 `waxwell_shadowstriker` 的实际召唤阵容／事件包内接线未核验；其他变体不据此推断。[范围赋值](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L339-L362)、[protectorfn](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L322-L340)、[workerfn](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L357-L367)、[MakeOblivionSeeker](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L122-L128)、[MakeMinion 返回表](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L668-L675)、[接线](../../../raw/current/scripts/prefabs/waxwell_shadowstriker.lua#L53)
- **边界：willow_ember 路径是死代码**：`spawnfirefx` 全树无调用（已对标识符全域检索），引用其行为前先确认自己的快照版本；不要把"willow 手雷火环"当作本 FX 的活跃消费者。[定义](../../../raw/current/scripts/prefabs/willow_ember.lua#L260-L274)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。fx.lua 6 条注册逐字段核对；生成点全部读到实际 SpawnPrefab 行（SGdragonfly 三状态、groundpounder 环逐点生成、SGshadowwaxwell lunge、willow_ember 死代码定义）；第 5 轮复核补：`attackrange=5` 由 SGshadowwaxwell `idle`／`ready_pre` 在 timer 存在且无 `shadowstrike_cd` 时设置；`shadowprotector` 与 `shadowworker` 的专属初始化都安装 timer，因而具备该门禁；有 timer 只是前提，不能据此断言必然突袭。仅 `waxwell_shadowstriker` 的事件包内实际阵容未核验。对 6 个注册名在 scripts 根全域（prefabs/components/stategraphs/brains）rg 反查，除上列消费与依赖声明外无其他命中；`spawnfirefx` 标识符全树检索确认无调用。事件数据包内接线、anim zip 内容未核验。
- 游戏运行：未运行；validation 保持 source-reviewed。
- 结构检查：validate_page、validate_source_links --strict 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
