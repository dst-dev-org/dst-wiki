---
title: "小爆裂、换装烟与火药爆裂（explode_small 族）"
created: "2026-11-02"
updated: "2026-11-07"
type: prefab
summary: "十个同工厂爆裂和换装烟注册；记录动画、声音、皮肤选名及生成方的玩法边界。"
aliases: ["explode_small", "explode_reskin", "reskin_tool_brush_explode_fx", "reskin_tool_bouquet_explode_fx", "reskin_tool_wand_explode_fx", "reskin_tool_toilet_explode_fx", "explode_small_slurtle", "explode_small_slurtlehole", "explode_firecrackers", "slingshotammo_gunpowder_explode", "小爆裂", "换装烟", "火药爆裂"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/explode_small.lua", "prefabs/gunpowder.lua", "prefabs/slurtleslime.lua", "prefabs/slurtle.lua", "prefabs/slurtlehole.lua", "prefabs/rock_light.lua", "prefabs/worm_boss_util.lua", "prefabs/reskin_tool.lua", "prefabs/skinprefabs.lua", "prefabs/beefalo.lua", "prefabs/wx78_backupbody.lua", "prefabs/wx78_possessedbody.lua", "prefabs/firecrackers.lua", "prefabs/slingshotammo.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 小爆裂、换装烟与火药爆裂（explode_small 族）

## 快速判断

- 十个注册使用 `MakeExplosion`；专属工厂创建网络代理，在非专服延迟创建本地动画实体，二者分别清理。伤害、换皮、惊吓等玩法属于生成者。[工厂](../../../raw/current/scripts/prefabs/explode_small.lua#L6-L79)
- 中文为按动画和调用语境核验的效果描述，非官方译名；简中 `languages/chinese_s.po` 定向检索十个注册名的 `STRINGS.NAMES` 键均未命中。动画素材内容与运行表现未核验。

## 注册与构造

`MakeExplosion(data)` 的本地实体从代理取初始变换；默认 bank/build 为 `explode`，默认 anim `small`，默认声 `dontstarve/common/blackpowder_explo`，LightOverride 1。表中资源字段相同处只记差异；工厂声明 `anim/explode.zip`，换皮 build／符号来自相关皮肤资源，不能仅凭源码声明保证资源内容。[资源](../../../raw/current/scripts/prefabs/explode_small.lua#L1-L4)、[数据消费](../../../raw/current/scripts/prefabs/explode_small.lua#L21-L50)、[注册](../../../raw/current/scripts/prefabs/explode_small.lua#L155-L164)

| 注册名 | 配置差异 | 证据 |
|---|---|---|
| `explode_small` | 无 data；默认动画、声音；由于 bloom 判断要求 data 非空，此注册不执行设置 bloom | [工厂](../../../raw/current/scripts/prefabs/explode_small.lua#L31-L35)、[注册](../../../raw/current/scripts/prefabs/explode_small.lua#L155-L155) |
| `explode_reskin` | bank `fx_shadow_dust`、build `reskin_tool_fx`、anim `puff`；换皮声；启用 bloom | [数据](../../../raw/current/scripts/prefabs/explode_small.lua#L84-L89) |
| `reskin_tool_brush_explode_fx` | bank `fx_shadow_dust`、anim `puff`；`reskin_tool_brush` 的 `shadow_dust` 符号覆盖，刷子声；禁 bloom，LightOverride 0 | [数据](../../../raw/current/scripts/prefabs/explode_small.lua#L91-L99) |
| `reskin_tool_bouquet_explode_fx`／`reskin_tool_wand_explode_fx`／`reskin_tool_toilet_explode_fx` | 与刷子同 bank／anim／符号及禁 bloom、LightOverride 0；各以对应 `reskin_tool_bouquet`／`_wand`／`_toilet` build 覆盖，使用换皮声 | [三项](../../../raw/current/scripts/prefabs/explode_small.lua#L101-L129) |
| `explode_small_slurtle`／`explode_small_slurtlehole` | 默认动画，分别使用蜗牛爆炸声／巢爆炸声；data 非空故设置 bloom | [数据](../../../raw/current/scripts/prefabs/explode_small.lua#L131-L137) |
| `explode_firecrackers` | anim `small_firecrackers`，本地缩放 0.5，声音随机 start 参数 | [数据](../../../raw/current/scripts/prefabs/explode_small.lua#L139-L145) |
| `slingshotammo_gunpowder_explode` | 默认动画及爆炸声，FinalOffset 3、本地缩放 1.8、LightOverride 0.2；设置 bloom | [数据](../../../raw/current/scripts/prefabs/explode_small.lua#L147-L151) |

## 关联机制导航

| 生成者／关系 | 触发与落点 | 证据 |
|---|---|---|
| 火药与蜗牛黏液 | 爆炸回调停止 hiss 并在物体位置生成 `explode_small`；伤害由 explosive 组件等生成者处理。 | [火药](../../../raw/current/scripts/prefabs/gunpowder.lua#L21-L24)、[黏液](../../../raw/current/scripts/prefabs/slurtleslime.lua#L21-L24) |
| 蜗牛及蜗牛巢 | 各自爆炸回调停止声音后在本体位置分别生成 `explode_small_slurtle`／`explode_small_slurtlehole`；声明中的 `explode_small` 不等于实际选名。 | [蜗牛](../../../raw/current/scripts/prefabs/slurtle.lua#L76-L79)、[巢](../../../raw/current/scripts/prefabs/slurtlehole.lua#L77-L80) |
| 火山岩与蠕虫首领 | `rock_light` 的爆裂在岩体坐标 y 加 2 处生成 `explode_small`；蠕虫首领工具函数在部分分段及尾部落点生成，分段有延时和随机门禁。 | [岩](../../../raw/current/scripts/prefabs/rock_light.lua#L92-L101)、[首领](../../../raw/current/scripts/prefabs/worm_boss_util.lua#L1536-L1550) |
| 清洁扫把换皮 | `reskin_tool` 默认选 `explode_reskin`；皮肤 `SKIN_FX_PREFAB` 数组首项存在时覆盖，四种皮肤名在 `skinprefabs.lua` 的 fx_prefab 中声明。目标与用户资格检查通过后生成并按目标位置及偏移设置代理；换皮操作另在延时回调。 | [选名与生成](../../../raw/current/scripts/prefabs/reskin_tool.lua#L213-L253)、[皮肤声明](../../../raw/current/scripts/prefabs/skinprefabs.lua#L10464-L10507) |
| 其他换装烟 | 皮弗娄牛睡眠时清衣且存在皮肤才生成 `explode_reskin`；WX78 备用身体及附身身体关联玩家所有者时，按 `_hide_body_skinfx` 门禁生成。 | [牛](../../../raw/current/scripts/prefabs/beefalo.lua#L411-L421)、[备用身体](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L139-L149)、[附身身体](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L57-L66) |
| 爆竹与弹弓火药 | 爆竹 `DoPop` 本体坐标生成 `explode_firecrackers`，惊吓事件在调用方；弹弓火药命中暴击时在目标位置生成其专属爆裂、另做 AOE 并抑制普通命中 FX。 | [爆竹](../../../raw/current/scripts/prefabs/firecrackers.lua#L11-L20)、[火药弹](../../../raw/current/scripts/prefabs/slingshotammo.lua#L810-L831)、[[reference/prefabs/slingshotammo_hitfx_rock#关联机制导航]] |

## 常见陷阱与限制

- 本地动画与网络代理不是同一实体：非专服的零延时任务创建动画、动画 `animover` 移除自身；服务端代理在一秒后移除。若代理在任务前移除，本地动画可能未创建。[工厂](../../../raw/current/scripts/prefabs/explode_small.lua#L6-L79)
- `explode_small` 本身不负责爆炸伤害；换皮工具设置的 Transform 作用于代理，本地动画创建时从代理取变换，不能据此推断其后持续跟随代理。[调用](../../../raw/current/scripts/prefabs/reskin_tool.lua#L239-L248)、[工厂](../../../raw/current/scripts/prefabs/explode_small.lua#L19-L23)
- 默认注册未设置 bloom，但带 data 且未明确 `bloom=false` 的变体会设置；不要把十项的 bloom 当作一致。[分支](../../../raw/current/scripts/prefabs/explode_small.lua#L31-L35)

## 验证记录

- 源码审阅：2026-11-02，Steam buildid 26335394；核验十个注册、数据表字段、工厂及上列直接生成者；对整个 `raw/current/scripts` 检索十个完整名及 `explode_small`、`explode_reskin`、`reskin_tool`、`fx_prefab` 等选名字段，追查皮肤数组选名并区分声明与生成。未核验动画包内内容、其他模组调用或运行时音画。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]。
