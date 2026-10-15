# 组件优先清单与索引重整检查点

建档日期：2026-10-23。基线：通用 C28 收尾，296 个组件已建、502 个候选未建；源码标识为 Steam buildid `25643519`。本文件用于工程推进，不进入 Wiki 知识检索。

## 固定范围与完成判定

- 本阶段固定为下列 98 个候选，取自旧路线 G01～G12 中截至建档时仍未建的项目；其他分组即使另有交付，也不抵扣本清单。后续不按不断缩小的“剩余数量”重新定义范围。
- “98 项”是待覆盖组件数，不代表已经确认需要 98 篇同等深度的长页。核验后按职责采用详细页、简要页或有明确去向的合并／迁移说明；每项均须有可查询说明及覆盖登记，不因文件短或名称像旧机制就直接跳过。
- 建档时完成 0／98。实时进度由下列标识符与 [覆盖清单](component-coverage.csv) 交叉核对；日常只读当前批次相关条目，不重读全部工程历史。完成依据是实际文档及登记，不能只用组件总数达到 394 来判定。
- 每批收尾更新 PLAN 中的检查点进度；发现本清单未覆盖数为 0 时，先明确提醒用户“98 项阶段已完成，接下来重整索引”，暂停继续生成下一批组件页，优先进入下面的索引阶段。提醒发生在后续实际推进中，无固定日期。

## 完成后先做索引重整

重新规划并编写 `wiki/index.md`、`wiki/reference/index.md`、`wiki/reference/components/index.md` 及相关主题导航；cookbook 导航按实际受影响范围处理。先讨论需求分组与检索路径，再修改索引，保持正文稳定地址与有效经验。

重点检查需求措辞能否命中正确入口、主题是否过大或混杂、相近组件能否快速选型、根导航与有依据的 Top 30 是否合理、各页是否唯一登记且链接有效。遵守现有懒加载和大小规则，以代表性查询核对路由；索引阶段完成后再恢复后续扩库。期间确有导航超限或失效，仍及时局部修复，不必等到阶段结束。

## 98 项基线清单

下列为固定标识符，文件位于 `raw/current/scripts/components/<标识符>.lua`；仅用于追踪范围，不证明接口职责或适用性。

| 原路线组 | 项数 | 固定标识符 |
|---|---:|---|
| G01 | 2 | `frostybreather`、`nonslipgritpool` |
| G02 | 10 | `aoeweapon_base`、`aoeweapon_leap`、`aoeweapon_lunge`、`groundpounder`、`slipperyfeet`、`slipperyfeettarget`、`spooked`、`targettracker`、`teamattacker`、`teamleader` |
| G03 | 6 | `autoterraformer`、`dataanalyzer`、`grabbable`、`summoningitem`、`tackler`、`tacklesketch` |
| G04 | 5 | `cyclable`、`decay`、`linkeditem`、`linkeditemmanager`、`powerload` |
| G05 | 8 | `cookbookupdater`、`digester`、`dryer`、`dryingracksaltcollector`、`feasts`、`forcecompostable`、`incinerator`、`incrementalproducer` |
| G07 | 19 | `batspawner`、`birdattractor`、`birdspawner`、`butterflyspawner`、`circler`、`followerherder`、`followermemory`、`formationfollower`、`formationleader`、`hatchable`、`hideout`、`leaderrollcall`、`lureplantspawner`、`minionspawner`、`objectspawner`、`rampingspawner`、`schoolspawner`、`shedder`、`squadmember` |
| G09 | 22 | `amorphous`、`amphibiouscreature`、`carefulwalker`、`coach`、`commander`、`crewmember`、`distancefade`、`distancetracker`、`embarker`、`grouptargeter`、`heavyobstaclephysics`、`heavyobstacleusetarget`、`joustsource`、`joustuser`、`physicsmodifiedexternally`、`playerspeedmult`、`pushable`、`replayproxy`、`strafer`、`stretcher`、`stuckdetection`、`unevenground` |
| G10 | 5 | `attunable`、`attuner`、`aura`、`simplemagicgrower`、`spell` |
| G11 | 12 | `custombuildmanager`、`forgerepair`、`forgerepairable`、`luckitem`、`luckuser`、`reloaditem`、`skinner`、`socketable`、`socketholder`、`symbolswapdata`、`transformer`、`wardrobe` |
| G12 | 9 | `giftreceiver`、`pointofinterest`、`recipestockpile`、`researchpointconverter`、`roseinspectable`、`roseinspectableuser`、`scrapbookable`、`shop`、`treasuremarked` |

合计 98 项；原路线 G06／G08 在本基线中没有未建项目。旧路线的历史数量不改写，当前完成状态继续由覆盖清单维护。

