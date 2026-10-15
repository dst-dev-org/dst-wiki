---
title: "带翅膀的火盆添柴烟焰（firepit_firebird_puff_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "火盆皮肤 firepit_firebird（官方名带翅膀的火盆）添柴时的一次性火蓬：网络代理由主模拟写入 fuel 等级 netvar，各端本地生成非联网子实体按 puff1/2/3 播一次；皮肤符号覆写自 dynamic 资源。"
aliases: ["firepit_firebird_puff_fx", "带翅膀的火盆", "火盆皮肤特效", "firepit firebird skin", "Winged Brazier"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/firepit_firebird_puff_fx.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 带翅膀的火盆添柴烟焰（firepit_firebird_puff_fx）

## 快速判断

- 职责与适用：火盆皮肤 `firepit_firebird`（官方名"带翅膀的火盆"，[SKIN_NAMES](../../../raw/current/scripts/languages/chinese_s.po#L396479-L396481)）装备后，每次添柴（takefuel）在火盆上方播一次火蓬动画；等级 1/2/3 对应 `puff1/2/3`（[添柴触发](../../../raw/current/scripts/prefabskin.lua#L2339-L2353)）。
- 优先使用：查皮肤如何给建筑附加特效 → prefabskin 的 `SKIN_FX_PREFAB` 机制；查 anim 覆写 → 本页注册与构造的 OverrideItemSkinSymbol。
- 重要副作用：纯表现，无玩法组件；代理 1 秒自毁，视觉 animover 自删（[代理清理](../../../raw/current/scripts/prefabs/firepit_firebird_puff_fx.lua#L59-L60)、[视觉清理](../../../raw/current/scripts/prefabs/firepit_firebird_puff_fx.lua#L31)）。
- 修改边界：只在 takefuel 时触发，火势等级自行变化（烧尽／下雨）不播；专服只建代理不建视觉。[触发条件](../../../raw/current/scripts/prefabskin.lua#L2341-L2353)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `firepit_firebird_puff_fx` | fn：Transform＋Network、FX 标签 → 非专服 `DoTaskInTime(0, PlayPuffAnim)`（SetPristine 前，双端本地执行）→ `level = net_tinybyte`（默认 2）→ SetPristine → 主模拟 `persists=false`、1 秒自毁；PlayPuffAnim 另建**非联网**子实体挂到代理父实体（火盆）：bank `firepit_firebird`、build `campfire_fire`、`OverrideItemSkinSymbol("fire_puff", "firepit_firebird", …, "campfire_fire")`、播 `puff{level}`（未显式传循环参数，实际是否重复未核验）、bloom、FinalOffset 3、animover 自删 | 单一注册；代理＋本地视觉两实体，level 经 netvar 从主模拟传到各端。[fn](../../../raw/current/scripts/prefabs/firepit_firebird_puff_fx.lua#L35-L63)、[PlayPuffAnim](../../../raw/current/scripts/prefabs/firepit_firebird_puff_fx.lua#L8-L33)、[注册](../../../raw/current/scripts/prefabs/firepit_firebird_puff_fx.lua#L65) |

皮肤键 `STRINGS.SKIN_NAMES.firepit_firebird` 为"带翅膀的火盆"；FX 注册名本身无 NAMES 条目。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 皮肤机制：SKIN_FX_PREFAB | 皮肤定义 `fx_prefab = { "firepit_firebird_puff_fx" }` 在注册期写入全局表；`firepit_init_fn` 取 `skin_fx[1]` 挂 takefuel 监听 | 仅 firepit 类皮肤走该 init；placer/客户端只设外观不挂监听 | [皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L5477-L5487)、[init_fn](../../../raw/current/scripts/prefabskin.lua#L2327-L2359)、[[reference/prefabs/cane_victorian_fx#关联机制导航|同类机制参照]] |
| 事件：takefuel | 监听火盆 `takefuel`：`fuelvalue ≥ LARGE_FUEL(6 段)` → level 3、`≥ MED_FUEL(1.5 段)` → 2、否则 1；父实体为火盆 | fuelvalue≤0 不生成（原版添柴均大于 0）；level 在生成后立即 set；晚加入时序未运行核验 | [等级映射](../../../raw/current/scripts/prefabskin.lua#L2341-L2353)、[燃料档](../../../raw/current/scripts/tuning.lua#L1674-L1677) |
| 资源 | anim/campfire_fire.zip＋dynamic/firepit_firebird.zip（含 PKGREF .dyn）——puff 符号在皮肤 build 里，其余取自 campfire_fire | dynamic 资源按需加载 | [assets](../../../raw/current/scripts/prefabs/firepit_firebird_puff_fx.lua#L1-L6)、[覆写](../../../raw/current/scripts/prefabs/firepit_firebird_puff_fx.lua#L24-L27) |

## 典型接入

### 给建筑皮肤附加一次性事件特效

- 前提与选择：皮肤定义 `prefabs` 声明依赖＋`fx_prefab` 注册表项；在对应 `*_init_fn` 里按 `SKIN_FX_PREFAB[build_name]` 取列表挂业务事件（原版挂 takefuel，可换任意事件）。
- 生效与撤销：SpawnPrefab 后 `SetParent(inst)`＋写 netvar；代理 1 秒自毁、视觉 animover 自删，无需手动清理；自定义实现也须在卸皮肤时移除本例 `_on_takefuel` 回调；原版 `firepit_clear_fn` 移除的是自己的 `_takefuel_fn`（[clear](../../../raw/current/scripts/prefabskin.lua#L2360-L2371)）。
- 原版依据：本特效即"皮肤依赖声明 → SKIN_FX_PREFAB → init_fn 事件挂接"的完整链路。

```lua
-- 皮肤 init_fn 内（主模拟）：
local skin_fx = SKIN_FX_PREFAB[build_name]
if skin_fx ~= nil and skin_fx[1] ~= nil then
    inst._on_takefuel = function(inst, data)
        local fuelvalue = data ~= nil and data.fuelvalue or 0
        if fuelvalue > 0 then
            local fx = SpawnPrefab(skin_fx[1])
            fx.entity:SetParent(inst.entity)
            fx.level:set((fuelvalue >= TUNING.LARGE_FUEL and 3)
                or (fuelvalue >= TUNING.MED_FUEL and 2) or 1)
        end
    end
    inst:ListenForEvent("takefuel", inst._on_takefuel)
end
```

- 调整与验证：等级阈值写死在 init_fn（LARGE/MED_FUEL）；动画名拼接 `puff..level`，自定义 build 需提供同名动画。

## 常见陷阱与限制

- **误区：火势变化会自动播烟焰**：触发点只有 takefuel 事件；火自己烧大／熄灭不会生成 puff（[唯一监听](../../../raw/current/scripts/prefabskin.lua#L2341-L2353)）。
- **边界：build 是 campfire_fire 不是皮肤 build**：AnimState 基础 build 用 campfire_fire，皮肤符号经 OverrideItemSkinSymbol 覆写并提供 `campfire_fire` 回退；漏 PKGREF/dyn 会加载失败（[覆写](../../../raw/current/scripts/prefabs/firepit_firebird_puff_fx.lua#L24-L27)、[资源](../../../raw/current/scripts/prefabs/firepit_firebird_puff_fx.lua#L1-L6)）。
- **边界：双端本地视觉**：视觉实体非联网，靠 level netvar 同步等级；主模拟 1 秒自毁意味着 level 必须在生成后立刻 set，本地视觉只在创建时读取一次 level，晚写不触发重播；晚加入表现未逐端核验。
- **边界：跟随父实体位置**：视觉 `SetFromProxy(proxy.GUID)` 取代理位置，父实体是火盆而非火焰子实体——高度动画由 puff 动画自带（[定位](../../../raw/current/scripts/prefabs/firepit_firebird_puff_fx.lua#L21-L22)）。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；工厂全文（代理／视觉、level netvar、清理）、skinprefabs 皮肤定义、prefabskin firepit_init_fn／clear_fn、燃料档数值、皮肤译名已核验；`puff1/2/3` 实际帧数、火盆其余皮肤（fangedp 等）是否复用同机制未逐一展开、晚加入时序未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
