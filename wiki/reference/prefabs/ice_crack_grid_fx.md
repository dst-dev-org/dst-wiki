---
title: "海冰网格裂纹（ice_crack_grid_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "海洋冰面破碎前的地面裂纹提示：联网地面动画实体，由 oceanicemanager 在冰面瓦片四周朝陆地一侧铺设，冰面销毁时统一清除。"
aliases: ["ice_crack_grid_fx", "海冰裂纹", "冰面裂纹", "ice crack fx"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/ice_crack_grid_fx.lua", "components/oceanicemanager.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 海冰网格裂纹（ice_crack_grid_fx）

## 快速判断

- 职责与适用：海洋冰面（oceanice）被判定即将销毁时，在冰面瓦片四边朝陆地一侧铺设的**裂纹提示动画**：联网地面实体（gridplacer bank），带 `ice_crack_fx` 标签作为管理器查重与清理的句柄。[工厂](../../../raw/current/scripts/prefabs/ice_crack_grid_fx.lua#L11-L39)
- 优先使用：查"冰面裂纹／破坏提示"选本页；冰面血量与销毁排程在 [[reference/components/oceanicemanager]]（其"关键流程：一次冰面破坏"一节）。
- 重要副作用：主动画随机播 `left` 或 `right` 各一次，未显式传循环参数；实体自身无移除路径，由管理器在瓦片销毁时按标签 4.5 距离内统一 Remove。[工厂](../../../raw/current/scripts/prefabs/ice_crack_grid_fx.lua#L18-L23)、[清除](../../../raw/current/scripts/components/oceanicemanager.lua#L102-L107)
- 修改边界：主模拟生成（spawncracks 在管理器销毁流程内），persists=false；客户端经网络同步动画，不参与玩法。[工厂](../../../raw/current/scripts/prefabs/ice_crack_grid_fx.lua#L30-L38)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `ice_crack_grid_fx` | fn：Transform＋AnimState（build `gridicecrack`、bank `gridplacer`、`math.random()<0.5` 选 `left`/`right`、OnGround 背景层 SortOrder 3）→ NOCLICK／NOBLOCK／FX／ice_crack_fx 标签 → SetPristine → 主模拟 persists=false | 单一注册；客户端分界前完成全部显示配置，主模拟只关持久化。[工厂](../../../raw/current/scripts/prefabs/ice_crack_grid_fx.lua#L11-L39)、[注册](../../../raw/current/scripts/prefabs/ice_crack_grid_fx.lua#L41) |

注册名无 NAMES 条目；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：oceanicemanager spawncracks | 瓦片销毁排队时按东南西北探测陆地，向每侧陆地铺 1 个：旋转 0/90/180/270 对应边，落点为瓦片中心±2；铺前查 1 距离内已有 `ice_crack_fx` 防重复 | 主模拟；N→(cx-2) rot0、S→(cx+2) rot180、E→(cz+2) rot90、W→(cz-2) rot270 | [spawncracks](../../../raw/current/scripts/components/oceanicemanager.lua#L383-L412) |
| 触发链：QueueDestroyForIceAtPoint | 冰面血量归零或外部脚本（crabking 等）排队销毁时进入销毁流程：排 `fx_ice_crackle` 特效→延时销毁→spawncracks | 延时 70+rand(10) 帧或调用方指定 | [[reference/components/oceanicemanager]]；[排队](../../../raw/current/scripts/components/oceanicemanager.lua#L414-L433)、[血量归零触发](../../../raw/current/scripts/components/oceanicemanager.lua#L460-L478) |
| 清理：removecrackedicefx | 瓦片实际销毁（destroy_ice_at_point）时按 `ice_crack_fx` 标签找 4.5 距离内全部裂纹逐个 Remove | 主模拟；不依赖实体自身生命周期 | [清除](../../../raw/current/scripts/components/oceanicemanager.lua#L102-L107)、[调用点](../../../raw/current/scripts/components/oceanicemanager.lua#L273) |
| 标签：ice_crack_fx | 查重（铺设前）与清理（销毁时）的统一句柄 | 全库仅本工厂添加、管理器消费 | [定义](../../../raw/current/scripts/components/oceanicemanager.lua#L37)、[工厂标签](../../../raw/current/scripts/prefabs/ice_crack_grid_fx.lua#L25-L28) |
| 资源 | anim/gridicecrack.zip（build 同名，bank 借 gridplacer） | 声明与使用已核验，包内容未解包 | [资源声明](../../../raw/current/scripts/prefabs/ice_crack_grid_fx.lua#L1-L4) |

## 典型接入

### 让自定义脚本破坏冰面时沿用裂纹提示

- 前提与选择：走 [[reference/components/oceanicemanager]] 的 `QueueDestroyForIceAtPoint`，裂纹铺设是销毁流程的一环，不要只手工铺本实体而不排队销毁。
- 生效与撤销：裂纹寿命完全由销毁流程管理——排队即铺、销毁即清；提前清理需自己按 `ice_crack_fx` 标签查找。
- 原版依据：crabking 同款调用即管理器公开接口；铺 4 侧、防重复、按边旋转都在 spawncracks 内完成。[spawncracks](../../../raw/current/scripts/components/oceanicemanager.lua#L383-L412)

- 调整与验证：裂纹朝向由陆地探测决定（无陆地一侧不铺）；落点偏移 ±2 世界单位写死在 spawncracks，改位置需换自定义实体。

## 常见陷阱与限制

- **边界：实体没有自我移除路径**：动画播完（未显式传循环参数）后停在末帧，直到管理器按标签清除；绕过管理器手工生成又不清理会残留到存档外（persists=false 不入档但会话内常驻）。[工厂](../../../raw/current/scripts/prefabs/ice_crack_grid_fx.lua#L30-L38)、[清除](../../../raw/current/scripts/components/oceanicemanager.lua#L102-L107)
- **边界：一瓦最多四条，重复铺设被 1 距离查重挡住**：同一落点再排队销毁不会叠新裂纹；查重半径 1 写死在 spawncracks。[查重](../../../raw/current/scripts/components/oceanicemanager.lua#L392-L393)
- **边界：裂纹位置不等于裂缝走向**：铺设点在瓦片边缘 ±2 处、朝陆侧，bank 是 gridplacer 的整格动画，实际显示范围由资源决定，未解包核验。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；41 行工厂全文、oceanicemanager 的 spawncracks／removecrackedicefx／QueueDestroyForIceAtPoint／伤害归零触发链已核验；动画包内容、裂纹在客户端的实际显示效果、冰面破坏全流程实测未核验（流程机制见 oceanicemanager 页）。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-water-index|水面、航海与船件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页 sources 所引文件原始字节本轮全部未变（哈希等价批量沿用，无语义修订），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-3）。
