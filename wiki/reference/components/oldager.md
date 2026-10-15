---
title: "oldager：旺达自然老化与剩余寿命"
created: "2026-10-27"
updated: "2026-10-28"
type: component
summary: "将旺达剩余寿命映射到 health，按时间结算自然老化、受伤加速与回春恢复。"
aliases: ["旺达", "Wanda", "旺达生命值流逝", "自然老化", "衰老", "剩余寿命", "回春表", "oldager"]
tags: [component, lifecycle, callback, mechanism, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/oldager.lua", "components/health.lua", "prefabs/wanda.lua", "prefabs/pocketwatch.lua", "prefabs/player_classified.lua", "widgets/wandaagebadge.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# oldager：旺达自然老化与剩余寿命

## 快速判断

- 职责与适用：把 health 作为剩余寿命，通过更新累计年龄；默认自然项为每游戏模拟秒 `1/40` 年、倍率 `rate=1`，无额外影响时约每40秒结算1年／扣1点剩余寿命。[默认值与更新注册](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L9-L23)、[自然项与整数结算](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L55-L90)
- 优先使用：查询“旺达为什么缓慢掉血”看下方自然老化流程；伤害和回春通过 Wanda 的 health.redirect 接入，不能用普通 health 再生解释。[宿主接线](../../../raw/archive/build-26148036/scripts/prefabs/wanda.lua#L396-L414)、[伤害转发](../../../raw/archive/build-26148036/scripts/prefabs/wanda.lua#L265-L267)
- 重要副作用：普通 DoDelta 会先被重定向；受伤进入待结算老化，只有登记 cause 的正向治疗才会转为回春。redirect 返回 true 时普通结算终止，不代表即时生命变化。[输入转换](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L33-L53)、[health 拦截](../../../raw/archive/build-26148036/scripts/components/health.lua#L623-L624)
- 修改边界：原版在服务端 master_postinit 添加；不能只给普通角色挂 oldager 就称完成旺达寿命机制，还需 health 重定向、数值配置和同步等宿主协作。本页未覆盖整套年龄外观、伤害加成或复活机制。[Wanda 配置](../../../raw/archive/build-26148036/scripts/prefabs/wanda.lua#L390-L415)

## 关键接口

| 接口／字段 | 约定 | 条件、作用与证据 |
|---|---|---|
| `base_rate`／`rate` | 自然年数每秒／单一倍率；默认 1/40、1 | 自然项为 dt×base_rate×rate。rate 不是多来源修饰器，也不控制待结算伤害的速度。[初值](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L12-L18)、[自然项](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L55-L72) |
| `AddValidHealingCause(cause_name)` | 向集合登记 cause | OnTakeDamage 仅把负 amount 或登记 cause 纳入转换；未登记的普通正向治疗被拦截而不累加。[登记与判断](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L29-L52) |
| `OnTakeDamage(amount, overtime, cause, ignore_invincible, afflicter, ignore_absorb)` | health.redirect 的实际参数；通常 true，内部时间结算时 false | 反转 amount 符号，按 OLDAGE_HEALTH_SCALE 换算后累加 damage_remaining；相反方向的既有待结算量先清零，再接收新输入。[实现](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L33-L53) |
| `damage_remaining`／`damage_per_second` | 待结算年数／带符号年数每秒 | 正值为老化，负值为回春；速度大小为 min(ceil(sqrt(abs(remaining))×1.5),30)，按输入重新计算，不是直接扣同等 health 点数。[转换](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L40-L49) |
| `StopDamageOverTime()` | 清除待结算量和速度 | 发内部 DoDelta(0) 刷新事件；不停止组件更新、不修改 rate，也不是暂停自然老化。[实现](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L115-L122) |
| `GetCurrentYearPercent()` | 返回 year_timer | 读取尚未折入整数生命值的年内进度；不是角色完整年龄。[实现](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L124-L126) |
| `LongUpdate(dt)` | 直接调用一次 OnUpdate(dt) | 使用同一累计与结算流程，不能当作离线不老化的保证。[实现](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L128-L130) |
| `FastForwardDamageOverTime()` | 仅剩余量、速度均为正时生效 | 用 remaining／speed 作为 dt 调 OnUpdate，再停止待结算；这段 dt 的自然项也参与，不用于快进负向回春。[实现](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L132-L139) |

## 关键流程

### 自然老化为何表现为生命缓慢流逝

1. 构造时注册组件更新。每次 OnUpdate(dt)，仅在 health 未死亡且 damage_remaining≥0 时累加自然项；正在负向回春时自然项为0。[更新注册](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L9-L23)、[自然项条件](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L55-L56)
2. 如有待结算伤害／回春，再按 damage_per_second×dt 结算；跨过0时截到剩余量，避免多结算。这一项与自然项分开。[待结算推进](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L60-L73)
3. 以 floor(year_timer＋本帧年龄变化) 得到 years_delta，余数留在 year_timer。整数变化时调用 health:DoDelta(-years_delta, false, "oldager_component")，所以不是每帧直接扣一小数点血量。[折算与扣值](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L75-L90)
4. 内部调用前后用 _taking_time_damage 防止再次被自身重定向。年内进度及待结算速率另通过 player_classified 同步；回春到满寿命或死亡有相应清理。本页不据此认定完整 UI 预测已审校。[防重入与死亡清理](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L79-L96)、[进度写回](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L99-L110)、[速率同步](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L3-L6)、[网络字段](../../../raw/archive/build-26148036/scripts/prefabs/player_classified.lua#L1444-L1446)

### 受伤与回春不是同一个计时入口

外部负 amount 经重定向变成正的待结算年龄，按 OLDAGE_HEALTH_SCALE=60／150 换算；新的回春与伤害相反时，先丢弃原方向剩余量再累加。普通正向治疗只有 cause 已登记才转换为负向待结算；其余也返回拦截信号，但不改变待结算量。不能用 DoDelta 的返回0判断回春没有发生。[输入转换](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L33-L53)、[换算常量](../../../raw/archive/build-26148036/scripts/tuning.lua#L2739-L2740)、[重定向返回](../../../raw/archive/build-26148036/scripts/components/health.lua#L623-L624)

## 典型接入

### 原版旺达的寿命与回春接线

Wanda 的服务端添加 oldager，登记 pocketwatch_heal／debug_key 治疗 cause；将最大剩余寿命设为 WANDA_OLDAGER=60、初始比例0.70，再安装 redirect，关闭普通 canheal 和生命惩罚。常量注释以20～80岁对应60点寿命；角色显示、攻击和资源仍由宿主其他逻辑负责。[宿主配置](../../../raw/archive/build-26148036/scripts/prefabs/wanda.lua#L396-L415)、[年龄常量](../../../raw/archive/build-26148036/scripts/tuning.lua#L2739-L2740)

回春表实际施法先检查 health 存在且未死亡，调用 oldager:StopDamageOverTime，再以自身 prefab 作为 cause 执行 health:DoDelta；随后进入上述允许 cause 的负向回春。这里不是直接把当前寿命一次性加满。[回春表调用](../../../raw/archive/build-26148036/scripts/prefabs/pocketwatch.lua#L29-L40)

### 调慢当前自然老化时应改什么

在已经完成原版接线的旺达服务端实例上，rate=0.5 只把自然项减半，默认且无伤害／回春时约80秒结算1年；rate=0 不清除已排队的受伤老化。修改自然项和停止待结算是不同需求；组件没有多来源 rate 管理，也没有在组件中保存这些字段的 OnSave／OnLoad，重建后的配置与恢复策略须由接入方明确。这里说明字段作用，不提供替代整套旺达工厂的完整模组示例。[构造初值](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L9-L23)、[更新分流](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L55-L77)、[停止待结算](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L115-L122)

客户端年龄表盘的年内进度预测另写为 `(1/40 + dps_rate*0.9) * dt`，该分支不读取服务端的 oldager.rate；仅改服务端倍率不能保证表盘预估也同步变速，须另处理预测接线。本页只核验此分支，不提供完整 UI 改造方案。[客户端预测](../../../raw/archive/build-26148036/scripts/widgets/wandaagebadge.lua#L201-L214)

## 常见陷阱与限制

- **把 health 当普通150点生命**：原版寿命池为60，伤害转换另乘60／150；不要直接把普通人物数值当成岁数。[常量](../../../raw/archive/build-26148036/scripts/tuning.lua#L2739-L2740)
- **普通回血或 StopDamageOverTime 就能返老还童**：前者可能被 cause 门禁截住，后者只清待结算并刷新事件；回春表的后续 DoDelta 才建立负向待结算。[转换](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L33-L53)、[清理](../../../raw/archive/build-26148036/scripts/components/oldager.lua#L115-L122)、[原版回春](../../../raw/archive/build-26148036/scripts/prefabs/pocketwatch.lua#L29-L40)
- **把 CanHeal 或界面外观当作伤害入口**：实际接线是 health.redirect，内部时间结算依靠防重入标志；修改或替换 redirect 需要保留原语义。[转发](../../../raw/archive/build-26148036/scripts/prefabs/wanda.lua#L265-L267)、[health 门禁](../../../raw/archive/build-26148036/scripts/components/health.lua#L623-L628)
- **本页覆盖整套旺达**：本页覆盖自然老化、输入转换、回春与必要同步；年龄阶段战斗加成、完整 UI、复活和其他物品消费者不在本页审阅范围。

## 验证记录

- 源码审阅：2026-10-27，Steam buildid 26148036；读取 oldager 全部实现，并核验 Wanda 服务端添加／redirect／数值、health 重定向分支、回春表实际调用、寿命换算常量、年内进度／速率同步字段及表盘固定自然速率预测分支。案例是原版接线与字段作用说明。
- 游戏运行：未运行；尚未实测多人预测、存读／长更新、修改 rate、伤害与回春交错、满寿命及死亡边界。未完整审校年龄阶段、复活或其他消费者，机械检查不证明游戏效果。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/health]]；[[reference/components/positionalwarp]]；[[reference/components/staffsanity]]。
