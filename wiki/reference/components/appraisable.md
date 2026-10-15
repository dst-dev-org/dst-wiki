---
title: "appraisable"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "让物品通过 APPRAISE 动作委托目标进行评价，区分动作展示、可用性检查与实际评价。"
aliases: ["物品评价", "牛玩偶鉴评", "APPRAISE", "appraisable"]
tags: [component, callback, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/appraisable.lua", "componentactions.lua", "actions.lua", "prefabs/yotb_beefalo_dolls.lua", "prefabs/yotb_stage.lua", "components/yotb_stager.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "stategraphs/SGyotb_stage.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# appraisable

## 快速判断

- 职责与适用：加在被评价的物品上，以两个回调委托检查和执行；评价目标通过参数传入，不是把组件加到评价者上。[组件实现](../../../raw/current/scripts/components/appraisable.lua#L1-L19)
- 优先使用：沿用物品对目标的 `USEITEM → APPRAISE` 流程；动作收集只检查目标的 `appraiser` 标签，执行时才调用 `CanAppraise`。[动作收集](../../../raw/current/scripts/componentactions.lua#L1038-L1044) [动作执行](../../../raw/current/scripts/actions.lua#L774-L785)
- 重要副作用：组件不自行移除、分堆或转交物品；`Appraise` 忽略执行回调返回值，动作在检查通过后直接返回成功。[委托方法](../../../raw/current/scripts/components/appraisable.lua#L5-L19) [成功分支](../../../raw/current/scripts/actions.lua#L775-L782)
- 修改边界：原版牛玩偶仅在主模拟端安装组件和回调；目标标签只是展示门槛，实际所需组件、状态图及业务数据须由接入双方保证。[牛玩偶安装](../../../raw/current/scripts/prefabs/yotb_beefalo_dolls.lua#L54-L68) [目标要求](../../../raw/current/scripts/prefabs/yotb_beefalo_dolls.lua#L20-L32)

## 关键接口

| 接口／字段 | 签名或返回 | 条件、作用与证据 |
|---|---|---|
| `canappraisefn` | `(inst, target) → 可用值, 原因?` | 直接赋值；组件原样返回其结果，不把失败原因压成布尔值。[CanAppraise](../../../raw/current/scripts/components/appraisable.lua#L5-L13) |
| `CanAppraise(target)` | 无回调时返回 `true` | 只作检查；原版动作仅对失败原因 `"NOTNOW"` 显式返回 `false, "NOTNOW"`，其他失败值没有相同的原因转发分支。[动作执行](../../../raw/current/scripts/actions.lua#L774-L785) |
| `appraisefn` | `(inst, target)` | 直接赋值；实例是物品，不传 `doer`。无回调则不执行业务。[Appraise](../../../raw/current/scripts/components/appraisable.lua#L15-L19) |
| `Appraise(target)` | 无显式返回 | 不自动调用 `CanAppraise`，直接调用者须自行检查；回调的返回值不参与动作成功判断。[组件方法](../../../raw/current/scripts/components/appraisable.lua#L5-L19) [动作调用顺序](../../../raw/current/scripts/actions.lua#L775-L778) |

## 典型接入

### 原版牛玩偶交给评委亭评价

这是原版业务接入参照，无需额外模组代码或资源：

1. `yotb_beefalo_dolls.lua` 的 `make` 工厂注册 `yotb_beefalo_doll_war` 等九种玩偶，主模拟端安装 `appraisable`，绑定检查／执行回调，并保存 `category`。[安装与类别](../../../raw/current/scripts/prefabs/yotb_beefalo_dolls.lua#L60-L88) [注册名](../../../raw/current/scripts/prefabs/yotb_beefalo_dolls.lua#L91-L99)
2. 目标是注册名 `yotb_stage` 的实体；它在公共构造中加 `appraiser` 标签，主模拟端安装 `yotb_stager` 和 `SGyotb_stage`。[标签与端门禁](../../../raw/current/scripts/prefabs/yotb_stage.lua#L81-L100) [目标组件与状态图](../../../raw/current/scripts/prefabs/yotb_stage.lua#L104-L123) [注册](../../../raw/current/scripts/prefabs/yotb_stage.lua#L230-L233)
3. 玩家端和服务端都将 `APPRAISE` 映射到 `give`；服务端在动画第 13 帧执行缓冲动作。动作此时检查目标状态，玩偶要求 `target.sg` 带 `ready` 状态标签，否则返回 `NOTNOW`。[双方动作处理](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L556-L559) [服务端映射](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1262-L1265) [执行时间](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12320-L12333) [检查回调](../../../raw/current/scripts/prefabs/yotb_beefalo_dolls.lua#L26-L32)
4. 执行回调调用目标的 `yotb_stager:appraisedoll(inst)`；接收方读取 `doll.category`，禁用锤击工作并启动评价队列，结束时恢复工作、比赛启动标签并移除语音实体。这条路径未消费玩偶。[转交](../../../raw/current/scripts/prefabs/yotb_beefalo_dolls.lua#L20-L24) [开始评价](../../../raw/current/scripts/components/yotb_stager.lua#L1229-L1261) [结束恢复](../../../raw/current/scripts/components/yotb_stager.lua#L1317-L1323)

自定义物品复用该组件时，在其服务端构造中配置两个回调；自定义评价目标还须提供双方约定的数据和处理逻辑。若沿用原版回调，仅加 `appraiser` 标签并不足够：检查回调直接访问 `target.sg`，接收方直接索引类别表。不要给任意物品填写未知 `category` 后送进原版评委亭。[原版约定](../../../raw/current/scripts/prefabs/yotb_beefalo_dolls.lua#L20-L32) [类别消费](../../../raw/current/scripts/components/yotb_stager.lua#L1229-L1235)

## 常见陷阱与限制

- **有动作就能立即评价**：展示阶段不调用 `CanAppraise`；目标离开 `ready` 后仍可能走到执行检查并失败。`ready` 是状态图标签，例如 `idle_closed_ready`，不是同名实体标签。[收集门槛](../../../raw/current/scripts/componentactions.lua#L1040-L1044) [状态定义](../../../raw/current/scripts/stategraphs/SGyotb_stage.lua#L69-L80)
- **执行回调返回 false 会中止动作**：返回值被丢弃；所有拒绝条件应在检查回调实现，或明确自行管理业务失败，不能指望 `APPRAISE` 自动识别。[组件执行](../../../raw/current/scripts/components/appraisable.lua#L15-L19) [动作成功](../../../raw/current/scripts/actions.lua#L775-L778)
- **直接 Appraise 等于完整动作**：它跳过检查，也不负责玩家动画、持有者或消耗。需要这些行为时保留动作入口，并在业务回调中明确实现额外规则。[两个独立方法](../../../raw/current/scripts/components/appraisable.lua#L5-L19)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：actions.lua（R-01）、componentactions.lua（R-04）、stategraphs/SGwilson.lua（SB-17）与 SGwilson_client.lua 对照 build-26148036 更新报告逐条核验，锚点重定位 5 处、结论修订 0 处：APPRAISE 收集（componentactions.lua:1038-1044）／执行与 NOTNOW 分支、双端 `give` 映射及第 13 帧执行（SGwilson.lua:12274-12287）均逐行无变化；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖两个委托方法、动作收集／执行、双方玩家动作映射、九种玩偶注册与目标安装、评价入口和结束恢复。未全量审校评委台词、评分数据或比赛生命周期。
- 结构检查：本页元数据、大小、链接与源码行号经定向校验；无新增 Lua 示例。
- 游戏运行：未运行；客户端展示、忙碌期间失败提示及评价时序待实测。
