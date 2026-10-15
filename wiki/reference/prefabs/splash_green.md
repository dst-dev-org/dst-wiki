---
title: "海面绿花与冲滩水洼（splash_green 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "通用表海面绿花三档（small/green/large）与冲滩地面水洼 washashore_puddle_fx：按体型选名、冲滩状态链伴生与主要生成者清单。"
aliases: ["splash_green", "splash_green_small", "splash_green_large", "washashore_puddle_fx", "绿色水花", "海面绿花", "冲滩水洼", "水洼特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "stategraphs/commonstates.lua", "componentutil.lua", "stategraphs/SGwilson.lua", "stategraphs/SGmalbatross.lua", "stategraphs/SGeyeofterror.lua", "components/submersible.lua", "components/floater.lua", "prefabs/oceantree.lua", "prefabs/chum.lua", "prefabs/boat.lua", "prefabs/boatrace_common.lua", "prefabs/singingshell.lua", "prefabs/messagebottle.lua", "prefabs/icefishing_hole.lua", "prefabs/junkball.lua", "components/sharkboimanager.lua", "prefabs/crabking.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 海面绿花与冲滩水洼（splash_green 族）

## 快速判断

- 职责与适用：通用表四个水陆边界表现注册——三档海面绿花 `splash_green_small`／`splash_green`／`splash_green_large`（pond_splash_fx 资源）与冲滩地面水洼 `washashore_puddle_fx`（water_puddle 资源）。纯表现，无组件；加湿等玩法由生成者实现。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[水洼条目](../../../raw/current/scripts/fx.lua#L1964-L1969)、[绿花条目](../../../raw/current/scripts/fx.lua#L2006-L2028)
- 优先使用：实体入水／沉底／冲滩／击退落水需要按体型选水花读本页注册与构造；普通漂浮物落水的冷热分支走 [[reference/prefabs/splash|splash 族]]（沉水物品与投掷瓶另有本族调用，见陷阱）。
- 重要副作用：无；`splash_green`／`_large` 在 fn 内 `Transform:SetScale(2/4)`（区别于表字段 `transform` 的 AnimState 缩放）；`washashore_puddle_fx` 是 LAYER_WORLD_BACKGROUND＋OnGround 地面贴图层。[fn](../../../raw/current/scripts/fx.lua#L2014-L2028)、[水洼层](../../../raw/current/scripts/fx.lua#L1964-L1969)
- 修改边界：全族为落点式一次性（本组消费点未见父挂）；玩家冲滩状态只落水洼不落绿花；malbatross 的选名是死代码（见陷阱）。[玩家分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13685-L13696)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `splash_green_small` | 通用表条目 → 加载器代理＋一帧后本地实体；fn=FinalOffset1 | bank/build=`pond_splash_fx`，anim=`pond_splash`，音效 `water/splash/small`。[条目](../../../raw/current/scripts/fx.lua#L2006-L2012) |
| `splash_green` | 同上协议 | 同资源同动画；音效 `medium`；fn 内 `Transform:SetScale(2,2,2)`＋FinalOffset1。[条目](../../../raw/current/scripts/fx.lua#L2014-L2020) |
| `splash_green_large` | 同上协议 | 音效 `large`；fn 内 `Transform:SetScale(4,4,4)`＋FinalOffset1。[条目](../../../raw/current/scripts/fx.lua#L2022-L2028) |
| `washashore_puddle_fx` | 同上协议 | bank/build=`water_puddle`，anim=`puddle`，无音效；fn 设 `LAYER_WORLD_BACKGROUND`＋`SetOrientation(ANIM_ORIENTATION.OnGround)` 成地面贴图。[条目](../../../raw/current/scripts/fx.lua#L1964-L1969) |

简中 `languages/chinese_s.po` 定向检索无这 4 个注册名的显示名键；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 四条目同协议，差异仅资源、音效与 fn | 全族同协议 | [[reference/mechanisms/generic-fx-factory]] |
| 选名 helper：`GetCombatFxSize` | 按目标返回 `r, sz, ht`，sz 取 tiny/small/med/large（物理半径＋标签＋`override_combat_fx_*` 覆写） | 击退落水时由消费点映射：tiny→small、large→large、其余→green | [定义](../../../raw/current/scripts/componentutil.lua#L959-L989)、[映射](../../../raw/current/scripts/stategraphs/commonstates.lua#L3300-L3306) |
| 冲滩链路 | 生物 `sink` 状态经 `DoWashAshore` 先落 `splash_green`（`skip_splash` 可跳过）；`on_washed_ashore` 转入 `washed_ashore` 状态再落水洼＋`splash_green` | 服务端状态图；水洼与绿花同点伴生 | [DoWashAshore](../../../raw/current/scripts/stategraphs/commonstates.lua#L2119-L2129)、[状态内双落](../../../raw/current/scripts/stategraphs/commonstates.lua#L2255-L2256) |
| 玩家冲滩差异 | SGwilson `washed_ashore` onenter 只生成 `washashore_puddle_fx`，无绿花 | 玩家侧；先结算溺水伤害再落水洼 | [状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13685-L13696) |
| 沉底／爬岸：submersible | 物品沉水组件两条路径落 `splash_green`：搬回陆地（旧位置）与 `MakeSunken` 沉底（`nosplash` 可关） | 服务端组件；沉底装入 underwater_salvageable | [[reference/components/submersible]]、[两处](../../../raw/current/scripts/components/submersible.lua#L172-L191) |
| 生成点群（代表例，非穷举） | 饵料入海 [chum](../../../raw/current/scripts/prefabs/chum.lua#L22-L30)；草船漏点 [boat](../../../raw/current/scripts/prefabs/boat.lua#L216-L222)；船赛部署成功 [boatrace_common](../../../raw/current/scripts/prefabs/boatrace_common.lua#L249-L258)；漂流瓶落海 [messagebottle](../../../raw/current/scripts/prefabs/messagebottle.lua#L285-L292)、[胶团瓶](../../../raw/current/scripts/prefabs/messagebottle.lua#L385-L392)；歌螺生成点失效移除 [singingshell](../../../raw/current/scripts/prefabs/singingshell.lua#L314-L321) | 落点前普遍先判 `IsVisualGroundAtPoint`／平台，仅水面才落花 | 各锚点 |
| 大水花（_large）消费点 | 冰钓洞玩家靠近 [icefishing_hole](../../../raw/current/scripts/prefabs/icefishing_hole.lua#L76-L80)；垃圾球落海 [junkball](../../../raw/current/scripts/prefabs/junkball.lua#L486-L490)；鲨鱼人竞技场清钓洞 [sharkboimanager](../../../raw/current/scripts/components/sharkboimanager.lua#L201-L206)；蟹王生成海堆 [crabking](../../../raw/current/scripts/prefabs/crabking.lua#L1153-L1157)；恐怖之眼潜入非地面 [SGeyeofterror](../../../raw/current/scripts/stategraphs/SGeyeofterror.lua#L869-L880) | 多数落点 y=0；冰钓洞随后推 knockback | 各锚点 |
| 海树按生长期选档 | oceantree 发芽落 `splash_green` [Sprout](../../../raw/current/scripts/prefabs/oceantree.lua#L172-L178)；倒树落水按 growth_stage 选 green/large 并配 scale/splash_dist [falling_tree_land](../../../raw/current/scripts/prefabs/oceantree.lua#L843-L859) | 服务端；三段生长期两档 | [锚点](../../../raw/current/scripts/prefabs/oceantree.lua#L843-L859) |
| 同型命中（未展开） | 海洋生物状态图十余文件同型落点：SGshark／SGsquid／SGhound／SGotter／SGsalty_dog／SGpowdermonkey／SGprimemate／SGBeefalo／SGcrabking_mob／SGgrassgator／SGspider_water 及 SGsharkboi | rg 命中 `SpawnPrefab("splash_green...")` 行，未逐个展开触发条件 | rg 检索 raw/current/scripts 全根 |
| 资源 | bank/build=`pond_splash_fx`（三档同资源）、`water_puddle`；音效 turnoftides `water/splash/small|medium|large` | 动画时长与循环性未核验 | [水洼条目](../../../raw/current/scripts/fx.lua#L1964-L1969)、[绿花条目](../../../raw/current/scripts/fx.lua#L2006-L2028) |

## 典型接入

### 按体型借用海面绿花

- 前提与选择：实体入水／沉底需要水花时，用全局 `GetCombatFxSize`（componentutil）取体型档再选名；只想区分普通/大型可仿消费点直接写死档位。
- 生效与撤销：一次性落点动画，animover 自删；落点 y 多为 0（贴海面）。
- 原版依据：击退落水映射 [commonstates](../../../raw/current/scripts/stategraphs/commonstates.lua#L3300-L3306)。

```lua
local _, sz = GetCombatFxSize(target)   -- componentutil 全局：tiny/small/med/large
local fx = SpawnPrefab(sz == "tiny" and "splash_green_small"
    or sz == "large" and "splash_green_large" or "splash_green")
fx.Transform:SetPosition(x, 0, z)
```

- 调整与验证：`GetCombatFxSize` 优先读 `override_combat_fx_radius/size` 覆写；无需分档时直接 `SpawnPrefab("splash_green")`。

## 常见陷阱与限制

- **误区：splash_green 与物品落水无关**：floater 的普通漂浮冷热分支确实走 [[reference/prefabs/splash|splash 族]]，但沉水物品与投掷瓶另有本族调用——`submersible` 的 `OnLanded` 在海上进入 `Submerge`，`MakeSunken` 默认落 `splash_green`（`nosplash` 可关）；投掷瓶／胶团瓶在无平台的非地面位置落 `splash_green_small`。不能从 floater 单一路径推出整族不用于物品落水。[floater 分支](../../../raw/current/scripts/components/floater.lua#L150-L174)、[沉水](../../../raw/current/scripts/components/submersible.lua#L73-L78)、[沉底](../../../raw/current/scripts/components/submersible.lua#L181-L193)、[瓶](../../../raw/current/scripts/prefabs/messagebottle.lua#L285-L292)、[胶团瓶](../../../raw/current/scripts/prefabs/messagebottle.lua#L385-L392)
- **边界：malbatross 的 med 档选名是死代码**：`spawnsplash` 给局部 `prefab` 按 size 赋值后，`SpawnPrefab` 仍写死 `"splash_green_large"`——无论传什么档位都出大花。[函数](../../../raw/current/scripts/stategraphs/SGmalbatross.lua#L22-L33)
- **边界：玩家冲滩没有绿花**：生物链路双落水洼＋绿花，玩家 SGwilson 只落水洼；复用冲滩表现时按宿主类型区分。[玩家](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13685-L13696)、[生物](../../../raw/current/scripts/stategraphs/commonstates.lua#L2255-L2256)
- **边界：washashore_puddle_fx 是地面贴图**：OnGround 定向＋背景层，无音效，与竖立水花不同层；不要当作通用地面水花替换 splash 系。[条目](../../../raw/current/scripts/fx.lua#L1964-L1969)
- **边界：绿色来自资源而非染色**：`pond_splash_fx` 是池塘水花资源，名称 green 与之对应；颜色未实测，条目内无 SetTint/染色字段。[条目](../../../raw/current/scripts/fx.lua#L2006-L2028)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。四条目配置、通用工厂协议（经 [[reference/mechanisms/generic-fx-factory|工厂页]] 复用）、commonstates 冲滩链与 knockback 落水选名、GetCombatFxSize、submersible 两路径（物品落水／沉底）、floater 普通冷热分支与投掷瓶落海的 green 族调用分工、SGwilson/SGmalbatross/SGeyeofterror、oceantree/chum/boat/boatrace_common/singingshell/messagebottle/icefishing_hole/junkball/sharkboimanager/crabking 逐点核验；海洋生物 SG 十余处为同型 rg 命中，未逐个展开触发条件；简中 .po 定向核验 4 注册名无显示名键；动画时长未核验。
- 游戏运行：未运行；水花视觉、声音与层级表现未实测。

返回 [[reference/prefabs/fx-water-index|水面、航海与船件]]、[[reference/prefabs/index|Prefab 索引]]。
