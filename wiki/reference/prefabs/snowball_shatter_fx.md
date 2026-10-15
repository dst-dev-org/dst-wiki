---
title: "雪球碎裂动画（snowball_shatter_fx）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "雪球实体原地消失时共用的碎裂表现：snowball_item 融尽、冬宴雪球管理器清场、雪人被锤碎三条服务端路径落点播放 snowball 资源 fx_place 动画；投掷命中表现是 splash_snow_fx，与本注册无关。"
aliases: ["snowball_shatter_fx", "雪球碎裂", "雪球碎裂特效", "雪球消失特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
size_class: simple
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/snowball_item.lua", "components/snowballmelting.lua", "components/snowballmanager.lua", "prefabs/snowman.lua", "prefabs/forest.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 雪球碎裂动画（snowball_shatter_fx）

## 快速判断

- 职责与适用：`fx.lua` 通用表单注册动画特效：雪球实体原地消失（融尽／清场／锤碎）时在实体位置播放 `snowball` 资源 `fx_place` 碎裂动画，无声，无玩法归属。simple 档：单注册、三个生成点、无专属逻辑。[条目](../../../raw/current/scripts/fx.lua#L3710-L3715)
- 优先使用：查"雪球／雪人消失的碎裂表现"选本页；雪球**投掷命中**的水花是 `splash_snow_fx`（[[reference/prefabs/splash|splash 族]]），不是本注册。
- 重要副作用：无音效、无染色；本地表现动画播完 `animover` 自删，生成者随后自行 `Remove` 宿主实体。[协议回收](../../../raw/current/scripts/prefabs/fx.lua#L98-L102)
- 修改边界：三条路径都在主模拟 SpawnPrefab 落点，且融尽／清场两路径有休眠门——实体 asleep 时无 FX，实体照常移除。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `snowball_shatter_fx` | 通用表条目 → 加载器代理＋一帧后本地实体 → fn＝`FinalOffset2`（`SetFinalOffset(2)`） | bank＝build＝`snowball`、anim＝`fx_place`；无 sound；未设 twofaced／eightfaced／sixfaced／nofaced 任一字段，走默认 `SetFourFaced`。[条目](../../../raw/current/scripts/fx.lua#L3710-L3715)、[fn](../../../raw/current/scripts/fx.lua#L5-L7)、[加载器](../../../raw/current/scripts/prefabs/fx.lua#L165-L172) |

注册名在 `languages/chinese_s.po` 无 `STRINGS.NAMES` 显示键；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目：网络代理＋一帧延迟本地表现、animover 自删 | 无 update_while_paused／animqueue 分支 | [[reference/mechanisms/generic-fx-factory#关键流程]]；[条目](../../../raw/current/scripts/fx.lua#L3710-L3715) |
| 生成者：雪球融尽 | snowball_item `OnDoMeltAction`：snowballmelting 温度检查（30±5 秒一拍）处于融化态且环境温度＞0、地表未积雪时触发，落点 (x,y,z) 后实体 `Remove` | 仅主模拟；入包停融、manager 登记期暂停融化、放下恢复；非休眠才有 FX | [[reference/components/snowballmelting]]；[生成](../../../raw/current/scripts/prefabs/snowball_item.lua#L151-L157)、[挂接](../../../raw/current/scripts/prefabs/snowball_item.lua#L265-L267)、[检查](../../../raw/current/scripts/components/snowballmelting.lua#L30-L72) |
| 生成者：冬宴清场 | snowballmanager `OnUpdate` 停用分支：`issnowcovered` 为假时每 0.5 秒（`SNOWBALLMANAGER_SECONDS_PER_DESPAWN`）移除一只登记雪球，非休眠先落 FX | 仅 forest 世界实体挂载（构造断言 mastersim）；冬宴降雪停止后逐只清 | [[reference/components/snowballmanager]]；[清场](../../../raw/current/scripts/components/snowballmanager.lua#L172-L190)、[挂载](../../../raw/current/scripts/prefabs/forest.lua#L688)、[调谐](../../../raw/current/scripts/tuning.lua#L8573-L8576) |
| 生成者：雪人锤碎 | snowman workable（HAMMER）`OnWork` 的锤碎分支走 `DoBreakApart`：落点后按尺寸掉 `snowball_item` 战利品，实体 `Remove` | 锤到底（isdestroyed=true）与裸球（false）两分支都经 DoBreakApart；非融化态才掉战利品，FX 无条件 | Wiki 待建：[DoBreakApart](../../../raw/current/scripts/prefabs/snowman.lua#L506-L529)、[OnWork](../../../raw/current/scripts/prefabs/snowman.lua#L531-L560)、[workable](../../../raw/current/scripts/prefabs/snowman.lua#L693-L696) |
| 仅声明 | forest 世界预载、snowball_item／snowman prefabs 数组 | 均非生成点 | [forest](../../../raw/current/scripts/prefabs/forest.lua#L504-L506)、[snowball_item](../../../raw/current/scripts/prefabs/snowball_item.lua#L9)、[snowman](../../../raw/current/scripts/prefabs/snowman.lua#L15) |
| 资源 | bank/build `snowball`（与 snowball_item 同资源）、anim `fx_place` | 动画内容与时长未核验 | [条目](../../../raw/current/scripts/fx.lua#L3710-L3715) |

## 典型接入

### 复刻"实体原地消失"的碎裂表现

- 前提与选择：主模拟任意位置 `SpawnPrefab` 落点后移除宿主；原版三路径都在移除前先落 FX（雪球堆清场还带休眠门）。
- 生效与撤销：一次 `fx_place` 后 `animover` 自删；无音效，需要声音由调用方另配。
- 原版依据：[snowball_item 融尽](../../../raw/current/scripts/prefabs/snowball_item.lua#L151-L157)。

```lua
local x, y, z = inst.Transform:GetWorldPosition()
if not inst:IsAsleep() then
    SpawnPrefab("snowball_shatter_fx").Transform:SetPosition(x, y, z)
end
inst:Remove()
```

- 调整与验证：落点用实体原坐标（不抬 y）；要命中水花效果换 `splash_snow_fx`。

## 常见陷阱与限制

- **误区：雪球投掷命中播本特效**：projectile `OnHit` 对目标落 `splash_snow_fx` 并 `Remove` 雪球本体，本注册不出现在投掷链路；本名只用于实体原地消失（融尽／清场／锤碎）。[OnHit](../../../raw/current/scripts/prefabs/snowball_item.lua#L41-L49)
- **边界：休眠实体无表现**：融尽路径 `not inst:IsAsleep()`、清场路径 `not snowball:IsAsleep()` 才生成 FX，远处雪球消失无动画（锤碎路径无此门）。[融尽](../../../raw/current/scripts/prefabs/snowball_item.lua#L151-L157)、[清场](../../../raw/current/scripts/components/snowballmanager.lua#L180-L183)
- **边界：清场要等积雪消融**：`issnowcovered` 为真时清场分支整个跳过，停雪但未化雪期间雪球留在原地；积雪没了才逐只移除。[分支](../../../raw/current/scripts/components/snowballmanager.lua#L174-L186)
- **边界：融尽不是实时判定**：温度检查 30±5 秒一拍，放下雪球后最早也要等到下一拍才融尽消失。[间隔](../../../raw/current/scripts/components/snowballmelting.lua#L1-L2)

## 验证记录

- 源码审阅：2026-11-01，steam-buildid:26335394。核验 fx.lua 条目与 FinalOffset2 helper、加载器 prefabs/fx.lua、snowball_item 融尽回调与组件挂接、snowballmelting 检查循环、snowballmanager 清场分支与 forest 挂载、snowman DoBreakApart／OnWork／workable、tuning 调谐值。检索范围：`snowball_shatter` 在 raw/current/scripts 全域（根级与 prefabs／components／stategraphs）rg，命中仅上述注册、三个生成点与三处声明，无动态拼名。`.po` 名称键定向核验无特效显示键。未核验：`fx_place` 动画资源内容（时长／循环）、引擎休眠判定细节、`GetTemperatureAtXZ` 的温度场实现。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]、[[reference/prefabs/index|Prefab 索引]]。
