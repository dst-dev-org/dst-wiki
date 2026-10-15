---
title: "prototyper"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "科技站向人物提供科技树、登记使用者及制作激活回调，区别配方注册和永久学习。"
aliases: ["prototyper","科技站","科学机器科技","多人使用科技站"]
tags: [component, recipe, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/prototyper.lua","components/builder.lua","prefabs/scienceprototyper.lua","techtree.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# prototyper

## 快速判断

- 职责与适用：挂在科技来源实体，提供 trees 与多人使用状态；人物 [[reference/components/builder]] 选择当前站点，不给产物加本组件。[prototyper.lua:3](../../../raw/current/scripts/components/prototyper.lua#L3)、[builder.lua:235](../../../raw/current/scripts/components/builder.lua#L235
- 优先使用：已有机器改实例 trees；新增机器共同端预加 prototyper 标签，服务端加组件与回调，沿原版工厂分界。[scienceprototyper.lua:196](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L196)、[scienceprototyper.lua:213](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L213
- 重要副作用：TurnOn 按 doer 去重并监听其 onremove；首人开启、末人离开才切整体 on；不等于制作成功。[prototyper.lua:36](../../../raw/current/scripts/components/prototyper.lua#L36)、[prototyper.lua:52](../../../raw/current/scripts/components/prototyper.lua#L52
- 修改边界：科技计算／使用者表在服务端，菜单经 builder 同步；改树不注册配方或永久学习，也不自动保存配置。[builder.lua:274](../../../raw/current/scripts/components/builder.lua#L274

## 关键接口

| 接口／字段 | 约定 | 条件、副作用与证据 |
|---|---|---|
| `trees`／`GetTechTrees()` | 科技等级表／深拷贝 | 构造用 TechTree.Create 补各科技 0；GetTechTrees 返回副本，但 builder 直接读 trees。[prototyper.lua:9](../../../raw/current/scripts/components/prototyper.lua#L9)、[prototyper.lua:76](../../../raw/current/scripts/components/prototyper.lua#L76)、[builder.lua:274](../../../raw/current/scripts/components/builder.lua#L274 |
| `TurnOn(doer)`／`TurnOff(doer)` | 有效使用者实体 | 维护 doers 与移除监听；直接调用不校验 restrictedtag 或距离。[prototyper.lua:36](../../../raw/current/scripts/components/prototyper.lua#L36)、[prototyper.lua:52](../../../raw/current/scripts/components/prototyper.lua#L52 |
| `onturnon/off(inst)` | 首人／末人或组件移除 | 回调在 on 字段切换前调用；不要在其中把 on 当已切换值。[prototyper.lua:43](../../../raw/current/scripts/components/prototyper.lua#L43)、[prototyper.lua:59](../../../raw/current/scripts/components/prototyper.lua#L59 |
| `onturnonfordoer/offfordoer(inst,doer)` | 每个使用者增删 | 整体仍开启时也可触发；移组件逐人清理，再清整体。[prototyper.lua:19](../../../raw/current/scripts/components/prototyper.lua#L19)、[prototyper.lua:40](../../../raw/current/scripts/components/prototyper.lua#L40 |
| `restrictedtag` | nil 或人物标签 | builder 选择站点时筛，组件 TurnOn 本身不验。[builder.lua:268](../../../raw/current/scripts/components/builder.lua#L268 |
| `CanUsePrototyper(doer)` | bool；可 overridecanuseprototyper(inst,doer) | 默认 IsNear(RESEARCH_MACHINE_DIST)；已核验用于显式 override 站点，普通扫描另按搜索半径／标签选择，不能当所有路径统一门禁。[prototyper.lua:68](../../../raw/current/scripts/components/prototyper.lua#L68)、[builder.lua:240](../../../raw/current/scripts/components/builder.lua#L240 |
| `Activate(doer,recipe)` | recipe 对象，无显式返回 | 先让 craftingstation:RecipeCrafted 处理限额，再 onactivate(inst,doer,recipe)；不是通用扣料／生成接口。[prototyper.lua:80](../../../raw/current/scripts/components/prototyper.lua#L80 |

builder 只选一个当前站点，其他候选 TurnOff，不将附近多台机器的科技树简单叠加。具名工作站配方由 [[reference/components/craftingstation]] 提供；永久学习／材料／制造站特例仍由 builder 处理。[builder.lua:265](../../../raw/current/scripts/components/builder.lua#L265)、[builder.lua:670](../../../raw/current/scripts/components/builder.lua#L670

## 典型接入

### 科学机器提供二级科学但保留原有动画与激活

目标模组 `modmain.lua`；改原版 researchlab 实例，复用资源和回调。工厂将共享 TUNING 表直接赋给 trees，修改前必须复制。[scienceprototyper.lua:216](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L216)、[scienceprototyper.lua:299](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L299

```lua
AddPrefabPostInit("researchlab", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local prototyper = inst.components.prototyper
    if prototyper ~= nil then
        prototyper.trees = GLOBAL.deepcopy(prototyper.trees)
        prototyper.trees.SCIENCE = 2
    end
end)
```

只改 SCIENCE 等级，不新增科技类型、不改变材料／角色门禁、配方注册或动画等级。运行中菜单等待 builder 重算；不承诺即时刷新。配置不由本组件存档，停用后工厂恢复，但玩家此前永久学会的配方不会自动遗忘。[builder.lua:235](../../../raw/current/scripts/components/builder.lua#L235)、[[reference/components/builder#关键接口]]

## 常见陷阱与限制

- **直接改 TUNING 共享树**：可能影响其他实例；复制实例表后改，避免通过当前机器改动共享配置。[scienceprototyper.lua:216](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L216
- **每个玩家离开都关整台机器**：onturnofffordoer 与 onturnoff 职责不同，末人才关整体；移组件也有整体清理。[prototyper.lua:52](../../../raw/current/scripts/components/prototyper.lua#L52)、[prototyper.lua:19](../../../raw/current/scripts/components/prototyper.lua#L19
- **Activate 自带资格和距离检查**：本方法只分发限额／回调，应走 builder 制作路径；改 CanUsePrototyper 也不代表普通扫描一定调用它。[prototyper.lua:80](../../../raw/current/scripts/components/prototyper.lua#L80)、[builder.lua:255](../../../raw/current/scripts/components/builder.lua#L255
- **新科技只填 trees 的一个键**：TechTree.Create 只补已有科技键，不完成注册／网络扩展；本页不提供新增科技系统。[techtree.lua:73](../../../raw/current/scripts/techtree.lua#L73

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整组件、builder 的站点选择／树消费／激活与制造站分支、科学机器注册／工厂、TechTree.Create。未全量审校配方菜单、科技网络协议或所有科技站。
- 结构检查：页面格式、来源、链接／源码锚点及登记；完整示例经 Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；多人切换、菜单刷新、烧毁／读档、皮肤／赠礼及跨模组行为待测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/craftingstation]]；[[reference/components/builder]]；[[reference/components/placer]]。
