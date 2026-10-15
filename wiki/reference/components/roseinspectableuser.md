---
title: "roseinspectableuser"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "由使用者执行玫瑰眼镜对象／地点检查，管理私人残影、激活效果和冷却任务，说明装备门槛与存读清理分工。"
aliases: ["roseinspectableuser", "玫瑰眼镜残影", "玫瑰检查冷却", "SetRoseInpection", "地点藤蔓桥检查"]
tags: [component, lifecycle, callback, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/roseinspectableuser.lua", "components/roseinspectable.lua", "prefabs/winona.lua", "prefabs/hats.lua", "prefabs/charlieresidue.lua", "componentutil.lua", "prefabs/player_classified.lua", "tuning.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# roseinspectableuser

## 快速判断

- 职责与适用：装在检查者上，保存当前目标或地点、当前残影与冷却任务；眼镜上的 [[reference/components/closeinspector]] 只负责委托，目标上的 [[reference/components/roseinspectable]] 只提供资格与效果回调。原版Winona在非lavaarena的master_postinit安装本组件，不是点技能时才创建。[winona.lua:189-217](../../../raw/current/scripts/prefabs/winona.lua#L189-L217) [hats.lua:5085-5097](../../../raw/current/scripts/prefabs/hats.lua#L5085-L5097)
- 优先使用：保留眼镜技能／卸装接线后修改实际检查与激活效果；自然流程先检查生成残影，激活该残影时才执行对象／地点效果。两步成功含义不同。[roseinspectableuser.lua:34-79](../../../raw/current/scripts/components/roseinspectableuser.lua#L34-L79) [roseinspectableuser.lua:163-202](../../../raw/current/scripts/components/roseinspectableuser.lua#L163-L202)
- 重要副作用：每次SpawnResidue先让旧残影衰退，新残影以检查者为classified接收者；激活前残影自己先Decay，再回调本组件。正在冷却仍可执行不要求冷却的效果，不能当全局禁止检查开关。[roseinspectableuser.lua:34-59](../../../raw/current/scripts/components/roseinspectableuser.lua#L34-L59) [roseinspectableuser.lua:90-109](../../../raw/current/scripts/components/roseinspectableuser.lua#L90-L109) [charlieresidue.lua:80-105](../../../raw/current/scripts/prefabs/charlieresidue.lua#L80-L105) [charlieresidue.lua:142-149](../../../raw/current/scripts/prefabs/charlieresidue.lua#L142-L149)
- 修改边界：Try方法不包办装备／技能门槛；本组件直接生成需要兼容charlieresidue与玩家视觉环境。组件删除只收尾当前冷却任务，不主动清当前目标／地点或ForceDecay残影。[roseinspectableuser.lua:8-14](../../../raw/current/scripts/components/roseinspectableuser.lua#L8-L14) [charlieresidue.lua:80-105](../../../raw/current/scripts/prefabs/charlieresidue.lua#L80-L105)

## 关键接口

| 接口／字段 | 参数／返回及条件 | 作用与证据 |
|---|---|---|
| `TryToDoRoseInspectionOnTarget(target)` | 已有实体目标；返回true或false,reason | 依次检查残影dismiss、公共目标类型、禁止标签、目标组件、生成资格及该效果的冷却意向；成功只设目标／生成残影并说检查台词。[roseinspectableuser.lua:161-195](../../../raw/current/scripts/components/roseinspectableuser.lua#L161-L195) |
| `TryToDoRoseInspectionOnPoint(pt)` | 有Get()的地点值；返回true | 不在此重新跑地点配置或冷却门槛，只设地点、生成残影并说话；正常可选地点由公共动作入口筛选。[roseinspectableuser.lua:197-202](../../../raw/current/scripts/components/roseinspectableuser.lua#L197-L202) [componentutil.lua:717-763](../../../raw/current/scripts/componentutil.lua#L717-L763) |
| `SetRoseInpectionOnTarget(target)`／`SetRoseInpectionOnPoint(point)` | 原API拼写为Inpection | 清另一种目标，直接生成残影；目标分支再HookupResidue并让残影监听目标onremove。它们绕过Try的门槛，不是安全资格检查。[roseinspectableuser.lua:63-79](../../../raw/current/scripts/components/roseinspectableuser.lua#L63-L79) |
| `OnCharlieResidueActivated(residue)` | 仅处理与self.residue同一实体 | 对象效果再次判断冷却后委托目标；地点则扫描配置。最后清本次target或point，不把对象回调结果当成功标志。[roseinspectableuser.lua:34-59](../../../raw/current/scripts/components/roseinspectableuser.lua#L34-L59) |
| `ForceDecayResidue()` | 当前残影存在时调用Decay | 先撤使用者监听残影onremove的清引用回调，不立即置self.residue=nil；正常SpawnResidue随后用新引用替换它，避免旧残影移除清掉新引用。[roseinspectableuser.lua:83-109](../../../raw/current/scripts/components/roseinspectableuser.lua#L83-L109) |
| `SetCooldownTime(seconds)`／`GoOnCooldown()` | 默认半游戏日；前者不重排当前任务 | Go先取消当前任务，再在cooldowntime非nil时ApplyCooldown。原版默认240秒；不要把nil作为已完整清理网络冷却标记的禁用协议。[roseinspectableuser.lua:18-30](../../../raw/current/scripts/components/roseinspectableuser.lua#L18-L30) [tuning.lua:19-20](../../../raw/current/scripts/tuning.lua#L19-L20) [tuning.lua:7235](../../../raw/current/scripts/tuning.lua#L7235 |
| `ApplyCooldown(duration)`／`OnCooldown()`／`IsInCooldown()` | 低层排任务／完成回调／查任务非nil | Apply不先取消旧任务；OnCooldown清任务字段并写网络false，不负责取消一个仍在排队的旧任务。一般重置走GoOnCooldown。[roseinspectableuser.lua:206-227](../../../raw/current/scripts/components/roseinspectableuser.lua#L206-L227) |
| `DoRoseInspectionOnPoint()` | 返回“是否诱发冷却”，不等于效果成功 | 按配置顺序执行首个成功且允许的回调，返回该项will_cooldown；无冷却的成功与无效果都可能返回false。[roseinspectableuser.lua:113-138](../../../raw/current/scripts/components/roseinspectableuser.lua#L113-L138) |

## 关键流程

### 对象检查与残影激活

对象Try遇到charlieresidue先ForceDecay当前残影，再返回`ROSEGLASSES_DISMISS`；其他对象排除公共门槛所禁类型，以及lunar_aligned／notroseinspectable标签。必须有roseinspectable，生成资格false报STUMPED，需要冷却且已在冷却时报COOLDOWN，其余失败为INVALID。公共类型门槛排除非零质量Physics、locomotor、inventoryitem及character；完整装备／动作分派复用closeinspector页。[roseinspectableuser.lua:161-195](../../../raw/current/scripts/components/roseinspectableuser.lua#L161-L195) [componentutil.lua:699-715](../../../raw/current/scripts/componentutil.lua#L699-L715)

对象设定后先生成残影，再调目标HookupResidue(owner,residue)；残影跟踪目标移除并衰退。激活时本组件只接受当前残影，重新取目标组件和WillInduceCooldownOnActivate：无需冷却或当前未冷却才DoRoseInspection；若要求冷却，调用效果后即GoOnCooldown，不查目标回调返回值。目标冷却意向可能在创建与激活之间改变，因此不能只检查第一次结果。[roseinspectableuser.lua:34-72](../../../raw/current/scripts/components/roseinspectableuser.lua#L34-L72)

### 地点配置的选择和返回值

地点Try本身不筛选地点；正常入口的CLOSEINSPECTORUTIL.IsValidPos先扫描ROSEPOINT_CONFIGURATIONS。当前原版只有Vine Bridge配置，check委托Bridge_DeployCheck_Helper，callback按返回spots向vinebridgemanager排创建／销毁；配置未设置forcedcooldown或cooldownfn，故本组件这条地点激活路径不因该配置主动GoOnCooldown。这里不把它推导为所有地图动作或桥梁效果都永远不触发冷却。[componentutil.lua:654-692](../../../raw/current/scripts/componentutil.lua#L654-L692) [componentutil.lua:717-738](../../../raw/current/scripts/componentutil.lua#L717-L738)

真正激活地点残影时，先按forcedcooldown非nil优先、否则cooldownfn的顺序求will_cooldown，再执行checkfn(inst,point)得到success,data；条件命中且允许时调用callbackfn(inst,point,data)并返回will_cooldown。被冷却拦下的配置会继续试后续配置，最终需要时说冷却失败台词。callbackfn的返回值被忽略；外层只在该方法返回真时GoOnCooldown。[roseinspectableuser.lua:54-58](../../../raw/current/scripts/components/roseinspectableuser.lua#L54-L58) [roseinspectableuser.lua:113-138](../../../raw/current/scripts/components/roseinspectableuser.lua#L113-L138)

可选cooldownfn分支有当前源码限制：使用者在声明`local success,data`之前就把data传给cooldownfn，不能假设第三参是checkfn本轮结果；公共IsValidPos该分支还引用未在函数内绑定的self.inst／self.point／data。默认Vine Bridge不走这两支；模组新增此类函数前须另行解决并验证，不能直接当作已核验扩展入口。[roseinspectableuser.lua:116-126](../../../raw/current/scripts/components/roseinspectableuser.lua#L116-L126) [componentutil.lua:717-733](../../../raw/current/scripts/componentutil.lua#L717-L733)

地图残影另有动作路径：DoCharlieResidueMapAction在虫洞／触手有效分支只让残影衰退，其他失败落到DecayCharlieResidueAndGoOnCooldownIfItExists。它与对象／地点激活方法是不同冷却入口，本页不展开地图选出口机制。[actions.lua:3002-3040](../../../raw/current/scripts/actions.lua#L3002-L3040) [componentutil.lua:584-598](../../../raw/current/scripts/componentutil.lua#L584-L598)

### 残影生命周期与私人表现

SpawnResidue先让旧残影衰退，再创建charlieresidue；对象点取目标位置并SetTarget，地点取point:Get，最后SetFXOwner(inst)。charlieresidue初始有下一任务自删安排，SetFXOwner及时取消它，将网络classified目标设为owner并监听owner移除。owner当前没有玫瑰眼镜视觉时立即Decay；有视觉则监听技能winona_charlie_1停用或roseglassesvision关闭来Decay。[roseinspectableuser.lua:83-109](../../../raw/current/scripts/components/roseinspectableuser.lua#L83-L109) [charlieresidue.lua:80-105](../../../raw/current/scripts/prefabs/charlieresidue.lua#L80-L105) [charlieresidue.lua:210-223](../../../raw/current/scripts/prefabs/charlieresidue.lua#L210-L223)

残影激活先Decay再通知doer的使用者组件；Decay幂等设置退出动画状态、NOCLICK并延后0.5秒移除。客户端FX从目标转发交互到classified残影，结束时清转发并自行播放退出动画；残影本体不存档，睡眠直接移除。用户组件移除不主动Decay，但owner实体移除、视觉／技能变化等由残影自己处理。[charlieresidue.lua:42-72](../../../raw/current/scripts/prefabs/charlieresidue.lua#L42-L72) [charlieresidue.lua:107-149](../../../raw/current/scripts/prefabs/charlieresidue.lua#L107-L149) [charlieresidue.lua:201-223](../../../raw/current/scripts/prefabs/charlieresidue.lua#L201-L223) [roseinspectableuser.lua:8-14](../../../raw/current/scripts/components/roseinspectableuser.lua#L8-L14)

### 冷却任务、网络与存读

ApplyCooldown把任务挂在检查者上并将player_classified.roseglasses_cooldown置true；任务到期OnCooldown置false，IsInCooldown仅看当前任务字段。classified用net_bool同步，变脏后发roseglassescooldownchanged。反复直接Apply会留下旧任务，提前清当前字段；GoOnCooldown才先取消已知任务。[roseinspectableuser.lua:22-30](../../../raw/current/scripts/components/roseinspectableuser.lua#L22-L30) [roseinspectableuser.lua:206-227](../../../raw/current/scripts/components/roseinspectableuser.lua#L206-L227) [player_classified.lua:504-510](../../../raw/current/scripts/prefabs/player_classified.lua#L504-L510) [player_classified.lua:1441](../../../raw/current/scripts/prefabs/player_classified.lua#L1441

OnSave仅保存正的任务剩余秒数cooldown，不保存target、point、residue、cooldowntime配置或台词间隔；OnLoad有cooldown则ApplyCooldown。LongUpdate减去dt，取消旧任务后按正剩余重排，否则OnCooldown；组件删除也取消当前任务并结束标记。修改cooldowntime只影响以后GoOnCooldown，不改已排剩余时间。[roseinspectableuser.lua:8-30](../../../raw/current/scripts/components/roseinspectableuser.lua#L8-L30) [roseinspectableuser.lua:231-262](../../../raw/current/scripts/components/roseinspectableuser.lua#L231-L262)

## 典型接入

原版Winona在普通服务端角色初始化就有使用者组件；眼镜装备者激活winona_charlie_1时，眼镜才动态安装closeinspector并接对象／地点委托，技能变化会刷新，卸装撤监听及检查入口。用户组件存在不等于当前能用玫瑰检查；已有宿主的真实装备／技能生命周期见 [[reference/components/closeinspector#修改玫瑰眼镜效果时保留技能与卸装边界]]。[winona.lua:189-217](../../../raw/current/scripts/prefabs/winona.lua#L189-L217) [hats.lua:5085-5139](../../../raw/current/scripts/prefabs/hats.lua#L5085-L5139)

这是服务端原版接线参照，不另造直接生成残影的demo。只改特定对象效果时，优先沿目标侧回调；只改冷却配置时确认是否要影响下次激活或当前任务，并保留残影与视觉关闭时的清理分工。

## 常见陷阱与限制

- **强制冷却false仍会执行判断回调：** 非nil强制值已覆盖回调；目标侧接口与签名只在 [[reference/components/roseinspectable]] 维护。[roseinspectable.lua:39-49](../../../raw/current/scripts/components/roseinspectable.lua#L39-L49)
- **台词被抑制就是检查失败：** 非失败DoQuip有4～5秒台词间隔，间隔内发silentcloseinspect后返回；Try仍能生成残影并返回true。失败台词不走这段间隔。[roseinspectableuser.lua:142-159](../../../raw/current/scripts/components/roseinspectableuser.lua#L142-L159)
- **把cooldowntime设nil就完全禁用并清标记：** Go会取消任务但在nil时不Apply也不OnCooldown，已有网络true可能保留；不能把这一调用组合当已验证的完整取消接口。[roseinspectableuser.lua:18-30](../../../raw/current/scripts/components/roseinspectableuser.lua#L18-L30) [roseinspectableuser.lua:210-223](../../../raw/current/scripts/components/roseinspectableuser.lua#L210-L223)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅使用者及目标组件全文、Winona安装、眼镜委托／技能刷新、charlieresidue全文、公共目标／地点门槛及Vine Bridge配置／回调、地图残影动作函数、classified冷却同步与默认时长，覆盖两步流程、对象／地点差异、残影与冷却生命周期。
- 游戏运行：未运行；未复现可选cooldownfn缺口、重复Apply任务或nil配置网络残留，未测多人视觉、目标移除竞态、读档／LongUpdate及桥梁完整生成效果。
- 结构检查：`validate_page.py`通过；两页合并执行`validate_source_links.py --strict`，78个行号引用、0错误／0警告。本页无新增Lua示例；机械检查不代替游戏测试。
