# 身体装备梳理与护甲骨架检视

日期：2026-10-11。沿用上一轮只读 equippable 扫描，对用户提供的 DSTKisaki、MCW、Sora 定向审阅身体装备，并参考 3173431824 多装备栏模组 2.4.0。通用骨架重新核验 Steam buildid 25643519 原版源码；未执行任何模组或游戏。

## 梳理范围与限制

47 个直接字面量 equippable 命中文件与上一轮哈希相同，不重复全量解析。按词法组中显式 BODY 或含 BODY／BACK 的候选表达式筛得 13 组／13 文件，DSTKisaki／MCW／Sora 分别为 3／6／4；其中四组显式 BODY 且同工厂有 armor。候选用于定位，不是最终 Prefab 数，也不证明 helper／后置 Hook 的最终槽位或完整能力已审校。

下表路径相对对应模组根，均在 `scripts/prefabs/`；按工厂、穿脱及所需 helper 阅读，私有组件／动作仅作需求线索。

| 模式 | 本轮观察 | 对通用骨架的影响 |
|---|---|---|
| 有限护甲、材料修复、条件显示 | MCW `mcw_magic_armor.lua:8–31` 对 MCW 角色不覆盖身体图，其他角色显示护甲；`:32–43` 按金子修复；`:93–106` 配置 BODY／armor／trader | 最小护甲用原生耐久与穿脱；修复作为 trader 扩展，省图与技能回调分开 |
| 不破损、等级与服装美术 | MCW `mcw_skirt.lua:31–38` 动态调整吸收与位面防御，`:50–85` 对部分皮肤 AddOverrideBuild／ClearOverrideBuild；`:234–238` 初始化不破损 armor | 名称为裙子仍有护甲；不可破损、升级与整套身体 build 不作为最小前置 |
| 保留角色外观、自动修复、改写组件方法 | Sora `soraclothes.lua:153–171` 清除 swap_body 并配对火伤修饰；`:244–262` 有 armor／位面／装备属性，覆盖 GetAbsorption／SetCondition 且另建修复任务 | 显示与防御独立；不复制私有覆写，不能用这件物品推断原生护甲归零后默认保留 |
| 同文件头部与身体工厂、角色分支 | Sora `sora2hat.lua:105–124` 对 Sora 角色省身体图，其他角色普通 symbol 或私有锁图；`:224–243` 身体工厂含耐久／位面／防雨／装备／trader | 不按文件名判断类型；选到身体工厂后再决定显示与扩展，不要求头部骨架作为前置 |
| 护符也能承担护甲／容器能力 | DSTKisaki `kisaki_multivariate_amulet.lua:286–295` 开关容器并重算功能；`:335–358` 使用 NECK or BODY，不破损 armor 与 container；`:123–139` 汇总属性 | 佩戴场景、槽位和能力分开；虚拟护符聚合为专属框架，不带入普通护甲 demo |
| 不设置人物装备图但有穿脱功能 | DSTKisaki `kisaki_pledge_amulet.lua:399–405` 仅应用／撤销能力，`kisaki_constellation_talismans.lua:51–56` 回调由配置传入；MCW `mcw_selene_lament.lua:29–49` 管理光源及私有恢复，`mcw_medal_maigo.lua:7–13` 回调为空 | 不显示装备图不等于回调可以全部删除；外观制作成本与实际功能分开，配置 helper 仍需核验 |
| 背包仍有布局／任务／外观清理 | MCW `mcw_bag.lua:29–71` 按皮肤分支显示并开关容器；Sora `sorabag.lua:196–241`、`sora2bag.lua:93–114` 有背部符号、容器和任务 | 本轮只看类别边界；普通 BODY 护甲不带容器，省图不能删容器／任务收尾；背包骨架留后续 |

其余身体候选 MCW `mcw_icetearribbon.lua:113–173` 配对人物修饰、光源及监听；此处不推荐恒温、私有暴击或物理改写方案。观察到的外观省略说明这是一种实际设计选项，不据样本推断全社区比例。

多装备栏的 `assign.lua:301–365` 依次按护符、背包、服装条件分配，剩余 BODY 也包装显示回调；`bodylayers.lua:60–75` 的服装条件排除 armor。护甲背包、带 armor 的服装或私有特例不能只按名称／组件概括。普通 BODY demo 仅作核验起点，未实测指定模组的运行配置或联合启用效果。

## 交付与读取设计

交付 [身体护甲骨架](../../../wiki/cookbook/equipment/body-armor.md)，在 [装备导航](../../../wiki/cookbook/equipment/index.md) 唯一登记。独立 dstwiki_bodyarmor 复用木甲地面／穿戴图和图标，600 耐久、0.80 基础吸收，仅接 inspectable／inventoryitem／armor／equippable。受击耐久、默认耗尽删除及组件存档用原生路径，不重复监听扣次或新增一份耐久存档。

原版依据：[木甲工厂及身体图](../../../raw/current/scripts/prefabs/armor_wood.lua#L10)、[armor 初始化与归零](../../../raw/current/scripts/components/armor.lua#L39)、[装备伤害结算](../../../raw/current/scripts/components/inventory.lua#L408)、[combat 调用装备链](../../../raw/current/scripts/components/combat.lua#L616)、[库存物品删除](../../../raw/current/scripts/components/inventoryitem.lua#L478)、[强制卸装](../../../raw/current/scripts/components/inventory.lua#L1424)。省图的边界另核验 [可选穿脱回调](../../../raw/current/scripts/components/equippable.lua#L99)。

完整工厂随页，共用注册规则只链接 new-item 章节。新增“省略人物穿戴外观”短小节：本例只删纯外观回调与注册，保留地面／库存资源、槽位与 armor；不通过清除他物符号隐藏装备，也不删其他技能／容器／监听／任务的生效和撤销。头部、皮肤／自定义角色及多槽位显示分别待核验，统一策略已留角色骨架计划，当前不展开教程。

## 验证与未覆盖

- 只读资源检查确认 `anim/armor_wood.zip` 包含 anim.bin／build.bin／纹理；当前 `images/inventoryimages1.xml` 有 armorwood.tex 且对应纹理存在。数据包还保留旧 `inventoryimages.xml` 图元，本次在当前分片图集确认配套纹理；骨架通过默认查找取得图集，不硬编码编号或复制资源。
- 四个发布 Lua 块经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返；最小工厂显式语法提取 complete、无截断，不代表运行正确性或全量语义。核验 server guard、组件集合、BODY、初始化数值、回调配对和 Prefab 登记。
- 页面／导航检查通过，共 34 篇知识页／9 索引／4 根快捷入口；7 个实际查询均将本页列为第一候选，宽泛“身体装备”省略 6 个其他候选，工具显式报告。5 次章节读取无预算截断，含最小 demo＋armor 接口在默认 16,000 UTF-8 输出字节内；数字仅作规模检查，不是实际模型 token。
- 未覆盖：游戏生成／穿脱、普通伤害与耗尽、可选省图、旧存档迁移、死亡／重载、多人显示、骑乘／假人／皮肤、自定义角色与多装备栏组合；未做独立模型评测或准确 token 计量，页面保持 source-reviewed。

实际扫描／定向读取、哈希、资源、语法和检索证据在 Git 忽略的 `scripts/output/body-armor-20261011/`，改动前备份在 `scripts/output/backups/body-armor-20261011/`。用户模组、游戏源码、独立 fork 和主仓暂存区未修改；日志只追加。下一篇头部装备见 [分类计划](../../../PLAN.md#装备分类与交付顺序)，W05 继续推进。
