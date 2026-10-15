# 手部装备梳理与首篇骨架检视

日期：2026-10-11。按用户提供的 DSTKisaki、MCW、Sora 只读梳理手持物，并参考工作坊 3173431824 多装备栏模组 2.4.0；通用 demo 重新核验 Steam buildid 25643519 原版源码。此次未运行任何模组或游戏，不是这些模组的完整正确性审校。

## 梳理范围与限制

用 `rg --files` 枚举三个模组的 Lua，排除注释后筛直接字面量 `AddComponent("equippable")`：DSTKisaki／MCW／Sora 分别命中 6／21／20 个文件，共 47 个。所读文件哈希与上一轮一致，复用已有 Full Moon facts，不重复全量解析；21 个 complete、26 个 partial，都是当前显式提取范围，不能据此证明运行行为或全部辅助函数已解析。

按函数范围筛显式 HANDS 或未在同范围重设槽位的默认 HANDS，得到 29 个文件中的 30 个词法候选组，三模组分别 3／14／13 组。候选不是最终 Prefab 数：kisaki_tool、kisaki_complexprojectile 是配置工厂，Sora 的 sora_tqy 同文件有物品与盒子两套工厂，辅助函数／后置 Hook 仍可能改状态。按候选定向阅读工厂、注册、装备／卸装回调及代表性 helper，不按文件名或整文件组件共现直接判能力。

## 已观察到的模式

下表路径相对对应模组根，行号为本轮文件定位；只说明存在的实现路径，不将项目私有组件、扩展动作或回调视为通用推荐。

| 模式 | 代表实现 | 对通用内容的影响 |
|---|---|---|
| 共用持握与穿脱 | kisaki_tool.lua:181／200／262；mcw_lollipopstaff.lua:15／41／124；sora2sword.lua:39／45／109（均在 scripts/prefabs/） | 持握图、ARM_carry／ARM_normal 与回调属于最小骨架；同槽的工具／武器共用该部分 |
| 属性与角色专属修饰 | mcw_butterflystaff.lua:171–175；mcw_elementspear.lua:413–465；sora_wq.lua:76–112 | equippable 属性与人物修饰分开；私有暴击／元素字段不进入通用 demo，修饰需匹配撤销 |
| 动作、损耗与切换 | kisaki_tool.lua:275–284／397–433；mcw_lollipopstaff.lua:130–176；sora2sword.lua:91–129 | 工具动作不等于攻击／耐久；配置 helper、动作可见性、耗尽门禁和存档另核验，复用 tool／finiteuses／useableitem |
| 远程与投掷 | kisaki_magic_staff.lua:899–901；kisaki_complexprojectile.lua:247–258／281–295；mcw_magicbow.lua:765–770；sora_tqy.lua:292–311 | weapon 与实际弹体／投掷工厂分工；可堆叠投掷物也可能用 equippable，不能默认当普通手持武器 |
| 施法、轮盘与充能 | kisaki_magic_staff.lua:465–480／895–901；mcw_elementsword.lua:860–924；sora2plant.lua:1003–1032 | 普通施法、轮盘、AOE／自定义动作及服务端校验分别接入；sora2plant 实为手持农作工具，不能按名字建植物骨架 |
| 灯光、跟随对象、周期任务 | kisaki_tool.lua:181–215／284；kisaki_magic_staff.lua:782–803／929–940；mcw_blossomstaff.lua:320–391 | 附加对象和任务有独立创建／撤销路径；只有 OnUnequip 不能证明所有移除／死亡情形都覆盖，当前骨架不附带完整效果框架 |
| 手持容器 | mcw_queen_console.lua:210–264／311–329；mcw_oceanstaff.lua:170–267／330；sora_tqy.lua:349–365／430–437 | container 不等于背包槽；手持容器保留 HANDS 组装，布局／replica／开关复用 container，内容武器继承是专属扩展 |
| 美术与私有资源 | sora2prop.lua:39–49；mcw_magicbow.lua:514–563；soramagic.lua:151–181 | 来源 symbol 不固定为 swap_weapon；私有皮肤／API／FX 不直接复制，通用 demo 使用已取得的原版资源 |

