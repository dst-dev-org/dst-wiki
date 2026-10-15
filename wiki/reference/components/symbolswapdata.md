---
title: "symbolswapdata"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "在物品上被动记录build、symbol和皮肤标志，供健身器等消费者决定如何在另一个AnimState显示该物品。"
aliases: ["symbolswapdata", "物品换符号数据", "重物显示资源", "build symbol is_skinned"]
tags: [component, data]
dst_build: "steam-buildid:26230496"
sources: ["components/symbolswapdata.lua", "prefabs/cavein_boulder.lua", "prefabskin.lua", "components/mightygym.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# symbolswapdata

## 快速判断

- 职责与适用：被动记录build、symbol、is_skinned三个字段，供其他宿主读取并显示物品；组件自己不调用OverrideSymbol、OverrideItemSkinSymbol或SetBuild。[symbolswapdata.lua:1-15](../../../raw/current/scripts/components/symbolswapdata.lua#L1-L15)
- 优先使用：自定义物品需要参与一个已明确读取这些字段的显示流程时，沿实际消费者的数据约定配置；原版巨石在服务端生产数据，健身器读取重量物品的字段决定覆盖方式。[cavein_boulder.lua:590-594](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L590-L594) [cavein_boulder.lua:625-627](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L625-L627) [mightygym.lua:126-157](../../../raw/current/scripts/components/mightygym.lua#L126-L157)
- 重要副作用：SetData只是赋值，不发事件或请求消费者重绘；is_skinned是真值时健身器走皮肤symbol接口，否则走普通OverrideSymbol。它不是自动联网的皮肤开关。[symbolswapdata.lua:5-9](../../../raw/current/scripts/components/symbolswapdata.lua#L5-L9) [mightygym.lua:132-145](../../../raw/current/scripts/components/mightygym.lua#L132-L145)
- 修改边界：构造只存inst，三个数据字段初始都未赋值；没有存读、更新、移除或网络同步钩子。资源有效性、使用端和清理归实际生产者／消费者。[symbolswapdata.lua:1-15](../../../raw/current/scripts/components/symbolswapdata.lua#L1-L15)

## 关键接口

| 接口／字段 | 签名或约束 | 条件、作用与证据 |
|---|---|---|
| `SetData(build,symbol,is_skinned)` | 三个值原样保存；无返回 | build／symbol应符合目标消费者的AnimState接口；省略第三参会写nil，不保留旧皮肤标志，也没有类型／资源验证。[symbolswapdata.lua:5-9](../../../raw/current/scripts/components/symbolswapdata.lua#L5-L9) |
| `build`、`symbol` | 消费者直接读取字段 | 这里的symbol是来源资源中的符号名；目标被覆盖槽由消费者另定，不是记录在此组件中。[mightygym.lua:132-137](../../../raw/current/scripts/components/mightygym.lua#L132-L137) |
| `is_skinned` | 消费者用真值／假值分支 | 健身器以此决定OverrideItemSkinSymbol或OverrideSymbol；不能只写true便认为皮肤资源已加载或完成同步。[mightygym.lua:132-145](../../../raw/current/scripts/components/mightygym.lua#L132-L145) |
| `GetDebugString()` | 返回字段格式化文本 | 未赋值字段用空串；is_skinned=false也因`or ""`变为空串，不能把调试文本空值当作区分false与nil的依据。该方法不应用外观。[symbolswapdata.lua:11-13](../../../raw/current/scripts/components/symbolswapdata.lua#L11-L13) |

## 典型接入

### 巨石记录来源资源，皮肤初始化改写记录

原版cavein_boulder服务端添加组件，写入`swap_cavein_boulder`／`swap_boulder`，省略is_skinned。SetVariation在选中非默认variation时，宿主一边OverrideSymbol改变巨石自身外观，一边SetData记录对应来源symbol；两步分别完成，说明数据记录本身不会改动画。[cavein_boulder.lua:590-594](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L590-L594) [cavein_boulder.lua:625-627](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L625-L627) [cavein_boulder.lua:222-240](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L222-L240)

PrefabSkin中的cavein_boulder_init_fn先清巨石swap_boulder覆盖，再SetData(build_name,"swap_boulder",true)，随后调用皮肤初始化。皮肤状态与记录更新来自这个宿主函数，symbolswapdata不主动观察GetSkinBuild或监听换皮肤；其他恢复路径应以各自实际调用为准。[prefabskin.lua:1786-1790](../../../raw/current/scripts/prefabskin.lua#L1786-L1790)

### 健身器读取记录，在自身显示重量物品

MightyGym.LoadWeight把物品放入自身inventory，再调用SetWeightSymbol(weight,slot)。SetWeightSymbol读取物品symbolswapdata：is_skinned为真时对健身器AnimState调用OverrideItemSkinSymbol，并传物品GUID；否则调用普通OverrideSymbol，目标槽由健身器slot_ids指定。不存在组件时该显示分支跳过；存在但build／symbol为空时没有额外保护。[mightygym.lua:126-157](../../../raw/current/scripts/components/mightygym.lua#L126-L157) [mightygym.lua:159-179](../../../raw/current/scripts/components/mightygym.lua#L159-L179)

UnloadWeight由健身器自己清swap_item／swap_item2覆盖并移除附加特效；移除数据组件或调用SetData(nil,nil,nil)都不会替消费者执行这份清理。自定义接入要确认实际消费者会重新读取和恢复外观，本页不将被动记录包装成通用换装demo。[mightygym.lua:210-229](../../../raw/current/scripts/components/mightygym.lua#L210-L229) [symbolswapdata.lua:1-15](../../../raw/current/scripts/components/symbolswapdata.lua#L1-L15)

## 常见陷阱与限制

- **只加组件就会在人物或机器上显示物品：** 构造没有默认资源或显示动作；先设置数据，再接已核验的具体读取流程。给任意物品添加同名组件不会自动创建消费者。[symbolswapdata.lua:1-9](../../../raw/current/scripts/components/symbolswapdata.lua#L1-L9)
- **改数据会立即刷新所有引用外观：** setter不通知、不持有显示目标；健身器在自己的LoadWeight／SetWeightSymbol流程读取数据，其他显示路径必须分别核验。[symbolswapdata.lua:5-9](../../../raw/current/scripts/components/symbolswapdata.lua#L5-L9) [mightygym.lua:159-179](../../../raw/current/scripts/components/mightygym.lua#L159-L179)
- **build／symbol就是目标槽名：** 健身器以slot_ids作为目标symbol、以数据symbol作为来源symbol，两者职责不同；同一记录可以被不同消费者放入不同槽位。[mightygym.lua:132-137](../../../raw/current/scripts/components/mightygym.lua#L132-L137)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件全文、巨石服务端安装／SetVariation、PrefabSkin巨石初始化、MightyGym完整SetWeightSymbol／LoadWeight／UnloadWeight，覆盖被动字段、数据生产与皮肤更新、实际覆盖分支和消费者清理责任。
- 游戏运行：未运行；未测换皮肤后已加载物品重绘、资源缺失、各客户端外观或读档重建。未展开绞盘等其他消费者、完整皮肤系统或健身器全状态机。
- 结构检查：`validate_page.py`通过；两页合并执行`validate_source_links.py --strict`，41个行号引用、0错误／0警告。本页无新增Lua示例；机械检查不代替游戏测试。
