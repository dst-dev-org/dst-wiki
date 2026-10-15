---
title: "火球（emberlight）"
created: "2026-11-02"
updated: "2026-11-04"
type: prefab
summary: "薇洛火焰法术生成的火球世界实体：照明、供热、火势传播、烹饪、精神光环及定时熄灭。"
aliases: ["emberlight", "火球", "薇洛火球", "火球照明", "薇洛火焰法术"]
tags: [prefab, aura, lifecycle, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/stafflight.lua", "prefabs/willow_ember.lua", "prefabs/skilltree_willow.lua", "prefabs/willow.lua", "prefabs/shadowhand.lua", "prefabs/charlie_boss_trial.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 火球（emberlight）

## 快速判断

- `emberlight` 是薇洛余烬火球法术生成的联网世界实体，具有照明、供热、烹饪、热量传播和精神光环，不是仅负责动画的特效；法术施放与余烬消耗在 `willow_ember` 一侧，法术名简中为“火球术（2余烬）”。[生成](../../../raw/current/scripts/prefabs/willow_ember.lua#L312-L318)、[组件](../../../raw/current/scripts/prefabs/stafflight.lua#L152-L186)、[法术名](../../../raw/current/scripts/languages/chinese_s.po#L354929-L354932)
- 简中 `STRINGS.NAMES.EMBERLIGHT` 为官方名“火球”，与法术名“火球术”不是同一个字符串；名称同其他火球／投射物易混，检索时以 `emberlight` 和薇洛法术区分。[名称](../../../raw/current/scripts/languages/chinese_s.po#L342479-L342482)
- 该实体与 [[reference/prefabs/stafflight|矮星]]、[[reference/prefabs/staffcoldlight|极光]] 共享工厂；普通热星分支、独立时长和 `pre`／`post` 动画参数决定本条目的具体差异。[注册](../../../raw/current/scripts/prefabs/stafflight.lua#L202-L205)

## 注册与构造

| 注册名 | 工厂参数与端 | 证据 |
|---|---|---|
| `emberlight` | `makestafflight` 传入 hot=true、is_fx=false，bank／build `flameball_fx`，`pre` 后循环 `idle_loop`，退出 `post`；Light 色值 RGB 为 250／255、149／255、18／255。公共构造加 Transform、AnimState、SoundEmitter、Light、Network 和 `net_float`；主模拟后续添加玩法组件与计时器。 | [参数](../../../raw/current/scripts/prefabs/stafflight.lua#L205-L205)、[公共构造](../../../raw/current/scripts/prefabs/stafflight.lua#L79-L108)、[主模拟](../../../raw/current/scripts/prefabs/stafflight.lua#L149-L196) |

资源表按动画名声明 `anim/flameball_fx.zip`；动画素材内部与实际显示效果未解包或运行验证。[资源声明](../../../raw/current/scripts/prefabs/stafflight.lua#L66-L72)

## 关联机制导航

| 对象／路径 | 已核验关系及边界 | 证据 |
|---|---|---|
| 余烬法术 `willow_fire_ball` | 玩家需带 `ember_master` 标签（`willow_embers` 技能授予）才能打开余烬法术菜单；具体条目还要对应技能已激活才加入列表。选中时主模拟将 aoespell 绑定 `FireBallSpellFn`。施法先检查库存中余烬达到配置花费，`TryBallFire` 在传入的目标 `pos.x/y/z` 生成 `emberlight`，成功后消耗配置数量；不足返回 `NOT_ENOUGH_EMBERS`。 | [菜单前提](../../../raw/current/scripts/prefabs/willow_ember.lua#L942-L942)、[技能门](../../../raw/current/scripts/prefabs/skilltree_willow.lua#L110-L123)、[菜单筛选](../../../raw/current/scripts/prefabs/willow_ember.lua#L853-L869)、[绑定](../../../raw/current/scripts/prefabs/willow_ember.lua#L695-L714)、[生成](../../../raw/current/scripts/prefabs/willow_ember.lua#L312-L318)、[消耗](../../../raw/current/scripts/prefabs/willow_ember.lua#L421-L429) |
| 配置与依赖 | 本快照 `WILLOW_EMBER_BALL = 2`，`EMBER_STAR_DURATION = total_day_time = 480` 秒；薇洛的 `prefabs` 表声明 `emberlight`，该声明本身不是生成。 | [余烬检查](../../../raw/current/scripts/prefabs/willow_ember.lua#L109-L111)、[配置](../../../raw/current/scripts/tuning.lua#L8098-L8101)、[一天长度](../../../raw/current/scripts/tuning.lua#L19-L20)、[依赖](../../../raw/current/scripts/prefabs/willow.lua#L18-L20) |
| 玩法组件 | 主模拟无条件加 `heater`（热星 heat 100）和 `sanityaura`（`TUNING.SANITYAURA_SMALL`）；热星另加 `propagator`（heatoutput 15、spreading 并启动更新）；`cooker` 只在热星且非 `is_fx` 时添加。本注册再加 `inspectable`、`hauntable` 和 `timer`；具体作用范围按组件规则。 | [组件](../../../raw/current/scripts/prefabs/stafflight.lua#L152-L186)、[[reference/components/heater#携带范围与地面热源的分工]]、[[reference/components/propagator#关键接口]]、[[reference/components/cooker#关键接口]]、[[reference/components/sanityaura#关键接口]] |
| 光照与同步 | 每 0.1 秒计算脉动，光半径 11～12、强度与衰减在 0.8～0.7 间插值；主模拟周期同步存活时间相位，客户端监听 dirty 更新偏移并启用客户端调制。 | [脉动](../../../raw/current/scripts/prefabs/stafflight.lua#L24-L52)、[调度与光色](../../../raw/current/scripts/prefabs/stafflight.lua#L89-L100)、[客户端](../../../raw/current/scripts/prefabs/stafflight.lua#L142-L150) |
| 熄灭与调用者 | 计时器到期经 `timerdone` 直接进入 `kill_light`；作祟或外部调用 `ForceExtinguish` 时先要求 `extinguish` 计时存在并停止它，再进入同一函数：推 `onstarkilled`、播 post、动画结束停循环声、1 秒后移除，期间设不存档并标 `_killed`。暗影手将 `staffstar` 当目标时可调用该方法，并监听熄灭事件。 | [熄灭](../../../raw/current/scripts/prefabs/stafflight.lua#L5-L22)、[主动与作祟](../../../raw/current/scripts/prefabs/stafflight.lua#L54-L64)、[暗影手](../../../raw/current/scripts/prefabs/shadowhand.lua#L51-L57)、[监听](../../../raw/current/scripts/prefabs/shadowhand.lua#L181-L185) |
| 标签与其他消费者 | 公共侧设 `HASHEATER`、`staffstar`，热星另设 `cooker`、`daylight`、`heatstar`；这些标签不代替主模拟组件。查理试炼的目标筛选接受 `staffstar`，实际选中还取决于该筛选条件。 | [标签](../../../raw/current/scripts/prefabs/stafflight.lua#L112-L134)、[试炼筛选](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L292-L323) |

## 常见陷阱与限制

- 不要把火球的光照半径当作取暖、烹饪或精神光环的作用范围；这些能力由独立组件处理。
- 熄灭开始不等于实体即时移除：`post`、停止声音及延时移除分阶段发生，期间脚本没有逐一移除玩法组件。[退出](../../../raw/current/scripts/prefabs/stafflight.lua#L9-L16)
- 火球配置是 `is_fx=false`；不能照搬同工厂的 [[reference/prefabs/staffcoldlight#月台附属变体-staffcoldlightfx|月台附属冷星]] 生命周期。`GetTimeAlive` 的跨读档相位连续性也未由当前脚本保证。[工厂注释](../../../raw/current/scripts/prefabs/stafflight.lua#L1-L3)

## 验证记录

- 源码审阅：2026-11-02，Steam buildid 26335394；核验 `emberlight` 的工厂参数、主模拟组件、法术选项／花费／实际生成和移除路径；全 `raw/current/scripts` 搜索完整注册名及 `emberlight` 前缀，区分薇洛依赖、图鉴数据和生成调用。
- 2026-11-04 批 C 独立复核：重新对读 `stafflight.lua` 全文，补核菜单所需 `ember_master` 标签与技能条目门（`willow_ember` L942、`skilltree_willow` L110-L123）、法术简中名，并纠正“计时器到期调用 ForceExtinguish”的表述；其余锚点按 `name`／赋值行起算。具体组件效果、素材内部、其他模组动态生成及读档相位未运行验证。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-light-index|灯光、火焰与音效代理]]。
