---
title: "burnable"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "配置冒烟、点燃、燃尽与熄灭，保留物品或建筑的燃烧清理并区分蔓延和供热。"
aliases: ["燃烧时间", "点燃物品", "建筑烧毁", "熄灭", "阴燃", "burnable"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/burnable.lua", "standardcomponents.lua", "prefabs/log.lua", "prefabs/treasurechest.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# burnable

## 快速判断

- 职责与适用：实体自身冒烟／点燃、火焰 FX、计时、燃尽及熄灭；火伤和热量传播归 [[reference/components/propagator]]，燃料补充归 [[reference/components/fuel]]／[[reference/components/fueled]]。[burnable.lua:352](../../../raw/current/scripts/components/burnable.lua#L352
- 优先使用：改燃尽时长用 `SetBurnTime(seconds)`；新物品可参考 MakeSmallBurnable 的 FX 与默认清理；建筑必须选结构燃尽路径，不能直接照搬物品 Remove。[standardcomponents.lua:164](../../../raw/current/scripts/standardcomponents.lua#L164
- 重要副作用：点燃会通知、启用 fueled 消耗与 propagator；熄灭清 FX／计时／相关消耗。默认物品燃尽成灰并删除，默认建筑变烧毁状态并移除多项功能。[burnable.lua:352](../../../raw/current/scripts/components/burnable.lua#L352 [standardcomponents.lua:18](../../../raw/current/scripts/standardcomponents.lua#L18 [standardcomponents.lua:52](../../../raw/current/scripts/standardcomponents.lua#L52
- 修改边界：服务端；`canlight=false` 不能挡全部直接 Ignite／StartWildfire。组件没有通用燃烧存档恢复，核对目标自己的保存逻辑。[burnable.lua:251](../../../raw/current/scripts/components/burnable.lua#L251 [burnable.lua:354](../../../raw/current/scripts/components/burnable.lua#L354 [treasurechest.lua:85](../../../raw/current/scripts/prefabs/treasurechest.lua#L85

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetBurnTime(time)` | 秒／nil | 只设 burntime；nil 不安排燃尽任务，已燃烧时改字段不重排原计时。[burnable.lua:116](../../../raw/current/scripts/components/burnable.lua#L116 [burnable.lua:399](../../../raw/current/scripts/components/burnable.lua#L399 |
| `ExtendBurning()` | 无参数 | 取消旧任务，从此刻按当前 burntime 重新安排；考虑 controlled burn 生物时长倍率。[burnable.lua:392](../../../raw/current/scripts/components/burnable.lua#L392 |
| `Ignite(immediate?, source?, doer?)` | FX 参数、来源与执行者 | 已燃烧或 fireimmune 时跳过；immediate 传给 FX，不表示立刻燃尽。先停冒烟再点燃。[burnable.lua:352](../../../raw/current/scripts/components/burnable.lua#L352 [burnable.lua:496](../../../raw/current/scripts/components/burnable.lua#L496 |
| `StartWildfire()`／`StopSmoldering(heatpct?)` | 冒烟开始／结束 | 冒烟周期结束才 Ignite；停止时清烟与任务并重置 propagator。DefaultIgniteFn 调的是 StartWildfire。[burnable.lua:212](../../../raw/current/scripts/components/burnable.lua#L212 [burnable.lua:251](../../../raw/current/scripts/components/burnable.lua#L251 [standardcomponents.lua:6](../../../raw/current/scripts/standardcomponents.lua#L6 |
| `Extinguish(resetpropagator?, heatpct?, smotherer?)` | boolean、热量比例、实体 | 停冒烟；燃烧中停止传播、计时、FX 与 fueled。reset=true 才在燃烧分支清累计热／暂停；smotherer 可消耗次数／堆叠。[burnable.lua:454](../../../raw/current/scripts/components/burnable.lua#L454 |
| `IsBurning()` | boolean 状态 | 用状态入口判断，不以 canlight 或 FX 存在替代。[burnable.lua:120](../../../raw/current/scripts/components/burnable.lua#L120 |
| `SetOnIgniteFn(fn)`／`SetOnExtinguishFn(fn)` | `(inst, source, doer)`／`(inst)` | 单槽替换；默认物品回调修改 persists，覆盖时保留原行为。[burnable.lua:81](../../../raw/current/scripts/components/burnable.lua#L81 [standardcomponents.lua:12](../../../raw/current/scripts/standardcomponents.lua#L12 |
| `SetOnBurntFn(fn)` | `(inst)` | setter 包装内部清理；不要直接赋 onburnt。原函数可能删实体／组件，包装后续先检查有效性。[burnable.lua:90](../../../raw/current/scripts/components/burnable.lua#L90 |

## 典型接入

### 将木头的燃尽基础时长设为二十秒

目标模组 `modmain.lua`；沿用 log 资源、火焰和灰烬／熄灭清理，只改本次初始化后的 burntime。原版 log 经 MakeSmallBurnable 配默认回调，传播另由 MakeSmallPropagator 配置。[log.lua:41](../../../raw/current/scripts/prefabs/log.lua#L41 [standardcomponents.lua:164](../../../raw/current/scripts/standardcomponents.lua#L164 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("log", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local burnable = inst.components.burnable
    if burnable ~= nil then
        burnable:SetBurnTime(20)
        if burnable:IsBurning() then
            burnable:ExtendBurning()
        end
    end
end)
```

未燃烧时影响后续点燃；已燃烧时重排是“从现在重新算”，不保存此前剩余时间。20 为基础秒数，特殊倍率／LongUpdate 不保证实际墙钟恰好 20 秒；本例保留原燃尽与熄灭回调，不保证跨重载继续倒计时。停用后新／重建实例恢复原初始化，燃料值不随本例改变。[burnable.lua:399](../../../raw/current/scripts/components/burnable.lua#L399 [burnable.lua:406](../../../raw/current/scripts/components/burnable.lua#L406

## 生命周期

点燃顺序：停止冒烟 → burning／监听 death／FX → 实体 `onignite` 事件 → onignite 回调 → fueled 开始消费 → propagator 开始传播 → 安排燃尽任务。监听 onignite 内停止传播会被后续 StartSpreading 覆盖；需另选生效时点或配置传播能力。[burnable.lua:369](../../../raw/current/scripts/components/burnable.lua#L369

燃尽先发实体 `onburnt`，再调 onburnt；实体仍有效时处理 explosive 与默认熄灭。熄灭回调先于 `onextinguish` 事件；移除 burnable 时 Extinguish 并取消任务／标签。[burnable.lua:291](../../../raw/current/scripts/components/burnable.lua#L291 [burnable.lua:489](../../../raw/current/scripts/components/burnable.lua#L489 [burnable.lua:593](../../../raw/current/scripts/components/burnable.lua#L593

存档必须看具体目标：原版默认可燃物品点燃将 persists=false，熄灭恢复；木箱则保存“正在燃烧或已烧毁”为 burnt，读档直接调燃尽回调，并非恢复剩余秒数。[standardcomponents.lua:12](../../../raw/current/scripts/standardcomponents.lua#L12 [standardcomponents.lua:46](../../../raw/current/scripts/standardcomponents.lua#L46 [treasurechest.lua:85](../../../raw/current/scripts/prefabs/treasurechest.lua#L85

## 常见陷阱与限制

- **canlight=false 就完全防火**：直接 Ignite／StartWildfire 不检查此字段；fireimmune 会阻止这些入口，但不能因此承诺免除 propagator 火伤，伤害资格另查 [[reference/components/propagator#常见陷阱与限制]]。[burnable.lua:251](../../../raw/current/scripts/components/burnable.lua#L251 [burnable.lua:354](../../../raw/current/scripts/components/burnable.lua#L354
- **换燃尽回调只影响特效**：默认清理还涉及灰烬、删除／烧毁状态、容器内容物和功能组件；先审阅所用 helper 与原回调，不用空函数代替整条流程。[standardcomponents.lua:18](../../../raw/current/scripts/standardcomponents.lua#L18 [standardcomponents.lua:52](../../../raw/current/scripts/standardcomponents.lua#L52
- **LongUpdate(dt) 推进精确剩余时间**：对有燃尽任务的燃烧对象直接 DoneBurning；不能把它当普通 dt 计时补偿。[burnable.lua:406](../../../raw/current/scripts/components/burnable.lua#L406

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；本页 sources 全部未变（burnable.lua、standardcomponents.lua、log.lua、treasurechest.lua、modutil.lua 均不在 25643519→26148036 变更清单）；已核关键消费者：燃尽侧 explosive.lua 新增 OnBurnt 可选参且省略时保持全范围爆炸，本组件无参调用兼容（[burnable.lua:304](../../../raw/current/scripts/components/burnable.lua#L304），actions.lua 的 SMOTHER 分支与拾取改动不涉及本页结论，tuning 燃烧／熄灭键未变；未覆盖：运行验证。

- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖上文计时／冒烟／点燃／熄灭、事件顺序、默认物品／结构回调及 log、木箱案例。全部 FX 类型、controlled burn 技能、燃烧存档方案与引擎任务生命周期未全量审校。
- 结构检查：元数据、证据行、链接／章节、大小、登记通过；发布示例及木头燃烧＋传播隔离接入经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；真实燃尽秒数、熄灭再点燃、存档、多人 FX 与其他模组覆盖待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/propagator#典型接入]]；[[reference/components/fuel#典型接入]]；[[reference/components/workable#工作顺序与原版完成位置]]。
