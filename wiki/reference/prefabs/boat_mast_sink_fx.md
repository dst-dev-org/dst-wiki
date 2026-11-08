---
title: "船件沉没与受击特效（boat_mast_sink_fx 族）"
created: "2026-11-01"
updated: "2026-11-08"
type: prefab
summary: "船件损坏表现八注册：桅杆沉没动画三名随宿主 mast 组件 sink_fx 字段选名、防撞条受击四名由 SGboatbumper 按 bumper 类型动态拼名、小龙舟灯燃料耗尽或烧毁时气球离场一名；全部为通用表落点式纯表现。"
aliases: ["boat_mast_sink_fx", "boat_malbatross_mast_sink_fx", "mast_yotd_sink_fx", "boat_bumper_hit_kelp", "boat_bumper_hit_shell", "boat_bumper_hit_crabking", "boat_bumper_hit_yotd", "miniboatlantern_loseballoon", "桅杆沉没特效", "防撞条受击特效", "气球丢失特效", "船件损坏特效族"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["fx.lua", "prefabs/fx.lua", "entityscript.lua", "physics.lua", "gamelogic.lua", "tuning.lua", "prefabs/mast.lua", "components/mast.lua", "prefabs/mast_broken.lua", "prefabs/boat_bumpers.lua", "stategraphs/SGboatbumper.lua", "components/hullhealth.lua", "components/walkableplatform.lua", "components/workable.lua", "stategraphs/SGboat.lua", "prefabs/boat.lua", "prefabs/oceanwhirlbigportal.lua", "prefabs/miniboatlantern.lua", "map/layouts.lua", "map/static_layouts/abandonedboat.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 船件沉没与受击特效（boat_mast_sink_fx 族）

## 快速判断

- 职责与适用：`fx.lua` 通用注册表中船件损坏相关的八名纯表现实体，分三个互不相同的消费系统——桅杆沉没（3 名）、防撞条受击（4 名，按 bumper 类型动态拼名）、小龙舟灯气球丢失（1 名）；无组件、无玩法作用，代理→一帧后本地表现→animover 自删的公共协议见 [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂·关键流程]]。
- 优先使用：查"船沉时桅杆怎么表现／防撞条撞一下出什么／灯笼气球怎么飞走"读本页；新增或修改注册条目走 [[reference/mechanisms/generic-fx-factory#典型接入|通用 FX 工厂·典型接入]]；船体移动航迹水花另见 [[reference/prefabs/boat_water_fx|boat_water_fx]]（非本页协议）。
- 重要副作用：八名全部落点式（生成方只 `SetPosition`，不父挂代理），位置一次性；防撞条四名带撞击音效（通用表 `sound` 字段，各端本地播放），其余四名无音效。[协议](../../../raw/current/scripts/prefabs/fx.lua#L37-L61)
- 修改边界：生成点全在主模拟（服务端）——桅杆与灯笼在组件／prefab 回调，防撞条在 SG；专用服务器只有隐形代理。防撞条拼名键 `sg.mem.bumpertype` 是 SG 内部记忆，不作为公开接口；桅杆选名键 `mast.sink_fx` 是组件公开字段。[分支](../../../raw/current/scripts/prefabs/fx.lua#L127-L136)、[选名](../../../raw/current/scripts/prefabs/mast.lua#L427-L429)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `boat_mast_sink_fx` | 通用表条目 → 加载器代理＋一帧后本地实体 | bank `mast_01`、build `boat_mast2_wip`、anim `sink`；无 fn／sound。宿主 `mast`。[条目](../../../raw/current/scripts/fx.lua#L1843-L1847)、[选名](../../../raw/current/scripts/prefabs/mast.lua#L427-L429) |
| `boat_malbatross_mast_sink_fx` | 同上协议 | bank `mast_malbatross`、build `boat_mast_malbatross_build`、anim `sink`。宿主 `mast_malbatross`。[条目](../../../raw/current/scripts/fx.lua#L1849-L1853)、[选名](../../../raw/current/scripts/prefabs/mast.lua#L464-L466) |
| `mast_yotd_sink_fx` | 同上协议 | bank `mast_01`、build `yotd_boat_mast`、anim `sink`。宿主 `mast_yotd`。[条目](../../../raw/current/scripts/fx.lua#L3280-L3284)、[选名](../../../raw/current/scripts/prefabs/mast.lua#L492-L494) |
| `boat_bumper_hit_kelp` | 同上协议 | bank `boat_bumper`、build `boat_bumper`、anim `fx_kelp`、sound `dontstarve/characters/woodie/moose/hit`。kelp 防撞条专用。[条目](../../../raw/current/scripts/fx.lua#L2801-L2806)、[拼名](../../../raw/current/scripts/stategraphs/SGboatbumper.lua#L5-L10) |
| `boat_bumper_hit_shell` | 同上协议 | build `boat_bumper_shell`、anim `fx_shell`，音效同上。shell 防撞条专用。[条目](../../../raw/current/scripts/fx.lua#L2808-L2813) |
| `boat_bumper_hit_crabking` | 同上协议 | build `boat_bumper_crabking`、anim `fx_shell`（与 shell 同动画、换 build），音效同上。crabking 防撞条专用。[条目](../../../raw/current/scripts/fx.lua#L2815-L2820) |
| `boat_bumper_hit_yotd` | 同上协议 | build `boat_bumper_yotd`、anim `fx_kelp`（与 kelp 同动画、换 build），音效同上。yotd 防撞条专用。[条目](../../../raw/current/scripts/fx.lua#L3286-L3291) |
| `miniboatlantern_loseballoon` | 同上协议 | bank `lantern_boat`、build `yotc_lantern_boat`、anim `balloon_fly`；fn 设 `SetSixFaced`（与宿主灯笼的六面朝向一致，复制来的旋转才能正确呈现）。[条目](../../../raw/current/scripts/fx.lua#L2146-L2153)、[宿主朝向](../../../raw/current/scripts/prefabs/miniboatlantern.lua#L243-L247) |

简中 `languages/chinese_s.po` 定向检索这 8 个注册名均无显示名键；标题与别名中文为已核验效果描述，非官方译名。

## 关联机制导航

### 桅杆沉没三名：随宿主销毁的选名与沉船链

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：mast（选名字段） | 三个宿主 prefab 在 fn 内写 `inst.components.mast.sink_fx = "<注册名>"`；组件 `OnRemoveEntity` 按该字段选名 | 服务端组件回调；普通／马瘤鲣／龙舟桅杆三名各归一宿主 | [[reference/components/mast]]；[OnRemoveEntity](../../../raw/current/scripts/components/mast.lua#L59-L74) |
| 生成条件 | `sink_fx ~= nil` 且（`boat_death` 或 `boat ~= nil`）才出沉没动画，否则 `collapse_small`；落点＝桅杆世界坐标 | `boat_death` 由 `Mast:OnDeath`（监听船 `death` 事件）置位并解绑 | [条件](../../../raw/current/scripts/components/mast.lua#L59-L74)、[OnDeath](../../../raw/current/scripts/components/mast.lua#L141-L147)、[death 监听](../../../raw/current/scripts/components/mast.lua#L119-L127) |
| 触发链：船沉没 | 船 `death` 事件 → SGboat `ready_to_snap`→`snapping`→`popping` 中 `inst:Remove()`（先派发 `onremove` 清 `mast.boat`，再走组件 `OnRemoveEntity`）→ walkableplatform `OnRemoveEntity` → `DestroyObjectsOnPlatform` 以船为 destroyer `DestroyEntity(桅杆)` → `Workable:Destroy`→`WorkedBy(船)`→onfinish `on_hammered`（掉落＋`collapse_small`；玩家锤拆在此 `SetBoat(nil)`；随船销毁时 `mast.boat` 已被 `onremove` 清空，或由 `Mast:OnDeath` 解绑，该调用为空操作）→ `inst:Remove()` → 组件回调按 `sink_fx` 与 `boat_death` 补出表现 | 服务端；掉落与坍塌烟在前、沉没动画在后是同一销毁链的两段 | [popping](../../../raw/current/scripts/stategraphs/SGboat.lua#L121-L132)、[销毁清单](../../../raw/current/scripts/components/walkableplatform.lua#L80-L98)、[DestroyEntity](../../../raw/current/scripts/physics.lua#L177-L220)、[Workable 链](../../../raw/current/scripts/components/workable.lua#L73-L165)、[Remove 回调](../../../raw/current/scripts/entityscript.lua#L1730-L1750) |
| 触发链：快速拆船 | `InstantlyBreakBoat` 不推 `death`，直接 `inst:Remove()`；移除序先推 `onremove` 清 `mast.boat`，再走平台销毁。此前未置 `boat_death` 的桅杆两条件都不成立，改出 `collapse_small`；只有此前已经 `death` 置位 `boat_death`（如漩涡大传送门先扣血致死）才仍出沉没动画 | 蟹王击破 leakproof 平台不置 `boat_death`；漩涡大传送门在扣血致死后调用 | [InstantlyBreakBoat](../../../raw/current/scripts/prefabs/boat.lua#L463-L480)、[移除序](../../../raw/current/scripts/entityscript.lua#L1730-L1750)、[漩涡](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L235-L240) |
| 复用：mast_broken | `OnHammered` 在 worker 带 `boat` 标签时改出 `boat_mast_sink_fx`（非 burnt）；玩家锤是 `collapse_small`＋木材材质 | worker 为船的路径即上述平台销毁链；mast_broken 由 AbandonedBoat1 静态布局放置在带伤 boat 上（AbandonedBoat2 的 mast_area 为空） | [OnHammered](../../../raw/current/scripts/prefabs/mast_broken.lua#L15-L31)、[布局](../../../raw/current/scripts/map/layouts.lua#L1126-L1152)、[布局数据](../../../raw/current/scripts/map/static_layouts/abandonedboat.lua#L56-L77) |

### 防撞条受击四名：SGboatbumper 动态拼名与触发窗口

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 拼名与生成 | `PlayHitFX`：`SpawnPrefab("boat_bumper_hit_" .. inst.sg.mem.bumpertype)`，落点＝防撞条世界坐标 | `bumpertype` 在 SG 安装时写入＝bumper 类型名（kelp/shell/yotd/crabking，四型数据表），与四注册一一对应 | [PlayHitFX](../../../raw/current/scripts/stategraphs/SGboatbumper.lua#L5-L10)、[写入](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L301-L302)、[四型数据](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L342-L386) |
| 触发窗口 1：hit 状态 | onenter 即调 `PlayHitFX` | 入口 `onhit` 有两道守卫（busy 标签直接返回、health>0）；来源为 `boatcollision` 事件与 HAMMER 每次受击 | [hit](../../../raw/current/scripts/stategraphs/SGboatbumper.lua#L40-L54)、[onhit](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L198-L208)、[事件监听](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L269-L271)、[workable 接线](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L293-L297) |
| `boatcollision` 来源 | hullhealth 碰撞吸收：撞击法向分量（速度×法向重叠比）超过 `TUNING.BOAT.OARS.MALBATROSS.FORCE`（0.8）且命中点被 bumper 覆盖时，bumper 吃伤害并收此事件 | 服务端碰撞处理；蟹王 bumper（`collision_world_safe`）对撞世界连伤害都免 | [碰撞分支](../../../raw/current/scripts/components/hullhealth.lua#L168-L184)、[阈值](../../../raw/current/scripts/tuning.lua#L4214-L4220)、[[reference/components/hullhealth]]、[[reference/components/boatring]] |
| 触发窗口 2：changegrade 降级 | health.ondelta 跨越 0.67/0.33 阈值进入 changegrade，仅非升级（降级）分支调 `PlayHitFX`；新值 ≤0 先转 `death` | 升级分支不调；`death` 状态自身也不调（致死碰撞可能已在 `hit` 调过） | [changegrade](../../../raw/current/scripts/stategraphs/SGboatbumper.lua#L56-L89)、[ondelta](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L20-L37)、[阈值表](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L4-L9)、[death](../../../raw/current/scripts/stategraphs/SGboatbumper.lua#L91-L107) |

防撞条本体（`boat_bumper_*`）与 SGboatbumper 无 Wiki 页（Wiki 待建），源码入口 prefabs/boat_bumpers.lua、stategraphs/SGboatbumper.lua。

### miniboatlantern_loseballoon：燃料耗尽／烧毁时气球离场

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成点 | `SpawnBalloonFX`：落点＝灯笼位置并复制灯笼旋转；`POPULATING`（世界生成填充期标志）为真时跳过生成，但藏符号层照做 | 服务端；正常读档期间 nofuel 复原状态但不生成气球 | [SpawnBalloonFX](../../../raw/current/scripts/prefabs/miniboatlantern.lua#L104-L115)、[POPULATING](../../../raw/current/scripts/gamelogic.lua#L358-L360) |
| 触发 1：燃料耗尽 | `nofuel`（`fueled:SetDepletedFn`）出 FX＋隐藏灯笼气球符号层＋TurnOff＋停自燃计时，若在烧则点燃；存档标记无燃料时读档重走 nofuel，仍受 `POPULATING` 抑制 | 服务端 fueled 耗尽回调；运行时耗尽才出气球 | [nofuel](../../../raw/current/scripts/prefabs/miniboatlantern.lua#L117-L124)、[接线](../../../raw/current/scripts/prefabs/miniboatlantern.lua#L275-L282)、[OnLoad](../../../raw/current/scripts/prefabs/miniboatlantern.lua#L195-L201) |
| 触发 2：烧毁／灭火 | `onburnt` 时若仍有燃料出 FX（不藏符号层）；`onextinguish` 复用 OnBurnt 后移除本体 | 服务端事件监听 | [OnBurnt／OnExtinguish](../../../raw/current/scripts/prefabs/miniboatlantern.lua#L168-L178)、[监听](../../../raw/current/scripts/prefabs/miniboatlantern.lua#L305-L308) |

宿主 `miniboatlantern`（小龙舟灯）无 Wiki 页（Wiki 待建），源码入口 prefabs/miniboatlantern.lua。

## 典型接入

### 给自定义桅杆接沉没表现

- 前提与选择：目标是已 `SetPristine` 的自定义 prefab 服务端 fn。`sink_fx` 是 mast 组件的公开字段（原版三宿主同款写法），指向任意通用表注册名；新注册名按 [[reference/mechanisms/generic-fx-factory#典型接入|通用 FX 工厂·典型接入]] 添加。
- 生效与撤销：正常船沉（`boat_death` 已置位）时随船销毁由组件回调生成；快速拆船只有在此前已置 `boat_death` 时生成，单纯 `InstantlyBreakBoat` 会因 `onremove` 先清 `mast.boat` 而走 `collapse_small`；玩家平时锤拆同样走 `collapse_small`。
- 原版依据：[三宿主选名](../../../raw/current/scripts/prefabs/mast.lua#L427-L429)、[选名消费](../../../raw/current/scripts/components/mast.lua#L59-L74)。

```lua
inst:AddComponent("mast")
inst.components.mast.sink_fx = "boat_mast_sink_fx" -- 或指向自有通用表注册名
```

- 调整与验证：字段值须是已注册 Prefab 名；防撞条一族的选名走 SG 内部 `sg.mem.bumpertype` 拼名，不属于组件公开接口，不应在模组里依赖其记忆时序。

## 常见陷阱与限制

- **误区：玩家锤拆桅杆会播沉没动画**：`on_hammered` 在 hammerer 非绑定船时先 `SetBoat(nil)` 再 Remove，组件条件不满足 → `collapse_small`。沉没动画只在 `boat_death` 已置位时出现：正常船沉，或快速拆船前已被 `death`（如漩涡大传送门扣血致死）置位；单纯 `InstantlyBreakBoat` 与玩家锤拆都走 `collapse_small`。[on_hammered](../../../raw/current/scripts/prefabs/mast.lua#L73-L85)、[条件](../../../raw/current/scripts/components/mast.lua#L59-L74)、[快速拆船](../../../raw/current/scripts/prefabs/boat.lua#L463-L480)
- **边界：船沉时桅杆是两段表现**：销毁链先把桅杆当可工作物"锤毁"（掉落＋`collapse_small`），Remove 后组件回调再补沉没动画；叠加是原版行为。烧毁路径更早退出——`onburnt` 先 `RemoveComponent("mast")`（只走 `OnRemoveFromEntity`，不出 FX），此后该桅杆即使随船沉没也只有 `collapse_small`。[RemoveComponent 语义](../../../raw/current/scripts/entityscript.lua#L670-L684)、[onburnt](../../../raw/current/scripts/prefabs/mast.lua#L234-L265)
- **边界：防撞条不是每次受击都出 FX**：busy 期间（place/hit/changegrade 动画中）锤击与撞击被 `onhit` 守卫拦下（该守卫只管 `onhit` 入口，降级走 `onhealthchange` 不受其限）；`death` 状态自身不额外生成，但致死碰撞仍可能先在 `hit` 生成（hullhealth 先推 `boatcollision`、后扣血）；升级换级也不出，只有降级出。[守卫](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L198-L208)、[碰撞顺序](../../../raw/current/scripts/components/hullhealth.lua#L168-L183)
- **边界：mast_broken 沉没分支不是玩家可见的常规锤毁**：分支条件是 worker 带 `boat` 标签，原版只有平台销毁链以船为 destroyer 时成立（AbandonedBoat1 残骸船沉没）；玩家锤是 `collapse_small`＋木材材质。[OnHammered](../../../raw/current/scripts/prefabs/mast_broken.lua#L15-L31)
- **边界：气球丢失要求烧毁时仍有燃料**：`OnBurnt` 内 `HasFuel` 条件不满足（燃料先耗尽的灯笼再烧）不出 FX；该名无音效。[OnBurnt](../../../raw/current/scripts/prefabs/miniboatlantern.lua#L168-L173)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。八条目 bank/build/anim/sound/fn 逐项核对（fx.lua 四处区间）；三个消费系统全链核验——mast 组件 `OnRemoveEntity`/`OnDeath` 与三宿主选名、mast_broken `OnHammered`、SGboatbumper 全状态与 boat_bumpers 的 onhit/ondelta/四型数据、hullhealth 碰撞吸收分支（先推 `boatcollision` 后扣血）、miniboatlantern 三触发与 POPULATING 读档抑制、沉船销毁链（SGboat popping → walkableplatform → DestroyEntity → Workable:Destroy → onfinish → Remove → 组件回调），以及 `EntityScript:Remove` 的 `onremove` 先于组件 `OnRemoveEntity` 导致快速拆船清空 `mast.boat` 的边界。生成点反查：8 个注册名（含 `boat_bumper_hit_` 拼名前缀）在全 scripts 根（根级＋prefabs/＋components/＋stategraphs/）`rg -n` 字面检索，命中仅本页所列；`boat_bumper_hit` 前缀拼接点仅 SGboatbumper 一处；`mast_broken` 生成点仅 AbandonedBoat1 布局一处。动画时长与循环性、专服／多人表现未核验。
- 游戏运行：未运行；沉没两段表现的实际观感、防撞条音效与气球旋转表现未实测。
- 结构检查：validate_page 与 validate_source_links --strict 通过；Lua 示例经 parser_probe 语法检查。

返回 [[reference/prefabs/fx-water-index|水面、航海与船件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-06）。
