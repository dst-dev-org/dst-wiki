---
title: "grouptargeter：加权目标集合与换目标尝试"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "维护实体目标的权重，通过独立概率门控制尝试选取，再由调用者决定是否接纳战斗目标。"
aliases: [grouptargeter, 加权选择目标, 换目标尝试概率, 蜂后玩家目标池, 目标权重分配]
tags: [component, lifecycle, ai, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/grouptargeter.lua, prefabs/beequeen.lua, brains/beequeenbrain.lua]
reliability: verified
validation: source-reviewed
status: active
---

# grouptargeter：加权目标集合与换目标尝试

## 快速判断

- 职责与适用：维护 `targets[实体]=权重`、成员数和当前尝试概率；既不自行搜敌或轮询，也不直接改 combat.target。原版蜂后由重选目标回调维护候选并消费结果。[grouptargeter.lua:1-16](../../../raw/current/scripts/components/grouptargeter.lua#L1-L16)、[beequeen.lua:136-188](../../../raw/current/scripts/prefabs/beequeen.lua#L136-L188)
- 优先使用：需要“失败后提高下次尝试概率，通过后按权重选候选并调整权重”的策略时查看 TryGetNewTarget；只需加权抽样时 SelectTarget 是另一个入口，不自动执行权重调整。[grouptargeter.lua:117-148](../../../raw/current/scripts/components/grouptargeter.lua#L117-L148)
- 重要副作用：OnPickTarget 降低所选目标权重，向其他目标均摊，但不校验成员或钳制权重；负值、总和漂移和选不到目标都必须由接入方考虑。[grouptargeter.lua:40-56](../../../raw/current/scripts/components/grouptargeter.lua#L40-L56)、[grouptargeter.lua:132-148](../../../raw/current/scripts/components/grouptargeter.lua#L132-L148)
- 修改边界：原版蜂后仅在服务端添加。组件只监听目标 onremove，不监听死亡、阵营或距离变化；没有网络字段、存读或内建更新任务，资格过滤与重建由宿主负责。[beequeen.lua:404-406](../../../raw/current/scripts/prefabs/beequeen.lua#L404-L406)、[beequeen.lua:451](../../../raw/current/scripts/prefabs/beequeen.lua#L451)、[grouptargeter.lua:1-150](../../../raw/current/scripts/components/grouptargeter.lua#L1-L150)

## 关键接口

| 接口／字段 | 参数、返回／默认值 | 条件、副作用与证据 |
|---|---|---|
| `AddTarget(target)`／`RemoveTarget(target)` | 支持事件的实体；无显式返回 | 重复 add／缺席 remove 无操作；维护数量、权重与 onremove 监听，具体再分配见下文。[grouptargeter.lua:58-103](../../../raw/current/scripts/components/grouptargeter.lua#L58-L103) |
| `GetTargets()`／`GetNumTargets()` | 内部权重表／维护计数 | GetTargets 直接返回原表，不是副本；外部增删表项会绕过计数和监听维护。[grouptargeter.lua:105-111](../../../raw/current/scripts/components/grouptargeter.lua#L105-L111) |
| `IsTargeting(target)` | 成员布尔值 | 仅判非 nil 且表项存在；零／负权重仍是成员，也不代表当前 combat 正在攻击它。[grouptargeter.lua:113-115](../../../raw/current/scripts/components/grouptargeter.lua#L113-L115) |
| `SelectTarget()` | 返回候选实体或无返回值 | 抽0–1范围随机数，按 pairs 遍历的累计区间取目标；不改 current_chance 或权重，不校验目标资格。[grouptargeter.lua:132-148](../../../raw/current/scripts/components/grouptargeter.lua#L132-L148) |
| `OnPickTarget(target)`、`weight_change` | 默认变化量0.1 | 成员数≤1时直接返回；否则所选者减变化量，所有其他成员各加 `weight_change/(N-1)`，无范围保护或成员验证。[grouptargeter.lua:5](../../../raw/current/scripts/components/grouptargeter.lua#L5)、[grouptargeter.lua:40-56](../../../raw/current/scripts/components/grouptargeter.lua#L40-L56) |
| `TryGetNewTarget()` | 目标或 nil | 先判概率；通过就重置概率、SelectTarget、OnPickTarget，再返回；失败才提高概率，成功尝试不保证抽到实体。[grouptargeter.lua:117-130](../../../raw/current/scripts/components/grouptargeter.lua#L117-L130) |
| `min_chance`／`max_chance`／`chance_delta`／`current_chance` | 0／0.7／0.1／0 | 概率门严格使用 random < current_chance；失败把 current+delta clamp 到 min/max。配置是直接字段，没有 setter 或参数校验，非按时间自动增长。[grouptargeter.lua:8-11](../../../raw/current/scripts/components/grouptargeter.lua#L8-L11)、[grouptargeter.lua:117-130](../../../raw/current/scripts/components/grouptargeter.lua#L117-L130) |
| `GetTotalWeight()`、`total_weight` | 方法仅打印；字段初始1 | 方法现场求和并 print，没有返回值；SelectTarget 不读取 total_weight 来缩放随机范围，改该字段不会自动归一化。[grouptargeter.lua:4](../../../raw/current/scripts/components/grouptargeter.lua#L4)、[grouptargeter.lua:32-38](../../../raw/current/scripts/components/grouptargeter.lua#L32-L38)、[grouptargeter.lua:132-148](../../../raw/current/scripts/components/grouptargeter.lua#L132-L148) |

## 关键流程

### 新增、移除与选择后的权重

空池新增目标给权重1；已有成员时，新的总数记为 N，每个旧权重乘 `(N-1)/N`，减少的总量成为新目标权重。旧权重总和为1时，新目标因此得到1/N；它不会把已有成员强制改为等权。移除目标则把它原权重均分加回剩余成员，最后一个移除时清空表、计数归零。[grouptargeter.lua:58-103](../../../raw/current/scripts/components/grouptargeter.lua#L58-L103)

OnPickTarget 是独立的反馈步骤：在目标确属集合且 N>1 时，一减多加使总量按公式保持不变。例如两名目标各0.5，选 A 后为0.4／0.6；持续调用可以令某项为负，代码没有把它截为0。SelectTarget 本身只抽样，不执行这一步，调用者不能把两方法混作同一个行为。[grouptargeter.lua:40-56](../../../raw/current/scripts/components/grouptargeter.lua#L40-L56)、[grouptargeter.lua:132-148](../../../raw/current/scripts/components/grouptargeter.lua#L132-L148)

### 尝试概率与清理

默认 current_chance=0，首次 Try 必走失败分支并增到0.1；此后失败按 delta 增长至 max，概率通过则先回到 min，再抽样并调整权重。空集合仍可消耗一次通过，返回 nil；抽样返回 nil 时也会继续 OnPickTarget(nil)，没有额外成功目标检查。[grouptargeter.lua:8-11](../../../raw/current/scripts/components/grouptargeter.lua#L8-L11)、[grouptargeter.lua:117-130](../../../raw/current/scripts/components/grouptargeter.lua#L117-L130)

新增监听目标自身 onremove，收到事件后 RemoveTarget 并重新分配；单独 StartTracking／StopTracking 只增删监听，不等于登记／移除。OnRemoveFromEntity 遍历撤所有目标监听，没有调用 RemoveTarget 清表、计数或概率。组件没有 OnSave／OnLoad，宿主重建时须重新建立目标集合和配置，不能期待保留跨存档选择历史。[grouptargeter.lua:13-30](../../../raw/current/scripts/components/grouptargeter.lua#L13-L30)、[grouptargeter.lua:58-103](../../../raw/current/scripts/components/grouptargeter.lua#L58-L103)、[grouptargeter.lua:1-150](../../../raw/current/scripts/components/grouptargeter.lua#L1-L150)

## 典型接入

### 蜂后：候选范围、换目标策略与 Brain 读取分工

蜂后服务端添加 grouptargeter，并给 combat 注册3秒 RetargetFn。回调先 UpdatePlayerTargets：围绕记忆 spawnpoint（不存在则自身坐标）调用 FindPlayersInRange，再对现有集合做增删。因此候选维护由宿主回调推进，不是组件自行持续扫描。[beequeen.lua:136-167](../../../raw/current/scripts/prefabs/beequeen.lua#L136-L167)、[beequeen.lua:404-406](../../../raw/current/scripts/prefabs/beequeen.lua#L404-L406)、[beequeen.lua:435-451](../../../raw/current/scripts/prefabs/beequeen.lua#L435-L451)

当前目标是玩家时，RetargetFn 才走 TryGetNewTarget，然后再检查新候选到蜂后的距离；抽中却不满足距离时仍可能不被接纳，但概率和权重已在 Try 内更新。当前不是玩家目标时，宿主则从集合中过滤近处玩家，均匀随机选一个，不走权重抽样。不要把“集合中的权重”理解为蜂后所有选敌分支都使用的最终攻击概率。[beequeen.lua:166-188](../../../raw/current/scripts/prefabs/beequeen.lua#L166-L188)

BeeQueenBrain 的 CalcDodgeMult 也遍历同一表的键：近攻击范围内至少两名候选时返回0.5，否则1，供 ShouldDodge 的等待时间判断使用；此消费不使用权重数值。目标池既能参与加权重选，也能仅作为实体集合读取。[beequeenbrain.lua:103-114](../../../raw/current/scripts/brains/beequeenbrain.lua#L103-L114)、[beequeenbrain.lua:120-134](../../../raw/current/scripts/brains/beequeenbrain.lua#L120-L134)

## 常见陷阱与限制

- **非成员／nil 的 OnPick 仍会改表**：N>1时若 target 不在集合，没有项会减权，所有项都会加 `weight_change/(N-1)`，总和因而漂移；Try 对 SelectTarget 返回 nil 也会走这条路径。传入有效成员并不是该方法自身提供的保证。[grouptargeter.lua:40-56](../../../raw/current/scripts/components/grouptargeter.lua#L40-L56)、[grouptargeter.lua:117-130](../../../raw/current/scripts/components/grouptargeter.lua#L117-L130)
- **总和为1不等于安全概率分布**：权重可为负，SelectTarget 又按固定0–1随机数和累计开闭区间遍历；负值会使累计值回退，缺口或总量不匹配可导致不返回目标，没有末项兜底。不能把负权重当作自动过滤、把 total_weight 当归一化开关。[grouptargeter.lua:40-56](../../../raw/current/scripts/components/grouptargeter.lua#L40-L56)、[grouptargeter.lua:132-148](../../../raw/current/scripts/components/grouptargeter.lua#L132-L148)
- **在 GetTargets 返回表上直接删成员**：会遗漏 onremove 监听、num_targets 和权重再分配；使用 AddTarget／RemoveTarget 管理成员，资格变化也由宿主显式维护，组件不会因目标死亡或超距自动拒选。[grouptargeter.lua:24-30](../../../raw/current/scripts/components/grouptargeter.lua#L24-L30)、[grouptargeter.lua:58-115](../../../raw/current/scripts/components/grouptargeter.lua#L58-L115)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、权重公式与无 clamp／非成员缺口、概率门与独立选择、监听／移除／存读边界；蜂后服务端安装、候选同步及 combat 重选消费者、Brain 按成员数量调整躲避等待的消费者。
- 游戏运行：未运行；示例权重及总量漂移为源码推导，未实测负权重抽样、浮点边界、空池通过、目标移除或真实重选频率；未审阅其他宿主、完整 combat 调度或全部蜂后 AI。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；无新增 Lua 示例，未运行语法探针。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/combat]]、[[reference/components/commander]]。
