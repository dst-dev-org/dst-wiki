---
title: "constructionplans"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "把计划物品指向的已有实体原位替换成建造站点，材料阶段随后交给 constructionsite。"
aliases: ["建造计划", "计划物品改造站点", "计划书替换实体", "constructionplans"]
tags: [component, item, building, recipe]
dst_build: "steam-buildid:26230496"
sources: ["components/constructionplans.lua", "prefabs/constructionplans.lua", "componentactions.lua", "actions.lua", "recipes.lua", "prefabs/multiplayer_portal.lua", "components/constructionbuilder.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# constructionplans

## 快速判断

- 职责与适用：挂在“计划物品”上的服务端组件，保存“目标 prefab → 站点 prefab”映射；玩家用该物品对目标实体执行 CONSTRUCT 时，生成站点、删除原实体，把后续建造交给站点与建造者。[组件](../../../raw/current/scripts/components/constructionplans.lua#L6-L11) [入口](../../../raw/current/scripts/actions.lua#L4443-L4450)
- 优先使用：把“已存在的实体”改造成可分阶段投料的站点（原版只有月亮门改造）；空地放置走 [[reference/components/builder]]，站点材料与进度走 [[reference/components/constructionsite]]，都不归本组件。
- 重要副作用：原实体在转换时被 `Remove()`，不掉落、不可撤销；计划物品在动作成功后由 `CONSTRUCT` 删除。站点材料清单由站点按 `CONSTRUCTION_PLANS` 读取，不来自本组件。[替换](../../../raw/current/scripts/components/constructionplans.lua#L32-L35)
- 修改边界：组件只在服务端；客户端靠物品 prefab 在 pristine 阶段写的 `<目标prefab>_plans` tag 显示动作。组件不校验距离，也不读配方。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddTargetPrefab(prefab, constructionprefab)` | 两个 prefab 名 | 登记映射；首次登记时给物品 `AddTag(prefab.."_plans")`，重复登记只更新映射。[L6-L11](../../../raw/current/scripts/components/constructionplans.lua#L6-L11) |
| `RemoveTargetPrefab(prefab)` | prefab 名 | 删除映射并 `RemoveTag`；不改客户端已复制的 tag。[L13-L18](../../../raw/current/scripts/components/constructionplans.lua#L13-L18) |
| `StartConstruction(target)` | 返回站点实体，或 `nil[, reason]` | `target` 为 nil 或已有 `constructionsite` 时返回 nil；映射缺失返回 `nil, "MISMATCH"`；`SpawnPrefab` 失败返回 nil；成功则站点定位到目标位置、`target:Remove()`、向站点推 `onstartconstruction`。[L20-L36](../../../raw/current/scripts/components/constructionplans.lua#L20-L36) |
| `targetprefabs` | 表 | 键是“被改造实体”的 prefab（如 `multiplayer_portal`），值是站点 prefab（如 `multiplayer_portal_moonrock_constr`）；不是站点材料表的键。[L3](../../../raw/current/scripts/components/constructionplans.lua#L3) [L24-L28](../../../raw/current/scripts/components/constructionplans.lua#L24-L28) |
| `OnRemoveFromEntity()` | 无参数 | 意图是遍历映射删 tag，但函数体写的是未定义全局 `targetprefabs` 而非 `self.targetprefabs`。[L38-L42](../../../raw/current/scripts/components/constructionplans.lua#L38-L42) |

### 谁写、谁读

- 写：服务端 prefab 在 `AddComponent` 后逐项 `AddTargetPrefab`；同一批 tag 还在 pristine 段（两端都执行）独立写一遍；客户端动作判断依赖这一份，服务端运行期再加的 tag 不会复制（源码推断，未运行）。[prefab 接线](../../../raw/current/scripts/prefabs/constructionplans.lua#L29-L32) [组件接线](../../../raw/current/scripts/prefabs/constructionplans.lua#L52-L55)
- 读：客户端 `componentactions.constructionplans` 读物品 tag 与目标 prefab，决定是否插入 `ACTIONS.CONSTRUCT`；服务端 `ACTIONS.CONSTRUCT.fn` 读物品组件执行转换。[收集器](../../../raw/current/scripts/componentactions.lua#L1107-L1111) [服务端分支](../../../raw/current/scripts/actions.lua#L4441-L4451)
- 不读：本组件不读 `CONSTRUCTION_PLANS`、不碰 constructionsite 的材料与 builder；改造完成后的所有材料记账归 [[reference/components/constructionsite]] 与 [[reference/components/constructionbuilder]]。

## 典型接入

### 原版参照：月亮门计划把石质传送门改成施工站点

- 目标文件与加载：`prefabs/constructionplans.lua` 的 `MakePlans` 工厂生成注册名 `multiplayer_portal_moonrock_constr_plans`；`prefabs` 依赖带上站点 prefab，保证 `SpawnPrefab` 可解析，站点本身由 portal 文件注册。[工厂](../../../raw/current/scripts/prefabs/constructionplans.lua#L6-L11) [注册名](../../../raw/current/scripts/prefabs/constructionplans.lua#L64-L73) [站点注册](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L456-L458)
- 两端与资源：物品在 pristine 段加 `multiplayer_portal_plans` tag 与 `donotautopick`，并设 `constructionname` 供动作文案；服务端才加 `constructionplans`、可燃烧/传播/闹鬼。动画用 `anim/construction_plans.zip`，播放名为站点 prefab 名。[pristine](../../../raw/current/scripts/prefabs/constructionplans.lua#L29-L35) [物品部件](../../../raw/current/scripts/prefabs/constructionplans.lua#L50-L59)
- 目标与流程：玩家手持计划对已放置的 `multiplayer_portal` 执行 CONSTRUCT；服务端确认玩家可开始建造、目标没有 constructionsite 后调 `StartConstruction`，原位生成 `multiplayer_portal_moonrock_constr` 并删除石门，站点收到 `onstartconstruction` 后进入 `placeconstruction`。[动作分支](../../../raw/current/scripts/actions.lua#L4441-L4451) [站点监听](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L215-L217) [站点接线](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L274-L286)
- 单位：CONSTRUCT 动作距离 2.5；计划物品由 `boards 1 + rope 1` 在 `TECH.CELESTIAL_ONE` 制作；站点材料是 `purplemooneye 1 + moonrocknugget 20`，由站点按 `CONSTRUCTION_PLANS` 记账，不在物品上。[动作](../../../raw/current/scripts/actions.lua#L481) [计划配方](../../../raw/current/scripts/recipes.lua#L833) [材料表](../../../raw/current/scripts/recipes.lua#L1520-L1522)
- 生效与撤销：计划物品在转换成功后 `Remove()`；原石门已被删除，没有取消改造的路径。若随后 `constructionbuilder:StartConstruction` 失败（例如站点容器打不开），物品已消耗、原实体已消失，只剩站点。[消耗与失败边界](../../../raw/current/scripts/actions.lua#L4443-L4454)

## 常见陷阱与限制

- **`OnRemoveFromEntity` 写错变量**：遍历的是未定义全局 `targetprefabs`，执行 `RemoveComponent("constructionplans")` 会在 `pairs(nil)` 处报错；删除实体不会调用该钩子，所以平时不显形。[L38-L42](../../../raw/current/scripts/components/constructionplans.lua#L38-L42)
- **运行期改映射，客户端看不到**：客户端动作只认 pristine 阶段复制的 tag；服务端运行期 `AddTargetPrefab`／`RemoveTargetPrefab` 改 tag 不会同步，客户端仍按旧 tag 显示或隐藏 CONSTRUCT（源码推断，未运行）。[写入](../../../raw/current/scripts/components/constructionplans.lua#L6-L18) [读取](../../../raw/current/scripts/componentactions.lua#L1108-L1109)
- **目标已有 constructionsite 时不会改造**：`StartConstruction` 直接返回 nil，`CONSTRUCT.fn` 把原目标交给建造者，计划物品于是被当成普通材料尝试放进站点（通常报 `NOTALLOWED`）。[L21-L23](../../../raw/current/scripts/components/constructionplans.lua#L21-L23) [动作](../../../raw/current/scripts/actions.lua#L4434-L4447)
- **映射缺失只返回 `MISMATCH`**：动作失败且没有具体台词；客户端若因 tag 与服务端映射不一致仍显示动作，玩家只会看到动作失败。[L24-L27](../../../raw/current/scripts/components/constructionplans.lua#L24-L27)
- **只替换、不清理旧实体**：`Remove()` 不掉落物品或容器内容；把计划指向带 inventory／container 的实体会直接丢内容。[L32-L33](../../../raw/current/scripts/components/constructionplans.lua#L32-L33)
- **两个 prefab 命名空间**：`targetprefabs` 的键是“被改造实体”（`multiplayer_portal`），`CONSTRUCTION_PLANS` 的键是“站点 prefab”（`multiplayer_portal_moonrock_constr`）；写错 `targetprefabs` 的键只会得到 `MISMATCH`，写错 `CONSTRUCTION_PLANS` 的键则会得到空 plan 站点（能开始施工但投不进料）。[映射](../../../raw/current/scripts/components/constructionplans.lua#L24-L25) [材料表](../../../raw/current/scripts/recipes.lua#L1520-L1553)
- **没有独立宿主**：组件只挂在计划物品上；不要给世界实体或站点加它，它们各自有自己的入口。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 漂移文件 actions.lua、componentactions.lua、recipes.lua、components/constructionbuilder.lua 对照 build-26148036 更新报告（R-01、R-04）逐条核验，锚点重定位 6 处、结论修订 0 处（CONSTRUCT 动作分支与候选收集器未变；recipes 漂移为 nightlight 成本、新雕塑图纸与 charlie_hand_keystone 材料改动，均不触及月亮门计划链；constructionbuilder 仅 stopconstruction 改同步派发）；未覆盖：运行验证。

- 源码审阅：2026-10-21，Steam buildid 25643519；通读组件（44 行）与唯一的 prefab 工厂（73 行），核验 CONSTRUCT 动作分支、客户端动作收集、portal 站点接线、材料表与配方；`constructionplans` 组件在源码中仅由 `prefabs/constructionplans.lua` 添加，宿主即计划物品本身。
- 游戏运行：未运行；客户端动作显示、多人同时改造，以及转换后 `StartConstruction` 失败的路径均未实测。
- 结构检查：`validate_page.py` 通过（0 issue；本页 65 行／9627 字节）；`validate_source_links.py --strict` 共 34 条源码引用、0 error／0 warning；本页无 Lua 示例，未生成临时 `.lua`。
