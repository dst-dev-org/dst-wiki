---
title: "sanityauraadjuster"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "理解温蒂技能的周期精神光环调整委托，区分任务停止与已施加来源倍率的撤销。"
aliases: ["精神光环调整", "温蒂负光环减免", "sanityauraadjuster"]
tags: [component, callback, lifecycle, aura, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/sanityauraadjuster.lua", "prefabs/wendy.lua", "prefabs/skilltree_wendy.lua", "prefabs/player_common.lua", "components/sanity.lua", "util/sourcemodifierlist.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sanityauraadjuster

## 快速判断

- 职责与适用：仅保存 `adjustmentfn`、上次结果 `players` 并每秒委托回调；本组件不搜索人物、不计算光环，也不直接改精神值。[sanityauraadjuster.lua:1-28](../../../raw/current/scripts/components/sanityauraadjuster.lua#L1-L28)
- 优先使用：理解／扩展温蒂 `wendy_sisturn_2` 的轮询协作；向附近人物提供精神速率用 [[reference/components/sanityaura]]，人物来源列表用 [[reference/components/sanity]]。
- 重要副作用：回调返回值直接覆盖 `players`；停止任务保留列表与已施加效果，没有自动撤销。[sanityauraadjuster.lua:7-24](../../../raw/current/scripts/components/sanityauraadjuster.lua#L7-L24)
- 修改边界：组件自身无主机检查或 netvar；原版在温蒂 `master_postinit` 添加并设置回调，属于服务端逻辑，不应据此在客户端调用精神组件。[wendy.lua:358-374](../../../raw/current/scripts/prefabs/wendy.lua#L358-L374)

## 关键接口

| 接口／字段 | 签名或已核验约定 | 条件、作用与证据 |
|---|---|---|
| `SetAdjustmentFn(fn)` | `fn(inst, oldplayers) -> newplayers`；原版返回人物数组 | 单一回调槽直接替换；传 nil 使后续 tick 无操作，不清已有结果。[sanityauraadjuster.lua:14-28](../../../raw/current/scripts/components/sanityauraadjuster.lua#L14-L28) |
| `StartTask()` | 无参数、无返回值 | 在 `inst.sanityAuraAdjuster_task` 无任务时建立 1 秒周期任务；重复调用不重复建任务，不立即手动调用回调。[sanityauraadjuster.lua:14-24](../../../raw/current/scripts/components/sanityauraadjuster.lua#L14-L24) |
| `StopTask()` | 无参数、无返回值 | Cancel 后清任务字段；不调用回调、不清 players、不删除来源。[sanityauraadjuster.lua:7-12](../../../raw/current/scripts/components/sanityauraadjuster.lua#L7-L12) |
| `players` | 初始 `{}`，此后为回调原样返回值 | 无类型／nil 校验；温蒂回调遍历旧数组，替换回调必须保持此契约或同步管理状态。[sanityauraadjuster.lua:1-5](../../../raw/current/scripts/components/sanityauraadjuster.lua#L1-L5)、[wendy.lua:101-150](../../../raw/current/scripts/prefabs/wendy.lua#L101-L150) |

## 典型接入

### 沿用温蒂技能的负精神光环减免链

这是原版专用协作说明，不提供强制启动技能的 modmain。`wendy` 通过 `MakePlayerCharacter` 注册；工厂调用 `master_postinit`，后者添加本组件并设置 `testForSanityAuraBuff`，技能 `wendy_sisturn_2` 的激活／停用分别 StartTask／StopTask。[wendy.lua:425](../../../raw/current/scripts/prefabs/wendy.lua#L425)、[player_common.lua:2924-2926](../../../raw/current/scripts/prefabs/player_common.lua#L2924-L2926)、[player_common.lua:2971](../../../raw/current/scripts/prefabs/player_common.lua#L2971)、[skilltree_wendy.lua:66-74](../../../raw/current/scripts/prefabs/skilltree_wendy.lua#L66-L74)

回调只有在世界 `sisturnregistry:IsActive()` 时收集温蒂周围 25 单位内人物；否则新列表为空。新增人物获得特效、音效和 `sanity.neg_aura_modifiers:SetModifier(inst, TUNING.WENDYSKILL_SISTURN_SANITY_MODIFYER, "wendyskill"..inst.GUID)`；已不在新列表中的旧人物以相同 source/key RemoveModifier。范围是温蒂周围，并非直接围绕姐妹骨灰坛。[wendy.lua:101-150](../../../raw/current/scripts/prefabs/wendy.lua#L101-L150)

这个值是负光环倍率来源，不是每秒精神增量。`sanity:GetAuraMultipliers()` 将来源列表与人物自身 `neg_aura_mult` 相乘，普通光环汇总仅在 aura<0 时采用；正光环、照明等精神贡献不由本委托一并调整。[sanity.lua:484-485](../../../raw/current/scripts/components/sanity.lua#L484-L485)、[sanity.lua:568-570](../../../raw/current/scripts/components/sanity.lua#L568-L570)；完整接收机制复用 [[reference/components/sanityaura#附近光环装备与随身效果如何选择]]。

扩展时应保留原回调调用、原返回数组和技能启停入口；新增来源用自有 key 并独立成对撤销。不要将 `SetAdjustmentFn` 当多来源注册器，直接覆盖会丢失温蒂的新增／移出处理。

### 停用、删除与读档的区别

- 离开范围／骨灰坛注册状态不活跃：任务仍运行时，下一次原回调比较旧新列表并撤销对应 key。[wendy.lua:101-150](../../../raw/current/scripts/prefabs/wendy.lua#L101-L150)
- 单纯技能停用：已核验入口只 StopTask；最后一轮已施加的倍率不会由组件立即撤销。若自定义需求要求即时结束，需先显式遍历旧列表 RemoveModifier，再清自有状态，不能只停止任务。[skilltree_wendy.lua:69-74](../../../raw/current/scripts/prefabs/skilltree_wendy.lua#L69-L74)
- 温蒂实体删除：`SourceModifierList` 对实体 source 监听 `onremove`，自动移除该来源；实体清理同时取消宿主 pending tasks。移除组件本身不是删除 source，本组件也没有 `OnRemoveFromEntity` 清理。[sourcemodifierlist.lua:64-99](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64-L99)、[entityscript.lua:1743-1751](../../../raw/current/scripts/entityscript.lua#L1743-L1751)
- 组件全文没有 OnSave／OnLoad；回调、players 与任务不是本组件存档数据。重建靠人物工厂和技能恢复入口（技能回调接收 fromload 但不分支）；不将此处审阅扩展为整个技能树存档流程已验证。[sanityauraadjuster.lua:1-30](../../../raw/current/scripts/components/sanityauraadjuster.lua#L1-L30)、[skilltree_wendy.lua:69-74](../../../raw/current/scripts/prefabs/skilltree_wendy.lua#L69-L74)

## 常见陷阱与限制

- **停止轮询等于取消 buff**：StopTask 不做列表差分，最后一批人物可能继续保留来源；即时结束应显式撤销 source/key，别等待一个已经停止的下一轮。
- **返回 nil 表示没人**：组件原样保存返回值；原回调下一轮需要可遍历数组。无人时返回 `{}`，额外状态自行清理。[wendy.lua:101-150](../../../raw/current/scripts/prefabs/wendy.lua#L101-L150)
- **RemoveComponent 就能安全停任务**：任务存在宿主上且没有移组件清理钩子；应先 StopTask 并清效果，不能靠组件卸载代替生命周期处理。[sanityauraadjuster.lua:1-30](../../../raw/current/scripts/components/sanityauraadjuster.lua#L1-L30)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 3 处（player_common 删 14 行、entityscript 移 3 行，区间文本逐段比对一致）；结论修订：无，仅锚点迁移。
- 依据：全局枢纽 hunk 地图（R-08）＋变更清单命中 entityscript.lua、prefabs/player_common.lua。

- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖组件全文、温蒂注册／服务端安装、技能启停、实际数组差分／负倍率消费者、实体来源撤销及任务清理；复用同 build 的 sanity／sanityaura 页。未审校整个技能树存档与所有骨灰坛注册变化。
- 结构检查：页面格式、链接、大小及源码行锚点检查通过；本页无完整 Lua 示例。
- 游戏运行：未运行；技能停用后残留时长、读档重新轮询、多人叠加与移出特效待实测。
