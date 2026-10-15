---
title: "objectspawner"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "保存一组关联实体的 GUID 并在新建或读档接管时调用宿主回调，位置、补生和删除由宿主负责。"
aliases: ["关联物体生成", "建筑部件存档", "对象 GUID 恢复", "objectspawner"]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/objectspawner.lua", "prefabs/telebase.lua", "prefabs/ruinsrespawner.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# objectspawner

## 快速判断

- 职责与适用：维护宿主的关联实体数组与保存引用，新生成、接管和读档恢复共用同一登记回调；不是周期生成、数量限制或通用巢穴系统。[objectspawner.lua:1](../../../raw/current/scripts/components/objectspawner.lua#L1 [objectspawner.lua:19](../../../raw/current/scripts/components/objectspawner.lua#L19
- 优先使用：生成新实体用 SpawnObject，已有实体用 TakeOwnership；先设置 onnewobjectfn，使新建与读档都能重接监听／代理关系。[objectspawner.lua:30](../../../raw/current/scripts/components/objectspawner.lua#L30
- 重要副作用：TakeOwnership 无去重，重复调用会重复入表并重复执行回调；“接管”不修改实体父子关系、旧宿主记录或归属字段。[objectspawner.lua:30](../../../raw/current/scripts/components/objectspawner.lua#L30
- 修改边界：原版 telebase 在服务端安装；组件自己不筛端、不定位、不自动监听子体移除，也没有移除组件时清场的钩子，均需宿主配套。[telebase.lua:257](../../../raw/current/scripts/prefabs/telebase.lua#L257 [telebase.lua:279](../../../raw/current/scripts/prefabs/telebase.lua#L279 [objectspawner.lua:1](../../../raw/current/scripts/components/objectspawner.lua#L1

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `objects` | 默认空数组 | 保存与装载均按 ipairs 顺序遍历；宿主维护成员增删，不宜制造中间空洞。[objectspawner.lua:1](../../../raw/current/scripts/components/objectspawner.lua#L1 [objectspawner.lua:7](../../../raw/current/scripts/components/objectspawner.lua#L7 |
| `onnewobjectfn` | `fn(inst, obj)`，默认 nil | 在追加数组后调用；既用于新生成，也用于 TakeOwnership 和成功找回的存档实体，不是仅首次出生的事件。[objectspawner.lua:19](../../../raw/current/scripts/components/objectspawner.lua#L19 [objectspawner.lua:30](../../../raw/current/scripts/components/objectspawner.lua#L30 |
| `TakeOwnership(obj)` | 已存在实体；无显式返回 | 直接 table.insert 并执行回调；不检查有效性、重复或容量，不从别的集合转移。[objectspawner.lua:30](../../../raw/current/scripts/components/objectspawner.lua#L30 |
| `SpawnObject(obj, linked_skinname, skin_id)` | prefab 名、透传参数；返回生成结果 | SpawnPrefab 后立即 TakeOwnership；没有位置参数或生成失败判空。存在回调时也会把失败结果交给回调，不能保证任意 prefab 安全生成。[objectspawner.lua:37](../../../raw/current/scripts/components/objectspawner.lua#L37 |

## 关键流程

OnSave 将数组成员 GUID 写入 `{objects={...}}`，同时返回同一批 GUID 引用；空数组不返回数据。它不嵌套保存一份实体记录，不验证成员是否已被移除，也不保存回调或补生时间。[objectspawner.lua:7](../../../raw/current/scripts/components/objectspawner.lua#L7

本组件没有 OnLoad；LoadPostPass 遍历保存的 GUID，只对 newents 中存在的条目调用 TakeOwnership(child.entity)，缺失条目跳过，**不会因引用缺失而生成替代物**。回调应在宿主构造时装好；恢复时重复使用回调是重建事件关联的入口，不能把它写成无条件再生成一套部件。[objectspawner.lua:19](../../../raw/current/scripts/components/objectspawner.lua#L19

## 典型接入

### 原版 telebase：建筑的独立宝石座部件

`telebase` 服务端工厂安装组件并将 onnewobjectfn 设为 NewObject；onbuilt 才按配置创建三个 gemsocket，调用返回后由宿主计算旋转后的世界坐标，再隐藏／延迟显示。NewObject 为每个部件重接 trade、picked 监听并设置 proxy_destroy_entity；读档接管会再次执行这层接线。宿主 onremove 明确遍历部件 Remove，组件自身不代做清场。[telebase.lua:279](../../../raw/current/scripts/prefabs/telebase.lua#L279 [telebase.lua:183](../../../raw/current/scripts/prefabs/telebase.lua#L183 [telebase.lua:165](../../../raw/current/scripts/prefabs/telebase.lua#L165 [telebase.lua:128](../../../raw/current/scripts/prefabs/telebase.lua#L128 [telebase.lua:294](../../../raw/current/scripts/prefabs/telebase.lua#L294

### 原版 ruinsrespawner：宿主补齐移除登记与遗迹重置

`MakeFn` 构造非联网的辅助实体并安装 objectspawner；onnewobjectfn 监听子体 onremove，通过 table.removearrayvalue 从数组删除。可选的换 prefab 监听也在此把新实体 TakeOwnership，说明动态维护名单是宿主回调职责。[ruinsrespawner.lua:1](../../../raw/current/scripts/prefabs/ruinsrespawner.lua#L1 [ruinsrespawner.lua:52](../../../raw/current/scripts/prefabs/ruinsrespawner.lua#L52

`resetruins` 事件由辅助宿主延迟触发 tryspawn，只有待重置且数组为空才 SpawnObject，再设置 spawnlocation 与位置、调用 onrespawnfn。宿主自己保存 resetruins 并在 OnLoadPostPass 续做；Inst／WorldGen 工厂分别注册相应 prefab。这些重置、等待及放置逻辑都不属于 objectspawner 的内建能力。[ruinsrespawner.lua:15](../../../raw/current/scripts/prefabs/ruinsrespawner.lua#L15 [ruinsrespawner.lua:36](../../../raw/current/scripts/prefabs/ruinsrespawner.lua#L36 [ruinsrespawner.lua:68](../../../raw/current/scripts/prefabs/ruinsrespawner.lua#L68 [ruinsrespawner.lua:84](../../../raw/current/scripts/prefabs/ruinsrespawner.lua#L84

## 常见陷阱与限制

- **对象消失就自动从 objects 消失**：组件没有该监听；残留引用可能影响宿主空表判断或后续遍历。需要动态成员时像 ruinsrespawner 一样在回调中登记移除监听并维护数组。[objectspawner.lua:30](../../../raw/current/scripts/components/objectspawner.lua#L30 [ruinsrespawner.lua:1](../../../raw/current/scripts/prefabs/ruinsrespawner.lua#L1
- **回调收到的对象已经放在最终位置**：SpawnObject 在返回前执行回调，而 telebase、ruinsrespawner 都在返回后定位；依赖最终坐标的逻辑应放在实际定位之后。[objectspawner.lua:37](../../../raw/current/scripts/components/objectspawner.lua#L37 [telebase.lua:189](../../../raw/current/scripts/prefabs/telebase.lua#L189 [ruinsrespawner.lua:24](../../../raw/current/scripts/prefabs/ruinsrespawner.lua#L24

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、telebase 部件创建／回调／移除，以及 ruinsrespawner 的动态数组、遗迹重置与宿主保存恢复。采用 simple 档：核心仅数组登记、回调与 GUID 引用恢复；未展开宝石座内部交互及其他嘉年华／洞口消费者。
- 结构检查：本页 validate_page 及源码引用 strict 检查通过；无新增完整 Lua 示例。
- 游戏运行：未运行；重复接管、子体单独删除、存档引用缺失、遗迹重置与组件主动拆卸待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/minionspawner]]；[[reference/components/childspawner]]。
