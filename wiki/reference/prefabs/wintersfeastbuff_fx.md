---
title: "冬季盛宴增益粒子与烤炉火焰 FX（wintersfeastbuff_fx、wintersfeastoven_fire）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "冬季盛宴两个外围 FX：冬宴餐桌进食 debuff 链按剩余时长变频投放的随机帧发光粒子（独立落点、animover 自删加休眠即删），与砖砌烤炉烹饪期父挂炉体的联网火焰层（madsciencelab 两阶段回调驱动动画、出菜后由宿主移除）。含与 fire/campfirefire 火焰体系的辨析。"
aliases: ["wintersfeastbuff_fx", "wintersfeastoven_fire", "冬季盛宴增益粒子", "冬宴buff特效", "节日精神增益", "烤炉火", "砖砌烤炉火层", "冬季盛宴烤炉"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/wintersfeastbuff.lua", "prefabs/wintersfeastoven.lua", "components/feasts.lua", "components/madsciencelab.lua", "stategraphs/SGwilson.lua", "widgets/healthbadge.lua", "widgets/hungerbadge.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 冬季盛宴增益粒子与烤炉火焰 FX（wintersfeastbuff_fx、wintersfeastoven_fire）

## 快速判断

- 职责与适用：冬季盛宴活动的 **2 个外围 FX**。①`wintersfeastbuff_fx`——冬宴餐桌进食 buff 的联网发光粒子：由 debuff 实体按剩余时长变频生成在目标**当时位置**（不父挂），`animover` 自删外加休眠即删；②`wintersfeastoven_fire`——砖砌烤炉（wintersfeastoven）烹饪期父挂炉体的联网火焰层：动画由宿主 `madsciencelab` 两阶段回调驱动，无自删，由宿主显式 Remove（[注册](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L168-L169)、[注册](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L431-L432)）。
- 重要副作用：两个 FX 自身都零玩法——数值链（回血／回饥饿／回理智、饥饿消耗清零、死亡掉落"节日欢愉"）全在 debuff 宿主 `wintersfeastbuff` 上，烤炉的灯（半径 2）也在炉体本体；火层只有 `FX`＋`DECOR` 标签的动画载体（[宿主工厂](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L103-L135)、[炉体 Light](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L327-L331)）。
- 优先使用：查"吃了宴席 buff 的粒子/生效链"走 buff 链；查"烤炉烹饪时炉口的火怎么播"走 oven_fire 链。要接入火焰玩法（燃烧/供热）请用 [[reference/prefabs/fire|火焰、火光与烟雾特效族]] 的 `fire`／`campfirefire` 体系，不要复用这两者。
- 修改边界：只覆盖 2 注册的工厂、生成点、动画驱动与清理；冬宴餐桌 feast 系统、烤炉 madsciencelab 烹饪全流程、`wintersfeastfuel` 掉落只辨析入口不展开。
- 名称：2 个覆盖注册名在简中 `chinese_s.po` 均无 `STRINGS.NAMES.*` 键，标题中文为已核验效果描述非官方译名；链上宿主官方名 `WINTERSFEASTOVEN`＝"砖砌烤炉"、`WINTERSFEASTFUEL`＝"节日欢愉"（锚点见[验证记录](#验证记录)）。

## 注册与构造

### 冬宴增益粒子（wintersfeastbuff.lua 2 注册，宿主同文件）

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wintersfeastbuff_fx` | `effectfn`：Transform／AnimState／Network；SetFourFaced；bank＝build `winters_feast_fx`；泛光 `shaders/anim.ksh`＋LightOverride 1；`PlayAnimation(math.random(1, 10))` 随机选 10 个动画之一；标签 `FX` → SetPristine 客户端早退 → 主模拟 `persists=false`＋`animover` 自删＋`OnEntitySleep` 即删 | 无 SoundEmitter、无 Follower——每次投放都是**独立实体**落在目标当前位置；随机的是动画编号（10 个单帧变体），非帧时间。[工厂](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L137-L166)、[自删与休眠](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L99-L101) |
| `wintersfeastbuff`（宿主 debuff，非本页覆盖注册） | `fn`：master-only 无 Network 实体（`Hide`＋`persists=false`＋`CLASSIFIED`）；debuff 组件 `keepondespawn=true`；timer `buffover` 初值 7 秒 | 承载 buff 全部数值链与循环声 `feast_buff_LP`（强度参数化）；粒子投放由它的周期任务驱动。[宿主工厂](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L103-L135)、[注册](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L168) |

### 烤炉火层（wintersfeastoven.lua 注册）

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wintersfeastoven_fire` | `fire_fn`：Transform／AnimState／Network；bank＝build `wintersfeast_oven`（**与炉体同库**，靠动画名分离火层）；泛光；开机动画 `idle_fire_off`；标签 `FX`＋`DECOR` → SetPristine 客户端早退 → 主模拟 `persists=false` | 无自删、无 SoundEmitter、无 Light；`DECOR` 标签使它可被"装饰物"类检索/避让语义命中。[工厂](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L387-L411)、[标签](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L399-L400) |

文件另注册炉体 `wintersfeastoven`、放置预览 `wintersfeastoven_placer` 与 18 个 `wintercooking_*` 占位（本地 Transform 壳，0 tick 自删，只为配方预载消日志）（[占位工厂](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L413-L428)）。

## 关联机制导航

### buff FX 的触发链（进食 → debuff → 变频投放）

| 环节 | 触发与条件 | 证据 |
|---|---|---|
| 进入盛宴 | 玩家对餐桌执行 `WINTERSFEAST_FEAST` 动作 → SGwilson 推 `feasterstarted`，进入 `winters_feast_eat` 状态循环进食；离席 onexit 推 `feasterfinished` | [ActionHandler](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1571-L1577)、[状态名](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22593)、[feasterfinished](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22664) |
| 结算投放 debuff | feasts 组件每 1/3 秒 `doFeastCheck`：按桌组统计冬宴菜（`wintersfeastcookedfood` 标签）→ `ApplyFeastBuff` 调 `AddDebuff("wintersfeastbuff", "wintersfeastbuff")` 并把（就餐人数、菜品种类数、总菜数）交给 `addeffectbonusfn` 延长时长 | [ApplyFeastBuff](../../../raw/current/scripts/components/feasts.lua#L57-L78)、[doFeastCheck](../../../raw/current/scripts/components/feasts.lua#L80-L124) |
| 附着 | `OnAttached`：debuff 父挂目标（位置归零防读档漂移）；2 秒周期 `OnTick` 回血 1／回饥饿（按 `WINTERSFEASTBUFF.HEALTH_GAIN`／`HUNGER_GAIN`）；监听目标 `death`（剩余时长≥50% 时掉落 `wintersfeastfuel`）；饥饿 burn rate 乘数清零＋理智外部修正；给目标挂 `wintersfeastbuff` 标签 | [OnAttached](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L37-L60)、[OnTick](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L9-L23) |
| **粒子投放** | 初始延迟 0.5＋0.75·rand 秒后 `OnFxTick`：在目标当前位置生成 `wintersfeastbuff_fx`，强度 = 剩余时长/50（封顶 1）写进循环声参数，再按 `Remap(强度, 0, 1, 1.8, 0.25)` 秒排下一次——**越接近满强度投放越密（1.8→0.25 秒）** | [OnFxTick](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L29-L35)、[调值](../../../raw/current/scripts/tuning.lua#L5176-L5186) |
| 时长加成 | `AddEffectBonus`：score = (人数^0.3×0.3×种类数) ＋ (总菜数−种类数)×0.2；加时 = score×总日长/2/180 秒 | [AddEffectBonus](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L78-L97) |
| 结束 | `buffover` 计时到点或目标死亡 → debuff `Stop` → `OnDetached`：移除两修正、播 `ANNOUNCE_WINTERS_FEAST_BUFF_OVER`、宿主自删（粒子靠 animover/休眠自然清完） | [OnDetached](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L62-L70)、[OnTimerDone](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L72-L76) |

UI 消费端：`healthbadge` 把该 debuff 计入"持续治疗"绿箭头判据（[healthbadge.lua](../../../raw/current/scripts/widgets/healthbadge.lua#L405-L412)）；`hungerbadge` 读目标 `wintersfeastbuff` 标签显示"获取中、无消耗"（[hungerbadge.lua](../../../raw/current/scripts/widgets/hungerbadge.lua#L20-L30)）。

### oven_fire 的档位与宿主装配

- **生成与重建**：`MakeFireFx` 先 `RemoveFireFx` 再生成并父挂炉体（[L65-L77](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L65-L77)）。两处调用：①`StartMakingScience`——prototyper 点火且配方产物是 `wintercooking_*` 冬宴菜时生成并同时开灯（[L122-L131](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L122-L131)）；②`OnStageStarted` 开头兜底——`_firefx` 无效（读档中途烹饪：madsciencelab OnLoad 重走 SetStage）时重建（[L200-L203](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L200-L203)、[组件 OnLoad](../../../raw/current/scripts/components/madsciencelab.lua#L63-L65)）。
- **档位驱动**：烤炉两阶段（[SCIENCE_STAGES](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L28-L33)），单阶段时长 = 冬宴基准 cooktime/2；各冬宴菜按 `wintersfeastcookedfoods` 表给倍率（[GetWinterFoodCookTimeMultiplier](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L103-L105)）。火层动画随阶段走：阶段 1 先播 `pre_cooking_fire` 再循环 `cooking_loop_fire`，阶段 2 续推同循环（[OnStageStarted](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L228-L231)）。
- **收尾**：`OnScienceWasMade`——火层播 `cooking_post_fire`，宿主 `DoTaskInTime(动画长度, RemoveFireFx)` 移除并关灯（[L246-L249](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L246-L249)）——这是火层唯一的消失路径，仍由宿主调度，实体自身无自删。
- **与 fire/campfirefire 体系辨析**：`wintersfeastoven_fire` 不挂 `firefx` 组件（无等级表、无灯位写入）、无 `heater`（不产温度）、无 SoundEmitter，光效在炉体本体；动画直接复用炉体 bank 而非独立 `fire` 库。它与 [[reference/prefabs/fire|火焰、火光与烟雾特效族]] 的 `fire`／`campfirefire`／`character_fire` 是两套互不相通的体系，不能走 `AddBurnFX`/`SetFXLevel` 接入。

## 典型接入

### 生成"随机变体、放完即走"的庆祝粒子

- 前提与选择：主模拟生成（客户端工厂早退）；要一次投放一个自动消失的光粒，模式是 `persists=false`＋`animover` 自删＋休眠即删，不需生成方管理生命周期。
- 生效与撤销：实体不父挂，落在生成时给定的坐标；想跟人走就在每次 tick 重新 SetPosition（原版即此做法），或改用父挂＋Follower 方案（见 [[reference/prefabs/wortox_soulecho_buff_fx|沃拓克斯灵魂增益与装备跟随特效族]]）。
- 原版依据：[工厂](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L137-L166)、[周期投放](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L29-L35)。

```lua
local function SpawnFeastBurst(x, y, z)
    local fx = SpawnPrefab("wintersfeastbuff_fx")
    fx.Transform:SetPosition(x, y, z) -- 不父挂：粒子留在原地，播完或休眠即消失
    return fx
end
```

- 调整与验证：替换点是 bank/build 与 `PlayAnimation` 的随机区间；动画包需在 Prefab 的 assets 里声明（原版 `anim/winters_feast_fx.zip`）。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：buff 粒子不跟随。** 每次投放是独立实体落在目标当时位置，目标走开后粒子留在原地；跟随语义由"高频重投"近似，别误当成父挂层。[OnFxTick](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L29-L35)
- **边界：粒子休眠即删。** `OnEntitySleep = ParticleOnEntitySleep` 直接 Remove——玩家远离后粒子提前消失，读档/切屏后不会补投（debuff 的 fxtask 继续跑，回到视野才有新粒子）。[L163](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L163)
- **边界：投放密度反着读。** `Remap(强度, 0, 1, 1.8, 0.25)` 是"剩余时长越少越密"：buff 快结束（强度→0）时间隔 1.8 秒，刚吃完满强度时 0.25 秒；强度参数同时驱动循环声音量包络。[OnFxTick](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L29-L35)
- **边界：`wintersfeastbuff` debuff `keepondespawn=true`。** 玩家下线重连 buff 保留（debuff 系统负责重建），但粒子投放由 debuff 实体的任务驱动，重建后从新的随机延迟继续；目标的 `wintersfeastbuff` 标签是 hungerbadge 的客户端判据，别在别处复用该标签名。[L128](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L128)
- **误区：饥饿不消耗是 buff 的"饱食"。** 实现是 `hunger.burnratemodifiers:SetModifier(inst, 0)` 把消耗速率清零，不是给 hunger 加 gain；hungerbadge 的"无消耗"图标读的是目标标签而非组件值。[L56](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L56)
- **边界：oven_fire 不能当火焰玩。** 无 firefx/heater/burnable，锤炉、燃烧、供热都别找它；它只是炉体 bank 上的火动画层（`DECOR` 标签例外地参与了"装饰"语义）。[工厂](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L387-L411)
- **边界：oven_fire 无自删、读档依赖宿主兜底。** 完成菜后有宿主定时移除；若烹饪中途存档读档，火层靠 `OnStageStarted` 的 IsValid 检查重建——直接照抄工厂而不接宿主回调会得到永不消失的火层。[OnStageStarted 兜底](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L200-L203)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/wintersfeastbuff.lua`、`prefabs/wintersfeastoven.lua`、`components/feasts.lua`、`components/madsciencelab.lua`；按需读取 `stategraphs/SGwilson.lua` feast 动作与 `winters_feast_eat` 状态段、`widgets/healthbadge.lua`／`widgets/hungerbadge.lua` 消费段、`tuning.lua`（WINTERSFEASTBUFF、WINTERSFEASTTABLE、WINTERS_FEAST_OVEN_BASE_COOK_TIME、night_time 推导）。对 2 个覆盖注册名全域反查：`wintersfeastbuff_fx` 命中 3 处（依赖声明 [wintersfeastbuff.lua:3](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L3)、生成 [L30](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L30)、注册 [L169](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L169)）——**生成点唯一**；`wintersfeastoven_fire` 命中 3 处（依赖 [L12](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L12)、生成 [L75](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L75)、注册 [L432](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L432)）——**生成点唯一**。两注册均不在 prefablist.lua（宿主 `wintersfeastbuff`:1459、`wintersfeastoven`:1463 在列）。台账批 9 的 2 注册清单复核无清单外 FX 注册。
- 名称核验：`languages/chinese_s.po` 定向检索——`WINTERSFEASTOVEN`＝"砖砌烤炉"（[L352220-L352223](../../../raw/current/scripts/languages/chinese_s.po#L352220-L352222)）、`WINTERSFEASTFUEL`＝"节日欢愉"（[L352215-L352218](../../../raw/current/scripts/languages/chinese_s.po#L352215-L352217)）、`ANNOUNCE_WINTERS_FEAST_BUFF`／`ANNOUNCE_WINTERS_FEAST_BUFF_OVER`／`ANNOUNCE_IS_FEASTING` 三键在列（[L7764-L7772](../../../raw/current/scripts/languages/chinese_s.po#L7764-L7772)）；`WINTERSFEASTBUFF_FX`／`WINTERSFEASTOVEN_FIRE` 无 NAMES 键；任务提示的 `WINTERASURE` 拼写全域 0 命中（疑为笔误，正确键系 `WINTERSFEAST*`）。标题中文使用已核验效果描述。
- 分档说明：`size_class: standard`——2 注册两条独立链（debuff 变频投放链／烤炉两阶段装配链），触发链用表格压缩；未选 complex，两链均无多状态机。
- 未覆盖：冬宴餐桌 feast 系统全貌（`wintersfeasttable` 组件的 DepleteFood／席间玩笑与 announce 调度）、烤炉 18 种冬宴菜 cooktime 表与产物结算、`wintersfeastfuel` 的使用链、冬宴装饰物掉落、动画包帧内容与观感；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/fire|火焰、火光与烟雾特效族]]（firefx 等级表／heater 体系对照——烤炉火不走该链）
- [[reference/prefabs/collapse_big|建筑倒塌特效]]（烤炉锤毁生成的 `collapse_small`）
- [[reference/prefabs/wortox_soulecho_buff_fx|沃拓克斯灵魂增益与装备跟随特效族]]（"buff 粒子父挂跟随"的对照模式）
- 返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]（冬宴活动/装备 buff）
