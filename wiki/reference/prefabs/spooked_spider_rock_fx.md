---
title: "惊吓蜘蛛虫群特效（spooked_spider_rock_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "万圣夜工作惊吓的表现层 FX：挖矿碎裂蹦蜘蛛动画、挖坟蠕虫动画，由 spooked 组件 TryCustomSpook/TryForcingCustomSpook 在工作物位置生成，联网可见、动画播完自删。"
aliases: ["spooked_spider_rock_fx", "spooked_worms_fx", "spooked_fx", "惊吓特效", "万圣夜惊吓", "挖矿惊吓"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/spooked_fx.lua", "components/spooked.lua", "prefabs/spiderhole.lua", "prefabs/mound.lua", "prefabs/leafpile.lua", "prefablist.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 惊吓蜘蛛虫群特效（spooked_spider_rock_fx）

## 快速判断

- 职责与适用：万圣夜"工作惊吓"的**通用表现层 FX**：两个注册名——挖矿系 `spooked_spider_rock_fx`（蹦蜘蛛动画）、挖掘系 `spooked_worms_fx`（蠕虫动画）；联网实体（AddNetwork）＋FX/NOCLICK 标签，无任何玩法组件。[工厂](../../../raw/current/scripts/prefabs/spooked_fx.lua#L20-L48)
- 优先使用：查"挖矿/挖坟被吓一跳的表现"选本页；生成入口是 spooked 组件的 TryCustomSpook／TryForcingCustomSpook，积累/动作/精神机制见组件页。
- 重要副作用：音效由**主模拟侧** OnEntityWake 首次唤醒时播放一次（回调自清，OnEntitySleep 亦清空两个回调）；动画单参播放、未显式传循环参数，animover 自删。[唤醒音效](../../../raw/current/scripts/prefabs/spooked_fx.lua#L14-L18)
- 修改边界：源码注释（V2C）说明不使用 fx.lua 工厂是因为这些动画较长；惊吓的积累与后果全部在 spooked 组件与其消费者，fx 本身只是画面与声音。[注释](../../../raw/current/scripts/prefabs/spooked_fx.lua#L1)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `spooked_spider_rock_fx` | MakeFx 共同工厂：CreateEntity → Transform＋AnimState＋SoundEmitter＋Network → bank/build＝build 参数、播 anim（单参）→ FX/NOCLICK 标签 → SetPristine → 主模拟侧 OnEntityWake/OnEntitySleep＋animover Remove＋persists=false | build/bank `spider_rock_fx`、anim `spiders_spawn`、音效 `hallowednights2025/spooks/spiders`；挖矿破坏点用。[注册](../../../raw/current/scripts/prefabs/spooked_fx.lua#L53) |
| `spooked_worms_fx` | 同工厂 | build/bank `worms_fx`、anim `worms_spawn`、音效 `hallowednights2025/spooks/worms`；挖坟/翻叶堆用。[注册](../../../raw/current/scripts/prefabs/spooked_fx.lua#L54) |

两注册名均无 NAMES 条目，标题中文为已核验效果描述，非官方译名。注意文件名 spooked_fx.lua 与 prefablist 登记的 "spooked_fx"（[prefablist.lua:1211](../../../raw/current/scripts/prefablist.lua#L1211)）都不是实际注册名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：spooked（被吓者侧） | TryCustomSpook(source, fxprefab, mult)：抽中后在 source 位置生成指定 fx、10 帧后推 `spooked` 事件并清零积累；TryForcingCustomSpook 绕过抽取强制生成（免疫检查先行） | 组件挂在玩家（服务端）；fx 生成与事件推送都在主模拟 | [[reference/components/spooked]]；[TryCustomSpook](../../../raw/current/scripts/components/spooked.lua#L102-L117)、[TryForcingCustomSpook](../../../raw/current/scripts/components/spooked.lua#L119-L129) |
| 生成者：挖矿系 | spiderhole 岩壳变档（LOW 倍率）与挖穿（HIGH 倍率）两次调用 | 工作者需有 spooked 组件；抽不中则无 fx | [spiderhole L70](../../../raw/current/scripts/prefabs/spiderhole.lua#L66-L81)；其余挖矿点见组件页消费者表 |
| 生成者：挖掘系 | mound 挖坟回调（DIG 倍率）；leafpile 翻找在 trick 掉落前先掷 LEAFPILE_TRICK_SPOOK_CHANCE，命中才强制触发 | leafpile 走 TryForcingCustomSpook、无视积累门槛 | [mound](../../../raw/current/scripts/prefabs/mound.lua#L123-L126)、[leafpile](../../../raw/current/scripts/prefabs/leafpile.lua#L228-L232) |
| 资源 | anim/spider_rock_fx.zip、anim/worms_fx.zip；音效 hallowednights2025/spooks/spiders 与 /worms | 声明与使用已核验，包内容未解包 | [注册行](../../../raw/current/scripts/prefabs/spooked_fx.lua#L53-L54) |

## 典型接入

### 复用"工作触发惊吓表现"

- 前提与选择：在自己的工作成功回调里调 `worker.components.spooked:TryCustomSpook(inst, fx注册名, mult)`；先确认 worker 有 spooked 组件、source 仍有 Transform。
- 生效与撤销：抽中才生成 fx 并在 10 帧后推事件；想无视概率用 TryForcingCustomSpook（仍尊重免疫）。fx 播完自删，无需清理。
- 原版依据：spiderhole 的两次调用展示 LOW/HIGH 倍率按工作阶段区分（[实现](../../../raw/current/scripts/prefabs/spiderhole.lua#L66-L81)）。

```lua
-- 在自定义可工作物的破坏回调里（主模拟侧）：
if worker.components.spooked ~= nil then
    worker.components.spooked:TryCustomSpook(inst, "spooked_spider_rock_fx", TUNING.MINE_SPOOKED_MULT_LOW)
end
```

- 调整与验证：倍率含义与积累公式见 spooked 页"积累与概率如何计算"；不要每帧调用（每次调用都在加积累）。

## 常见陷阱与限制

- **边界："spooked_fx" 不是注册名**：文件名与 prefablist 登记名都是家族统称，`SpawnPrefab("spooked_fx")` 会失败；实际只有两个注册名（[注册行](../../../raw/current/scripts/prefabs/spooked_fx.lua#L53-L54)）。
- **误区：生成 fx 就等于吓到了玩家**：fx 生成与 `spooked` 事件是同一回调里先后两步，但抽取失败时 fx 也不生成；强制路径只绕过抽取不绕过免疫。触发≠动作执行≠扣精神，三层判定见组件页。
- **边界：音效回调只挂主模拟侧**：OnEntityWake/OnEntitySleep 在 mastersim 分支内设置（[L37-L42](../../../raw/current/scripts/prefabs/spooked_fx.lua#L37-L42)），客户端本地不播；联网实体按服务端声音路径传播。
- **边界：动画是否重复未核验**：`PlayAnimation(anim)` 单参播放；设计预期是播完 animover 自删，别依赖循环。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；55 行工厂全文、spooked 组件两个自定义接口、spiderhole/mound/leafpile 生成点、prefablist 登记名、NAMES 检索（无命中）已核验；动画包内容、stalagmite/rocks/tree_rocks 等其余挖矿点仅 grep 命中未逐行读、晚加入时序未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-shadow-index|暗影生物、裂隙生物与附着]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
