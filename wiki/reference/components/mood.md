---
title: "mood"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "控制群体发情的天数循环、季节校准与进入／退出回调，区分成员表现、实际繁殖和保存归一化。"
aliases: ["延长牛群非发情等待", "牛群发情周期", "春季发情", "mood"]
tags: [component, creature, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/mood.lua", "prefabs/beefaloherd.lua", "prefabs/beefalo.lua", "prefabs/penguinherd.lua", "prefabs/penguin.lua", "tuning.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# mood

## 快速判断

- 职责与适用：宿主侧管理 isinmood、以游戏天数计的切换计数、季节列表与进入／退出回调；原牛的 mood 在 beefaloherd 上，成员通过群体查询，不是每只牛自带该组件。[组件构造](../../../raw/current/scripts/components/mood.lua#L8-L27) [牛群安装](../../../raw/current/scripts/prefabs/beefaloherd.lua#L125-L139) [成员读取](../../../raw/current/scripts/prefabs/beefalo.lua#L246-L255)
- 优先使用：首次配置 SetMoodTimeInDays，再设回调／季节；状态切换用 SetIsInMood，当前季节校准用 ValidateMood。只改后续等待见下文，避免重配方法直接重置当前状态。
- 重要副作用：真正切换时执行宿主回调；原牛群启停 periodicspawner 并向成员发 entermood／leavemood，成员另改标签／外观。单独改 isinmood 会绕开协作。[切换](../../../raw/current/scripts/components/mood.lua#L114-L143) [牛群回调](../../../raw/current/scripts/prefabs/beefaloherd.lua#L28-L55) [成员消费者](../../../raw/current/scripts/prefabs/beefalo.lua#L292-L310)
- 修改边界：enabled 与 worldsettingsenabled 分工不同；Enable(false) 退出状态但不暂停 cycles 扣天，进入状态仍受两个开关限制。发情只是繁殖条件之一，亲本、群体容量、密度和离屏另查 [[reference/components/herd]]／[[reference/components/periodicspawner]]。[扣天](../../../raw/current/scripts/components/mood.lua#L1-L5) [开关](../../../raw/current/scripts/components/mood.lua#L33-L49) [进入门禁](../../../raw/current/scripts/components/mood.lua#L114-L128)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetMoodTimeInDays(length, wait, forcemood, in_mult, out_mult, world_enabled)` | 游戏天数／boolean／保存倍率 | 赋 length／wait、把 daystomoodchange 重设为 wait 并直接写 isinmood=false；不调用退出回调，不清全部季节状态。两个倍率缺省 1，world_enabled 只有显式 false 禁用。首次配置后再 Check／保存，不能用 nil 计数当暂停。[实现](../../../raw/current/scripts/components/mood.lua#L38-L49) [检查](../../../raw/current/scripts/components/mood.lua#L85-L89) [保存](../../../raw/current/scripts/components/mood.lua#L149-L152) |
| `moodtimeindays.length/wait`／`daystomoodchange` | 当前参数／当前剩余天数 | length 用于普通进入，wait 用于普通退出或退出季节模式；修改参数不自动改当前计数。cycles 每次回调在 worldsettingsenabled 且有计数时减 1，并非按秒计时或按 cycles 差值补扣。[扣减](../../../raw/current/scripts/components/mood.lua#L1-L5) [状态赋值](../../../raw/current/scripts/components/mood.lua#L99-L143) |
| `SetIsInMood(inmood, entireseason)` | 无显式返回 | 进入需两个开关开启；状态不同或 entireseason 为真时执行。普通进入设 length，整季进入设 seasonmood=true 和当前季节总长度（缺失才用 length）；不是剩余季节天数。普通退出设 wait 并调退出回调。[实现](../../../raw/current/scripts/components/mood.lua#L110-L143) |
| `SetMoodSeason(season)` | 季节标识 | 追加到数组，首次调用才监听世界 season；不是替换集合，也不立即应用当前季节，重复调用可重复条目。[实现](../../../raw/current/scripts/components/mood.lua#L77-L83) |
| `ValidateMood()` | 无参数 | 调当前季节处理：enabled 时匹配季节就 SetIsInMood(true,true)，不匹配就 ResetMood。可重复进入回调／重设季节计数，不是无副作用检查；不匹配也不保证退出普通非季节周期。[季节处理](../../../raw/current/scripts/components/mood.lua#L51-L75) |
| `ResetMood()` | 无参数 | 只在 seasonmood 为真时清标志／状态、设 wait 并调 onleavemood；不是任意状态的通用重置或无条件退出。[实现](../../../raw/current/scripts/components/mood.lua#L99-L108) |
| `Enable(enabled)`／`IsInMood()` | boolean／boolean | Enable 总会请求 SetIsInMood(false,false)，启用不会自动校准季节，原来正在发情时 Enable(true) 也可能退出；IsInMood 只返回 isinmood。[启停](../../../raw/current/scripts/components/mood.lua#L33-L36) [查询](../../../raw/current/scripts/components/mood.lua#L145-L147) |
| `CheckForMoodChange()`／`forcemood` | 无参数／候选强制保持标志 | 计数<=0 才尝试 `not IsInMood() or forcemood`；forcemood 不绕过 enabled／worldsettingsenabled，也不保证首次调用立刻进入。已在状态且普通切换目标仍真时，不会自动重置计数／重调进入回调。[检查](../../../raw/current/scripts/components/mood.lua#L85-L89) [切换条件](../../../raw/current/scripts/components/mood.lua#L114-L143) |
| `SetInMoodFn(fn)`／`SetLeaveMoodFn(fn)` | fn(host) | 替换单个回调。牛群退出回调还会 CheckForMoodChange；wait=0 时可能当场再次进入，扩展时保留原逻辑并考虑重入，不把回调当只发一次的通知。[设置](../../../raw/current/scripts/components/mood.lua#L91-L108) [牛群退出](../../../raw/current/scripts/prefabs/beefaloherd.lua#L39-L48) |

### 季节、保存与宿主顺序

普通循环到期可切换发情／等待，季节回调在匹配时按整季模式进入，离开匹配季节时只重置 seasonmood。原牛群注册春季、先 CheckForMoodChange，再排零延迟 ValidateMood 校准初始季节；不要每次查询或每帧调用 ValidateMood，否则可重复启动回调和覆盖剩余计数。[季节逻辑](../../../raw/current/scripts/components/mood.lua#L51-L89) [牛群初始化](../../../raw/current/scripts/prefabs/beefaloherd.lua#L98-L100) [配置顺序](../../../raw/current/scripts/prefabs/beefaloherd.lua#L125-L139)

OnSave 按当前状态选 in/out multiplier，用剩余天数除以该倍率存 daysleft，并保存季节数组、inmood、version=2；这些倍率是保存／恢复的归一化尺度，不是每天扣天速度。OnLoad 恢复季节列表，先将 isinmood 设为保存值的反值，再 SetIsInMood 尝试恢复并触发相应回调；v2 用当前对应倍率乘 daysleft 并 RoundBiasedUp，无 version 的旧分支用 math.min。enabled、forcemood、length／wait 和世界开关仍由工厂配置。[保存／恢复](../../../raw/current/scripts/components/mood.lua#L149-L175)

牛群后续零延迟 ValidateMood 还可再次校准季节，因此不能仅凭 OnLoad 公式承诺当前季节倒计时完全不变。配置倍率须为有效正数、计数须先初始化；不手工伪造缺字段存档，也不把移除组件当退出回调／成员清理方案。本组件没有专用卸载退出方法，宿主周期生成与成员表现需显式协作。

## 典型接入

### 延长牛群后续非发情等待，不直接重置当前计数

放目标模组 `modmain.lua`，无新资源，只处理 beefaloherd。原工厂已配置 length／wait；仅把后续使用的 wait 乘 2，保留当前 daystomoodchange、状态、季节列表、回调和保存倍率，避免再次 SetMoodTimeInDays 无回调地写 isinmood=false。[原配置](../../../raw/current/scripts/prefabs/beefaloherd.lua#L125-L131) [参数消费](../../../raw/current/scripts/components/mood.lua#L99-L143) [重配副作用](../../../raw/current/scripts/components/mood.lua#L38-L49) [Hook](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("beefaloherd", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local mood = inst.components.mood
    if mood ~= nil then
        mood.moodtimeindays.wait = mood.moodtimeindays.wait * 2
    end
end)
```

`2` 为后续等待倍率，单位是游戏天；快照默认 wait=20 时，后续普通退出或季节重置将使用 40，当前初始／已保存等待不因本行立即加倍。春季进入、forcemood 和世界禁用仍走原逻辑，原工厂的 ValidateMood 仍可能校准季节计数。本例不改普通发情 length，也不保证“所有发情间隔精确翻倍”。[默认值](../../../raw/current/scripts/tuning.lua#L879-L884) [重置](../../../raw/current/scripts/components/mood.lua#L99-L108) [切换](../../../raw/current/scripts/components/mood.lua#L114-L143)

本例不改 multiplier，原计数仍按原归一化尺度存读，不把缩放后的 wait 再当当前剩余天数。新建／重载重新应用参数，停用并重载恢复后续工厂 wait，已存剩余天数仍按原规则处理；不是对既有存档计数的回滚或迁移。[存读](../../../raw/current/scripts/components/mood.lua#L149-L175)

原版协作有两种代表：牛群进入／退出负责生育调度和成员事件，新成员登记也同步当前发情状态；企鹅群配置 wait=0、未注册季节，成员事件只将 nesting 设真／假。企鹅群虽有判空启停 periodicspawner 的回调，本工厂没有实际挂载该组件，不能据函数名推断会定时产蛋。[牛群事件与登记](../../../raw/current/scripts/prefabs/beefaloherd.lua#L28-L55) [企鹅群完整工厂](../../../raw/current/scripts/prefabs/penguinherd.lua#L37-L65) [企鹅回调](../../../raw/current/scripts/prefabs/penguinherd.lua#L6-L31) [成员消费](../../../raw/current/scripts/prefabs/penguin.lua#L176-L182)

## 常见陷阱与限制

- **给 beefalo 单独 AddComponent("mood") 就能控制发情**：实际成员从 herdmember 指向的群体取 mood，回调另管理 scarytoprey 与外观；新挂一个组件不接通这些消费者。[成员查询](../../../raw/current/scripts/prefabs/beefalo.lua#L246-L271) [表现回调](../../../raw/current/scripts/prefabs/beefalo.lua#L292-L310)
- **Enable(false) 暂停剩余天数／forcemood 绕过世界禁用**：cycles 只看 worldsettingsenabled 和计数，进入仍要求两个开关；长期禁用与暂停剩余天数是不同需求。[扣天](../../../raw/current/scripts/components/mood.lua#L1-L5) [门禁](../../../raw/current/scripts/components/mood.lua#L114-L128)
- **重配／ResetMood／ValidateMood 都是安全无副作用的统一重置**：重配直接改状态且不调退出，Reset 只处理 seasonmood，Validate 可重复整季进入；按目标选择，不让生成器与成员表现留在旧状态。[配置](../../../raw/current/scripts/components/mood.lua#L38-L49) [季节处理](../../../raw/current/scripts/components/mood.lua#L51-L83) [重置](../../../raw/current/scripts/components/mood.lua#L99-L108)
- **改 mood 就决定全部产仔／攻击行为**：回调只接通部分路径，产仔仍有亲本／容量／密度与离屏门禁；牛成员读取 mood 还影响外观和其他消费者，本页未全面审校所有战斗／骑乘行为。[产仔资格](../../../raw/current/scripts/prefabs/beefaloherd.lua#L57-L81) [生成配置](../../../raw/current/scripts/prefabs/beefaloherd.lua#L133-L139)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 2 处（tuning +2、penguin −4，区间文本逐段比对一致）；结论修订：无（BEEFALO 发情周期键值未变，不涉 WORM_BOSS 系改值键）。
- 依据：tuning 键级 hunk 地图＋变更清单命中 tuning.lua、prefabs/penguin.lua。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖组件天数／季节／开关／强制标志、配置和回调顺序、两种存档恢复分支、牛群启停／登记、成员查询／表现、企鹅群实际安装与 nesting 消费，以及 wait 参数例。未全面审校世界 cycles 跳变机制、所有旧版／损坏存档、战斗／骑乘／驯化或企鹅产蛋系统。
- 结构检查：格式、来源、链接／锚点、大小和唯一登记通过；完整 wait 调整例通过已有 Full Moon Lua 5.1 syntax-only，不证明游戏时序／产仔或外观效果。
- 游戏运行：未运行；初始春季／换季、普通循环／强制发情、禁用／重启、零等待回调重入、存读归一化／零延迟校准与多模组参数叠加待实测。
