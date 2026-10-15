---
title: "playbill"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "戏单数据组件：承载一副戏的脚本、服装表、起始/当前幕与讲台外观，供戏台讲读器换幕、写文本并注册演出。"
aliases: ["戏单", "playbill", "演出剧本数据"]
tags: [component, item]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/playbill.lua", "prefabs/playbill.lua", "components/playbill_lecturn.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# playbill

## 快速判断

- 职责与适用：戏单物品（playbill_the_doll / _veil / _vault / _princess_yoth）的数据容器：`scripts`（按幕索引的台词表）、`costumes`（角色服装表）、`starting_act`/`current_act`、`book_build`（讲台贴图 override）。[数据字段](../../../raw/current/scripts/components/playbill.lua#L2-L8)
- 优先使用：要做可换幕的剧目物品时挂本组件并按原版填表；消费方是 [playbill_lecturn](../../../raw/current/scripts/components/playbill_lecturn.lua#L46-L85)（讲台）与舞台演出（stageactingprop）。[物品侧挂载](../../../raw/current/scripts/prefabs/playbill.lua#L99-L104)
- 重要副作用：`OnSave/OnLoad` 只持久化 `current_act`，脚本与服装表在 prefab 构造时从 `play_<name>` 数据重填。[存读](../../../raw/current/scripts/components/playbill.lua#L14-L25)
- 修改边界：组件无事件、无网络同步；换幕是纯服务端数据修改，讲台文本经 writeable 组件刷新。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `scripts` / `costumes` | `table`（幕→脚本；角色→服装） | prefab 侧由 `require("play_"..name)` 填入； lectern 的 `UpdateText` 按 `pb.scripts[pb.current_act]` 拼台词与 cast 名单。[填表](../../../raw/current/scripts/prefabs/playbill.lua#L100-L104) |
| `SetCurrentAct(act)` | `act: any -> nil` | 写 `current_act`；lectern 换幕时调用。[实现](../../../raw/current/scripts/components/playbill.lua#L10-L12) |
| `starting_act` / `current_act` | `any` | 换上讲台时重置为 `starting_act`；演出注册也用该值。[SwapPlayBill](../../../raw/current/scripts/components/playbill_lecturn.lua#L49) |
| `book_build` | `string \| nil` | 讲台 AnimState 的 `AddOverrideBuild` 外观；仅 the_veil/_vault/_princess 有值。[使用](../../../raw/current/scripts/components/playbill_lecturn.lua#L67-L69) |
| `OnSave()` / `OnLoad(data)` | `-> {current_act} \| nil`；`data` 同构 | 只存取 `current_act`，其余字段不持久化。[实现](../../../raw/current/scripts/components/playbill.lua#L14-L25) |

## 典型接入

### 复刻"把戏单放上讲台开演"链路（原版参照）

- 前提与选择：服务端；戏单物品挂 playbill，讲台挂 [playbill_lecturn](../../../raw/current/scripts/components/playbill_lecturn.lua#L1-L14)（自带 `playbill_lecturn` 标签）。玩家把戏单放上讲台走 `ACTIONS` 的判定：目标带 `playbill_lecturn` 标签且 `invobject.components.playbill` 存在。[动作判定](../../../raw/current/scripts/actions.lua#L2296-L2297)
- 生效与撤销：`SwapPlayBill` 把旧戏单弹回地面、新戏单设为 `playbill_item`、按 `book_build` 换讲台外观，并在有舞台时把 costumes/scripts/act 注册给 `stageactingprop:AddPlay`；换幕走 `ChangeAct` → `SetCurrentAct` → `UpdateText` 重写 writeable 文本。[SwapPlayBill 全流程](../../../raw/current/scripts/components/playbill_lecturn.lua#L46-L85)
- 原版依据：四副剧目 prefab 由 `makeplay` 工厂生成，剧目数据来自 `play_the_doll` 等模块的 `scripts/costumes/starting_act` 字段。[工厂](../../../raw/current/scripts/prefabs/playbill.lua#L124-L128)

```lua
-- 服务端：自定义剧目物品（等价 makeplay 的核心数据填充）
inst:AddComponent("playbill")
inst.components.playbill.scripts = require("play_mymod_play").scripts
inst.components.playbill.costumes = require("play_mymod_play").costumes
inst.components.playbill.starting_act = "act1"
inst.components.playbill.current_act = "act1"
```

- 调整与验证：`scripts[current_act]` 需含 `playbill`（标题文本）与 `cast`（角色名列表）字段供 UpdateText 消费（[UpdateText](../../../raw/current/scripts/components/playbill_lecturn.lua#L30-L44)）；缺字段会在换讲台时直接报错。

## 常见陷阱与限制

- **把组件当完整演出系统：** playbill 只存数据；实际演出、观众 buff 由 stageactingprop 侧处理，戏单离开讲台后 `AddPlay` 的注册不随之撤销。[分工](../../../raw/current/scripts/components/playbill_lecturn.lua#L70-L82)
- **期待 current_act 跨会话完整恢复：** 只有 `current_act` 进存档；`scripts/costumes/starting_act/book_build` 每次载入由 prefab 重填，自定义剧目必须保证 prefab 构造路径能拿到同名数据模块。[存读边界](../../../raw/current/scripts/components/playbill.lua#L14-L25)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（27 行）、prefabs/playbill.lua 工厂与四剧目、playbill_lecturn.lua 全文、actions.lua 放置动作判定；grep 确认 `components.playbill` 消费者仅 actions.lua 与 playbill_lecturn.lua。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；换幕与演出注册未实测。

相关：[[reference/components/writeable]]
