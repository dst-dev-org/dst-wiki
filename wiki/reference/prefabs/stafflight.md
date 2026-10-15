---
title: "矮星（stafflight）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "唤星法杖生成的矮星实体：脉动光照、供热、烹饪、火势传播、精神光环和定时熄灭，与施法物品分开维护。"
aliases: ["stafflight", "矮星", "Dwarf Star", "矮星光源", "矮星持续时间", "矮星供热", "矮星烹饪"]
tags: [prefab, aura, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["prefabs/stafflight.lua", "prefabs/staff_common.lua", "tuning.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 矮星（stafflight）

## 快速判断

- 职责与适用：这是召出的世界实体，提供照明、附近热源、烹饪、传播热量和精神光环；施法物品是 [[reference/prefabs/yellowstaff|唤星者魔杖]]。[组件配置](../../../raw/current/scripts/prefabs/stafflight.lua#L152-L186)
- 优先使用：改产物持续／光照／供热读本页；改召唤次数和精神支出读法杖页。相同“矮星”称呼不说明两个对象共享耐久或销毁生命周期。
- 重要副作用：propagator 以 heatoutput=15 启动传播；heater 输出 100 与 propagator 热量是两个不同消费者体系。[热量配置](../../../raw/current/scripts/prefabs/stafflight.lua#L152-L172)
- 修改边界：公共端建网络光源与脉动任务，主模拟加玩法组件和熄灭计时。其近亲 staffcoldlight、staffcoldlightfx、emberlight 用相同工厂但参数不同，本文不将它们都按热星处理。[公共构造](../../../raw/current/scripts/prefabs/stafflight.lua#L79-L147)、[注册参数](../../../raw/current/scripts/prefabs/stafflight.lua#L202-L205)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `stafflight` | makestafflight(name, is_hot=true, …, is_fx=false) → 公共 Light／Network／net_float／动画 → SetPristine → 主模拟组件 | star_hot 资源，普通热星；定时和作祟熄灭均启用。[工厂](../../../raw/current/scripts/prefabs/stafflight.lua#L66-L147)、[注册](../../../raw/current/scripts/prefabs/stafflight.lua#L202) |
| `emberlight` | 同函数另一组 name／动画／pre／pst 参数 | 同源关联、Wiki 待建；有独立时长配置，不在本页扩展其来源与生命周期。[注册区别](../../../raw/current/scripts/prefabs/stafflight.lua#L205)、[定时分支](../../../raw/current/scripts/prefabs/stafflight.lua#L184-L186) |

名称键 `STRINGS.NAMES.STAFFLIGHT` 为“矮星”／Dwarf Star。[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L349335-L349337)、[默认取名](../../../raw/current/scripts/entityscript.lua#L840-L844)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源：yellowstaff | createlight 显式生成 stafflight，并放到目标 pos | 主模拟；生成后未在此函数登记为法杖子实体 | [[reference/prefabs/yellowstaff#施法与消耗应修改哪一层]]；[生成](../../../raw/current/scripts/prefabs/staff_common.lua#L782-L800) |
| cooker | 热星且非 is_fx 时挂载；用于直接烤制，不是 stewer 烹饪锅 | stafflight 满足两个条件 | [[reference/components/cooker#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/stafflight.lua#L152-L155) |
| propagator／heater | 前者 heatoutput=15 并 StartUpdating；后者 heat=100 | 主模拟热星；携带者专属输出没有配置 | [[reference/components/propagator#关键接口]]；[[reference/components/heater#携带范围与地面热源的分工]]；[配置](../../../raw/current/scripts/prefabs/stafflight.lua#L157-L169) |
| sanityaura | aura=TUNING.SANITYAURA_SMALL | 主模拟；实际距离／叠加和接收条件沿用组件规则 | [[reference/components/sanityaura#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/stafflight.lua#L171-L172) |
| inspectable／hauntable／timer | 普通星体有检视，作祟调用 ForceExtinguish，extinguish 定时触发退出 | is_fx=false；主模拟 | [[reference/components/inspectable#关键接口]]；[[reference/components/hauntable#关键接口]]；[[reference/components/timer#生命周期]]；[配置](../../../raw/current/scripts/prefabs/stafflight.lua#L174-L188) |
| Light／net_float／资源 | 每 0.1 秒 pulse_light，主模拟约每 30 秒同步时间相位；两端用 Light 客户端调制 | 不是按每次网络传输开关灯；资源 star_hot | [脉动函数](../../../raw/current/scripts/prefabs/stafflight.lua#L24-L52)、[调度与资源](../../../raw/current/scripts/prefabs/stafflight.lua#L79-L108) |
| 冷星同族 | 不同 is_hot 参数生成 staffcoldlight；月台附属体用 is_fx=true | 行为差异维护在冷星页，避免重复复制 | [[reference/prefabs/staffcoldlight#注册与构造]]；[注册](../../../raw/current/scripts/prefabs/stafflight.lua#L202-L205) |

## 典型接入

### 能力与持续时长的修改入口

本版 YELLOWSTAFF_STAR_DURATION=total_day_time×3.5，timer 在主体构造时启动，修改这个初始化入口影响新实例；现存实例的剩余时间应通过 timer 的对应接口处理。法杖的有限次数不负责维护该计时。[时长](../../../raw/current/scripts/tuning.lua#L208-L212)、[启动](../../../raw/current/scripts/prefabs/stafflight.lua#L184-L186)；通用保存／恢复边界见 [[reference/components/timer#生命周期]]。

加热、火势传播、精神光环和烹饪分别由 heater、propagator、sanityaura、cooker 承载。只改 Light 半径不会自动改变热源搜索范围或光环规则；按目标能力进入相应组件页。[分工配置](../../../raw/current/scripts/prefabs/stafflight.lua#L152-L172)

### 光照脉动与消失

pulse_light 根据 GetTimeAlive 和 _pulseoffs 计算正弦插值：半径在 11～12 间，强度／衰减在 0.8～0.7 间；客户端由同步时间修正相位，Light 启用客户端调制。[插值与同步](../../../raw/current/scripts/prefabs/stafflight.lua#L24-L52)、[调制开关](../../../raw/current/scripts/prefabs/stafflight.lua#L96-L100)

timerdone 只在 name="extinguish" 时调用 kill_light；主动 ForceExtinguish 先确认计时存在并 Stop，再走同一退出。kill_light 推 onstarkilled、播 disappear、监听动画结束停声、1 秒后 Remove，并置 persists=false／_killed=true。它不是立刻移除实体或逐个撤掉全部能力组件。[计时与退出](../../../raw/current/scripts/prefabs/stafflight.lua#L5-L22)、[主动熄灭](../../../raw/current/scripts/prefabs/stafflight.lua#L54-L64)

## 常见陷阱与限制

- **误区：矮星是只负责发光的 FX**：本注册参数 is_fx=false，包含 cooker／propagator／heater／sanityaura 和 timer，不能只复制 Light 得到完整行为。[参数](../../../raw/current/scripts/prefabs/stafflight.lua#L202)、[组件](../../../raw/current/scripts/prefabs/stafflight.lua#L152-L186)
- **误区：ForceExtinguish 对所有同源对象都可直接调用**：函数和 timer 只在非 is_fx 分支设置；[[reference/prefabs/staffcoldlight#月台附属变体-staffcoldlightfx]] 不满足该分支。[分支](../../../raw/current/scripts/prefabs/stafflight.lua#L174-L186)
- **误区：timer 能保存就保证灯光相位也完全恢复**：本文件仍有 time alive 保存 TODO；本页不保证跨读档脉动相位连续，需区分组件计时与 GetTimeAlive。[原注释](../../../raw/current/scripts/prefabs/stafflight.lua#L3)、[相位来源](../../../raw/current/scripts/prefabs/stafflight.lua#L28-L51)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整读取 stafflight.lua，核验普通热星参数、直接组件、公共脉动同步、计时／作祟退出和 staff_common 实际生成。通用组件内部行为复用已有 Wiki，不展开所有消费者。
- 游戏运行：未运行；未验证实际温度／光环叠加、引燃、读档相位、暂停与长更新或底层渲染，未解包资源。无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
