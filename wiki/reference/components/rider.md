---
title: "rider"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "管理骑手侧上下骑转换、坐骑隐藏与恢复、鞍跟踪和保存，提供 mounted／dismounted 接入点。"
aliases: ["骑乘时每分钟加三点精神", "玩家下骑", "骑手状态", "rider"]
tags: [component, creature, event, network, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/rider.lua", "prefabs/player_common.lua", "stategraphs/SGwilson.lua", "actions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# rider

## 快速判断

- 职责与适用：原玩家主模拟安装 rider，管理人物骑乘状态、坐骑与鞍；坐骑端条件见 [[reference/components/rideable]]。[玩家安装](../../../raw/current/scripts/prefabs/player_common.lua#L2803-L2806)
- 优先使用：状态用 IsRiding／GetMount／GetSaddle；普通下骑请求用 Dismount，响应完整转换用 mounted／dismounted 事件。不要把 ActualDismount 当普通玩法入口。[查询](../../../raw/current/scripts/components/rider.lua#L246-L252) [下骑分工](../../../raw/current/scripts/components/rider.lua#L173-L182)
- 重要副作用：Mount 改人物动画／方向／伤害重定向／可钉住状态，将坐骑挂为子实体并隐藏，再写双向引用／发 mounted；不是单个布尔开关。[转换](../../../raw/current/scripts/components/rider.lua#L90-L170)
- 修改边界：Mount 只做其内部资格检查，正常 MOUNT 动作还查战斗、死亡、冰冻、被拴与占用。直接调方法不能据此绕过实际玩法限制。[方法门禁](../../../raw/current/scripts/components/rider.lua#L67-L77) [动作门禁](../../../raw/current/scripts/actions.lua#L4141-L4164)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `IsRiding()`／`GetMount()`／`GetSaddle()` | 读骑乘状态／坐骑／跟踪鞍；riding/mount/saddle 属性更新 replica，不能手动赋其中一项模拟完整转换。[同步](../../../raw/current/scripts/components/rider.lua#L9-L38) [查询](../../../raw/current/scripts/components/rider.lua#L63-L65) [坐骑查询](../../../raw/current/scripts/components/rider.lua#L246-L252) |
| `Mount(target, instant)` | 已骑、无 rideable 或目标被骑时返回；服从／自定义测试失败发 refusedmount 和 refusedrider。成功过程无显式 true 返回；instant 选 idle 而非 mount 状态，不代表跳过其他协作。[门禁](../../../raw/current/scripts/components/rider.lua#L67-L85) [状态选择](../../../raw/current/scripts/components/rider.lua#L117-L125) |
| `Dismount()` | 只给玩家发 dismount；SG 决定是否响应、何时清理，不能立即假设 GetMount=nil。[请求](../../../raw/current/scripts/components/rider.lua#L173-L182) [SG 响应](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2376-L2386) |
| `ActualDismount()` | SG／清理使用，非骑乘时直接返回；恢复人物、坐骑场景／位置、双向引用和跟踪，发 dismounted 后返回 ex_mount。[实现](../../../raw/current/scripts/components/rider.lua#L179-L243) [SG 调用](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19692-L19694) |
| `StartTracking/StopTracking(mount)` | 只管理 saddlechanged 监听和当前鞍，不代办完整 Mount／Dismount；卸载也仅取消伤势提示任务和跟踪。[跟踪／卸载](../../../raw/current/scripts/components/rider.lua#L40-L65) |
| `mounted`／`dismounted` | 事件都在骑手上，data.target 是坐骑／旧坐骑；mounted 在引用建立后，dismounted 在引用清空后，适合条件效果成对增删。[上骑尾部](../../../raw/current/scripts/components/rider.lua#L160-L171) [下骑尾部](../../../raw/current/scripts/components/rider.lua#L237-L243) |

上骑时的伤害转交对投射物、部分远程、电与黑暗等有排除，不保证骑手完全免伤；下骑会清掉该 redirectdamagefn。额外修改须考虑双方所有权，不能不留恢复路径地覆盖同一字段。[重定向](../../../raw/current/scripts/components/rider.lua#L127-L145) [清理](../../../raw/current/scripts/components/rider.lua#L212-L218)

（26148036 核验：C-34——伤害转交判定新增排除“穿在骑手身上的攻击者”（slurper/bat_boss 类，`inventory:IsItemEquipped(attacker)`），此类攻击不再触发转交、也不再借受击强制下马——[rider.lua:133-139](../../../raw/current/scripts/components/rider.lua#L133-L139)）

OnSave 在坐骑存在且其 ShouldSave 为真时嵌入保存记录，OnLoad SpawnSaveRecord 后 Mount(...,true)。源码注明普通退出／加载通常会先离骑，这条保存也服务自动保存；不能承诺所有重进场景仍保持骑乘，更不能把本地临时回调写进存档。[保存](../../../raw/current/scripts/components/rider.lua#L254-L267)

## 典型接入

### 骑乘时添加每分钟三点精神贡献，下骑撤销

目标模组 `modmain.lua`，无需资源，作用于有 rider／sanity 的人物。用完整转换事件更新独立 key，不轮询、不替换上骑方法。精神列表接口、点／秒单位和最终 rate_modifier 复用 [[reference/components/sanity#给人物增加每分钟三点精神贡献]]；这里维护骑乘条件。[玩家入口](../../../raw/current/scripts/modutil.lua#L585-L590) [事件](../../../raw/current/scripts/components/rider.lua#L160-L171)

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local function Refresh(player)
        local sanity = player.components.sanity
        if sanity == nil then return end
        local rider = player.components.rider
        if rider ~= nil and rider:IsRiding() then
            sanity.externalmodifiers:SetModifier(player, 3 / 60, "my_mod_riding")
        else
            sanity.externalmodifiers:RemoveModifier(player, "my_mod_riding")
        end
    end
    inst:ListenForEvent("mounted", Refresh)
    inst:ListenForEvent("dismounted", Refresh)
    Refresh(inst)
end)
```

`3/60` 为额外 0.05 点／秒，仍参与人物最终精神速率，不保证净每分钟上涨三点。相同 source/key 更新不累积；更换前缀时保留增删一致。实际下骑才撤销，发出尚未完成的 Dismount 请求不会提前取消；加载通过 Mount 恢复时会再次发 mounted。永久移除人物时来源随实体消失，列表清理规则见上述 sanity 页。

停用后重新加载不再注册效果；不手动保存 modifier 或骑乘布尔值。其他模组若直接改字段、不发完整转换事件，本例不会替它修复；动态卸载效果代码时应先移除同一 key。

## 常见陷阱与限制

- **Mount 没返回 true 就是失败**：方法成功也无显式 true，动作甚至可在方法拒绝后返回 true；看实际状态／事件，不能按返回值再生成坐骑。[方法](../../../raw/current/scripts/components/rider.lua#L160-L171) [动作](../../../raw/current/scripts/actions.lua#L4163-L4164)
- **GetMount 等于网络已在全部客户端可用**：服务端引用和 replica 更新是不同消费路径，本页不承诺预测／dirty 时序。[属性转发](../../../raw/current/scripts/components/rider.lua#L9-L19)
- **卸载 rider 自动下骑**：OnRemoveFromEntity 只清跟踪／提示任务，不恢复父子关系和人物状态；避免用卸载替代正常 SG 转换。[卸载](../../../raw/current/scripts/components/rider.lua#L40-L46)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 中漂移文件 rider.lua、player_common.lua、SGwilson.lua、actions.lua（更新报告 C-34、P-83、SB-17、R-01）——结论修订 1 处：骑乘伤害转交新增“穿在骑手身上的攻击者”豁免（C-34，见正文“26148036 核验”）；SGwilson 骑乘区本轮为大段位移（SB-17），dismount 事件与 ActualDismount 调用文本未变；锚点重定位 19 处；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖上／下骑转换、事件顺序、鞍跟踪、replica 转发、保存及必要玩家／动作／SG 消费。精神 API 复用已核验 sanity 页面；未全面审校预测、所有 SG／坐骑或战斗重定向消费者。
- 结构检查：本批格式、来源、链接／行号、大小和登记通过；完整事件例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；上下骑动画、拒绝／甩落、精神模式／速率、自动保存／重进、死亡与多模组事件变更待实测。
