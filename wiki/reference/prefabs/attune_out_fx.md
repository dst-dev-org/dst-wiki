---
title: "复活调谐得失特效（attune_out_fx）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "三个通用表注册承载玩家复活调谐的建立与解除表现：从无到有绑定肉像或温蒂复活墓时父挂 attune_out_fx，当前调谐 tag 已无来源时按幽灵态父挂 attune_in_fx 或 attune_ghost_in_fx；同一实际 tag 内换绑仅在 `_isrezattuned` 为 true 时静默，等价不同 tag 互换会清标志、使下一次同 tag 换绑重播 out；attune_out_fx 另被墓碑挖掘、装饰与温蒂墓碑合成三流程落点复用。"
aliases: ["attune_out_fx", "attune_in_fx", "attune_ghost_in_fx", "调谐特效", "调谐得失特效", "调谐建立特效", "调谐解除特效", "复活绑定特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "prefabs/gravestone.lua", "prefabs/wendy_recipe_gravestone.lua", "components/attuner.lua", "components/attunable.lua", "prefabs/attunable_classified.lua", "constants.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 复活调谐得失特效（attune_out_fx）

## 快速判断

- 职责与适用：`fx.lua` 通用表三注册（`attune_out_fx`／`attune_in_fx`／`attune_ghost_in_fx`），同 bank 同声效，承载玩家复活调谐（[[reference/components/attuner|attuner]] 机制）得失的表现；`attune_out_fx` 另被墓碑三流程落点复用。工厂协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目组](../../../raw/current/scripts/fx.lua#L1111-L1130)
- 优先使用：查"调谐绑定／解除特效、墓碑起灵表现"到本页；调谐机制本体读 [[reference/components/attunable|attunable]]（目标侧）与 [[reference/components/attuner|attuner]]（玩家侧），本页只管表现。
- 重要副作用：三注册均无 fn、无染色；玩家侧生成全部父挂玩家实体（随宿主移动），墓碑侧全部按世界坐标落点一次性。[玩家回调](../../../raw/current/scripts/prefabs/player_common.lua#L442-L470)
- 修改边界：生成点全部在服务端（组件事件回调）；三注册在简中 `.po` 无显示名称键，标题中文为效果描述非官方译名；动画素材内容与时长未核验。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `attune_out_fx` | 通用表条目，无 fn | bank／build=`attune_fx`，anim=`attune_out`，sound=`dontstarve/ghost/ghost_haunt`。[条目](../../../raw/current/scripts/fx.lua#L1111-L1116) |
| `attune_in_fx` | 同上协议，无 fn | 同 bank，anim=`attune_in`，同声效。[条目](../../../raw/current/scripts/fx.lua#L1118-L1123) |
| `attune_ghost_in_fx` | 同上协议，无 fn | 同 bank，anim=`attune_ghost_in`，同声效；与 `attune_in_fx` 仅动画名不同，选名条件见下。[条目](../../../raw/current/scripts/fx.lua#L1125-L1130) |

三注册在简中 `languages/chinese_s.po` 均无 `STRINGS.NAMES` 键（2026-10-31 定向检索无命中），标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 三注册均为代理＋一帧本地表现，动画结束自删；父挂式本地表现成为宿主子实体 | 落点式（墓碑侧）或父挂式（玩家侧）两路 | [[reference/mechanisms/generic-fx-factory]] |
| 生成者：玩家调谐监听 | `OnGotNewAttunement` 仅在 `_isrezattuned` 为 false 时按"从无到有"落 `attune_out_fx`；`OnAttunementLost` 仅在当前 tag（`remoteresurrector` 或 `gravestoneresurrector`）再无来源时按 `playerghost` tag 选 `attune_ghost_in_fx`／`attune_in_fx`，均 `entity:SetParent(玩家)` | 服务端：监听只在 `OnPlayerJoined` 的 mastersim 分支注册；`_isrezattuned` 在注册时按已有调谐预置 | [回调](../../../raw/current/scripts/prefabs/player_common.lua#L442-L470)、[注册窗口](../../../raw/current/scripts/prefabs/player_common.lua#L860-L866) |
| 触发链：attunable → classified → attuner | [[reference/components/attunable|attunable]] `LinkToPlayer` 创建专属 `attunable_classified` 代理父接玩家；代理注册进玩家 attuner 推 `gotnewattunement`，代理移除推 `attunementlost` | 服务端建链；客户端本地玩家代理也会注册，但本页监听仅服务端 | [LinkToPlayer](../../../raw/current/scripts/components/attunable.lua#L93-L118)、[代理注册／注销](../../../raw/current/scripts/prefabs/attunable_classified.lua#L1-L22)、[事件推送](../../../raw/current/scripts/components/attuner.lua#L64-L79) |
| 复用：墓碑三流程 | `gravestone` 挖除（`OnDugUp`）、装饰期定时落恶花（`try_evil_flower`）、装饰完成（`OnDecorated`）三处按坐标落点 `attune_out_fx` | 服务端；与调谐机制无关，借同一注册作墓碑侧"起灵"表现 | [挖除](../../../raw/current/scripts/prefabs/gravestone.lua#L67-L68)、[恶花](../../../raw/current/scripts/prefabs/gravestone.lua#L89-L100)、[装饰](../../../raw/current/scripts/prefabs/gravestone.lua#L124-L128)（gravestone 页 Wiki 待建） |
| 复用：温蒂墓碑合成 | `wendy_recipe_gravestone` 写完墓志铭先落点 `attune_out_fx`，再在原地生成 `gravestone`，代理随后移除 | 服务端；合成代理本体是可书写临时物 | [OnWritingEnded](../../../raw/current/scripts/prefabs/wendy_recipe_gravestone.lua#L28-L55)（Wiki 待建） |
| 资源 | bank／build=`attune_fx`（加载器按 build 声明 `anim/attune_fx.zip`），声效为幽灵 haunt | 表内 `pandorachest_reset`、`cavehole_flick_warn` 复用同 bank 的 `attune_in` 动画（带黑色染色），与调谐机制无关 | [复用条目](../../../raw/current/scripts/fx.lua#L522-L537) |

调谐来源当前为 [[reference/components/attunable|attunable]] 的两类宿主：肉像 `resurrectionstatue`（tag `remoteresurrector`）与温蒂复活墓 `wendy_resurrectiongrave`（tag `gravestoneresurrector`），两类在 `EQUIVALENT_ATTUNABLE_TAGS` 互为等价（[等价表](../../../raw/current/scripts/constants.lua#L2962-L2963)）；等价表只用于换绑时自动解除旧源，`HasAttunement` 与失去判定仍按单一 tag，不查等价表。两个宿主页 Wiki 待建，目标侧关系见 [[reference/components/attunable]]。

## 典型接入

### 模组绑定表现复用调谐特效

- 前提与选择：模组自定义"绑定／解除"逻辑想沿用原版调谐表现时，在服务端回调直接 `SpawnPrefab`；跟随宿主用父挂式，固定位置用落点式。
- 生效与撤销：父挂式本地表现成为宿主子实体、动画结束自删；代理 1 秒后自行回收，无需清理代码。
- 原版依据：父挂与幽灵态选名写法同 [player_common 回调](../../../raw/current/scripts/prefabs/player_common.lua#L442-L470)。

```lua
-- 服务端（如玩家 master_postinit 之后的模组事件回调）：
local function OnMyBond(inst)
    SpawnPrefab("attune_out_fx").entity:SetParent(inst.entity)  -- 随玩家移动
end

local function OnMyUnbond(inst)
    local name = inst:HasTag("playerghost") and "attune_ghost_in_fx" or "attune_in_fx"
    SpawnPrefab(name).entity:SetParent(inst.entity)
end
```

- 调整与验证：三注册仅动画名不同，换名即换表现；ghost 选名应保持与原版同款 `playerghost` tag 判断，勿按注册名语义选名（见陷阱）。

## 常见陷阱与限制

- **误区：按注册名方向猜用途**：`attune_out_fx` 在**获得**调谐时播放（anim `attune_out`），失去调谐播的是 `attune_in_fx`／`attune_ghost_in_fx`；命名方向与事件语义相反，选名以生成条件为准。[回调](../../../raw/current/scripts/prefabs/player_common.lua#L442-L470)
- **边界：同 tag 换绑的静默依赖 `_isrezattuned`**：标志为 true 时，已持有同一实际 tag 的调谐再绑新源，`gotnewattunement` 被拦下；旧源解除时 `HasAttunement` 仍查到新源，`attunementlost` 分支也不落 FX。等价但不同 tag 的互换（肉像 ↔ 温蒂复活墓）不静默：新源先注册，旧源解除时 `HasAttunement("remoteresurrector")` 查不到 `gravestoneresurrector` 新源，会进入 `attune_in_fx`／`attune_ghost_in_fx` 分支并把 `_isrezattuned` 置 false（FX 本身仍受 `remoterezsource` 抑制）。标志被清 false 后，下一次即使同 tag 换绑，`OnGotNewAttunement` 也会按"从无到有"重播 `attune_out_fx` 并把标志设回 true——静默不是同 tag 的固有属性。[回调](../../../raw/current/scripts/prefabs/player_common.lua#L442-L470)、[等价解除](../../../raw/current/scripts/components/attunable.lua#L20-L30)、[单 tag 判定](../../../raw/current/scripts/components/attuner.lua#L26-L32)
- **边界：远程复活静默**：玩家从调谐来源复活时 `remoterezsource` 置位，期间的调谐解除不生成 attune_in 系特效；标志在复活流程多处清除（源失效回退分支与各复活收尾）。[置位](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L624-L629)、[回退清除](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L484-L514)、[复活收尾清除](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L763-L763)、[藤蔓保存复活清除](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L821-L821)
- **边界：加载与重连不出特效**：监听注册安排在 `ms_playerjoined` 之后，存档恢复的 relink 不再命中回调；初值 `_isrezattuned` 按已有调谐预置。[注册窗口](../../../raw/current/scripts/prefabs/player_common.lua#L860-L866)
- **边界：`attune_out_fx` 不只属于调谐**：墓碑挖除、恶花落点、装饰完成与温蒂墓碑合成四处落点复用同一注册，修改条目或 `attune_fx` 资源会同时改变这些表现。[复用点](../../../raw/current/scripts/prefabs/gravestone.lua#L67-L128)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。三注册 bank／build／anim／sound／fn 逐项核对；对三个注册名在 `raw/current/scripts` 全域 `grep -rn`（根级与 prefabs/、components/、stategraphs/ 含动态选名拼接）反查生成点，全部命中为 player_common 2 处、gravestone 3 处、wendy_recipe_gravestone 1 处，无其他消费点；触发链读到 attunable → classified → attuner 事件与监听注册端界；等价不同 tag 换绑的注册顺序（新源先注册、旧源再解除）与单 tag 判定已核，会落 in 并清 `_isrezattuned`；清标志后下一次同 tag 换绑会被 `OnGotNewAttunement` 当作从无到有重播 out 并设回 true；`remoterezsource` 置位／清除路径核对；简中 `.po` 显示名称键定向检索无命中。未覆盖：动画素材内容与时长、调谐成本与动作侧流程（[[reference/components/attunable|attunable]] 页范围）、其他语言 `.po`。
- 游戏运行：未运行；特效视觉未实测。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/prefabs/index|Prefab 索引]]。
