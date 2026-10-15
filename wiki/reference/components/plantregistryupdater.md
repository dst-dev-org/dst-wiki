---
title: "plantregistryupdater"
created: "2026-10-18"
updated: "2026-10-28"
type: component
summary: "承接人物植物／肥料研究，区分服务器去重、本地档案持久化、客户端通知与新知识奖励边界。"
aliases: ["植物图鉴登记", "新研究奖励精神", "研究去重与重登", "plantregistryupdater"]
tags: [component, event, network, data, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/plantregistryupdater.lua", "plantregistrydata.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "networkclientrpc.lua", "main.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# plantregistryupdater

## 快速判断

- 职责与适用：人物侧包装 plantregistrydata，学习植物阶段／肥料并在适当条件通知本人客户端；不是只存在服务端的组件，也不是完整图鉴注册 API。[构造与更新](../../../raw/current/scripts/components/plantregistryupdater.lua#L1-L48) [双端安装位置](../../../raw/current/scripts/prefabs/player_common.lua#L2516-L2518) [客户端返回位置](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2615)
- 优先使用：正常研究从人物 learnplantstage／learnfertilizer 事件进入；明确授予知识时用实例 LearnPlantStage／LearnFertilizer，不手动复制 RPC 或直接改私有表。[人物监听](../../../raw/current/scripts/prefabs/player_common.lua#L692-L695) [事件转发](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L929-L948)
- 重要副作用：本地玩家激活时改用 ThePlantRegistry 并启用保存；远端／专服角色持有单独记录，服务器不能据此知道完整客户端图鉴。方法自身没有新增布尔返回。[激活](../../../raw/current/scripts/components/plantregistryupdater.lua#L1-L14) [新增／通知](../../../raw/current/scripts/components/plantregistryupdater.lua#L16-L48)
- 修改边界：涉及角色奖励只在服务端；人物实例去重不等于账号终身成就或跨重登防刷，组件本身无 OnSave／OnLoad。[完整组件](../../../raw/current/scripts/components/plantregistryupdater.lua#L1-L64) 本地保存范围见下文。

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `plantregistry` | 构造新建的数据对象；本地激活时换引用 | 当前数据层含植物／肥料／照片等；本地玩家实际使用已加载的 ThePlantRegistry，不把服务器临时表当客户端档案镜像。[构造／切换](../../../raw/current/scripts/components/plantregistryupdater.lua#L1-L14) [本地加载](../../../raw/current/scripts/main.lua#L421-L423) |
| `LearnPlantStage(plant,stage)` | 已定义植物键、对外研究阶段整数；无显式返回 | plant／stage 为真值才调用数据层；数据层返回 updated，updater 用于音效和通知但不向调用方传播。原数据层按键／阶段去重，不验证所有输入形状。[包装](../../../raw/current/scripts/components/plantregistryupdater.lua#L16-L31) [数据层](../../../raw/current/scripts/plantregistrydata.lua#L213-L247) |
| `LearnFertilizer(key)` | 肥料键；无显式返回 | 新记录才触发对应音效／通知判断；数据层按键去重。不是研究物品消耗接口。[包装](../../../raw/current/scripts/components/plantregistryupdater.lua#L33-L48) [数据层](../../../raw/current/scripts/plantregistrydata.lua#L250-L268) |
| `plantregistry:KnowsPlantStage(plant,stage)`／`:KnowsFertilizer(key)` | boolean | 只查询该数据对象；奖励需明确是哪一端／哪次角色实体的记录，不能跨端合并推断。[查询](../../../raw/current/scripts/plantregistrydata.lua#L35-L40) [肥料查询](../../../raw/current/scripts/plantregistrydata.lua#L64-L66) |
| `TakeOversizedPicture(plant,weight,beardskin?,beardlength?)` | 无显式返回 | 向数据层传自身人物；需 userid 且角色在 DST_CHARACTERLIST，模组角色被数据层拒绝。服务器／非本地人物不能比较个人纪录时数据层直接 true，不能套用普通阶段的去重理解。[包装](../../../raw/current/scripts/components/plantregistryupdater.lua#L50-L61) [限制／转发依据](../../../raw/current/scripts/plantregistrydata.lua#L277-L297) |

植物／肥料 updated 为真时，若专服或服务端上的远端玩家且有 userid，发对应 CLIENT_RPC；客户端 handler 再调用本人的 updater。音效还要求 TheFocalPoint 的父实体为该人物。不要在本地处理后自行再发同一 RPC。[植物通知](../../../raw/current/scripts/components/plantregistryupdater.lua#L16-L31) [肥料通知](../../../raw/current/scripts/components/plantregistryupdater.lua#L33-L48) [接收](../../../raw/current/scripts/networkclientrpc.lua#L1440-L1459)

数据层的 save_enabled 初始未设；学习新增且保存启用时才走本地 Save(true)，写 PersistentString("plantregistry")，原版非 modded 定义还可能更新平台档案。保存不是人物世界存档字段；新键能进入记录不代表已有图鉴展示、完整定义或平台同步支持。[初值](../../../raw/current/scripts/plantregistrydata.lua#L7-L15) [保存](../../../raw/current/scripts/plantregistrydata.lua#L101-L107) [植物保存](../../../raw/current/scripts/plantregistrydata.lua#L219-L247) [肥料保存](../../../raw/current/scripts/plantregistrydata.lua#L256-L267)

## 典型接入

### 当前服务端人物登记中新学会一个植物阶段时，奖励 1 点精神

放目标模组 `modmain.lua`，无需新资源。包装已有 LearnPlantStage，比较登记前后状态，而不是把方法 nil 返回当失败；原学习／客户端通知仍由原方法执行。即时精神值使用当前 [[reference/components/sanity#关键接口|DoDelta]]。[原更新](../../../raw/current/scripts/components/plantregistryupdater.lua#L16-L31) [玩家 PostInit](../../../raw/current/scripts/modutil.lua#L586-L590)

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local updater = inst.components.plantregistryupdater
    if updater == nil then return end
    local original = updater.LearnPlantStage
    updater.LearnPlantStage = function(self, plant, stage)
        local known = plant and stage
            and self.plantregistry:KnowsPlantStage(plant, stage)
        original(self, plant, stage)
        if plant and stage and not known
                and self.plantregistry:KnowsPlantStage(plant, stage)
                and self.inst.components.sanity ~= nil then
            self.inst.components.sanity:DoDelta(1)
        end
    end
end)
```

同一当前登记对象、同一键／阶段重复提交不再奖励；肥料和照片不在目标内。宿主的本地玩家可使用已有档案，远端服务端记录则无法证明对方以往未知；重登／角色重建可能再次进入“本对象新增”。本例**不是可用于永久成就的防刷方案**，需要这类玩法时另建模组自己的服务端持久化记录，不读取玩家本地文件猜测。

停用并重新初始化恢复原方法，不回滚已经保存的研究或已发放精神。植物源事件见 [[reference/components/plantresearchable]]，肥料源事件见 [[reference/components/fertilizerresearchable]]；新条目的注册／展示超出本例。

## 常见陷阱与限制

- **服务器 Knows=false 就是账号从未研究**：服务器持有的是当前记录，不掌握完整个人本地历史；通知是减流量的有限去重，不是权威账号成就库。[原说明与通知](../../../raw/current/scripts/components/plantregistryupdater.lua#L25-L29)
- **用 updater:LearnPlantStage 的返回判断新增**：updater 不传播数据层 updated；比较实际查询前后，或设计自己的权威记录。[包装](../../../raw/current/scripts/components/plantregistryupdater.lua#L16-L31)
- **照片可直接用于所有模组角色，且与阶段同样去重**：数据层明确排除非原版角色，远端路径无法比较本人历史而直接返回 true；照片／皮肤 UI 全链需另行核验。[照片判断](../../../raw/current/scripts/plantregistrydata.lua#L277-L297)

## 验证记录

- 增量复查：2026-10-25，Steam buildid 26148036；仅锚点重定位（player_common −12/−14 段、player_common_extensions +4 段、main +7 段），组件与数据层无变化；main 新增反馈系统、networkclientrpc 新增 SyncCharlieDefeated 均不涉本页语义，结论维持。
- 源码审阅：2026-10-18，Steam buildid 25643519；覆盖双端安装／本地激活、人物事件、数据层阶段／肥料去重与保存、RPC 发送／接收、照片角色与远端比较限制、服务端奖励前后检查示例。sanity API 复用当前已核验页；未全面审校档案加载／平台交互、图鉴 UI、照片皮肤及跨平台存储。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明网络、持久化或奖励防重效果。
- 游戏运行：未运行；主机／专服／远端玩家、playeractivated 时序、重复提交、重登／角色更换、档案保存与其他模组方法包装待实测。
