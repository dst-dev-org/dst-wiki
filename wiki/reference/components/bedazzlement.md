---
title: "bedazzlement"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "原蜘蛛巢装饰状态、成长暂停与周期 Buff 投放；停止不立即移除蜘蛛 Buff，资源和清理由宿主协作。"
aliases: ["服务端解除蜘蛛巢装饰", "装饰蜘蛛巢停止成长", "bedazzlement"]
tags: [component, building, creature, lifecycle, callback, event]
dst_build: "steam-buildid:26230496"
sources: ["components/bedazzlement.lua", "prefabs/spiderden.lua", "prefabs/spider_buffs.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# bedazzlement

## 快速判断

- 职责与适用：目标蜘蛛巢上的装饰状态机，Start 加标签／切外观与图标、停 growable，并周期给附近蜘蛛 AddDebuff；工具端见 [[reference/components/bedazzler]]。[开始](../../../raw/current/scripts/components/bedazzlement.lua#L5-L36) [投放](../../../raw/current/scripts/components/bedazzlement.lua#L65-L80)
- 优先使用：原巢用 Start／Stop 管完整状态，不手动只删 bedazzled 标签；PacifySpiders 是一次范围投放，不检查装饰开关。[停止](../../../raw/current/scripts/components/bedazzlement.lua#L38-L63) [投放入口](../../../raw/current/scripts/components/bedazzlement.lua#L67-L80)
- 重要副作用：Stop 取消周期、还原符号和地图图标；陆地上恢复成长／地面网半径，平台上不走该恢复分支。已有蜘蛛 Buff 等自身倒计时到期，不由 Stop 逐一撤销。[停止分支](../../../raw/current/scripts/components/bedazzlement.lua#L38-L63) [Buff 倒计时](../../../raw/current/scripts/prefabs/spider_buffs.lua#L1-L27)
- 修改边界：强依赖原巢的 anims、data.stage、AnimState／SoundEmitter／MiniMapEntity／GroundCreepEntity 和 growable；不是只挂组件就能装饰任意建筑的方案。[开始依赖](../../../raw/current/scripts/components/bedazzlement.lua#L11-L27) [恢复依赖](../../../raw/current/scripts/components/bedazzlement.lua#L43-L53)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `Start()` | 已有 bedazzled 标签直接返回；否则写标签、符号、anims 动画／声音，停成长、换图标，陆地排零延迟网半径更新，再开周期。不自动做工具燃烧／冻结测试。[实现](../../../raw/current/scripts/components/bedazzlement.lua#L5-L36) |
| `Stop()` | 未装饰直接返回；装饰中去标签／符号、还原图标；陆地恢复成长和半径，清 shaving 标志，取消周期。不是恢复原成长剩余时间或撤销全部蜘蛛效果的保证。[实现](../../../raw/current/scripts/components/bedazzlement.lua#L38-L63) |
| `PacifySpiders()` | 按 growable 阶段选 BEDAZZLEMENT_RADIUS，阶段只截上界到 3；FindEntities 要 spider、排除 spiderqueen／creaturecorpse／NOCLICK，对结果 AddDebuff。不要由名字推断立即清仇恨或永久中立。[筛选](../../../raw/current/scripts/components/bedazzlement.lua#L65-L80) |
| 当前周期／范围 | BEDAZZLEMENT_RATE=1 秒，半径 {12,16,20}，Buff duration=2 秒；改全局 TUNING 可能影响全部实例，已创建周期任务不会因字段赋值自动重排。[参数](../../../raw/current/scripts/tuning.lua#L6281-L6284) [任务创建](../../../raw/current/scripts/components/bedazzlement.lua#L30-L35) |
| `OnSave`／`OnLoad` | 只存装饰布尔；恢复零延迟 Start，实际周期重新创建，不存原周期相位。组件无专用移除钩子恢复成长／外观。[存读](../../../raw/current/scripts/components/bedazzlement.lua#L82-L95) |

Buff 实际使用 `target.bedazzled` 字段，非目标巢的 bedazzled 标签；续期重排定时，首次且无 leader 时切 hit／笑脸，退出在目标有效且未死时恢复对应字段／表情。完整战斗选目标行为不在此推断；Buff 基础生命周期复用 [[reference/components/debuff]]／[[reference/components/debuffable]]。[Buff 实现](../../../raw/current/scripts/prefabs/spider_buffs.lua#L73-L118) [注册名](../../../raw/current/scripts/prefabs/spider_buffs.lua#L156-L158)

## 典型接入

### 为三种原蜘蛛巢提供服务端解除装饰事件

目标模组 `modmain.lua`，无新资源；服务端调用方对目标巢发 `den:PushEvent("my_mod_stop_den_decoration")`。只交付接收端，不提供客户端 RPC、权限、返还工具或完整建筑功能禁用。

```lua
local function AddDecorationStop(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_stop_den_decoration", function(den)
        if den.components.bedazzlement ~= nil then
            den.components.bedazzlement:Stop()
        end
    end)
end

for _, name in ipairs({ "spiderden", "spiderden_2", "spiderden_3" }) do
    AddPrefabPostInit(name, AddDecorationStop)
end
```

调用时解除当前装饰、停止后续周期，不立即回收已加 Buff，后续仍能正常再次装饰；重载按已保存装饰状态恢复，不保存接收端自身状态，停用重载移除自定义事件。Start 的地面半径更新是未保存在字段中的零延迟任务，**不保证 Start 同一帧立即 Stop 的网半径最终一致**；本例用于已稳定装饰的原巢，不用它组合无等待的开关脉冲。[延迟开始](../../../raw/current/scripts/components/bedazzlement.lua#L23-L35) [恢复](../../../raw/current/scripts/components/bedazzlement.lua#L38-L63) [工厂](../../../raw/current/scripts/prefabs/spiderden.lua#L838-L840)

原宿主还在生蜘蛛时主动投放，受击和剃降级时延迟 Stop；装饰中升级门禁另在 CanUpgrade。事件关闭不绕过这些流程，也不改 [[reference/components/shaveable]] 的产物／降级协议。[新蜘蛛](../../../raw/current/scripts/prefabs/spiderden.lua#L318-L325) [受击](../../../raw/current/scripts/prefabs/spiderden.lua#L381-L392) [剃取](../../../raw/current/scripts/prefabs/spiderden.lua#L650-L662) [升级](../../../raw/current/scripts/prefabs/spiderden.lua#L560-L566)

## 常见陷阱与限制

- **直接去标签就关装饰**：周期仍可继续，Stop 反而因无标签提前返回，留下成长／外观与任务不一致；用 Stop 管原状态。[清理门禁](../../../raw/current/scripts/components/bedazzlement.lua#L38-L62)
- **停止后附近蜘蛛立刻全部恢复**：Stop 不遍历 Buff 接收者；Buff 的持续／续期由蜘蛛侧定时与生命周期控制。[续期](../../../raw/current/scripts/prefabs/spider_buffs.lua#L1-L27)
- **PacifySpiders 只在已装饰时有效**：方法本身无此门禁，独立调用方须定义条件，不能把“函数可调用”当“正常玩法可达”。[独立投放](../../../raw/current/scripts/components/bedazzlement.lua#L67-L80)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 2 处（tuning BEDAZZLEMENT 参数段 +14、spiderden 注册表 -1）；无结论修订，仅锚点迁移——BEDAZZLEMENT_RATE／RADIUS／DURATION 值未变（tuning-hunk-map 5 改值键不含本组键），spiderden 本版仅删 batbat 吸取用 lifedrainable 标签行（旧 711），不涉装饰流程。依据：tuning-hunk-map 键级审阅＋变更清单命中 tuning.lua、prefabs/spiderden.lua。
- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、原巢工厂／安装／生蜘蛛／受击／剃取／升级消费者，Buff 实际注册、续期／退出与当前参数。未全面审校全部蜘蛛战斗／Brain、平台成长政策、宿主删除和多模组同帧时序。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整三巢停止事件例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；周期／Buff 到期、陆地／平台、剃取／受击、Start 同帧 Stop、存读和多模组重排待实测。
