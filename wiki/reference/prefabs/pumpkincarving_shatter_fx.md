---
title: "南瓜雕刻碎裂动画（pumpkincarving_shatter_fx）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "雕刻会话定稿时在宿主位置落点播放的 farm_plant_pumpkin fx_cut 一次性碎裂动画：pumpkincarvable／pumpkinhatcarvable 两组件的弹窗回调在数据校验通过后生成，与 pumpkincarving_swap_fx 换装代理是不同注册。"
aliases: ["pumpkincarving_shatter_fx", "南瓜雕刻碎裂", "雕刻碎裂特效", "雕刻定稿特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
size_class: simple
sources: ["fx.lua", "prefabs/fx.lua", "components/pumpkincarvable.lua", "components/pumpkinhatcarvable.lua", "components/inventoryitem.lua", "prefabs/veggies.lua", "prefabs/farm_plant_defs.lua", "prefabs/hats.lua", "prefabs/snowman.lua", "components/snowmandecoratable.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 南瓜雕刻碎裂动画（pumpkincarving_shatter_fx）

## 快速判断

- 职责与适用：`fx.lua` 通用表单注册动画特效：雕刻会话定稿瞬间在宿主位置播放 `farm_plant_pumpkin` 资源 `fx_cut` 碎裂动画，半比例、无声，无玩法归属。[条目](../../../raw/current/scripts/fx.lua#L3700-L3708)
- 优先使用：查"雕刻南瓜／南瓜帽定稿的碎裂表现"选本页；持久雕刻显示是另一注册 [[reference/prefabs/pumpkincarving_swap_fx#注册与构造|pumpkincarving_swap_fx]]（UGC 换装数据代理），与本名由同一组件流程分别生成。
- 重要副作用：无音效、无染色；本地表现动画播完 `animover` 自删，生成者无需清理代理。[协议回收](../../../raw/current/scripts/prefabs/fx.lua#L98-L102)
- 修改边界：仅主模拟生成（两组件会话回调只在 master 分支存在），表现由各模拟按工厂协议本地创建；落点固定 `(x, 1, z)`，缩放 0.5 写死在条目 fn。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `pumpkincarving_shatter_fx` | 通用表条目 → 加载器代理＋一帧后本地实体 → fn：`SetFinalOffset(2)`＋`SetScale(0.5, 0.5)` | bank＝build＝`farm_plant_pumpkin`、anim＝`fx_cut`；无 sound；未设 twofaced／eightfaced／sixfaced／nofaced 任一字段，走默认 `SetFourFaced`。[条目](../../../raw/current/scripts/fx.lua#L3700-L3708)、[加载器](../../../raw/current/scripts/prefabs/fx.lua#L165-L172) |

注册名在 `languages/chinese_s.po` 无 `STRINGS.NAMES` 显示键（`PUMPKINHAT_CARVED` 是雕刻后帽子的显示名，不是特效名）；标题中文为已核验效果描述，非官方译名。雕刻宿主也没有 `prefabs/pumpkincarving.lua`——组件装在超大南瓜蔬菜与南瓜帽实体上（见下表）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目：网络代理＋一帧延迟本地表现、animover 自删 | 无 update_while_paused／animqueue 分支 | [[reference/mechanisms/generic-fx-factory#关键流程]]；[条目](../../../raw/current/scripts/fx.lua#L3700-L3708) |
| 生成者：巨型蔬菜雕刻 | pumpkincarvable `onclosepumpkin`（`ms_closepopup` 过滤 `POPUPS.PUMPKINCARVING`）校验后 cutdata 非空即在宿主位置落点 | 仅主模拟；`_ValidateCutData` 剔除无对应雕刻刀（pumpkincarver1～3）支持的切口，有效切口为空返回空串、不生成 | [[reference/components/pumpkincarvable]]；[回调](../../../raw/current/scripts/components/pumpkincarvable.lua#L330-L349)、[校验](../../../raw/current/scripts/components/pumpkincarvable.lua#L245-L275) |
| 生成者：南瓜帽雕刻 | pumpkinhatcarvable `onclosepumpkin`（`POPUPS.PUMPKINHATCARVING`）脸数据校验通过且帽子未被持有（`IsHeld()`）、未休眠时落点 | 仅主模拟 | [[reference/components/pumpkinhatcarvable]]；[回调](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L81-L92)、[IsHeld](../../../raw/current/scripts/components/inventoryitem.lua#L462-L464) |
| 宿主：超大蔬菜 | `fn_oversized`／`fn_oversized_waxed` 按 `plant_def.iscarvable` 挂 pumpkincarvable | 仅 pumpkin（`PLANT_DEFS` 唯一 iscarvable）→ 注册名 `pumpkin_oversized`／`pumpkin_oversized_waxed` | Wiki 待建：[挂载](../../../raw/current/scripts/prefabs/veggies.lua#L853-L856)、[第二处](../../../raw/current/scripts/prefabs/veggies.lua#L943-L946)、[注册](../../../raw/current/scripts/prefabs/veggies.lua#L1088-L1089)、[iscarvable](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L8) |
| 宿主：南瓜帽 | `MakeHat("pumpkin")` 的 custom_init 挂 pumpkinhatcarvable | 预制名 `pumpkinhat`（prefabname＝name.."hat"） | Wiki 待建：[挂载](../../../raw/current/scripts/prefabs/hats.lua#L6114-L6115)、[命名](../../../raw/current/scripts/prefabs/hats.lua#L16-L20)、[注册](../../../raw/current/scripts/prefabs/hats.lua#L7947) |
| 仅声明／注释 | snowman prefabs 数组与 veggies oversized_prefabs 收集依赖；snowmandecoratable 的落点调用整段注释 | 均非生成点 | [snowman](../../../raw/current/scripts/prefabs/snowman.lua#L11)、[veggies](../../../raw/current/scripts/prefabs/veggies.lua#L441)、[注释](../../../raw/current/scripts/components/snowmandecoratable.lua#L385-L389) |
| 资源 | bank/build `farm_plant_pumpkin`（与雕刻切口 pieces 同 build）、anim `fx_cut` | 动画内容与时长未核验 | [条目](../../../raw/current/scripts/fx.lua#L3700-L3708) |

## 典型接入

### 复刻雕刻定稿碎裂反馈

- 前提与选择：主模拟任意位置 `SpawnPrefab`；原版两处都固定 `(x, 1, z)`——y=1 抬到宿主体高度，不贴地。
- 生效与撤销：一次 `fx_cut` 后 `animover` 自删；无音效，需要声音由调用方另配。
- 原版依据：[pumpkincarvable 生成](../../../raw/current/scripts/components/pumpkincarvable.lua#L342-L345)。

```lua
local fx = SpawnPrefab("pumpkincarving_shatter_fx")
fx.Transform:SetPosition(x, 1, z)   -- 原版约定 y=1（宿主体高度）
```

- 调整与验证：缩放 0.5 写死在条目 fn，要全尺寸表现需另建注册或换条目。

## 常见陷阱与限制

- **误区：雕刻显示就是本特效**：可见切口 pieces 由 [[reference/prefabs/pumpkincarving_swap_fx#关联机制导航|pumpkincarving_swap_fx]] 数据代理＋ApplyCuts 持久贴宿主符号；本名只在定稿瞬间播一次碎裂动画，两个注册互不替代。[换装生成](../../../raw/current/scripts/components/pumpkincarvable.lua#L283-L297)
- **边界：校验不通过不生成**：cutdata 经 `_ValidateCutData`（无对应雕刻刀的切口被剔除，全滤空返回空串），FX 与雕刻数据一起不生效；南瓜帽路径同理走 `_ValidateFaceData`。[校验](../../../raw/current/scripts/components/pumpkincarvable.lua#L245-L275)
- **边界：帽子被持有或休眠时无表现**：pumpkinhatcarvable 在 `IsHeld()`（inventoryitem.owner 非空）为真或实体休眠时跳过生成；巨型蔬菜路径无此判断。[条件](../../../raw/current/scripts/components/pumpkinhatcarvable.lua#L86-L89)

## 验证记录

- 源码审阅：2026-11-01，steam-buildid:26335394。核验 fx.lua 条目与 FinalOffset helper、加载器 prefabs/fx.lua、pumpkincarvable／pumpkinhatcarvable 会话回调与校验函数、veggies／hats 宿主挂载与预制名、farm_plant_defs iscarvable、snowman 声明与 snowmandecoratable 注释调用。检索范围：`pumpkincarving_shatter` 在 raw/current/scripts 全域（根级与 prefabs／components／stategraphs）rg，命中仅上述注册、两个生成点、一处注释与两处声明，无动态拼名。`.po` 名称键定向核验无特效显示键。未核验：`fx_cut` 动画资源内容（时长／循环）、雕刻弹窗客户端细节（组件页）。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。

返回 [[reference/prefabs/fx-material-index|采集、制作、碎裂与生成烟]]、[[reference/prefabs/index|Prefab 索引]]。
