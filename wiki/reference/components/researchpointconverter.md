---
title: researchpointconverter：已弃用的研究回调组件
created: 2026-10-27
updated: "2026-10-28"
type: component
summary: 识别明确弃用的 researchpointconverter，并区分旧回调签名与现用 prototyper 接入。
aliases: [researchpointconverter, 研究点转换器, 旧研究组件]
tags: [component, callback, recipe]
dst_build: "steam-buildid:26230496"
sources: [components/researchpointconverter.lua, components/prototyper.lua, components/builder.lua, prefabs/scienceprototyper.lua]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# researchpointconverter

> 源文件首行明确要求不要使用，改用 `prototyper.lua`。本页保留旧实现辨识与迁移边界，新科技站优先查 [[reference/components/prototyper]]。[researchpointconverter.lua:1](../../../raw/current/scripts/components/researchpointconverter.lua#L1

## 快速判断

- 职责与适用：旧组件仅初始化字段并提供开启／关闭／激活回调包装；没有实际研究点转换或科技树计算。[researchpointconverter.lua:3](../../../raw/current/scripts/components/researchpointconverter.lua#L3
- 优先使用：原版研究站使用 `prototyper`；科学机器工厂服务端添加该组件、设置科技树和回调。[scienceprototyper.lua:213](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L213
- 重要副作用：旧 `TurnOn`／`TurnOff` 只有相应回调存在时才改变 `on`，且在回调之后改变；未设置回调时不会切状态。[researchpointconverter.lua:11](../../../raw/current/scripts/components/researchpointconverter.lua#L11
- 修改边界：未找到本组件的原版 Lua 消费者，不提供新接入教程；弃用说明不代表把名称替换为 prototyper 就能保持参数及行为兼容。[researchpointconverter.lua:1](../../../raw/current/scripts/components/researchpointconverter.lua#L1)、[prototyper.lua:36](../../../raw/current/scripts/components/prototyper.lua#L36

## 关键接口

| 旧接口／字段 | 实际行为 | 证据 |
|---|---|---|
| `val=0`、`active=false`、`level=1`、`on=false` | 构造赋值；本文件没有读取前三项，也没有转换／累计研究点的逻辑。 | [researchpointconverter.lua:3](../../../raw/current/scripts/components/researchpointconverter.lua#L3 |
| `TurnOn()` | `not self.on` 且 `onturnon` 存在才调用 `onturnon(self.inst)`，随后令 `on=true`。 | [researchpointconverter.lua:11](../../../raw/current/scripts/components/researchpointconverter.lua#L11 |
| `TurnOff()` | `self.on` 且 `onturnoff` 存在才调用 `onturnoff(self.inst)`，随后令 `on=false`。 | [researchpointconverter.lua:18](../../../raw/current/scripts/components/researchpointconverter.lua#L18 |
| `Activate()` | 仅在存在 `onactivate` 时无参数调用；没有检查 `on`／`active`，没有消耗、科技解锁或结果返回。 | [researchpointconverter.lua:26](../../../raw/current/scripts/components/researchpointconverter.lua#L26 |

## 常见陷阱与限制

- **按旧签名迁移**：现用 `prototyper:TurnOn(doer)`／`TurnOff(doer)` 维护使用者集合，首人开启、末人离开才切整体状态，还监听使用者移除；需要有效 doer，不能沿用旧组件无参数调用习惯。[prototyper.lua:36](../../../raw/current/scripts/components/prototyper.lua#L36)、[prototyper.lua:52](../../../raw/current/scripts/components/prototyper.lua#L52
- **照搬激活回调**：旧版 `onactivate()` 无参数，现用 `prototyper:Activate(doer, recipe)` 会传 `onactivate(inst,doer,recipe)`，并可先通知 craftingstation。builder 的实际入口 `ActivateCurrentResearchMachine(recipe)` 将玩家和配方传给当前站点。[researchpointconverter.lua:26](../../../raw/current/scripts/components/researchpointconverter.lua#L26)、[prototyper.lua:80](../../../raw/current/scripts/components/prototyper.lua#L80)、[builder.lua:68](../../../raw/current/scripts/components/builder.lua#L68
- **把旧字段当现用科技配置**：当前 builder 选站并读取的是 `components.prototyper.trees`；没有在该路径中消费旧组件的 `val`／`level`。迁移应沿现用树与站点规则核验，而非假定旧数值等价。[builder.lua:265](../../../raw/current/scripts/components/builder.lua#L265

## 验证记录

- 2026-10-27，`steam-buildid:26148036`：审阅 researchpointconverter、prototyper 全文；定向核验 builder 激活及选站消费路径、scienceprototyper 工厂添加与实际注册 `researchlab`／`researchlab2`。[scienceprototyper.lua:299](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L299
- 对所选 `scripts` 全部 Lua 文件作不区分大小写的 `researchpointconverter` 搜索，命中仅组件自身；未找到显式原版消费者，未排除动态构造组件名或引擎侧使用。替代关系以弃用注释和现用调用链为界，不声称覆盖全部科技机制。
- 未运行游戏、未执行旧模组迁移；无完整 Lua 示例，不涉及语法探针。`validate_page.py` 与 `validate_source_links.py --strict` 检查通过，范围为页面格式、预算与源码链接。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。