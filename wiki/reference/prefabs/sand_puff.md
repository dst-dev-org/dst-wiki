---
title: "沙尘与暗影烟团（sand_puff 族）"
created: "2026-10-30"
updated: "2026-11-08"
type: prefab
summary: "通用表九个烟团注册：沙色三项、黑影五项（透明／实心／放大／前后分层）与拟态烟团，覆盖懒惰掠夺者拾取、传送法杖 poof、玫瑰眼镜燃料呈现、表演瞬移等生成者。"
aliases: ["sand_puff", "sand_puff_large_front", "sand_puff_large_back", "shadow_puff", "shadow_puff_solid", "shadow_puff_solid_large", "shadow_puff_large_front", "shadow_puff_large_back", "itemmimic_puff", "沙尘烟团", "暗影烟团", "黑烟", "烟团", "传送烟", "拟态烟团", "影子烟"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/fx.lua", "fx.lua", "prefabs/amulet.lua", "tuning.lua", "prefabs/staff_common.lua", "components/blinkstaff.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "componentutil.lua", "play_commonfn.lua", "prefabs/abigail.lua", "prefabs/boatrace_start.lua", "components/yoth_knightmanager.lua", "stategraphs/SGitemmimic_revealed.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 沙尘与暗影烟团（sand_puff 族）

## 快速判断

- 职责与适用：通用表里以 `sand_puff` bank／`forage_out` 动画为底的烟团家族：沙色（拾取／拾荒）与黑影（暗影主题皮肤、技能、演出）两组加拟态烟团；纯表现，无玩法组件。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L200-L311)
- 优先使用：查"拾取烟／传送烟／黑影消散"选本页；传送烟的生成时机（起点与终点各一对）见下表 blinkstaff 行。
- 重要副作用：无；分层变体靠 `Hide("front")／Hide("back")` 拆同一 bank 的前后层，成对生成才完整。[分层](../../../raw/current/scripts/fx.lua#L207-L227)
- 修改边界：一次性落点表现；动画是否循环未核验。[代理回收](../../../raw/current/scripts/prefabs/fx.lua#L156-L157)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `sand_puff` | 通用表条目 → 加载器代理＋一帧后本地实体 | 沙色基准：anim=`forage_out`，音效 deathpoof，无 fn／tint。[条目](../../../raw/current/scripts/fx.lua#L200-L205) |
| `sand_puff_large_front` | 同上＋fn 内联 | 1.5 倍缩放，`SetFinalOffset(2)` 且 `Hide("back")` 只显前层。[条目](../../../raw/current/scripts/fx.lua#L207-L217) |
| `sand_puff_large_back` | 同上＋fn 内联 | 1.5 倍缩放，`Hide("front")` 只显后层，无 FinalOffset。[条目](../../../raw/current/scripts/fx.lua#L219-L227) |
| `shadow_puff` | 同上＋fn 内联 | 黑 tint×0.5 透明，`SetFinalOffset(2)`。[条目](../../../raw/current/scripts/fx.lua#L252-L262) |
| `shadow_puff_solid` | 同上＋fn 内联 | 黑 tint 全不透明，`SetFinalOffset(2)`。[条目](../../../raw/current/scripts/fx.lua#L264-L273) |
| `shadow_puff_solid_large` | 同上＋fn 内联 | 实心＋1.5 倍缩放＋FinalOffset 2。[条目](../../../raw/current/scripts/fx.lua#L275-L285) |
| `shadow_puff_large_front` | 同上＋fn 内联 | 半透明黑＋1.5 倍＋FinalOffset 2＋`Hide("back")`。[条目](../../../raw/current/scripts/fx.lua#L287-L299) |
| `shadow_puff_large_back` | 同上＋fn 内联 | 半透明黑＋1.5 倍＋`Hide("front")`。[条目](../../../raw/current/scripts/fx.lua#L301-L311) |
| `itemmimic_puff` | 同上＋fn=`FinalOffset2` helper | 半透明黑＋FinalOffset 2，音效换 `rifts4/mimic/item_become_poof`。[条目](../../../raw/current/scripts/fx.lua#L3690-L3698) |

定向检索简中 .po 未发现这 9 个注册名的 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，差异仅缩放／tint／分层 fn／音效 | 全族同协议 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L200-L311) |
| 生成者：懒惰掠夺者（orangeamulet） | 装备后每 0.33 秒在 4 距离内吸附一件拾取物，原物品位置播 `sand_puff` 并消耗 1 耐久（总 225 次） | 服务端；官方名 The Lazy Forager／懒人护符 | [拾取](../../../raw/current/scripts/prefabs/amulet.lua#L188-L223)、[周期挂载](../../../raw/current/scripts/prefabs/amulet.lua#L228-L231)、[调参](../../../raw/current/scripts/tuning.lua#L214-L216)、[译名](../../../raw/current/scripts/languages/chinese_s.po#L346050-L346052) |
| 生成者：懒人魔杖（orangestaff） | `blinkstaff:SetFX(front, back)` 配对；起跳与落点各 `SpawnEffect` 一对：back 层先于 front 层、低 0.1 落点 | 服务端施法链；0.25 秒后传送并二次生成 | [配置](../../../raw/current/scripts/prefabs/staff_common.lua#L542-L543)、[起点](../../../raw/current/scripts/components/blinkstaff.lua#L26-L34)、[落点](../../../raw/current/scripts/components/blinkstaff.lua#L51-L56)、[[reference/components/blinkstaff|blinkstaff]] |
| 皮肤覆盖：orangestaff_ancient | 皮肤 fx_prefab[3]/[4] 把传送烟换成 `shadow_puff_large_front/back`，[2] 另挂 cane_ancient_fx 拖尾 | 仅远古皮肤；init_fn 主模拟执行 | [皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8842-L8852)、[应用](../../../raw/current/scripts/prefabskin.lua#L3315-L3330) |
| 生成者：玫瑰眼镜燃料呈现 | 残余物激活产出 nightmarefuel／horrorfuel（温娜 winona_charlie_2 升级），0.5 秒呈现链：charlie_snap（[[reference/prefabs/charlie_snap|charlie_snap 页]]；升级**或洞穴**选 `_solid`）→ 25 帧后 `shadow_puff`（**仅升级**选 `_solid`）→ 物品回场 | 服务端；洞穴条件只作用于前一步 charlie_snap | [入口](../../../raw/current/scripts/componentutil.lua#L616-L632)、[烟团步](../../../raw/current/scripts/componentutil.lua#L606-L615) |
| 生成者：舞台剧瞬移 | 表演 castmember 台词中途 `teleport_to_position`：起点必生成 `shadow_puff_solid`，终点仅当位移平方距离 >0.01（0.1 距离）才再生成一个 | 服务端演出脚本调用 | [瞬移](../../../raw/current/scripts/play_commonfn.lua#L333-L341)、[调用](../../../raw/current/scripts/play_commonfn.lua#L364-L365) |
| 生成者：阿比盖尔击杀增益 | 温蒂技能 murder buff 挂上／摘除时在目标位置播 `shadow_puff_large_front` 并额外 1.2 倍缩放 | 服务端；随 planardefense bonus 生命周期 | [挂上](../../../raw/current/scripts/prefabs/abigail.lua#L1233-L1237)、[摘除](../../../raw/current/scripts/prefabs/abigail.lua#L1259-L1263) |
| 其他生成者 | 皮影船移除 3 倍缩放 [boatrace_start](../../../raw/current/scripts/prefabs/boatrace_start.lua#L332-L336)；骑士揭晓 `shadow_puff_solid_large` [yoth_knightmanager](../../../raw/current/scripts/components/yoth_knightmanager.lua#L75-L79)；物品拟态成功 `itemmimic_puff` 后自删 [SGitemmimic_revealed](../../../raw/current/scripts/stategraphs/SGitemmimic_revealed.lua#L145-L148) | 服务端；各自一次性触发 | — |
| 资源 | bank／build=`sand_puff`，全族共用 `forage_out` 动画 | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L200-L311) |

## 典型接入

### 借用烟团表现

- 前提与选择：拾取／出现类用 `sand_puff`；暗影主题换 `shadow_puff` 组；需要"放大＋前后层"完整大烟团时按 back＋front 成对生成。
- 生效与撤销：落点一次性动画，animover 自删；起点／落点各播一次是 blinkstaff 的调用方式，不是工厂行为。
- 原版依据：[blinkstaff SpawnEffect](../../../raw/current/scripts/components/blinkstaff.lua#L26-L34)。

```lua
local back = SpawnPrefab("sand_puff_large_back")
back.Transform:SetPosition(x, y - 0.1, z)
local front = SpawnPrefab("sand_puff_large_front")
front.Transform:SetPosition(x, y, z)
```

- 调整与验证：自定义皮肤想换传送烟时，仿照 fx_prefab 第 3／4 项并在 init_fn 调 `blinkstaff:SetFX`；第 1／2 项空字符串会被置 nil（[置空逻辑](../../../raw/current/scripts/prefabskin.lua#L3322-L3325)）。

## 常见陷阱与限制

- **误区：large_front／large_back 是两个不同动画**：同一 bank 同一 anim，靠 `Hide` 互斥符号拆前后层；只生成一个会缺半团。[分层](../../../raw/current/scripts/fx.lua#L207-L227)
- **误区：传送烟默认就是黑影**：懒人魔杖默认用沙色 large 前后对，仅 orangestaff_ancient 皮肤换成 shadow 对。[皮肤](../../../raw/current/scripts/prefabskin.lua#L3328-L3330)
- **边界：shadow_puff_large_back 无直接生成点**：全库检索仅皮肤 fx_prefab 第 4 项与皮肤依赖声明引用它，经 blinkstaff backfx 间接生成；不要把它当独立入口推荐。[间接](../../../raw/current/scripts/prefabskin.lua#L3328-L3330)
- **边界：solid 变体的实心程度来自 tintalpha**：`shadow_puff`×0.5 与 `_solid`×1.0 同资源；仅升级选 `shadow_puff_solid`，升级或洞穴选 `charlie_snap_solid`，均为生成者逻辑，不是工厂分支。[玫瑰眼镜链](../../../raw/current/scripts/componentutil.lua#L606-L615)
- **边界：动画是否循环未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。九条目配置、加载器协议、懒惰掠夺者拾取链、blinkstaff 起落双段烟团与皮肤覆盖、玫瑰眼镜呈现链、舞台剧／阿比盖尔／皮影船／骑士／拟态各生成点逐条核验；.po 名称键定向核验无结果（懒惰掠夺者／懒人魔杖为消费者译名）。未覆盖：sand_puff 动画资源内容、bathbombable 等未列消费者（sand_puff 在本快照仅懒惰掠夺者一个生成点）。
- 游戏运行：未运行；烟团视觉层次与皮肤替换效果未实测。

返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
