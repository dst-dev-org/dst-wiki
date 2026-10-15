---
title: "socketholder"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "管理实体的固定网络插槽，将镶嵌物品转换为存档数据，并在卸下、死亡和读档时协作重建实体与效果。"
aliases: ["插槽接收方", "WX78暗影插槽", "镶嵌物品存档", "socketholder"]
tags: [component, item, callback, lifecycle, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/socketholder.lua", "componentutil.lua", "prefabs/wx78_common.lua", "prefabs/wx78_backupbody.lua", "prefabs/wx78_classified.lua", "prefabs/player_common.lua", "stategraphs/SGwilson.lua", "networkclientrpc.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# socketholder

## 快速判断

- 职责与适用：宿主实体的固定镶嵌槽；双端公开占用／品质／名字 netvar，服务端保存物品序列化数据，不是持有活实体的普通容器。[构造](../../../raw/current/scripts/components/socketholder.lua#L10-L28) [存入](../../../raw/current/scripts/components/socketholder.lua#L268-L312)
- 优先使用：双端初始化一次 `MakeInstSocketHolder_Client` 配数量／名称，服务端配置门禁与装卸回调；正常装入用 `TryToSocket`，恢复／内部执行才用跳过门禁的 `DoSocket`。[helper](../../../raw/current/scripts/componentutil.lua#L2306-L2319) [门禁链](../../../raw/current/scripts/components/socketholder.lua#L315-L328)
- 重要副作用：装入保存并删除原物品，卸下重新 Spawn；装入回调在保存之前，卸下先清槽再回调。不能把 socketdata 当容器实体表或把 GUID 当持续身份。[装入](../../../raw/current/scripts/components/socketholder.lua#L279-L312) [卸下](../../../raw/current/scripts/components/socketholder.lua#L332-L348)
- 修改边界：`SetMaxSockets` 创建网络变量且只准调用一次，须在双端网络初始化、`SetPristine` 前完成；服务端数据方法不能在纯客户端直接调用。原版备用机体明确采用该顺序。[断言与netvar](../../../raw/current/scripts/components/socketholder.lua#L34-L47) [宿主](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L901-L902)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `MakeInstSocketHolder_Client(inst, names)` | table：按长度建槽并逐项设名字；string：建一个同名槽；number：只建指定数量的未命名槽，须后续激活。不因组件已经存在而跳过 SetMaxSockets，不可再次调用扩容。[helper](../../../raw/current/scripts/componentutil.lua#L2306-L2319) |
| `SetSocketPositionName(position, name)` | 服务端把旧的非零槽名改为不同值时先卸下并在宿主位置掉落；原版以 `0` 停用槽。名称决定可用性，不是只改显示文字。[改名与掉落](../../../raw/current/scripts/components/socketholder.lua#L50-L66) [原版停用](../../../raw/current/scripts/prefabs/wx78_common.lua#L771-L779) |
| `GetAllSocketPositions(name)`／`GetAllEmptySocketPositions(name)`／`GetAllFullSocketPositions(name)` | 名称可省略以取各已命名槽；从未找到候选时可返回 nil，过滤后也可返回空表。`GetFirstEmptySocketPosition`／`GetFirstFullSocketPosition` 返回位置或 nil。[列表与过滤](../../../raw/current/scripts/components/socketholder.lua#L71-L147) |
| `GetHighestQualitySocketed(name)`／`GetQualityForPosition(position)` | 前者按已装同名槽取最高品质，无匹配返回 NONE；后者在位置无 netvar 时也返回 NONE。位置集合版 `GetHighestQualitySocketedPositions` 返回最高品质的各槽。[品质查询](../../../raw/current/scripts/components/socketholder.lua#L152-L184) |
| `CanTryToSocket(item, doer)` | 检查 socketable、同名空槽和 CLIENT 回调；成功返回 `true,nil,position`，失败为 false 或 false,reason。该函数在服务端 TryToSocket 中也会调用，CLIENT 后缀不表示只在客户端运行。[定义](../../../raw/current/scripts/components/socketholder.lua#L195-L218) |
| `SetShouldAllowSocketableFn_CLIENT(fn)`／`SetShouldAllowSocketableFn_SERVER(fn)` | 两者 fn(holder,item,doer)；TryToSocket 先通用判断再 SERVER 判断。CLIENT 需使用两端可得信息；额外权威限制仍由 SERVER 执行，不把动作候选当授权。[通用检查](../../../raw/current/scripts/components/socketholder.lua#L195-L218) [服务端检查](../../../raw/current/scripts/components/socketholder.lua#L244-L245) [执行顺序](../../../raw/current/scripts/components/socketholder.lua#L315-L328) |
| `TryToSocket(item, doer)`／`DoSocket(item_or_savedata, doer, position)` | 前者选槽并检查，后者不重查槽名／容量／权限，直接拆堆或移出 owner、更新占用／品质、回调、保存并 Remove；position 必须对应已建槽，doer 可 nil。[实现](../../../raw/current/scripts/components/socketholder.lua#L279-L328) |
| `SetOnGetSocketableFn(fn)`／`SetOnRemoveSocketableFn(fn)` | getfn(holder,item,doer)，removefn(holder,item)；get 后发 `onsocketeditem`，remove 后发 `onunsocketeditem`，事件均发在宿主。装入时 netvar／metadata 已更新，socketdata 尚未写入。[setter](../../../raw/current/scripts/components/socketholder.lua#L248-L253) [get顺序](../../../raw/current/scripts/components/socketholder.lua#L297-L311) [remove顺序](../../../raw/current/scripts/components/socketholder.lua#L332-L345) |
| `UnsocketPosition(position)`／`UnsocketEverything()` | 前者重建物品并返回实体或 nil；后者返回重建成功的实体数组，不保留空槽编号。方法不自动给玩家，也不保证新实体落在宿主身边；由调用者放置／接收。[卸下](../../../raw/current/scripts/components/socketholder.lua#L332-L362) [原版放入库存](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28109-L28116) |
| `TryToUnsocket(position)` | 只立即发 `socketholder_unsocket` 请求事件，不直接卸下；客户端也可发。原版 classified 转 RPC，再由玩家 SG 真正取物，新宿主须自己实现适合的请求、授权和交付路径。[请求](../../../raw/current/scripts/components/socketholder.lua#L221-L224) [客户端转发](../../../raw/current/scripts/prefabs/wx78_classified.lua#L161-L172) [SG执行](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28109-L28116) |

## 典型接入

### 新实体需要固定镶嵌槽：按工厂与服务端效果分工

在目标模组自己的 Prefab 工厂组装，复用已有网络／资源／生命周期；这里提供经核验的接入位置与原版模式，不提供缺少 UI、RPC 与 SG 的伪完整 Prefab：

- 双端网络区一次性创建确定数量的槽。WX78 公共初始化先以数字 `1` 创建未命名槽，备用机体在 `SetPristine` 前调用公共初始化；只有技能／主人关系满足才激活 SHADOW 或 GESTALTTRAPPER 名称，未满足则设 0。不要给所有机体无条件预开暗影槽。[公共初始化](../../../raw/current/scripts/prefabs/wx78_common.lua#L1165-L1171) [工厂位置](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L901-L902) [技能选择](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L114-L125)
- 名称与物品 [[reference/components/socketable]] 一致；CLIENT 回调使用可见所有权信息筛候选，SERVER 回调另做自定义权限、距离或状态核验。WX78 的通用回调允许给自己装，或给 userid 匹配的 [[reference/components/linkeditem]] 装；这不是任意 mod 的默认授权策略。[原版判断](../../../raw/current/scripts/prefabs/wx78_common.lua#L598-L603) [服务端入口](../../../raw/current/scripts/components/socketholder.lua#L315-L328)
- 服务端 get／remove 回调根据当前槽状态刷新增益，避免永久保存瞬时物品引用；WX78 按最高暗影品质刷新组件／外观／无人机限制，GESTALTTRAPPER 则增删对应标记和效果。成对回调用来同时覆盖装入和卸下，恢复时 doer=nil 也必须安全。[安装回调](../../../raw/current/scripts/prefabs/wx78_common.lua#L1201-L1203) [增益与撤销](../../../raw/current/scripts/prefabs/wx78_common.lua#L715-L768)
- 独立拆卸 UI 不能只调用 TryToUnsocket 就结束：原版请求经 classified 的 RPC 到玩家，服务端 RPC 检查暗影技能，特定 SG 状态再按槽名取物并 GiveItem。新名字或非 WX78 宿主需自己接线，不能直接依赖该专用通路。[RPC](../../../raw/current/scripts/networkclientrpc.lua#L1325-L1365) [请求状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28019-L28022) [实际取物](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28109-L28116)

### 宿主销毁／角色重选时释放物品

服务端默认监听 death 并 `UnsocketEverything` 后 Launch2；可用 EnableDropOnDeath／DisableDropOnDeath 切换。WX78 玩家禁用默认监听，交给 SG 的死亡掉落流程；备用机体被锤毁与角色重选各自显式取出并掉落。自定义无死亡事件建筑被 Remove 时不能依赖默认死亡监听，须在实际拆除路径安排卸下和物品落点。[默认死亡](../../../raw/current/scripts/components/socketholder.lua#L1-L27) [开关](../../../raw/current/scripts/components/socketholder.lua#L230-L241) [玩家禁用](../../../raw/current/scripts/prefabs/wx78_common.lua#L1165-L1171) [玩家SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L237-L244) [机体锤毁](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L31-L48) [重选](../../../raw/current/scripts/prefabs/player_common.lua#L1113-L1119)

## 生命周期

1. **装入：** 堆叠材料只取一个，否则从 inventory owner 移除；填 metadata、占用／品质，调用 getfn 和事件，最后保存记录并 Remove 物品。回调可以读取物品当下字段；此时按 `socketdata[position]` 找刚插入物品会过早。[DoSocket](../../../raw/current/scripts/components/socketholder.lua#L279-L312)
2. **卸下：** 清 metadata／data／占用／品质，再以保存的 prefab、皮肤与持久数据生成新实体；成功才调用 removefn 和事件。重建失败返回 nil，槽内数据已经清除，不能假定失败仍保留原物品。[重建](../../../raw/current/scripts/components/socketholder.lua#L256-L265) [卸下](../../../raw/current/scripts/components/socketholder.lua#L332-L348)
3. **存档与恢复：** OnSave 只返回 socketdata；恢复逐槽重建物品并调用 DoSocket，`isloading=true`，getfn／事件重复执行且 doer=nil，然后再次保存删除。这是效果重建，不是离线活物品继续运行。[保存／恢复](../../../raw/current/scripts/components/socketholder.lua#L391-L401)
4. **玩家与非玩家分开：** 玩家在 OnLoad 恢复，技能握手 READY 时立即 VerifySockets，否则等 `ms_skilltreeinitialized`；非玩家在 LoadPostPass 恢复。加载中 GetAllSocketPositions 可借 metadata 认出尚未激活的已装槽；VerifySockets 发现名称不匹配会卸下并抛出，非玩家该钩子不自动调用 VerifySockets。[加载分支](../../../raw/current/scripts/components/socketholder.lua#L405-L421) [加载查询](../../../raw/current/scripts/components/socketholder.lua#L71-L85) [校正](../../../raw/current/scripts/components/socketholder.lua#L365-L383)

## 常见陷阱与限制

- **PostInit 扩容已初始化原版槽：** 第二次 SetMaxSockets 直接 assert，且新网络变量需要匹配双端初始化。改既有 WX78 槽数不是普通服务端 PostInit 数值配置；新宿主在工厂一开始确定数量。[断言](../../../raw/current/scripts/components/socketholder.lua#L34-L47)
- **把 socketed=true 当成活实体存在：** 物品已序列化并删除，slot netvar 只表示占用。需持续运转的物品／容器用 [[reference/components/container]] 等实体持有机制；此处持续效果由宿主回调重建。[序列化](../../../raw/current/scripts/components/socketholder.lua#L268-L311)
- **无校验直接 DoSocket 或不可信位置直接 UnsocketPosition：** 两者面向内部执行，不做完整位置、授权和重复装入保护；正常装入走 TryToSocket，网络拆卸先校验整数范围、槽类型和当前可操作状态，再调用内部方法。[装入实现](../../../raw/current/scripts/components/socketholder.lua#L279-L328) [卸下实现](../../../raw/current/scripts/components/socketholder.lua#L332-L348)
- **改槽名只想隐藏 UI 却导致掉物：** 服务端变更原非零名称先卸下；停用、切技能、重建效果时都应接受这一副作用，不能用改名保留不匹配材料。[SetSocketPositionName](../../../raw/current/scripts/components/socketholder.lua#L50-L66)
- **恢复回调依赖玩家或同时增删插槽：** getfn 在每个恢复槽的序列化完成前执行，doer=nil；保持幂等的效果刷新并遵守宿主技能初始化顺序，勿把初始化当一次新的玩家支付行为。[恢复流程](../../../raw/current/scripts/components/socketholder.lua#L297-L311) [加载调用](../../../raw/current/scripts/components/socketholder.lua#L395-L414)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 7 处（componentutil +231、SGwilson 取物／请求状态 +166 与玩家死亡掉落 +12，区间文本逐段比对一致）；结论修订：无（networkclientrpc 与 player_common 引用区间未变）。
- 依据：player-hubs hunk 地图（SB-17）＋变更清单命中 componentutil.lua、stategraphs/SGwilson.lua、networkclientrpc.lua、prefabs/player_common.lua。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；覆盖组件完整实现、双端 helper、WX78／备用机体构造与回调、玩家／非玩家加载、死亡／锤毁／重选释放、拆卸 classified／RPC／SG 链，依据见上述锚点。
- 游戏运行：未运行；未验证自定义槽 UI／RPC、多人并发、跨分片、皮肤恢复或自定义物品存档。本文是原版组装模式，无完整 Lua demo。
- 结构检查：本页元数据、大小和链接、受影响导航及全库登记检查通过；源码引用 strict 检查无错误／警告，不代表游戏行为通过。

相关：[[reference/components/socketable]]、[[reference/components/useabletargeteditem]]、[[reference/components/linkeditem]]。
