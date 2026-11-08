# 明细：界面（U）

build 26335394 → 26474873。条目按五字段契约，编号 U-序号 在本报告内唯一。

| 编号 | 变化 | 影响接入／条件 | 证据位置 | 核验状态 |
|---|---|---|---|---|
| U-1 | `screens/mapscreen.lua` 地图开／关音效：`OnBecomeInactive` 补 `map_close`、`OnBecomeActive` 补 `map_open`（各 +2 行，净 +4）。 | 地图界面打开／关闭路径替换或监听的模组：两处直接 `TheFrontEnd:GetSound():PlaySound`，无事件派发；替换地图界面的模组需自带音效。 | 旧:scripts/screens/mapscreen.lua:109-134；新:scripts/screens/mapscreen.lua:109-140（close :113、open :139） | 已核验 |
| U-2 | `screens/redux/scrapbookdata.lua` 图鉴数据表：新增 5 行 `halloween_ornament_7`／`_8`／`_9`／`_10`、`turf_vent`（净 +5 行）；既有行更新：`bat_boss` 掉落列去 `batwin`；`bunnyman`／`pigman`／`pigguard`／`mermking`／`mound` 掉落列补万圣节糖果／南瓜弹／装饰（与 P-3~P-6 掉落门对应）；`leafpile` 补 `burnable=true`＋掉落列（装饰 1-10、灰、小饰品）；`turfcraftingstation` deps 补 `turf_vent`；`witchbroom` 行燃料字段（`fueledmax`/`fueledrate`/`fueledtype1`）替换为 `finiteuses=125`。 | 读 scrapbookdata 表做图鉴扩展或数据比对的模组：5 个新条目可索引；witchbroom 的耐久来源字段名从 fueled 族变 finiteuses。掉落列与 [P-3](detail-prefabs.md)~[P-6](detail-prefabs.md)、[R-5](detail-root.md) 同源，仅为图鉴展示数据。 | 旧:scripts/screens/redux/scrapbookdata.lua:106-1150；新:scripts/screens/redux/scrapbookdata.lua:106-1159（witchbroom 行 :269 附近、turf_vent :578 附近、halloween 组 :832 附近） | 已核验 |
