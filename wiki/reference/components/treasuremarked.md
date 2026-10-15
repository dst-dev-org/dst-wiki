---
title: treasuremarked：附属宝藏地图标记的开关与恢复
created: 2026-10-27
updated: "2026-10-28"
type: component
summary: 为宿主生成附属的漂流瓶宝藏地图标记，并保存开启状态以在加载后重建。
aliases: [treasuremarked, 宝藏标记, 水下宝藏地图图标, 漂流瓶寻宝标记]
tags: [component, lifecycle, ui, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/treasuremarked.lua, prefabs/messagebottletreasure_marker.lua, prefabs/underwater_salvageable.lua, components/messagebottlemanager.lua]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# treasuremarked

## 快速判断

- 职责与适用：给宿主附加 `messagebottletreasure_marker` 子实体，关闭时移除当前引用的标记；不生成宝藏、选择奖励或执行打捞。[treasuremarked.lua:1-22](../../../raw/current/scripts/components/treasuremarked.lua#L1-L22)
- 优先使用：参考原版 `underwater_salvageable` 的服务端安装及漂流瓶管理器的沉没事件调用，适合复用这套特定地图标记语义。[underwater_salvageable.lua:58-73](../../../raw/current/scripts/prefabs/underwater_salvageable.lua#L58-L73) [messagebottlemanager.lua:148-156](../../../raw/current/scripts/components/messagebottlemanager.lua#L148-L156)
- 重要副作用：标记自身带网络与 MiniMapEntity，服务端还向世界发送增删事件，影响 `messagebottlemanager.active_treasure_hunt_markers`，不是纯装饰子实体。[messagebottletreasure_marker.lua:10-43](../../../raw/current/scripts/prefabs/messagebottletreasure_marker.lua#L10-L43) [messagebottlemanager.lua:158-171](../../../raw/current/scripts/components/messagebottlemanager.lua#L158-L171)
- 修改边界：原版宿主仅在主模拟端安装；组件方法本身没有端检查，也没有防重复开启逻辑，调用方须控制调用次数。[underwater_salvageable.lua:58-67](../../../raw/current/scripts/prefabs/underwater_salvageable.lua#L58-L67) [treasuremarked.lua:7-12](../../../raw/current/scripts/components/treasuremarked.lua#L7-L12)

## 关键接口

| 接口／字段 | 条件与作用 | 依据 |
|---|---|---|
| `marker` | 构造时为 nil；保存当前一次 `SpawnPrefab` 的结果，不是稳定的布尔开关。 | [treasuremarked.lua:1-12](../../../raw/current/scripts/components/treasuremarked.lua#L1-L12) |
| `TurnOn()` | 无条件生成标记；生成成功后将标记实体父级设为宿主实体。未返回标记或成功值。 | [treasuremarked.lua:7-12](../../../raw/current/scripts/components/treasuremarked.lua#L7-L12) |
| `TurnOff()` | 当前引用非 nil 且有效时调用 `Remove()`；不将字段赋回 nil。 | [treasuremarked.lua:14-18](../../../raw/current/scripts/components/treasuremarked.lua#L14-L18) |
| `OnRemoveFromEntity()` | 调用 `TurnOff()`，清理当前有效标记。 | [treasuremarked.lua:20-22](../../../raw/current/scripts/components/treasuremarked.lua#L20-L22) |
| `OnSave()` | 当前标记有效才返回 `{ on = true }`；不保存标记 GUID 或标记本体。 | [treasuremarked.lua:24-29](../../../raw/current/scripts/components/treasuremarked.lua#L24-L29) |
| `LoadPostPass(ents, data)` | `data` 存在且 `data.on` 为真时调用 `TurnOn()`；没有利用 `ents` 解析旧标记。 | [treasuremarked.lua:31-35](../../../raw/current/scripts/components/treasuremarked.lua#L31-L35) |

标记 Prefab 使用 `messagebottletreasure_marker.png` 小地图图标，优先级为 6，带 `NOCLICK`、`NOBLOCK` 标签；主模拟端设置 `persists = false`。因此开启状态由宿主组件保存，标记在加载后重新生成。[messagebottletreasure_marker.lua:18-43](../../../raw/current/scripts/prefabs/messagebottletreasure_marker.lua#L18-L43)

## 典型接入

### 漂流瓶宝藏沉没后显示水下位置

原版 `underwater_salvageable` 声明标记 Prefab 依赖，并在服务端添加本组件。其普通 `flotsam_heavy.png` 图标与本组件创建的宝藏标记是两个实体上的图标，不应将两者当成同一字段切换。[underwater_salvageable.lua:8-11](../../../raw/current/scripts/prefabs/underwater_salvageable.lua#L8-L11) [underwater_salvageable.lua:28-38](../../../raw/current/scripts/prefabs/underwater_salvageable.lua#L28-L38) [underwater_salvageable.lua:58-67](../../../raw/current/scripts/prefabs/underwater_salvageable.lua#L58-L67)

`messagebottlemanager:UseMessageBottle` 生成宝藏后监听该宝藏的 `on_submerge`；回调检查事件数据中的 `underwater_object` 及其组件，调用一次 `TurnOn()`，随后撤掉此事件回调。这里是沉没后得到的水下对象拥有组件，事件监听对象则是原宝藏。[messagebottlemanager.lua:148-156](../../../raw/current/scripts/components/messagebottlemanager.lua#L148-L156) [messagebottlemanager.lua:197-213](../../../raw/current/scripts/components/messagebottlemanager.lua#L197-L213)

标记生成后在零延迟任务中向世界发 `messagebottletreasure_marker_added`；标记移除时发 `messagebottletreasure_marker_removed`。管理器用标记实体作集合键增删，再以集合大小限制活动寻宝数量；到达上限时可返回已有有效标记的位置。这条消费链说明随意创建额外标记会进入同一寻宝计数。[messagebottletreasure_marker.lua:10-16](../../../raw/current/scripts/prefabs/messagebottletreasure_marker.lua#L10-L16) [messagebottletreasure_marker.lua:37-41](../../../raw/current/scripts/prefabs/messagebottletreasure_marker.lua#L37-L41) [messagebottlemanager.lua:158-171](../../../raw/current/scripts/components/messagebottlemanager.lua#L158-L171) [messagebottlemanager.lua:203-238](../../../raw/current/scripts/components/messagebottlemanager.lua#L203-L238)

宿主打捞内容由 inventory 与 winchtarget 协作；失去物品后延迟一帧移除宿主，本组件不负责奖励交付。要改打捞结果应从这些宿主入口继续核验，不能只改标记组件。[underwater_salvageable.lua:18-26](../../../raw/current/scripts/prefabs/underwater_salvageable.lua#L18-L26) [underwater_salvageable.lua:64-73](../../../raw/current/scripts/prefabs/underwater_salvageable.lua#L64-L73)

## 常见陷阱与限制

- **重复开启可能遗失旧引用：** `TurnOn` 不检查已有标记，也不移除旧标记，直接覆盖 `self.marker`；旧实体不会由这次调用清理。若新生成失败，字段还会被 nil 覆盖。随后 `TurnOff`、保存和卸载只能观察当前引用，不能据此清理或保存所有历史标记。调用方应避免重复开启。[treasuremarked.lua:7-29](../../../raw/current/scripts/components/treasuremarked.lua#L7-L29)
- **字段非 nil 不代表开启：** `TurnOff` 移除标记后保留字段，判断当前有效状态须同时考虑 `IsValid()`；`OnSave` 本身也采用这两个条件。[treasuremarked.lua:14-29](../../../raw/current/scripts/components/treasuremarked.lua#L14-L29)
- **加载不是找回旧子实体：** `LoadPostPass` 根据布尔状态新建标记；不能在加载流程外先无条件开启，再指望恢复函数复用已有实体。[treasuremarked.lua:31-35](../../../raw/current/scripts/components/treasuremarked.lua#L31-L35)

## 验证记录

- 2026-10-27，`steam-buildid:26148036`：全文审阅本组件及标记 Prefab，核对原版水下宿主、管理器的沉没回调、世界事件消费者与活动寻宝位置消费；未扩展为完整宝藏生成／打捞系统审校。
- 游戏运行：未运行；未测重复开启、生成失败、保存重载、联网地图显示或打捞后的实际清理表现。
- 结构检查：页面格式与严格源码链接检查通过；无新增完整 Lua 示例，未执行 Lua 解析测试。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。