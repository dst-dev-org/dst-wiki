---
title: "epicscare"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "筛选周围实体并发送 epicscare 事件，由目标的脑行为决定恐慌、失去追随或其他反应。"
aliases: ["epicscare", "群体恐吓", "Boss恐惧", "恐慌范围", "吓跑随从"]
tags: [component, event, ai, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/epicscare.lua", "prefabs/beequeen.lua", "stategraphs/SGbeequeen.lua", "brains/braincommon.lua", "brains/pigbrain.lua", "behaviours/panic.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# epicscare

## 快速判断

- 职责与适用：以宿主世界坐标为中心查找实体，向合格目标发送 `epicscare`；组件本身不强制进入恐惧状态。[完整实现](../../../raw/current/scripts/components/epicscare.lua#L1-L32)
- 优先使用：已有恐惧接收脑的生物周围范围恐吓；猪人脑确实使用 `BrainCommon.PanicWhenScared`，不能把任意 combat／locomotor 实体当作必定逃跑。[猪人接入](../../../raw/current/scripts/brains/pigbrain.lua#L382-L388)、[接收端](../../../raw/current/scripts/brains/braincommon.lua#L126-L179)
- 重要副作用：通用接收节点在恐惧开始／结束切换时清空 combat.target；传入丢忠诚概率时，每三秒尝试使追随者脱离领袖。[接收分支](../../../raw/current/scripts/brains/braincommon.lua#L139-L169)
- 修改边界：遵循原版服务端挂载；蜂后 prefab 配置组件，由服务端 SG 在时间轴显式调用，不是自动光环。[服务端门禁](../../../raw/current/scripts/prefabs/beequeen.lua#L404-L408)、[挂载与SG](../../../raw/current/scripts/prefabs/beequeen.lua#L455-L469)、[时间轴](../../../raw/current/scripts/stategraphs/SGbeequeen.lua#L432-L448)

## 关键接口

| 接口／字段 | 签名或默认值 | 条件、作用与证据 |
|---|---|---|
| `SetRange(range)` | 数值，默认 15 世界距离单位 | 直接替换查询半径，不验证正负；宿主可另行覆盖。[实现](../../../raw/current/scripts/components/epicscare.lua#L1-L12) |
| `SetDefaultDuration(duration)` | 数值秒，默认 5 | 仅在 Scare 未提供 duration 时使用。[实现](../../../raw/current/scripts/components/epicscare.lua#L14-L30) |
| `Scare(duration?, fn?)` | 秒，无返回值 | 用 `duration or defaultduration` 发送事件；显式 0 不回退。发送事件后若传入 `fn`，逐目标调用 `fn(scarer, victim, self, duration)`。跳过自己、不可见与存在 health 且已死的实体；不要求目标有 health。[实现](../../../raw/current/scripts/components/epicscare.lua#L18-L30) |
| 三组查询字段 | `scaremusttags=nil`；排除 `{ "epic", "INLIMBO" }`；任一 `{ "_combat", "locomotor" }` | 直接传给 FindEntities；无按阵营或领袖排除，也未默认排除 player／notarget。改字段影响下一次扫描。[筛选与发送](../../../raw/current/scripts/components/epicscare.lua#L1-L30) |
| 事件载荷 | `{ scarer=宿主, duration=秒 }` | 在**目标实体** PushEvent；通用接收端只用 duration，把结束时刻延至 `max(旧值, 当前时间+duration+math.random())`，不按来源独立计时。[发送](../../../raw/current/scripts/components/epicscare.lua#L18-L30)、[监听](../../../raw/current/scripts/brains/braincommon.lua#L126-L131) |

组件无周期任务、监听或保存／加载／移除钩子；配置需在宿主构造时重建。恐惧期限属于目标脑的局部变量，`PanicWhenScared` 节点停止时移除监听；恐惧期间 `Panic` 随机方向跑动，并非必然远离 scarer。[组件全文](../../../raw/current/scripts/components/epicscare.lua#L1-L32)、[期限与清理](../../../raw/current/scripts/brains/braincommon.lua#L156-L179)、[跑动实现](../../../raw/current/scripts/behaviours/panic.lua#L7-L22)

（26148036 核验：`Scare` 新增可选第二参 `fn`，为逐目标回调 `fn(scarer, victim, self, duration)`，在事件发送后调用；"epicscare" 事件载荷与旧签名不变，已有调用兼容。）

## 典型接入

### 蜂后原有尖叫扩大五单位范围

完整目标文件为模组 `modmain.lua`，普通生存模式；无需资源。使用现有蜂后组件，只改扫描半径，保留原 SG 的尖叫、警戒士兵与触发时间。[prefab注册](../../../raw/current/scripts/prefabs/beequeen.lua#L503)、[post-init注册](../../../raw/current/scripts/modutil.lua#L592-L599)、[原版配置](../../../raw/current/scripts/prefabs/beequeen.lua#L455-L469)

```lua
AddPrefabPostInit("beequeen", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local scare = inst.components.epicscare
    if scare ~= nil then
        scare:SetRange(scare.range + 5)
    end
end)
```

范围增加在该实例之后每次 Scare 时生效，读档重建重新应用；组件不产生待清理任务。撤销后重新载入世界恢复原版配置，已发送事件不会被撤回。若运行中恢复半径，也只影响之后的扫描，目标既有恐惧期限继续由目标脑处理。[瞬时发送](../../../raw/current/scripts/components/epicscare.lua#L18-L30)、[接收期限](../../../raw/current/scripts/brains/braincommon.lua#L126-L131)

蜂后 `DoScreechAlert` 显式 `Scare(5)`，因此只改 `SetDefaultDuration` 不会改变这条链的持续参数；它还同时警戒士兵。若要改此参数，需保留该回调的其他行为，不应替换整个 SG。[原版调用](../../../raw/current/scripts/stategraphs/SGbeequeen.lua#L30-L33)

## 常见陷阱与限制

- **查到目标就必定恐惧**：默认过滤使 epic／INLIMBO 不进入发送；不可见／已死实体也跳过。通过筛选仍只发事件，目标必须有生效中的接收行为；此组件没有统一的其他恐惧免疫判定。[发送门禁](../../../raw/current/scripts/components/epicscare.lua#L1-L30)、[实际猪人接收](../../../raw/current/scripts/brains/pigbrain.lua#L382-L388)
- **指定五秒就精确五秒结束**：通用脑会添加随机延时，并与旧结束时间取最大值；重复发送可延长，不能缩短既有期限，节点执行还受脑调度影响。[接收](../../../raw/current/scripts/brains/braincommon.lua#L126-L169)
- **移除发射组件会清除目标恐惧**：目标独立保存局部结束时间，发射端无撤销接口；不应通过停止整个目标脑强行清除。[两端实现](../../../raw/current/scripts/components/epicscare.lua#L18-L32)、[目标清理](../../../raw/current/scripts/brains/braincommon.lua#L156-L179)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 24 处、结论修订 2 处（Scare 可选回调，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；完整组件、默认排除／存活可见门禁、蜂后服务端挂载与尖叫链、猪人通用脑监听／目标切换／丢忠诚／停止清理及 Panic 跑动已核验；未逐物种审校接收差异与其他免疫规则。
- 结构检查：本页元数据、大小、本地链接和源码行锚点通过；完整 `modmain.lua` 经 Lua 5.1 语法解析通过。
- 游戏运行：未运行；边缘距离、脑执行优先级、实际恐慌时长与多模组协作待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。