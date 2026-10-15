# 头部装备梳理与帽子骨架检视

日期：2026-10-11。复用三个用户模组的只读装备扫描，定向审阅头部工厂与穿脱回调；通用骨架回查 Steam buildid 25643519 原版。未执行模组或 DST，W05 继续推进。

## 梳理范围与取舍

上一轮 47 个直接字面量 equippable 命中文件中，46 个哈希未变，DSTKisaki 的 kisaki_pledge_amulet.lua 在本轮检查期间变化；重新只读提取并审阅当前文件，仍为 NECK or BODY，不增加头部候选。沿用未变化文件的词法组筛选：6 组／6 文件，MCW／Sora／DSTKisaki 分别 1／5／0。其中四组显式 HEAD，两组 NECK or HEAD；这是所选语法范围的候选数，不是完整 Prefab 计数，也不证明 DSTKisaki 没有通过 helper／Hook 接入头部装备。

路径相对对应模组根，以下文件均在 `scripts/prefabs/`；私有技能、皮肤或角色框架只用于识别需求，不带入通用工厂。

| 模式 | 定向观察 | 对骨架的影响 |
|---|---|---|
| 角色／皮肤分支、附着特效、额外修饰 | MCW `mcw_hat.lua:62–152` 设置角色皮肤与前后 FX、减伤修饰，`:154–178` 撤销；`:299–308` 为 HEAD 与不破损 armor | 最小 demo 不复制私有 FX／技能；其卸装 HEAD_HAT_HELM 显示与原版不同，不照搬恢复逻辑 |
| 不替换帽子符号，仍改变头发 | Sora `sora_pickhat.lua:97–124` 注释 swap_hat 替换，但保留 Show／Hide，同时开关能力／特效／监听；`:34–66` 和 `:173–196` 管容器与耗能 | 省图必须一起处理纯显示分组；功能、容器与清理不能随显示回调整体删除 |
| 普通帽显示与多种能力共存 | Sora `sorahat.lua:160–180` 替换帽子并恢复头发，`:209–210` 加原版／私有 goggles 标签，`:237–254` 有位面、防御及装备属性 | 显示、槽位、防御、精神和护目镜能力独立组合；方法覆写／自动修复不进入最小骨架 |
| 对特定角色省穿戴图 | Sora `sora2hat.lua:85–101` 对 Sora 提前返回，其他角色替换帽子；`:157–184` 为头部工厂，另有身体工厂 | 必须选对工厂；省图是显示选项，角色分支的完整正确性未运行验证 |
| 可选护符槽回退头部 | Sora `sorabowknot.lua:142–175` 与 `sora2amulet.lua:91–123` 配对能力／修饰，分别 `:338`、`:227` 使用 NECK or HEAD | 不能只按名字／候选槽位把它们当帽子；不显示普通帽图也可能需要穿脱功能 |

DSTKisaki 当前 `kisaki_pledge_amulet.lua:93` 和 `:183` 另有 goggles 功能标签／装备检查，`:167–190` 重算装备视野、`:407–418` 配对穿脱处理，说明头部相关能力不等于显式 HEAD 工厂。指定多装备栏模组 3173431824／2.4.0 的 `assign.lua:273–299` 通用分配对非 BODY 槽提前返回，仅包装指定附加槽；该局部分支不证明 HEAD、多槽位消费者或显示完整兼容。

## 交付与读取设计

交付 [头部骨架](../../../wiki/cookbook/equipment/head.md)，只在 [装备导航](../../../wiki/cookbook/equipment/index.md) 登记。独立 dstwiki_hat 使用草帽动画／图标，三个服务端组件 inspectable／inventoryitem／equippable，明确 HEAD；精神基础贡献 3 / 60／秒由原版 sanity 读取装备槽，不创建周期任务。未继承原版草帽的防雨、隔热或耗时耐久。

原版核验：[MakeHat 命名](../../../raw/current/scripts/prefabs/hats.lua#L17)、[共同工厂／网络分界](../../../raw/current/scripts/prefabs/hats.lua#L187)、[普通帽显示与恢复](../../../raw/current/scripts/prefabs/hats.lua#L49)、[精神消费](../../../raw/current/scripts/components/sanity.lua#L510)。普通、露顶、全包头盔的外观模式分别链接原版短范围；防御／防雨／隔热／耗能链接已存在机制章节，不再增加必读页。

原版 [goggleshat 注释](../../../raw/current/scripts/prefabs/hats.lua#L2240) 明确只有装饰外观，没有 goggles 标签收益；[deserthat](../../../raw/current/scripts/prefabs/hats.lua#L2206) 的共同初始化另加该标签。页内保留选择边界，不将标签／名字当完整防沙实现；实际消费者、滤镜和专属联动仍需定向核验。

省穿戴图短节明确：本例删除纯外观回调及其注册，保留槽位、属性和地面／库存资源；已有技能／容器／耗能／任务／监听仍保持生效与撤销。统一角色显示策略留后续角色骨架，不在本页展开资源制作教程。

## 验证与限制

- 只读确认 hat_straw.zip 内有 anim.bin／build.bin／纹理，当前 inventoryimages3.xml 的 strawhat.tex 有配套纹理；旧库存 XML 也有同名图元但缺旧配套纹理。本例默认查找图集，不硬编码分片编号或复制资源。
- 四个发布 Lua 块与隔离省图版本通过 Full Moon 3.0.0／Lua 5.1 语法和原字节往返；两版显式提取 complete、无截断。检查组件集合、服务端分界、HEAD、dapperness、回调及 Prefab 工厂；所有输出锚点的原字节哈希匹配。complete 仅表示所选显式语法覆盖。
- 新页 169 行／11,554 UTF-8 字节，低于 cookbook 双上限；44 份知识／导航检查通过，共 35 篇知识页、9 索引、4 根快捷入口。未新增根快捷链接或重复登记到父导航。
- 六次实际搜索均以本页为首候选；宽泛“头部装备”省略 7 个其他候选并显式报告。五次章节读取无截断，完整 demo 输出 10,547 字节，demo＋armor 接口 14,610 字节，均在默认 16,000 字节内；保留适用／验证范围与完整代码。Windows 捕获 stdout 的 CRLF 多 1 字节，两种口径同时归档，不当作准确模型 token。
- 未覆盖游戏生成、穿脱／换帽／省图、精神变化、存档／死亡、多人显示、骑乘／假人／皮肤、自定义角色、多装备栏及扩展能力；未做独立模型评测或准确 token 计量。页面保持 source-reviewed。

证据／只读哈希／资源／语法／检索结果归 Git 忽略的 `scripts/output/head-equipment-20261011/`，改动前备份在 `scripts/output/backups/head-equipment-20261011/`。用户模组、raw、独立 fork 与主仓暂存区未修改，维护日志只追加。同步计划与设计后，下一篇为配方，再推进建筑；背包、服装、护符按后续需求建设。
