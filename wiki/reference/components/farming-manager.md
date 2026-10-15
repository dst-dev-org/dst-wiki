---
title: "farming_manager"
created: "2026-10-18"
updated: "2026-10-28"
type: component
summary: "读写世界农田地块的三养分与水分，区分坐标、资格检查、增量限幅和作物压力结算。"
aliases: ["土壤养分", "地块浇水", "锄地补水施肥", "farming_manager"]
tags: [component, lifecycle, data, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/farming_manager.lua", "prefabs/world.lua", "components/map.lua", "components/farmtiller.lua", "prefabs/farm_hoe.lua", "standardcomponents.lua", "components/wateryprotection.lua", "prefabs/wateringcan.lua", "prefabs/farm_plants.lua", "prefabs/farm_plant_defs.lua", "tuning.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# farming_manager

## 快速判断

- 职责与适用：服务端世界组件，按地块管理土壤养分／水分及覆盖层。本页只覆盖这组读写、作物结算与必要初始化／存读，不是全部农作系统说明。[宿主](../../../raw/current/scripts/prefabs/world.lua#L568-L583) [数据网格](../../../raw/current/scripts/components/farming_manager.lua#L127-L138)
- 优先使用：三养分增减用 `AddTileNutrients(tx,ty,...)`；浇水用 `AddSoilMoistureAtPoint(x,y,z,delta)`。前者接地块坐标，后者接世界坐标；不要混用。[养分接口](../../../raw/current/scripts/components/farming_manager.lua#L368-L398) [水分接口](../../../raw/current/scripts/components/farming_manager.lua#L461-L475)
- 重要副作用：养分增量各限 0～100，但不检查是不是农田；水分增量仅对有覆盖层的地块生效，无显式成功返回。`CycleNutrientsAtPoint` 返回是否缺养分，true 不是成功。[增量](../../../raw/current/scripts/components/farming_manager.lua#L394-L405) [浇水](../../../raw/current/scripts/components/farming_manager.lua#L461-L470)
- 修改边界：从 `TheWorld.components.farming_manager` 取实例，不往物品／客户端重新挂载；组件构造断言主模拟，网格等 worldmapsetsize 才初始化，安装 Hook 时不立刻调用地块接口。[服务端断言](../../../raw/current/scripts/components/farming_manager.lua#L25-L27) [初始化](../../../raw/current/scripts/components/farming_manager.lua#L127-L138)

## 关键接口

### 坐标、单位与资格

世界坐标为 `(x,y,z)`，其中 y 是高度；`local tx, ty = TheWorld.Map:GetTileCoordsAtPoint(x,y,z)` 得到地块二维坐标，ty 不是高度。下表名字含 Tile 的三养分接口接 tx／ty，AtPoint 接世界点。农田资格用 `Map:IsFarmableSoilAtPoint`；原肥料部署先做这项检查，再转换坐标写养分。[地图资格](../../../raw/current/scripts/components/map.lua#L185-L187) [部署完整路径](../../../raw/current/scripts/standardcomponents.lua#L1797-L1814)

三养分顺序为催长素／堆肥／粪肥。养分与水分是各自的数据单位，不是人物湿度或压力点；本快照水分上限 100、刷新周期 5 秒，水分 Setter 的下限采用世界 wetness。[顺序](../../../raw/current/scripts/tuning.lua#L5854-L5856) [水分参数](../../../raw/current/scripts/tuning.lua#L5774-L5780) [限幅](../../../raw/current/scripts/components/farming_manager.lua#L96-L106)

| 接口 | 输入／返回 | 作用、前提与证据 |
|---|---|---|
| `GetTileNutrients(tx,ty)` | 返回三个数字，不是 table | 优先网格值，无记录则查当前地皮定义的 nutrients，再无则 0,0,0；有返回不证明该处是农田。[实现](../../../raw/current/scripts/components/farming_manager.lua#L368-L382) |
| `AddTileNutrients(tx,ty,d1,d2,d3)` | 三项增量；返回 true | 读当前值后分别夹到 0～100，再写网格及已有覆盖层；不内建农田检查。负增量可减养分。[实现](../../../raw/current/scripts/components/farming_manager.lua#L394-L398) |
| `SetTileNutrients(tx,ty,n1,n2,n3)` | 三项绝对值；无显式返回 | 直接按位编码存储并更新已有覆盖层，不按 0～100 限幅；调用方须保证有效整数范围，不把任意大值／小数当作安全输入。[编码](../../../raw/current/scripts/components/farming_manager.lua#L88-L94) [写入](../../../raw/current/scripts/components/farming_manager.lua#L384-L392) |
| `AddSoilMoistureAtPoint(x,y,z,delta)` | 水分增量；无显式返回 | delta 非 0 且目标有农田覆盖层时才更新；Setter 按世界 wetness～上限夹取，干湿跨零时通知该地块饮水者。不是范围浇水或创建农田。[实现](../../../raw/current/scripts/components/farming_manager.lua#L461-L470) [通知](../../../raw/current/scripts/components/farming_manager.lua#L96-L123) |
| `IsSoilMoistAtPoint(x,y,z)` | boolean | 网格水分 >0 **或**世界 wetness >0；不是水量 getter，也不验证农田或给出 0～1 比例。[实现](../../../raw/current/scripts/components/farming_manager.lua#L472-L475) |
| `GetTileBelowSoil(tx,ty)` | undertile 的返回 | 读取被农田覆盖的原地皮记录，不是当前地皮查询，也不是农田资格判定。[实现](../../../raw/current/scripts/components/farming_manager.lua#L457-L459) |

### 阶段养分消费与压力

`CycleNutrientsAtPoint(x,y,z,consume,restore,test_only)` 返回 **depleted（缺养分）**；没有覆盖层直接 true。consume 用三项连续数组，restore 用养分类别索引到 true 的集合；原定义将不消耗的类别写为 true，其余 nil。[实现入口](../../../raw/current/scripts/components/farming_manager.lua#L400-L418) [原数据构造](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L126-L131)

有 consume 且 test_only=true 时，只检查不足并返回；正式调用即使不足也会扣掉实际可取部分，再将**实际消耗总数**分配回 restore 中的类别，余数随机分配，最后写入增量。restore 为 nil 时不返还；不要传空 restore 表，代码会以其类别数作除数并用于 math.random。[预览／返还／写入](../../../raw/current/scripts/components/farming_manager.lua#L420-L454)

原作物查看检测传 test_only=true，阶段转换才正式消费，返回值写 nutrients 压力项后 checkpoint；仅施肥不会回减之前累计压力。水分压力另看阶段湿润时间比例，见 [[reference/components/farmsoildrinker]]；压力累计见 [[reference/components/farmplantstress]]。[检测分工](../../../raw/current/scripts/prefabs/farm_plants.lua#L203-L217) [阶段消费者](../../../raw/current/scripts/prefabs/farm_plants.lua#L238-L253)

## 典型接入

### 普通园艺锄成功锄地后，为所在农田地块补 10 水分及各 5 养分

放目标模组 `modmain.lua`，只包装 `farm_hoe` 实例的 Till，无新资源。保留原土堆生成、返回值和动作耗损；先成功再补给，不在 PostInit 中直接操作尚未初始化的世界网格。[原 Till](../../../raw/current/scripts/components/farmtiller.lua#L5-L16) [锄头接入](../../../raw/current/scripts/prefabs/farm_hoe.lua#L112-L122) [注册名](../../../raw/current/scripts/prefabs/farm_hoe.lua#L256-L258) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("farm_hoe", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local tiller = inst.components.farmtiller
    if tiller == nil then return end
    local original = tiller.Till
    tiller.Till = function(self, pt, doer)
        local success = original(self, pt, doer)
        if success then
            local world = GLOBAL.TheWorld
            local manager = world.components.farming_manager
            if manager ~= nil then
                local x, y, z = pt:Get()
                local tx, ty = world.Map:GetTileCoordsAtPoint(x, y, z)
                manager:AddTileNutrients(tx, ty, 5, 5, 5)
                manager:AddSoilMoistureAtPoint(x, y, z, 10)
            end
        end
        return success
    end
end)
```

数值均为每次成功的增量，受上限限制；影响**整个地块**及其中所有作物，不只新土堆。多次合法锄地可重复补给，本例没有每格一次或额外资源扣费；金锄／热泉锄不在目标内。停用并重新初始化后恢复原方法，已写入的水分／养分继续由世界保存和消费，不回滚。新工具骨架／动作耐久见 [[reference/components/farmtiller]]。

原版参照：肥料经部署 helper 转地块坐标再加三养分；洒水壶配置 wateryprotection，后者虽会对附近实体施加效果，土壤水分只更新指定点所在的一格，不随实体效果半径扩展。普通浇水接口不提供施肥物品消费或洒水壶补水逻辑，相关工具待单独核验。[施肥](../../../raw/current/scripts/standardcomponents.lua#L1797-L1814) [洒水壶配置](../../../raw/current/scripts/prefabs/wateringcan.lua#L131-L138) [浇水消费者](../../../raw/current/scripts/components/wateryprotection.lua#L43-L56) 肥料配置见 [[reference/components/fertilizer]]。

## 常见陷阱与限制

- **AddTileNutrients 返回 true 就代表合法施肥**：它不检查地皮、部署资格或材料；复用原部署资格／消费路径，或像本例放在原 Till 成功之后。[接口](../../../raw/current/scripts/components/farming_manager.lua#L394-L398) [原资格](../../../raw/current/scripts/standardcomponents.lua#L1812-L1814)
- **只 SetTileNutrients 就能造农田**：写网格不会创建覆盖层或切换地皮。地形事件进入 FARMING_SOIL 时才建立覆盖层并初始化水分；离开时清水分／覆盖层，但这条分支不删除养分网格记录，不声称重铺必然重抽养分。[地形处理](../../../raw/current/scripts/components/farming_manager.lua#L144-L181)
- **养分充足等于无压力，或浇一次永久湿润**：养分只是一项压力；天气／温度与各作物负饮水率仍按周期更新水量，LongUpdate 也会刷新。不要通过反复调用私有刷新方法模拟公共浇水接口。[周期](../../../raw/current/scripts/components/farming_manager.lua#L127-L137) [水量更新](../../../raw/current/scripts/components/farming_manager.lua#L299-L327) [长更新](../../../raw/current/scripts/components/farming_manager.lua#L355-L362)
- **把加载的覆盖层／饮水者当持久化原对象**：当前保存压缩后的养分与水分网格；读档恢复网格并按农田地皮重建覆盖层，饮水者另走登记事件。模组不要覆盖原世界 OnSave／OnLoad 或把私有网格当公开字段。[保存](../../../raw/current/scripts/components/farming_manager.lua#L482-L497) [恢复](../../../raw/current/scripts/components/farming_manager.lua#L530-L555) [登记](../../../raw/current/scripts/components/farming_manager.lua#L613-L614)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 3 处（world 宿主分支 +4~+6、tuning 养分顺序／水分参数段 +14）；无结论修订，仅锚点迁移——SOIL_MOISTURE_UPDATE_TIME=5、SOIL_MAX_MOISTURE_VALUE=100 及养分顺序值未变（tuning-hunk-map 5 改值键不含），farming_manager.lua、farmtiller、farm_hoe、farm_plants 系全部未变（world 本版改动在 MakeWorld 其他区域；map.lua 改动在 925 行之后，IsFarmableSoilAtPoint 引用未变）。依据：tuning-hunk-map 键级审阅＋变更清单命中 prefabs/world.lua、components/map.lua、tuning.lua。
- 源码审阅：2026-10-18，Steam buildid 25643519；覆盖世界安装／初始化、坐标与资格、三养分编码／读写／增量、干湿限幅与周期、阶段预览／消费及缺养分返回、地形／存读和锄地补给例。洒水壶仅核验配置及 wateryprotection 的土壤调用，不覆盖完整补水／动作流程；杂草、果蝇、旧存档迁移与网络显示未全面审校。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整 modmain 示例通过已有 Full Moon Lua 5.1 syntax-only，不证明实际养分、水量或耗损效果。
- 游戏运行：未运行；重复锄地、上限与世界湿度下限、同格多植物、存读／地形切换、分片及其他模组包装顺序待实测。
