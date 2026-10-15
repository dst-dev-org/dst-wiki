---
title: "月坛破碎演出（moon_altar_break 等 3 注册）"
created: "2026-11-05"
updated: "2026-11-07"
type: prefab
summary: "月亮装置竣工倒计时破碎时三种天体祭坛的塌落 FX：工厂按祭坛动画库名拼 _break 注册，用 AddOverrideBuild 叠本体 build 播 break；唯一生成点是 moon_device 清场回调按祭坛 prefab 名映射生成，并连带天体裂缝与祭坛移除。"
aliases: ["moon_altar_break", "moon_altar_claw_break", "moon_altar_crown_break", "月坛破碎", "天体祭坛破碎", "天体贡品破碎", "天体圣殿破碎", "月坛塌落"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/moon_altar_break.lua", "prefabs/moon_device.lua", "prefabs/moon_altar.lua", "mainfunctions.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 月坛破碎演出（moon_altar_break 等 3 注册）

## 快速判断

- 职责与适用：3 个注册都是**天体祭坛在月亮装置竣工倒计时结束时的塌落演出**：月亮装置（level 3 的 `moon_device`）建成后 9.5 秒执行破碎序列，清场回调在半径 15 内找到三种祭坛本体（`moon_altar`／`moon_altar_cosmic`／`moon_altar_astral`），按映射生成对应破碎 FX 并移除祭坛、留下天体裂缝。注册处：[moon_altar_break.lua#L40-L42](../../../raw/current/scripts/prefabs/moon_altar_break.lua#L40-L42)（文件头注释列名 [L39](../../../raw/current/scripts/prefabs/moon_altar_break.lua#L39-L39)）
- 优先使用：查"月坛被月亮装置砸碎的动画"或三个 `_break` 后缀注册选本页；月亮装置自身的三阶段破碎（`moon_device_break_stage1/2/3`）分属 [[reference/prefabs/moon_device_pillar|月装置支柱、顶盖、一阶段破碎与天体裂缝裂纹层]]（stage1）与 [[reference/prefabs/moonstorm_glass_fx|月之装置破碎与月暴玻璃爆裂]]（stage2/3），不在本页。
- 重要副作用：壳是联网自播动画（AnimState／SoundEmitter 在壳上，客户端 `SetPristine` 后早退）；`animover` 自删，**未置 `persists=false`、无兜底定时**——与同类焦壳工厂（蘑菇树焦壳双保险）不同，见[清理辨析](#清理辨析)。生成时对每座祭坛额外落一枚 `moon_fissure`（天体裂缝宿主，带月相裂纹层的实体）。
- 修改边界：只覆盖 3 注册的工厂、映射与清理；月亮装置的建造链（construction1/2）、天体英雄 Boss 战、祭坛本体玩法（ hammer、连接、修理）只在破碎链需要处点到。锤毁祭坛（`onhammered`）只生成 `collapse_small`＋`moon_fissure`，**不生成本页 FX**（[moon_altar.lua#L282-L294](../../../raw/current/scripts/prefabs/moon_altar.lua#L282-L294)）。
- 名称：宿主官方名 `MOON_ALTAR`＝"天体祭坛"、`MOON_ALTAR_COSMIC`＝"天体贡品"、`MOON_ALTAR_ASTRAL`＝"天体圣殿"（锚点见[验证记录](#验证记录)）；3 个 FX 注册名均无 `STRINGS.NAMES.*` 键，且 **FX 名沿用的是祭坛动画库名（crown／claw）而非祭坛注册名**——检索时注意"天体贡品→`moon_altar_crown_break`、天体圣殿→`moon_altar_claw_break`"的错位。

## 注册与构造

`make_break_fx(name)` 以**祭坛动画库名**为输入：`break_prefab_name = name.."_break"`；assets 带本体 build 与 break 动画两个 zip。壳构造：Transform＋AnimState＋SoundEmitter＋Network → bank／build 均为 `*_break` → **`AddOverrideBuild(name)` 把祭坛本体的 build 叠加在 break 动画上**（塌落的碎块显示为对应祭坛的外观）→ 播 `break` 一次 → `FX` 标签（无 NOCLICK）→ `SetPristine` → 客户端早退 → 主模拟仅挂 `animover` 自删（[工厂](../../../raw/current/scripts/prefabs/moon_altar_break.lua#L10-L34)）。

| FX 注册名 | 输入 name（动画库） | 对应祭坛本体（prefab → 官方名） | 映射证据 |
|---|---|---|---|
| `moon_altar_break` | `moon_altar` | `moon_altar`（bank/build `moon_altar`）→ 天体祭坛 | [MakeAltar 注册](../../../raw/current/scripts/prefabs/moon_altar.lua#L721-L732) |
| `moon_altar_crown_break` | `moon_altar_crown` | `moon_altar_cosmic`（bank/build `moon_altar_crown`）→ 天体贡品 | [MakeAltar 注册](../../../raw/current/scripts/prefabs/moon_altar.lua#L733-L743) |
| `moon_altar_claw_break` | `moon_altar_claw` | `moon_altar_astral`（bank/build `moon_altar_claw`）→ 天体圣殿 | [MakeAltar 注册](../../../raw/current/scripts/prefabs/moon_altar.lua#L744-L754) |

映射表在消费方 moon_device.lua：`ALTAR_FX_PREFABS = { moon_altar = "moon_altar_break", moon_altar_cosmic = "moon_altar_crown_break", moon_altar_astral = "moon_altar_claw_break" }`（[表](../../../raw/current/scripts/prefabs/moon_device.lua#L239-L244)）——键是祭坛 **prefab 名**、值才是 FX 名，crown／claw 与 cosmic／astral 的错位由此而来。

## 关联机制导航

### 破碎链：从竣工到清场（唯一生成路径）

月亮装置是玩家建造链的终点产物（`moon_device_construction1` → `moon_device_construction2` → `moon_device`，[construction_data](../../../raw/current/scripts/prefabs/moon_device.lua#L52-L55)）。level 3 实体每次构造（建造完成 ReplacePrefab 或**读档重建**均算）都会排定破碎：`BREAK_DELAY - 1`（8.5 秒）播陨石预警声、`BREAK_DELAY`（9.5 秒）进 `breaksequence`（[排定](../../../raw/current/scripts/prefabs/moon_device.lua#L457-L468)；`OnLoad` 不重置也不取消该倒计时，[OnLoad](../../../raw/current/scripts/prefabs/moon_device.lua#L379-L383)）。

| 步骤 | 生成／移除 | 证据 |
|---|---|---|
| `breaksequence` | 生成 `alterguardian_phase1fallfx`（天体英雄坠落层，归 [[reference/prefabs/alterguardian_spintrail_fx|天体英雄轨迹]] 页），9 帧后进 `break_device` | [L329-L335](../../../raw/current/scripts/prefabs/moon_device.lua#L329-L335) |
| `stage3_break` | 装置顶盖位置生成 `moon_device_break_stage3`，1 帧后移除顶盖实体 | [L190-L194](../../../raw/current/scripts/prefabs/moon_device.lua#L190-L194) |
| `stage2_break` | 四根支柱各自位置（带原朝向）生成 `moon_device_break_stage2`，1 帧后移除支柱 | [L196-L204](../../../raw/current/scripts/prefabs/moon_device.lua#L196-L204) |
| `stage1_break` → `ClearArea` | 震屏；半径 15 全量检索并分派处置；装置位置生成 `moon_device_break_stage1`＋`moon_geyser_explode`＋多枚焦痕 | [L289-L305](../../../raw/current/scripts/prefabs/moon_device.lua#L289-L305) |
| **祭坛分派（本页 FX 生成点）** | `FindEntities` 命中 `ALTAR_FX_PREFABS` 三键时：在**祭坛坐标**生成对应 `*_break` FX＋在祭坛地面生成 `moon_fissure`（天体裂缝）→ **移除祭坛本体** | [ClearArea L251-L281](../../../raw/current/scripts/prefabs/moon_device.lua#L251-L281) |
| 收尾 | 1 帧后生成 `alterguardian_phase1` Boss（prespawn_idle）并移除月亮装置；装置先播 `spawn_pre` 声 | [break_device L320-L327](../../../raw/current/scripts/prefabs/moon_device.lua#L320-L327) |

`ClearArea` 对非祭坛目标的处置（同一回调，理解 FX 触发语境用）：`smashable` 血量目标直接 `Kill`；可工作目标除月玻璃外落 `collapse_small` 再 `Destroy`；5.5 半径内存活生物吃 `ALTERGUARDIAN_PHASE1_ROLLDAMAGE` 一击；散落物品按陨石抛掷逻辑弹飞（[L264-L278](../../../raw/current/scripts/prefabs/moon_device.lua#L264-L278)）。

### 清理辨析

- 自删路径只有 `animover`：动画播完即 Remove（实际时长未核验）；联网壳无本地子实体、无父挂（[工厂尾](../../../raw/current/scripts/prefabs/moon_altar_break.lua#L31-L31)）。
- **`persists` 未置 false**：与 forcefieldfx 同属"未置 persists"的存档语义——FX 在场窗口较短，正常流程通常不会入存档；但壳带 `FX` 标签（可被点击命中）且离屏动画睡眠会延后 animover，此窗口内可能入档。读档时 `SpawnSaveRecord` 会重新 `SpawnPrefab`（[SpawnSaveRecord](../../../raw/current/scripts/mainfunctions.lua#L464-L474)），工厂再次 `PlayAnimation("break")` 并装回 `animover`，不是"静止第一帧"；离屏睡眠对播放／退场的实际影响未运行核验。复用或模改时建议补 `persists = false` 或兜底 Remove。
- 生成方不持有引用：`ClearArea` 生成后即移除祭坛、不记字段，无显式 Kill 协议。

## 典型接入

### 手动播放一次祭坛塌落

- 前提与选择：3 注册位置定位＋自删，直接 `SpawnPrefab` 即用；塌落块外观由注册名决定（三种祭坛各一），与现场是否真有祭坛无关。
- 生效与撤销：主模拟生成即可；若复刻"祭坛消失留裂缝"，裂缝（`moon_fissure`）与祭坛移除须调用方自理——FX 不带任何结算。
- 原版依据：[ClearArea 祭坛分支](../../../raw/current/scripts/prefabs/moon_device.lua#L256-L263)。

```lua
-- 服务端：在 (x, y, z) 播一次天体圣殿（claw）塌落
local fx = SpawnPrefab("moon_altar_claw_break")
if fx ~= nil then
    fx.Transform:SetPosition(x, y, z)
end
```

- 调整与验证：替换点为注册名（3 选 1）；想要连带裂缝与移除祭坛，复刻 ClearArea 的三连写法。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：`moon_altar_crown_break` 属于"月亮王冠"。** crown／claw 是祭坛**动画库**名（`moon_altar_crown`＝天体贡品的 build、`moon_altar_claw`＝天体圣殿的 build），不是可穿戴王冠或爪形物的 FX；按祭坛 prefab 名检索时走 `ALTAR_FX_PREFABS` 映射。[映射表](../../../raw/current/scripts/prefabs/moon_device.lua#L239-L244)
- **边界：唯一生成点是月亮装置清场。** 全 `raw/current/scripts` 反查三注册名仅命中 moon_altar_break.lua（注册）与 moon_device.lua（声明＋映射＋生成）；锤毁祭坛、作祟、燃烧等其它破坏路径都不生成它们。[反查记录](#验证记录)
- **边界：壳未置 `persists = false` 且无兜底。** 同族一次性 FX 多有双保险（蘑菇树焦壳动画长度兜底、蛛网焦壳同款）；本注册只靠 animover——离屏睡眠会延后移除，并留下极小的入档窗口。[清理辨析](#清理辨析)
- **边界：塌落外观由 `AddOverrideBuild` 决定，与皮肤无关。** 叠加的是祭坛本体**基础 build 名**；工厂不接受皮肤参数，祭坛若有皮肤也不反映在碎块上。[工厂](../../../raw/current/scripts/prefabs/moon_altar_break.lua#L18-L21)
- **边界：FX 与 `moon_fissure` 是两回事。** 每座被砸的祭坛额外落一枚 `moon_fissure`（可存档的裂缝宿主，裂纹层 `moon_fissure_fx` 按月相显隐），归 moon_device_pillar 页；本页 3 注册只是塌落动画本体。[连带](../../../raw/current/scripts/prefabs/moon_device.lua#L262-L262)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/moon_altar_break.lua` 全文（42 行）；完整读取 `prefabs/moon_device.lua` 的破碎链（ClearArea／stage1-3_break／breaksequence／break_device／do_boss_spawn／BREAK_DELAY 排定／OnSave／OnLoad）；按需读取 `prefabs/moon_altar.lua` 的 MakeAltar 工厂与三祭坛注册（bank／build 对照 crown／claw 错位）、onhammered 破坏路径。
- 穷举复核：对 `moon_altar_break`／`moon_altar_claw_break`／`moon_altar_crown_break` 逐一全域 `grep -rn "注册名" --include=*.lua` 反查——命中仅 `prefabs/moon_altar_break.lua`（注册）、`prefabs/moon_device.lua`（prefabs 声明 L23-L25＋ALTAR_FX_PREFABS 映射 L241-L243＋生成 L261）与 `prefablist.lua` 文件清单；**生成点唯一**（ClearArea 一处，动态值来自映射表）。三祭坛本体 prefab 名经 `prefabs/moon_altar.lua` 注册清单实读核对（moon_altar／moon_altar_cosmic／moon_altar_astral）。
- 名称核验：`languages/chinese_s.po` 定向检索——`MOON_ALTAR`＝"天体祭坛"（[po](../../../raw/current/scripts/languages/chinese_s.po#L345744-L345747)）、`MOON_ALTAR_ASTRAL`＝"天体圣殿"（[po](../../../raw/current/scripts/languages/chinese_s.po#L345749-L345752)）、`MOON_ALTAR_COSMIC`＝"天体贡品"（[po](../../../raw/current/scripts/languages/chinese_s.po#L345759-L345762)）；`MOON_ALTAR_BREAK`／`MOON_ALTAR_CLAW_BREAK`／`MOON_ALTAR_CROWN_BREAK` 均无 `STRINGS.NAMES.*` 键，标题"月坛破碎演出"为已核验效果描述，非官方译名。
- 分档说明：`size_class: standard`——3 注册共享一个 42 行小文件工厂、单一生成链；破碎链虽横跨 moon_device 多函数，但按步骤表一层展开即可完整覆盖，未选 complex 因不存在多状态机或多种清理模式。
- 未覆盖：月亮装置建造玩法与材料计划、天体英雄 Boss 战（三阶段）、`moon_fissure`／`moon_fissure_fx` 裂缝层（归 moon_device_pillar 页）、`moon_device_break_stage1/2/3`（分别归 moon_device_pillar 与 moonstorm_glass_fx 页）、祭坛连接与月脉冲（归 moon_altar_link_fx 页）、`moon_altar_link_contained`（归 king_cane_fx 页）、动画包内容与观感。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

- 第 15 轮补正（2026-11-07）：读档改重新 `SpawnPrefab` 并重播 `break`，补 `SpawnSaveRecord` 锚点，删除静止第一帧与 1～2 秒估计；同步维护日期。

## 相关链接

- [[reference/prefabs/moon_device_pillar|月装置支柱、顶盖、一阶段破碎与天体裂缝裂纹层]]（同一次破碎序列的 stage1 与裂缝层；月亮装置本体的关联页）
- [[reference/prefabs/moonstorm_glass_fx|月之装置破碎与月暴玻璃爆裂]]（stage2/3 破碎动画）
- [[reference/prefabs/alterguardian_spintrail_fx|天体英雄轨迹]]（坠落层 `alterguardian_phase1fallfx` 与月喷泉爆发，同序列的演出件）
- [[reference/prefabs/moon_altar_link_fx|月坛连接光效]]（祭坛连接与月脉冲光点）
- 返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]（月坛）
