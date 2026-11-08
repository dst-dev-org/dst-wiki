# Wiki 查询资产

## Prefab 名称速查

[prefab-names.csv](prefab-names.csv) 按注册名排序。两列分别为 `prefab` 和 `zh_name`。来源为 Steam build **26474873**（游戏版本 **762172**）简中运行时的 `Prefabs` 与 `STRINGS.NAMES`。原版注册共 8,053 个，2,713 个同名键值已与该版本的简中译文逐字对照。空值表示名称表中未取到该键值。个别名称保留显示换行。

用 `rg -n '皮弗娄牛|^beefalowool,' "<K>/wiki/assets/prefab-names.csv"` 查对应注册名，再定位 Wiki 的标题、别名或源码定义。不同注册可以共享显示名。动态显示名按实际显示路径核验，玩家俗称归各知识页。

更新时从已确认 Steam build 的简中游戏重新导出，用工具箱 `prepare_prefab_catalog.py` 生成此表并核对名称差异。随表更新本说明。表中游戏名称保留 Klei 及相关权利人的原授权，参见 [授权范围](../../NOTICE)。
