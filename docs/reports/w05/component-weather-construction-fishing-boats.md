# 天气防护、施工、垂钓与船只组件检视

2026-10-21 完成第 92～97 组，共 12 篇组件页（第 92 组为接收修正 2 页，第 93～97 组为新建 10 页），依据 Steam buildid 25643519。按用户分工启动 5 个独立子代理分组核验，主代理统一语义复核、覆盖登记与公共文件收尾；源码只读，未运行游戏。

## 分工与交付

| 组 | 子代理范围 | 页面数量 |
|---|---|---:|
| 92：显示与同步（接收） | healthbar、healthsyncer | 2 |
| 93：天气防护 | sheltered、rainimmunity、lightningblocker | 3 |
| 94：分阶段施工 | constructionsite、constructionbuilder | 2 |
| 95：垂钓 | fishingrod、oceanfishingrod | 2 |
| 96：划船与平台 | oar、walkableplatform | 2 |
| 97：船体物理 | boatphysics | 1 |

每个代理只写所属页面与临时 Lua，按 Skill 读取必要规则与源码，执行所选页格式、源码引用与示例语法检查。主代理逐页复核关键结论与锚点，统一修改覆盖 CSV、主题索引、PLAN、README、设计与日志，写前重读并保留并发 AI 的 C01～C03 增量。

| 页面 | 行数／UTF-8 字节 | 已核验宿主与接入 |
|---|---|---|
| [healthbar](../../../wiki/reference/components/healthbar.md) | 84／9,229 | 无原版宿主；双端显示实体与 net_float，模块安装示例 |
| [healthsyncer](../../../wiki/reference/components/healthsyncer.md) | 72／10,018 | 船（boat／dragonboat）与 lavaarena 玩家；boatmeter、teammatehealthbadge |
| [sheltered](../../../wiki/reference/components/sheltered.md) | 83／11,558 | player_common、wx78_possessedbody；moisture 防水贡献 |
| [rainimmunity](../../../wiki/reference/components/rainimmunity.md) | 89／11,886 | alterguardian_phase4_lunarrift、dryingrack、raindome |
| [lightningblocker](../../../wiki/reference/components/lightningblocker.md) | 113／11,529 | oceantree_pillar、watertree_pillar；weather 落雷判定 |
| [constructionsite](../../../wiki/reference/components/constructionsite.md) | 89／14,291 | mermthrone_construction 等 22 个注册站点 |
| [constructionbuilder](../../../wiki/reference/components/constructionbuilder.md) | 82／13,251 | player_common；所有 MakePlayerCharacter 角色 |
| [fishingrod](../../../wiki/reference/components/fishingrod.md) | 106／15,506 | prefabs/fishingrod.lua；pond／oasislake 目标 |
| [oceanfishingrod](../../../wiki/reference/components/oceanfishingrod.md) | 111／16,174 | prefabs/oceanfishingrod.lua；oceanfish 与 bobber 链 |
| [oar](../../../wiki/reference/components/oar.md) | 125／10,873 | prefabs/oar.lua 五种桨；ROW／ROW_CONTROLLER／ROW_FAIL |
| [walkableplatform](../../../wiki/reference/components/walkableplatform.md) | 117／16,154 | boat 系列、dragonboat、abysspillar |
| [boatphysics](../../../wiki/reference/components/boatphysics.md) | 111／15,780 | boat 系列、dragonboat、boatrace_seastack |

合计 12 页，均在 200 行／16 KiB 上限内；oceanfishingrod、walkableplatform 余量不足 300 字节，后续增补需先精简。第 92 组按检视意见修正双端创建描述、直接移除组件的 nil 回调风险、未发现宿主与引擎裁剪的证据边界，并删除未接线的 HUD 辅助函数；“检查结果见交接”已改为页面内实际数值。

## 语义核验与重要边界

