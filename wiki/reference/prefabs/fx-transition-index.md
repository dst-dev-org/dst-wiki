---
title: "FX · 变身、召唤、传送与退场"
created: "2026-11-07"
updated: "2026-11-07"
type: index
tags: [overview]
---

# FX · 变身、召唤、传送与退场

形态切换、年龄变化、召唤和复活、传送与出入场。

## 变身、重生与召唤

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/abigailsummonfx|阿比盖尔之花召唤、收回与升级特效]] | 五注册：骑乘选名、动作清理与等级／惊吓复用 | abigailsummonfx_mount、abigailunsummonfx、abigailunsummonfx_mount、abigaillevelupfx / active |
| [[reference/prefabs/wendy_gravestone_rebirth_fx|温蒂墓碑重生双层演出]] | 2 注册：同一重生演出的阿比盖尔层与温蒂幽灵层（仅温蒂触发、无 SetPristine 全端对称） | `abigail_gravestone_rebirth_fx`、`wendy_gravestone_rebirth_fx` / active |
| [[reference/prefabs/ghost_transform_overlay_fx|幽灵复活转生光效]] | 复活叠层动画；闪光时间线与队列退场 | — / active |
| [[reference/prefabs/attune_out_fx|复活调谐得失特效（attune_out_fx）]] | 3 注册：新源为复活调谐类型且 `_isrezattuned` 为 false 时父挂 out（不查此前是否实际持有），当前 tag 失去来源时按幽灵态选 in／ghost_in 父挂；同一 tag 换绑仅在进入前标志为 true 时静默，等价但不同 tag 互换会清标志、使下一次同 tag 换绑重新满足 out 门禁；加载重连与远程复活静默不出；墓碑挖除／恶花／装饰完成与温蒂墓碑合成四处落点复用 out | attune_in_fx、attune_ghost_in_fx、调谐特效、调谐得失特效、调谐建立特效、调谐解除特效、复活绑定特效 / active |
| [[reference/prefabs/monkey_cursed_pre_fx|猴诅咒与变形特效]] | 6 注册：猴诅咒、变形前后与玩家变化 | monkey_morphin_power_players_fx、monkey_de_morphin_fx、monkey_cursed_pst_fx、monkey_deform_pre_fx、monkey_deform_pst_fx / active |
| [[reference/prefabs/beefalo_transform_fx|皮弗娄牛驯化变身特效（beefalo_transform_fx）]] | 完成驯化与已驯化后野性化共用变身状态；骑乘退出补枚 | 皮弗娄牛变身特效、驯化变身特效、牛变身特效 / active |
| [[reference/prefabs/werebeaver_transform_fx|变身切换烟（werebeaver_transform_fx）]] | 形态切换烟 8 名：伍迪三形态变身、露西斧收编还原、切斯特／哈奇切换；lucy_transform_fx 被全玩家接住抛射物复用 | weremoose_transform_fx、weremoose_transform2_fx、weremoose_revert_fx、weregoose_transform_fx、lucy_transform_fx、lucy_ground_transform_fx、chester_transform_fx、变身烟、接住烟、变形特效 / active |
| [[reference/prefabs/weregoose_splash|鹅形态水花与羽毛（weregoose_splash）]] | 鹅形态水上移动 10 名：三档水花与羽毛水陆分流、非扑翼状态周期涟漪、起飞羽毛落点；海猴划船复用本名水花 | weregoose_splash_med1、weregoose_splash_med2、weregoose_splash_less1、weregoose_splash_less2、weregoose_ripple1、weregoose_ripple2、weregoose_feathers1、weregoose_feathers2、weregoose_feathers3、水上羽毛、涟漪特效 / active |
| [[reference/prefabs/oldager_become_younger_front_fx|年龄转换特效（oldager_become_younger_front_fx）]] | 旺达年龄档切换 6 名（三 anim×站立／骑乘）：healthdelta 重选档、年轻段前后两枚、骑乘后缀选资源、静默变身无表现；父挂旺达持续跟随 | oldager_become_younger_back_fx、oldager_become_older_fx、oldager_become_younger_front_fx_mount、oldager_become_younger_back_fx_mount、oldager_become_older_fx_mount、变老特效、变年轻特效 / active |
| [[reference/prefabs/small_puff|变身与出生烟（small_puff）]] | 单注册通用烟团；变身 helper DoChangePrefab 标准开场烟，另有月变／移除／出生等约 24 文件直接落点 | 变身烟、出生烟、小烟团 / active |

## 移动、传送与退场

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/pocketwatch_warp_marker|怀表与传送门附属表现 FX 族]] | 5 注册：warp／recall 标记与传送门 overlay／underlay／exit FX | `pocketwatch_recall_marker`、`pocketwatch_portal_exit_fx` 等 / active |
| [[reference/prefabs/shadow_teleport_in|暗影传送、孢子咳出与宝盒玩家特效]] | 4 注册：暗影生物海陆交换、蘑菇地精咳孢子与宝盒授予特效 | shadow_teleport_out、spore_moon_coughout、archive_lockbox_player_fx / active |
| [[reference/prefabs/wanda_attack_pocketwatch_old_fx|旺达怀表攻击与治疗特效（wanda_attack_pocketwatch_old_fx）]] | 7 注册：年龄与武器选攻击特效、骑乘治疗及复活倒流 | wanda_attack_pocketwatch_normal_fx、wanda_attack_shadowweapon_old_fx、wanda_attack_shadowweapon_normal_fx、pocketwatch_heal_fx、pocketwatch_heal_fx_mount、pocketwatch_ground_fx、旺达攻击特效、警钟攻击特效、怀表治疗特效、复活倒流地面特效 / active |
| [[reference/prefabs/fallingswish_clouds|坠落云与线条]] | 3 注册：虚空坠落、深渊落下与沉没选名 | fallingswish_clouds_fast、fallingswish_lines / active |
| [[reference/prefabs/superjump_fx|超级跳落点特效]] | 超级跳落点表现；目标定位、碎屑与音效 | `superjump_debris`、`player_superjump`、`superjump` / active |
| [[reference/prefabs/maxwell_smoke|麦斯威尔烟（maxwell_smoke）]] | 麦斯威尔 talker 三分支：台词收尾两模式立即、普通出现 Sleep 后、取消延迟任务；消散声在 talker 分支 | 消散烟、maxwell 特效 / active |
| [[reference/prefabs/charlie_snap|暗影拍指（charlie_snap）]] | 两注册呈现链开场拍指；音效由数据 fn 延迟 21 帧自挂播放，升级或洞穴走实心变体 | charlie_snap_solid、暗影拍指、查理拍指 / active |

返回 [[reference/prefabs/fx-index|FX 主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
