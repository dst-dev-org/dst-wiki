---
title: "rainimmunity"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "在目标实体上按来源集合授予雨水免疫，令雨水增湿与若干淋雨消费者视为无雨，并可成对撤销。"
aliases: ["免雨", "雨水免疫", "雨淋免疫", "防雨组件", "rainimmunity"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/rainimmunity.lua", "components/moisture.lua", "entityscript.lua", "entityreplica.lua", "components/raindome.lua", "components/dryingrack.lua", "components/acidlevel.lua", "components/inventoryitemmoisture.lua", "components/inventoryitem.lua", "components/lunarhailbuildup.lua", "prefabs/alterguardian_phase4_lunarrift.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# rainimmunity

## 快速判断

- 职责与适用：挂在**要被保护的目标实体**上，本身不改数值，只维护“哪些来源正在提供免雨”与 `rainimmunity` 标签，供其他组件把“正在下雨”判成否。[构造](../../../raw/current/scripts/components/rainimmunity.lua#L1-L18) [来源集合](../../../raw/current/scripts/components/rainimmunity.lua#L32-L39)
- 优先使用：目标已存在时 `AddSource(src)`，撤销用同一 `src` 的 `RemoveSource(src)`；需要临时“绝对干燥”而不是免雨时选 `moisture:ForceDry`，见 [[reference/components/moisture#防雨还是强制干燥]]。
- 重要副作用：最后一个来源离开会**自动移除本组件**，摘标签并发 `loserainimmunity`；来源集合不存档，读档后必须由驱动方重建。[自动移除](../../../raw/current/scripts/components/rainimmunity.lua#L7-L12) [移除清理](../../../raw/current/scripts/components/rainimmunity.lua#L20-L30)
- 修改边界：授予与撤销在服务端；它是普通组件而非复刻组件，客户端只共享 `rainimmunity` 标签，不能调用方法或读来源集合。[客户端查询](../../../raw/current/scripts/entityscript.lua#L794-L810) [可复刻清单](../../../raw/current/scripts/entityreplica.lua#L5-L26)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `sources` | table，`src`→true | 当前提供者集合；同 `src` 重复添加即忽略，`next(sources) == nil` 时移除组件。[L3-L12](../../../raw/current/scripts/components/rainimmunity.lua#L3-L12) |
| `AddSource(src)` | entity | 首次加入；`src ~= inst` 时注册其 `onremove` 以便提供者消失时自动撤销，自身来源不监听。[L32-L39](../../../raw/current/scripts/components/rainimmunity.lua#L32-L39) |
| `RemoveSource(src)` | entity | 撤销该来源并解绑监听；可能顺带移除整个组件。[L41-L48](../../../raw/current/scripts/components/rainimmunity.lua#L41-L48) |
| `OnRemoveFromEntity()` | 无参 | 摘 `rainimmunity` 标签、解绑其余来源的 `onremove`、发 `loserainimmunity`；不再回调各来源。[L20-L30](../../../raw/current/scripts/components/rainimmunity.lua#L20-L30) |
| 事件 `gainrainimmunity`／`loserainimmunity` | 目标实体上的事件 | 构造期即发前者，移除时发后者；构造期 `inst.components.rainimmunity` 尚未登记。[L14-L17](../../../raw/current/scripts/components/rainimmunity.lua#L14-L17) |
| 标签 `rainimmunity` | 无值标签 | 随组件增删；是客户端与无 moisture 实体唯一能看到的信息。[增删](../../../raw/current/scripts/components/rainimmunity.lua#L5) [查询](../../../raw/current/scripts/entityscript.lua#L787) |

组件没有 `OnSave`／`OnLoad`，也没有可复刻的网络字段；加组件前的 `inst.components.rainimmunity == nil` 判断是原版避免重复添加的写法，不是接口要求。[原版判断](../../../raw/current/scripts/components/raindome.lua#L198-L201)

### 它挡的是雨水路径，不是“潮湿”

- 雨水增湿速率：moisture 的 `_GetMoistureRateAssumingRain()` 直接返回 0，因此雨中不再累积湿度；已有湿度不因此清零。[L212-L215](../../../raw/current/scripts/components/moisture.lua#L212-L215)
- 雨中自然干燥：因为雨水速率是 0，`GetDryingRate` 的“下雨不干燥”判断不再成立，湿透的角色在雨中仍会自然变干。[L287-L289](../../../raw/current/scripts/components/moisture.lua#L287-L289)
- 不挡非雨水来源：`GetMoistureRate` 的持浮具／浸浴分支在雨天判断之前返回，这两条增湿路径不受影响。[L253-L264](../../../raw/current/scripts/components/moisture.lua#L253-L264)
- 不改防水汇总：`GetWaterproofness()` 只汇总装备、固有值与 modifier，免雨不是“防水值 1”。[L239-L251](../../../raw/current/scripts/components/moisture.lua#L239-L251)
- 目标没有 moisture 组件时：`GetWetMultiplier`／`GetIsWet` 的回退分支用 `rainimmunity` 标签判为不湿，客户端同样走标签。[服务端](../../../raw/current/scripts/entityscript.lua#L770-L791) [客户端](../../../raw/current/scripts/entityscript.lua#L794-L810)

本页逐行核对的消费者：物品的目标湿度按 0 结算、酸雨与普通酸度 tick 提前返回、月雹不积累、放到免疫目标上的物品不再继承世界湿度。[物品目标](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L221-L224) [酸雨](../../../raw/current/scripts/components/acidlevel.lua#L147-L153) [酸度](../../../raw/current/scripts/components/acidlevel.lua#L207-L213) [月雹](../../../raw/current/scripts/components/lunarhailbuildup.lua#L76) [继承](../../../raw/current/scripts/components/inventoryitem.lua#L158-L164)

## 典型接入

### 目标自持常驻免疫（原版 Boss 模式）

- 目标与位置：`alterguardian_phase4_lunarrift` 在服务端守卫后用 `AddComponent("rainimmunity")` 加 `AddSource(inst)`；自身来源永不过期，读档时工厂重建即恢复。[装配](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1014-L1015)
- 端与资源：仅服务端逻辑；无资源；不需要额外组件。
- 客户端一致性：同一 prefab 在 `SetPristine` 之前手动 `AddTag("rainimmunity")`，注释说明这是为优化而直接写进 pristine 状态，组件稍后添加时不会再补一次标签同步。[标签](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L948-L949) [守卫](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L996-L1007)
- 撤销：只有显式 `RemoveSource(inst)` 或移除组件才会结束；自身来源不会被自动清理。

### 由另一个组件授予或撤销（“组件挂组件”方向）

方向是**提供者持有驱动逻辑，免疫组件挂在目标上**，每次以提供者实体作为 `src`：

- 晾晒架：容器与架子不是同一实体时，架子在雨／酸雨与骑乘变化时给容器 `AddComponent("rainimmunity")` 并 `AddSource(self.inst)`，恢复时 `RemoveSource`；架子自身或骑手免疫也算数。[判定](../../../raw/current/scripts/components/dryingrack.lua#L318-L324) [授予与撤销](../../../raw/current/scripts/components/dryingrack.lua#L326-L337) 案例见 [[reference/components/dryingrack#普通晾晒架在雨天也能继续工作]]。
- 雨棚光环：`raindome` 每 1～3 秒在该半径内查找 `inspectable` 目标，逐个补组件与来源，离开集合或关掉光环时逐个 `RemoveSource`；关停还会停更新并摘 `raindome` 标签。[周期授予](../../../raw/current/scripts/components/raindome.lua#L183-L216) [关停清理](../../../raw/current/scripts/components/raindome.lua#L146-L163)
- 这两种写法的共同点：先判 `target.components.rainimmunity` 是否存在再 `AddComponent`；撤销时先判组件仍在再 `RemoveSource`，因为移除最后一个来源会让组件本身消失。

### 模组接入要点

- 授予点必须是服务端，且目标实体有效；对玩家授予时同时考虑其装备／庇护是否已足够，避免用免雨掩盖 `ForceDry` 语义。
- `src` 选提供者实体可获得自动撤销（`onremove` 监听）；选目标自身则必须在停用路径显式 `RemoveSource(inst)`。
- 需要读档后仍生效的状态，应在实体初始化或 `Enable` 路径重新授予，而不是期待来源集合被保存。

## 常见陷阱与限制

- **构造期回调里查不到组件**：`gainrainimmunity` 在构造函数内发出，此时 `inst.components.rainimmunity` 还没赋值；源码注释明确要求按“已有组件”假设处理。`AddComponent` 先做可复刻登记、再构造、最后登记组件，顺序可对照。[注释](../../../raw/current/scripts/components/rainimmunity.lua#L14-L16) [顺序](../../../raw/current/scripts/entityscript.lua#L648-L656)
- **移除最后一个来源会连带移除组件**：在 `RemoveSource` 之后继续用 `components.rainimmunity` 可能已是 nil；遍历清理时要像原版一样先判存在再调用。[L7-L12](../../../raw/current/scripts/components/rainimmunity.lua#L7-L12) [原版写法](../../../raw/current/scripts/components/raindome.lua#L206-L211)
- **客户端只按标签判断**：本组件不在可复刻清单里，客户端拿不到 `sources` 与方法，只能用 `rainimmunity` 标签走 `GetIsWet` 回退；原版 Boss 还把标签提前写进 pristine 状态，注释说明是为优化，避免生成后由组件再补一次标签。[客户端回退](../../../raw/current/scripts/entityscript.lua#L794-L810) [可复刻清单](../../../raw/current/scripts/entityreplica.lua#L5-L26) [pristine 优化](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L948-L949)
- **来源不存档**：组件无保存函数，读档只恢复实体本身；由 `raindome`／`dryingrack` 这类驱动方按各自时机重建（雨棚在更新周期、晾晒架在启用与天气事件），自定义授予必须自己重建。[授予](../../../raw/current/scripts/components/raindome.lua#L183-L205)
- **自身来源不会自动撤销**：`AddSource(inst)` 不注册 `onremove`，实体整体移除时随之消失；“实体还在、效果该停”的场景必须显式 `RemoveSource(inst)`。[L32-L39](../../../raw/current/scripts/components/rainimmunity.lua#L32-L39)
- **免雨 ≠ 免湿 ≠ 防水**：不清已有湿度、不算防水值、不阻止浮具与浸浴增湿；与 `moistureimmunity` 的“强制干燥”语义也不同。[速率分支](../../../raw/current/scripts/components/moisture.lua#L253-L264) 对照 [[reference/components/moistureimmunity]]。
- **酸雨只被部分拦截**：酸雨 tick 因组件提前返回，但装备融化／`acidrainimmune` 判断走的是另一条分支，免雨不提供该标签。[酸雨 tick](../../../raw/current/scripts/components/acidlevel.lua#L147-L153)
- **不要靠 `AddTag` 手写免疫**：标签与来源集合脱节，摘除组件时会被一并清掉；需要客户端可见时按原版在 pristine 阶段加。[移除](../../../raw/current/scripts/components/rainimmunity.lua#L20-L23)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 5 处；结论修订：无。R-08 使 GetWetMultiplier／GetIsWet 新增无效实体早退与 CanPointHaveRain 位置门控，但 rainimmunity 标签分支语义原样保留；inventoryitem 世界湿度继承新增同一位置门控，免雨目标的拦截条件不变。
- 依据：全局枢纽 hunk 地图（R-08）＋变更清单命中 entityscript.lua、components/inventoryitem.lua、components/acidlevel.lua、prefabs/alterguardian_phase4_lunarrift.lua。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖构造与事件时序、来源增删与自动移除、移除清理、三个原版宿主（自身常驻、晾晒架容器、雨棚光环）的调用方向、moisture 雨水速率与干燥分支、物品湿度、酸雨 tick、月雹与客户端标签回退。
- 未运行游戏；未覆盖：全部消费者、多人联机与读档实测、雨棚半径数值来源、其它模组对来源键的复用。
- 本页无完整 Lua 示例，未执行 Lua 语法检查。
- 结构检查：`validate_page.py` 无 issue；`validate_source_links.py --strict` 为 0 error／0 warning（本页 44 处源码引用）；无完整示例，未执行 Lua 语法检查。

相关：[[reference/components/moisture]]；[[reference/components/dryingrack]]；[[reference/components/moistureimmunity]]；[[reference/components/sheltered]]。