- healthbar 的 `OnInit` 先建两个显示实体再按端分派，双端都会创建；只在服务端安装时远端玩家看不到。组件没有 `OnRemoveFromEntity`，残留回调会读 nil 的 `inst.components.healthbar`，停用应 `Enable(false)` 而非动态移除。全库未发现原版宿主，也未实现距离／遮挡／淡出。
- healthsyncer 原版只装在船与 lavaarena 玩家；普通玩家用 replica。`max_health` 不联网，船在共同初始化赋予同一来源；`_status` 只写 0／1／2／3，趋势不是净值；移除组件不注销监听且会触发 nil 回调。
- sheltered 的庇护事件比状态晚约 0.2 秒（副本遮罩过渡），离开同帧；`mounted` 为真时低等级不算庇护；`Stop`／移除传空 level 会对 nil 比较；`canopytrees` 是裸计数，提供者移除须补偿。
- rainimmunity 只维护来源集合与标签，最后一个来源移除会连带删组件；它把雨水速率判 0，不清已有湿度、不算防水值、不挡浮具与浸浴；来源不存档。
- lightningblocker 自身不判定：weather 在 40 距离内先收集 blocker，按遍历顺序取首个命中，命中后把雷种改为 `thunder`、以原落点回调，不产生电击与点燃；直接生成雷实体不走该入口。
- constructionsite 的 plan 是放置后的第二笔材料，缺失时 `IsComplete` 因空循环返回 true；删除实体或组件都不返还材料，读档不回绑 builder 与 enabled；`OnConstruct` 先清 builder 再回调。
- constructionbuilder 的 `StartConstruction` 先停止当前施工再检查 `INUSE`／`BURNT`；容器打不开即失败；完成必须经 `FinishConstruction` → `construct_pst` → `OnFinishConstruction`；读档只恢复容器内容，不重开、不重绑。
- fishingrod 不实现存读与移除清理；`StopFishing` 不归还上钩鱼；`Hook` 只在存在 `finiteuses` 时排脱杆超时；`fishingcollect` 才扣耐久。oceanfishingrod 的成功捕获不推 stopped；`reason`／`lost_tackle` 决定是否销毁容器内容；客户端最大抛距只是 `dist_max` 之和，不含随机精度。
- oar 的推进依赖脚下平台与 `boatphysics`，方向在控制器分支改为“向船外侧”；失败只加湿半径 2 内实体、通知随从并返回台词键，不自行扣耐久。walkableplatform 每帧按半径重建成员，`uid` 存档并供 `puid` 相对坐标；移除实体时逐级发 `onsink`、清甲板。
- boatphysics 是服务端组件；`rudder_turn_speed` 是死参数，`max_velocity` 是倍率且被多个碰撞 prefab 当伤害分母；`stopturning` 每帧重发；无 `OnRemoveFromEntity`，运行期移除会留下碰撞／死亡悬空回调。

## 验证与限制

- 12 页 `validate_page.py --check-indexes` 通过：12 索引、221 知识页、唯一登记与父入口无 issue。
- `validate_source_links.py --strict`：566 处引用全部带行锚点，0 error／0 warning。
- 完整示例由子代理以已有 `parser_probe.exe` 做 Lua 5.1 syntax-only：healthbar 756 B、healthsyncer 335 B、boatphysics 296 B、fishingrod 385 B、oceanfishingrod 555 B、oar 1,553 B、walkableplatform 943 B、lightningblocker 753 B；均通过，未执行。
- 全部页面保持 `source-reviewed`、未运行游戏；多人同步、读档回归、实机手感、模组叠加与并发写公共文件的完整冲突检测未覆盖。

## 登记与收尾

- 覆盖 CSV：12 行由 unbuilt 改为 built（批次 92～97），页面路径与已核验源码指纹写入；合并后 213 组件已建、585 未建。
- 主题索引：character 加 sheltered／healthsyncer；world 加 healthbar、rainimmunity、lightningblocker、constructionsite、constructionbuilder、boatphysics、walkableplatform；items 加 fishingrod、oceanfishingrod、oar；各页仅登记一次。
- PLAN／README／design 同步 213 组件、585 未建、221 知识页；日志按 Skill 追加一条。
