# 明细：状态图与 AI（SB）

build 26335394 → 26474873。条目按五字段契约，编号 SB-序号 在本报告内唯一（AI 行为树并入本文件）。

| 编号 | 变化 | 影响接入／条件 | 证据位置 | 核验状态 |
|---|---|---|---|---|
| SB-1 | `SGwilson.lua` 与 `SGwilson_client.lua` 的 `run_witchbroom_pre` 态（女巫扫帚起飞前跑）起飞机音效 `dontstarve/movement/foley/cactus_armor`→`hallowednights2025/broom/broom_takeoff`。两文件各 1 行 1:1 替换，全文件无行号偏移。 | 消费该状态音效事件（soundemitter 事件监听 / `SoundEmitter:PlaySound` 参数核对）的模组：服务端与客户端状态图同步换新音效事件；监听 `run_witchbroom_pre` 进入时序的逻辑不变。 | 旧:scripts/stategraphs/SGwilson.lua:28843、scripts/stategraphs/SGwilson_client.lua:8727；新:同位置（1:1） | 已核验 |
| SB-2 | `SGwerepig.lua` hit 态 onenter 补 `CommonHandlers.UpdateHitRecoveryDelay(inst)`（+1 行）。 | werepig 受击硬直现在计入通用 hit-recovery 延迟追踪；对 werepig 做受击时序或读该延迟字段的模组行为对齐其他生物。 | 旧:scripts/stategraphs/SGwerepig.lua:359-364；新:scripts/stategraphs/SGwerepig.lua:359-365 | 已核验 |
| SB-3 | `SGmossling.lua` 补沉没／虚空处理：事件层挂 `CommonHandlers.OnSink()`／`OnFallInVoid()`（即监听 `onsink`／`onfallinvoid`）；处理器在事件后仍检查存活、未处于 `drowning`／`falling` 状态，且 `drownable:ShouldDrown()`／`ShouldFallInVoid()` 的环境判定成立才进 `sink`／`abyss_fall`；状态层新增 `CommonStates.AddSinkAndWashAshoreStates(states)` 与 `CommonStates.AddVoidFallStates(states)`（净 +4 行）。 | mossling（苔藓仔）落水进入通用沉没＋冲岸状态、掉虚空进入虚空坠落状态（前提是上述门禁成立）；配合 [P-2](detail-prefabs.md) 的 `drownable` 组件。对 mossling 监听 `onsink`／`onfallinvoid` 的模组现在能收到通用事件（事件本身不保证进状态）。 | 旧:scripts/stategraphs/SGmossling.lua:28-33、:432-437；新:scripts/stategraphs/SGmossling.lua:28-35、:434-439 | 已核验 |
| SB-4 | `SGkrampus.lua` taunt 态 onenter 顶部插入一行**被注释**的重复 taunt 音效调用（+2 行，含空行）。 | 死代码，无行为变化；仅行号偏移。 | 旧:scripts/stategraphs/SGkrampus.lua:121-128；新:scripts/stategraphs/SGkrampus.lua:121-130 | 已核验 |
| SB-5 | `brains/bird_mutant_brain.lua`（月盲鸦 bird_mutant／Misshapen Bird 行为树）：① `BreakSkeletons` 搜寻半径 1.25→1.3，且破坏玩家骨架（`playerskeleton`+`HAMMER_workable`）成功时返回 `true`；② 行为树重组：原 `SequenceNode{BreakSkeletons, AttackWall, ResetCooldown}`（AttackWall 前版本已被注释）删除，改为 `IfNode(BreakSkeletons, "Break Skeletons", ActionNode(ResetCooldown))`——有骨架才破坏并重置冷却，AttackWall 分支移除。 | 消费月盲鸦行为时序或给 `bird_mutant` 换行为树的模组：破坏骨架现在是有条件的独立节点（无骨架时跳过整个分支）；半径放宽到 1.3。 | 旧:scripts/brains/bird_mutant_brain.lua:56-62、:90-100；新:scripts/brains/bird_mutant_brain.lua:56-63、:91-101 | 已核验 |
