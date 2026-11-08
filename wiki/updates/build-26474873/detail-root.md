# 明细：根脚本（R）

build 26335394 → 26474873。条目按五字段契约：变化／影响接入条件／证据位置／核验状态，编号 R-序号 在本报告内唯一。

| 编号 | 变化 | 影响接入／条件 | 证据位置 | 核验状态 |
|---|---|---|---|---|
| R-1 | `tuning.lua` `Tune()` 默认档新增键 `SQUID_MIN_COUNT = 2`（单群投放的尝试次数下限；插在 `SQUID_MAX_FISH` 与 `SQUID_MAX_NUMBERS` 之间，仅 +1 行）。 | 消费 `TUNING.SQUID_*` 的代码与模组：新增可读键，无删除、无既有键改值；不读该键的用法不受影响。鱿鱼群生成消费见 [C-2](detail-components.md)（作 `math.random` 下限）、世界设置档位见 [R-2](detail-root.md)。 | 旧:scripts/tuning.lua 无该键；新:scripts/tuning.lua:6099 | 已核验 |
| R-2 | `worldsettings_overrides.lua` `applyoverrides_pre` 鱿鱼群（`squids`）五档预设各补 `SQUID_MIN_COUNT`：never 0／rare 1／default 2／often 3／always 3；另 always 档 `SQUID_MAX_NUMBERS["full"]` 2→3（其余档位数值未动）。净 +5 行。 | 世界设置读取 `SQUID_MIN_COUNT` 的新链路：档位值随 squid 选项走；按旧默认值（无该键）写死的模组在 default 档行为不变（2），never 档从「无键」变 0。满月 `SQUID_MAX_NUMBERS.full` 仅在 always 档变大（2→3；该档满月概率 0.065 本就非零，默认档满月概率 0）。 | 旧:scripts/worldsettings_overrides.lua:1089-1174；新:scripts/worldsettings_overrides.lua:1089-1180（never :1092、always full :1174） | 已核验 |
| R-3 | `versioning_prefabs.lua` `VERSION` 9→10。`versioning.lua` 用它拼 `VERSIONING.CURRENTVERSION`（`<major>.<N>_<skins>`）。 | 消费 `VERSIONING.CURRENTVERSION` 字符串的模组：子版本段从 `.9` 变 `.10`（延续上版报告 R-5 的「可能带 `.N` 子版本段」结论）；按等值比较该串的逻辑需容忍子版本递增。与 P-3／P-4／P-5 的存档修正同属本版存档兼容收口，但迁移逻辑本身不经该常量。 | 旧:scripts/versioning_prefabs.lua:5；新:scripts/versioning_prefabs.lua:5；消费:scripts/versioning.lua:76-80 | 已核验 |
| R-4 | `prefabskin.lua` waterballoon 皮肤的 `waterballoon_init_fn`／`waterballoon_clear_fn` build 名 `swap_waterballoon`→`waterballoon`。 | waterballoon 皮肤挂载／卸载用的 build 名；引用旧 build 名的皮肤类模组需跟改。与本版 P-1 的 floater `swap_data` 批量修复同方向（重挂皮肤强制恢复动画的资源正确性）。 | 旧:scripts/prefabskin.lua:759-760；新:scripts/prefabskin.lua:759-760 | 已核验 |
| R-5 | `scrapbook_prefabs.lua` 图鉴收录白名单新增 5 项：`halloween_ornament_7`／`_8`／`_9`／`_10`、`turf_vent`（官方注释 "this one was missed"）。 | 图鉴数据消费方（scrapbook UI）：5 个 prefab 进入图鉴显示范围；对应数据行见 [U-2](detail-ui.md)。 | 旧:scripts/scrapbook_prefabs.lua:2133-2135；新:scripts/scrapbook_prefabs.lua:2136-2142 | 已核验 |
