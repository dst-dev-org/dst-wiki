---
title: "果蝇王、果蝇、友好果蝇与友好果蝇果（lordfruitfly、fruitfly、friendlyfruitfly、fruitflyfruit）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
summary: "fruitfly.lua 的四个注册：果蝇王由 farming_manager 的 worldsettingstimer 计时器放行、同一时刻只允许一只，死亡额外掉友好果蝇果或种子（plantmeat 仍照掉）；小果蝇由 buzz 状态孵化并绑 leader，果蝇王死后只有“已闹过（hascausedhavoc）且当前无 leader”的才会转敌对；友好果蝇被果绑定后按 FindFarmPlant 给作物做一次照料式“授粉”。"
aliases: ["lordfruitfly", "fruitfly", "friendlyfruitfly", "fruitflyfruit", "果蝇王", "果蝇", "友好果蝇", "友好果蝇果", "Lord of the Fruit Flies", "Fruit Fly"]
tags: [prefab, creature, item, loot]
size_class: complex
dst_build: "steam-buildid:26474873"
sources: ["prefabs/fruitfly.lua", "brains/fruitflybrain.lua", "brains/friendlyfruitflybrain.lua", "stategraphs/SGfruitfly.lua", "components/farming_manager.lua", "components/lordfruitflytrigger.lua", "components/worldsettingstimer.lua", "components/follower.lua", "components/leader.lua", "components/health.lua", "components/lootdropper.lua", "components/perishable.lua", "entityreplica.lua", "components/sanityaura.lua", "components/combat.lua", "behaviours/findfarmplant.lua", "prefabs/farm_plants.lua", "actions.lua", "componentactions.lua", "prefabs/forest.lua", "prefabs/cave.lua", "prefabs/world.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 果蝇王、果蝇、友好果蝇与友好果蝇果（lordfruitfly、fruitfly、friendlyfruitfly、fruitflyfruit）

## 快速判断

- 职责与适用：`prefabs/fruitfly.lua` 返回四注册——首领 `lordfruitfly`、爪牙 `fruitfly`、友好版 `friendlyfruitfly`、绑定物 `fruitflyfruit`。三者共用同一份 `SGfruitfly`，差别在 brain、tag 与组件；果是纯物品。[返回](../../../raw/current/scripts/prefabs/fruitfly.lua#L587-L590)
- 优先使用：查“果蝇王为什么不来／为什么只有一只”看[刷新门](#果蝇王由-farmingmanager-的计时器放行一次只一只)；查掉落与种子看[掉落](#果蝇王的掉落果一只或种子八抽)；查作物被啃的实际后果看[小果蝇](#小果蝇从-buzz-出生绑-leader果蝇王倒下且已闹过才转敌对)；查友好果蝇怎么授粉看[友好果蝇](#友好果蝇果--leader--照料式授粉)。
- 重要副作用：`ATTACKPLANT` 与 `INTERACT_WITH` 改的是作物的 `happiness` 压力位与可照料状态，**不扣作物血量**；果蝇王死亡推 `ms_lordfruitflykilled` 并重开 20 天重生计时；友好果蝇果的 leader 一旦失去子体就退化成干壳（只挂 `perishable` 与 6 天定义值、从不启动计时，实际不腐烂；可当燃料）。[植物动作](../../../raw/current/scripts/actions.lua#L1570-L1578)、[死亡事件](../../../raw/current/scripts/prefabs/fruitfly.lua#L146-L148)、[退化](../../../raw/current/scripts/prefabs/fruitfly.lua#L473-L506)
- 修改边界：本页只写这四注册与它们的触发／掉落／AI 差异。种植机制、压力与菜谱归 [[reference/components/topics/farming]] 与 [[reference/prefabs/farm-plants]]；触发组件本身见 [[reference/components/lordfruitflytrigger]]（该组件在当前快照未被任何 prefab 挂载）。

## 注册与构造

中文名取自 `STRINGS.NAMES`：果蝇王／果蝇／友好果蝇／友好果蝇果（S2）。[果蝇王](../../../raw/current/scripts/languages/chinese_s.po#L344790-L344792)、[果蝇](../../../raw/current/scripts/languages/chinese_s.po#L343045-L343047)、[友好果蝇](../../../raw/current/scripts/languages/chinese_s.po#L343000-L343002)、[友好果蝇果](../../../raw/current/scripts/languages/chinese_s.po#L343050-L343052)

三个生物（果蝇王／果蝇／友好果蝇）共用 `common()`（Transform／AnimState／SoundEmitter／DynamicShadow／Network／`SetFourFaced`／`MakeGhostPhysics(1,.5)`／bank `fruitfly`）与 `common_server()`（inspectable／sleeper 抗性 3／locomotor 6／冻结与燃烧符号 `fruit2`／`MakeHauntablePanic`）；果是自建物品工厂，`fruitfn` 自己建实体、不调 `common()`。[生物公共构造](../../../raw/current/scripts/prefabs/fruitfly.lua#L56-L96)、[果工厂](../../../raw/current/scripts/prefabs/fruitfly.lua#L546-L585)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lordfruitfly` | `fn`：build `fruitfly_evil`、缩放 ×2、pristine 前 tags 为公共组 `flying`／`ignorewalkableplatformdrowning`／`insect`／`small`／`plantcreature` 再加 `lordfruitfly`／`fruitfly`／`hostile`／`epic`／`smallepic` → `SetPristine` → 主模拟 `SetStateGraph("SGfruitfly")`＋`SetBrain(fruitflybrain)` | 组件：leader（无 follower）、combat、health（1500）、knownlocations（`home` 记为 y=20）、lootdropper（`SetChanceLootTable("lordfruitfly")`＋`SetLootSetupFn`）、sanityaura −`SANITYAURA_SMALL`；`sg.mem.burn_on_electrocute/noknockback`；移速 `WALKSPEED/2`；存读 `hascausedhavoc`。[工厂](../../../raw/current/scripts/prefabs/fruitfly.lua#L198-L272) |
| `fruitfly` | `minifn`：build `fruitfly_evil_minion`、缩放 ×0.5、pristine 前 tags 为公共的 `flying` 组再加 `fruitfly`／`hostile`（**无** `lordfruitfly`）→ 主模拟 combat／health／loot → 末尾 `SetBrain(brain)`＋`SetStateGraph("SGfruitfly")` | 组件：follower（由 buzz 或外部 `SetLeader` 绑定）、combat（5 伤害／1 距离）、health（100）、lootdropper（`seeds` 0.1）、sanityaura −`SANITYAURA_TINY`；移速 `WALKSPEED/0.5`；`CanTargetAndAttack` 只在“没 leader 且已闹过”时为真。[爪牙工厂](../../../raw/current/scripts/prefabs/fruitfly.lua#L303-L369) |
| `friendlyfruitfly` | `friendlyfn`：build `fruitfly_good`、缩放 ×0.75、`AddMiniMapEntity`（图标 `friendlyfruitfly.png`、优先级 5）、tags `friendlyfruitfly`／`cattoyairborne`、`MakeInventoryFloatable("med")` → `SetPristine` → 主模拟 | 组件：follower（监听 `startfollowing`／`stopfollowing` 切 `companion` tag）、health（**不调** `SetMaxHealth`，取组件默认 100）、combat（`SetKeepTargetFunction` 恒 false，无伤害／间隔设置）、lootdropper（空）、sanityaura +`SANITYAURA_TINY`、sleeper `testperiod=GetRandomWithVariance(6,2)`；brain 是 `friendlyfruitflybrain`。[友好工厂](../../../raw/current/scripts/prefabs/fruitfly.lua#L404-L460) |
| `fruitflyfruit` | `fruitfn`：bank／build `fruitflyfruit`、`idle`、tags `fruitflyfruit`／`nonpotatable`／`irreplaceable`、`MakeInventoryFloatable("med",nil,0.7)` → `SetPristine` → 主模拟 | 组件：leader（`onremovefollower=OnLoseChild`）、inspectable（`getstatus` 在失子后返回 `DEAD`）、inventoryitem；`MakeHauntableLaunch`；`DoTaskInTime(0, OnInit)` 复绑友好果蝇；存读 `deadchild`。[果工厂](../../../raw/current/scripts/prefabs/fruitfly.lua#L546-L585) |

三者共用一个 `SGfruitfly`，初始状态都是 `taunt`；`ActionHandler` 按 tag 分派：`ATTACKPLANT`／`PLANTWEED` 在带 `lordfruitfly` 时进 `plant_attack`、否则进 `plant_attack_minion`，`INTERACT_WITH` 一律进 `plant_dance`。所以“果蝇王与果蝇的 SG 差异”不是两份状态机，而是同一状态机里的 tag 分支。[分派](../../../raw/current/scripts/stategraphs/SGfruitfly.lua#L3-L14)、[初始状态](../../../raw/current/scripts/stategraphs/SGfruitfly.lua#L328)

### 果蝇王：由 farming_manager 的计时器放行，一次只一只

计时器住在 `worldsettingstimer` 组件里（组件名，不是游戏内的世界设置开关），名字 `lordfruitfly_spawntime`，注册时把 `TUNING.SPAWN_LORDFRUITFLY` 当 `enabled` 参数传入；世界建好后先起一发 35 天（16800 秒）的首刷计时（S4）。[计时器注册与首刷](../../../raw/current/scripts/components/farming_manager.lua#L74-L86)、[宿主顺序](../../../raw/current/scripts/prefabs/world.lua#L580-L583)

放行条件全在 `OnFruitFlySpawnerActive` 的一个 if 里：常量为假、作物在睡眠、**已有活动计时器**、**世界里已存在带 `lordfruitfly` tag 的实体**、或（`check_others` 时）作物 15 距离内没有玩家，任一命中就直接返回。通过后再数作物 4 距离内的 `fruitflyspawner` 实体，够 15 个才在作物正上方 y=20 生成并 `GoToState("land")` 落下；不够且 `check_others` 为真时，还会把 4 距离内每个 `fruitflyspawner` 依次以 `check_others=false` 再判一遍。[放行判定](../../../raw/current/scripts/components/farming_manager.lua#L217-L234)、[数值](../../../raw/current/scripts/tuning.lua#L5895-L5910)

| 项目 | 定义处 | 值 | 结算与条件 |
|---|---|---:|---|
| `LORDFRUITFLY_INITIALSPAWN_TIME` | [tuning](../../../raw/current/scripts/tuning.lua#L5902) | 35 天＝16800 秒 | 模块加载时 `StartFruitFlyTimer` 起算，闸门关着；但**每次作物腐烂都会把剩余时间前拨 240 秒**，腐烂多时首刷会明显早于 35 天。[首刷](../../../raw/current/scripts/components/farming_manager.lua#L86)、[前拨](../../../raw/current/scripts/components/farming_manager.lua#L80-L84) |
| `LORDFRUITFLY_RESPAWN_TIME` | [tuning](../../../raw/current/scripts/tuning.lua#L5903) | 20 天＝9600 秒 | 果蝇王死亡推 `ms_lordfruitflykilled`，`farming_manager` 收到后重开此计时。[重开](../../../raw/current/scripts/components/farming_manager.lua#L616) |
| `LORDFRUITFLY_CROP_ROTTED_ADVANCE_TIME` | [tuning](../../../raw/current/scripts/tuning.lua#L5904) | 0.5 天＝240 秒 | 任一作物进入腐烂阶段推 `ms_oncroprotted`，把当前计时**前拨** 240 秒；剩余不足 240 秒时夹到 0，等于让计时立刻到期，计时没在跑时则是空操作。[前拨](../../../raw/current/scripts/components/farming_manager.lua#L80-L84)、[腐烂推送](../../../raw/current/scripts/prefabs/farm_plants.lua#L540-L543)、[夹取](../../../raw/current/scripts/components/worldsettingstimer.lua#L141-L151) |
| `LORDFRUITFLY_SPAWNERRADIUS`／`SPAWNERCOUNT` | [tuning](../../../raw/current/scripts/tuning.lua#L5905-L5906) | 4／15 | 以触发作物的位置为心数 `fruitflyspawner`，够数才刷。[数量判定](../../../raw/current/scripts/components/farming_manager.lua#L224-L228) |
| 存读 | — | — | `farming_manager` 只额外存一个“存档时没有活动计时器”的布尔，读档据此清掉计时器；真正的时间进度由 `worldsettingstimer` 自己存读。[存档标记](../../../raw/current/scripts/components/farming_manager.lua#L492)、[读档](../../../raw/current/scripts/components/farming_manager.lua#L563-L566)、[计时器存读](../../../raw/current/scripts/components/worldsettingstimer.lua#L176-L203) |

推论（推导）：计时器结束后 `timeleft` 被清空（`OnTimerDone` 先 `StopTimer` 再回调），所以“计时器跑完”不等于“立刻刷出”，而是**打开下一次作物触发的闸门**。闸门打开后，门要按三层看：**①共通门禁（任一候选点都要过）**——`TUNING.SPAWN_LORDFRUITFLY` 为真、该点作物不在睡眠、没有活动计时器、世界内没有果蝇王；**②首轮追加门禁（仅 `check_others=true`）**——该作物 15 距离内有玩家；**③每个候选点各自的 4 距离计数**——以该点为中心数 `fruitflyspawner`，≥15 就在该点生成，否则不再生成。首轮点计数不足并不代表整体不刷：它会以 4 距离内每个 `fruitflyspawner` 作为**新的候选点**递归（`check_others=false`，因此不再查玩家距离），任一新候选点的 4 距离计数达到 15 即在该点生成——与[放行条件](#果蝇王由-farmingmanager-的计时器放行一次只一只)里那段递归一致。[到期处理](../../../raw/current/scripts/components/worldsettingstimer.lua#L53-L60)、[闸门](../../../raw/current/scripts/components/farming_manager.lua#L220)

谁推 `ms_fruitflyspawneractive`：**只有**作物进入 `full`（成熟）阶段时以 `check_others=true` 推一次，同时把该作物设为可采（非 `farm_plant_killjoy` 才加 `fruitflyspawner` tag）；腐烂阶段推的是另一个事件 `ms_oncroprotted`，不触发刷新。作物实体上的 `_activatefn`（供触发器式组件调用）以 `check_others=false` 推同事件。当前快照里没有任何 prefab 挂 `AddComponent("lordfruitflytrigger")`，因此“计时结束→玩家附近作物被重新激活”那条触发器链并未接线，实际节奏由作物自身的阶段事件决定。[成熟推送](../../../raw/current/scripts/prefabs/farm_plants.lua#L520-L530)、[可采与 tag](../../../raw/current/scripts/prefabs/farm_plants.lua#L305-L320)、[`_activatefn`](../../../raw/current/scripts/prefabs/farm_plants.lua#L680-L683)、[挂载点](../../../raw/current/scripts/prefabs/farm_plants.lua#L839)、[未接线](../../../raw/current/scripts/components/lordfruitflytrigger.lua#L19-L39)

### 果蝇王的掉落：果一只或种子八抽

掉落走两条并行通道，每次 `GenerateLoot` 都先重跑 `lootsetupfn`：共享表 `lordfruitfly` 固定给 1 个 `plantmeat`；`LordLootSetupFunction` 再决定给果还是给种子——世界里找不到带 `friendlyfruitfly` tag 的实体时给 `fruitflyfruit`×1（必掉），否则改为 4 轮“种子 1.0 ＋ 种子 0.25”，种子按季节权重从作物表里抽。`AddChanceLoot` 本身是无条件建表追加（表为 nil 就新建），真正的原因是 setup 函数每次在开头把 `chanceloot` 置 nil，所以每次掉落都是重新构造的列表。[共享表](../../../raw/current/scripts/prefabs/fruitfly.lua#L51-L54)、[setup 函数](../../../raw/current/scripts/prefabs/fruitfly.lua#L110-L120)、[抽取链](../../../raw/current/scripts/components/lootdropper.lua#L186-L234)、[种子权重](../../../raw/current/scripts/prefabs/fruitfly.lua#L39-L49)

注意“世界里已有友好果蝇”是**全局**判定，且变动的只是**额外**掉落：共享表的 `plantmeat` 仍必掉 1 个，只是不再给果、改为 8 抽种子（4 轮 × 概率 1.0＋0.25）。

### 小果蝇：从 buzz 出生、绑 leader，果蝇王倒下且已闹过才转敌对

- 出生：果蝇王 brain 在 `PriorityNode` 第 2 位插 `MinPeriod(30, ...)` 节点，条件是存活 >5 秒、`NumFruitFliesToSpawn() > 0`、且当前有战斗目标或已锁定作物／耕地，命中就 `GoToState("buzz")`。[节点](../../../raw/current/scripts/brains/fruitflybrain.lua#L11-L15)、[插入](../../../raw/current/scripts/brains/fruitflybrain.lua#L99-L105)
- 数量：`min(已有随从 + 4/2, 4)` 再取 `log(x)/0.4 + 1` 四舍五入、减去已有随从。按此公式推导：0 只时首轮得 3 只，已有 2 只时得 2 只，已有 4 只时为 0（不再刷）。[算法](../../../raw/current/scripts/prefabs/fruitfly.lua#L163-L172)
- 生成：`buzz` 第 32 帧进 `SpawnFruitFly`，在果蝇王周围 ±10 的随机点、y=20 处生成，然后 `GoToState("land")` 落地并 `follower:SetLeader(果蝇王)`；这条函数外面还套了一层 `TheWorld.components.hounded ~= nil` 的判空，森林与洞穴都装了 hounded，实际恒真。[生成](../../../raw/current/scripts/stategraphs/SGfruitfly.lua#L40-L55)、[buzz](../../../raw/current/scripts/stategraphs/SGfruitfly.lua#L235-L258)、[hounded](../../../raw/current/scripts/prefabs/forest.lua#L643)
- 对作物的实际效果：`ATTACKPLANT.fn` 只做两件事——把 `farmplantstress` 的 `happiness` 位置成 true、把 `farmplanttendable` 设成可照料；不扣生命、不改阶段。动作在 `plant_attack`（37 帧）／`plant_attack_minion`（63 帧）里 `PerformBufferedAction`，同时把 `inst.hascausedhavoc = true`。[动作](../../../raw/current/scripts/actions.lua#L1570-L1578)、[帧位](../../../raw/current/scripts/stategraphs/SGfruitfly.lua#L147-L157)、[爪牙帧位](../../../raw/current/scripts/stategraphs/SGfruitfly.lua#L179-L184)
- 还会种杂草：brain 里 `ShouldSowWeeds` 在以跟随点为心 20 内找 `soil`，找到就 `PLANTWEED`，同一株作物有 `IsTargetedByOther` 互斥，避免一群果蝇围着一株啃。[种杂草](../../../raw/current/scripts/brains/fruitflybrain.lua#L49-L66)、[互斥](../../../raw/current/scripts/prefabs/fruitfly.lua#L174-L184)
- 果蝇王死后：leader 组件 `RemoveAllFollowersOnDeath` 把随从全部解绑，爪牙的 `CanTargetAndAttack` 变成“**当前没有 leader 且已置 `hascausedhavoc`**”——满足就按 15 距离找玩家打，否则继续漂着。这里的标记不等于“一定啃成功”：`plant_attack_minion` 在第 63 帧调 `PerformBufferedAction()` 后**不检查返回值**，紧接着就把 `hascausedhavoc = true`（首领的 `plant_attack` 在第 37 帧同样无条件置位），所以动作失败也可能置上。[解绑](../../../raw/current/scripts/components/leader.lua#L11-L13)、[转敌对条件](../../../raw/current/scripts/prefabs/fruitfly.lua#L274-L284)、[追人节点](../../../raw/current/scripts/brains/fruitflybrain.lua#L114-L115)

### 友好果蝇：果 → leader → 照料式授粉

- 绑定点：果的主模拟挂 `leader` 组件，`onremovefollower=OnLoseChild`；`OnInit`（0 秒延迟）在果仍带 `fruitflyfruit` tag 时，**全世界**找第一只 `friendlyfruitfly`，找不到才在 35 距离内按可行走偏移生成一只，然后 `SetLeader(果)`；跳过条件只有两个——它已经绑在**本果**上（`leader == inst`）或它已死。若它正绑在**别的活果**上，会被转绑过来，旧果因失去随从触发 `onremovefollower` → `OnLoseChild` 变干壳。[复绑](../../../raw/current/scripts/prefabs/fruitfly.lua#L533-L544)、[生成](../../../raw/current/scripts/prefabs/fruitfly.lua#L522-L531)、[挂载](../../../raw/current/scripts/prefabs/fruitfly.lua#L571-L582)
- 跟随与拴绳：`follower:SetLeader` 遇到带 `inventoryitem` 的 leader 会注册 item source 并 `StartLeashing`，所以果在背包／容器里时果蝇仍被拴在该物品上；`startfollowing` 时若 leader 带 `fruitflyfruit` tag 就加 `companion` tag，解除时移除。[拴绳](../../../raw/current/scripts/components/follower.lua#L319-L334)、[companion](../../../raw/current/scripts/prefabs/fruitfly.lua#L388-L396)
- 授粉＝照料：brain 用 `FindFarmPlant(inst, ACTIONS.INTERACT_WITH, true, GetFollowPos)`，第三个参数 `wantsstressed=true` 表示只挑 `farmplantstress.stressors.happiness == true`（正被该压力压着）且 `tendable` 的作物；`FindEntity` 的 20 以果蝇自身为心，再用 `IsNearFollowPos` 要求作物距跟随点也在 20 内；SG 进 `plant_dance`，第 77 帧执行动作。`INTERACT_WITH` 对 `tendable_farmplant` 作物最终调 `ontendto` → `SetStressed("happiness", false)`，与玩家亲手照料同一条回调。[挑目标](../../../raw/current/scripts/behaviours/findfarmplant.lua#L24-L58)、[动作注册](../../../raw/current/scripts/componentactions.lua#L377-L385)、[回调](../../../raw/current/scripts/prefabs/farm_plants.lua#L115-L119)、[舞蹈帧位](../../../raw/current/scripts/stategraphs/SGfruitfly.lua#L95-L117)
- 不参战：友好果蝇没有攻击节点（brain 只有 panic／电场恐慌／找作物／跟随／面向／漫游），combat 的 `SetKeepTargetFunction` 恒 false，也没有设置伤害与间隔；它的 `health` 是默认 100。[brain](../../../raw/current/scripts/brains/friendlyfruitflybrain.lua#L35-L47)、[combat](../../../raw/current/scripts/prefabs/fruitfly.lua#L442-L444)
- 果的死亡态：随从被移除／死亡都会走 leader 的 `onremovefollower` → `OnLoseChild`：加 `perishable` 并 `SetPerishTime(PERISH_FAST)`（2880 秒定义值），同时加 `show_spoilage`、换图标 `fruitflyfruit_dead`、播 `idle_dead`、去掉 `fruitflyfruit` tag；但 `OnLoseChild` 没有调 `StartPerishing`，周期任务不存在、鲜度不推进，干壳永远停在满鲜度，地面 1.5 倍速率也就无从生效。另补 `fuel`（`fuelvalue`＝`MED_LARGE_FUEL`＝90，是作燃料时给火堆加的时间）与 `MakeSmallBurnable(SMALL_BURNTIME)`＋`MakeSmallPropagator`（自身被点燃只烧 7.5 秒）。[失子](../../../raw/current/scripts/prefabs/fruitfly.lua#L473-L506)、[存读](../../../raw/current/scripts/prefabs/fruitfly.lua#L512-L520)、[常量](../../../raw/current/scripts/tuning.lua#L1676)
- 存读：`fruitflyfruit` 只存一个 `deadchild` 布尔，读档时 `OnPreLoad` 若看到它就直接复用 `OnLoseChild`，因此干壳读档后仍是干壳，不会自己长回活果；活果则靠 `OnInit` 重新找／生成子体并绑 leader。[存读](../../../raw/current/scripts/prefabs/fruitfly.lua#L512-L520)、[复绑](../../../raw/current/scripts/prefabs/fruitfly.lua#L533-L544)

### 战斗数值（定义处 → 结算处）

| 项目 | `lordfruitfly` | `fruitfly` | `friendlyfruitfly` | 定义处 | 结算处 |
|---|---:|---:|---:|---|---|
| 最大生命 | 1500 | 100 | 100（组件默认） | [tuning](../../../raw/current/scripts/tuning.lua#L5900)、[tuning](../../../raw/current/scripts/tuning.lua#L5914)、[默认值](../../../raw/current/scripts/components/health.lua#L64-L66) | `SetMaxHealth` [首领](../../../raw/current/scripts/prefabs/fruitfly.lua#L241-L243)、[爪牙](../../../raw/current/scripts/prefabs/fruitfly.lua#L340-L342) |
| 单次伤害 | 25 | 5 | — | [tuning](../../../raw/current/scripts/tuning.lua#L5898)、[tuning](../../../raw/current/scripts/tuning.lua#L5915) | `SetDefaultDamage` [首领](../../../raw/current/scripts/prefabs/fruitfly.lua#L236)、[爪牙](../../../raw/current/scripts/prefabs/fruitfly.lua#L335) |
| 攻击间隔 | 2 秒 | 2 秒 | — | [tuning](../../../raw/current/scripts/tuning.lua#L5897)、[tuning](../../../raw/current/scripts/tuning.lua#L5917) | `SetAttackPeriod` [首领](../../../raw/current/scripts/prefabs/fruitfly.lua#L235)、[爪牙](../../../raw/current/scripts/prefabs/fruitfly.lua#L334) |
| 攻击距离 | 2 | 1 | — | [tuning](../../../raw/current/scripts/tuning.lua#L5899)、[tuning](../../../raw/current/scripts/tuning.lua#L5916) | `SetRange` [首领](../../../raw/current/scripts/prefabs/fruitfly.lua#L237)、[爪牙](../../../raw/current/scripts/prefabs/fruitfly.lua#L336) |
| 锁定玩家范围 | 15 | 15 | — | [tuning](../../../raw/current/scripts/tuning.lua#L5896)、[tuning](../../../raw/current/scripts/tuning.lua#L5918) | 传给 `RetargetFn` 的 `FindEntity` 半径；重定向周期是代码字面量 3 秒，不是 tuning 项。[首领](../../../raw/current/scripts/prefabs/fruitfly.lua#L238)、[爪牙](../../../raw/current/scripts/prefabs/fruitfly.lua#L337)、[半径用法](../../../raw/current/scripts/prefabs/fruitfly.lua#L131-L134) |
| 脱战距离 | 30（以 `home` 为心） | 20 | — | [tuning](../../../raw/current/scripts/tuning.lua#L5895)、[tuning](../../../raw/current/scripts/tuning.lua#L5913) | `KeepTargetFn` [首领](../../../raw/current/scripts/prefabs/fruitfly.lua#L122-L127)、[爪牙](../../../raw/current/scripts/prefabs/fruitfly.lua#L278-L280) |
| 移速 | 2（4÷缩放 2） | 16（8÷缩放 0.5） | 6 | [tuning](../../../raw/current/scripts/tuning.lua#L5909-L5919) | [首领](../../../raw/current/scripts/prefabs/fruitfly.lua#L259-L260)、[爪牙](../../../raw/current/scripts/prefabs/fruitfly.lua#L354-L355)、[公共 6](../../../raw/current/scripts/prefabs/fruitfly.lua#L85-L87) |
| 火焰伤害倍率 | 1.25 | 1.25 | 1.25 | [tuning](../../../raw/current/scripts/tuning.lua#L9462) | `fire_damage_scale` [首领](../../../raw/current/scripts/prefabs/fruitfly.lua#L243)、[爪牙](../../../raw/current/scripts/prefabs/fruitfly.lua#L342)、[友好](../../../raw/current/scripts/prefabs/fruitfly.lua#L441) |
| 理智光环 | −`SANITYAURA_SMALL`≈−0.4167/秒 | −`SANITYAURA_TINY`≈−0.1042/秒 | +`SANITYAURA_TINY` | [tuning](../../../raw/current/scripts/tuning.lua#L2205-L2207) | [首领](../../../raw/current/scripts/prefabs/fruitfly.lua#L256-L257)、[爪牙](../../../raw/current/scripts/prefabs/fruitfly.lua#L351-L352)、[友好](../../../raw/current/scripts/prefabs/fruitfly.lua#L452-L453) |

首领与爪牙都 `ShouldSleep=false`、`ShouldWake=true`，即使 `sleeper` 抗性是 3 也不会真的入睡；`MakeHauntablePanic` 让它们被闹鬼时只进入恐慌而不是睡眠。首领燃烧时长 8×`PLANTMOB_BURNTIME_MULT`(4/3)≈10.7 秒。[睡眠测试](../../../raw/current/scripts/prefabs/fruitfly.lua#L155-L161)、[燃烧](../../../raw/current/scripts/prefabs/fruitfly.lua#L89-L93)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：leader` | 果蝇王与果各持一个；果的 `onremovefollower` 触发退化 | 果蝇王专职带爪牙；果专职带一只友好果蝇 | [[reference/components/leader]]、[果挂载](../../../raw/current/scripts/prefabs/fruitfly.lua#L571-L572) |
| `组件：follower` | 爪牙被绑到果蝇王、友好果蝇被绑到果 | item source 拴绳在 leader 是物品时自动启用 | [[reference/components/follower]]、[拴绳条件](../../../raw/current/scripts/components/follower.lua#L319-L334) |
| `组件：health` | 1500／100／默认 100，统一 `fire_damage_scale=1.25` | 仅主模拟 | [[reference/components/health]]、[默认值](../../../raw/current/scripts/components/health.lua#L64-L66) |
| `组件：lootdropper` | 共享表 + setup 函数；每次掉落重跑 setup | 仅主模拟；`SetLootSetupFn` 在 `GenerateLoot` 内被调用 | [[reference/components/lootdropper]]、[调用点](../../../raw/current/scripts/components/lootdropper.lua#L186-L191) |
| `组件：combat` | 伤害／间隔／距离／重定向／脱战四件套 | 爪牙与首领同配置不同数值；友好版只有 `KeepTarget=false` | [[reference/components/combat]]、[配置](../../../raw/current/scripts/prefabs/fruitfly.lua#L233-L239) |
| `组件：sanityaura` | 负光环（首领与爪牙）／正光环（友好） | 仅主模拟；`sanityaura` 不在 `REPLICATABLE_COMPONENTS` 白名单，没有客户端 replica，光环由服务端 `sanity` 在 `OnUpdate` 里算进 `aura_delta` | [[reference/components/sanityaura]]、[赋值](../../../raw/current/scripts/prefabs/fruitfly.lua#L256-L257)、[白名单](../../../raw/current/scripts/entityreplica.lua#L5-L37) |
| `SG：SGfruitfly` | 一套状态机供三者共用，按 tag 分派攻击动画 | `taunt` 起手；`buzz` 只有果蝇王会进 | 本页[注册与构造](#注册与构造)、[状态表](../../../raw/current/scripts/stategraphs/SGfruitfly.lua#L235-L277) |
| `Brain：fruitflybrain` | 首领与爪牙共用；`lordfruitfly` tag 决定是否插 summon／chase／dodge 三节点 | 仅主模拟 `SetBrain` | [brain 分支](../../../raw/current/scripts/brains/fruitflybrain.lua#L99-L115) |
| `Brain：friendlyfruitflybrain` | 只跟随、面向、找作物、漫游 | 无攻击节点 | [节点表](../../../raw/current/scripts/brains/friendlyfruitflybrain.lua#L35-L47) |
| `组件：farming_manager` | 拥有 `lordfruitfly_spawntime` 计时器、放行判定与死亡／腐烂回调 | 世界级单例，主模拟 | [[reference/components/farming-manager]]、[事件接线](../../../raw/current/scripts/components/farming_manager.lua#L613-L617) |
| `组件：worldsettingstimer` | 真正存时间的容器；到期回调推 `ms_fruitflytimerfinished` | 世界级；`enabled` 传 `TUNING.SPAWN_LORDFRUITFLY` | [[reference/components/worldsettingstimer]]、[到期](../../../raw/current/scripts/components/worldsettingstimer.lua#L53-L60) |
| `组件：lordfruitflytrigger` | 设计上在计时结束时清空重叠表以重触发周边作物，实际无挂载者 | 全库仅自身引用 | [[reference/components/lordfruitflytrigger]]、[构造](../../../raw/current/scripts/components/lordfruitflytrigger.lua#L19-L39) |
| `行为：FindFarmPlant` | 三者共用的“找作物”节点，参数决定挑谁 | `wantsstressed` 与动作类型由调用方给 | [实现](../../../raw/current/scripts/behaviours/findfarmplant.lua#L24-L58) |
| `关联 Prefab：plantmeat`／`seeds`／作物种子 | 首领掉落 | 见[掉落](#果蝇王的掉落果一只或种子八抽) | [共享表](../../../raw/current/scripts/prefabs/fruitfly.lua#L51-L54)（Wiki 待建） |
| `关联 Prefab：farm_plant` | 触发源与目标 | 只链作物页，不重复其压力机制 | [[reference/prefabs/farm-plants]] |
| `资源：anim/fruitfly.zip`／`fruitfly_evil.zip`／`fruitfly_evil_minion.zip`／`fruitfly_good.zip`／`fruitflyfruit.zip`、`friendlyfruitfly.png` | 动画与图标 | 声明即依赖 | [资产](../../../raw/current/scripts/prefabs/fruitfly.lua#L186-L196) |

## 典型接入

### 给果蝇王追加一件必掉物（必须包 setup 函数，不能只 AddChanceLoot）

- 前提与选择：目标 `modmain.lua`，无新资源。必须包 `lootsetupfn` 而不是在 PostInit 里直接 `AddChanceLoot`：每次 `GenerateLoot` 都会先调 setup，而原版 setup 第一行就把 `chanceloot` 置 nil，直接加的条目会被清掉。[清空](../../../raw/current/scripts/prefabs/fruitfly.lua#L110-L112)、[每次重跑](../../../raw/current/scripts/components/lootdropper.lua#L186-L191)
- 生效与撤销：只影响之后生成的果蝇王；已经死掉／已生成的实例保留旧函数。删除补丁后新实例恢复原掉落。
- 原版依据：`SetLootSetupFn(LordLootSetupFunction)` 在构造时设一次，同时在构造末尾立刻执行一次以初始化“果 or 种子”的分支；真正的掉落发生在死亡时的 `GenerateLoot`。[设置与预执行](../../../raw/current/scripts/prefabs/fruitfly.lua#L248-L251)

```lua
-- modmain.lua：让果蝇王额外必掉一块怪物肉
AddPrefabPostInit("lordfruitfly", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local dropper = inst.components.lootdropper
    if dropper == nil then
        return
    end
    local original = dropper.lootsetupfn
    dropper:SetLootSetupFn(function(self)
        if original ~= nil then
            original(self)
        end
        self:AddChanceLoot("monstermeat", 1.0)
    end)
end)
```

- 调整与验证：`AddChanceLoot` 的第二个参数是概率，1.0 走“必定入列”分支；想在“世界已有友好果蝇、额外只掉种子”那条分支上加东西，同样放在包一层之后即可，因为 setup 会在每次掉落重算分支（共享表的 `plantmeat` 不受这条分支影响）。数值单位与掉落表语义见 [[reference/components/lootdropper]]。

## 常见陷阱与限制

- **以为计时器跑完就会立刻刷果蝇王**：到期只是 `StopTimer`＋推事件，真正生成要等下一次 `ms_fruitflyspawneractive`（作物进入成熟阶段），而且必须世界里没有果蝇王、该首轮作物 15 距离内有玩家（这条距离判定只对 `check_others=true` 生效，递归的邻株不再查）。[到期](../../../raw/current/scripts/components/worldsettingstimer.lua#L53-L60)、[判定](../../../raw/current/scripts/components/farming_manager.lua#L220)
- **以为果蝇王会刷第二只**：放行判定直接 `FindFirstEntityWithTag("lordfruitfly")` 短路；只要第一只还活着，田里再怎么成熟也刷不出第二只。[判定](../../../raw/current/scripts/components/farming_manager.lua#L220)
- **以为小果蝇啃作物会掉血／催熟**：`ATTACKPLANT` 只把 `happiness` 压力置 true 并设可照料，作物血量与阶段不变；“作物腐烂”是作物自己的阶段，腐烂时才反过来把果蝇王计时前拨 240 秒。[动作](../../../raw/current/scripts/actions.lua#L1570-L1578)、[前拨](../../../raw/current/scripts/components/farming_manager.lua#L617)
- **以为友好果蝇“授粉”是加产量**：它走的是和玩家照料同一条 `ontendto`，只清 `happiness` 压力；是否成大果由压力总账决定，本页不展开。[回调](../../../raw/current/scripts/prefabs/farm_plants.lua#L115-L119)、[[reference/components/topics/farming]]
- **给果蝇王加掉落时直接 `AddChanceLoot`**：下一次掉落会被 setup 清空；必须包 `lootsetupfn`。[清空行](../../../raw/current/scripts/prefabs/fruitfly.lua#L111)
- **以为果会为新果蝇另生一只、且旧果不受影响**：`OnInit` 先在**全图**找第一只 `friendlyfruitfly`，找到就把它 `SetLeader` 过来——同一世界里两只果会抢同一只果蝇，第二只不生成新的；被抢走的那只旧果会因失去随从触发 `onremovefollower` → `OnLoseChild` 直接变干壳。[复绑](../../../raw/current/scripts/prefabs/fruitfly.lua#L533-L544)
- **把友好果蝇果当永久伙伴**：随从一旦死亡／被移除，果立刻变干壳（加 `perishable` 定义 2880 秒、加 `show_spoilage`、掉 `fruitflyfruit` tag），只能当燃料；活果状态只在“有子体绑着”时存在。注意干壳的 2880 秒只是定义值：`OnLoseChild` 没调 `StartPerishing`，鲜度不会推进，所以干壳**不会腐烂**，地面 1.5 倍速率也无从生效。[退化](../../../raw/current/scripts/prefabs/fruitfly.lua#L473-L506)

## 验证记录

- 源码审阅：2026-11-09，steam-buildid:26474873（与 `raw/current/BUILD.txt` 一致）。逐行读过 `prefabs/fruitfly.lua`（589 行）、`brains/fruitflybrain.lua`（119 行）、`brains/friendlyfruitflybrain.lua`、`stategraphs/SGfruitfly.lua`（327 行）、`components/farming_manager.lua` 的果蝇段、`components/lordfruitflytrigger.lua`、`components/worldsettingstimer.lua`、`components/follower.lua`／`leader.lua` 的关键函数、`components/lootdropper.lua` 的 `GenerateLoot`、`behaviours/findfarmplant.lua`、`prefabs/farm_plants.lua` 的触发与照料段、`actions.lua` 的 `ATTACKPLANT`、`componentactions.lua` 的照料动作、`tuning.lua` 对应常量与 `chinese_s.po` 的四条 NAMES。
- 未核验（S7）：果蝇王实际刷新节奏与“实战触发概率”、首刷 35 天内玩家是否会看到果蝇王、友好果蝇授粉对巨型作物的实际影响、果蝇王 1500 血的实际战斗时长、掉种子那条分支的期望产出、多人下 `ms_lordfruitflykilled`／`ms_oncroprotted` 的竞争时序。
- 游戏运行：未运行；全部结论为源码审阅与推导，写作中标注“推导”的段落不构成实测。
- 结构检查：本批 `validate_page.py` 与 `validate_source_links.py --strict` 均为 0 error（见交付回执）。
- 外部复验回改（2026-11-10，R04／R13）：R04——推论拆成“共通门禁／首轮玩家门禁／每个候选点各自的 4 距离计数”三层，并说明首轮计数不足时以邻株为新候选点递归（`check_others=false`）通过，消除与放行段递归的矛盾；R13——陷阱改为“无 leader 且已置 `hascausedhavoc`”，并写明 `plant_attack_minion` 第 63 帧 `PerformBufferedAction()` 后不检查返回值即置位（首领第 37 帧同理），动作失败也可能置上；示例的“只掉种子”改为“额外只掉种子”。索引条目由 B 同步。
- 外部检视回改（2026-11-10，R04／R13）：R04——首刷行补“作物腐烂每次前拨 240 秒可提前开门”，推论改为列出闸门打开后仍需的六项条件（事件来源 check_others=true、作物清醒、`SPAWN_LORDFRUITFLY`、场上无王、15 内有人、4 距离内 ≥15 个 spawner），陷阱注明玩家距离只对首轮 `check_others=true` 生效；R13——摘要与标题补“已闹过（hascausedhavoc）且当前无 leader”，掉落句改为“仅额外掉落变化、plantmeat 照掉”，改绑句改为“只有绑在本果或已死才跳过，绑在别果会被转绑并使旧果变干壳”，对应陷阱同步。索引行由主代理同步。
- 检视回改（2026-11-09）：按独立检视报告只改点名条目——干壳改为“只挂组件与 6 天定义值、计时未启动、不腐烂”（覆盖 L23／L78／L156 与陷阱条）；`common()` 说明限定为三个生物、果为自建工厂；`AddChanceLoot` 描述改为“无条件追加＋setup 每次清空”；sanityaura 去掉“客户端 replica 同步”，改为服务端结算；范围 20 的锚点改为“自身为心＋跟随点 20 内”；燃料区分 `fuelvalue` 90 与自身 `SMALL_BURNTIME` 7.5 秒；补 15 个 spawner 不够时的递归判定；页尾补 Prefab 索引返回行。P3 的锚点下划线条未采纳（见回执）。
- 档位：`complex`。需要整体保留的范围是四注册的构造差异、果蝇王“计时器 → 放行 → 掉落 → 重生”整链、爪牙的召唤与转敌对、友好果蝇的绑定与照料链、以及三者共用 SG／两份 brain 的分叉点；这些共享同一批数值与事件，拆页会重复同一张战斗表，故不降档。

## 相关链接

- [[reference/components/topics/farming]]、[[reference/prefabs/farm-plants]]、[[reference/prefabs/mushrooms]]
- [[reference/components/farming-manager]]、[[reference/components/worldsettingstimer]]、[[reference/components/lordfruitflytrigger]]
- [[reference/components/leader]]、[[reference/components/follower]]、[[reference/components/health]]、[[reference/components/combat]]、[[reference/components/lootdropper]]、[[reference/components/sanityaura]]

返回 [[reference/prefabs/index|Prefab 索引]]
