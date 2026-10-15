---
title: "selfstacker"
created: "2026-10-21"
updated: "2026-10-28"
type: "component"
summary: "让掉落在地面的同类可堆叠物品在实体唤醒时自动寻找伙伴并合并，减少散落堆叠。"
aliases: ["自动堆叠", "自我堆叠", "掉落物自动合并", "selfstacker"]
tags: ["component", "lifecycle"]
dst_build: "steam-buildid:26230496"
sources: ["components/selfstacker.lua", "prefabs/houndstooth.lua", "prefabs/stinger.lua", "prefabs/meats.lua", "prefabs/spoiledfood.lua", "components/stackable.lua", "entityscript.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# selfstacker

## 快速判断

- 职责与适用：给地面物品增加“唤醒时自动找同类合并”的行为；它不是 stackable 的替代，也不参与拾取、容器或存档逻辑。[selfstacker.lua:1-12](../../../raw/current/scripts/components/selfstacker.lua#L1-L12)
- 优先使用：原版狗牙、毒刺、人肉／怪物肉／藤壶、腐烂食物等掉落小物；它们都在 Prefab 的 pristine 阶段加标签，再在 master sim 加组件。[houndstooth.lua:29-32](../../../raw/current/scripts/prefabs/houndstooth.lua#L29-L32) [stinger.lua:21-22](../../../raw/current/scripts/prefabs/stinger.lua#L21-L22) [spoiledfood.lua:104-105](../../../raw/current/scripts/prefabs/spoiledfood.lua#L104-L105)
- 重要副作用：唤醒时排一个 0～0.1 秒的一次性任务，命中伙伴后直接改写双方 `stackpartner` 并搬走对方堆叠；没有事件、没有存档字段。[selfstacker.lua:42-73](../../../raw/current/scripts/components/selfstacker.lua#L42-L73)
- 修改边界：原版只在 master sim 加入；依赖 `Physics`、`stackable`、`inventoryitem`。手持、陷阱中、燃烧、满堆或移动过快的物品会跳过，且只由实体唤醒触发。[selfstacker.lua:18-26](../../../raw/current/scripts/components/selfstacker.lua#L18-L26)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `AddComponent("selfstacker")` | 设 `searchradius = 20` 并 `AddTag("selfstacker")`；标签也常由 Prefab 在 pristine 前预加。[selfstacker.lua:3-12](../../../raw/current/scripts/components/selfstacker.lua#L3-L12) |
| `searchradius` | number，默认 20 | 传给 `FindEntity` 的世界单位半径；可在加组件后直接改字段。[selfstacker.lua:6](../../../raw/current/scripts/components/selfstacker.lua#L6 [selfstacker.lua:43](../../../raw/current/scripts/components/selfstacker.lua#L43 |
| `SetIgnoreMovingFast(flag)` | `true` 或 `nil` | 为 true 时跳过速度检查；赋 false 等同 nil，仍检查速度。[selfstacker.lua:14-16](../../../raw/current/scripts/components/selfstacker.lua#L14-L16) |
| `CanSelfStack()` | boolean | 同时满足：不在陷阱（bait 空闲）、未燃烧、有未满的 stackable、有未被持有的 inventoryitem、速度平方 < 1（或忽略速度）、当前无 stackpartner。[selfstacker.lua:18-26](../../../raw/current/scripts/components/selfstacker.lua#L18-L26) |
| `FindItemToStackWith()` | 伙伴实体或 nil | `FindEntity` 必须带 `selfstacker` 标签、排除 `outofreach`；兼容性用自身 stackable 的 `CanStackWith`；命中后无条件写伙伴的 `stackpartner`。[selfstacker.lua:36-50](../../../raw/current/scripts/components/selfstacker.lua#L36-L50) |
| `DoStack()` | 无返回值 | 先取消旧任务；找伙伴后取自身剩余空间 `RoomLeft()`，从伙伴 `Get(num)`，再 `Put` 进自身；不复查 `CanSelfStack`。[selfstacker.lua:52-62](../../../raw/current/scripts/components/selfstacker.lua#L52-L62) |
| `OnEntityWake()` | 生命周期回调 | 清空 `stackpartner`；条件满足时取消旧任务，并在 `math.random() * .1` 秒后执行一次 `DoStack`。[selfstacker.lua:64-73](../../../raw/current/scripts/components/selfstacker.lua#L64-L73) |
| `OnRemoveEntity()` | 生命周期回调 | 取消 `stacktask` 并移除 `selfstacker` 标签；只在实体被移除时调用，不是组件移除回调。[selfstacker.lua:28-34](../../../raw/current/scripts/components/selfstacker.lua#L28-L34) |

### 触发点与顺序

- 唯一自动触发点是 `OnEntityWake`：唤醒 → 清伙伴 → 判条件 → 随机延迟一次任务；没有周期轮询，也没有监听拾取／落地事件。[selfstacker.lua:64-73](../../../raw/current/scripts/components/selfstacker.lua#L64-L73)
- 合并顺序：`DoStack` 先取消待执行任务，再重新 `FindEntity`，用自身 `stackable:RoomLeft()` 决定数量，从伙伴 `Get(num)` 后 `Put` 进自身；`Put` 完全吸收时调用来源实体的 `Remove()`。[selfstacker.lua:52-62](../../../raw/current/scripts/components/selfstacker.lua#L52-L62) [stackable.lua:109-116](../../../raw/current/scripts/components/stackable.lua#L109-L116) [stackable.lua:155-189](../../../raw/current/scripts/components/stackable.lua#L155-L189)
- 单位与上限：半径 20（世界单位）；延迟 0～0.1 秒；速度按平方 < 1 判断，即约小于 1 单位／秒；单次合并量受 maxsize 与 `RoomLeft` 限制。[selfstacker.lua:24](../../../raw/current/scripts/components/selfstacker.lua#L24 [selfstacker.lua:58-59](../../../raw/current/scripts/components/selfstacker.lua#L58-L59)

## 典型接入

### 狗牙：pristine 标签＋服务端组件（原版宿主）

- 前提与选择：`prefabs/houndstooth.lua` 注册名 `houndstooth`；在 `SetPristine` 前预加 `selfstacker` 标签，令客户端也能看到同一实体标签，master sim 段再加组件。[houndstooth.lua:29-32](../../../raw/current/scripts/prefabs/houndstooth.lua#L29-L32) [houndstooth.lua:47](../../../raw/current/scripts/prefabs/houndstooth.lua#L47 [houndstooth.lua:59](../../../raw/current/scripts/prefabs/houndstooth.lua#L59
- 依赖顺序：`MakeInventoryPhysics` → 标签 → `SetPristine` → `stackable` → `inventoryitem` → `selfstacker`；组件本身也会补标签，重复 `AddTag` 无副作用。[houndstooth.lua:13-47](../../../raw/current/scripts/prefabs/houndstooth.lua#L13-L47)
- 生效与撤销：加入后在下次唤醒生效；移除实体时组件取消任务并移除标签。若用 `RemoveComponent`，标签与待执行任务不会被清理（见陷阱）。[selfstacker.lua:28-34](../../../raw/current/scripts/components/selfstacker.lua#L28-L34)

### 毒刺：跳过速度检查（原版宿主）

- `prefabs/stinger.lua` 注册名 `stinger`；同样先加 pristine 标签，master sim 中 `AddComponent` 后立刻 `SetIgnoreMovingFast(true)`，使移动中的毒刺也能参与合并。[stinger.lua:21-22](../../../raw/current/scripts/prefabs/stinger.lua#L21-L22) [stinger.lua:37-38](../../../raw/current/scripts/prefabs/stinger.lua#L37-L38) [stinger.lua:48](../../../raw/current/scripts/prefabs/stinger.lua#L48
- 该开关只放宽速度条件，不绕过满堆、手持、燃烧、陷阱等其它条件。[selfstacker.lua:18-26](../../../raw/current/scripts/components/selfstacker.lua#L18-L26)

### 肉类与腐烂食物：数据工厂统一加标签

- `prefabs/meats.lua` 的 `common` 在 pristine 前把 `tags` 数组逐项 `AddTag`；`humanmeat`、`monstermeat`、`barnacle` 分别在数组中带 `selfstacker`，再在各自 master sim 加组件。[meats.lua:154-159](../../../raw/current/scripts/prefabs/meats.lua#L154-L159) [meats.lua:225-241](../../../raw/current/scripts/prefabs/meats.lua#L225-L241) [meats.lua:283-300](../../../raw/current/scripts/prefabs/meats.lua#L283-L300) [meats.lua:691-709](../../../raw/current/scripts/prefabs/meats.lua#L691-L709)
- 注册名分别为 `humanmeat`、`monstermeat`、`barnacle`；`spoiled_food` 走自己的 fn，但同样是标签在 pristine 前、组件在 master sim。[meats.lua:920](../../../raw/current/scripts/prefabs/meats.lua#L920 [meats.lua:902](../../../raw/current/scripts/prefabs/meats.lua#L902 [meats.lua:925](../../../raw/current/scripts/prefabs/meats.lua#L925 [spoiledfood.lua:104-105](../../../raw/current/scripts/prefabs/spoiledfood.lua#L104-L105) [spoiledfood.lua:133](../../../raw/current/scripts/prefabs/spoiledfood.lua#L133 [spoiledfood.lua:263](../../../raw/current/scripts/prefabs/spoiledfood.lua#L263
- 这套写法让“客户端有标签、服务端才有组件”，客户端逻辑只能依赖标签，不能索引 `components.selfstacker`。

### 最小接线骨架

```lua
-- 目标：物品 prefab 函数；entity 已通过 MakeInventoryPhysics 获得 Physics
inst:AddTag("selfstacker") -- 可选：在 SetPristine 之前加，客户端也能看到
-- ... SetPristine 与 ismastersim 判断之后 ...
inst:AddComponent("stackable")
inst.components.stackable.maxsize = TUNING.STACK_SIZE_SMALLITEM
inst:AddComponent("inventoryitem")
inst:AddComponent("selfstacker")
```

- 调整与验证：标签与组件必须成对出现在服务端；只加标签会让命中后的 `components.selfstacker` 为 nil。[selfstacker.lua:43-47](../../../raw/current/scripts/components/selfstacker.lua#L43-L47)

## 常见陷阱与限制

- **只在唤醒时尝试一次**：`OnEntityWake` 才排队；对着已唤醒实体 `AddComponent` 不会立刻合并，且首次找不到伙伴后没有周期重试，要等下次唤醒或手动 `DoStack`。[selfstacker.lua:64-73](../../../raw/current/scripts/components/selfstacker.lua#L64-L73)
- **标签不等于组件**：`FindEntity` 只看 `selfstacker` 标签，命中后直接写 `partner.components.selfstacker.stackpartner`；服务端实体若只有标签没有组件会在此处索引 nil 报错。[selfstacker.lua:36-47](../../../raw/current/scripts/components/selfstacker.lua#L36-L47)
- **`RemoveComponent` 不清标签／任务**：组件只实现 `OnRemoveEntity`（实体移除时调用），而 `RemoveComponent` 只调用 `OnRemoveFromEntity`；因此组件移除后标签与已排队任务仍在，任务还可能执行一次合并。[selfstacker.lua:28-34](../../../raw/current/scripts/components/selfstacker.lua#L28-L34) [entityscript.lua:670-680](../../../raw/current/scripts/entityscript.lua#L670-L680) [entityscript.lua:1748-1752](../../../raw/current/scripts/entityscript.lua#L1748-L1752)
- **延迟窗口不复查条件**：条件在唤醒时判定，`DoStack` 不再检查 `CanSelfStack`；0～0.1 秒内被拾取或状态改变，任务仍会执行且不复查；只要附近仍有合法伙伴就可能合并一次。[selfstacker.lua:64-73](../../../raw/current/scripts/components/selfstacker.lua#L64-L73) [selfstacker.lua:52-62](../../../raw/current/scripts/components/selfstacker.lua#L52-L62)
- **伙伴指针未加锁**：`FindItemToStackWith` 无条件覆盖伙伴的 `stackpartner`，不检查对方是否已被占用；同帧互相唤醒时可能互指，先后执行的结果取决于任务顺序，需要严格顺序的玩法不要依赖它。[selfstacker.lua:42-50](../../../raw/current/scripts/components/selfstacker.lua#L42-L50)
- **自动流程下一次唤醒最多合并一次**：成功后 `stackpartner` 保留，`CanSelfStack` 因 `not self.stackpartner` 变为 false，直到下次唤醒清空；手动再调 `DoStack()` 仍可再次合并，但不要期望自动连续吸完周围所有堆叠。[selfstacker.lua:18-26](../../../raw/current/scripts/components/selfstacker.lua#L18-L26) [selfstacker.lua:64-73](../../../raw/current/scripts/components/selfstacker.lua#L64-L73)
- **伙伴缺 stackable 会崩**：筛选只用自身 stackable 比较 Prefab／皮肤，不检查伙伴是否有 stackable；带标签但未加 stackable 的候选会在 `DoStack` 的 `partner.components.stackable:Get` 处索引 nil。[selfstacker.lua:57-60](../../../raw/current/scripts/components/selfstacker.lua#L57-L60)
- **缺少 Physics 会报错**：速度检查直接调用 `self.inst.Physics:GetVelocity()`；同时具备 stackable／inventoryitem、但没有 Physics 的纯逻辑实体加到该组件后会在 `CanSelfStack` 处崩。[selfstacker.lua:24](../../../raw/current/scripts/components/selfstacker.lua#L24
- **只负责地面合并**：不处理容器内物品、不同 Prefab 或皮肤差异；兼容规则与数量上限见 [[reference/components/stackable]]，持有与拾取见 [[reference/components/inventoryitem]]。[stackable.lua:72-79](../../../raw/current/scripts/components/stackable.lua#L72-L79)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 1 处（entityscript 移除逻辑后的行号迁移，回调内容一致）；结论修订：无，仅锚点迁移。
- 依据：全局枢纽 hunk 地图（R-08）＋变更清单命中 entityscript.lua。

- 源码审阅：2026-10-21，steam-buildid:25643519；通读 `components/selfstacker.lua`（74 行），核对 `houndstooth`、`stinger`、`meats`、`spoiled_food` 四个宿主文件的标签与组件顺序，并核对 `entityscript` 的组件移除／实体移除回调；正文源码链接 54 处，结论只覆盖这些路径。
- 校验：`validate_page.py` 0 error／0 warning；`validate_source_links.py --strict` 0 issue；接线骨架经 `parser_probe --syntax-only` 通过。
- 游戏运行：未运行；同帧多物唤醒的竞争、延迟窗口内拾取、移除组件后的残留任务未实测。

相关：[[reference/components/container]]；[[reference/components/stackable]]；[[reference/components/inventoryitem]]。
