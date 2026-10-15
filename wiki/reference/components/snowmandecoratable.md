---
title: "snowmandecoratable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "雪人装饰组件：管理雪球/雪人的堆叠、帽子与装饰物串数据（netvar 同步），主持 POPUP 装饰会话并校验消耗背包物品。"
aliases: ["雪人装饰", "堆雪人", "snowmandecoratable"]
tags: [component, item]
dst_build: "steam-buildid:26230496"
sources: ["components/snowmandecoratable.lua", "prefabs/snowball_item.lua", "prefabs/snowman.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# snowmandecoratable

## 快速判断

- 职责与适用：挂在 `snowball_item`（小型，[装配](../../../raw/current/scripts/prefabs/snowball_item.lua#L200-L201)）与 `snowman`（推滚后的大雪人）上，是 DECORATESNOWMAN／STACK 动作的后端：堆雪球、戴帽子、摆装饰，全部数据经 netvar 双端可见（[动作入口](../../../raw/current/scripts/actions.lua#L6106-L6130)）。
- 优先使用：装饰会话用 `CanBeginDecorating` → `BeginDecorating(doer, obj)` → 关闭弹窗自动校验；堆叠走 `CanStack`/`Stack`/`Unstack`（[L664-L720](../../../raw/current/scripts/components/snowmandecoratable.lua#L664-L720)、[L457-L514](../../../raw/current/scripts/components/snowmandecoratable.lua#L457-L514)）。
- 重要副作用：装饰/帽子是跟随符号的非网络 FX 实体，进 `highlightchildren`；数据本体在 `snowmandecoratable.decordata/basesize/stacks/stackoffsets` 四个 netvar（[L348-L354](../../../raw/current/scripts/components/snowmandecoratable.lua#L348-L354)、[FX 创建](../../../raw/current/scripts/components/snowmandecoratable.lua#L122-L143)）。
- 修改边界：仅主端改数据（多数主端函数入口即判 `ismastersim`）；客户端只监听 dirty 事件重摆 FX（[L357-L362](../../../raw/current/scripts/components/snowmandecoratable.lua#L357-L362)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `decordata` net_string | ZipAndEncodeString 的 5 元组序列 `{hash, rot, flip, x, y}` | 装饰数据的唯一存储；客户端 dirty 后 `DoRefreshDecorData` 重摆（[L348](../../../raw/current/scripts/components/snowmandecoratable.lua#L348)、[解码](../../../raw/current/scripts/components/snowmandecoratable.lua#L223-L242)）。 |
| `basesize` net_tinybyte / `stacks`+`stackoffsets` net_smallbytearray | STACK_IDS 小/中/大；offset 0~63 | 底座尺寸与叠放层；`GetSize`/`SetSize`/`GetStacks`/`IsStacked` 读取（[L350-L355](../../../raw/current/scripts/components/snowmandecoratable.lua#L350-L355)、[L437-L455](../../../raw/current/scripts/components/snowmandecoratable.lua#L437-L455)、[STACK_DATA](../../../raw/current/scripts/components/snowmandecoratable.lua#L51-L101)）。 |
| `CanBeginDecorating(doer)` | → bool, reason | busy/负重/被推动/被占用返回 false；融化中返回 `"MELTING"`、他人使用中 `"INUSE"`（[L664-L679](../../../raw/current/scripts/components/snowmandecoratable.lua#L664-L679)）。 |
| `BeginDecorating(doer, obj)` / `EndDecorating(doer)` | — | 记 doer、监听其 onremove 与 `ms_closepopup`、玩家进 `snowmandecorating` 状态并启动更新；结束推 `ms_endsnowmandecorating`（immediate）（[L681-L720](../../../raw/current/scripts/components/snowmandecoratable.lua#L681-L720)）。 |
| `onclosepopup` → `onclosesnowman` | POPUPS.SNOWMANDECORATING | 弹窗提交的装饰串经 `_ValidateAndAppendDecorData` 校验：超出 `TUNING.SNOWMAN_MAX_DECOR`（底座 5/15/20＋叠层递减，[tuning](../../../raw/current/scripts/tuning.lua#L8558)）截断，物品从发起物与背包扣除后才入档（[L373-L392](../../../raw/current/scripts/components/snowmandecoratable.lua#L373-L392)、[校验](../../../raw/current/scripts/components/snowmandecoratable.lua#L244-L297)）。 |
| `CanStack(doer, obj)` / `Stack(doer, obj)` | → bool, reason | 只接受带本组件的可抛雪球；`"HASHAT"`/`"STACKEDTOOHIGH"`（总 stackheight > 6）/`"INUSE"`；Stack 移除雪球并把其 basesize 追加进 stacks（[L457-L514](../../../raw/current/scripts/components/snowmandecoratable.lua#L457-L514)、[MAX_STACK_HEIGHT](../../../raw/current/scripts/components/snowmandecoratable.lua#L103)）。 |
| `Unstack(isdestroyed)` | — | 逆操作：small 层掉 `snowball_item`，其余按尺寸掉 `snowman` 或摧毁；底座是 small 时整体移除自身（[L535-L573](../../../raw/current/scripts/components/snowmandecoratable.lua#L535-L573)）。 |
| `EquipHat(hat)` / `UnequipHat()` | HEAD 槽装备 | 生成 `snowmanhat_fx`（自带 inventory）挂帽子；有帽子时禁止堆叠（[L588-L654](../../../raw/current/scripts/components/snowmandecoratable.lua#L588-L654)、[L462-L464](../../../raw/current/scripts/components/snowmandecoratable.lua#L462-L464)）。 |
| `DropAllDecor()` | — | 按串统计各物品数量掉落并清空 decordata（[L722-L756](../../../raw/current/scripts/components/snowmandecoratable.lua#L722-L756)）。 |
| `SetMelting(bool)` / `IsMelting()` | — | 融化标记，融化中不可开始装饰（[L656-L662](../../../raw/current/scripts/components/snowmandecoratable.lua#L656-L662)）。 |
| `TransferComponent(newinst)` | — | 雪球升级为雪人时搬运尺寸/叠层/装饰数据（[L827-L839](../../../raw/current/scripts/components/snowmandecoratable.lua#L827-L839)）。 |

## 关键流程

### 一次装饰会话

1. 玩家手持普通物品对雪人触发 DECORATESNOWMAN → `CanBeginDecorating` + 可见性检查（黑暗静默失败，[L6116-L6118](../../../raw/current/scripts/actions.lua#L6116-L6118)）；手持 HEAD 装备则直接 `EquipHat`。
2. 会话中 `OnUpdate` 自动收口：玩家离开 3 距离（`self.range`）或看不到目标即 `EndDecorating`（[L845-L851](../../../raw/current/scripts/components/snowmandecoratable.lua#L845-L851)）；玩家移除、进入 limbo 同样收口（[L334-L337](../../../raw/current/scripts/components/snowmandecoratable.lua#L334-L337)）。
3. 弹窗提交后数据串落 netvar；被拾起在手上时用 `snowmandecorating_swap_fx` 展示装饰（equipped/unequipped 处理，[L309-L332](../../../raw/current/scripts/components/snowmandecoratable.lua#L309-L332)）。

### 存档

保存 decor 串、stacks/stackoffsets、帽子存档记录与 hatrnd（[L769-L825](../../../raw/current/scripts/components/snowmandecoratable.lua#L769-L825)）。

## 常见陷阱与限制

- **装饰物清单是硬编码表**：39 种物品的 bank/build/anim 在 `ITEM_DATA`（按 prefab 名 hash 索引），表外物品无法放置；判定与摆位和装饰界面共享坐标约定（需两处同步，[L3-L49](../../../raw/current/scripts/components/snowmandecoratable.lua#L3-L49)、[注释](../../../raw/current/scripts/components/snowmandecoratable.lua#L164-L165)）。
- **雪球可装饰但装饰不随堆叠转移**：Stack 注释明确不转移装饰（假设可拾起的不能预先装饰，[L493-L495](../../../raw/current/scripts/components/snowmandecoratable.lua#L493-L495)）。
- **上限按底座与叠层动态扣减**：每叠一层从预算里再扣 5，下限保底 5（[L251-L254](../../../raw/current/scripts/components/snowmandecoratable.lua#L251-L254)）。
- **FX 是非网络实体**：`CreateDecor` 标 `FX`、不 persists，纯靠各端按 decordata 重建；修改摆放算法要主客户端一致（[L122-L143](../../../raw/current/scripts/components/snowmandecoratable.lua#L122-L143)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、snowball_item/snowman 装配与消费、DECORATESNOWMAN 动作、SNOWMAN_MAX_DECOR 调值；未覆盖装饰界面（snowmandecoratingscreen）与 SGwilson 会话状态细节。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/snowmandecor]]。
