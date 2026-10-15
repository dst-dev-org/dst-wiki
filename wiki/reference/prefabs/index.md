---
title: 原版 Prefab 索引
created: 2026-10-29
updated: 2026-10-29
type: index
tags: [overview]
---

# 原版 Prefab

已知注册名可直查 `reference/prefabs/<注册名>.md`；中文名和俗称先查本表或页面 aliases，再读命中页的相关章节及验证记录。页面描述原版工厂与一层关联导航，完整造物工程仍从 [[cookbook/index|cookbook]] 选择。

## 温度物品与容器

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/heatrock|暖石（heatrock）]] | 温差分档、携带／地面换热、灯光归属与冷热循环耐久 | Thermal Stone、暖石发光、暖石耐久、携带取暖、暖石降温 / active |
| [[reference/prefabs/backpack|背包（backpack）]] | BODY 槽八格背包、穿脱开关、收纳限制、燃烧与皮肤跟随 FX | 普通背包、Backpack、背包格子、背包放冰箱、穿戴开包 / active |
| [[reference/prefabs/icebox|冰箱（icebox）]] | 九格过滤、fridge 保鲜／暖石冷却、锤击与放置预览 | Ice Box、冰箱保鲜、冰箱放暖石、暖石冷却、冰箱收纳条件 / active |

## 生物与生成行为

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/krampus|坎普斯（krampus）]] | 淘气值生成入口、偷物砸箱、Brain／SG 分工、离场及掉落 | 小偷、Krampus、坎普斯偷东西、小偷砸箱子、坎普斯掉落 / active |

## 装备与施法物品

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/cane|步行手杖（cane）]] | 装备加速、持握，七皮肤雪花／花瓣／彩纸／紫烟／闪光／影子／棋子与 FX 入口 | 手杖、加速手杖、Walking Cane、手杖皮肤特效、手杖雪花特效、手杖花瓣特效、手杖彩纸特效、手杖紫烟特效、手杖闪光粒子、手杖影子残影、手杖棋子拖尾；远古手杖、拐杖糖、城堡手杖、愚人手杖、玫瑰红手杖、腐败手杖、山羊头手杖 / active |
| [[reference/prefabs/yellowstaff|唤星者魔杖（yellowstaff）]] | 施法物品：次数／精神消耗、生成 stafflight、月台转换与皮肤对应 | 唤星、矮星、唤星法杖、Star Caller's Staff、召唤矮星 / active |
| [[reference/prefabs/opalstaff|唤月者魔杖（opalstaff）]] | 施法物品：50 次地点召唤、生成 staffcoldlight、来源与皮肤 build | 唤月、冷星、极光、唤月法杖、Moon Caller's Staff、召唤极光 / active |

## 召唤实体与附属变体

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/stafflight|矮星（stafflight）]] | 生成实体：照明、供热、烹饪、传播火势、精神光环和定时熄灭 | Dwarf Star、矮星光源、矮星持续时间、矮星供热、矮星烹饪 / active |
| [[reference/prefabs/staffcoldlight|极光（staffcoldlight）]] | 生成实体：制冷／精神光环／持续时间；同页区分月台附属 staffcoldlightfx | 冷星、Polar Light、极光持续时间、冷星降温、staffcoldlightfx、月台冷星特效 / active |

## 皮肤附属特效

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/cane_candy_fx|拐杖糖雪花特效（cane_candy_fx）]] | 拐杖糖：雪花跟随粒子、寿命／位移发射、卸装清理 | 拐杖糖雪花特效、手杖雪花特效、雪花粒子、cane_candycane / active |
| [[reference/prefabs/cane_rose_fx|玫瑰红手杖花瓣特效（cane_rose_fx）]] | 玫瑰红：红白黄花瓣、位移门槛与三通道发射取整 | 玫瑰红手杖花瓣特效、手杖花瓣特效、花瓣粒子、cane_rose / active |
| [[reference/prefabs/cane_harlequin_fx|愚人手杖彩纸特效（cane_harlequin_fx）]] | 愚人：五色彩纸、向上初速度、五通道发射与清理 | 愚人手杖彩纸特效、手杖彩纸特效、彩纸粒子、cane_harlequin / active |
| [[reference/prefabs/cane_sharp_fx|腐败手杖紫烟特效（cane_sharp_fx）]] | 腐败：紫烟、固定基准发射与 FollowEmitter | 腐败手杖紫色烟雾、手杖紫烟特效、手杖烟雾特效、cane_sharp / active |
| [[reference/prefabs/cane_victorian_fx|山羊头手杖闪光特效（cane_victorian_fx）]] | 山羊头：淡黄闪光、闪烁包络、移动增量发射 | 山羊头手杖闪光特效、手杖闪光粒子、手杖星光特效、cane_victorian / active |
| [[reference/prefabs/cane_ancient_fx|远古手杖影子拖尾（cane_ancient_fx）]] | 远古：影子动画拖尾、三编号变体、代理与本地动画分别回收 | 远古手杖影子拖尾、手杖影子残影、cane_ancient、cane_ancient_fx1、cane_ancient_fx2、cane_ancient_fx3 / active |
| [[reference/prefabs/cane_chess_fx|城堡手杖棋子拖尾（cane_chess_fx）]] | 城堡：黑白棋子拖尾、阵营选色、追踪与最短存续／退场 | 城堡手杖棋子特效、手杖棋子拖尾、黑白棋子特效、棋子拖尾消失、cane_chess；cane_chess_fx_bishop_b、cane_chess_fx_king_b、cane_chess_fx_knight_b、cane_chess_fx_pawn_b、cane_chess_fx_queen_b、cane_chess_fx_rook_b、cane_chess_fx_bishop_w、cane_chess_fx_king_w、cane_chess_fx_knight_w、cane_chess_fx_pawn_w、cane_chess_fx_queen_w、cane_chess_fx_rook_w / active |

“小偷”按用户提供映射收录，其他语境可能指不同对象。暖石、背包、冰箱已确认无额外俗称。各页为源码审阅，未运行游戏。

“矮星／冷星／极光”可能同时指施法物品与产物；查次数、施法、转换选法杖页，查温度、光照、持续时间选生成实体页。皮肤映射留主体，特效参数／回收查 FX 页；如“手杖雪花特效”同时命中主体与产物，按查询目的选章节。尚未成页的 FX 保留源码入口，已成页者回补真实 Wiki 链接。

返回 [[reference/index|机制与接口参考]]。
