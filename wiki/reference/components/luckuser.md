---
title: "luckuser"
created: "2026-10-24"
updated: "2026-10-28"
type: component
summary: "按来源累加实体幸运值并更新负幸运的猎犬修正；具体概率须由调用方选择公式。"
aliases: ["luckuser", "角色幸运值", "幸运来源叠加", "负幸运", "TryLuckRoll", "SetLuckSource"]
tags: [component, callback, lifecycle, loot, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/luckuser.lua", "util/sourcemodifierlist.lua", "componentutil.lua", "components/houndedtarget.lua", "components/lootdropper.lua", "prefabs/player_common.lua", "prefabs/wx78_possessedbody.lua", "prefabs/hats.lua", "prefabs/wes.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# luckuser

## 快速判断

- 职责与适用：持有实体的来源加法表，初始总值0；原版玩家服务端与 WX 附身躯体都有组件。物品携带／装备贡献由 [[reference/components/luckitem]] 提供，也可由套装等宿主直接登记。[构造](../../../raw/current/scripts/components/luckuser.lua#L1-L9) [玩家](../../../raw/current/scripts/prefabs/player_common.lua#L2866) [附身躯体](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L657)
- 优先使用：`SetLuckSource(value, source)` 添加／替换一个稳定来源，`RemoveLuckSource(source)` 成对撤销，`GetLuck()` 读总值；不要直接覆写 luckmodifiers 或混用不同来源标识。[接口](../../../raw/current/scripts/components/luckuser.lua#L16-L49)
- 重要副作用：总值为负会给 houndedtarget 添加自己的目标权重与偷猎犬来源修正；回到非负只撤本组件这份修正，不删除其他系统来源。不是只影响抽奖的无副作用字段。[更新](../../../raw/current/scripts/components/luckuser.lua#L20-L34) [读取方](../../../raw/current/scripts/components/houndedtarget.lua#L1-L17)
- 修改边界：普通 Lua 来源表无 netvar 或存档接口；幸运值不是概率，也不全局接管 math.random。随机路径主动调用 GetEntityLuckChance／TryLuckRoll 才使用相应公式。[概率入口](../../../raw/current/scripts/componentutil.lua#L2203-L2214) [抽取](../../../raw/current/scripts/componentutil.lua#L2237-L2251)

## 关键接口

| 接口／字段 | 参数、合并与副作用 |
|---|---|
| `SetLuckSource(luck, source)` | luck=0 转 Remove；否则以 source 同时作来源和键，写数值后刷新附加修正。同 source 重复设置覆盖旧值，不累加同一来源；不同来源相加。[实现](../../../raw/current/scripts/components/luckuser.lua#L38-L43) [加法器](../../../raw/current/scripts/util/sourcemodifierlist.lua#L29-L31) [覆盖](../../../raw/current/scripts/util/sourcemodifierlist.lua#L60-L98) |
| `RemoveLuckSource(source)` | 移除对应键并刷新附加修正；用原 source，不能用“等价描述”的另一个实体代替。[实现](../../../raw/current/scripts/components/luckuser.lua#L47-L49) |
| `GetLuck()` | 直接返回来源表总值，不夹到0～1、不百分比转换，也不产生抽取。数值正负效果由具体公式／消费者解释。[读取](../../../raw/current/scripts/components/luckuser.lua#L16-L17) |
| `_UpdateLuck_Internal()` | luck<0 时确保 houndedtarget 存在，目标权重乘数为 `1+abs(luck)`；luck<=-1 再启用该来源的偷猎犬标志。非负时撤销这两个修正，不删除 houndedtarget。[实现](../../../raw/current/scripts/components/luckuser.lua#L20-L34) |
| `OnRemoveFromEntity()` | 先 Reset 自己的来源表，再刷新，以撤掉自己的负幸运附加修正；不是清空整个 houndedtarget 的其他来源。[移除](../../../raw/current/scripts/components/luckuser.lua#L11-L13) [Reset](../../../raw/current/scripts/util/sourcemodifierlist.lua#L124-L135) |

### 幸运值如何进入概率

`GetEntityLuck(inst)` 无组件时返回0；GetEntityLuckChance 读取幸运后调用 `formula(inst, base_chance, luck)`，公式返回 false／nil 时回退基础值。TryLuckRoll 只做一次 math.random，用 `roll <= new_chance` 判断；inst 为空则直接用基础 chance，不应用 formula。它没有通用的概率夹取或“每点幸运等于固定百分比”规则。[值与公式](../../../raw/current/scripts/componentutil.lua#L2203-L2214) [抽取](../../../raw/current/scripts/componentutil.lua#L2237-L2251)

例如概率表中的 CriticalStrike 使用加法公式，而 [[reference/components/spooked#积累与概率如何计算]] 的惊吓采用另一公式；不能把一个系统的倍率搬到所有概率上。源码的幸运表现调用已注释，抽取改变也不保证播放幸运特效。[公式定义](../../../raw/current/scripts/componentutil.lua#L2078-L2081) [公式选择](../../../raw/current/scripts/componentutil.lua#L2129-L2140) [表现注释](../../../raw/current/scripts/componentutil.lua#L2243-L2248)

## 典型接入

### 参考原版套装登记与撤销，不复制物品内部逻辑

骑士套装启用时，向拥有者 luckuser 以头盔实体为 source 调 SetLuckSource；停用时用同一实体 RemoveLuckSource。它另外遍历库存，给 luckitem 物品推 updateownerluck，使马蹄铁的套装条件贡献也重新求值。这是**两份不同来源策略**，不是给套装头盔额外挂 luckitem 就自动获得的协议。[完整宿主回调](../../../raw/current/scripts/prefabs/hats.lua#L6458-L6480)

目标模组新增效果可在服务端生效／撤销位置沿用同源增删，已有玩家不重挂组件；来源数值、技能状态与是否持久化由自己的宿主负责。本页仅给组装位置与原版参照，不提供省略存读／撤销的新 Buff demo。[玩家安装](../../../raw/current/scripts/prefabs/player_common.lua#L2866) [同源协议](../../../raw/current/scripts/components/luckuser.lua#L38-L49)

### 判断某条掉落是否真的读取了幸运

LootDropper:GetLuckyUser 优先取仍有效的 health.causeofdeath，其次取 workable.lastworker；不是任意附近玩家。GetChance 对基础概率>=1 直接返回，其余有归属实体时应用 LootDropperChance。生成列表中固定 loot 不走这次概率变换，因此“玩家更幸运”不能解释成所有掉落数量都翻倍。完整掉落机制仍见 [[reference/components/lootdropper]]。[归属与概率](../../../raw/current/scripts/components/lootdropper.lua#L163-L184) [条件与固定掉落](../../../raw/current/scripts/components/lootdropper.lua#L193-L243)

## 常见陷阱与限制

- **靠实体来源自动移除完成全部撤销：** SourceModifierList 自动 onremove 只删来源并重算；luckuser 构造未传 dirty callback，所以不会自动调用 _UpdateLuck_Internal，猎犬修正可能仍保留旧值。主动效果结束必须走 RemoveLuckSource；普通 luckitem 的来源 helper 有显式移除路径，不能把二者混为一谈。[构造](../../../raw/current/scripts/components/luckuser.lua#L1-L8) [自动移除](../../../raw/current/scripts/util/sourcemodifierlist.lua#L84-L91) [显式移除](../../../raw/current/scripts/components/luckuser.lua#L47-L49)
- **以为0会保留一份中性来源或幸运有统一上限：** SetLuckSource(0,source) 实际删除该来源；加法表不限制总值，正负相抵后是否仍触发附加修正看最终总数。调用者保证有效数值，不能把字符串／函数直接当此方法的数值参数。[接口](../../../raw/current/scripts/components/luckuser.lua#L38-L49)
- **看到 TUNING.WES_LUCK 就认定维斯已应用负幸运：** 当前 wes 工厂的 SetLuckSource 这一行被注释；它另行直接配置 houndedtarget。不能从常量或注释意图推断当前角色幸运值。[实际代码](../../../raw/current/scripts/prefabs/wes.lua#L32-L47)
- **客户端或载入后自动恢复所有来源：** 组件不保存来源表，也不注册网络变量；物品、套装或模组效果在各自生命周期重新登记，显示总值需另设计同步。移除本组件也不自动改掉物品／宿主代码中对它的后续访问。[完整定义](../../../raw/current/scripts/components/luckuser.lua#L1-L56)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 9 处（componentutil 幸运公式族随 +231 大改平移至 2065-2238，逐处内容核对公式体未变；player_common luckuser 挂载 -14；hats yoth_knight 段 -2）；无结论修订——luckuser.lua、houndedtarget、wes 未变，wx78_possessedbody 引用行未变；lootdropper 引用行（163-243）未变，本版其改动仅在 FlingItem（新增 `nofling` 抛掷开关，默认行为不变）；C-19 GetGrandOwner 重写（inventoryitem 内部）语义等价，不影响本页归属结论。依据：组内共享 componentutil diff＋变更清单命中 player_common.lua、prefabs/wx78_possessedbody.lua（fn 内 +1-1 净零）、prefabs/hats.lua、components/lootdropper.lua、components/inventoryitem.lua。
- 源码审阅：2026-10-24，Steam buildid 25643519；完整 luckuser、SourceModifierList 加法与自动清理，核对 houndedtarget 字段、玩家／WX附身安装、套装登记／撤销、维斯注释及幸运公式入口和掉落归属。未全面审阅所有幸运公式、猎犬生成算法或随机系统。
- 结构检查：本页格式、来源、本地链接与源码锚点严格检查通过；无新增完整 Lua 示例，导航登记由本批统一检查。
- 游戏运行：未运行；负幸运修正撤销、来源实体强制删除、套装重入、掉落归属和自定义来源存读未实测。

## 相关链接

- [[reference/components/houndedtarget]]：猎犬目标权重与偷猎犬者身份。
