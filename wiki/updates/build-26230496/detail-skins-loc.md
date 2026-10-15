# 明细：语音／本地化／皮肤资源

资源类条目按 write-update 规则降审阅优先级；但 Lua 皮肤脚本与注册数据不作"无兼容影响"结论。

| 编号 | 变化 | 影响接入／条件 | 证据位置 | 核验状态 |
|---|---|---|---|---|
| M-1 | 17 个角色语音文件（speech_walter/wanda/warly/wathgrithr/waxwell/webber/wendy/wickerbottom/willow/wilson/winona/wolfgang/woodie/wormwood/wortox/wurt/wx78）新增 Halloween 2026 描述条目：LEAFPILE、PUMPKIN_BOMB、WITCHBROOM（含 LIVING 变体键）。 | 文案数据；WITCHBROOM 用嵌套表（GENERIC/LIVING 两键，配合 P-2 的 GetStatus/DisplayNameFn）。 | 各 speech_*.lua 新增块（"-- Halloween 2026" 注释） | 已核验 |
| M-2 | 3 个本地化文件（languages/strings.pot、chinese_s.po、chinese_t.po）各新增约 532 行条目（新动作/物品/皮肤键的模板与简繁翻译）。 | 翻译数据；模组引用新键走 STRINGS 常规路径。 | 各 languages/*.po|pot 新增块 | 已核验 |
| M-3 | `skin_assets.lua` 新增 24 行皮肤资源登记（hallowed 建筑皮肤等）。 | 皮肤资源注册；配合 M-5。 | 新:skin_assets.lua（新增块） | 已核验 |
| M-4 | `skin_strings.lua` 新增 32 行皮肤名称/描述字符串。 | 文案数据。 | 新:skin_strings.lua（新增块） | 已核验 |
| M-5 | `skins_defs_data.lua` 新增约 195 行皮肤定义（fence/fencegate/wall_hay/pighouse 的 hallowed 变体、armor_rocky_spikey、hatbat_bosscorpse_vampire 等，release_group 187）。 | 皮肤定义数据；与 R-14 组、P-32 注册、M-3 资源配套。 | 新:skins_defs_data.lua（新增块） | 已核验 |
| M-6 | `versioning_skins.lua` `VERSION = 187`（旧 186）。 | 皮肤版本位前移；模组皮肤若硬编码版本位需同步。 | 新:versioning_skins.lua:VERSION 行 | 已核验 |
