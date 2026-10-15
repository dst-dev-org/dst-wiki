---
title: "cooldown"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "实体布尔就绪与单任务倒计时，包含回调、存档及离屏推进，区别物品充能和人物多技能冷却。"
aliases: ["cooldown","布尔冷却","冷却完成回调","骨甲冷却"]
tags: ["component","callback","lifecycle"]
dst_build: "steam-buildid:26230496"
sources: ["components/cooldown.lua","prefabs/armor_skeleton.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# cooldown

## 快速判断

- 职责与适用：一个实体的一次充能任务与 charged 布尔状态，适合定时就绪；物品百分比充能见 [[reference/components/rechargeable]]，人物多技能共享冷却见 [[reference/components/spellbookcooldowns]]。[components/cooldown.lua:1](../../../raw/current/scripts/components/cooldown.lua#L1
- 优先使用：配置 cooldown_duration 或显式 StartCharging(seconds)，用 onchargedfn 恢复能力；调用者检查 IsCharged，不自动拦技能。[components/cooldown.lua:31](../../../raw/current/scripts/components/cooldown.lua#L31)、[components/cooldown.lua:56](../../../raw/current/scripts/components/cooldown.lua#L56
- 重要副作用：StartCharging 重启并清 charged；FinishCharging 只在已有 deadline 时立即完成，不是无条件设已充能。[components/cooldown.lua:31](../../../raw/current/scripts/components/cooldown.lua#L31)、[components/cooldown.lua:46](../../../raw/current/scripts/components/cooldown.lua#L46
- 修改边界：原版骨甲在服务端挂组件，由装备／受击／燃料逻辑控制回调，不把此组件当网络 UI。[prefabs/armor_skeleton.lua:246](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L246)、[prefabs/armor_skeleton.lua:133](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L133

## 关键接口

| 接口／字段 | 参数／默认 | 条件、副作用与证据 |
|---|---|---|
| `cooldown_duration`／`StartCharging(time?)` | 默认 nil；实际需秒数 | 省参取配置，仍 nil 会算术报错；取消旧任务、重设 deadline，最后调用 startchargingfn(inst)。[components/cooldown.lua:31](../../../raw/current/scripts/components/cooldown.lua#L31 |
| `FinishCharging()` | 无显式返回 | 有 deadline 时清任务、置 charged=true，再 onchargedfn(inst)；初始未启动时不变。[components/cooldown.lua:17](../../../raw/current/scripts/components/cooldown.lua#L17)、[components/cooldown.lua:46](../../../raw/current/scripts/components/cooldown.lua#L46 |
| `IsCharged()`／`IsCharging()` | 初始 false／false | 未启动与已完成的剩余时间都可能为零，不能只用 GetTimeToCharged 判断就绪。[components/cooldown.lua:52](../../../raw/current/scripts/components/cooldown.lua#L52 |
| `GetTimeToCharged()` | 剩余秒或 0 | 不夹取非负值，任务执行前可能有微小负值；不是进度百分比。[components/cooldown.lua:52](../../../raw/current/scripts/components/cooldown.lua#L52 |
| `startchargingfn`／`onchargedfn` | `(inst)` | 启动／完成调用；load 已 charged 也调用完成函数，恢复逻辑需允许重入。[components/cooldown.lua:17](../../../raw/current/scripts/components/cooldown.lua#L17)、[components/cooldown.lua:87](../../../raw/current/scripts/components/cooldown.lua#L87 |
| `OnSave/OnLoad`／`LongUpdate(dt)` | charged／剩余秒 | 只保存状态，不存配置／函数；LongUpdate 超时完成，否则重新 StartCharging，可再次调用启动回调。[components/cooldown.lua:64](../../../raw/current/scripts/components/cooldown.lua#L64)、[components/cooldown.lua:75](../../../raw/current/scripts/components/cooldown.lua#L75 |

## 典型接入

### 骨甲常规格挡恢复间隔改为十秒

目标模组 `modmain.lua`，只改原版 armor_skeleton 的默认 duration；不覆盖燃料消耗、抗性、穿脱／首充回调或 FX。原版受击时 StartCharging()，穿戴／补料另显式使用首充时间。[prefabs/armor_skeleton.lua:57](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L57)、[prefabs/armor_skeleton.lua:123](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L123)、[prefabs/armor_skeleton.lua:133](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L133

```lua
AddPrefabPostInit("armor_skeleton", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local cooldown = inst.components.cooldown
    if cooldown ~= nil then
        cooldown.cooldown_duration = 10
    end
end)
```

十秒仅用于随后省参的 StartCharging；不立即改已排任务／读档剩余值，不覆盖显式首充时间。停用后工厂配置恢复，但已存剩余时间仍按 OnLoad 恢复；无额外资源／任务。[components/cooldown.lua:31](../../../raw/current/scripts/components/cooldown.lua#L31)、[components/cooldown.lua:87](../../../raw/current/scripts/components/cooldown.lua#L87)、[prefabs/armor_skeleton.lua:145](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L145

## 常见陷阱与限制

- **初始 false 就会自动恢复**：构造未排任务，需工厂明确初始化或启动；FinishCharging 也只处理已有 deadline。[components/cooldown.lua:1](../../../raw/current/scripts/components/cooldown.lua#L1)、[components/cooldown.lua:46](../../../raw/current/scripts/components/cooldown.lua#L46
- **改 duration 自动重排当前任务**：StartCharging 才写新 deadline；只有明确需要重置才再启动，别悄悄覆盖存档恢复。[components/cooldown.lua:31](../../../raw/current/scripts/components/cooldown.lua#L31
- **完成回调仅自然计时触发一次**：FinishCharging、LongUpdate、OnLoad 已就绪状态都可进入完成逻辑；回调不是独一无二的奖励凭据。[components/cooldown.lua:17](../../../raw/current/scripts/components/cooldown.lua#L17)、[components/cooldown.lua:75](../../../raw/current/scripts/components/cooldown.lua#L75)、[components/cooldown.lua:87](../../../raw/current/scripts/components/cooldown.lua#L87
- **移除组件保留计时**：OnRemoveFromEntity 取消 task；配置／能力撤销另由宿主负责，不自动恢复骨甲抗性。[components/cooldown.lua:10](../../../raw/current/scripts/components/cooldown.lua#L10

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；cooldown.lua 与 armor_skeleton.lua 均不在 25643519→26148036 变更清单，另一宿主 resurrectionstone.lua 未变；已核关键消费者：本轮 rechargeable.lua 改动（LongUpdate 别名、净变量死代码清理）不触及本组件，tuning 新增 COOLDOWN 键均为新 boss 专属；未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整 cooldown、骨甲工厂／受击／穿脱／补料消费点。未审校所有其他 cooldown 宿主、抗性内部和网络显示。
- 结构检查：发布时核对元数据、大小、链接、源码锚点及登记；完整示例作 Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；格挡恢复、燃料耗尽、换装、离屏及读档剩余时间待测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/timer]]；[[reference/components/rechargeable]]；[[reference/components/spellbookcooldowns]]。
