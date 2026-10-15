# FX 新增页独立检视：第 1 轮

日期：2026-11-05。源码：`raw/current/scripts`，实读 `steam-buildid:26335394`。

范围按版本更新统一收口后的新增页计算，本轮对应原生成批 1～2，共 **8 页、74 个通用表注册**。此前已独立检视 U15～U17 九页；这两部分互不重复。写作者／其他代理的历史复审记录只作线索，不作为本轮独立验收依据。

结论：**需局部修复；P1×0、P2×7、P3×2**。未发现需要重写整族协议的问题。仅新增本报告并追加日志，没有修知识页、record、改变台账验收状态、提交或推送；未运行游戏。下面涉及特殊时序的结果均为源码路径推导，未冒充游戏复现。

以上及下方 R01～R09 保留首次检视记录；修改后的最新状态见文末“修后复验”。

## 本轮范围与逐页结论

页面路径均相对 `wiki/reference/prefabs/`。

| 页面 | 注册数 | 已核对的主要范围 | 结论 |
|---|---:|---|---|
| `leafpile_spawn_fx.md` | 6 | 三组颜色、动态拼名；放置／生长／碎裂；休眠门差异、59 帧同步、Decay／烧毁与 FX 回收对象 | 未发现 P1/P2 |
| `halloween_firepuff_1.md` | 7 | 冷热选组与共享模块复用；投火 fuel 回调、代理父挂与符号跟随；月药水失败落点；两处月玻璃再生回调 | R01（P2） |
| `pumpkincarving_shatter_fx.md` | 1 | 蔬菜／帽子两处实际生成；切口校验、持有／休眠门；雪人注释与依赖声明不算生成 | 未发现 P1/P2 |
| `snowball_shatter_fx.md` | 1 | 融尽／清场／锤碎三处生成；温度检查与积雪门；投掷使用另一 FX；清理对象 | 未发现 P1/P2 |
| `ghostlyelixir_slowregen_fx.md` | 36 | 注册资源、符号与音效；buff attach／drip／extend／detach；玩家饮用状态、槽位、普通／super 药分流、官方药名 | R02（P2） |
| `ghostflower_spirit1_fx.md` | 2 | 同构双名、动态随机选名；花的自续任务、墓碑固定 1 号；官方宿主名与来源声明 | R04、R05（P2），R08（P3） |
| `wathgrithr_spirit.md` | 14 | 击杀升灵门禁；受歌者与歌手两种循环；瞬时歌消费点；复活保护／阿比盖尔复用；任务归属和移除 | R03、R06（P2） |
| `wanda_attack_pocketwatch_old_fx.md` | 7 | 年龄与武器标签选名；未格挡回调窗口；治疗／复活落点、骑乘变体与施法状态；同名 bank 辨析 | R07（P2），R09（P3） |

共用加载器 `prefabs/fx.lua` 另核对了代理／本地实体分工、父挂、随机 anim 求值、音效调度及两种回收；没有把 `SetParent` 直接解释为 Lua children 登记。74 个注册名均在通用表中存在，简中 `.po` 的对应 `STRINGS.NAMES.<注册名大写>` 均未命中；页面实际引用的宿主／药水显示名另按 `.po` 原键核对。

## 待修问题

### R01 · P2：月玻璃恢复没有限定 fullmoon 为真

