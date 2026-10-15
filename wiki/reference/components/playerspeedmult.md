---
title: "playerspeedmult：玩家自身跑速与预测倍率"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "为不应叠到坐骑基速上的玩家特性提供来源倍率、预测与可封顶加成，依赖玩家 classified 接线。"
aliases: [playerspeedmult, 玩家角色跑速, 不影响坐骑速度, 预测跑速倍率, 跑速加成封顶]
tags: [component, lifecycle, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/playerspeedmult.lua, util/sourcemodifierlist.lua, components/locomotor.lua, prefabs/player_common.lua, prefabs/player_classified.lua, prefabs/wormwood.lua, entityscript.lua]
reliability: verified
validation: source-reviewed
status: active
---

# playerspeedmult：玩家自身跑速与预测倍率

## 快速判断

- 职责与适用：覆盖玩家自己的 `locomotor.runspeed`，支持本地预测的 `predictrunspeed`；服务端和客户端的 locomotor 都在骑乘时优先取坐骑跑速，因此这些玩家基速变化不叠到坐骑基速上。[playerspeedmult.lua:178-199](../../../raw/current/scripts/components/playerspeedmult.lua#L178-L199)、[locomotor.lua:91-114](../../../raw/current/scripts/components/locomotor.lua#L91-L114)
- 优先使用：角色自身特性／预测跑步状态等“应忽略坐骑基速”的加成；普通独立移动效果沿用 [[reference/components/locomotor#关键接口]] 的来源倍率，装备效果优先沿用装备接线。不要把此组件当作任意实体的通用速度组件。[player_common.lua:2497-2506](../../../raw/current/scripts/prefabs/player_common.lua#L2497-L2506)
- 重要副作用：四组来源倍率相乘，修改时立即重算基速；可封顶组还受其他 locomotor 倍率影响，但封顶不能压低不可封顶组。`SetSpeedMultCap` 本身只赋值，不触发重算。[playerspeedmult.lua:65-68](../../../raw/current/scripts/components/playerspeedmult.lua#L65-L68)、[playerspeedmult.lua:178-199](../../../raw/current/scripts/components/playerspeedmult.lua#L178-L199)
- 修改边界：原版玩家公共构造在客户端返回前添加组件并设 cap=2，服务端随后绑定 `player_classified`。服务端 setter／初始化缓存直接访问 classified，不能仅 AddComponent 就认定依赖已齐。[player_common.lua:2505-2506](../../../raw/current/scripts/prefabs/player_common.lua#L2505-L2506)、[player_common.lua:2611-2615](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2615)、[player_common.lua:2645-2649](../../../raw/current/scripts/prefabs/player_common.lua#L2645-L2649)

## 关键接口

| 接口 | 参数／端 | 条件、副作用与证据 |
|---|---|---|
| `SetSpeedMult(source, m)`／`RemoveSpeedMult(source)` | 服务端普通组 | 客户端调用无操作；设置前尝试缓存无加成基速，更新来源后重算跑速并发布 `psm_servermult`。[playerspeedmult.lua:116-142](../../../raw/current/scripts/components/playerspeedmult.lua#L116-L142) |
| `SetCappedSpeedMult(source, m)`／`RemoveCappedSpeedMult(source)` | 服务端可封顶组 | 走独立来源表，并发布 `psm_cappedservermult`；不是替代普通组，而是与之相乘后参与 cap 计算。[playerspeedmult.lua:144-159](../../../raw/current/scripts/components/playerspeedmult.lua#L144-L159) |
| `SetPredictedSpeedMult(source, m)`／`RemovePredictedSpeedMult(source)` | 两端本地普通预测组 | 只修改当前端来源表，不经这些方法发送 netvar；预测状态须由调用方协调服务端与客户端及成对撤销。[playerspeedmult.lua:161-167](../../../raw/current/scripts/components/playerspeedmult.lua#L161-L167) |
| `SetCappedPredictedSpeedMult(source, m)`／对应 `RemoveCappedPredictedSpeedMult(source)` | 两端本地可封顶预测组 | 本地重算，叠入可封顶组；同样没有自动跨端复制这些来源的逻辑。[playerspeedmult.lua:169-175](../../../raw/current/scripts/components/playerspeedmult.lua#L169-L175) |
| `SetSpeedMultCap(cap)` | `nil` 或 ≥0；组件初始为 `nil` | 断言检查并赋值；cap 是相对基速的倍率阈值，不是世界距离／秒。要让已存在加成立即按新 cap 重算，需要另一个实际重算触发。[playerspeedmult.lua:19](../../../raw/current/scripts/components/playerspeedmult.lua#L19)、[playerspeedmult.lua:65-68](../../../raw/current/scripts/components/playerspeedmult.lua#L65-L68)、[playerspeedmult.lua:185-192](../../../raw/current/scripts/components/playerspeedmult.lua#L185-L192) |
| `SetClassified(classified)`／`AttachClassified(classified)` | 前者断言服务端；后者用于客户端接线 | `SetClassified` 直接赋引用；`AttachClassified` 还监听 classified 的 `onremove`，触发 `DetachClassified` 清引用。[playerspeedmult.lua:32-46](../../../raw/current/scripts/components/playerspeedmult.lua#L32-L46) |

### 来源标识与基速缓存

`source` 为实体时作为来源本身；非实体时作为宿主来源表中的 key，原版使用稳定字符串。每组独立维护，同一来源／key 替换，不同来源相乘。`m` 是数值倍率，本层不校验其非负性或上限；底层列表默认积为 1，设置 `nil` 或 1 等同移除。实体来源会注册 `onremove` 自动移除本地项。[playerspeedmult.lua:73-87](../../../raw/current/scripts/components/playerspeedmult.lua#L73-L87)、[sourcemodifierlist.lua:8-26](../../../raw/current/scripts/util/sourcemodifierlist.lua#L8-L26)、[sourcemodifierlist.lua:64-99](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64-L99)

服务端构造安排零延迟静态初始化任务；`TryRecacheBaseSpeed_Internal` 只有在四张表都没有来源时才把当前 `locomotor.runspeed` 写入 `psm_basespeed`。服务端普通／可封顶 setter 也会在加来源前尝试该步骤；有加成时直接改 `runspeed` 不会成为新缓存基速。[playerspeedmult.lua:7-28](../../../raw/current/scripts/components/playerspeedmult.lua#L7-L28)、[playerspeedmult.lua:116-151](../../../raw/current/scripts/components/playerspeedmult.lua#L116-L151)

## 关键流程

### 四组倍率与可封顶部分

设 A=服务端普通组×本地普通预测组，B=服务端可封顶组×本地可封顶预测组，L=`locomotor:GetSpeedMultiplier()`。当 `A×B×L` 不超过 cap 或 cap 未设置时，写入基速×A×B；超限时改写为基速×`A×max(1, B×cap/(A×B×L))`。L 包括外部、装备／鞍具、地面与 throttle 的结算，参与是否裁剪加成的判断。[playerspeedmult.lua:178-199](../../../raw/current/scripts/components/playerspeedmult.lua#L178-L199)、[locomotor.lua:178-236](../../../raw/current/scripts/components/locomotor.lua#L178-L236)

由 `max(1, ...)` 可知裁剪的是 B 带来的额外加速，不会把 A 压低：例如 A=1.5、B=2、L=1、cap=2，调整后为基速×2；若 A=3、B=1，则仍为基速×3。cap 因而不是所有加速后的硬上限。该示例为公式推导，未游戏实测。[playerspeedmult.lua:182-192](../../../raw/current/scripts/components/playerspeedmult.lua#L182-L192)

### 重算、网络与退出

底层来源表 dirty 回调调用 `ApplyRunSpeed_Internal`。服务端从自身来源表取值并写 `runspeed`；客户端从 classified 读取服务端乘积，有本地预测来源时才写 `predictrunspeed`，没有则置 `nil`。locomotor 还要求玩家有 `autopredict` 标签才采用该预测跑速，否则读取 classified 的普通 `runspeed`。[playerspeedmult.lua:12-14](../../../raw/current/scripts/components/playerspeedmult.lua#L12-L14)、[playerspeedmult.lua:89-111](../../../raw/current/scripts/components/playerspeedmult.lua#L89-L111)、[playerspeedmult.lua:178-199](../../../raw/current/scripts/components/playerspeedmult.lua#L178-L199)、[locomotor.lua:101-114](../../../raw/current/scripts/components/locomotor.lua#L101-L114)

`player_classified` 定义三个 `psm_*` net_float，客户端复制时向已有组件 `AttachClassified`，`playerspeedmultdirty` 再触发重算。组件本身不做逐帧轮询；只改变其他 locomotor 倍率或 cap 时，本组件不会因此自动收到 dirty 通知。[player_classified.lua:247-262](../../../raw/current/scripts/prefabs/player_classified.lua#L247-L262)、[player_classified.lua:673-677](../../../raw/current/scripts/prefabs/player_classified.lua#L673-L677)、[player_classified.lua:1225](../../../raw/current/scripts/prefabs/player_classified.lua#L1225)、[player_classified.lua:1626-1631](../../../raw/current/scripts/prefabs/player_classified.lua#L1626-L1631)、[playerspeedmult.lua:16-28](../../../raw/current/scripts/components/playerspeedmult.lua#L16-L28)

撤销效果应配对调用相应 `Remove*`，不建议删除组件。`OnRemoveFromEntity` 写有依次 Reset 来源表、取消初始化任务的步骤，但标准移除路径存在下述 dirty 回调缺口，不能保证这些步骤完成。组件没有 `OnSave`／`OnLoad`，来源、cap 和基速恢复需依赖角色／玩法重建，不保存加成剩余时间。[playerspeedmult.lua:50-63](../../../raw/current/scripts/components/playerspeedmult.lua#L50-L63)、[playerspeedmult.lua:1-219](../../../raw/current/scripts/components/playerspeedmult.lua#L1-L219)

## 典型接入

### 沃姆伍德开花：角色加速不叠坐骑基速

`wormwood.lua` 的 `SetStatsLevel(inst, level)` 把 0–3 的开花等级映射成 1–1.2 倍，以稳定字符串 `wormwood_bloom_level` 调 `SetSpeedMult`；同函数还按该倍率改饥饿消耗。该参照说明此处选择玩家基速层的理由，不是给所有移动来源增加同一倍率。回到倍率1会由底层列表移除这一项；临时自定义项仍应使用自己唯一的来源标识并成对撤销。[wormwood.lua:410-415](../../../raw/current/scripts/prefabs/wormwood.lua#L410-L415)、[sourcemodifierlist.lua:64-75](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64-L75)

## 常见陷阱与限制

- **删组件不是可靠撤销路径**：`EntityScript:RemoveComponent` 先清空 `self.components[name]` 再调用移除钩子；钩子的首个 Reset 即使来源表为空也会触发已配置的 dirty 回调，而 `OnDirty` 直接从已清空的 `inst.components.playerspeedmult` 调方法。因此标准路径可在首个 Reset 内发生 nil 访问，后续来源清理和初始化任务取消不能视为已完成。这是源码控制流推导，未游戏复现；正常撤效果仍使用配对 `Remove*`。[entityscript.lua:670-685](../../../raw/current/scripts/entityscript.lua#L670-L685)、[playerspeedmult.lua:50-63](../../../raw/current/scripts/components/playerspeedmult.lua#L50-L63)、[sourcemodifierlist.lua:124-135](../../../raw/current/scripts/util/sourcemodifierlist.lua#L124-L135)、[playerspeedmult.lua:12-14](../../../raw/current/scripts/components/playerspeedmult.lua#L12-L14)
- **实体来源自动移除不等于发布预测数据**：SourceModifierList 的 `onremove` 只重算列表并触发本地 Apply；`psm_servermult`／`psm_cappedservermult` 的写入在组件显式 Set／Remove 方法内。依赖自动删除时，不能据本地跑速恢复就宣称这些预测 netvar 已更新；需要同步撤销的效果应显式配对移除。此为源码路径核验，未实测网络表现。[sourcemodifierlist.lua:84-94](../../../raw/current/scripts/util/sourcemodifierlist.lua#L84-L94)、[playerspeedmult.lua:127-159](../../../raw/current/scripts/components/playerspeedmult.lua#L127-L159)
- **运行中直接覆盖基速或单边预测**：已有来源时基速不重新缓存；预测组又没有自动复制来源。改旧 `runspeed` 或只加客户端预测项可能与下一次重算／服务端状态冲突；沿角色已有接线添加独立来源，不把当前结算值当原始基速。[playerspeedmult.lua:116-175](../../../raw/current/scripts/components/playerspeedmult.lua#L116-L175)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、SourceModifierList 的积／替换／移除／Reset、EntityScript 移除顺序与 dirty 回调缺口、玩家两端安装与 classified 接线、locomotor 骑乘基速／预测选择及总倍率、沃姆伍德等级 setter。普通 locomotor 来源选型复用既有页面。
- 游戏运行：未运行；未实测网络时序、源实体删除、运行中 cap／装备变化、骑乘切换或组件移除；未审阅所有角色和 SG 预测状态的成对接线。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；无新增 Lua 示例，未运行语法探针。

相关：[[reference/components/locomotor]]、[[reference/components/equippable]]。
