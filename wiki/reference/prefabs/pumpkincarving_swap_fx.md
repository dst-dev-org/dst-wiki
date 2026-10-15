---
title: "南瓜雕刻换装代理（pumpkincarving_swap_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "UGC 换装的 CLASSIFIED 数据代理实体：net_string 携带南瓜雕刻数据，挂到穿戴者／健身房／绞车／展示架后由 ApplyCuts 生成非联网切口 pieces 贴在宿主 swap_body 符号上；同文件同工厂注册 snowmandecorating_swap_fx（雪人装饰）。"
aliases: ["pumpkincarving_swap_fx", "snowmandecorating_swap_fx", "ugc_swap_fx", "南瓜雕刻", "雪人装饰", "UGC 换装", "换装代理"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/ugc_swap_fx.lua", "components/pumpkincarvable.lua", "components/snowmandecoratable.lua", "components/mightygym.lua", "prefabs/winch.lua", "prefabs/trophyscale_oversizedveggies.lua", "prefabs/veggies.lua", "prefabs/snowman.lua", "prefablist.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 南瓜雕刻换装代理（pumpkincarving_swap_fx）

## 快速判断

- 职责与适用：玩家自定义内容（UGC）换装的**数据代理实体**：本体无动画无资源，CLASSIFIED＋colouraddersync＋net_string 数据通道；可见部分是 ApplyCuts／ApplyDecor 在宿主身上生成的非联网 pieces（南瓜雕刻切口／雪人装饰物）。同文件 MakeUgcSwapFx 工厂注册两个兄弟名。[工厂](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L46-L88)
- 优先使用：查"雕刻南瓜穿戴显示／健身房杠铃片雕刻／雪人装饰显示"选本页；雕刻编辑、数据校验与上限在 pumpkincarvable／snowmandecoratable 组件页。
- 重要副作用：`SetData` 触发三档宿主判断（玩家 swap_body 帧 6／gym swap_item 帧 6／其余 swap_body 帧 13）后按端应用 pieces；南瓜路径另在宿主符号上做昼夜 LightOverride 并随移除恢复。[OnDataDirty](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L16-L38)
- 修改边界：pieces 只在非专服本地生成（每端各自按 net_string 数据重建）；本体 persists=false 不入存档，移除时 pieces 全清。[端分界](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L34-L36)、[清理](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L10-L14)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `pumpkincarving_swap_fx` | MakeUgcSwapFx：Transform＋Network＋CLASSIFIED → colouraddersync＋pieces 表＋net_string data（"datadirty"）＋net_bool flag → SetPristine → 客户端监听 datadirty 早退 → 主模拟暴露 SetData＋persists=false → master_postinit 把 _ondatadirtyfn 换成 server 版（追加昼夜 LightOverride 与 OnRemove 恢复） | apply 调 `PumpkinCarvable.ApplyCuts(data, pieces, owner, followsymbol, swapframe, 0, -145)`；无 common_postinit。[注册](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L137)、[server 链](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L97-L121) |
| `snowmandecorating_swap_fx` | 同工厂；common_postinit 增加 net_tinybyte size（默认 `STACK_IDS.large`），无 master_postinit | apply 调 `SnowmanDecoratable.ApplyDecor(..., inst.size:value(), ...)`；无昼夜逻辑。[注册](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L138)、[size](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L130-L133) |

页名取消费面更广的 `pumpkincarving_swap_fx`。两注册名均无 NAMES 条目，标题中文为已核验效果描述，非官方译名。注意文件名 ugc_swap_fx.lua 与 prefablist 登记的 "ugc_swap_fx"（[L1323](../../../raw/current/scripts/prefablist.lua#L1323)）都不是实际注册名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 数据通道 | `data` net_string 携带压缩编码后的雕刻／装饰数据；`flag` net_bool 由调用方自定义（健身房传 `slot == 2` 区分杠铃片槽位）；雪人另有 size | SetData 仅主模拟暴露；客户端靠 datadirty 事件应用 | [工厂内](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L58-L59)、[SetData](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L40-L44) |
| 宿主三档 | owner＝`entity:GetParent()`：isplayer→`swap_body` 帧 6（heavylifting）；`HasTag("gym")`→按 flag 选 `swap_item2`／`swap_item` 帧 6；其余（展示架／绞车）→`swap_body` 帧 13 | 必须先 SetParent，否则 owner＝nil 全程不应用 | [OnDataDirty](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L16-L38) |
| pieces 生成 | ApplyCuts 解码后两遍放置：切口实体（bank/build `farm_plant_pumpkin`，非联网 FX）经 `Follower:FollowSymbol` 贴到宿主符号的指定帧；雪人 ApplyDecor 按 ITEM_DATA 哈希放置装饰 | 只在 not IsDedicated 执行；专服无 pieces（纯数据同步） | [ApplyCuts](../../../raw/current/scripts/components/pumpkincarvable.lua#L225-L241)、[CreateCut](../../../raw/current/scripts/components/pumpkincarvable.lua#L129-L177)、[FollowSymbol](../../../raw/current/scripts/components/pumpkincarvable.lua#L199-L223)、[ApplyDecor](../../../raw/current/scripts/components/snowmandecoratable.lua#L223-L239) |
| 昼夜光效（南瓜路径） | server 版 OnDataDirty 后 WatchWorldState isday：夜间宿主 `inst.followsymbol` 设 SymbolLightOverride 0.12（普通宿主为 swap_body，gym 为 swap_item／swap_item2），白天清零；OnRemoveEntity_Server 恢复 0 | 仅主模拟；pieces 自身另有昼夜高亮处理（组件页） | [server 链](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L97-L117)、[常量](../../../raw/current/scripts/components/pumpkincarvable.lua#L39) |
| 换色同步 | 非专服在应用成功后注册 colouraddersync 回调：代理自身 colouraddersync 收到颜色变化后逐 piece 更新；宿主到代理的颜色传递需另有挂接，不是本文件监听宿主颜色 | 客户端与非专服服务端各自生效 | [ColourChanged](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L4-L8)、[挂接](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L34-L36) |
| 消费者 | 穿戴：pumpkincarvable／snowmandecoratable 的 OnEquipped_Server（有数据才生成、SetParent 到装备者、卸下 Remove）；健身房放杠铃片（flag＝slot==2）；绞车吊起雕刻重物（两处）；超大蔬菜展示架（两处，含读档恢复） | veggies 把本名列入可雕刻 oversized 蔬菜的 prefab 依赖；snowman 依赖 snowmandecorating_swap_fx | [南瓜组件](../../../raw/current/scripts/components/pumpkincarvable.lua#L283-L297)、[雪人组件](../../../raw/current/scripts/components/snowmandecoratable.lua#L309-L324)、[健身房](../../../raw/current/scripts/components/mightygym.lua#L147-L154)、[绞车](../../../raw/current/scripts/prefabs/winch.lua#L284-L291)、[第二处](../../../raw/current/scripts/prefabs/winch.lua#L416-L423)、[展示架](../../../raw/current/scripts/prefabs/trophyscale_oversizedveggies.lua#L226-L234)、[读档](../../../raw/current/scripts/prefabs/trophyscale_oversizedveggies.lua#L345-L349)、[依赖](../../../raw/current/scripts/prefabs/veggies.lua#L441)、[snowman](../../../raw/current/scripts/prefabs/snowman.lua#L10) |

## 典型接入

### 复用"自定义外观贴宿主符号"

- 前提与选择：主模拟生成后先 `SetParent` 再 `SetData`；数据格式须是组件侧校验过的压缩编码串，否则 apply 解码失败静默不出 pieces。
- 生效与撤销：SetData 内同步触发一次应用，数据变化重复 SetData 即可（ApplyCuts 先清旧 pieces）；宿主或 fx 移除时 pieces 全清。
- 原版依据：pumpkincarvable OnEquipped_Server 的三步（生成→SetParent→SetData）即完整接入（[实现](../../../raw/current/scripts/components/pumpkincarvable.lua#L283-L297)）。

```lua
-- 主模拟侧：把一段自定义外观数据挂到宿主实体上显示
local fx = SpawnPrefab("pumpkincarving_swap_fx")
fx.entity:SetParent(host.entity)  -- 必须先挂父级，OnDataDirty 用 GetParent 取宿主
fx:SetData(cutdata)               -- 第二参 flag 可选（gym 路径区分槽位）
```

- 调整与验证：swapframe 由宿主档位决定，自定义宿主会落入 else 档（swap_body 帧 13）；数据生成与上限校验见 [[reference/components/pumpkincarvable]]。

## 常见陷阱与限制

- **误区：本体是可见特效**：fx 实体没有 AnimState；可见切口是 ApplyCuts 生成的 pieces 实体，挂在宿主而非 fx 上，随宿主动画符号走。[pieces](../../../raw/current/scripts/components/pumpkincarvable.lua#L199-L223)
- **边界：必须 SetParent 且在 SetData 前**：OnDataDirty 用 `entity:GetParent()` 找宿主，未挂父级时 owner＝nil，跳过应用且南瓜昼夜逻辑不装。[OnDataDirty](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L16-L38)
- **边界：专服不生成 pieces**：`not TheNet:IsDedicated()` 才调 applyfn；专服上只有 net_string 数据流，表现由各客户端按数据重建。[端分界](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L34-L36)
- **边界："ugc_swap_fx" 不是注册名**：`SpawnPrefab("ugc_swap_fx")` 会失败；实际只有两个注册名（[注册行](../../../raw/current/scripts/prefabs/ugc_swap_fx.lua#L137-L138)）。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；139 行工厂全文（MakeUgcSwapFx、OnDataDirty 三档、南瓜 server 链、雪人 size）、pumpkincarvable ApplyCuts／CreateCut／_DoCut 与装备链、snowmandecoratable ApplyDecor 与装备链、mightygym／winch／trophyscale 消费点、veggies／snowman 依赖、prefablist 登记名、NAMES 检索（无命中）已核验；cutdata 压缩编码的字节布局细节、晚加入时序、pieces 各端重建的显示差异未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-material-index|采集、制作、碎裂与生成烟]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页 sources 所引文件原始字节本轮全部未变（哈希等价批量沿用，无语义修订），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-3）。
