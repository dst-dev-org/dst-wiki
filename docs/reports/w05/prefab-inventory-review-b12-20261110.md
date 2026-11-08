# B12 动物掉落与巢材下外部源码检视

- 初检日期：2026-11-10。Steam 安装清单与 `raw/current/BUILD.txt` 均为 **26474873**。源码根为 `raw/current/scripts`，按当前工作树直接检视。
- 范围：[台账](prefab-inventory-ledger-20261108.md#L164)所列 6 个主页面、10 个物品注册，以及[动物掉落索引 B12 六行](../../../wiki/reference/prefabs/animal-drops-index.md#L31-L36)和直接相关的既有页断言。
- 初检结论：**待修订，暂不收口。P1 × 1、P2 × 7、P3 × 2。** 下列编号供原写作者回填，原外部检视方在本报告逐项复验。源码静态推导，未运行游戏。
- 当前结论：**R01–R10 全部通过，可以收口。** 见[第二轮外部定向复验](#第二轮外部定向复验)。

## P1

### R01 外壳碎片修补示例在无声音发射器的结构上会扣料后报错

[示例前提与回调](../../../wiki/reference/prefabs/shell-materials.md#L128-L150)只要求服务端有 `health`、无现成的 `repairable`，回调却直接调用 `target.SoundEmitter:PlaySound`。没有 `SoundEmitter` 的受损自定义结构也会按示例接入。材料匹配后，修补组件先回血，再从堆叠扣一件，最后才调用 `onrepaired`；此时访问空字段会抛错，回血和扣料已发生。验收时以「有 health、无 SoundEmitter、血量未满」为反例，确认照抄示例能完成一次修补与一次扣料。可选音效应判空或省去。证据见 [回血分支](../../../raw/current/scripts/components/repairable.lua#L106-L115)和[执行顺序](../../../raw/current/scripts/components/repairable.lua#L148-L183)。

## P2

### R02 统一普通燃烧与受控燃烧的掉落口径

[熊皮获得与改写](../../../wiki/reference/prefabs/boss-materials.md#L67-L96)、[外壳获得与边界](../../../wiki/reference/prefabs/shell-materials.md#L77-L90)、[钢丝绵获得](../../../wiki/reference/prefabs/steelwool.md#L68-L75)仍把「燃烧中死亡」写成必变灰，[伏特羊宿主页](../../../wiki/reference/prefabs/lightninggoat.md#L55-L79)和[羊奶获得表](../../../wiki/reference/prefabs/goatmilk.md#L47-L50)也把羊奶或羊角写成必变灰。羊角页已给出受控保留，与宿主页冲突。

`DropLoot` 按**实际调用时**的 `burnt` 或燃烧和燃料条件进入改写。覆盖表、熟态后缀、熟态前缀先判，受控燃烧分支会保留尚未换名的条目，最后才到非豁免宿主的灰烬分支。受控时钢丝绵可保留原名，落地会尝试闷烧；外壳、羊奶和羊角的原物没有可燃组件。伏特羊须仍处在充能表才有羊奶，羊角须先掷中。核对上述各页的摘要、来源表、陷阱与[索引 L31、L33–35](../../../wiki/reference/prefabs/animal-drops-index.md#L31-L35)。钢丝绵原物自身烧尽另走 `DefaultBurntFn`，索引和示例也应带上海面不产灰、灰烬堆叠上限的条件。证据见[改写顺序](../../../raw/current/scripts/components/lootdropper.lua#L414-L445)、[落地闷烧](../../../raw/current/scripts/components/lootdropper.lua#L382-L407)、[钢丝绵构造](../../../raw/current/scripts/prefabs/steelwool.lua#L25-L44)、[羊奶构造](../../../raw/current/scripts/prefabs/goatmilk.lua#L11-L60)、[原物烧尽](../../../raw/current/scripts/standardcomponents.lua#L18-L44)。

### R03 死亡掷表、成尸资格和熄灭结算需要分开

[熊皮通用结算](../../../wiki/reference/prefabs/boss-materials.md#L67-L73)和[羊角结算](../../../wiki/reference/prefabs/lightninggoathorn.md#L58-L64)把 `GenerateLoot` 放在死亡分支前。实际仅化尸分支显式生成并转存。直接掉落分支把位置交给 `DropLoot`，由后者生成。尸体结算把处理过的列表显式传给 `DropLoot`，羊角在尸体阶段不会再掷一次。

[Boss 器官的尸体资格](../../../wiki/reference/prefabs/boss-parts.md#L79-L81)和[羊角的尸体资格](../../../wiki/reference/prefabs/lightninggoathorn.md#L62-L62)又把「未燃烧」放到 `forcecorpse` 之前。源码先要求状态图有 `corpse` 且没有 `sg.mem.nocorpse`，再判 `forcecorpse`，之后才用燃烧阻止其余分支。羊角页还漏了常规熄灭结算，熄灭先清燃烧和受控标记，再回调。尸体正在变异或有跳过／保护标记时不会走普通熄灭掉落。验收时分开写直接掉落、转存、燃尽、熄灭和啃空，别让通用强制成尸分支被读成这些原版宿主默认会用。证据见[死亡入口](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[生成入口](../../../raw/current/scripts/components/lootdropper.lua#L415-L421)、[状态图前置](../../../raw/current/scripts/componentutil.lua#L1472-L1475)、[资格顺序](../../../raw/current/scripts/componentutil.lua#L1546-L1561)、[尸体回调](../../../raw/current/scripts/prefabs/corpses.lua#L38-L52)、[熄灭顺序](../../../raw/current/scripts/components/burnable.lua#L465-L490)、[尸体提交掉落](../../../raw/current/scripts/prefabs/corpses.lua#L489-L495)。

### R04 熊皮换毛丛有列表门禁，啃空会用旧肉量级别结算

[熊皮导航、结算、陷阱](../../../wiki/reference/prefabs/boss-materials.md#L61-L95)和[索引行](../../../wiki/reference/prefabs/animal-drops-index.md#L31-L31)只提到 `meat_level >= 2`。换算还要求列表多于一项且含 `bearger_fur`，单项列表走保留概率。仍有肉时，2 级的 11–20、3 级的 1–10 可作为公式区间。啃空时先写剩余肉量为 0，再在写入 4 级前执行侵蚀结算。旧级别已是 2／3 且多项列表仍有熊皮时，得到 1 撮毛丛。旧级别仍是 1 就一口啃空，则跳过换算。

另一个可区分的反例是尸体还剩一半肉且已有 2 级，换算出 15 撮，若当时满足普通燃烧改写门禁，再逐项变灰。[熊皮燃尽表](../../../wiki/reference/prefabs/boss-materials.md#L93-L96)应把「熊皮原条目变 1 灰」限定在未换成毛丛时。[毛丛关联页](../../../wiki/reference/prefabs/furtuft.md#L51-L51)已有多项列表门禁，可据此对齐。证据见[侵蚀先于写级别](../../../raw/current/scripts/prefabs/corpses.lua#L158-L184)、[写肉量与读比例](../../../raw/current/scripts/prefabs/corpses.lua#L192-L214)、[列表分支与缩放](../../../raw/current/scripts/prefabs/corpses.lua#L465-L495)、[熊皮覆写](../../../raw/current/scripts/prefabs/corpses_defs.lua#L1467-L1472)。

### R05 龙蝇补鳞阈值计入启用期间的正向血量变化

[熊皮与鳞片摘要](../../../wiki/reference/prefabs/boss-materials.md#L6-L6)、[鳞片来源及解释](../../../wiki/reference/prefabs/boss-materials.md#L82-L87)和[索引](../../../wiki/reference/prefabs/animal-drops-index.md#L31-L31)写「累计受伤 2500」。跟踪器在启用时累加每个 `healthdelta` 的 `math.abs(data.amount)`，正值也计入。保留「启用窗口内、同一实例只跨一次、重建后重新累计」三层条件，并与[组件页](../../../wiki/reference/components/damagetracker.md#L59-L59)一致。证据见[事件与累加](../../../raw/current/scripts/components/damagetracker.lua#L4-L35)、[龙蝇启停](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L279-L327)、[补鳞回调](../../../raw/current/scripts/prefabs/dragonfly.lua#L381-L384)。

### R06 蛞蝓龟窝的 1 件碎片取决于延迟回调能否执行

[外壳来源表](../../../wiki/reference/prefabs/shell-materials.md#L80-L83)、[燃烧段](../../../wiki/reference/prefabs/shell-materials.md#L90-L90)、[陷阱](../../../wiki/reference/prefabs/shell-materials.md#L159-L159)和[索引](../../../wiki/reference/prefabs/animal-drops-index.md#L33-L33)要区分普通击杀与烧尽爆炸。普通击杀由 `OnKilled` 播放破坏动画并安排 0.66 秒后的 `OnDoKilled`，掉落表在回调执行时才读取。窝烧尽走 `explosive`，发出死亡事件后立即移除自身，移除会取消待执行任务。即使结构标签能保留表内原物，也无法让已取消的掉落回调执行。证据见[表](../../../raw/current/scripts/prefabs/slurtlehole.lua#L18-L24)、[延迟](../../../raw/current/scripts/prefabs/slurtlehole.lua#L44-L55)、[爆炸组件接线](../../../raw/current/scripts/prefabs/slurtlehole.lua#L149-L162)、[烧尽入口](../../../raw/current/scripts/components/burnable.lua#L291-L305)、[移除顺序](../../../raw/current/scripts/components/explosive.lua#L151-L157)、[取消任务](../../../raw/current/scripts/entityscript.lua#L1730-L1747)。

### R07 贝壳保险杠的站位门禁和实际回血量需要限定

[外壳导航](../../../wiki/reference/prefabs/shell-materials.md#L57-L62)把保险杠门禁写成水面同格站着可见玩家就拒绝。[示例说明](../../../wiki/reference/prefabs/shell-materials.md#L128-L150)又写固定回 20，容易把示例与原版的附加检查混在一起。源码先直接放行陆地，且只在 `IsVisualGroundAtPoint` 分支执行半径 1 的玩家检索。玩家需可见、无 `placer`、无实体父节点，并且 X/Z 分别取整后相等。示例没装这条保险杠门禁。20 是基础回复请求，最终生命值按上限截断，例如 95/100 会回 5 并正常扣 1 碎片。同步示例、导航、索引的数值含义。证据见[站位检查](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L59-L79)、[原版接线](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L264-L267)、[生命上限](../../../raw/current/scripts/components/health.lua#L565-L574)。

### R08 钢丝绵接入示例会覆盖已有的火传播配置

[钢丝绵示例](../../../wiki/reference/prefabs/steelwool.md#L97-L119)称追加接线可保留别人的设置，但只检查 `burnable`。已有 `propagator`、无 `burnable` 时，`MakeSmallPropagator` 仍会执行。重复挂载会取回旧组件，helper 随即写入引燃回调、闪燃点、传播范围、热输出和伤害开关。验收时分别覆盖「已有 propagator、无 burnable」和「已有 burnable、无 propagator」两种配置，独立守卫或明确示例仅适用于全新接线。证据见[组件重复挂载](../../../raw/current/scripts/entityscript.lua#L632-L646)、[helper 写入字段](../../../raw/current/scripts/standardcomponents.lua#L200-L212)。

## P3

### R09 蘑菇皮的血线须在摘要与索引可直接识别

[洞穴掉落摘要](../../../wiki/reference/prefabs/cave-drops.md#L6-L6)、[陷阱](../../../wiki/reference/prefabs/cave-drops.md#L149-L150)与[索引](../../../wiki/reference/prefabs/animal-drops-index.md#L36-L36)应直接标「血量阈值 70%／40%，悲惨另有 20%」。正文阈值表已列清跨档与秒杀门禁，短入口的「各 1」也应能读出阶段上限。普通形态 71%→39% 单击只抛 1 张，致死一击不抛阶段皮。证据见[阈值](../../../raw/current/scripts/prefabs/toadstool.lua#L676-L678)、[阶段回调](../../../raw/current/scripts/prefabs/toadstool.lua#L695-L737)、[跨阈值执行顺序](../../../raw/current/scripts/components/healthtrigger.lua#L25-L38)。

### R10 修正龙蝇可燃性否定断言的证据范围

[鳞片燃烧表](../../../wiki/reference/prefabs/boss-materials.md#L93-L96)把只到掉落表的 L1–75 标作「龙蝇全文」。对本版 `prefabs/dragonfly.lua` 全文件执行 `rg -n 'burnable|Make.*Burnable' raw/current/scripts/prefabs/dragonfly.lua` 无命中。尸体侧由定义与通用构造共同判断，建议引用实际范围，并把结论限定于原版接线。证据见[龙蝇构造文件](../../../raw/current/scripts/prefabs/dragonfly.lua)、[龙蝇尸体定义](../../../raw/current/scripts/prefabs/corpses_defs.lua#L1081-L1090)、[尸体可燃组件门禁](../../../raw/current/scripts/prefabs/corpses.lua#L750-L756)。

## 源码事实与页面命中

| 源码链 | 页面命中与缺口 |
|---|---|
| 熊獾与龙蝇常规掉落各有一条 1.00，克劳斯袋按各自抽取组结算。龙蝇另有跟踪器回调。 | [熊皮与鳞片](../../../wiki/reference/prefabs/boss-materials.md#L75-L96)的常规数量与克劳斯袋口径吻合。死亡、燃烧、啃尸、补鳞见 R02–R05。 |
| 独眼巨鹿与守护者各有一条 1.00，变异巨鹿无眼球；眼球还能走独眼巨鹿以外的克劳斯袋和拆解回收。 | [Boss 器官](../../../wiki/reference/prefabs/boss-parts.md#L69-L81)的来源主体吻合。抽核眼球伞／炮塔两条消费链，成尸门禁见 R03。 |
| 饼干切割机壳两次独立掷骰；外壳碎片的机会表兜底、死亡回调和修补组件分别结算。 | [海陆外壳](../../../wiki/reference/prefabs/shell-materials.md#L55-L90)的基础概率与首轮兜底条件吻合。修补、燃烧、窝爆炸见 R01、R02、R06、R07。 |
| 钢丝绵的 45 秒燃料值与自身 15 秒燃烧时间由不同组件读取，钢羊表为两条保底加一次 0.50 基础概率。 | [钢丝绵](../../../wiki/reference/prefabs/steelwool.md#L50-L75)命中。抽核燃料结算、枕头材质接入，受控燃烧和示例见 R02、R08。 |
| 伏特羊普通与带电表各有 0.25 羊角，正向幸运提高；暗夜灯、龙卷风法杖、奔雷矛和便携锅消费。 | [伏特羊角](../../../wiki/reference/prefabs/lightninggoathorn.md#L52-L80)及直接关联的奔雷矛入口已对照；羊角、羊奶、宿主页的燃烧口径见 R02、R03。 |
| 蟾蜍阶段抛皮直接生成；击杀表独立。啜食者皮为 0.50 基础概率，食性为 HORRIBLE。 | [洞穴掉落](../../../wiki/reference/prefabs/cave-drops.md#L64-L112)的阶段回调、击杀表、食性命中。抽核蓝图和饥饿腰带两条消费链，入口血线见 R09。 |

## 机械校验与范围

- 同版注册清单对账：`bearger_fur`、`dragon_scales`、`deerclops_eyeball`、`minotaurhorn`、`cookiecuttershell`、`slurtle_shellpieces`、`steelwool`、`lightninggoathorn`、`shroom_skin`、`slurper_pelt` 各有一个主页面入口。直接相关的宿主和消费品是关联入口。
- 严格源码链接检查覆盖 6 主页面、叶索引及 `lightninggoat`、`event-weapons`、`living-tools`、`furtuft`，共 11 文件。947 处已检查引用（944 条行引用、3 条文件引用）0 错、0 警，另有 2 处工具标作不支持并跳过。链接校验单独记录。
- 已核验范围为上述主注册、六页核心获取和构造断言、对应索引行、报告点名的跨页断言与 Lua 反例。消费端按上表抽查。宿主 AI、海上宝箱的概率分布、完整配方科技树及实际运行未覆盖。

## 修订回执与定向复验

同一事实若在摘要、导航、获取表、陷阱、验证记录、关联页或索引有复述，请在回执注明已同步的位置。外部复验按原编号记录证据。

| 编号 | 原写作者修订位置与自验 | 外部定向复验 |
|---|---|---|
| R01 | shell-materials L130／L133-L150／L152：示例给 `target.SoundEmitter` 判空；前提写明回调在回血与扣料之后执行；补「20 是请求值、按上限截断、示例未装站位门禁」与「有 health、无 SoundEmitter、血量未满」反例。自验：luaparser 通过，空 `SoundEmitter` 不取字段，回血与扣料各一次。 | 通过。无音效组件的目标可以修补并扣一件。 |
| R02 | boss-materials L6／L22／L60／L73／L96-L98／L128／L138／L141：摘要、副作用、导航、改写段、燃尽表、陷阱与验证记录统一普通／受控燃烧口径——熊皮未换算受控保留原名不冒烟、已换算 furtuft 受控保留并冒烟、鳞片不进入；shell L80／L81／L84／L90 与 steel L72／L75／L121／L125 同步（普通→ash，受控保留并闷烧；钢丝绵补海上不产灰、灰烬上限 40）。 | 通过（第二轮）。普通／受控改写、碎片闷烧门禁和索引已对齐。 |
| R03 | boss-materials L70-L74：化尸显式 `GenerateLoot` 转存／直接掉落由 `DropLoot` 生成；成尸资格 `EntityHasCorpse`（含 `sg.mem.nocorpse`）→`forcecorpse`→燃烧；尸体列表显式传入不重掷；补燃尽／熄灭／啃空。boss-parts L80-L83 同口径。lightninggoathorn L58-L76：五情形表（直接／转存／燃尽／熄灭／侵蚀）＋资格顺序＋「尸体阶段不再掷」。 | 通过（第二轮）。单次死亡只掷一次，熄灭例外已入表。 |
| R04 | boss-materials L61／L74／L90／L96-L97／L133：补 `#corpse_loot>1` 且含 `bearger_fur` 的列表门禁、单项列表走保留判定、啃空按旧 `meat_level`（旧 2／3→1 撮、旧 1 跳过）、燃尽表区分未换算／已换算并给 2 级剩半肉 15 撮例子。 | 通过（第二轮）。单项表可清空，多项且含熊皮才换毛丛。 |
| R05 | boss-materials L6／L84／L87／L129：改为「累计血量变化 2500（正负都计）」，说明每个 `healthdelta` 按 `math.abs` 累加、治疗也计；保留启用窗口、同一实例只跨一次、重建后重新累计三层条件。 | 通过。启用期间正负变化都计入，复述已对齐。 |
| R06 | shell-materials L82／L90／L161：蛞蝓龟窝拆成「普通击杀 1 个／烧尽 0 个」；烧尽走 `explosive` 发 `death` 后立即 `Remove`，取消 0.66 秒 `OnDoKilled` 回调。 | 通过（第二轮）。0.66 秒从死亡回调计，烧尽仍取消掉落。 |
| R07 | shell-materials L59／L62／L152：导航补全站位门禁（陆地放行；水面分支半径 1 内可见、无 `placer`、无父节点、X／Z 取整相等才拒绝）与「回血按上限截断」；示例说明补 95/100 只回 5、20 为请求值、示例未装站位门禁。 | 通过（第二轮）。20 点请求值、上限和保险杠站位门禁已对齐。 |
| R08 | steelwool L99／L102-L119／L121：示例拆成 `burnable`／`propagator` 两个独立守卫；前提写明 `AddComponent` 会取回旧表并覆盖闪燃点、传播范围、热输出与伤害开关；说明补「只可燃不传播时不要补 `propagator`」。 | 通过。两组件独立守卫，已有配置保留。 |
| R09 | cave-drops L6／L149／L150：摘要与陷阱写明血量阈值 70%／40%（悲惨版另 20%）、每次跨档各 1 张（最多 2／3 张）、71%→39% 一击只抛 1 张、致死一击不产阶段皮。 | 通过（第二轮）。阶段产出条件与跨档、秒杀反例一致。 |
| R10 | boss-materials L98：证据范围改为 `dragonfly.lua#L1-L624` 全文，并补 `corpses_defs.lua#L1081-L1090` 与 `corpses.lua#L750-L756` 的 `makeburnablefn` 门禁；结论限定本版原版接线。 | 通过。已核对完整构造及尸体挂载条件。 |

修订后机械校验（主代理，2026-11-10）：6 主页面＋叶索引＋`lightninggoat`／`goatmilk`／`event-weapons`／`living-tools`／`furtuft` 共 12 文件 `validate_page --check-indexes` 逐页 0 issue（导航另报 9 个非本批 B13 页未登记）；`validate_source_links --strict` 1,021 处引用（1,018 行引用＋3 文件引用）0 错 0 警，2 处工具标不支持并跳过；6 主页面基线已重录（2 页 refreshed、4 页 already_recorded）。未运行游戏。

## 第一轮外部定向复验

2026-11-10，Steam build **26474873**。按当前工作树逐项复验 R01–R10，并回扫修订涉及的摘要、正文、陷阱、索引和直接关联页。**4 项通过，6 项部分修复，暂不收口。** 原 P1 R01 已通过。以下沿用原编号，未运行游戏。

### 仍待修订

- **R02（P2）**：[外壳来源表和通用段](../../../wiki/reference/prefabs/shell-materials.md#L80-L90)把碎片写成受控燃烧「保留原名、落地闷烧」，并仍对蛞蝓龟／蜗牛龟／发光蟹给出无条件「先灭火再杀」。碎片构造没有 `burnable`，受控分支在掉落时保留原名，只有自身可燃的掉落物才会尝试闷烧。钢丝绵来源表的「需先灭火」也应限于普通火，正文下一段已写明受控可保留。[索引 L33](../../../wiki/reference/prefabs/animal-drops-index.md#L33-L33)仍是「可燃宿主转灰」，须写明普通火与受控的区别。熊皮、鳞片共用的 [L31](../../../wiki/reference/prefabs/animal-drops-index.md#L31-L31)把转灰限定到熊皮；伏特羊、羊奶的普通／受控保留对照已通过。证据见[碎片构造](../../../raw/current/scripts/prefabs/slurtle_shellpieces.lua#L27-L43)、[闷烧门禁](../../../raw/current/scripts/components/lootdropper.lua#L398-L407)、[改写顺序](../../../raw/current/scripts/components/lootdropper.lua#L415-L445)及[钢丝绵两处复述](../../../wiki/reference/prefabs/steelwool.md#L70-L75)。
- **R03（P2）**：[伏特羊角 L58](../../../wiki/reference/prefabs/lightninggoathorn.md#L58-L58)新增「`GenerateLoot` 只被调用两次」，与紧接的五情形表和 L76 冲突。直接掉落与转存是互斥路径，单次死亡只生成一次。前者由 `DropLoot(pt)` 生成，后者由 `DropDeathLoot` 显式生成并转存，尸体提交显式列表。五情形表的[熄灭行](../../../wiki/reference/prefabs/lightninggoathorn.md#L65-L65)可直接带上未变异、无跳过／保护标记的条件，避免只看该表就读成任何熄灭必结算。其余尸体资格与熄灭顺序已对齐。证据见[死亡分支](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[生成入口](../../../raw/current/scripts/components/lootdropper.lua#L415-L421)、[熄灭门禁](../../../raw/current/scripts/prefabs/corpses.lua#L45-L52)、[尸体提交](../../../raw/current/scripts/prefabs/corpses.lua#L489-L495)。
- **R04（P2）**：[熊皮 L133](../../../wiki/reference/prefabs/boss-materials.md#L133-L133)把「单项列表」并入「仍是 1 张熊皮」，同一句括号又说单项走保留概率。拆开数量和变换：旧级别低于 2 会跳过折算；已达 2 且仅一项时可能清空列表，存活条目才保持原名；已达 2 且多项含熊皮时才换毛丛。正文 L74 的分支与旧级别、零肉量结算已通过。证据见[单项清空与多项换算](../../../raw/current/scripts/prefabs/corpses.lua#L465-L485)、[先侵蚀后写级别](../../../raw/current/scripts/prefabs/corpses.lua#L158-L184)。
- **R06（P2）**：[外壳 L82](../../../wiki/reference/prefabs/shell-materials.md#L82-L82)仍写「在死亡动画后 0.66 秒」，容易读成动画播完再计时。普通击杀在同一个 `OnKilled` 里启动破坏动画并安排 0.66 秒后的掉落，起点是死亡回调；与[同页陷阱](../../../wiki/reference/prefabs/shell-materials.md#L161-L161)及索引对齐即可。烧尽爆炸取消掉落已核对。证据见[死亡回调](../../../raw/current/scripts/prefabs/slurtlehole.lua#L44-L55)、[爆炸移除](../../../raw/current/scripts/components/explosive.lua#L151-L157)、[取消任务](../../../raw/current/scripts/entityscript.lua#L1730-L1746)。
- **R07（P2）**：[外壳 L22](../../../wiki/reference/prefabs/shell-materials.md#L22-L22)仍断言「回 20 点血」，与示例、导航和索引的 95/100 回 5 冲突，应同步为「请求回复 20，受生命上限截断」。[站位导航](../../../wiki/reference/prefabs/shell-materials.md#L62-L62)应直接写 `IsAboveGroundAtPoint` 未放行且 `IsVisualGroundAtPoint` 为真才查玩家，不宜用「水面（可视地面）」代称这一双重门禁。其余半径、标签和坐标条件已补。证据见[站位检查](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L59-L79)、[回复请求](../../../raw/current/scripts/components/repairable.lua#L106-L115)、[生命值上限](../../../raw/current/scripts/components/health.lua#L565-L574)。
- **R09（P3）**：[洞穴掉落 L149](../../../wiki/reference/prefabs/cave-drops.md#L149-L150)新增「每次跨档各 1 张」，随即用 71%→39% 只出 1 张作反例。把 L149 写成「各阶段满足回调条件时各 1 张，最多 2／3 张」。摘要、索引的血线单位和 L150 的反例已通过。证据见[阶段附加门禁](../../../raw/current/scripts/prefabs/toadstool.lua#L695-L737)。

### 已通过

- **R01**：[示例](../../../wiki/reference/prefabs/shell-materials.md#L128-L152)的可选音效已判空。`slurtle_shellpieces` 的 `repairer` 只设置材料和回血请求值，修补过程中不会经由物品额外播放船只修补声。血量未满且材料匹配时，组件先应用回复并扣一件，示例回调在无 `SoundEmitter` 时正常跳过。证据见[物品接线](../../../raw/current/scripts/prefabs/slurtle_shellpieces.lua#L34-L36)与[修补顺序](../../../raw/current/scripts/components/repairable.lua#L148-L183)。
- **R05**：[熊皮与鳞片 L84–87](../../../wiki/reference/prefabs/boss-materials.md#L84-L87)与[索引](../../../wiki/reference/prefabs/animal-drops-index.md#L31-L31)已改为正负都计。启用窗口内累加 `math.abs(data.amount)`，同实例仅跨一次，重建后从零开始。证据见[跟踪器](../../../raw/current/scripts/components/damagetracker.lua#L4-L35)及[启停](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L279-L327)。
- **R08**：[钢丝绵示例](../../../wiki/reference/prefabs/steelwool.md#L99-L121)分别检查 `burnable`、`propagator`。已有传播组件、缺可燃组件时只补后者；反向配置只补传播组件，并有「只想可燃则省去传播接线」的说明。证据见[重复组件返回值](../../../raw/current/scripts/entityscript.lua#L632-L646)及[传播 helper](../../../raw/current/scripts/standardcomponents.lua#L200-L212)。
- **R10**：[鳞片燃烧表](../../../wiki/reference/prefabs/boss-materials.md#L98-L98)已锚定龙蝇构造全文 624 行。对该文件搜 `burnable|Make.*Burnable` 无命中，尸体定义未设 `makeburnablefn`，通用尸体构造按该字段才挂载。证据见[龙蝇全文](../../../raw/current/scripts/prefabs/dragonfly.lua#L1-L624)、[尸体定义](../../../raw/current/scripts/prefabs/corpses_defs.lua#L1081-L1090)、[通用门禁](../../../raw/current/scripts/prefabs/corpses.lua#L750-L756)。

独立重跑同一范围的 12 文件严格源码链接检查，BUILD `26474873`，1,021 处引用 0 错 0 警，另有 2 处标作不支持并跳过。本轮定向复验覆盖原问题和修订触及的调用链。

## 第二轮外部定向复验

2026-11-10，Steam build **26474873**。定向复验第一轮余下的 R02、R03、R04、R06、R07、R09，六项全部通过。加上首轮已通过的 R01、R05、R08、R10，**R01–R10 全部通过，可以收口**。

| 编号 | 本轮核验结果 |
|---|---|
| R02 | [外壳来源和改写段](../../../wiki/reference/prefabs/shell-materials.md#L80-L90)已写明受控燃烧保留碎片且不闷烧，[钢丝绵来源](../../../wiki/reference/prefabs/steelwool.md#L70-L75)保留「落地尝试闷烧」并将灭火限定到普通燃烧。[叶索引 L31–34](../../../wiki/reference/prefabs/animal-drops-index.md#L31-L34)区分熊皮、鳞片和外壳的适用范围。对应[碎片构造](../../../raw/current/scripts/prefabs/slurtle_shellpieces.lua#L27-L43)、[闷烧与改写门禁](../../../raw/current/scripts/components/lootdropper.lua#L398-L442)。 |
| R03 | [伏特羊角结算段和表](../../../wiki/reference/prefabs/lightninggoathorn.md#L58-L68)明确直接掉落、化尸转存互斥，单次死亡生成一次名单，尸体结算传入显式列表。熄灭表同步补齐变异、跳过及保护标记。对应[死亡入口](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[尸体熄灭](../../../raw/current/scripts/prefabs/corpses.lua#L45-L52)和[列表提交](../../../raw/current/scripts/prefabs/corpses.lua#L489-L495)。 |
| R04 | [熊皮陷阱](../../../wiki/reference/prefabs/boss-materials.md#L133-L133)已将单项的随机清空与多项换毛丛分开，并保留啃空按旧级别处理。与[同页结算](../../../wiki/reference/prefabs/boss-materials.md#L74-L74)、[毛丛页](../../../wiki/reference/prefabs/furtuft.md#L51-L51)一致。对应[写级顺序](../../../raw/current/scripts/prefabs/corpses.lua#L158-L184)和[单项、多项分支](../../../raw/current/scripts/prefabs/corpses.lua#L465-L485)。 |
| R06 | [蛞蝓龟窝来源表](../../../wiki/reference/prefabs/shell-materials.md#L82-L82)、[陷阱](../../../wiki/reference/prefabs/shell-materials.md#L161-L161)和[索引](../../../wiki/reference/prefabs/animal-drops-index.md#L33-L33)均以死亡回调为 0.66 秒起点，烧尽爆炸取消待执行掉落。对应[死亡回调](../../../raw/current/scripts/prefabs/slurtlehole.lua#L44-L55)、[爆炸移除](../../../raw/current/scripts/components/explosive.lua#L151-L157)、[任务取消](../../../raw/current/scripts/entityscript.lua#L1730-L1746)。 |
| R07 | [外壳概述](../../../wiki/reference/prefabs/shell-materials.md#L22-L22)已把 20 写成受上限截断的请求值。[导航](../../../wiki/reference/prefabs/shell-materials.md#L59-L62)写清先判 `IsAboveGroundAtPoint`、再在 `IsVisualGroundAtPoint` 为真时查玩家。[示例](../../../wiki/reference/prefabs/shell-materials.md#L152-L152)仍明确它未安装原版站位门禁。对应[站位条件](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L59-L79)、[回血请求](../../../raw/current/scripts/components/repairable.lua#L106-L115)、[生命上限](../../../raw/current/scripts/components/health.lua#L565-L574)。 |
| R09 | [洞穴掉落陷阱](../../../wiki/reference/prefabs/cave-drops.md#L149-L150)已把「每次跨档」改为满足各阶段回调条件时各 1 张，普通／悲惨形态上限 2／3 张。普通形态 71%→39% 只抛 1 张，致死一击不抛阶段皮。与[阶段回调](../../../raw/current/scripts/prefabs/toadstool.lua#L700-L737)、[阈值触发顺序](../../../raw/current/scripts/components/healthtrigger.lua#L25-L38)及[索引血线](../../../wiki/reference/prefabs/animal-drops-index.md#L36-L36)一致。 |

同一范围的 12 文件严格源码链接检查通过，1,023 处引用（1,020 条行引用、3 条文件引用）0 错、0 警，另有 2 处工具标作不支持并跳过。本轮覆盖上述定向断言与涉及的入口，未运行游戏。

## 第二轮修订回执（2026-11-10）

针对第一轮定向复验的 6 项「部分修复」，按原编号再修订；只改页面与索引，不改日志、台账、基线。

| 编号 | 修订位置与自验 |
|---|---|
| R02 | shell L80／L81／L84／L90：碎片受控燃烧只保留原名、**不闷烧**（slurtle_shellpieces 无 burnable）；闷烧门禁限定自身带 burnable 的掉落物（lootdropper#L398-L407）；「先灭火」限定普通火。steel L72／L75 同步。索引 L31 改为「熊皮未换算时普通燃烧转灰、受控保留（不闷烧），鳞片不进入」，L33 改为「普通燃烧转灰、受控保留（碎片自身无 burnable、不闷烧）」并改「死亡回调后 0.66 秒」。 |
| R03 | lightninggoathorn L58 改为「单次死亡只生成一次；直接掉落与转存互斥」；L65 熄灭行带「未变异、无 _skip_extinguish_fade／no_destroy_on_burn」条件。 |
| R04 | boss-materials L133 拆三分支：旧 meat_level<2 跳过；=2 且单项走保留判定（可能清空，存活条目保持原名）；=2 且多项含熊皮才换毛丛。 |
| R06 | shell L82 改为「同一个 OnKilled 启动动画并排 0.66 秒后的 OnDoKilled，计时起点是死亡回调」。 |
| R07 | shell L22 改为「请求回 20，按生命上限截断（95/100 回 5）」；L62 改为 IsAboveGroundAtPoint 未放行且 IsVisualGroundAtPoint 为真才查半径 1 玩家。 |
| R09 | cave-drops L149 改为「各阶段满足回调条件时各 1 张（最多 2／3 张）」。 |

机械校验（主代理）：12 文件 validate_page --check-indexes 逐页 0 issue（导航另报 11 个非本批 B13 页未登记）；validate_source_links --strict 1,023 处引用（1,020 行＋3 文件）0 错 0 警，2 处标不支持并跳过；6 主页面基线复核一致。未运行游戏。