- 页面：[halloween_firepuff_1.md:42](../../../wiki/reference/prefabs/halloween_firepuff_1.md#L42)；摘要第 6 行也写成“洞穴月圆月玻璃再生”。第 42 行还把监听动作归给 `set_full`。
- 源码：[池面](../../../raw/current/scripts/prefabs/grotto_pool_moonglass.lua#L32-L48)、[小瀑布](../../../raw/current/scripts/prefabs/grotto_waterfall_small.lua#L31-L47)：采空函数登记 `WatchWorldState("iscavefullmoon", ...)`，回调虽接收 `fullmoon`，却不检查它，直接按概率恢复并取消监听。[worldstate](../../../raw/current/scripts/components/worldstate.lua#L24-L46) 在变量值变化时把新值传给普通变量监听；[夜晚状态更新](../../../raw/current/scripts/components/worldstate.lua#L63-L69) 可将该布尔量改为 false。
- 建议：写成“采空后监听 iscavefullmoon 的变化；回调未区分 true／false，概率命中后恢复并生成月绒”。不要仅凭回调名字补出“仅进入满月夜”的门禁；同步摘要与导航行。

### R02 · P2：revive 的滴落不能概括为实际不触发

- 页面：[ghostlyelixir_slowregen_fx.md:114](../../../wiki/reference/prefabs/ghostlyelixir_slowregen_fx.md#L114) 由单次时长短于首滴得出“实际不触发”；同页第 63 行已正确写出续杯保留 driptask，结论冲突。
- 源码：[首滴任务](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L489-L493) 为 3.75 秒；[同槽 Extend](../../../raw/current/scripts/components/debuffable.lua#L95-L105) 重用 buff；[OnExtended](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L511-L532) 重设 decay 而不重设 driptask。[实际药水入口](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L348-L365) 同样能到达此分支。
- 反例路径：有效非玩家目标的同一 revive buff 在到期前反复续期，使总存活跨过 3.75 秒；若该次回调目标不在 limbo、SG 非 busy，滴落仍可生成。此路径不要求修改 FX 条目。
- 建议：限定为“未延长的单次基础时长下，buff 先于首滴到期；连续 Extend 保留原首滴任务，不能据此判定 revive drip 不可达”。不要把它列为无生成点或死注册。

### R03 · P2：战歌谱号任务的字段持有者与实际任务宿主需要分开

- 页面：[wathgrithr_spirit.md:53](../../../wiki/reference/prefabs/wathgrithr_spirit.md#L53) 只描述“在受歌目标上挂 battlesong_fx_task”，未说明任务实际归属于创建它的 buff，读者容易据目标字段理解其存活期。
- 源码：[CheckForLoopFx](../../../raw/current/scripts/prefabs/battlesongs.lua#L198-L214) 是 `target.battlesong_fx_task = inst:DoPeriodicTask(...)`，`inst` 为 buff。[detach](../../../raw/current/scripts/prefabs/battlesongs.lua#L272-L300) 仅最后一层清空目标字段，却每次都会移除当前 buff；[任务登记](../../../raw/current/scripts/entityscript.lua#L1537-L1544) 与 [Remove](../../../raw/current/scripts/entityscript.lua#L1730-L1746) 表明移除 buff 会取消其 pendingtasks。
- 建议：注明“引用存于 target，任务由某个 buff 创建”。若创建者先于其他歌曲 buff 移除，其任务也会取消，而目标字段可能仍非 nil；这与最后一层显式清理是不同路径。保留为源码边界，不声称已复现音符中断。复用周期任务时按真实宿主设计取消与字段清空。

### R04 · P2：幽灵花拾入并不会立即取消飘光任务

- 页面：[ghostflower_spirit1_fx.md:37](../../../wiki/reference/prefabs/ghostflower_spirit1_fx.md#L37) 写“拾入背包后链条停止，再落地重启”，遗漏任务要到执行时才检查 limbo 的时序。
- 源码：[dofx](../../../raw/current/scripts/prefabs/ghostflower.lua#L14-L21) 仅在回调执行时检查 `inlimbo`，未保存任务句柄；[toground](../../../raw/current/scripts/prefabs/ghostflower.lua#L34-L39) 每次另排任务，[ondropped 接线](../../../raw/current/scripts/prefabs/ghostflower.lua#L72-L76) 无去重。[拾入](../../../raw/current/scripts/components/inventoryitem.lua#L348-L360) 与 [RemoveFromScene](../../../raw/current/scripts/entityscript.lua#L348-L375) 不取消这些 pendingtasks。
- 建议：改为“待执行回调若发现仍在 limbo，则不生成也不再续排”。同一实体在旧任务到期前拾起又丢下，旧任务可能与新落地任务并存；不要保证只有一条 3～9 秒链。堆叠合并／实体移除等其他路径不包含在这个条件反例内。

### R05 · P2：幽灵花页有两份正文证据未纳入来源依赖

- 页面：[ghostflower_spirit1_fx.md:11](../../../wiki/reference/prefabs/ghostflower_spirit1_fx.md#L11) 的 sources 未声明 `prefabs/fx.lua`、`recipes.lua`；正文第 36、39、46 行却直接引用两者，分别支持加载回收、资产与九种药水配方关系。
- 影响：行锚点检查通过不代表来源依赖完整；这两份文件后续变化不会进入本页的来源基线比较。
- 建议：保留这些直接证据时补 sources，并在整页复核后按现行规则 record；若决定将公共工厂证据完全收口到机制页，则删去本页重复的原始引用，保留机制链接。不要只为让本次检查变绿而机械删证据。

### R06 · P2：武神升灵生成条件漏了 nofadeout

- 页面：[wathgrithr_spirit.md:51](../../../wiki/reference/prefabs/wathgrithr_spirit.md#L51) 写有效受害者满足 epic 或 10% 后延迟生成，仅另列火化排除。
- 源码：[onkilled](../../../raw/current/scripts/prefabs/wathgrithr.lua#L51-L64) 在随机／epic 条件之外还要求 `not victim.components.health.nofadeout`；这不包含在 [IsValidVictim](../../../raw/current/scripts/prefabs/wathgrithr.lua#L41-L49) 内。
- 建议：同一生成条件行补上 nofadeout 排除，避免把 epic 写成绕过全部死亡表现门禁的充分条件。

### R07 · P2：治疗生成前提漏了 oldager

- 页面：[wanda_attack_pocketwatch_old_fx.md:46](../../../wiki/reference/prefabs/wanda_attack_pocketwatch_old_fx.md#L46) 将治疗 FX 前提写为“doer 有 health 且未死”。
- 源码：[Heal_DoCastSpell](../../../raw/current/scripts/prefabs/pocketwatch.lua#L29-L39) 在生成前无判空调用 `doer.components.oldager:StopDamageOverTime()`；仅有存活 health 并不足以让该回调走到 FX。原版旺达满足这个条件。
- 建议：补“原版施法者还具备 oldager；不能只给自定义角色补 pocketwatchcaster 标签便认为此回调可用”。保持 FX 本身与治疗回调前提的区分，无需扩写 oldager 机制。

### R08 · P3：幽灵花验证记录仍留待登记状态

- 页面：[ghostflower_spirit1_fx.md:56](../../../wiki/reference/prefabs/ghostflower_spirit1_fx.md#L56) 仍写“叶级索引登记待主代理处理”，实际已登记且本轮导航校验通过。
- 建议：清理当前页面中的临时待办，历史批次报告原文保留。

### R09 · P3：怀表施法工厂已建页，待建提示可改互链

- 页面：[wanda_attack_pocketwatch_old_fx.md:102](../../../wiki/reference/prefabs/wanda_attack_pocketwatch_old_fx.md#L102) 仍将施法色链写为“Wiki 待建”。
- 实际：[staffcastfx.md](../../../wiki/reference/prefabs/staffcastfx.md) 第 34～39 行已覆盖 pocketwatch 两注册与 SetUp 端分界，第 47 行覆盖 castfxcolour 到施法状态的调用。
- 建议：链接已有页的对应章节，动画资源未解包等真实缺口继续保留。

## 定向校验与证据边界

- `validate_page.py --check-indexes`：所选 8 页全部 passed；导航 passed、0 issues。导航扫描全库索引关系，不代表全库内容已复审。
- `validate_source_links.py --strict`：8 页、337 处行引用、64 个源文件，0 error／0 warning，退出 0。
- `validate_source_hashes.py check --fresh-hours 0`：8 页、98 组比较，98 unchanged，0 invalid／changed／unrecorded；只读检查，未 record。此结果不校验 sources 完整性，R05 仍成立。
- 生成点定位覆盖整个 scripts 树，核对字面名、公共前缀及共享字段／模块消费点；没有用依赖表或注释充当生成证据。特别复核了 `SpawnPuffFx` 共享入口、`leafpile_` 动态拼名、`ghostflower_spirit` 动态二选一、灵药 `potion_tunings` 选名，以及战歌 `ATTACH_FX`／`LOOP_FX`／`DETACH_FX` 的消费者。
- 未运行游戏、未解包动画，未重跑已有 Lua 示例语法探针；本轮只读审查示例的端、前提、可见性与调用链，不沿用历史语法检查冒称本轮实跑。
- 输入保护：报告写入前后对 8 页、声明来源及 source-baseline 共 79 个输入文件复核哈希，全部未变；额外追读的机制文件不计入这 79 个保护样本。报告 30 处本地链接的目标／行区间检查通过，知识仓 `git diff --check` 通过。

## 剩余队列

本轮前仍待本对话独立检视 **149 页**，本轮完成 8 页后剩 **141 页**。其中“检视完成”与“问题修复完成”分开，本轮七项 P2 当前仍待修，不能据此改为已通过独立验收。

下一轮按原生成批 3：`attune_out_fx`、`beefalo_transform_fx`、`ocean_splash_med1`、`splash_green`、`splash_sink`、`round_puff_fx_sm`，共 **6 页、25 注册**。该轮尚未执行；其他轮次不混入本报告。

## 修后复验（2026-11-05）

范围：本轮实际修订的 5 页；对照首次检视快照，其他 3 页与原声明的源码输入未变化。R01～R09 的整改点均已覆盖，但新增两项 P2，尚不能将本轮标为全部通过。未扩展到下一轮、未修改知识页或 record。

| 原项 | 复验结果 |
|---|---|
| R01 | 摘要改为满月前后；正文区分采空登记、回调忽略 fullmoon 和恢复取消监听 |
| R02 | 保留单次基础时长边界，补 Extend 保留 driptask、可跨过首滴的条件 |
| R03 | 第 53 行已区分 target 字段与 buff 任务宿主，并补移除取消；第 85 行同步引入 R10 |
| R04 | 明确回调执行时才检查 limbo、旧任务与落地新任务可能并存 |
| R05 | 幽灵花页已补 prefabs/fx.lua、recipes.lua，基线检查无漂移 |
| R06 | 升灵条件补 nofadeout，说明不属于 IsValidVictim 内部条件 |
| R07 | 治疗入口补 oldager 调用前提与自定义角色边界 |
| R08 | 待登记提示已移除，链接现有索引 |
| R09 | 待建提示改为 staffcastfx 的现有章节链接 |

### R10 · P2：示例说明把两种任务宿主合并成 buff

- 当前位置：[wathgrithr_spirit.md:85](../../../wiki/reference/prefabs/wathgrithr_spirit.md#L85)。新增括号将 `target.battlesong_fx_task`／`display_fx_task` 并列后，统一说“任务实体归创建它的 buff”。
- 实际：只有目标侧 `battlesong_fx_task` 由 [buff inst 创建](../../../raw/current/scripts/prefabs/battlesongs.lua#L198-L214)。歌手侧 `self.display_fx_task` 由组件所属的歌手 `self.inst:DoTaskInTime` 创建，见 [首排](../../../raw/current/scripts/components/singinginspiration.lua#L270-L272)、[续排](../../../raw/current/scripts/components/singinginspiration.lua#L230-L238)；其句柄存在组件上，清槽 [取消并置空](../../../raw/current/scripts/components/singinginspiration.lua#L282-L297)。
- 建议：第 85 行拆清两个主体：“目标侧句柄记在 target、任务归 buff；歌手侧句柄记在 singinginspiration 组件、任务归歌手实体”。保留第 53 行正确的目标侧边界，避免把本次修正扩展到另一条链。

### R11 · P2：新增 entityscript 引用仍未纳入战歌页 sources

- 当前位置：[wathgrithr_spirit.md:10](../../../wiki/reference/prefabs/wathgrithr_spirit.md#L10) 的 sources 没有 `entityscript.lua`，而第 53 行新增 [Remove 取消任务证据](../../../raw/current/scripts/entityscript.lua#L1730-L1746)。
- 建议：将该文件加入 sources，按现有规则复核并刷新本页基线。本轮检查的 75 组 unchanged 仅代表已声明依赖，不能证明新增证据已被追踪。

复跑结果：5 页 `validate_page --check-indexes` 全部通过、导航 0 issues；`validate_source_links --strict` 检查 264 处行引用，0 error／0 warning，另 1 个非 Lua／PO 目标按工具规则跳过；`validate_source_hashes check --fresh-hours 0` 为 75 组 unchanged、0 invalid／changed／unrecorded。未运行游戏、未重跑 Lua 示例语法探针、未提交或推送。剩余独立检视队列仍为 141 页，本次为第 1 轮修后复验，不扣减下一轮页数。
