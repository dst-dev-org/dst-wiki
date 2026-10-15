---
title: modinfo 自定义游戏模式
created: 2026-10-11
updated: "2026-10-28"
type: api
summary: 在 modinfo.lua 声明 game_modes，理解模式条目、settings 缺省与世界设置覆盖
aliases: [game_modes, 游戏模式, settings, AddGameMode, invalid_recipes, ghost_enabled, portal_rez, reset_time]
tags: [modding, global]
dst_build: "steam-buildid:26230496"
sources: [modindex.lua, gamemodes.lua, recipe.lua, recipes.lua]
reliability: verified
validation: source-reviewed
status: active
---

# modinfo 自定义游戏模式

## 快速判断

- 优先使用：确需一个可选模式时在 `modinfo.lua` 写 `game_modes`；普通物品／角色模组不用声明。其他元信息见 [[reference/api/modinfo/fields]]。
- 重要副作用：同名模式直接覆写已有 `GAME_MODES[name]`；`settings` 逐键复制到模式表，可以覆盖初始化字段，不是配置下拉框。[注册:186](../../../../raw/current/scripts/gamemodes.lua#L186)、[复制:565](../../../../raw/current/scripts/modindex.lua#L565)
- 修改边界：注册模式不等于选中模式或实现完整玩法。`settings` 是开放表，未知键只保存、不自动产生效果；下文区分初始化值与实际查询行为。

## 关键接口

| 模式条目字段 | 示例 | 缺省与作用 |
|---|---|---|
| `name` | `name = "mod_starter_mode"` | 必须提供唯一字符串标识；勿复用 `survival` 等原版名字，注册函数无冲突保护。 |
| `label` | `label = "示例模式"` | 提供界面名称，写入模式的 `text`；可用 `ChooseTranslationTable`，选择器缺省显示 blank。 |
| `description` | `description = "一个自定义模式入口。"` | 缺省空字符串；写入模式说明。 |
| `settings` | `settings = {invalid_recipes = {"torch"}}` | 可省略；存在时逐键覆盖新模式表，选实际需要的键。不是 `configuration_options`。 |

依据：[装载:565–573](../../../../raw/current/scripts/modindex.lua#L565-L573)、[选择器:213](../../../../raw/current/scripts/gamemodes.lua#L213)、[名称与说明查询:245–263](../../../../raw/current/scripts/gamemodes.lua#L245-L263)。

### settings 初始化值与查询边界

以下是 `AddGameMode` 实际生成的字段，示例是 `settings` 表内的成员；省略则保留初始化值。不是完整开放键枚举，也不保证每个初始化值仍驱动当前玩法。[初始化:186–202](../../../../raw/current/scripts/gamemodes.lua#L186-L202)

| 键 | 初始化值／声明示例 | 查询边界 |
|---|---|---|
| `invalid_recipes` | `{}`；`invalid_recipes = {"torch"}` | `IsRecipeValidInGameMode` 按配方名查排除表；不是 Prefab 名清单。 |
| `level_type` | `LEVELTYPE.SURVIVAL` | `GetLevelType` 读模式表；modinfo 环境没有 `LEVELTYPE`，通常省略，不直接粘贴全局常量表达式。 |
| `mod_game_mode` | true；`mod_game_mode = true` | `GetIsModGameMode` 读此标志；默认已标为模组模式。 |
| `spawn_mode` | `"fixed"`；`spawn_mode = "fixed"` | 当前 `GetSpawnMode` 读取世界设置，缺省 fixed；不能承诺改模式表就改变出生行为。 |
| `resource_renewal` | false；`resource_renewal = false` | 当前 `GetHasResourceRenewal` 读取世界设置，缺省 false。 |
| `ghost_sanity_drain` | false；`ghost_sanity_drain = false` | 当前同名查询读取世界设置，缺省 true，与新模式初始化值不同。 |
| `ghost_enabled` | true；`ghost_enabled = true` | 当前查询读取世界设置；另受 `revivable_corpse` 属性影响。 |
| `portal_rez` | false；`portal_rez = false` | 当前 `GetPortalRez` 读取世界设置，缺省 false。 |
| `reset_time` | nil；`reset_time = {time = 120, loadingtime = 180}` | 当前查询读取世界设置，缺省为此表；时间单位与引擎消费未核验，仅在模式表写 nil 不保证关闭世界重置。 |
| `text` / `description` | 来自条目的 label／description | `settings` 同名键会覆盖外层文字；优先使用条目字段，避免重复赋值。 |
| `modded_mode` | true | 注册内部标志，通常无需覆盖。 |

通用 `GetGameModeProperty` 优先读取非 nil 的世界设置，再回退模式表；上表专用查询中多项直接读取世界设置，没有此回退。[通用:205](../../../../raw/current/scripts/gamemodes.lua#L205)、[专用:269–308](../../../../raw/current/scripts/gamemodes.lua#L269-L308)

其他原版事件模式也有专属键／函数；这些不是普通模式的固定字段契约。要添加额外键，继续核验对应消费者，不以“装载器接受任意键”推断功能已实现。

## 典型接入

### 注册一个最小模式入口

合并到已有 `modinfo.lua`，保留基本元信息；将前缀改成自己的唯一名称。不需要美术资源；不附带世界生成、事件玩法或参数面板。

```lua
game_modes = {
    {
        name = "mod_starter_mode",
        label = "示例模式",
        description = "自定义模式入口，排除火把配方。",
        settings = {
            invalid_recipes = {"torch"},
        },
    },
}
```

`torch` 是原版火把配方名；`GetValidRecipe` 调用模式有效性查询并在被排除时返回 nil。本例覆盖这条查询路径，未核验所有制作入口是否都经过它，不承诺完整禁止制作。模式选择、创建世界和规则实现需另行接入。[配方:425](../../../../raw/current/scripts/recipes.lua#L425)、[查询:211](../../../../raw/current/scripts/recipe.lua#L211)、[排除判断:302](../../../../raw/current/scripts/gamemodes.lua#L302)

## 常见陷阱与限制

- **照搬旧无尽／荒野模式字段**：当前源码将 `endless`、`wilderness` 的错误回退到 survival；不要把旧教程的模式字段当当前世界设置实现。[回退:169](../../../../raw/current/scripts/gamemodes.lua#L169)
- **访问 `GLOBAL`／`LEVELTYPE` 出错**：modinfo 的初始环境只注入 folder_name、locale 与语言 helper，无法直接使用 modmain 的全局代理；保留注册默认 level_type，确需更换再核验值与世界生成。[环境:599](../../../../raw/current/scripts/modindex.lua#L599)
- **改模式定义但仍是旧值**：同版本可能使用缓存元信息；命名冲突、世界设置覆盖与缓存需分别排查。缓存及版本说明见 [[reference/api/modinfo/fields#常见陷阱与限制]]。

## 验证记录

- 增量复查：2026-10-25，Steam buildid 26148036；锚点重定位（modindex +6 段：559→565、559–567→565–573、593→599），装载／settings 复制逻辑未变，新增 CanDownloadWorkshopMods 不涉 game_modes；gamemodes／recipe 无变化，recipes 变更（nightlight 成本、新图纸配方）不涉 invalid_recipes 语义，结论维持。
2026-10-11，Steam buildid 25643519：已审阅 game_modes 装载、AddGameMode 初始化／同名覆盖、模式选择器、名称／说明与世界设置取值、火把配方及 GetValidRecipe 调用链。示例经 Full Moon 3.0.0／Lua 5.1 语法与原字节往返检查。**未运行 DST**；模式选择、世界生成／重载、配置覆盖和所有制作入口未验证，开放 settings 与专属事件模式不声称已完整审校。
