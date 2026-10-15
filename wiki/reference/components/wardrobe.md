---
title: "wardrobe：衣柜会话与皮肤提交"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "管理衣柜使用资格、玩家弹窗和换装状态，维护距离退出及共享／目标装扮分支，分层处理皮肤提交检查。"
aliases: [wardrobe, 衣柜换装, 多人衣柜, 换装会话, 稻草人装扮, ApplySkins]
tags: [component, lifecycle, callback, ui, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/wardrobe.lua, prefabs/wardrobe.lua, prefabs/scarecrow.lua, actions.lua, componentactions.lua, stategraphs/SGwilson.lua, popupmanager.lua, networkclientrpc.lua, prefabs/player_common.lua, screens/playerhud.lua, screens/redux/wardrobepopupgridloadout.lua, screens/redux/scarecrowpopupgridloadout.lua, skinsutils.lua]
reliability: verified
validation: source-reviewed
status: active
---

# wardrobe：衣柜会话与皮肤提交

## 快速判断

- 职责与适用：挂在衣柜／可装扮宿主上，保存 changers 会话集合，推动玩家 SG、弹窗提交和 [[reference/components/skinner]] 应用。它不保存皮肤内容，也不独立实现客户端选择界面。[wardrobe.lua:18-57](../../../raw/current/scripts/components/wardrobe.lua#L18-L57)、[wardrobe.lua:124-174](../../../raw/current/scripts/components/wardrobe.lua#L124-L174)
- 优先使用：沿 CHANGEIN → CanBeginChanging → BeginChanging 的原版动作协议；给目标装扮用 SetCanBeDressed 与宿主回调，不直接调用 ApplySkins 冒充一次完整衣柜交互。[actions.lua:2800-2824](../../../raw/current/scripts/actions.lua#L2800-L2824)、[wardrobe.lua:186-258](../../../raw/current/scripts/components/wardrobe.lua#L186-L258)
- 重要副作用：使用者进入 openwardrobe，控制／背包／动作显示由 SG 暂停；关闭会话会发 onskinschanged，即便取消或没有差异。该事件不是“皮肤成功更换”的证明。[SGwilson.lua:6785-6819](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6785-L6819)、[wardrobe.lua:152-174](../../../raw/current/scripts/components/wardrobe.lua#L152-L174)
- 修改边界：原版衣柜仅服务端添加组件；BeginChanging 不自行调用资格检查，ActivateChanging 也不独立验证 changers 成员资格，低层方法不能脱离会话／SG／库存协议作为公共换肤入口。[prefabs/wardrobe.lua:107-118](../../../raw/current/scripts/prefabs/wardrobe.lua#L107-L118)、[wardrobe.lua:124-150](../../../raw/current/scripts/components/wardrobe.lua#L124-L150)、[wardrobe.lua:235-246](../../../raw/current/scripts/components/wardrobe.lua#L235-L246)

## 关键接口

| 接口／字段 | 默认值或参数 | 条件、副作用与证据 |
|---|---|---|
| `CanBeginChanging(doer)` | `true` 或 `false, reason?` | enabled=false 返回 INUSE；拒绝已登记、无 SG、busy 且非 opengift，燃烧返回 BURNING，非共享已有他人返回 INUSE；不检查距离、视线或 skinner。[wardrobe.lua:109-122](../../../raw/current/scripts/components/wardrobe.lua#L109-L122) |
| `BeginChanging(doer)` | 新会话 true，已登记 false | 仅防重复，登记使用者与 onremove／ms_closepopup 监听，进入 SG；首人开始更新并调用 onopenfn(inst)。调用方先检查资格。[wardrobe.lua:124-150](../../../raw/current/scripts/components/wardrobe.lua#L124-L150) |
| `EndChanging(doer)`／`EndAllChanging()` | 结束一个／全部 | 撤监听和成员、发 onskinschanged；必要时出 inwardrobe 状态，最后一人离开停更并 onclosefn(inst)。不回滚已应用皮肤。[wardrobe.lua:152-184](../../../raw/current/scripts/components/wardrobe.lua#L152-L184) |
| `Enable(enable)` | 初始 true；仅传 false 才禁用 | 只改字段，不立即 EndAllChanging；OnUpdate 也不因 enabled=false 自动结束旧会话。[wardrobe.lua:68-70](../../../raw/current/scripts/components/wardrobe.lua#L68-L70)、[wardrobe.lua:302-317](../../../raw/current/scripts/components/wardrobe.lua#L302-L317) |
| `SetCanUseAction(value)`／`SetCanBeDressed(value)` | 初始 true／nil | 属性回调增删 wardrobe／dressable 标签；前者影响动作收集，不是 CanBegin 的资格门；后者切到给宿主装扮分支。[wardrobe.lua:1-16](../../../raw/current/scripts/components/wardrobe.lua#L1-L16)、[wardrobe.lua:60-66](../../../raw/current/scripts/components/wardrobe.lua#L60-L66) |
| `SetCanBeShared(value)` | 构造设 false；仅 true 启用 | 允许多人并改成柜外换装；共享时还撤本组件 onignite 监听，恢复独占时重新监听。[wardrobe.lua:33](../../../raw/current/scripts/components/wardrobe.lua#L33)、[wardrobe.lua:90-99](../../../raw/current/scripts/components/wardrobe.lua#L90-L99) |
| `SetRange(range)`／`SetChangeInDelay(delay)` | 默认3世界距离／0秒 | 前者用于持续距离检查；后者传给独占 changeinwardrobe 的等待时间，无共享延迟或范围输入校验。[wardrobe.lua:26-27](../../../raw/current/scripts/components/wardrobe.lua#L26-L27)、[wardrobe.lua:101-107](../../../raw/current/scripts/components/wardrobe.lua#L101-L107)、[wardrobe.lua:221-230](../../../raw/current/scripts/components/wardrobe.lua#L221-L230) |

## 关键流程

### 进入、双端弹窗与提交

1. 动作收集需要 wardrobe 标签、非 fire，目标可装扮时还涉及右键条件；CHANGEIN 先 CanBeginChanging，通过后只有 CanEntitySeeTarget 才 BeginChanging。黑暗中不能看见目标时动作仍返回 true，但不打开会话；不能用动作布尔值单独证明 UI 已打开。[componentactions.lua:959-963](../../../raw/current/scripts/componentactions.lua#L959-L963)、[actions.lua:2800-2824](../../../raw/current/scripts/actions.lua#L2800-L2824)
2. Begin 登记玩家并进入 openwardrobe。普通换自己不传 target；可装扮时把宿主作为 target。SG 暂停预测、控制、背包等，再 ShowPopUp(WARDROBE)。玩家 ShowPopUp 服务端发客户端 RPC；客户端调用 popup.fn，交 HUD 打开普通服装或稻草人选择屏。[wardrobe.lua:124-150](../../../raw/current/scripts/components/wardrobe.lua#L124-L150)、[SGwilson.lua:6785-6819](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6785-L6819)、[player_common.lua:1561-1565](../../../raw/current/scripts/prefabs/player_common.lua#L1561-L1565)、[networkclientrpc.lua:1409-1415](../../../raw/current/scripts/networkclientrpc.lua#L1409-L1415)、[playerhud.lua:519-554](../../../raw/current/scripts/screens/playerhud.lua#L519-L554)
3. 普通 UI Close 根据在线／离线皮肤支持、名称有效性和本地 CheckOwnership 整理五个字段，再调用 POPUPS.WARDROBE:Close；客户端发 ClosePopup RPC，主模拟端则直接发 ms_closepopup。RPC 只验证 popup 编号及各参数为可选字符串；失败仍无参数 Close，触发取消路径，不代表对所有皮肤权限做完校验。[wardrobepopupgridloadout.lua:202-222](../../../raw/current/scripts/screens/redux/wardrobepopupgridloadout.lua#L202-L222)、[popupmanager.lua:11-19](../../../raw/current/scripts/popupmanager.lua#L11-L19)、[popupmanager.lua:105-117](../../../raw/current/scripts/popupmanager.lua#L105-L117)、[networkclientrpc.lua:1073-1087](../../../raw/current/scripts/networkclientrpc.lua#L1073-L1087)
4. 会话监听器只接 WARDROBE popup，从参数还原 skins；外层 onclosewardrobe 先检查 changers[doer]，再 ActivateChanging。Activate 要求 skins 非空、当前 openwardrobe、doer 有 skinner；自身不重新调用 CanBegin，也不查登记，失败由外层 EndChanging 收口。[wardrobe.lua:35-51](../../../raw/current/scripts/components/wardrobe.lua#L35-L51)、[wardrobe.lua:235-246](../../../raw/current/scripts/components/wardrobe.lua#L235-L246)

### 换自己：差异、应用检查与时间点

DoDoerChanging 先把不属于 GetCharacterSkinBases(doer.prefab) 的 base 回退为角色 none，再将非 nil 且不同于旧值的字段组成 diff；无差异返回 false，外层关闭。缺字段意味着不改，空字符串可表示脱掉该槽。[wardrobe.lua:198-217](../../../raw/current/scripts/components/wardrobe.lua#L198-L217)、[skinsutils.lua:1460-1471](../../../raw/current/scripts/skinsutils.lua#L1460-L1471)

独占模式先 ApplySkins，再进入 changeinwardrobe(changeindelay)，随后 onchangeinfn(inst)；共享模式则进入 changeoutsidewardrobe，把 ApplySkins 作为回调在 SG 时间线或中断 onexit 中执行。ApplySkins 的检查是基础皮肤须有 Prefabs 条目、服装先清对应槽再在 CLOTHING 有条目时交 SetClothing；此方法本身不检查会话、角色 base 范围、所有权或服装 type 是否等于提交字段的槽。SetClothing 的进一步限制见 [[reference/components/skinner#关键接口]]，不能把此方法当不可信输入的完整校验器。[wardrobe.lua:219-233](../../../raw/current/scripts/components/wardrobe.lua#L219-L233)、[wardrobe.lua:260-296](../../../raw/current/scripts/components/wardrobe.lua#L260-L296)、[SGwilson.lua:6941-6976](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6941-L6976)

### 换目标：宿主回调与皮肤分工

canbedressed 时进入 dressupwardrobe，SG 触发回调后调用 ondressupfn(宿主inst, apply_callback)，或直接 ApplyTargetSkins。宿主回调可延迟效果，但须执行 apply_callback 才应用。ApplyTargetSkins 先用 doer.userid 调目标 AnimState:AssignItemSkins，再 ClearAllClothing／逐项 SetClothing，最后向目标发 dressedup；这里不换目标基础角色皮肤。[wardrobe.lua:186-196](../../../raw/current/scripts/components/wardrobe.lua#L186-L196)、[wardrobe.lua:248-258](../../../raw/current/scripts/components/wardrobe.lua#L248-L258)

目标 UI 取消会无参数 Close，确认才在有效性／拥有权整理后提交；服装管理和库存／引擎皮肤校验是不同层。本文未核验引擎 AssignItemSkins 的全部授权内部，不把直接调用 SetSkin／ApplySkins 作为绕过原版流程的接入方式。[scarecrowpopupgridloadout.lua:155-181](../../../raw/current/scripts/screens/redux/scarecrowpopupgridloadout.lua#L155-L181)

### 距离、可见性、退出与持久化

有 changers 时逐更新检查使用者 IsNear(range) 且 CanEntitySeeTarget，不满足则 EndChanging；无成员停更。独占衣柜监听 onignite 结束全部会话并给仍在 openwardrobe 的玩家提示；共享模式没有该组件监听。EndAll 先复制成员再逐个结束，避免遍历中修改表。[wardrobe.lua:72-99](../../../raw/current/scripts/components/wardrobe.lua#L72-L99)、[wardrobe.lua:176-184](../../../raw/current/scripts/components/wardrobe.lua#L176-L184)、[wardrobe.lua:302-317](../../../raw/current/scripts/components/wardrobe.lua#L302-L317)

SG 退出 openwardrobe 会关 UI，并根据 ischanging 等标记决定立即恢复控制还是交给后续状态；最终换装状态恢复控制／背包／动作并 Close popup。退出不是皮肤回滚：changeoutsidewardrobe 与 dressupwardrobe 的 onexit 仍会执行尚未执行的应用回调，宿主自排的延迟也须自己管理。[SGwilson.lua:6836-6855](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6836-L6855)、[SGwilson.lua:6890-6911](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6890-L6911)、[SGwilson.lua:6958-6976](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6958-L6976)、[SGwilson.lua:7013-7031](../../../raw/current/scripts/stategraphs/SGwilson.lua#L7013-L7031)

组件移除时 EndAll、撤 onignite 监听及两个动作标签；宿主实体删除也映射到 EndAll。没有 OnSave／OnLoad 保存 changer 或会话，服装由对象自己的 skinner 保存，重载不自动续开弹窗。[wardrobe.lua:321-330](../../../raw/current/scripts/components/wardrobe.lua#L321-L330)

## 典型接入

### 原版衣柜与稻草人的不同宿主职责

普通 wardrobe prefab 在 pristine 前添加动作标签，服务端设20帧柜内延迟和开／关／换装动画回调；受敲击时 EndAllChanging，砸毁则移除实体。保留这些宿主退出路径与原 SG，不只复制组件字段。[prefabs/wardrobe.lua:49-58](../../../raw/current/scripts/prefabs/wardrobe.lua#L49-L58)、[prefabs/wardrobe.lua:94-125](../../../raw/current/scripts/prefabs/wardrobe.lua#L94-L125)

scarecrow 服务端设置 canbedressed、ondressupfn，并给 skinner 调 SetupNonPlayerData；装扮回调安排自身变换时间后才调用 apply_callback，暂时 Enable(false) 阻止新会话。它还消费 dressedup 保存头像数据。这是目标装扮参照，不是普通衣柜改玩家 base 的分支。[scarecrow.lua:84-123](../../../raw/current/scripts/prefabs/scarecrow.lua#L84-L123)、[scarecrow.lua:185-216](../../../raw/current/scripts/prefabs/scarecrow.lua#L185-L216)

## 常见陷阱与限制

- **关闭动作入口等于结束旧会话**：SetCanUseAction(false) 只删动作标签；Enable(false) 只影响新 CanBegin。需要结束使用者显式 EndAllChanging，共享切换也会改变燃烧监听，不能把它只当人数开关。[wardrobe.lua:60-99](../../../raw/current/scripts/components/wardrobe.lua#L60-L99)
- **ApplySkins 返回就是已授权换装成功**：它没有成功布尔值及完整验证；无效服装字段会先清旧槽，字段槽位也不保证与服装 type 相符。沿原会话、UI／库存和 skinner 协议处理可信选择，不直接接受任意字典。[wardrobe.lua:260-296](../../../raw/current/scripts/components/wardrobe.lua#L260-L296)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖 wardrobe 全文、CHANGEIN 资格与动作入口、标准衣柜／稻草人宿主、SGwilson 四类换装状态、服务端 ShowPopup／客户端 HUD／两类选择屏与 ClosePopup 参数协议，以及角色 base 集合查询。
- 游戏运行：未运行；未测多人竞态、共享／燃烧／距离中断、宿主延迟回调、离线皮肤、权限变化和控制恢复；未展开引擎授权内部、其他特殊宿主或全部皮肤资产。无新增 Lua 示例。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；未运行语法探针。

相关：[[reference/components/skinner]]、[[reference/components/skinner-beefalo]]。
