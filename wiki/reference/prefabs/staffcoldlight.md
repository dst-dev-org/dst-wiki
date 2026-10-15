---
title: "极光（staffcoldlight）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "冷星／极光这一召唤实体的制冷、精神光环和存续时间，并区分月台附属 staffcoldlightfx 与唤月法杖物品。"
aliases: ["staffcoldlight", "极光", "冷星", "Polar Light", "极光持续时间", "冷星降温", "staffcoldlightfx", "月台冷星特效"]
tags: [prefab, aura, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["prefabs/stafflight.lua", "prefabs/staff_common.lua", "prefabs/moonbase.lua", "tuning.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 极光（staffcoldlight）

## 快速判断

- 职责与适用：这是 [[reference/prefabs/opalstaff|唤月者魔杖]] 通常召出的冷星实体，有照明、制冷、精神光环和自动退出计时；“冷星、极光”也可能指法杖，需按上下文区分。[生成选择](../../../raw/current/scripts/prefabs/staff_common.lua#L782-L784)、[能力](../../../raw/current/scripts/prefabs/stafflight.lua#L163-L186)
- 优先使用：本页查制冷、时长及月台 FX 差异；通用 Light 脉动和 kill_light 流程复用 [[reference/prefabs/stafflight#光照脉动与消失]]，法杖次数查 opalstaff。
- 重要副作用：heater.heat=-100 并 SetThermics(false,true)，sanityaura 独立存在；普通冷星不走 is_hot 的 cooker／propagator 分支。[组件分流](../../../raw/current/scripts/prefabs/stafflight.lua#L152-L186)
- 修改边界：同页维护普通 staffcoldlight 和同族附属 staffcoldlightfx；后者没有普通计时／ForceExtinguish，却仍有 heater／sanityaura。其余同源 emberlight 不在本页覆盖。[工厂分支](../../../raw/current/scripts/prefabs/stafflight.lua#L163-L188)、[注册参数](../../../raw/current/scripts/prefabs/stafflight.lua#L202-L205)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `staffcoldlight` | makestafflight(name, is_hot=false, star_cold, …, is_fx=false) → 公共网络灯／脉动 → 主模拟组件 | 正常冷星，有 inspectable／hauntable／timer 和 ForceExtinguish；随机切换三种 idle 动画。[构造](../../../raw/current/scripts/prefabs/stafflight.lua#L66-L147)、[主模拟](../../../raw/current/scripts/prefabs/stafflight.lua#L163-L194)、[注册](../../../raw/current/scripts/prefabs/stafflight.lua#L203) |
| `staffcoldlightfx` | 同冷星函数，但 is_fx=true | 缩放 0.92、隐藏 shadow，persists=false；跳过普通星体的检视／作祟／自动计时分支，玩法组件仍保留。[公共差异](../../../raw/current/scripts/prefabs/stafflight.lua#L114-L126)、[分支](../../../raw/current/scripts/prefabs/stafflight.lua#L163-L188)、[注册](../../../raw/current/scripts/prefabs/stafflight.lua#L204) |

普通名称键 `STRINGS.NAMES.STAFFCOLDLIGHT` 为“极光”／Polar Light；“冷星”按用户提供收录。附属 FX 不据此假定具有同一检视名称。[译名](../../../raw/current/scripts/languages/chinese_s.po#L349330-L349332)、[默认取名](../../../raw/current/scripts/entityscript.lua#L840-L844)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源：opalstaff | createlight 实际生成 staffcoldlight 并设置位置 | 普通冷星；不生成 staffcoldlightfx | [[reference/prefabs/opalstaff#查询冷星时先区分施法物品与产物]]；[选择](../../../raw/current/scripts/prefabs/staff_common.lua#L782-L784) |
| heater／sanityaura | 制冷输出 -100、关闭供热；精神光环取 SANITYAURA_SMALL | 主模拟；两种冷星工厂均挂载 | [[reference/components/heater#携带范围与地面热源的分工]]；[[reference/components/sanityaura#关键接口]]；[配置](../../../raw/current/scripts/prefabs/stafflight.lua#L163-L172) |
| inspectable／hauntable／timer | 普通星体检视、作祟主动熄灭、extinguish 定时退出 | 仅 staffcoldlight，is_fx=false | [[reference/components/inspectable#关键接口]]；[[reference/components/hauntable#关键接口]]；[[reference/components/timer#生命周期]]；[普通分支](../../../raw/current/scripts/prefabs/stafflight.lua#L174-L188) |
| moonbase：附属体宿主 | ShowColdStar 生成 staffcoldlightfx 并挂为子实体，HideColdStar 直接移除 | 月台 Wiki 待建；本页只核验附属体拥有与释放接口 | [生成与父实体](../../../raw/current/scripts/prefabs/moonbase.lua#L161-L169)、[清理](../../../raw/current/scripts/prefabs/moonbase.lua#L171-L179) |
| Light／net_float／动画 | 与热星共用 0.1 秒脉动任务、相位同步与客户端调制；资源换为 star_cold | 公共端；随机 idle，_killed 后不继续随机播 idle | [[reference/prefabs/stafflight#光照脉动与消失]]；[随机动画](../../../raw/current/scripts/prefabs/stafflight.lua#L66-L77)、[注册资源参数](../../../raw/current/scripts/prefabs/stafflight.lua#L203-L204) |

## 典型接入

### 制冷、光环与存续时间

本版 OPALSTAFF_STAR_DURATION=total_day_time×2，用于普通 staffcoldlight 的 extinguish timer；附属 FX 虽也填写 scrapbook 时长展示字段，却没有在 is_fx 分支启动 timer，不能据展示字段推定自动消失。[时长](../../../raw/current/scripts/tuning.lua#L211-L212)、[展示字段](../../../raw/current/scripts/prefabs/stafflight.lua#L92)、[实际计时分支](../../../raw/current/scripts/prefabs/stafflight.lua#L174-L186)

温度作用由 heater 的附近通道消费，输出 -100 不等于人物被直接设成 -100；实际换热仍依赖接收者 temperature。sanityaura 的接收范围和同名叠加规则由 [[reference/components/sanityaura#关键接口]] 维护，光照半径不是两种玩法效果的统一半径。[输出配置](../../../raw/current/scripts/prefabs/stafflight.lua#L163-L172)

普通冷星的 timerdone／ForceExtinguish 共用热星页所述 kill_light：先推 onstarkilled 和播退场，再延迟 1 秒 Remove；本页不重复整套过程。[退出函数](../../../raw/current/scripts/prefabs/stafflight.lua#L9-L22)、[主动入口](../../../raw/current/scripts/prefabs/stafflight.lua#L54-L64)

### 月台附属变体 staffcoldlightfx

ShowColdStar 创建且仅保留一个 _staffstar，并设为 moonbase 子实体；HideColdStar Remove 后清空引用。该变体跳过普通 timer，由宿主接口管理存在期。这里只核验这对接口和转换分支的 ShowColdStar 调用，不声称覆盖月台全部调用时机。[宿主管理](../../../raw/current/scripts/prefabs/moonbase.lua#L161-L179)、[转换后展示](../../../raw/current/scripts/prefabs/moonbase.lua#L468-L483)

工厂刻意未添加 FX／NOCLICK 标签，注释说明其会妨碍精神光环；但这不等于该实体具备普通检视和 ForceExtinguish。冷星本体的行为不能直接整套复制到该变体。[公共差异与注释](../../../raw/current/scripts/prefabs/stafflight.lua#L114-L128)、[主模拟差异](../../../raw/current/scripts/prefabs/stafflight.lua#L163-L188)

## 常见陷阱与限制

- **误区：带 fx 后缀就只有视觉效果**：staffcoldlightfx 仍有 heater 和 sanityaura；生成／移除会改变这两个组件的存在范围。[实际挂载](../../../raw/current/scripts/prefabs/stafflight.lua#L163-L175)
- **误区：每个带 staffstar 标签的实体都能 ForceExtinguish**：标签在公共段添加，函数只在非 is_fx 分支赋值；调用方仍须检查实际接口，不能只看标签。[标签](../../../raw/current/scripts/prefabs/stafflight.lua#L128)、[函数赋值](../../../raw/current/scripts/prefabs/stafflight.lua#L174-L186)
- **误区：冷星会像热星一样烹饪或传播火势**：cooker／propagator 位于 is_hot 分支，两个冷星注册参数均为 false。[热星分支](../../../raw/current/scripts/prefabs/stafflight.lua#L152-L161)、[冷星参数](../../../raw/current/scripts/prefabs/stafflight.lua#L203-L204)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整读取 stafflight.lua，核验两种冷星的参数、条件组件、脉动与计时退出，核验 staff_common 普通生成、moonbase 附属体拥有／清理及中文名称；共用光照过程主要维护在 stafflight 页。
- 游戏运行：未运行；未实测冷星与热源混用、光环叠加、月台全部转换／恢复时序或实际点击表现，未检查全部 staffstar 消费者。无新增 Lua 示例；资源未解包验证。

返回 [[reference/prefabs/index|Prefab 索引]]。
