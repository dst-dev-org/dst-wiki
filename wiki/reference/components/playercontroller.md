---
title: "playercontroller"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家输入枢纽组件：缓存并执行左右键与动作按钮动作，把本地输入转成预测 + RPC 发往主机重推导校验；本页聚焦动作按钮与鼠标主链路，移动/攻击/建造等系统只给流程索引。"
aliases: ["玩家控制器", "动作按钮", "playercontroller"]
tags: [component, callback, network-sync]
size_class: complex
dst_build: "steam-buildid:26230496"
sources: ["components/playercontroller.lua", "prefabs/player_common.lua", "components/rider_replica.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# playercontroller

## 快速判断

- 职责与适用：`inst == ThePlayer 或主机` 时与 [[reference/components/playeractionpicker]] 一起挂载（([player_common.lua:1023-1027](../../../raw/current/scripts/prefabs/player_common.lua#L1023-L1027)）；picker 只产出候选，本组件负责缓存（`self.LMBaction/RMBaction`）、执行（预测或直推 locomotor）与跨端发送/校验。候选生成规则见 playeractionpicker 页与 componentactions。
- 优先使用：接管动作按钮用 `actionbuttonoverride` 字段（[L2111-L2116](../../../raw/current/scripts/components/playercontroller.lua#L2111-L2116)，原版用例 ([rider_replica.lua:143-146](../../../raw/current/scripts/components/rider_replica.lua#L143-L146)）；执行任意 BufferedAction 用 `DoAction(buffaction)`（[L4545-L4653](../../../raw/current/scripts/components/playercontroller.lua#L4545-L4653)）；查询当前鼠标动作用 `GetLeftMouseAction()/GetRightMouseAction()`（[L5324-L5330](../../../raw/current/scripts/components/playercontroller.lua#L5324-L5330)）。
- 重要副作用：`DoAction` 会清 `attack_buffer`、触发目标高亮闪烁并可能自动装备手上物品（`DoActionAutoEquip`，[L4655-L4681](../../../raw/current/scripts/components/playercontroller.lua#L4655-L4681)）；客户端调用会经 `preview_cb`/`non_preview_cb` 发 RPC 给主机。
- 修改边界：本组件同时存在于本地端与主机端同一玩家实体；`self.handler ~= nil` 表示本地输入端（仅 ThePlayer，[Activate](../../../raw/current/scripts/components/playercontroller.lua#L313-L346)），主机上的远程玩家走 `OnRemote*` 系列。改输入行为优先用 override 钩子或动作过滤栈（picker 页），不要整方法替换。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetActionButtonAction(force_target)` | `-> BufferedAction 或 nil` | 动作按钮（空格）主查询：门禁（远端按住中／`IsEnabled`／`IsBusy`／force_target 可见性）→ `actionbuttonoverride` → 满手拦截 → 幽灵 HAUNT、捕虫网、gestalt 捕捉、月光静态捕捉、CATCH、UNPIN、复活尸体等专项 → 白名单 FindEntities（直接行走 3 否则 6）逐个 `GetPickupAction`（[L2098-L2283](../../../raw/current/scripts/components/playercontroller.lua#L2098-L2283)）。 |
| `GetPickupAction(self, target, tool)`（文件内 local） | `-> ACTION 或 nil` | 拾取优先级链：smolder→SMOTHER；`TOOLACTIONS` 标签对（目标 `k.."_workable"` + 工具 `k.."_tool"` → `ACTIONS[k]`）；陷阱/矿；inactive→ACTIVATE；可拾取→PICKUP（需格子）；pickable→PICK；harvestable/readyforharvest→HARVEST；unsaddler/brush 工具；corpse→REVIVE_CORPSE；sweeper 工具+sweepable→SWEEP（[L2005-L2067](../../../raw/current/scripts/components/playercontroller.lua#L2005-L2067)）。 |
| `DoActionButton()` | `-> nil` | 有 placer 走 `MakeRecipeAtPoint`；否则 `GetActionButtonAction()`：主机 `locomotor:PushAction`；无 locomotor 客户端先 `pre_action_cb` 再 `RemoteActionButton`；预测客户端挂 `preview_cb` 后 `PreviewAction`（[L2285-L2326](../../../raw/current/scripts/components/playercontroller.lua#L2285-L2326)）。 |
| `RemoteActionButton(action, isreleased)` | `-> nil` | 客户端发送：`SendRPCToServer(RPC.ActionButton, actioncode, target, isreleased, noforce, mod_name)`，并写 `remote_controls[CONTROL_ACTION]` 冷却（[L2353-L2360](../../../raw/current/scripts/components/playercontroller.lua#L2353-L2360)）。 |
| `OnRemoteActionButton(actioncode, target, ...)` | 主机 RPC 入口 | `SetClientRequestedAction(actioncode, mod_name)` 后重跑 `GetActionButtonAction(target)`，**仅当重推导结果的 code＋mod_name 与客户端声明一致才执行**，`canforce` 时可强制目标点（[L2328-L2351](../../../raw/current/scripts/components/playercontroller.lua#L2328-L2351)；[SetClientRequestedAction](../../../raw/current/scripts/actions.lua#L281)）。 |
| `DoAction(buffaction, spellbook)` | `-> nil` | 通用执行：有效性（invobject/target/doer、busy，CASTAOE 可重铸例外）→ 去重（同 action+target+pos 不重发）→ ATTACK retarget、高亮 → autoequip → 主机 PushAction／非预测客户端 `non_preview_cb`／预测 `PreviewAction`（[L4545-L4653](../../../raw/current/scripts/components/playercontroller.lua#L4545-L4653)）。 |
| `OnLeftClick(down)` / `OnRemoteLeftClick(...)` | 鼠标左键链 | 本地：双击动作（picker `GetDoubleClickActions`）→ `GetLeftMouseAction()` 或 WALKTO 兜底 → 地图目标转拉起大地图 → RPC.LeftClick（平台相对坐标，[GetPlatformRelativePosition](../../../raw/current/scripts/components/playercontroller.lua#L4972-L4979)）。主机：`DoGetMouseActions` 重推导后按 code+mod_name 匹配，接受 lmb/rmb/dblclick 与 LOOKAT/WALKTO 兜底（[L4683-L4891](../../../raw/current/scripts/components/playercontroller.lua#L4683-L4891)、[L4893-L4970](../../../raw/current/scripts/components/playercontroller.lua#L4893-L4970)）。 |
| `OnRightClick(down)` / `OnRemoteRightClick(...)` | 鼠标右键链 | 本地：`GetRightMouseAction()`；nil 时关合成/法轮、退回active item、尝试 AOE 瞄准；部署时写 placer 旋转与落点；RPC.RightClick。主机重推导校验同左键（[L4981-L5090](../../../raw/current/scripts/components/playercontroller.lua#L4981-L5090)、[L5092-L5123](../../../raw/current/scripts/components/playercontroller.lua#L5092-L5123)）。 |
| `LMBaction/RMBaction` 缓存 | BufferedAction 或 nil | 仅本地玩家＋鼠标模式在 `OnUpdate` 里刷新：`playeractionpicker:DoGetMouseActions()`（[L2827-L2830](../../../raw/current/scripts/components/playercontroller.lua#L2827-L2830)）；手柄模式置 nil 并改走控制器目标选择。 |
| `GetGroundUseAction(position, spellbook)` | `-> lmb, rmb` | 空地点击（手柄动作键与 `RPC.ControllerActionButtonPoint` 主机侧重推导用）：手物品 aoetargeting 校验 `CanCastAtPoint` 后取 picker `GetPointActions`，含 TERRAFORM_REMOVE 换 target 特例（[L5376-L5451](../../../raw/current/scripts/components/playercontroller.lua#L5376-L5451)；调用点 [L831](../../../raw/current/scripts/components/playercontroller.lua#L831)、[L1004](../../../raw/current/scripts/components/playercontroller.lua#L1004)）。 |
| `actionbuttonoverride(inst, force_target)` | 钩子字段 `-> buffaction, usedefault` | 返回 buffaction（或 `nil, true` 回落默认链）；rider_replica 在骑乘时设置、下骑清除（[L2111-L2116](../../../raw/current/scripts/components/playercontroller.lua#L2111-L2116)、([rider_replica.lua:141-147](../../../raw/current/scripts/components/rider_replica.lua#L141-L147)）。 |
| `IsEnabled()/IsBusy()/CanLocomote()` | `-> bool[, bool]` | 开关由 `classified.iscontrollerenabled`＋HUD 焦点决定（第二返回值 ishudblocking 放行移动）；busy 主机看状态图、客户端还看 pauseprediction；CanLocomote 是预测执行门（[L491-L502](../../../raw/current/scripts/components/playercontroller.lua#L491-L502)、[L760-L777](../../../raw/current/scripts/components/playercontroller.lua#L760-L777)）。 |
| `client_forward_target` / `client_forward_action_target` 标签 | 实体字段／标签 | 动作按钮搜索与控制器交互目标把替身实体（茶铺柜台、残影 FX、全息图、高尔夫道具）前转到真实目标再判定（[L2248](../../../raw/current/scripts/components/playercontroller.lua#L2248)、[L2267](../../../raw/current/scripts/components/playercontroller.lua#L2267)、[L3467](../../../raw/current/scripts/components/playercontroller.lua#L3467)）；"动作按钮前进动作"实例见 [[reference/components/sweeper]]。 |

## 典型接入

### 让自定义动作进入右键/左键/动作按钮

- 前提与选择：候选登记在 componentactions（picker 消费），执行与校验在 playercontroller。主机 RPC 会按 code+mod_name 比对重推导结果，所以客户端候选与主机候选必须由同一套 componentactions 产出——自定义 ACTION 的 fn 需在主端可执行（客户端 `server_only=false` 时双端候选一致最稳）。

```lua
-- modmain.lua：登记候选（componentactions 侧）
AddComponentAction("SCENE", "mycomponent", function(inst, doer, actions, right)
    if doer:HasTag("myuser") then
        table.insert(actions, right and ACTIONS.MYRMB or ACTIONS.MYLMB)
    end
end)

-- modmain.lua：接管动作按钮（仿 rider_replica 的 override 模式）
AddPlayerPostInit(function(inst)
    if inst.components.playercontroller ~= nil then
        inst.components.playercontroller.actionbuttonoverride = function(player, force_target)
            local target = force_target
                or (player:HasTag("myuser") and FindEntity(player, 6, nil, { "mytargetable" }, { "FX", "NOCLICK", "INLIMBO" }) or nil)
            if target ~= nil then
                return BufferedAction(player, target, ACTIONS.MYRMB)
            end
            return nil, true -- 放行默认动作按钮链
        end
    end
end)
```

- 调整与验证：`FindEntity 半径 6` 与动作按钮白名单搜索一致；override 返回的 buffaction 会走 `DoActionButton` 的预测/RPC 链路，`ACTIONS.MYRMB.canforce` 决定主机是否接受强制目标点。验证按 [[reference/components/playeractionpicker]] 的 override 用例同法。

## 常见陷阱与限制

- **主机重推导是安全门**：`OnRemoteActionButton`/`OnRemoteLeftClick`/`OnRemoteRightClick` 都要求重推导候选的 `action.code` 与 `mod_name` 匹配客户端声明，否则静默丢弃（源码里被注释的 print 是调试残留）。模组若在客户端改写了候选产出（如过滤栈）而主机没同步，动作会"点了没反应"。
- **LMB/RMB 缓存只对本地鼠标玩家有效**：`GetLeftMouseAction()` 在手柄模式或远端返回 nil（OnUpdate 仅 `self.handler ~= nil` 且非 controller 模式刷新，[L2827-L2830](../../../raw/current/scripts/components/playercontroller.lua#L2827-L2830)）；HUD/界面代码不能依赖它拿到主机玩家动作。
- **去重与 busy 抑制**：`DoAction` 对同 action+target+pos 的重复 BufferedAction 直接返回（卡等待除外，[L4590-L4603](../../../raw/current/scripts/components/playercontroller.lua#L4590-L4603)）；busy 时仅 CASTAOE 可重铸例外（[L4568-L4581](../../../raw/current/scripts/components/playercontroller.lua#L4568-L4581)）。
- **自动装备白名单**：`DoActionAutoEquip` 对 DROP/STORE/EQUIP/GIVE/DEPLOY/CONSTRUCT 等一批动作豁免（[L4655-L4681](../../../raw/current/scripts/components/playercontroller.lua#L4655-L4681)）；给新动作配手持物品行为时核对是否需要加入豁免表。
- **非预测客户端双回调**：`locomotor == nil` 时发送依赖 `buffaction.non_preview_cb`（有 `pre_action_cb` 时需在 cb 内触发），旧模组只写 `pre_action_cb` 会导致无预测客户端不发 RPC（[L4634-L4644](../../../raw/current/scripts/components/playercontroller.lua#L4634-L4644)）。

## 未覆盖范围（流程索引）

本页为精选覆盖，未逐方法审校全部 5943 行。已读区域：L1-460（构造/Activate/classified）、L491-660（开关/远端控制）、L760-780、L1988-2360（动作按钮链）、L2653-2900（OnUpdate 缓存与攻击缓冲段）、L4545-5130（DoAction/左右键链）、L5324-5560（查询类）。以下系统只登记入口，未逐行审校，接入前需按行号自查：

- **攻击链**：`ValidateAttackTarget`(L1747)、`GetAttackTarget`(L1790)、`DoAttackButton`(L1879)、`OnRemoteAttackButton`(L1933)、attack_buffer 结算（L2779-L2798）。
- **建造放置**：`CancelPlacement`(L1522)、`StartBuildPlacementMode`(L1547)、placer 复原（L2811 起）。
- **手柄目标选择与动作键**：`DoControllerActionButton`(L794)、`DoControllerAltActionButton`(L1066)、`DoControllerAttackButton`(L1281)、`UpdateControllerTargets` 系列（L3188-L3670）。
- **移动与预测**：OnUpdate 移动段（L2900-L3180）、`DoPredictWalking`(L4006)、`DoDragWalking`(L4129)、`DoDirectWalking`(L4309)、双击冲刺 `DoDoubleTapDir`(L4185)、Remote 预测/橡皮筋（L3700-L3960）。
- **地图控制**：`PullUpMap`(L444)、`GetMapActions`(L5167)、`OnMapAction`(L5275)。
- **其余**：AOE/reticule（L1567-L1735）、inspect/resurrect 按钮（L2362-L2476）、命令轮盘（L2477-L2564）、相机控制 `DoCameraControl`(L4441)。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。本页为精选覆盖，未逐方法审校全部 5943 行：按上节行号区间通读约 1800 行并逐行核对引用锚点；挂载行（player_common.lua:1023-1027）、override 原版用例（rider_replica.lua:141-147）、`SetClientRequestedAction`（actions.lua:281）实读核对；与 [[reference/components/playeractionpicker]] 交叉引用的行号（2830/4913-4916/5103/5513）与该页已核验记录一致。未覆盖系统以上节清单为准。
- 结构检查：validate_page、validate_source_links --strict 通过（见批级记录）。
- 游戏运行：未运行；RPC 校验行为与预测链路未实测。

相关：[[reference/components/playeractionpicker]]、[[reference/components/sweeper]]、[[reference/components/locomotor]]
