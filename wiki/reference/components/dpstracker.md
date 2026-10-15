---
title: "dpstracker"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "按宿主生命事件采样短窗口净失血速率，供 Boss 威胁等级等本地服务端逻辑使用。"
aliases: ["dpstracker", "短窗口失血速率", "Boss威胁等级", "净失血DPS"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/dpstracker.lua", "components/health.lua", "prefabs/alterguardian_phase4_lunarrift.lua", "prefabs/wagboss_robot.lua", "entityscript.lua", "modutil.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# dpstracker

## 快速判断

- 职责与适用：宿主每次 `healthdelta` 后读取 `health.currenthealth`，用环形缓冲最旧／最新样本的净生命差除以时间差；单位生命点／秒。[dpstracker.lua:1](../../../raw/current/scripts/components/dpstracker.lua#L1)、[:72](../../../raw/current/scripts/components/dpstracker.lua#L72)、[:84](../../../raw/current/scripts/components/dpstracker.lua#L84)
- 优先使用：Boss 根据近期净失血调整压力；不是攻击者输出统计，没有来源归因，回血能令结果为负。
- 重要副作用：默认最多 100 个不同时间样本、窗口 2 秒；事件驱动，没有每秒更新任务，静默时 `GetDps()` 保留最后结果。[dpstracker.lua:5](../../../raw/current/scripts/components/dpstracker.lua#L5)、[:26](../../../raw/current/scripts/components/dpstracker.lua#L26)
- 修改边界：主模拟端、已有 health 的实体。原版第四阶段月亮守护者在服务端构造后添加 tracker；组件不提供 HUD 网络同步，也不保证所有扣血路径都会产生采样事件。[alterguardian_phase4_lunarrift.lua:998](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L998)、[:1046](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1046)

## 关键接口

| 接口／字段 | 签名或默认值 | 条件、作用与证据 |
|---|---|---|
| `SetOnDpsUpdateFn(fn)` | nil 或 `fn(inst, dps)` | 覆盖单个回调；每次 DoUpdate 完成都调用，包括结果为零或负值。[dpstracker.lua:22](../../../raw/current/scripts/components/dpstracker.lua#L22)、[:90](../../../raw/current/scripts/components/dpstracker.lua#L90) |
| `GetDps()` | 返回当前缓存数值，初始 0 | 纯读取，不采样、不清过期样本。[dpstracker.lua:26](../../../raw/current/scripts/components/dpstracker.lua#L26 |
| `DoUpdate()` | 无参数、无返回值 | 使用 `GetTime()` 和当前生命更新环形缓冲、计算结果并回调；需 health 仍存在。[dpstracker.lua:30](../../../raw/current/scripts/components/dpstracker.lua#L30 |
| `max_size`／`max_window` | 默认 100／2 秒 | 裸字段，无校验 setter；不要在已有缓冲上随意缩小容量，环绕算法依赖一致的索引与容量。[dpstracker.lua:10](../../../raw/current/scripts/components/dpstracker.lua#L10)、[:40](../../../raw/current/scripts/components/dpstracker.lua#L40) |
| `tbl`／`i0`／`sz`／`dps` | 样本表／首索引／数量／结果 | 内部缓存；每样本为 hp 与 t，不记录单击 amount、attacker 或伤害类型。[dpstracker.lua:7](../../../raw/current/scripts/components/dpstracker.lua#L7)、[:72](../../../raw/current/scripts/components/dpstracker.lua#L72) |
| `OnRemoveFromEntity()` | 删除钩子 | 解绑 healthdelta；不再执行更新回调。[dpstracker.lua:18](../../../raw/current/scripts/components/dpstracker.lua#L18)、[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670 |

同一个 `GetTime()` 值的重复更新覆盖最后样本；满容量时覆盖最旧样本。每次写入后剔除与最新样本相差**严格大于** 2 秒的旧样本，至少保留当前一个。计算为 `(最旧hp - 最新hp) / dt`；dt 为零时用 max_window 作分母，此时仅一个样本，结果为零。若事件极密，100 个样本也可能不足覆盖两秒。[dpstracker.lua:40](../../../raw/current/scripts/components/dpstracker.lua#L40)、[:53](../../../raw/current/scripts/components/dpstracker.lua#L53)、[:75](../../../raw/current/scripts/components/dpstracker.lua#L75)

## 典型接入

### 调整第四阶段月亮守护者的威胁敏感度

以下为已启用服务器模组的完整 `modmain.lua` 内容；工厂后接入现有 `alterguardian_phase4_lunarrift`，沿用原版资源。把原威胁回调收到的速率乘 1.25，让正向净失血更早达到同一威胁门槛；保留原映射、上限和降级流程。[modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)、[mainfunctions.lua:387](../../../raw/current/scripts/mainfunctions.lua#L387)、[alterguardian_phase4_lunarrift.lua:1046](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1046

```lua
AddPrefabPostInit("alterguardian_phase4_lunarrift", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local tracker = inst.components.dpstracker
    if tracker == nil or tracker.ondpsupdatefn == nil then return end
    local oldfn = tracker.ondpsupdatefn
    tracker:SetOnDpsUpdateFn(function(actor, dps)
        oldfn(actor, dps * 1.25)
    end)
end)
```

新建／读档重建实例后，生命事件采样进入包装回调；1.25 是无量纲倍率，不提高实际伤害，也不修改 GetDps 的原始缓存。首次孤立样本仍为零，静默时不新增更新。停用模组并重载恢复原回调；运行中撤销包装须恢复捕获的 oldfn，不应直接清空原版回调。未新增资源、轮询或客户端 UI，示例未在游戏运行。

原版回调把速率 150～375 映射到威胁等级，提升／维持等级后由**宿主**独立 30 秒任务逐级降级，最终改变攻击间隔；组件本身不会在无伤害时归零或降级。[alterguardian_phase4_lunarrift.lua:530](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L530 机器人同样使用此组件，但在 `SetCombatEnabled` 内动态添加，停战时先取消威胁任务，再移除 tracker 和 health；只在 Prefab postinit 查它可能尚不存在。[wagboss_robot.lua:1126](../../../raw/current/scripts/prefabs/wagboss_robot.lua#L1126)、[:1178](../../../raw/current/scripts/prefabs/wagboss_robot.lua#L1178)

## 常见陷阱与限制

- **第一击数值等于伤害／两秒**：空缓冲第一次只记录变化后的生命，结果为 0，首击前生命没有被采样；有前置样本时才可计入两样本之间的净差。同时间的多次变化只保留该时刻最终生命。[dpstracker.lua:40](../../../raw/current/scripts/components/dpstracker.lua#L40)、[:84](../../../raw/current/scripts/components/dpstracker.lua#L84)
- **停手两秒后读取会自动归零**：GetDps 只取缓存，直到下一次 DoUpdate 才淘汰旧样本；如需实时空窗显示，应由自有展示／采样机制明确处理。[dpstracker.lua:26](../../../raw/current/scripts/components/dpstracker.lua#L26)、[:75](../../../raw/current/scripts/components/dpstracker.lua#L75)
- **当作累计伤害或 amount 速率**：算法忽略事件 data，使用封顶后的当前生命，回血抵扣、净回血可负；直接 SetCurrentHealth 不触发事件，后续事件采样仍可能把期间生命变化反映到差值。事件监听顺序和嵌套改血也可能影响读取时生命。[dpstracker.lua:1](../../../raw/current/scripts/components/dpstracker.lua#L1)、[health.lua:499](../../../raw/current/scripts/components/health.lua#L499)、[:642](../../../raw/current/scripts/components/health.lua#L642)
- **缓冲跨存档延续**：无保存／加载钩子，构造重新创建空缓冲；宿主保存威胁或阶段不代表保存历史样本。机器人显式保存 threat，第四阶段守护者保存 engaged 并恢复阶段。[entityscript.lua:1928](../../../raw/current/scripts/entityscript.lua#L1928)、[wagboss_robot.lua:1300](../../../raw/current/scripts/prefabs/wagboss_robot.lua#L1300)、[alterguardian_phase4_lunarrift.lua:747](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L747

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 4 处（mainfunctions PostInit +26、wagboss_robot SetCombatEnabled／威胁任务／OnSave +7、entityscript GetPersistData +3）；无结论修订，仅锚点迁移——dpstracker.lua、health.lua 未变；alterguardian_phase4_lunarrift 本版仅改 CalcSanityAura 数值（820-826），998/1046/530/747 行内容未变。依据：全局枢纽 diff 审阅＋变更清单命中 alterguardian_phase4_lunarrift.lua、wagboss_robot.lua、entityscript.lua、mainfunctions.lua。
- 源码审阅：2026-10-21，Steam buildid 25643519；完整审阅组件及环形缓冲、health 事件／读档，实体保存与组件删除；核验两个 Boss 的威胁消费／降级、服务端构造、第四阶段宿主读档、机器人动态添加移除与保存边界。
- 游戏运行：未运行；首样本、同帧覆盖、两秒空窗、净回血、满容量、宿主威胁降级和示例倍率效果仍待实测；不推广为全部 Boss／战斗统计实现。
- 结构检查：页面元数据、大小、链接和源码行锚点及本页完整 Lua 示例语法检查通过；不代表游戏效果验证。

相关：[[reference/components/health]]；[[reference/components/healthtrigger]]；[[reference/components/damagetracker]]。
