---
title: "skinner：角色皮肤、服装与模式重绘"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "维护角色基础皮肤、四类服装与显示模式，通过原版重绘和存读协议同步外观，区分名称合法性与拥有权。"
aliases: [skinner, 角色换肤, 服装槽, SetSkinMode, SetClothing, 皮肤模式恢复]
tags: [component, lifecycle, callback, event, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/skinner.lua, skinsutils.lua, constants.lua, prefabs/player_common.lua, components/mightiness.lua, prefabs/scarecrow.lua]
reliability: verified
validation: source-reviewed
status: active
---

# skinner：角色皮肤、服装与模式重绘

## 快速判断

- 职责与适用：保存基础皮肤名、body／hand／legs／feet 四槽服装和 skintype，按模式重绘 AnimState 并调用 Network:SetPlayerSkin；不是装备槽、皮肤物品注册或拥有权授予器。牛的五部位装扮归 [[reference/components/skinner-beefalo]]。[skinner.lua:1-9](../../../raw/current/scripts/components/skinner.lua#L1-L9)、[skinner.lua:459-483](../../../raw/current/scripts/components/skinner.lua#L459-L483)
- 优先使用：正常玩家换装沿 [[reference/components/wardrobe]] 会话提交；形态系统用 SetSkinMode 选择已有皮肤的模式。直接 SetSkinName／SetClothing 是内部状态／渲染入口，不能取代库存拥有权与原版选择流程。[skinner.lua:577-650](../../../raw/current/scripts/components/skinner.lua#L577-L650)
- 重要副作用：服装 setter 会先发卸下／穿上事件，再重绘；重绘会清服装符号覆盖、交换顺序和部分隐藏状态。自定义 OverrideSymbol 不能假设跨模式切换保持不变。[skinner.lua:43-56](../../../raw/current/scripts/components/skinner.lua#L43-L56)、[skinner.lua:614-629](../../../raw/current/scripts/components/skinner.lua#L614-L629)
- 修改边界：玩家公共构造在服务端添加；SetSkinMode 直接使用宿主 AnimState 和 Network，非玩家也须有对应接线。组件没有专用移除回滚或持续更新任务，删除组件不会由本组件恢复旧服装。[player_common.lua:2613-2615](../../../raw/current/scripts/prefabs/player_common.lua#L2613-L2615)、[player_common.lua:2834](../../../raw/current/scripts/prefabs/player_common.lua#L2834)、[skinner.lua:459-483](../../../raw/current/scripts/components/skinner.lua#L459-L483)

## 关键接口

| 接口／字段 | 参数、返回／默认值 | 条件、副作用与证据 |
|---|---|---|
| `SetSkinName(name, skip_beard_setup?, skip_skins_set?)` | 空字符串转 `prefab.."_none"` | 保存名称，重建 skin_data 并从 Prefabs[name].skins 复制模式表；缺 normal_skin 仅打印提示，不完成统一名称／拥有权校验。默认处理匹配胡须，最后 SetSkinMode；第三参真才跳重绘。[skinner.lua:577-612](../../../raw/current/scripts/components/skinner.lua#L577-L612) |
| `SetSkinMode(skintype?, default_build?)`／`GetSkinMode()` | 初始 normal_skin；省略模式沿用当前值 | 缺 skin_data 会初始化角色默认皮肤；按模式找基础 build，重绘后调用无参数 base_change_cb，再发布 Network 皮肤字段。default_build 仅本次参数，不保存成字段。[skinner.lua:455-483](../../../raw/current/scripts/components/skinner.lua#L455-L483) |
| `SetClothing(name)` | CLOTHING 中有效非默认服装名 | IsValidClothing 只查非空、已登记、非 is_default，不查用户拥有权；槽位取 CLOTHING[name].type。空字符串不会清除当前槽，也不返回成功布尔值。[skinner.lua:646-650](../../../raw/current/scripts/components/skinner.lua#L646-L650)、[skinsutils.lua:1488-1490](../../../raw/current/scripts/skinsutils.lua#L1488-L1490) |
| `GetClothing()` | 新表 `{base,body,hand,legs,feet}` | base 来自 skin_name；返回表是副本，修改它不直接改变组件。[skinner.lua:652-660](../../../raw/current/scripts/components/skinner.lua#L652-L660) |
| `ClearClothing(type)`／`ClearAllClothing()` | 单槽／全部当前槽 | 单槽走事件并立即重绘；全部逐槽发事件后只统一重绘一次。ClearClothing 没有槽名校验，接入只使用已核验四槽。[skinner.lua:614-629](../../../raw/current/scripts/components/skinner.lua#L614-L629)、[skinner.lua:672-681](../../../raw/current/scripts/components/skinner.lua#L672-L681) |
| `HideAllClothing(anim_state)` | 传入动画状态 | 只清当前服装配置涉及的符号覆盖，不改 clothing、事件或存档，也不等于持久脱装。[skinner.lua:662-670](../../../raw/current/scripts/components/skinner.lua#L662-L670) |
| `SetMonkeyCurse(value)`／`ClearMonkeyCurse()` | 记录／清空后重绘 | 由渲染辅助按诅咒阶段覆盖部分猴子符号；不会据此改变人物实际形态／能力。[skinner.lua:16-41](../../../raw/current/scripts/components/skinner.lua#L16-L41)、[skinner.lua:631-643](../../../raw/current/scripts/components/skinner.lua#L631-L643) |

## 关键流程

### 模式重绘与事件顺序

SetSkinMode 先确定当前模式。ghost_skin 的 build 依次回退为 skin_data 对应项、inst.ghostbuild、本次 default_build、`ghost_<prefab>_build`；其他模式回退为对应项、default_build、prefab。名称可回退不代表任意自造模式都具备服装符号适配。[skinner.lua:459-477](../../../raw/current/scripts/components/skinner.lua#L459-L477)

SetSkinsOnAnim 除 NO_BASE 外先 SetSkin；随后清 CLOTHING_SYMBOLS、ClearSymbolExchanges 并显示 HIDE_SYMBOLS，再仅对常量列表允许的模式应用服装。常见 normal／wimpy／mighty、成长／年龄等模式及 NO_BASE 在列表中，ghost_skin 不在；服装基本处理顺序为 legs→body→feet→hand，另有身体适配和符号排序分支，本页不逐件展开。[skinner.lua:9-14](../../../raw/current/scripts/components/skinner.lua#L9-L14)、[skinner.lua:43-56](../../../raw/current/scripts/components/skinner.lua#L43-L56)、[constants.lua:2764-2776](../../../raw/current/scripts/constants.lua#L2764-L2776)

重绘完成后才 base_change_cb()，再 Network:SetPlayerSkin。服装内部 setter 则先对旧非空名称发 unequipskinneditem、赋新值，再对新非空名发 equipskinneditem 和成就，最后按标志重绘；同一名字重复 SetClothing 也没有跳过这些步骤。因此回调观察点与“网络字段已发布”不能混为一谈。[skinner.lua:477-483](../../../raw/current/scripts/components/skinner.lua#L477-L483)、[skinner.lua:614-629](../../../raw/current/scripts/components/skinner.lua#L614-L629)

### 保存、恢复与附加外观

OnSave 返回 skin_name、当前 clothing 表、monkey_curse、skin_mode、copiedplayer_data 和 skinoverrides。OnLoad 先恢复诅咒与复制数据，再直接接管保存的 clothing；只有 InGamePlay() 且宿主有 userid 时，逐项清除 CheckClientOwnership 不通过的衣物。快照世界加载阶段特意跳过该拥有权复核，不能把 OnLoad 当任意外部输入的通用验证函数。[skinner.lua:811-845](../../../raw/current/scripts/components/skinner.lua#L811-L845)

基础皮肤恢复中，NON_PLAYER 走 SetupNonPlayerData（NO_BASE）；其余先以 `prefab.."_none"` 为回退，再按是否处于游戏及拥有权条件接纳保存皮肤。只有 useskintypeonload 为真才直接采用 data.skin_mode，默认不会无条件恢复该字段。随后 SetSkinName(skin_name, true)：跳胡须匹配，但仍重绘并调用 base_change_cb／发布网络；衣物是直接赋表，未逐项重发穿脱事件。[skinner.lua:571-612](../../../raw/current/scripts/components/skinner.lua#L571-L612)、[skinner.lua:834-862](../../../raw/current/scripts/components/skinner.lua#L834-L862)

最后才恢复 skinoverrides；无保存值时还有 WX-78 的特定回退分支。SetSkinOverrides 先对旧列表执行 onclearfn，再按游戏／拥有权、角色和禁用标签规则筛新项，逐项 oninitfn，最后保存通过的列表。它是附加外观系统，不是第五个服装槽；其详细数据配置不在本页范围。[skinner.lua:485-545](../../../raw/current/scripts/components/skinner.lua#L485-L545)、[skinner.lua:864-871](../../../raw/current/scripts/components/skinner.lua#L864-L871)

### 非玩家和复制外观的边界

SetupNonPlayerData 把 skin_name 设 NON_PLAYER、skin_data 设空表，再用 NO_BASE 重绘；跳过的是基础 SetSkin，服装与网络调用仍有前提。原版稻草人在服务端同时添加 wardrobe 与 skinner，并显式走这个入口。[skinner.lua:571-575](../../../raw/current/scripts/components/skinner.lua#L571-L575)、[skinner.lua:43-56](../../../raw/current/scripts/components/skinner.lua#L43-L56)、[scarecrow.lua:185-213](../../../raw/current/scripts/prefabs/scarecrow.lua#L185-L213)

CopySkinsFromPlayer／SwapCopiedPlayerSkins 是另一路专用流程：借 userid 调 AssignItemSkins，再用玩家 prefab／模式重绘并直接写服装及 copiedplayer_data；不等于逐项 SetClothing 的事件协议。本文只辨认该职责与保存字段，不把它发布为任意实体复制玩家外观的接入示例。[skinner.lua:683-721](../../../raw/current/scripts/components/skinner.lua#L683-L721)、[skinner.lua:723-805](../../../raw/current/scripts/components/skinner.lua#L723-L805)

## 典型接入

### 玩家基础皮肤与力量形态分别驱动

原版玩家公共构造服务端已有 skinner；普通衣柜通过 wardrobe 的提交／状态图链改变名称和服装，详见 [[reference/components/wardrobe#关键流程]]。沃尔夫冈 Mightiness:UpdateSkinMode 则根据宿主提供的 skin_data 调 SetSkinMode(skin_mode, default_build)，可延迟到形态切换时执行；皮肤名称、服装与当前身体模式因此应分开维护，不能靠存档 skin_mode 就替代宿主形态重绘。[player_common.lua:2834](../../../raw/current/scripts/prefabs/player_common.lua#L2834)、[mightiness.lua:270-286](../../../raw/current/scripts/components/mightiness.lua#L270-L286)

## 常见陷阱与限制

- **有效名就是已授权**：IsValidClothing 没查库存，SetSkinName 对基础皮肤也不做完整拥有权流程；它只在匹配胡须时单独检查 linked_beard。正常用户换装应沿 wardrobe／库存流程，不使用本地 SetSkin 或直接 setter 作为授权替代。[skinsutils.lua:1488-1490](../../../raw/current/scripts/skinsutils.lua#L1488-L1490)、[skinner.lua:577-612](../../../raw/current/scripts/components/skinner.lua#L577-L612)
- **清皮肤和删除显示等价**：HideAllClothing 不改保存数据，后续 SetSkinMode 可重新覆盖；清服装须使用对应 Clear 方法，并考虑宿主已有 base_change_cb 和形态接线。不要覆盖回调或移除组件来假设恢复所有旧符号。[skinner.lua:662-681](../../../raw/current/scripts/components/skinner.lua#L662-L681)、[skinner.lua:477-483](../../../raw/current/scripts/components/skinner.lua#L477-L483)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件整体实现及核心皮肤／服装／模式、动画清理顺序、保存恢复与回调时序；核验服装有效性、接收服装的模式常量、玩家服务端安装、mightiness 模式消费者和稻草人非玩家安装。
- 游戏运行：未运行；未逐件核验服装／角色分支的数据与美术结果，未测试所有权变更、快照恢复、非玩家复制／交换、形态延迟和多人显示；引擎库存／皮肤授权内部、idle animation 与尾巴专用消费未展开。无新增 Lua 示例。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；未运行语法探针。

相关：[[reference/components/wardrobe]]、[[reference/components/skinner-beefalo]]。
