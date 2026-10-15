---
title: "witherable"
created: "2026-10-18"
updated: "2026-10-28"
type: component
summary: "管理草丛等植物的高温枯萎、复壮与定时保护，区分自然检查、强制调用和贫瘠／再生。"
aliases: ["采草后防枯萎一天", "高温枯萎", "植物复壮", "witherable"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/witherable.lua", "prefabs/grass.lua", "components/pickable.lua", "components/wateryprotection.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# witherable

## 快速判断

- 职责与适用：控制宿主高温枯萎及条件恢复，实际对 crop 或 pickable 操作；不是现代农作物的水分／快乐压力。[枯萎处理](../../../raw/current/scripts/components/witherable.lua#L51-L89) [复壮资格](../../../raw/current/scripts/components/witherable.lua#L191-L197) 现代作物见 [[reference/components/farmplantstress]]。
- 优先使用：临时保护用 `Protect(seconds)`，暂停自然检查用 `Enable(false)`；需要立即复壮时先核验 CanRejuvenate，不直接改 withered 标签。[保护](../../../raw/current/scripts/components/witherable.lua#L248-L259) [开关](../../../raw/current/scripts/components/witherable.lua#L171-L197)
- 重要副作用：枯萎会记录 pickable 原循环数并 MakeBarren；保护可强制复壮后延迟枯萎，但不使植物立即可采收，也不修复普通缺肥贫瘠。[枯萎](../../../raw/current/scripts/components/witherable.lua#L59-L69) [复壮](../../../raw/current/scripts/components/witherable.lua#L102-L133)
- 修改边界：本页服务端。ForceWither 不检查 enabled 或 Protect 状态；定时保护不是所有强制操作的免疫开关，crop 也不通过本组件复壮。[强制接口](../../../raw/current/scripts/components/witherable.lua#L191-L215)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `withered`／`IsWithered()` | boolean，默认 false | 属性回调维护 withered 标签；单改字段不执行 crop／pickable 枯萎／恢复流程。[属性](../../../raw/current/scripts/components/witherable.lua#L1-L7) [初值](../../../raw/current/scripts/components/witherable.lua#L13-L36) |
| `wither_temp`／`rejuvenate_temp` | 构造分别随机取 TUNING 阈值 | 自然枯萎要求局部温度 > wither_temp 且未按本组件规则淋雨；复壮要求温度 < rejuvenate_temp 或淋雨。不是同一个比较阈值。[构造](../../../raw/current/scripts/components/witherable.lua#L19-L22) [枯萎判定](../../../raw/current/scripts/components/witherable.lua#L71-L89) [复壮判定](../../../raw/current/scripts/components/witherable.lua#L119-L133) |
| `Enable(enable)` | false 停用；其他值启用 | 控制自然调度，不自动复壮或清理已有保护定时器；不能把禁用直接称恢复健康。[实现](../../../raw/current/scripts/components/witherable.lua#L171-L181) |
| `CanWither()`／`CanRejuvenate()` | boolean | 前者只看尚未 withered；后者要求 withered 且没有 crop，不等于正在调度或保护已到期。[实现](../../../raw/current/scripts/components/witherable.lua#L191-L197) |
| `ForceWither()`／`ForceRejuvenate()` | 无显式返回 | 先清等待时间并 Stop，再按各自资格直接执行；ForceWither 可绕过自然温度／雨水条件及 enabled。没有 crop／pickable 的宿主无法靠组件自动获得完整表现。[实现](../../../raw/current/scripts/components/witherable.lua#L199-L215) [实际处理](../../../raw/current/scripts/components/witherable.lua#L75-L133) |
| `Protect(duration)`／`IsProtected()` | 秒数／是否有保护任务 | 仅在新截止时间更晚时更新，取消旧保护任务，ForceRejuvenate 后 DelayWither；不是累计加时，也不是每次都重新复壮。[实现](../../../raw/current/scripts/components/witherable.lua#L243-L259) [查询](../../../raw/current/scripts/components/witherable.lua#L187-L189) |
| `DelayWither(seconds)`／`DelayRejuvenate(seconds)` | 延长延迟截止时间，无显式返回 | 资格不满足则无动作；后者若正按规则淋雨不新增等待。等待复壮时监听 startrain，可提前结束延迟并重启检查。[延迟](../../../raw/current/scripts/components/witherable.lua#L145-L167) [接口](../../../raw/current/scripts/components/witherable.lua#L217-L241) |

本组件的“淋雨”判断仅为世界 israining 且宿主没有 rainimmunity，不自行解析全部遮蔽效果。自然检查通常间隔随机 30～60 秒；Start 还要求 enabled、未睡眠且无已有主任务。Stop／睡眠取消主任务并保留截止时间，醒来再 Start；保护任务不是主任务，不由 Stop 取消。[雨水](../../../raw/current/scripts/components/witherable.lua#L71-L97) [复壮间隔](../../../raw/current/scripts/components/witherable.lua#L135-L140) [启停／睡醒](../../../raw/current/scripts/components/witherable.lua#L261-L285)

存档记录枯萎、恢复循环及等待／检查／保护的剩余秒数；不保存 enabled 或随机温度阈值。OnLoad 采用已提供数据恢复，保护通过 Protect 重建；不保证跨版本或任意缺字段输入可用。移除组件会取消保护、Stop 并移除两个标签。[存读](../../../raw/current/scripts/components/witherable.lua#L287-L322) [移除](../../../raw/current/scripts/components/witherable.lua#L38-L46)

## 典型接入

### 采收普通草丛后，提供一个游戏日的自然枯萎保护

放目标模组 `modmain.lua`，无需新资源；仅包装 `grass` 的 pickable.onpickedfn，保留原声音／动画和草蜥相关行为。一天使用游戏 TOTAL_DAY_TIME 秒，不是现实 24 小时。[原回调](../../../raw/current/scripts/prefabs/grass.lua#L118-L135) [组件安装](../../../raw/current/scripts/prefabs/grass.lua#L245-L257) [草丛注册](../../../raw/current/scripts/prefabs/grass.lua#L321-L323) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("grass", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local pickable = inst.components.pickable
    if pickable == nil then return end
    local original = pickable.onpickedfn
    pickable.onpickedfn = function(plant, picker, loot)
        if original ~= nil then original(plant, picker, loot) end
        if plant:IsValid() and plant.components.witherable ~= nil then
            plant.components.witherable:Protect(GLOBAL.TUNING.TOTAL_DAY_TIME)
        end
    end
end)
```

保护从原成功采收回调时开始，重复采收取更晚截止时间，不累加整天；没有改变原产物、冬季生长或移植后肥力。耗尽循环造成的普通贫瘠不会因本例自动施肥。停用并重新初始化后不再新增保护，但存档中已有保护仍可由原组件恢复，不声称立即撤销。[采收时序](../../../raw/current/scripts/components/pickable.lua#L535-L560) [保护存读](../../../raw/current/scripts/components/witherable.lua#L299-L322)

原版浇水保护还经过 wateryprotection 的 burnable／witherable 条件，见 [[reference/components/wateryprotection#关键接口]]；采收／缺肥／再生见 [[reference/components/pickable]]。本例不直接调用 MakeBarren／MakeEmpty，不复制宿主的动画流程。

## 常见陷阱与限制

- **有 withered 标签就说明流程完整**：属性只维护标签；实际枯萎需要 crop／pickable 协作，恢复也不是简单删标签。[状态与实际处理](../../../raw/current/scripts/components/witherable.lua#L75-L89)
- **Enable(false) 阻止所有枯萎并复原现状**：只停自然调度，ForceWither 仍可执行，也不会自动恢复已有枯萎；复壮资格须单独判断。[启停](../../../raw/current/scripts/components/witherable.lua#L171-L181) [强制](../../../raw/current/scripts/components/witherable.lua#L191-L215)
- **Protect 一定把所有植物救活并满产**：旧 crop 不在 CanRejuvenate 范围；pickable 恢复涉及循环与空态，不等于立即再生。自然枯萎和现代农作压力分别维护。[资格](../../../raw/current/scripts/components/witherable.lua#L195-L197) [恢复](../../../raw/current/scripts/components/witherable.lua#L102-L128)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-18，Steam buildid 25643519；覆盖温度／雨水、枯萎／复壮资格与强制入口、保护／延迟、睡醒／移除、存读、grass 宿主及采后保护示例。未全面审校所有植物、旧 crop 或动态遮蔽消费者。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明保护时间、动画或采收效果。
- 游戏运行：未运行；热浪、下雨／rainimmunity、睡醒／存读、移植肥力、强制枯萎和其他模组回调顺序待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。