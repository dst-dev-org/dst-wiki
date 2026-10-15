---
title: "枕头大战坑与兔人火炬（yotr_fightring 族）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "兔人年枕头大战的四种注册＋放置预览：坑本体是擂台控制器，套装部署成坑，钟负责开局／锤毁回收，八支兔人火炬是事件驱动的非持久 FX 壳。"
aliases: ["yotr_fightring", "yotr_fightring_kit", "yotr_fightring_torch", "yotr_fightring_bell", "yotr_fightring_kit_placer", "枕头大战坑", "枕头大战钟", "枕头大战坑套装", "兔人火炬"]
tags: [prefab, fx, building]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/yotr_fightring.lua", "prefabs/torchfire_yotrpillowfight.lua", "prefabs/torchfire.lua", "prefabs/cozy_bunnyman.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 枕头大战坑与兔人火炬（yotr_fightring 族）

## 快速判断

- 职责与适用：`prefabs/yotr_fightring.lua` 一次返回 4 个注册＋1 个 placer：`yotr_fightring`（枕头大战坑，擂台控制器）、`yotr_fightring_kit`（可部署套装）、`yotr_fightring_torch`（**本页主对象：非持久火炬 FX 壳**）、`yotr_fightring_bell`（开赛／锤毁回收的钟），以及 `yotr_fightring_kit_placer`（部署预览，含 8 支预览火炬＋钟）。[注册](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L1105-L1109)
- 优先使用：需要"擂台周围一圈火炬，开场逐支点亮、残局逐支变小、结束熄灭"的演出时，照搬火炬的四个事件（`onplaced`／`turnon`／`gosmall`／`turnoff`）；火炬本身不判定比赛，只负责火苗与灯光。[火炬工厂](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L841-L870)
- 重要副作用：火炬只有 `FX` 标签、没有 `NOCLICK`／physics；火苗是另一个联网实体 `torchfire_yotrpillowfight`，父挂并 `FollowSymbol("wick")`，灯光用 `AttachLightTo` 挂到火炬上。`gosmall` 会把大火换成普通 `torchfire`，之后再 `turnon` 不会恢复大火（`_fire` 已存在）。[点火](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L809-L839)
- 修改边界：只覆盖 4 注册＋placer 的工厂、生成／回收链与火炬事件；枕头大战的参赛者、作弊判定、计分、奖励与 AI 不展开。
- 名称：`YOTR_FIGHTRING`＝"枕头大战坑"、`YOTR_FIGHTRING_BELL`＝"枕头大战钟"、`YOTR_FIGHTRING_KIT`＝"枕头大战坑套装"、`YOTR_FIGHTRING_TORCH`＝"兔人火炬"（[po](../../../raw/current/scripts/languages/chinese_s.po#L353064-L353079)）；placer 无 `STRINGS.NAMES` 键。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `yotr_fightring` | `ringfn()`：Transform／AnimState／SoundEmitter／MiniMapEntity／Network → 图标 `yotr_fightring.png`、bank/build `yotr_fightring`、`idle`、OnGround、背景层、FinalOffset 1 → `birdblocker`／`antlion_sinkhole_blocker`／`yotr_arena` 标签、相机与音乐 netvar → `SetPristine`；客户端监听两个 dirty；主模拟加 `minigame`（`bunnyman_pillowfighting`）／`entitytracker`、监听 `onplaced`／`onremove`、0 帧 `SetUpAuxiliaryObjects`、注册世界擂台表、`OnSave`／`OnLoad`。 | 擂台控制器；不是 FX。生成 8 支火炬与 1 口钟，移除时连带删除。[工厂](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L596-L718) |
| `yotr_fightring_kit` | `kitfn()`：Transform／AnimState／Network、`deploykititem`／`usedeployspacingasoffset`、inventory physics／float、`item` 动画 → `SetPristine`；主模拟加 `deployable`（`DEPLOYMODE.CUSTOM`、`ondeploy=on_kit_deployed`）。 | 部署后生成 `yotr_fightring`、推 `onplaced`、删除自己。[部署](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L729-L737) |
| `yotr_fightring_torch` | `torchfn()`：Transform／AnimState／Network → bank/build `yotr_fightring`、`torch`、FinalOffset 2 → `FX` → `SetPristine`；主模拟监听 `onplaced`／`turnon`／`gosmall`／`turnoff`、`persists=false`。 | **纯 FX 壳**：无 physics／combat／NOCLICK；火苗由子实体提供；不自删，随擂台移除。[火炬](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L841-L870) |
| `yotr_fightring_bell` | `bellfn()`：Transform／AnimState／SoundEmitter／Network → bank/build `yotr_fightring_bell`、`idle`、FinalOffset 2 → `SetPristine`；主模拟加 `activatable`（需手持 `pillow` 标签）、`inspectable`（PLAYING 状态）、`workable`（锤 4 下）、`lootdropper`（掉套装）、监听 `onplaced`／`pillowfight_playhit`、`SetParentRing`／`FinishGame`、`persists=false`。 | 开赛与回收都在这口钟；锤毁时若挂擂台会连擂台一起移除。[钟](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L968-L1023) |
| `yotr_fightring_kit_placer` | `MakePlacer(...)` 直接生成；`ring_placer_postinit` 在非专服 0 帧建 8 支本地预览火炬＋1 口预览钟、设置背景层与自定义 `override_testfn`。 | 只做放置预览；预览实体都是非联网 `FX`。[预览](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L1025-L1109) |

## 擂台、套装与钟

### 从套装到擂台

- 套装来自兔人供奉配方 `Recipe2("yotr_fightring_kit", {boards×2}, TECH.RABBITOFFERING_THREE, {nounlock=true, actionstr="PERDOFFERING"})`（[配方](../../../raw/current/scripts/recipes.lua#L1120-L1120)）。
- 部署回调 `on_kit_deployed` 生成 `yotr_fightring`、放到落点、推 `onplaced`，然后删套装（[部署](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L729-L737)）。
- 擂台 `DoTaskInTime(0, SetUpAuxiliaryObjects)`：建 8 支火炬（`WAITING_FIGHTERS_COUNT=8`，沿半径 `RING_SIZE-0.4` 均布）和 1 口钟；钟 `SetParentRing(inst)`；若擂台已放置，则给火炬补 `onplaced`、给钟补 `onplaced`。[布置](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L546-L562)、[常量](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L17-L22)
- 擂台把自身注册进 `TheWorld.yotr_fightrings`，移除时注销；兔人 `cozy_bunnyman` 用这张表找最近的擂台（[注册](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L473-L480)、[消费者](../../../raw/current/scripts/prefabs/cozy_bunnyman.lua#L75-L90)）。

### 钟的开赛与锤毁

- 激活钟要求玩家手持带 `pillow` 标签的物品；没有则返回 `PILLOWFIGHT_NO_HANDPILLOW`。擂台不空时推 `pillowfight_arenanotclear` 并拒绝。[激活](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L890-L917)
- 激活成功播放 `ring` 动画与钟声，调用 `_parent_ring:OnRingActivated(doer)`，并把 `workable` 暂时关掉。[开赛](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L910-L917)
- 锤 4 下：若挂擂台，在擂台位置生成 `collapse_big`（木材质）、掉落 `yotr_fightring_kit`、移除擂台；无擂台则生成 `collapse_small`、掉落套装、删除自己。[锤毁](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L929-L950)
- 擂台移除时 `OnRingRemoved` 反向删除所有火炬与钟，并给等待中的兔人推 `pillowfight_deactivated`。[清理](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L572-L590)

## 兔人火炬

### 生成与出场

- 擂台生成 8 支火炬；若擂台是"已放置"状态，`make_torch_placed` 按 `(2+2×index)` 帧错开延迟，再生成并推 `onplaced`，形成依次亮起／出现的节奏；未放置则立即生成。[生成](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L527-L544)
- 火炬工厂只播 `torch` 动画并设 `FinalOffset(2)`；`onplaced` 时改播 `spawn_torch` 再推 `torch`（非循环），所以每支火炬的出场动画独立。[出场](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L804-L807)
- 火炬没有 `persists` 存档、没有自删；存亡完全由擂台 `OnRingRemoved` 的显式 `Remove` 决定。

### 四个事件与火苗

| 事件 | 行为 | 证据 |
|---|---|---|
| `onplaced` | 播 `spawn_torch`，再推 `torch` | [L804-L807](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L804-L807) |
| `turnon` | 若 `_fire` 为空：生成 `torchfire_yotrpillowfight`，父挂火炬、`Follower:FollowSymbol("wick", 0, -50, 0)`、`AttachLightTo(torch)` | [L809-L818](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L809-L818) |
| `gosmall` | 先删旧火，再生成普通 `torchfire`，同样父挂／跟随 `wick`／挂灯 | [L820-L832](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L820-L832) |
| `turnoff` | 删除 `_fire` 并置 nil | [L834-L839](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L834-L839) |

火苗实体本身是独立的联网 FX（`torchfire_yotrpillowfight`／`torchfire`），火炬只负责"挂在哪、什么时候换"；灯光由火苗 `AttachLightTo(torch)` 绑到火炬，而不是火炬自带 Light。[火苗注册](../../../raw/current/scripts/prefabs/torchfire_yotrpillowfight.lua#L178-L182)、[通用火苗](../../../raw/current/scripts/prefabs/torchfire.lua#L180-L184)

### 开赛时的点亮与收缩

擂台 `OnRingActivated` 为每支火炬排两个任务（[排程](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L410-L433)）：

- `turnon`：第 `d × (MINIGAME_INTRO_TIME / 8)` 秒，`d=1..8`，`MINIGAME_INTRO_TIME=4`，即开场 4 秒内逐支点亮；
- `gosmall`：第 `MINIGAME_INTRO_TIME + d × (MINIGAME_TIMEOUT_TIME − MINIGAME_INTRO_TIME) / 8` 秒，`MINIGAME_TIMEOUT_TIME=2×SEG_TIME`，即开场后按剩余赛程均分、逐支变小；
- 每次事件都伴随擂台 `torch_swing` 声。

`GoToDeactivateMinigame` 对 8 支火炬逐个 `turnoff`，并取消尚未执行的 `turnon`／`gosmall` 任务；所以提前结束比赛不会留下"到点又亮"的定时器。[收尾](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L207-L228)

## 关联机制导航

| 类型／对象 | 本注册如何使用 | 变体、端与条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `Prefab：yotr_fightring` | 擂台控制器；生成 8 火炬＋1 钟；开赛排程；移除清理 | 只有套装部署生成 | [注册](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L1105-L1105) |
| `Prefab：yotr_fightring_kit` | 部署入口；配方产物 | 兔人供奉配方 | [配方](../../../raw/current/scripts/recipes.lua#L1120-L1120) |
| `Prefab：yotr_fightring_bell` | 开赛／锤毁回收 | 需手持 pillow 才能开赛 | [钟](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L968-L1023) |
| `Prefab：torchfire_yotrpillowfight` | 火炬的大火苗；`gosmall` 后换成普通 `torchfire` | 独立注册，不属本页 4 注册 | [注册](../../../raw/current/scripts/prefabs/torchfire_yotrpillowfight.lua#L178-L182) |
| `世界表：TheWorld.yotr_fightrings` | 擂台注册表；兔人选最近擂台 | 擂台 `OnRemove` 注销 | [消费](../../../raw/current/scripts/prefabs/cozy_bunnyman.lua#L75-L90) |
| `Prefab：collapse_big`／`collapse_small` | 钟被锤毁时的木屑爆裂 | 由钟生成，不属本页 | [锤毁](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L929-L950) |

## 典型接入

### 手动点亮／熄灭一支兔人火炬

- 前提与选择：只在主模拟推事件；客户端没有这些监听。火炬要先放好位置，`turnon` 才会把火苗父挂到它身上。
- 生效与撤销：`turnon` 生成大火；`gosmall` 换成普通火；`turnoff` 删除火。重复 `turnon` 不会叠火（`_fire` 守卫），但 `gosmall` 之后再 `turnon` 也不会恢复大火，要先 `turnoff`。
- 原版依据：[火炬事件](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L804-L839)、[擂台排程](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L410-L433)。

```lua
local torch = SpawnPrefab("yotr_fightring_torch")
torch.Transform:SetPosition(x, 0, z)
torch:PushEvent("onplaced") -- 出场动画
torch:PushEvent("turnon")   -- 挂 torchfire_yotrpillowfight
-- 残局：torch:PushEvent("gosmall")
-- 结束：torch:PushEvent("turnoff")
```

- 调整与验证：替换点是火炬落点与事件时机；大火／小火的切换写死在事件处理里，要改需复制工厂。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：火炬是 8 支，不是 4 支。** `WAITING_FIGHTERS_COUNT=8`，注释明确"这也是火炬数"；按 4 支复刻会少一半。[常量](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L17-L22)
- **边界：火炬没有 `NOCLICK`。** 工厂只加 `FX`，无 physics；它不会挡路，但按 `NOCLICK` 做筛选时不会命中它。[标签](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L853-L853)
- **边界：`gosmall` 之后不能直接 `turnon` 回大火。** `turnon` 只在 `_fire == nil` 时生成大火；`gosmall` 已留下普通 `torchfire`，要恢复得先 `turnoff`。[守卫](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L809-L818)
- **边界：火炬不自删，靠擂台清理。** `persists=false` 只表示不入存档；擂台 `OnRingRemoved` 才逐个 `Remove`。单独生成火炬要自己删。[清理](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L572-L590)
- **边界：钟锤毁会连擂台一起移除。** 挂擂台的钟被锤 4 下后生成 `collapse_big`、掉套装、`_parent_ring:Remove()`；想保留擂台就不要用锤。[锤毁](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L929-L950)
- **边界：预览火炬不是实体注册。** placer 的 8 支火炬与钟由 `CreatePlacerTorch`／`CreatePlacerBell` 本地创建，`FX`＋非联网；不要拿预览名当正式注册。[预览](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L1025-L1102)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/yotr_fightring.lua`（1109 行）的注册、擂台、套装、火炬、钟、placer 段；按需读取 `prefabs/torchfire_yotrpillowfight.lua`（L1-L30、L170-L182）、`prefabs/torchfire.lua`（L182-L182）、`prefabs/cozy_bunnyman.lua`（L75-L90）、`recipes.lua`（L1120）、`tuning.lua`（`BUNNY_RING_*` 与 `SEG_TIME`）。全域反查：`yotr_fightring_torch` 仅 `yotr_fightring.lua:528` 生成；`yotr_fightring` 仅 `:730` 部署生成；`yotr_fightring_bell` 仅 `:556` 生成；placer 仅 `MakePlacer` 注册。台账分诊结论（4 注册＋placer、生成点 kit L730／torch L528／bell L556／recipes L1120、火炬事件与官方名）复核属实；补充实核：8 支火炬、`gosmall` 后 `turnon` 不恢复大火、火炬无 `NOCLICK`、钟锤毁连擂台移除。
- 名称核验：`languages/chinese_s.po` 定向检索——`YOTR_FIGHTRING`＝"枕头大战坑"、`YOTR_FIGHTRING_BELL`＝"枕头大战钟"、`YOTR_FIGHTRING_KIT`＝"枕头大战坑套装"、`YOTR_FIGHTRING_TORCH`＝"兔人火炬"（[L353064-L353079](../../../raw/current/scripts/languages/chinese_s.po#L353064-L353079)）；placer 无 `STRINGS.NAMES` 键。
- 分档说明：`size_class: standard`——4 注册＋placer，但核心是火炬的四个事件与擂台生成链；擂台比赛规则不展开，未升 complex。
- 未覆盖：枕头大战参赛者／作弊／计分／奖励／AI、`minigame` 组件内部、`torchfire` 粒子细节、`collapse_big/small` 工厂；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/collapse_big|坍塌特效]]（钟锤毁时的 `collapse_big`）
- 返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]（枕头大战坑）
