# 帆舵、船体损伤、钓鱼目标与容器数据组件检视

2026-10-21 完成第 98～101 组，共 12 篇组件页，依据 Steam buildid 25643519。四个子代理分组核验，主代理逐页复核、覆盖登记与公共文件收尾；源码只读，未运行游戏。

## 分工与交付

| 组 | 子代理范围 | 页面数量 |
|---|---|---:|
| 98：帆舵与补丁 | mast、steeringwheel、boatpatch、boatdrag | 4 |
| 99：船体损伤 | hullhealth、boatring | 2 |
| 100：钓鱼目标链 | fishable、oceanfishable、oceanfishinghook | 3 |
| 101：容器与建造数据 | container_transform、constructionplans、constructionbuilderuidata | 3 |

| 页面 | 行数／UTF-8 字节 | 已核验宿主与接入 |
|---|---|---|
| [mast](../../../wiki/reference/components/mast.md) | 77／10,486 | mast、mast_malbatross、mast_yotd |
| [steeringwheel](../../../wiki/reference/components/steeringwheel.md) | 74／10,256 | steeringwheel、yotd_steeringwheel |
| [boatpatch](../../../wiki/reference/components/boatpatch.md) | 75／10,518 | boatpatch、boatpatch_kelp；sewing_tape、treegrowthsolution |
| [boatdrag](../../../wiki/reference/components/boatdrag.md) | 75／10,844 | anchor／yotd_anchor、winch；crabking_claw 未注册 |
| [hullhealth](../../../wiki/reference/components/hullhealth.md) | 119／13,724 | boat 系 7 个与 dragonboat_body／shadowboat |
| [boatring](../../../wiki/reference/components/boatring.md) | 99／13,389 | 同上；控件 boat_rotator、bumper boat_bumper_* |
| [fishable](../../../wiki/reference/components/fishable.md) | 93／8,423 | pond／pond_mos／pond_cave、oasislake |
| [oceanfishable](../../../wiki/reference/components/oceanfishable.md) | 95／9,214 | oceanfish_*、bobber floater、flotsam、squid／wobster |
| [oceanfishinghook](../../../wiki/reference/components/oceanfishinghook.md) | 88／8,557 | 仅 bobber 的 `*_floater` |
| [container_transform](../../../wiki/reference/components/container_transform.md) | 63／8,989 | wx78_possessedbody（产物 wx78_backupbody） |
| [constructionplans](../../../wiki/reference/components/constructionplans.md) | 65／9,687 | multiplayer_portal_moonrock_constr_plans |
| [constructionbuilderuidata](../../../wiki/reference/components/constructionbuilderuidata.md) | 68／9,539 | player_common 公共段（无独立注册名） |

12 页均在 180 行／14 KiB 以内。主代理修正 5 页示例的 `GLOBAL` 前缀、3 处相对链接改为 Wiki 链接，并把 constructionplans 的 tag 复制结论标为源码推断。

## 语义核验与重要边界

- steeringwheel 的 `StopSteering` 把 `RemoveEventCallback` 写成了 `ListenForEvent`，旧驾驶者的 `onremove` 不会清理；页面已记录，不能把 Stop 当完全清场。
- boatdrag 的四个因子只有在 `boatphysics:AddBoatDrag` 注册后才结算；`crabking_claw` 虽配置了四项，但全库无注册调用，当前不进入船速计算。
- hullhealth 修理只改 boatleak 状态、不复位 `leak_damage`：同一点再次碰撞会按旧累计立即重开；固定漏点写自己的存档，`selfdegradingtime` 不在其中。
- boatring 是船侧整船旋转与 bumper 登记，不是独立物品；旋转速度按 rotator 数量乘 `rotate_speed` 并夹到上限，不使用 `dt`。
- mast 在构造时生成 `rudder`、延迟 0 秒取当前平台绑定；`AddMast` 后由 `boatphysics` 读帆力与最大速度，`CloseSail` 是其收帆入口。
- fishable 的 `HookFish` 立即扣鱼量并挂隐藏子实体；`RemoveFish` 不还原显示／物理／存档，需钓竿的 Reel／Collect 补；oasislake 上限 15 而初始鱼量 10，刷新任务要等 OnLoad 或首次 RemoveFish。
- oceanfishable 与 oceanfishingrod 双向互链；`catch_distance` 与 `MAX_HOOK_DIST`／`MAX_CAST_DIST` 是两个不同限制。oceanfishinghook 只做 charm／interest 诱鱼，不参与咬钩交接。
- constructionplans 的 `OnRemoveFromEntity` 遍历未定义全局 `targetprefabs`，`RemoveComponent` 会在 `pairs(nil)` 报错；tag 只在 pristine 复制（源码推断，未运行）。
- container_transform 未设置 `ontransform` 时返回 `false`，RUMMAGE 用 `targ ~= nil` 判真后访问 `false.components`，会抛 Lua 错误；未设置 `cantransform` 则静默失败。

## 验证与限制

- 12 页 `validate_page.py` 通过（0 issue，尺寸见上表）；`validate_source_links.py --strict` 398 处引用、0 error／0 warning。
- 示例语法：hullhealth 492 B、boatring 409 B（GLOBAL 前缀修正后重跑，通过）；其余页示例由子代理检查通过，均未执行。
- 全部 `source-reviewed`、未运行游戏；多人同步、读档回归、实机手感与模组叠加未覆盖。

## 登记与收尾

- 覆盖 CSV：12 行改为 built（批次 98～101），合并后 225 组件／573 未建。
- 新增主题索引 `topics/world-ships`（8 条船只页），world 索引相应缩减至 159 行；items 增 4 条（3 钓鱼目标＋container_transform），world 科技与建造接入增 2 条；组件索引增第 6 个主题入口。
- PLAN／README／design 同步 225／573／233／13；日志追加一条。
