---
title: "通用圆烟、滑行烟与木屑（round_puff 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "命中冲撞圆烟 round_puff_fx 三档、滑行闪避尘烟 slide_puff 与饼干切割者钻船木屑 wood_splinter 双名：标签选名、代理绑定符号特例与无生成点缺口。"
aliases: ["round_puff_fx_sm", "round_puff_fx_lg", "round_puff_fx_hi", "slide_puff", "wood_splinter_jump", "wood_splinter_drill", "圆烟", "命中烟", "滑行烟", "木屑特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/woodie.lua", "prefabs/yoth_lance.lua", "prefabs/dumbbells.lua", "prefabs/spider_web_spit.lua", "prefabs/crabking.lua", "stategraphs/SGwilson.lua", "stategraphs/SGmerm.lua", "stategraphs/SGpigelite.lua", "stategraphs/SGeyeofterror.lua", "stategraphs/SGcookiecutter.lua", "prefabs/cookiecutter.lua", "prefabs/pigelite.lua", "prefabs/merm.lua", "prefabs/mermking.lua", "prefabs/powdermonkey.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 通用圆烟、滑行烟与木屑（round_puff 族）

## 快速判断

- 职责与适用：六个生物／武器动作表现注册——round_puff 三档通用圆烟（命中、冲撞、疾驰碰撞）、slide_puff 滑行闪避尘烟（四处状态图）、wood_splinter 双名木屑（饼干切割者钻船）。全部纯表现，无组件；协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L1824-L1829)、[圆烟](../../../raw/current/scripts/fx.lua#L1971-L1991)、[木屑](../../../raw/current/scripts/fx.lua#L1993-L2003)
- 优先使用：命中／冲撞按体型选烟→round_puff sm/lg（标签判据见典型接入）；滑行／闪避尘→slide_puff；钻船木屑借用→`wood_splinter_drill`（jump 无生成点）。
- 重要副作用：无；sm/lg 条目自带 moose hit 音效、hi 无声；crabking 的 lg 代理父挂并绑定符号，本地动画不保证持续跟随符号。[hi 档](../../../raw/current/scripts/fx.lua#L1987-L1991)
- 修改边界：round_puff／slide_puff 均为落点式一次性（crabking 除外）；slide_puff 的 build 另被多个角色当符号覆盖源（不是生成）；木屑音效在条目内为空，声音由宿主播放。[符号复用](../../../raw/current/scripts/prefabs/pigelite.lua#L178-L178)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `round_puff_fx_sm` | 通用表条目 → 加载器代理＋一帧后本地实体 | bank/build=`round_puff_fx`，anim=`puff_sm`，音效 `woodie/moose/hit`，fn=FinalOffset1。[条目](../../../raw/current/scripts/fx.lua#L1971-L1977) |
| `round_puff_fx_lg` | 同上协议 | anim=`puff_lg`，音效与 fn 同 sm。[条目](../../../raw/current/scripts/fx.lua#L1979-L1985) |
| `round_puff_fx_hi` | 同上协议 | anim=`puff_hi`，无音效、无 fn（不提升显示层）。[条目](../../../raw/current/scripts/fx.lua#L1987-L1991) |
| `slide_puff` | 同上协议 | bank=`fx_slidepuff`，build=`slide_puff`，anim=`anim`，无音效，fn=FinalOffset1；缩放由消费点设定。[条目](../../../raw/current/scripts/fx.lua#L1824-L1829) |
| `wood_splinter_jump` | 同上协议 | bank/build=`cookiecutter_fx`，anim=`wood_splinter_jump`，无声；当前快照无生成点。[条目](../../../raw/current/scripts/fx.lua#L1993-L1997) |
| `wood_splinter_drill` | 同上协议 | 同资源，anim=`wood_splinter_drill`，无声；仅钻船状态周期生成。[条目](../../../raw/current/scripts/fx.lua#L1999-L2003) |

简中 `languages/chinese_s.po` 定向检索无这 6 个注册名的显示名键；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 六条目同协议，差异仅资源、音效与 fn | 全组同协议 | [[reference/mechanisms/generic-fx-factory]] |
| 圆烟标签选名 | `(HasTag("largecreature") or HasTag("epic")) and lg or sm`：伍迪（驼鹿形态）冲撞踩踏与骑枪命中同判据 | 服务端回调；无 mid 档 | [OnTackleTrample](../../../raw/current/scripts/prefabs/woodie.lua#L1568-L1570)、[yoth_lance](../../../raw/current/scripts/prefabs/yoth_lance.lua#L26-L29) |
| round_puff 生成点 | 冲撞落点取连线上目标边缘＋调用方播 bounce／震屏 [woodie](../../../raw/current/scripts/prefabs/woodie.lua#L1552-L1566)；哑铃落地 [dumbbells](../../../raw/current/scripts/prefabs/dumbbells.lua#L175-L175)；酸液弹爆裂 [spider_web_spit](../../../raw/current/scripts/prefabs/spider_web_spit.lua#L22-L27)；骑乘疾驰碰撞 [TryGallopCollideUpdate](../../../raw/current/scripts/stategraphs/SGwilson.lua#L829-L829)（[run_gallop](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12145-L12145) 内调用 [L12208](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12208-L12208)） | 全部落点式；gallop 为 sm 固定档 | 各锚点 |
| 特例：crabking | 损伤换装时给 lg 代理父挂蟹王并用 `AddFollower`／`FollowSymbol(GUID, "damage"..art)` 绑定损伤符号 | 本地动画继承代理父实体、创建时取代理变换；工厂未给本地动画设置 Follower，不能据此保证持续跟随符号 | [SetDamagedArt](../../../raw/current/scripts/prefabs/crabking.lua#L1404-L1420)、[本地动画](../../../raw/current/scripts/prefabs/fx.lua#L18-L37) |
| slide_puff 状态图生成点 | 玩家 divegrab 第 5 帧 scale 1.3 落身前 [SGwilson](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26996-L26998)；鱼人／鱼人守卫 `dodge_attack` 第 6 帧 [SGmerm](../../../raw/current/scripts/stategraphs/SGmerm.lua#L798-L804)（状态 [L777-L794](../../../raw/current/scripts/stategraphs/SGmerm.lua#L777-L794)；SGmerm 由 [鱼人守卫](../../../raw/current/scripts/prefabs/merm.lua#L1303-L1303)、[鱼人](../../../raw/current/scripts/prefabs/merm.lua#L1467-L1467) 安装，鱼人王用 [SGmermking](../../../raw/current/scripts/prefabs/mermking.lua#L624-L626)）；猪卫 dive 第 5 帧、摔地声在调用方 [SGpigelite](../../../raw/current/scripts/stategraphs/SGpigelite.lua#L404-L407)；恐怖之眼有地面才落（否则换 boss_ripple_fx）[spawn_ground_fx](../../../raw/current/scripts/stategraphs/SGeyeofterror.lua#L121-L127) | 除玩家外均生物服务端状态；scale 1.3 出现在 divegrab 与恐怖之眼 | 各锚点 |
| slide_puff 符号复用（非生成） | 猪卫 `AddOverrideBuild("slide_puff")`、猴子投手 `OverrideSymbol("fx_slidepuff01", ...)`：借 build 补角色动画符号 | 资源复用路径，不产生 FX 实体 | [pigelite](../../../raw/current/scripts/prefabs/pigelite.lua#L178-L178)、[powdermonkey](../../../raw/current/scripts/prefabs/powdermonkey.lua#L232-L232) |
| wood_splinter_drill | SGcookiecutter `drill` 状态（对船钻探）`DoPeriodicTask(动画长, 0)` 周期落点于切割者自身位置 | 服务端；进入条件为 target_wood 有 boat 标签 | [状态](../../../raw/current/scripts/stategraphs/SGcookiecutter.lua#L548-L582)、进度组件 [[reference/components/cookiecutterdrill]] |
| wood_splinter_jump | 仅 cookiecutter 的 prefabs 依赖声明与 fx.lua 注册，无生成点 | 缺口见陷阱 | [声明](../../../raw/current/scripts/prefabs/cookiecutter.lua#L10-L16) |
| 资源 | `round_puff_fx`（三 anim）、`fx_slidepuff`／`slide_puff`、`cookiecutter_fx` | 音效仅 sm/lg 有 moose hit | [条目](../../../raw/current/scripts/fx.lua#L1824-L2003) |

## 典型接入

### 按目标体型借用命中圆烟

- 前提与选择：命中／冲撞表现采用原版标签判据选 sm/lg；需要 tiny/med 细分档时改用 GetCombatFxSize（见 [[reference/prefabs/splash_green|海面绿花页]] 接入）。
- 生效与撤销：一次性落点动画，animover 自删；sm/lg 自带命中音效，勿重复播放。
- 原版依据：骑枪命中 [yoth_lance](../../../raw/current/scripts/prefabs/yoth_lance.lua#L26-L29)。

```lua
local fx = SpawnPrefab((target:HasTag("largecreature") or target:HasTag("epic"))
    and "round_puff_fx_lg" or "round_puff_fx_sm")
fx.Transform:SetPosition(target.Transform:GetWorldPosition())
```

- 调整与验证：判据只认两个标签，普通生物全落 sm；若需连线上目标边缘，可仿 woodie `OnTackleCollide` 的 hi 档 `r / (r + 1)` 插值；本族 sm／lg 的原版 `OnTackleTrample` 与骑枪都直接取目标中心。[hi 插值](../../../raw/current/scripts/prefabs/woodie.lua#L1552-L1557)、[Trample](../../../raw/current/scripts/prefabs/woodie.lua#L1568-L1570)

## 常见陷阱与限制

- **边界：wood_splinter_jump 当前无生成点**：rg `wood_splinter` 全 scripts 根仅命中 fx.lua 注册、cookiecutter 依赖声明与 SGcookiecutter 的 drill 生成；动态 SpawnPrefab 拼名全清单核对无 wood_splinter 前缀拼接——SGwilson L21154 的 `"wood_splinter_"..product` 是 OverrideSymbol 动画符号拼接，不是 SpawnPrefab。可借用但原版未使用。[符号拼接](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21152-L21154)
- **误区：slide_puff 出现在猪卫／猴子身上是生成特效**：那几处是 AddOverrideBuild／OverrideSymbol 借 build 补符号；真正生成仅四处状态图（见导航行）。[复用](../../../raw/current/scripts/prefabs/powdermonkey.lua#L232-L232)
- **边界：hi 档无声且不抬层**：woodie OnTackleCollide 的 bounce 声与震屏由调用方实现；想要声音需自播。[调用方](../../../raw/current/scripts/prefabs/woodie.lua#L1552-L1566)
- **边界：crabking 的 lg 代理绑定符号**：代理父挂并跟随损伤符号；本地动画继承父实体并在创建时取代理变换，持续符号跟随尚未由脚本证明。[特例](../../../raw/current/scripts/prefabs/crabking.lua#L1404-L1420)、[工厂](../../../raw/current/scripts/prefabs/fx.lua#L18-L37)
- **对比：round_puff 与其他烟团**：[[reference/prefabs/sand_puff|sand_puff 系]] 是沙尘／暗影烟团，[[reference/prefabs/small_puff|small_puff]] 是变身／出生烟；round_puff 定位命中／冲撞烟，选名判据也不同（标签而非事件）。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。六条目配置、woodie 两回调（Collide 的 hi 插值与 Trample 的目标中心已分开）、yoth_lance、dumbbells、spider_web_spit、crabking 特例、SGwilson（gallop 函数与 run_gallop 调用、divegrab）、SGmerm 宿主归属（merm／鱼人守卫安装，鱼人王用 SGmermking）、SGpigelite／SGeyeofterror、SGcookiecutter drill 周期任务、符号复用四处逐点核验；`wood_splinter` 全根检索与动态 SpawnPrefab 拼名清单核对完成；简中 .po 定向核验 6 注册名无显示名键；各状态图上游触发链（谁 GoToState）未展开，动画时长未核验。
- 游戏运行：未运行；烟团视觉、音效与符号跟随表现未实测。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/prefabs/index|Prefab 索引]]。
