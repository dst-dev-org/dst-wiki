---
title: "spell"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "在独立效果载体上维护目标、持续时间与回调，参考光莓发光的开始、续期、结束和存读恢复。"
aliases: ["持续效果计时", "光莓发光续期", "wormlight_light", "效果载体存档", "spell"]
tags: [component, callback, lifecycle, item, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/spell.lua", "prefabs/wormlight.lua", "prefabs/player_common.lua", "prefabs/critters.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# spell

## 快速判断

- 职责与适用：挂在**效果载体**，保存目标并按时调用效果／结束回调；原版宿主是 wormlight_light 等非联网代理，跟随目标并持有独立发光 FX。它不提供物品施法动作，右键施法另见 [[reference/components/spellcaster]]。[组件](../../../raw/current/scripts/components/spell.lua#L1-L21) [宿主](../../../raw/current/scripts/prefabs/wormlight.lua#L298-L323) [注册名](../../../raw/current/scripts/prefabs/wormlight.lua#L428-L430)
- 优先使用：先配置回调、duration 等，再 SetTarget、检查载体仍有效且 target 非 nil 后 StartSpell；续期由宿主明确重置 lifetime 并 ResumeSpell，不能只调用 StartSpell 期望计时归零。[首次调用](../../../raw/current/scripts/prefabs/wormlight.lua#L30-L47) [启动实现](../../../raw/current/scripts/components/spell.lua#L114-L126)
- 重要副作用：SetTarget 立即调用 ontargetfn，回调可直接删除载体；OnFinish 先回调再停更新，removeonfinish=true 才删实体，**不清 active／target／计时**。[目标设置](../../../raw/current/scripts/components/spell.lua#L139-L143) [目标拒绝](../../../raw/current/scripts/prefabs/wormlight.lua#L246-L249) [结束](../../../raw/current/scripts/components/spell.lua#L31-L39)
- 修改边界：组件无网络／端检查；原版由服务端食用／目标存读链创建非联网效果代理，视觉另由 fx 协作。回调、variables 和配置不由组件保存；目标引用保存排除 player，玩家有专门的恢复链。[载体](../../../raw/current/scripts/prefabs/wormlight.lua#L298-L323) [保存](../../../raw/current/scripts/components/spell.lua#L75-L94) [玩家特例](../../../raw/current/scripts/prefabs/player_common.lua#L1299-L1308)

## 关键接口

| 接口／字段 | 参数、时序与证据 |
|---|---|
| `SetTarget(target)` | 要求非 nil 且支持 HasTag；仅当目标**没有** spellname 标签才赋 self.target 并调用 OnTarget。拒绝时不清旧 target、不返回成功值，不自动添加标签／解绑旧目标。[实现](../../../raw/current/scripts/components/spell.lua#L139-L143) |
| `spellname`／`target` | 默认 "spell"／nil；spellname 仅作 SetTarget 的标签门禁，并非全局注册／去重表。光莓另外通过 target.wormlight 记录载体，负责同类续期和不同类替换。[默认](../../../raw/current/scripts/components/spell.lua#L3-L13) [宿主策略](../../../raw/current/scripts/prefabs/wormlight.lua#L30-L47) [目标记录](../../../raw/current/scripts/prefabs/wormlight.lua#L256-L260) |
| `StartSpell()`／`OnStart()` | 无目标则 StartSpell 直接返回；有目标先 StartUpdatingComponent，再 OnStart 设置 active=true 并调 onstartfn(inst)。不重置 lifetime／timer，也不做已有 active 门禁。[实现](../../../raw/current/scripts/components/spell.lua#L24-L27) [启动](../../../raw/current/scripts/components/spell.lua#L114-L119) |
| `ResumeSpell()` | **只有 resumefn 非 nil 才恢复更新**：先 resumefn(inst, duration-lifetime)，再 StartUpdatingComponent；不查 target、不设置 active、不重置计时、不调用 onstartfn。[实现](../../../raw/current/scripts/components/spell.lua#L122-L126) |
| `duration`／`lifetime` | 默认 3／0，按 OnUpdate(dt) 累加；以秒计。到期检查发生在本轮 fn 之后，duration 到期的那帧仍可能执行效果。[默认](../../../raw/current/scripts/components/spell.lua#L13-L16) [更新](../../../raw/current/scripts/components/spell.lua#L43-L65) |
| `period`／`timer`／`fn(inst, target)` | 默认 nil／nil；首轮 timer=nil 立即调 fn，再设 timer=period。之后每轮扣 dt，<=0 触发一次并重设；period=nil 每次更新都触发，不按漏掉的周期次数补调用。[更新](../../../raw/current/scripts/components/spell.lua#L43-L60) |
| `OnFinish()`／`removeonfinish` | 无完成门禁，每次调用都可能重复执行 onfinishfn(inst)；之后停止更新，按 removeonfinish 删除载体。默认 false，active 仍可能是 true，不能作为是否仍在更新的可靠查询。[默认](../../../raw/current/scripts/components/spell.lua#L17) [结束实现](../../../raw/current/scripts/components/spell.lua#L31-L39) |
| `SetVariables(table)` | 直接保留表引用；非表打印提示后返回。variables 不参与存档，也不会自动传给回调，回调可通过 inst.components.spell.variables 读取。[约定](../../../raw/current/scripts/components/spell.lua#L19-L21) [设置](../../../raw/current/scripts/components/spell.lua#L130-L136) |

### 计时、保存与恢复的边界

OnUpdate 不检查 active／target 有效性；宿主必须安排开始、目标失效处理与停止。光莓 ontargetfn 遇到无效状态立即 Remove，否则跟随目标，并监听目标移除、人物变鬼等事件来 OnFinish；组件自己不监听这些事件。[逐帧函数](../../../raw/current/scripts/components/spell.lua#L43-L65) [实际目标协作](../../../raw/current/scripts/prefabs/wormlight.lua#L246-L271)

OnSave 只返回 lifetime／timer／active，以及非 player 目标的 GUID 和引用列表；duration／period／spellname、variables 和回调都要由工厂重新配置。OnLoad 只回填三个计时／状态字段；LoadPostPass 尝试用 newents 找目标并 SetTarget，载体未被删除且 active 时 ResumeSpell。没有 resumefn 就不会恢复更新。[保存／回填](../../../raw/current/scripts/components/spell.lua#L75-L94) [后置恢复](../../../raw/current/scripts/components/spell.lua#L98-L110) [恢复条件](../../../raw/current/scripts/components/spell.lua#L122-L126)

## 典型接入

### 参考光莓创建持续发光效果

这是原版效果载体协作指引，不是完整模组 demo；只装 spell 不会创建灯光、跟随或联网状态。

1. **工厂／资源**：prefabs/wormlight.lua 的 light_commonfn 创建 Transform／Follower 的隐藏非联网实体，安装 spell，设置 spellname、duration、目标／开始／结束／恢复回调并启用 removeonfinish；再 SpawnPrefab(fxprefab)。三种 light 工厂共用此结构，实际效果资源在独立 FX 上。[组装](../../../raw/current/scripts/prefabs/wormlight.lua#L298-L337) [注册](../../../raw/current/scripts/prefabs/wormlight.lua#L428-L433)
2. **目标／生效**：light_ontarget 拒绝 playerghost／overcharge，绑定 target.wormlight、Follower 和清理事件；对人物／宠物关闭载体自身持久化。SetTarget 后调用者检查 IsValid 和 spell.target，确认接入成立再 StartSpell。[目标接入](../../../raw/current/scripts/prefabs/wormlight.lua#L246-L275) [调用者判断](../../../raw/current/scripts/prefabs/wormlight.lua#L41-L47)
3. **续期／替换**：create_light 遇到相同 light prefab 把 lifetime=0，再 ResumeSpell；不同 prefab 则先令旧载体 OnFinish，再生成新载体。ResumeSpell 的回调只按剩余时间更新 FX 进度，本身不重置 lifetime；“续期”是调用者的策略。[替换链](../../../raw/current/scripts/prefabs/wormlight.lua#L30-L47) [开始／恢复视觉](../../../raw/current/scripts/prefabs/wormlight.lua#L166-L171)
4. **结束／撤销**：light_onfinish 清 target.wormlight 和 bloom，包括骑手 bloom；removeonfinish 随后删载体，宿主 OnRemoveEntity 再移除 fx。复制本模式时保留这种效果清理分工，不能把 OnFinish 简化为只停计时。[完成清理](../../../raw/current/scripts/prefabs/wormlight.lua#L278-L295) [工厂清理挂接](../../../raw/current/scripts/prefabs/wormlight.lua#L314-L320)

### 玩家与宠物随目标存读恢复

网络玩家不走 spell 的 GUID 引用恢复：player_common 保存 inst.wormlight:GetSaveRecord，读档 SpawnSaveRecord 后显式 SetTarget(player)，再次检查载体／目标并 ResumeSpell。宠物 critters 同样保存完整载体记录并显式绑定自己。要复用到其他目标，先选清目标引用与载体保存责任，不让两侧各保存一份重复效果。[玩家保存](../../../raw/current/scripts/prefabs/player_common.lua#L1233-L1235) [玩家恢复](../../../raw/current/scripts/prefabs/player_common.lua#L1299-L1308) [宠物存读](../../../raw/current/scripts/prefabs/critters.lua#L76-L91)

## 常见陷阱与限制

- **ResumeSpell 没有 resumefn 仍期待续跑**：函数把 StartUpdatingComponent 放在回调存在分支；为需要恢复的自定义载体配置恢复回调，并明确计时／目标由谁恢复，不能靠 active=true 代替。[恢复实现](../../../raw/current/scripts/components/spell.lua#L122-L126)
- **重新 StartSpell 就算重新施加**：lifetime／timer／旧目标效果均不重置，onstartfn 又可能重复执行；按宿主生命周期安排续期或先清理重建。光莓只在首次 Start、同种续期走 lifetime=0 与 Resume。[启动](../../../raw/current/scripts/components/spell.lua#L114-L119) [真实策略](../../../raw/current/scripts/prefabs/wormlight.lua#L30-L47)
- **目标带 spellname 标签还用旧 target 开始**：SetTarget 拒绝时保留旧值；该门禁不代表通用可施法检测，也不保证目标存活。新建载体仿原版检查目标是否接入，复用载体另行处理旧关系。[设置](../../../raw/current/scripts/components/spell.lua#L139-L143) [原版检查](../../../raw/current/scripts/prefabs/wormlight.lua#L41-L47)
- **只 StopUpdatingComponent／RemoveComponent 就撤销效果**：它们不会调用 Spell:OnFinish 或宿主 onfinishfn；需要撤销 target.wormlight、bloom 等状态时走已设计的结束链。OnFinish 本身也非幂等，宿主应防重复清理；组件没有 OnRemoveFromEntity。[完整组件](../../../raw/current/scripts/components/spell.lua#L1-L146) [结束](../../../raw/current/scripts/components/spell.lua#L31-L39) [原版清理](../../../raw/current/scripts/prefabs/wormlight.lua#L278-L295)
- **存档中 active=true 就认为仍生效**：OnFinish 不清 active，载体若保留且保存，后置恢复仍可能 Resume；结束后保留实体的自定义实现需同时设计状态与存读策略。默认 removeonfinish=false 不能直接照搬光莓 true 的行为假设。[结束](../../../raw/current/scripts/components/spell.lua#L31-L39) [保存](../../../raw/current/scripts/components/spell.lua#L75-L84) [后置启动](../../../raw/current/scripts/components/spell.lua#L109-L110)

## 验证记录

- 增量复查：2026-10-25，Steam buildid 26148036；spell.lua／wormlight／critters 均无变化，player_common 两锚点位于未变段（wormlight 存读链），无需重定位；nomagiccast 门控（R-01/R-04）作用于 spellcaster/动作层，不涉本组件，施法入口结论以 spellcaster 页为准，本页结论维持。
- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整 spell、wormlight 三种效果载体注册／工厂、目标绑定／续期／清理、玩家与宠物的光莓记录存读。未全面审校独立 FX 的网络更新、所有食物／技能施加入口或任意新载体的恢复安全性。
- 结构检查：2026-10-24，本页格式、来源、本地链接及源码锚点严格检查通过，本批导航与全库登记检查通过；没有新增完整 Lua 示例。
- 游戏运行：未运行；跨存读续期、目标死亡／变鬼／移除、重复结束、末帧效果与多人视觉待实测。
