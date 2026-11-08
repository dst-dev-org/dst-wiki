---
title: "伏特羊（lightninggoat）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
summary: "prefabs/lightninggoat.lua 的生物注册：雷击／电击充能、充能期间掉落表切到带电的羊奶、单条 cycles 监听下 3 个世界周期后自动放电（重复充能会叠加监听、加快放电），以及充能状态下的标签、光照、反伤与存档行为。"
aliases: ["lightninggoat", "伏特羊", "伏特加羊", "电羊"]
tags: [prefab, creature]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/lightninggoat.lua", "components/lootdropper.lua", "components/weather.lua", "components/worldstate.lua", "components/electricattacks.lua", "components/combat.lua", "components/herdmember.lua", "components/saltlicker.lua", "stategraphs/SGlightninggoat.lua", "stategraphs/commonstates.lua", "standardcomponents.lua", "entityscript.lua", "componentutil.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 伏特羊（lightninggoat）

## 快速判断

- 职责与适用：群居中立生物，自带 `lightningrod` 标签会“接雷”；未充能时被雷击、电系伤害或 `electrocute` 事件命中后进入充能状态（`charged`），此时掉落表切换到带 `goatmilk` 的带电表，正常单次充能下 3 个世界周期后自动放电。[标签](../../../raw/current/scripts/prefabs/lightninggoat.lua#L210-L218)、[充能](../../../raw/current/scripts/prefabs/lightninggoat.lua#L109-L131)、[放电](../../../raw/current/scripts/prefabs/lightninggoat.lua#L84-L107)
- 优先使用：查奶的获得条件看[充能与放电](#充能与放电)与[掉落表](#掉落表)；查奶本身的三维／用途看 [[reference/prefabs/goatmilk]]。
- 重要副作用：充能不是纯表现——它会换 `build`、开灯、加 `electricattacks`、加电伤免疫、**替换 `lootdropper` 的表**，并把 `chargeleft` 设为 3 后逐周期递减。
- 修改边界：本页只写到决定“能否拿到奶”为止。羊角 `lightninggoathorn` 的用途见 [[reference/prefabs/lightninggoathorn]]；群猎／繁殖等行为不在本页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lightninggoat` | `prefabs/lightninggoat.lua` 的 `fn`，单注册；先做动画／物理／光照／网络，`SetPristine` 后非主模拟返回 | 主模拟挂 `health`（700）、`combat`（伤害 25、射程 3、间隔 2）、`sleeper`、`lootdropper`（默认普通表）、`inspectable`、`knownlocations`、`herdmember`（群类型 `lightninggoatherd`）、`timer`、`saltlicker`（`SetUp(1)`，每次舔盐扣 1 点耐久）、中型可燃／可冻，最后 `SetStateGraph("SGlightninggoat")` 与 `SetBrain`。[主体](../../../raw/current/scripts/prefabs/lightninggoat.lua#L187-L304) |

固定细节：实体标签为 `lightninggoat`／`animal`／`lightningrod`（pristine 之前加），并预先声明 `herdmember`／`saltlicker` 两个组件标签；`Light` 默认关闭、半径 0.85；监听 `attacked`／`electrocute`／`lightningstrike`／`spawnedforhunt` 四个事件；`OnSave` 只在充能时写 `charged` 与 `chargeleft`。[标签与光照](../../../raw/current/scripts/prefabs/lightninggoat.lua#L210-L224)、[监听与存档](../../../raw/current/scripts/prefabs/lightninggoat.lua#L270-L290)

### 名称核验

简中实显为**伏特羊**（英文 Volt Goat），`STRINGS.NAMES.LIGHTNINGGOAT` 一个键命中一次。任务与玩家口语里常见的“伏特加羊”不是官方名，按俗称收进 aliases。[名称](../../../raw/current/scripts/languages/chinese_s.po#L344735-L344737)

## 充能与放电

| 触发 | 成立条件 | 结果与证据 |
|---|---|---|
| 世界雷击 | 落点 40 单位内的候选实体（`lightningrod`／`lightningtarget`／`lightningblocker`）中，第一个带 `lightningrod` 的实体把雷击改到自己的位置；但若落点位于某个 `lightningblocker` 的阻挡半径内，则先由 blocker 处理（改判为 `thunder`，不推事件）。只有雷击类型仍是 `lightning`／`moonstorm_lightning` 时才向被改道的实体推 `lightningstrike` | 伏特羊因此“接雷”并进入充能。[改道](../../../raw/current/scripts/components/weather.lua#L627-L693)、[blocker 优先](../../../raw/current/scripts/components/weather.lua#L678-L693)、[推事件](../../../raw/current/scripts/components/weather.lua#L695-L699)、[监听](../../../raw/current/scripts/prefabs/lightninggoat.lua#L286-L286) |
| 电系攻击 | 被攻击时 `data.stimuli == "electric"`，或武器自带 `stimuli == "electric"`；攻击者带 `electricattacks` 时，`combat` 结算也会把该次攻击标成电系 | **未充能时**进入充能；已充能时同一分支不会再 `setcharged`，而是按 `inst.charged` 走反伤逻辑（见[关联机制导航](#关联机制导航)的 combat 行）。[判定](../../../raw/current/scripts/prefabs/lightninggoat.lua#L152-L153)、[combat 侧](../../../raw/current/scripts/components/combat.lua#L1138-L1144) |
| `electrocute` 事件 | 任意来源向它推 `electrocute` 事件时（推事件的来源不在本页范围） | 直接 `setcharged`，不接收额外参数。[接线](../../../raw/current/scripts/prefabs/lightninggoat.lua#L161-L163)、[监听](../../../raw/current/scripts/prefabs/lightninggoat.lua#L285-L285) |
| 猎杀生成 | 被 `spawnedforhunt` 事件生成时 | 在自己位置请求一次雷击，通常随后自行充能。[请求](../../../raw/current/scripts/prefabs/lightninggoat.lua#L165-L167) |
| 充能结果 | 生命组件存在且未死 | 加 `charged`／`electricdamageimmune` 标签、装 `electricattacks` 并登记自身为源、换 `chargedlightninggoat` 表、换 `shocked` 外观与 bloom、开灯、`chargeleft = 3`，并挂 `cycles` 世界状态监听（每次 `setcharged` 都会追加一条，见“自动放电”行）。[setcharged](../../../raw/current/scripts/prefabs/lightninggoat.lua#L109-L131) |
| 自动放电 | 每过一个世界周期 `chargeleft` 减 1（只有一条 `cycles` 监听、即正常单次充能时才是每周期减 1；重复 `setcharged` 会叠加监听，同一周期扣多次、放电更快），减到 0 | 移除两个标签与 `electricattacks` 源、换回普通表、进 `discharge` 状态、清 bloom、关灯、`chargeleft = nil`。[递减](../../../raw/current/scripts/prefabs/lightninggoat.lua#L100-L107)、[discharge](../../../raw/current/scripts/prefabs/lightninggoat.lua#L84-L98) |
| 存档恢复 | 存档里 `charged` 且有 `chargeleft` | 读档以 `setcharged(inst, true)` 恢复（跳过 `shocked` 状态，其余外观照旧）。[存读](../../../raw/current/scripts/prefabs/lightninggoat.lua#L169-L181) |

状态机侧的相关状态是 `shocked`（进入充能时播放）与 `discharge`（放电时换回普通 build），另有把旧名 `electrocute` 转发到 `shocked` 的兼容状态。[状态](../../../raw/current/scripts/stategraphs/SGlightninggoat.lua#L181-L236)

## 掉落表

| 表名 | 条目（权重） | 说明 |
|---|---|---|
| `lightninggoat`（默认） | `meat` 1.00×2、`lightninggoathorn` 0.25 | 未充能死亡时的结果，**没有奶**。[普通表](../../../raw/current/scripts/prefabs/lightninggoat.lua#L21-L26) |
| `chargedlightninggoat`（充能） | `meat` 1.00×2、`goatmilk` 1.00、`lightninggoathorn` 0.25 | 充能期间死亡、且掉落未被燃烧改写时必掉 1 瓶奶；常规燃烧／已燃死亡会被改写成 `ash`，受控燃烧保留原物。[带电表](../../../raw/current/scripts/prefabs/lightninggoat.lua#L28-L34)、[切换](../../../raw/current/scripts/prefabs/lightninggoat.lua#L119-L119)、[燃烧改写](../../../raw/current/scripts/components/lootdropper.lua#L424-L442) |

权重语义按 `lootdropper` 的概率表：`chance >= 1.0` 的条目必出，其余逐条按概率判定，所以两张表里 1.00 的肉、以及充能表里 1.00 的奶都是固定产出（前提是未被 S2 改写），0.25 的羊角才是随机项。上表是正常死亡按当前掉落表结算的结果；常规燃烧／已燃状态死亡会先被 S2 改写（受控燃烧保留原物；见下节“共享规则接入”）。[判定](../../../raw/current/scripts/components/lootdropper.lua#L213-L224)

数值：生命 700、伤害 25、攻击射程 3、攻击间隔 2、走速 4／跑速 8、索敌 8、追击 30；`saltlicker:SetUp(1)` 让每次舔盐块扣盐块 1 点耐久。[常量](../../../raw/current/scripts/tuning.lua#L662-L672)、[舔盐块次数](../../../raw/current/scripts/tuning.lua#L3271-L3271)、[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L779-L779)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：lootdropper | 默认挂普通表；充能／放电时 `SetChanceLootTable` 切换 | 主模拟；切换即改掉落结果 | [[reference/components/lootdropper]]；[默认](../../../raw/current/scripts/prefabs/lightninggoat.lua#L254-L255)、[充能切换](../../../raw/current/scripts/prefabs/lightninggoat.lua#L119-L119)、[放电切回](../../../raw/current/scripts/prefabs/lightninggoat.lua#L92-L92) |
| 组件：electricattacks | 充能时加上并把自己登记为源；放电时移除源 | 主模拟；源清空后组件自删 | [[reference/components/electricattacks]]；[增删](../../../raw/current/scripts/prefabs/lightninggoat.lua#L88-L89)、[组件](../../../raw/current/scripts/components/electricattacks.lua#L8-L17) |
| 组件：combat | 充能时攻击算电系；被攻击时按条件反伤电击 | 主模拟；反伤排除灵魂、远程武器、绝缘背包与投石机 | [[reference/components/combat]]；[反伤](../../../raw/current/scripts/prefabs/lightninggoat.lua#L137-L159) |
| 组件：herdmember | 群类型 `lightninggoatherd`；追出 `LIGHTNING_GOAT_CHASE_DIST` 就放弃目标 | 主模拟 | [[reference/components/herdmember]]；[挂载](../../../raw/current/scripts/prefabs/lightninggoat.lua#L264-L266)、[追击](../../../raw/current/scripts/prefabs/lightninggoat.lua#L80-L82) |
| 组件：saltlicker | `SetUp(1)`：每次舔盐扣盐块 1 点耐久，并启用寻盐／加盐状态 | 主模拟 | [[reference/components/saltlicker]]；[配置](../../../raw/current/scripts/prefabs/lightninggoat.lua#L274-L276)、[扣耐久](../../../raw/current/scripts/components/saltlicker.lua#L50-L58) |
| 天气：`weather` | 靠 `lightningrod` 标签被雷击改道，再由 `lightningstrike` 事件充能 | 主模拟；只在 `lightning`／`moonstorm_lightning` 类型推事件 | [[reference/components/weather]]；[改道](../../../raw/current/scripts/components/weather.lua#L627-L699) |
| SG／Brain | `SGlightninggoat` 管 `shocked`／`discharge`；brain 管行为决策 | 服务端 | [状态](../../../raw/current/scripts/stategraphs/SGlightninggoat.lua#L181-L236)、[安装](../../../raw/current/scripts/prefabs/lightninggoat.lua#L300-L301) |
| 关联 Prefab：`goatmilk` | 充能表的必掉产物 | 死时结算 | [[reference/prefabs/goatmilk]]；[表](../../../raw/current/scripts/prefabs/lightninggoat.lua#L28-L34) |
| 关联 Prefab：`lightninggoathorn` | 两张表都按 0.25 掉；物品侧见 [[reference/prefabs/lightninggoathorn]] | 主模拟；死时结算 | [表](../../../raw/current/scripts/prefabs/lightninggoat.lua#L25-L33) |
| 关联 Prefab：`lightninggoatherd`／`lightninggoatcorpse` | 群与尸体的预载依赖 | 声明不等于实际生成 | [依赖](../../../raw/current/scripts/prefabs/lightninggoat.lua#L10-L17) |

### 共享规则接入（S2 适用；S1／S3–S6 不适用）

- **S2 燃烧掉落适用**：伏特羊可燃烧（`MakeMediumBurnableCharacter`），死亡统一走 `DropDeathLoot`；未化尸时直接 `lootdropper:DropLoot()`，而燃烧中的实体不能化尸，所以燃烧／已燃状态的死亡会进入 S2 的换名分支——两份 `meat` → `cookedmeat`，`goatmilk` 与 `lightninggoathorn` 都没有熟态变体 → `ash`。**结论：常规燃烧烧死的羊拿不到奶与羊角；受控燃烧保留原物。** [可燃烧](../../../raw/current/scripts/standardcomponents.lua#L258-L269)、[死亡状态调用](../../../raw/current/scripts/stategraphs/commonstates.lua#L1446-L1456)、[掉落分支](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[不能化尸](../../../raw/current/scripts/componentutil.lua#L1546-L1553)、[换名](../../../raw/current/scripts/components/lootdropper.lua#L424-L442)
- **S1／S3／S4／S6 不适用**：本页对象不是食物、没有 `perishable`，不走 COOK、晾晒或烹饪登记；S5 的堆叠口径也不涉及（生物没有 `stackable`）。

## 典型接入

### 手动把一只伏特羊置为充能（复制原版入口）

- 前提与选择：服务端 `modmain.lua` postinit。用 prefab 已经暴露的 `inst.setcharged`（与雷击监听同一个函数），不要自己拼标签与掉落表。
- 生效与撤销：置充能后 `chargeleft = 3`，每个世界周期自动减 1 并最终放电；想提前结束可把 `chargeleft` 设为 1，让它下个周期自然放电，而不是手动删标签。
- 原版依据：`fn` 把 `setcharged` 挂在实例上，雷击／电击事件都指向它；`setcharged(inst, instant)` 的第二个参数为真时跳过 `shocked` 状态。[暴露](../../../raw/current/scripts/prefabs/lightninggoat.lua#L287-L287)、[实现](../../../raw/current/scripts/prefabs/lightninggoat.lua#L109-L131)

```lua
-- modmain.lua：给新生成的伏特羊加一次充能（例如自定义天气）
AddPrefabPostInit("lightninggoat", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst:DoTaskInTime(0, function(inst)
        if inst.setcharged ~= nil and not inst.charged then
            inst.setcharged(inst) -- 省第二个参数 → 会播放 shocked 状态
        end
    end)
end)
```

- 调整与验证：进游戏后用 `c_spawn("lightninggoat")` 观察是否亮灯／换 build，击杀确认掉 1 瓶 `goatmilk`；再过 3 个世界周期确认放电且掉落表恢复（单次充能、单条监听时才是 3 个周期；重复充能会叠加监听、更快放电）。

## 常见陷阱与限制

- **误区：只有被雷劈才会充能。** 电系攻击、`electrocute` 事件同样直接进入充能；攻击者带 `electricattacks` 时该次攻击也会被标成电系。[判定](../../../raw/current/scripts/prefabs/lightninggoat.lua#L152-L153)、[combat](../../../raw/current/scripts/components/combat.lua#L1138-L1144)
- **误区：充能会一直保持。** `chargeleft` 硬编码为 3，靠 `WatchWorldState("cycles", ReduceCharges)` 逐周期递减，减到 0 立刻放电并恢复普通掉落表。[递减](../../../raw/current/scripts/prefabs/lightninggoat.lua#L100-L107)
- **边界：重复充能会叠加周期监听。** 每次 `setcharged` 都会再追加一条 `cycles` 监听（旧监听不移除）并把 `chargeleft` 重置为 3；`AddWatcher` 只是 `table.insert`、不去重，所以充能中再被雷击（或再收到 `electrocute` 事件）后，同一周期会多次扣减，“重置到 3 个周期”的实际可用窗口更短；普通电系攻击在已充能时走反伤分支、不会再次 `setcharged`（未运行验证实际递减次数）。[监听入表](../../../raw/current/scripts/components/worldstate.lua#L281-L295)、[重复注册](../../../raw/current/scripts/prefabs/lightninggoat.lua#L127-L129)
- **边界：反伤有排除条件。** 灵魂伤害、远程武器、绝缘背包与投石机不触发反伤；电免疫的攻击者只吃 1 倍基础电伤，不乘电伤害与水湿加成。[反伤](../../../raw/current/scripts/prefabs/lightninggoat.lua#L137-L159)
- **未核验**：本页未运行游戏；雷击概率与天气条件的实际频率、`lightninggoatherd` 的群体生成参数、以及尸体 Prefab 的生成时机都只按源码推导。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `prefabs/lightninggoat.lua` 全文（两张掉落表、`setcharged`／`discharge`／`ReduceCharges`、反伤、存读、标签与组件挂载）；`components/weather` 的雷击改道与事件推送；`components/lootdropper` 的概率表判定；`components/combat` 的电系判定与 `electricattacks`；`components/herdmember`／`saltlicker` 的挂载；`SGlightninggoat` 的 `shocked`／`discharge`／兼容状态；`OnAttacked` 的分支界线（未充能时的电系充能 vs 已充能时的反伤）；`commonstates` 的死亡状态与 `DropDeathLoot`、`entityscript` 的 `DropDeathLoot` 分支、`componentutil` 的“燃烧不能化尸”、`standardcomponents` 的 `MakeMediumBurnableCharacter`；`tuning` 的数值与 `scrapbookdata` 交叉核对。
- 名称核验：`languages/chinese_s.po` 的 `NAMES.LIGHTNINGGOAT`＝伏特羊（英文 Volt Goat）；“伏特加羊”仅作俗称。
- 未覆盖：群猎与繁殖、`lightninggoatcorpse` 的表现、化尸分支的实际触发条件、舔盐块的实际交互、以及 `spawnedforhunt` 的上游调用方。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 奶与其用途：[[reference/prefabs/goatmilk]]
- 掉落与战斗：[[reference/components/lootdropper]]、[[reference/components/combat]]、[[reference/components/electricattacks]]
- 群与天气：[[reference/components/herdmember]]、[[reference/components/weather]]
- 同批：[[reference/prefabs/tallbird-egg]]

返回 [[reference/prefabs/index|Prefab 索引]]。