多装备栏实际在 slots.lua:23–39 按配置注册 BELLY／NECK／BACK；assign.lua:275–299 对非 body 槽返回，只对已经在其额外槽位的物品包装回调。因此本轮 HANDS demo 可沿原版槽位核验，没有必要嵌入三类额外槽位的代码；这条源码观察不证明两个模组一起启用已通过运行。

身体分类与功能能力会交叉：其 bodylayers.lua:60–75 的服装识别包含 insulator／waterproofer／fueled，并排除 armor；assign.lua:214–222 又支持特例。MCW 战斗服和 Sora 服装本身有 armor，原版 setbonus 还只检查 HEAD／BODY。后续 Wiki 分类按制作场景选主骨架，实际槽位、功能和兼容条件单独核验，分类结果不照搬此模组的启发式规则。

## 交付与读取设计

已交付 [装备导航](../../../wiki/cookbook/equipment/index.md) 与 [手部骨架](../../../wiki/cookbook/equipment/hand.md)。最小工厂使用独立 dstwiki_handstaff，复用原版 cane 两个动画包与图标；只接 inspectable／inventoryitem／equippable，装备显示原版手杖、基础步行倍率 1.20，卸下恢复空手显示。无 weapon／tool／finiteuses／技能／特效，组件能力不会因为借用美术而继承。

地面 bank 与人物持握 symbol 分别写明；modmain 追加注册保留已有数组。完整必要代码随手部页提供，不强制先读整个 new-item；共用加载原理只链接注册章节，扩展直接链接实际机制页。其他五类页面按计划逐步编写，不建空页或悬空导航。

分类索引允许登记本目录独立页并路由到子主题，子目录页仍只有自己的叶级登记：cookbook/index 继续登记起步／道具，equipment/index 独占手部登记。原校验器“有子索引就禁止任何页登记”会强迫无关页增加目录，此轮改为允许本目录页；仍拒绝父级接管子目录页、重复登记、跨类登记和循环。回归覆盖混合导航、直接检索、错误父级登记与重复。

## 验证与未覆盖

- 原版依据：[cane 工厂与穿脱](../../../raw/current/scripts/prefabs/cane.lua#L7)、[装备组件](../../../raw/current/scripts/components/equippable.lua#L25)、[inventory 实际装配](../../../raw/current/scripts/components/inventory.lua#L1364)、[速度消费](../../../raw/current/scripts/components/locomotor.lua#L194)、[武器选择](../../../raw/current/scripts/components/combat.lua#L828)。未把原版 cane 的皮肤／漂浮／武器行为复制成最小必要功能。
- 安装资源：只读确认 cane.zip 含 anim.bin，swap_cane.zip 含 build.bin／纹理；images 数据包存在 cane.tex 图元与对应图集纹理。两包分别保存动画与材质，不要求每包都具备同样文件；骨架用默认图集查找，不硬编码编号，也不复制原版文件。
- 结构检查：四个 Lua 代码块经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返；核对独立 Prefab 登记、服务端组件分界、HANDS／回调与资源名。页面／导航／标签／大小、本地链接和实际需求检索／章节读取通过，章节保留适用条件与运行缺口。
- 当前规模：33 篇知识页／9 个索引／4 条根快捷入口，42 份知识／导航页检查通过；手部页 151 行／10,849 UTF-8 字节。索引／页面定向回归 40 项通过；9 次装备目录内需求查询及 4 次全知识查询返回目标页，5 次章节读取均无预算截断，返回 5,425～13,352 字节；其中完整最小 demo 为 9,692 字节，同时读 weapon 接口为 13,352 字节。新增主题不要求检索先读索引。
- 未覆盖：游戏生成与穿脱、实际移速、存档／死亡、多人同步、骑乘、假人／皮肤、自定义角色动画和任何模组组合。没有独立模型评测或准确 token 测量；全文与章节返回字节只用来检查读取规模，不能当普遍节省率。

本地扫描、所选回调、文件哈希、资源与校验结果在 Git 忽略的 `scripts/output/hand-equipment-20261011/`，改动前备份在 `scripts/output/backups/hand-equipment-20261011/`。三个模组、额外装备栏、raw 源码、独立 fork 及 Git 暂存区保持；日志只追加。分类与后续验收见 [PLAN](../../../PLAN.md#装备分类与交付顺序)，W05 尚未完成。
