# 磁力、吸湿、防滑与船体漏点组件检视

2026-10-21 完成第 107～110 组，共 8 篇组件页，依据 Steam buildid 25643519。四个子代理分组核验，主代理复核、覆盖登记与公共文件收尾；源码只读，未运行游戏。

## 分工与交付

| 组 | 子代理范围 | 页面数量 |
|---|---|---:|
| 107：磁力与信标 | boatmagnet、boatmagnetbeacon | 2 |
| 108：吸湿物品 | moistureabsorbersource、moistureabsorberuser | 2 |
| 109：防滑砂 | nonslipgritsource、nonslipgrituser | 2 |
| 110：漏点与残骸 | boatleak、flotsamgenerator | 2 |

| 页面 | 行数／UTF-8 字节 | 已核验宿主与接入 |
|---|---|---|
| [boatmagnet](../../../wiki/reference/components/boatmagnet.md) | 107／14,104 | boat_magnet（套件 boat_magnet_kit） |
| [boatmagnetbeacon](../../../wiki/reference/components/boatmagnetbeacon.md) | 87／13,324 | boat_magnet_beacon |
| [moistureabsorbersource](../../../wiki/reference/components/moistureabsorbersource.md) | 102／10,098 | desiccant／desiccantboosted |
| [moistureabsorberuser](../../../wiki/reference/components/moistureabsorberuser.md) | 71／8,663 | 持有者动态添加（无自有 prefab） |
| [nonslipgritsource](../../../wiki/reference/components/nonslipgritsource.md) | 94／13,664 | nonslipgrit／nonslipgritboosted |
| [nonslipgrituser](../../../wiki/reference/components/nonslipgrituser.md) | 79／12,247 | 带 slipperyfeet 的玩家与 wx78_possessedbody |
| [boatleak](../../../wiki/reference/components/boatleak.md) | 105／12,228 | boat_leak（由 boat 的 spawnnewboatleak 生成） |
| [flotsamgenerator](../../../wiki/reference/components/flotsamgenerator.md) | 104／13,279 | 世界 forest |

主代理修正 moistureabsorbersource 示例的 `GLOBAL` 前缀；8 页均在 180 行／14 KiB 以内。

## 语义核验与重要边界

- boatphysics 的多磁铁合力先累加方向、再乘“当前这块磁铁”的力，第二块起不是各自方向；`PairWithBeacon` 不检查磁铁已配对，与信标侧不对称，改配要先解除。
- moistureabsorbersource 的速率回调未设置时返回 0；雨天只把给持有者的速率乘 0.07，`ApplyDrying` 仍全速给物品充能，二者不对称。user 只扫物品栏／装备／手持／BODY 容器，背包在物品栏时可能扫不到。
- nonslipgrituser 用 `Inventory:FindItem` 取第一个来源，不遍历容器；空 user 组件会让消费方认为仍防滑；强化砂按更新帧扣燃料并铺池。
- boatleak 的 `repair_state` 拼接只匹配四个内建修复分支，自制 `patch_type` 会落空；`hullhealth.leak_indicators_dynamic` 只插入不删除，销毁后可能残留无效引用。
- flotsamgenerator 的 `FLOTSAM_SPAWN_MAX` 只作为 `ToggleUpdate` 的 0／非 0 开关，生成路径不计数，实际在场数可超过 4；调试串中的“/4”不是硬上限。
- 两对 source／user 都是“物品侧注册、持有者侧集合”的结构，撤销由持有者变化或来源移除驱动，不需要手工 AddSource。

## 验证与限制

- 8 页 `validate_page.py` 通过（0 issue）；`validate_source_links.py --strict` 305 处引用、0 error／0 warning。
- 子代理示例 syntax-only 通过、未执行；boatleak／flotsamgenerator 示例 756／594 字节，boatmagnet 510 字节。
- 全部 `source-reviewed`、未运行游戏；多人同步、读档回归与实机表现未覆盖。

## 登记与收尾

- 覆盖 CSV：8 行改为 built（批次 107～110），合并后 246 组件／552 未建。
- 索引：world-ships 加 boatmagnet／boatmagnetbeacon／boatleak／flotsamgenerator；items 物品湿度与温度加两个 source；character 温度与环境加两个 user。
- PLAN／README／design 同步 246／552／254／13；日志追加一条。
