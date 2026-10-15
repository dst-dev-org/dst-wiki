---
title: "papereraser"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "让工作台承接擦除纸张动作，区分擦除目标、被消耗纸张与产物配置的职责。"
aliases: ["擦除纸张工具", "制图桌回收纸张", "papereraser"]
tags: [component, building, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/papereraser.lua", "components/erasablepaper.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "prefabs/cartography_prototyper.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# papereraser

## 快速判断

- 职责与适用：挂在接受纸张的目标上，原版为制图桌 `cartographydesk`；被擦除的物品挂 [[reference/components/erasablepaper]]，两者方向不要反转。[目标挂载](../../../raw/current/scripts/prefabs/cartography_prototyper.lua#L100-L106) [动作执行](../../../raw/current/scripts/actions.lua#L3931-L3939)
- 优先使用：需要回收地图、蓝图等已有纸张时，沿用 `ERASE_PAPER`；产物与数量在纸张侧配置，本组件只有转发入口。[DoErase](../../../raw/current/scripts/components/papereraser.lua#L14-L16)
- 重要副作用：构造添加 `papereraser` 标签，移除组件删除标签；擦除时不消耗工作台，也不包含耐久、动画或回调配置。[标签生命周期](../../../raw/current/scripts/components/papereraser.lua#L1-L16)
- 修改边界：原版客户端提前拥有目标标签，服务端才挂组件；动作还拒绝带 `fire` 或 `burnt` 标签的目标。直接调用组件不会重复这些门禁。[工厂端边界](../../../raw/current/scripts/prefabs/cartography_prototyper.lua#L90-L106) [动作门禁](../../../raw/current/scripts/actions.lua#L3931-L3939)

## 关键接口

| 接口／入口 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `DoErase(paper, doer)` | boolean | 直接调用 `paper.components.erasablepaper:DoErase(tool, doer)`，以结果 `~= nil` 判成功；不检查参数有效性或组件存在。[papereraser.lua:14-16](../../../raw/current/scripts/components/papereraser.lua#L14-L16) |
| `OnRemoveFromEntity()` | 无返回 | 仅删除目标的 `papereraser` 标签，无任务、监听或存档状态要清理。[移除](../../../raw/current/scripts/components/papereraser.lua#L10-L12) |
| `ACTIONS.ERASE_PAPER` | `invobject`＝纸张，`target`＝擦除工具 | 动作要求双方组件存在且目标未带 fire／burnt；返回工具转发结果。[actions.lua:3931-3939](../../../raw/current/scripts/actions.lua#L3931-L3939) |
| 纸张的动作发现 | 物品使用在目标上 | `erasablepaper` 的动作采集根据目标标签提供动作；双端玩家 SG 都选 `dolongaction`。[动作采集](../../../raw/current/scripts/componentactions.lua#L1221-L1228) [服务端](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1283) [客户端](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L583) |

## 典型接入

### 给新工作台沿用制图桌的纸张回收入口

本节是原版组装路径，不是完整新建筑工程。制图桌注册名并非文件名：`cartography_prototyper.lua` 的 `fn` 实际注册为 `cartographydesk`。[注册](../../../raw/current/scripts/prefabs/cartography_prototyper.lua#L134-L135)

1. 新建筑的共同构造阶段，在 `SetPristine()` 前加 `papereraser` 标签，保证客户端动作发现可见；原版同时构造网络、物理与动画。[构造](../../../raw/current/scripts/prefabs/cartography_prototyper.lua#L71-L98)
2. `TheWorld.ismastersim` 门禁之后挂 `papereraser`；它不要求 `prototyper`，原版两组件并列挂载，科研树不参与擦除动作。[服务端组装](../../../raw/current/scripts/prefabs/cartography_prototyper.lua#L100-L111) [实际调用](../../../raw/current/scripts/actions.lua#L3931-L3939)
3. 输入物品需有 `erasablepaper`；产物交付、单次消耗与失败边界统一查 [[reference/components/erasablepaper#典型接入]]，不在工作台重复实现。

正常擦除由玩家长动作执行，工具位置作为产物交付／掉落参考；若把工作台做成可燃建筑，沿用原版火焰／烧毁标签门禁。原版建筑还包含科研、锤毁等独立机制，无需为了擦纸复制全部组件。[动作门禁](../../../raw/current/scripts/actions.lua#L3931-L3939) [产物位置](../../../raw/current/scripts/components/erasablepaper.lua#L1-L9) [建筑其他机制](../../../raw/current/scripts/prefabs/cartography_prototyper.lua#L108-L129)

## 常见陷阱与限制

- **给工具配置产物无效：** 源码中的 `self.stacksize`、`self.erased_prefab` 只是注释，不是有效配置；实际字段与 setter 属于纸张的 `erasablepaper`。[注释与转发](../../../raw/current/scripts/components/papereraser.lua#L1-L16) [实际配置](../../../raw/current/scripts/components/erasablepaper.lua#L12-L29)
- **只有服务端组件，客户端没有目标标签：** 原版显式在 pristine 前添加标签；新网络 Prefab 应保留此共同构造步骤，不只依赖服务端组件构造。[原版顺序](../../../raw/current/scripts/prefabs/cartography_prototyper.lua#L90-L106)
- **把 false 当作失败返回：** 转发条件为 `~= nil`；若自行替换纸张方法并返回 false，工具反而报告 true。与内置纸张接口一致地用 nil 表示失败、产物实体表示成功。[工具判定](../../../raw/current/scripts/components/papereraser.lua#L14-L16) [原版失败／成功](../../../raw/current/scripts/components/erasablepaper.lua#L31-L65)
- **脚本直接调用绕过门禁：** `DoErase` 不检查纸张组件、火焰状态或 doer；调用者自己核验条件。需要玩家交互时走已有动作，而不是把裸方法当作完整资格判断。[组件](../../../raw/current/scripts/components/papereraser.lua#L14-L16) [动作](../../../raw/current/scripts/actions.lua#L3931-L3939)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 变更文件（actions.lua、componentactions.lua、stategraphs/SGwilson.lua、stategraphs/SGwilson_client.lua，更新报告 R-01/R-04）逐条核验，ERASE_PAPER 定义／执行与 erasablepaper 动作采集均无语义变化，锚点重定位 8 处、无结论修订；未覆盖：运行验证。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；覆盖完整组件、纸张侧实现、动作发现／执行、玩家双端 SG、制图桌工厂注册与标签／组件端边界。组装说明不包含建筑科研、燃烧和锤毁的完整审校。
- 游戏运行：未运行；新增建筑与自定义纸张组合未实测。
- 结构检查：页面格式／链接与源码锚点严格检查通过；本页为原版组装说明，无新增完整 Lua 示例。
