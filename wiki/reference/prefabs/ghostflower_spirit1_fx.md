---
title: "幽灵花灵体飘光（ghostflower_spirit1_fx）"
created: "2026-11-01"
updated: "2026-11-05"
type: prefab
summary: "两个数据完全同构的通用表注册：幽灵花物品落地期间的周期灵体飘光与装饰墓碑花瓣安放落光；随机二选一或固定 1 号，纯表现，与幽灵灵药机制无关。"
aliases: ["ghostflower_spirit1_fx", "ghostflower_spirit2_fx", "幽灵花灵体", "灵体飘光", "幽灵花特效", "哀悼荣耀特效"]
tags: [prefab, fx]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/ghostflower.lua", "prefabs/gravestone.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 幽灵花灵体飘光（ghostflower_spirit1_fx）

## 快速判断

- 职责与适用：两个通用表注册承载幽灵花（ghostflower，官方名"哀悼荣耀"Mourning Glory）相关的一次性灵体飘光；纯表现，无 fn、无任何玩法作用。[条目组](../../../raw/current/scripts/fx.lua#L1141-L1154)
- 优先使用：给灵体类落点表现选资源时直接 `SpawnPrefab` 本族名（自带飘光动画与 wisp 音效）；工厂公共协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。
- 重要副作用：条目自带 `dontstarve/characters/wendy/small_ghost/wisp` 音效，本地表现创建时播放一次。[条目](../../../raw/current/scripts/fx.lua#L1141-L1146)
- 修改边界：落点式一次性表现，不父挂、不跟随；与幽灵灵药无机制关系（ghostflower 只是灵药配方材料，见 [[reference/prefabs/ghostlyelixir_slowregen_fx|幽灵灵药特效族]]）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `ghostflower_spirit1_fx` | 通用表条目，无 fn：bank=`build`=`ghostflower`、anim=`fx1`、sound=`.../small_ghost/wisp` | 两注册条目数据完全同构，唯一差异是注册名。[条目](../../../raw/current/scripts/fx.lua#L1141-L1146) |
| `ghostflower_spirit2_fx` | 同上协议 | 数据与 1 号逐字段相同（同 bank／build／anim／sound），仅注册名不同；两名的分工只在生成方选名逻辑。[条目](../../../raw/current/scripts/fx.lua#L1148-L1154) |

两注册名在简中 .po 无 `STRINGS.NAMES` 显示名键；标题中文为已核验效果描述、非官方译名。宿主物品名 `STRINGS.NAMES.GHOSTFLOWER`="哀悼荣耀"见 [po:343199-343202](../../../raw/current/scripts/languages/chinese_s.po#L343199-L343202)。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 经 `prefabs/fx.lua` 加载器注册；落点式：生成方不父挂代理，本地表现一次性复制代理位置 | 服务端 `SpawnPrefab`，各非专服模拟建表现；animover 自删、不入存档 | [[reference/mechanisms/generic-fx-factory]]；[回收](../../../raw/current/scripts/prefabs/fx.lua#L98-L102) |
| 生成者：幽灵花物品 `dofx` | 花落地期间自续任务：每 3＋随机×6 秒生成 `ghostflower_spirit"..tostring(math.random(2)).."_fx`（动态拼名随机二选一）后重排下次 | 回调执行时才检查 `not inst.inlimbo`：拾入背包不会立即取消已排队任务，下一次回调发现仍在 limbo 后不再生成／续排；再落地（`ondropped`→`toground`）另排一条，旧任务到期前拾起又丢下时两条链可能并存 | [dofx](../../../raw/current/scripts/prefabs/ghostflower.lua#L14-L21)、[落地重启](../../../raw/current/scripts/prefabs/ghostflower.lua#L34-L39) |
| 生成者：装饰墓碑 `OnPetalAdded` | 装饰墓碑 upgradeable 升级（安放花瓣）时在墓碑位置固定生成 `ghostflower_spirit1_fx` | 服务端；固定 1 号、不经随机选名 | [OnPetalAdded](../../../raw/current/scripts/prefabs/gravestone.lua#L131-L134)、[注册回调](../../../raw/current/scripts/prefabs/gravestone.lua#L275-L275) |
| 资源 | anim：`ghostflower.zip`（与宿主物品同 build） | 加载器按 build 声明 ANIM 资产 | [资产](../../../raw/current/scripts/prefabs/fx.lua#L5-L16) |

依赖声明与实际生成分开核验：ghostflower.lua 的 `prefabs` 数组（[L8-L12](../../../raw/current/scripts/prefabs/ghostflower.lua#L8-L12)）只是声明，实际生成在 `dofx` 的动态拼名处。

## 常见陷阱与限制

- **边界：两注册是同构双名**：条目数据逐字段相同，选 2 号只会改变注册名（表现是否因动画内部随机帧而异属动画数据，scripts 不可核验）；墓碑路径固定用 1 号，只有幽灵花随机路径会出 2 号。[生成方](../../../raw/current/scripts/prefabs/ghostflower.lua#L16-L16)、[墓碑](../../../raw/current/scripts/prefabs/gravestone.lua#L133-L133)
- **边界：与幽灵灵药无机制关系**：ghostflower 是九种灵药的配方材料（[recipes.lua:151-159](../../../raw/current/scripts/recipes.lua#L151-L159)），但本族两注册不进入灵药 buff 链，也不受任何 tunings 引用；归属见 [[reference/prefabs/ghostlyelixir_slowregen_fx|幽灵灵药特效族]] 的拆页说明。
- **边界：表现节奏由生成方任务决定**：本族条目无 `update_while_paused`，飘光周期（3-9 秒随机）是幽灵花物品的自续任务，不是 fx 实体自身行为。[dofx](../../../raw/current/scripts/prefabs/ghostflower.lua#L19-L19)

## 验证记录

- 源码审阅：2026-10-31，Steam buildid 26335394。fx.lua 两条目逐字段核对；生成方核验 ghostflower.lua 全文（dofx 自续链条、ondropped 重启、inlimbo 门）与 gravestone.lua 花瓣回调；另引 `prefabs/fx.lua` 的工厂回收协议与 `recipes.lua` 的九药水配方关系；`rg "ghostflower_spirit"` 全 scripts 根仅命中 fx.lua、ghostflower.lua、gravestone.lua 三处，生成点已全覆盖。未覆盖：`fx1` 动画内容与随机帧、两注册在动画层面的实际差异（若有）。
- 游戏运行：未运行；飘光视觉与音效未实测。
- 结构检查：2026-10-31 validate_page.py 与 validate_source_links.py --strict 通过（见本页文件）；当前已登记于 [fx-table-index](fx-table-index.md)。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/prefabs/index|Prefab 索引]]。
