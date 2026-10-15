---
title: "狂欢节庆典特效族（carnival_confetti_fx 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "狂欢节活动七名通用表一次性动画特效：炮庆三连（彩纸／闪光／彩带，条目自带炮声）、盲盒雕塑开包三连（按赛季动态拼名）与射击游戏投射物伴生特效；全部主模拟落点生成，表现协议归通用 FX 工厂。"
aliases: ["carnival_confetti_fx", "carnival_sparkle_fx", "carnival_streamer_fx", "carnival_unwrap_fx", "carnival_unwrap_fx_s2", "carnival_unwrap_fx_s3", "carnivalgame_shooting_projectile_fx", "狂欢节特效", "狂欢节彩纸", "彩带特效", "开包特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/carnival_cannons.lua", "prefabs/carnivaldecor_figure.lua", "prefabs/carnivalgame_shooting.lua", "prefabs/carnivalgame_herding.lua", "components/golfpropitem.lua", "stategraphs/SGcarnivalgame_herding_chick.lua", "prefabutil.lua", "prefabs/carnivalgame_golfball.lua", "prefabs/carnivalgame_golf_tee.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 狂欢节庆典特效族（carnival_confetti_fx 等）

## 快速判断

- 职责与适用：七名注册同属 fx.lua 通用表的一次性动画特效（代理＋本地动画协议见 [[reference/mechanisms/generic-fx-factory#关键流程]]），按消费方分三组：狂欢节炮庆三连、盲盒雕塑开包三连、射击游戏投射物伴生；`carnival_confetti_fx` 另被高尔夫道具回传与小鸡赛跑夺冠复用。本页回答"谁、何时、在何处生成哪个名"；纯表现，无玩法组件。
- 优先使用：查生成点与触发窗口看下方"注册与构造"与"生成者导航"；父挂／落点、音效延迟、三类回收等协议一律链接 [[reference/mechanisms/generic-fx-factory#常见陷阱与限制]]，本页不复制正文。
- 重要副作用：四名带 `sound` 字段（`summerevent/cannon/fire1/2/3`），由特效本地实体在落点播放一次；开包三名条目无声——雕塑部署的 `packaged` 音效由雕塑本体播放。[四声条目](../../../raw/current/scripts/fx.lua#L2334-L2383)、[雕塑播放](../../../raw/current/scripts/prefabs/carnivaldecor_figure.lua#L149-L149)
- 修改边界：检索到的生成点全部在主模拟回调中 `SpawnPrefab` 后 `SetPosition` 落点，无父挂；专用服务器只建隐形代理无表现（[工厂边界](../../../raw/current/scripts/prefabs/fx.lua#L127-L136)）。动画时长与实际外观未核验。

## 注册与构造

七名均为通用表条目，由加载器批量注册成 Prefab；除 `fn` 与字段差异外无专属工厂代码。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `carnival_confetti_fx` | 通用表条目；`fn = FinalOffset1`（`SetFinalOffset(1)`，[定义](../../../raw/current/scripts/fx.lua#L1-L3)） | bank/build `carnival_confetti`、anim `win`、sound `fire1`；炮庆／高尔夫回传／小鸡赛跑三链共用。[条目](../../../raw/current/scripts/fx.lua#L2334-L2339) |
| `carnival_sparkle_fx` | 同上 | anim `sparkle`、sound `fire2`；仅狂欢节炮生成。[条目](../../../raw/current/scripts/fx.lua#L2342-L2347) |
| `carnival_streamer_fx` | 同上 | anim `streamer`、sound `fire3`；仅狂欢节炮生成。[条目](../../../raw/current/scripts/fx.lua#L2350-L2355) |
| `carnival_unwrap_fx` | 同上，无声 | anim `unwrap`；盲盒雕塑赛季 1 用名。[条目](../../../raw/current/scripts/fx.lua#L2358-L2362) |
| `carnival_unwrap_fx_s2` | 同上，无声 | anim `unwrap_yellow`（源码注释 yellow box）；赛季 2。[条目](../../../raw/current/scripts/fx.lua#L2365-L2369) |
| `carnival_unwrap_fx_s3` | 同上，无声 | anim `unwrap_red`（源码注释 red box）；赛季 3。[条目](../../../raw/current/scripts/fx.lua#L2372-L2376) |
| `carnivalgame_shooting_projectile_fx` | 同表条目，无 `fn`（不设 FinalOffset） | bank/build `carnivalgame_shooting_projectile`、anim `fx1`、sound `fire3`。[条目](../../../raw/current/scripts/fx.lua#L2379-L2383) |

标题中文为已核验效果描述（bank／anim 名与生成者语境）；`languages/chinese_s.po` 定向检索无这七个注册名的显示名键，非官方译名。名称依据变化需复核时以本节锚点为准。

## 生成者导航（触发窗口与落点）

| 生成者 | 触发窗口 | 落点与端 | 证据 |
|---|---|---|---|
| 狂欢节炮 `carnivalcannon_confetti`／`_sparkle`／`_streamer`（Wiki 待建，源码见证据列） | `FireCannon` 后 10 帧（约 0.33 秒）调 `SpawnFx`；开炮由玩家激活（activatable）或投入 `carnival_gametoken`（trader）触发，随后 4 秒冷却 | 炮位落点；主模拟（组件在 ismastersim 分界后安装） | [defs→fx 名](../../../raw/current/scripts/prefabs/carnival_cannons.lua#L4-L16)、[延时生成](../../../raw/current/scripts/prefabs/carnival_cannons.lua#L50-L57)、[激活两路](../../../raw/current/scripts/prefabs/carnival_cannons.lua#L65-L72)、[端分界](../../../raw/current/scripts/prefabs/carnival_cannons.lua#L123-L125) |
| 高尔夫道具回传 `GolfPropItem:TeleportBack`（[[reference/components/golfpropitem#典型接入]]） | 道具滚出边界，组件 0.5 秒周期 `CheckTeleport` 命中 | 回传点（记存的 teleport 点或边界中心，y=0）；主模拟（组件仅主模拟侧安装，[例](../../../raw/current/scripts/prefabs/carnivalgame_golf_tee.lua#L123-L127)） | [回传生成](../../../raw/current/scripts/components/golfpropitem.lua#L53-L56)、[出界检查](../../../raw/current/scripts/components/golfpropitem.lua#L59-L62)、[周期](../../../raw/current/scripts/components/golfpropitem.lua#L66-L70) |
| 小鸡赛跑夺冠 SGcarnivalgame_herding_chick（Wiki 待建，源码见证据列） | 状态 `arived_home`（小鸡归栏获胜）timeline 第 28 帧（约 0.93 秒） | 小鸡位置落点；主模拟 SG | [状态](../../../raw/current/scripts/stategraphs/SGcarnivalgame_herding_chick.lua#L55-L55)、[生成帧](../../../raw/current/scripts/stategraphs/SGcarnivalgame_herding_chick.lua#L72-L72)、[SG 安装](../../../raw/current/scripts/prefabs/carnivalgame_herding.lua#L344-L358) |
| 盲盒雕塑 `carnivaldecor_figure`／`_season2`／`_season3`（Wiki 待建，源码见证据列） | kit 部署成功，`deployablekititem_ondeploy` 推 `onbuilt` | 雕塑位置落点；主模拟（部署回调） | [拼名生成](../../../raw/current/scripts/prefabs/carnivaldecor_figure.lua#L146-L147)、[onbuilt](../../../raw/current/scripts/prefabs/carnivaldecor_figure.lua#L139-L149)、[onbuilt 推送](../../../raw/current/scripts/prefabutil.lua#L67-L73) |
| 射击游戏投射物 `carnivalgame_shooting_projectile`（弹体生成链见 [[reference/components/carnivalgameshooter#快速判断]]） | 命中回调 `projectile_OnHit` 立即生成；回合结束余弹监听 `carnivalgame_endofround` 后延时 0.2～0.8 秒 `self_destruct` 时生成 | 弹体当前位置；主模拟（complexprojectile 回调） | [命中](../../../raw/current/scripts/prefabs/carnivalgame_shooting.lua#L805-L811)、[自毁](../../../raw/current/scripts/prefabs/carnivalgame_shooting.lua#L796-L798)、[回合事件](../../../raw/current/scripts/prefabs/carnivalgame_shooting.lua#L288-L295) |

unwrap 拼名规则：`"carnival_unwrap_fx"..suffix`，suffix 由雕塑注册时的 `carnival_season`（1／2／3）决定为 `""`／`"_s2"`／`"_s3"`；三名各自先在其 Prefab 的 prefabs 数组声明。[季赋值](../../../raw/current/scripts/prefabs/carnivaldecor_figure.lua#L209-L209)、[三工厂](../../../raw/current/scripts/prefabs/carnivaldecor_figure.lua#L241-L250)、[三声明](../../../raw/current/scripts/prefabs/carnivaldecor_figure.lua#L17-L29)、[三注册](../../../raw/current/scripts/prefabs/carnivaldecor_figure.lua#L270-L276)

仅依赖声明、无生成点的位置：carnival_cannons 的 defs→prefabs [组装](../../../raw/current/scripts/prefabs/carnival_cannons.lua#L171-L171)、carnivaldecor_figure 雕塑 [三声明](../../../raw/current/scripts/prefabs/carnivaldecor_figure.lua#L17-L29)、carnivalgame_herding 小鸡 [chick_prefabs](../../../raw/current/scripts/prefabs/carnivalgame_herding.lua#L28-L30)、carnivalgame_shooting 弹体 [prefabs](../../../raw/current/scripts/prefabs/carnivalgame_shooting.lua#L26-L26)——声明用于保证该名可生成，不构成生成证据。

资源复用辨析：carnivalgame_golfball 的 `AddOverrideBuild("carnival_streamer")` 是动画 build 叠加，与 `carnival_streamer_fx` 注册无关。[位置](../../../raw/current/scripts/prefabs/carnivalgame_golfball.lua#L270-L270)

## 典型接入

### 在自定义 Prefab 中借用炮庆彩纸

- 前提与选择：服务端逻辑中落点生成；本 Prefab 的 `prefabs` 表声明该名（原版各生成者均如此声明），动画资源由 fx.lua 条目自带，无需自备。
- 生效与撤销：非专服各端一帧后出表现、动画播完自删，生成方无需清理（协议见 [[reference/mechanisms/generic-fx-factory#典型接入]]）。

```lua
local prefabs = { "carnival_confetti_fx" }

local function Celebrate(inst)
    local fx = SpawnPrefab("carnival_confetti_fx")
    fx.Transform:SetPosition(inst.Transform:GetWorldPosition())
end
```

- 调整与验证：换名即换效果（同族七名协议一致）；音效随条目 `sound` 字段播放，生成侧不可关掉，需静音走工厂页的协议限制。

## 常见陷阱与限制

- **误区：unwrap 三名是礼物包开包烟**：礼盒／种子包开包烟 `carnival_seedpacket_unwrap` 是另一注册（bank/build 复用 `bundle`），见 [[reference/prefabs/bundle_unwrap#注册与构造]]；本族三名 bank/build 为 `carnival_unwrap`，仅盲盒雕塑部署触发。
- **误区：`carnival_confetti_fx` 即大赛彩纸 `confetti_fx`**：两独立注册；本名生成点是炮庆／高尔夫回传／小鸡赛跑，大赛彩纸（粒子发射器实现）见 [[reference/prefabs/confetti_fx#常见陷阱与限制]]（该页亦载此辨析）。
- **边界：开包无声是条目行为**：unwrap 三名条目无 `sound` 字段；`dontstarve/common/together/packaged` 由雕塑本体在 `onbuilt` 中播放，移除特效不影响该音效。[播放点](../../../raw/current/scripts/prefabs/carnivaldecor_figure.lua#L149-L149)
- **边界：生成点全部在主模拟回调**（组件方法、SG timeline、部署事件）：纯客户端不生成；专用服务器只有隐形代理（[[reference/mechanisms/generic-fx-factory#常见陷阱与限制]]）。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。七条目逐项核验 bank/build/anim/sound/fn（fx.lua L2334-L2383）与 `FinalOffset1` 定义（fx.lua L1-L3）。生成点全域反查：`raw/current/scripts` 递归（含根级、prefabs/、components/、stategraphs/，`*.lua`），先用七个完整注册名，再用去后缀短名 `carnival_confetti`／`carnival_sparkle`／`carnival_streamer`／`carnival_unwrap`／`shooting_projectile` 二次检索覆盖动态拼名，另查 `.xml`／`.po`／`.txt` 非 Lua 文件无引用；命中本页所列 5 生成链＋4 声明，无其余生成点。`chinese_s.po` 定向检索七名无显示名键。未覆盖：动画实际外观与时长、炮庆三连在活动事件中的摆放布局。
- 游戏运行：未运行。
- 结构检查：2026-10-31 validate_page.py 页面级通过（standard 档，86 行／11.2 KiB，上限 200 行／16 KiB）；validate_source_links.py --strict 通过（36 引用 0 错 0 警）。叶级索引（fx-table-index）登记待办，未执行。Lua 示例经 parser_probe --syntax-only 解析通过。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]。
