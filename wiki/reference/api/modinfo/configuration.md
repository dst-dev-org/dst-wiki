---
title: modinfo 配置项与名称
created: 2026-10-11
updated: "2026-10-28"
type: api
summary: 查 modinfo 原生字段入口，生成配置项、分组标题和客户端选项，读取配置值，区分本地名称与语言
aliases: [modinfo, configuration_options, GetModConfigData, 配置项, 配置标题, 数值选项, 本地测试名称, folder_name, ChooseTranslationTable]
tags: [modding, global, server-client]
dst_build: "steam-buildid:26230496"
sources: [modindex.lua, modutil.lua, mods.lua, util.lua, screens/redux/modconfigurationscreen.lua, languages/loc.lua]
reliability: verified
validation: source-reviewed
status: active
---

# modinfo 配置项与名称

## 快速判断

- 优先使用：在 `modinfo.lua` 声明 `configuration_options`，在模组入口／`modimport` 文件用 `GetModConfigData` 读取；起步文件见 [[cookbook/mod-starter#最小-demo|模组骨架]]。[modindex.lua:599](../../../../raw/current/scripts/modindex.lua#L599)、[modutil.lua:329](../../../../raw/current/scripts/modutil.lua#L329
- 重要副作用：配置保存按 `name` 匹配；名称应稳定、唯一。`label` 是展示文字，不能作为读取键；改默认值不覆盖已有保存值。[modindex.lua:764](../../../../raw/current/scripts/modindex.lua#L764
- 修改边界：元信息有独立环境，注入 `folder_name`、`locale` 和 `ChooseTranslationTable`；没有入口的 `GLOBAL`、`env`、`GetModConfigData`。示例是可选扩展片段，合并到已有文件，不覆盖整个模组。[modindex.lua:599](../../../../raw/current/scripts/modindex.lua#L599

可选元信息的完整清单／示例见 [[reference/api/modinfo/fields]]；优先级、图标、依赖、版本兼容等按字段查阅，自定义模式见 [[reference/api/modinfo/game-modes]]。

## 关键接口

| 字段／接口 | 用法与范围 |
|---|---|
| `configuration_options` | 配置条目的顺序数组；每项有 `name`、`options`、非 nil 的 `default`；可选 `label`、`hover`。配置界面按数组遍历并筛格式。[界面:21](../../../../raw/current/scripts/screens/redux/modconfigurationscreen.lua#L21) |
| `options` | 每项 `{description = 展示文本, data = 实际值, hover = 可选提示}`；字符串、布尔或数值保持实际类型，`default` 选其中一个 `data`。[界面:156](../../../../raw/current/scripts/screens/redux/modconfigurationscreen.lua#L156) |
| 配置项 `client = true` | 非纯客户端模组的该条目在客户端配置界面显示，服务器界面不显示；它不代替 `client_only_mod`，也不自动让服务端读取本地设置。[界面:19](../../../../raw/current/scripts/screens/redux/modconfigurationscreen.lua#L19) |
| `GetModConfigData(key, get_local_config?)` | 入口环境包装器自动绑定 `modname`；默认取正常／服务器配置，第二参 `true` 请求本地配置。[包装器](../../../../raw/current/scripts/modutil.lua#L329)、[选择](../../../../raw/current/scripts/modindex.lua#L779) |
| 全局 `GetModConfigData(key, modname, get_local_config?)` | 在入口环境之外调用需显式传实际模组标识；返回保存值或默认值，无配置／无键时为 nil。[modutil.lua:34](../../../../raw/current/scripts/modutil.lua#L34 |

### 配置字段清单

以下是作者声明的配置结构；界面按 `ipairs` 遍历两层数组，`data` 与 `default` 保持同一类型，不使用 nil 作为选项值。

| 所在层／字段 | 示例 | 缺省与用途 |
|---|---|---|
| 配置项 `name` | `name = "feature_enabled"` | 提供稳定唯一的读取／保存键；缺失不显示。 |
| 配置项 `label` | `label = "启用功能"` | 可省略，显示文字退回 name。 |
| 配置项 `hover` | `hover = "是否启用功能"` | 可省略，选中条目时显示的说明。 |
| 配置项 `options` | `options = {{description = "开启", data = true}}` | 提供顺序数组；缺失不显示，不是任意文本输入框。 |
| 配置项 `default` | `default = true` | 声明一个选项的 data；false、0、空字符串可用。新条目无默认／保存值不显示。 |
| 配置项 `client` | `client = true` | 缺省／false 在两种界面均可显示；true 在非纯客户端模组中仅客户端界面显示，纯客户端模组不受该筛选。 |
| 选项 `description` | `description = "开启"` | 提供下拉展示文字；单个空字符串选项会被识别为标题。 |
| 选项 `data` | `data = true` | 实际读取值，常用 boolean／number／string；按业务选择，无游戏统一枚举。 |
| 选项 `hover` | `hover = "开启这一效果"` | 可省略，当前选项的独立说明，区别于条目的 hover。 |

`saved`／`saved_client`／`saved_server` 是加载保存值生成的状态，`value`／`is_header` 是界面派生状态，不作为作者可选配置手写；界面容许已有 saved 替代 default，不是省略新条目 default 的理由。[界面筛选:19](../../../../raw/current/scripts/screens/redux/modconfigurationscreen.lua#L19)、[说明:93](../../../../raw/current/scripts/screens/redux/modconfigurationscreen.lua#L93)、[标题:156](../../../../raw/current/scripts/screens/redux/modconfigurationscreen.lua#L156)、[恢复:761](../../../../raw/current/scripts/modindex.lua#L761)

## 典型接入

### 配置项与分组标题

将以下完整配置块放入 `modinfo.lua`，保留已有基本信息和运行端标志。已有 `configuration_options` 时合并条目，只保留一份赋值；替换 `feature_enabled`／`power_multiplier` 时同步读取键。

```lua
local function Section(id, label)
    return {
        name = id,
        label = label,
        options = {{description = "", data = 0}},
        default = 0,
    }
end

local function NumberOptions(first, steps, step)
    local options = {}
    for i = 0, steps do
        local value = first + i * step
        options[i + 1] = {description = value .. "", data = value}
    end
    return options
end

configuration_options = {
    Section("_section_basic", "基础设置"),
    {
        name = "feature_enabled",
        label = "启用功能",
        hover = "是否启用本模组的可选功能",
        options = {
            {description = "开启", data = true},
            {description = "关闭", data = false},
        },
        default = true,
    },
    {
        name = "power_multiplier",
        label = "效果倍率",
        options = NumberOptions(1, 4, 0.25),
        default = 1,
    },
}
```

`NumberOptions(1, 4, 0.25)` 生成 5 项：1、1.25、1.5、1.75、2；`steps` 是递增次数，取非负整数，`step` 为正数。值的单位为倍数；若展示百分比，改变 `description`，仍按消费者需要保留 `data` 单位和类型。生成函数是本例的本地 helper，不是 DST API。

标题也要满足配置条目格式。界面根据“恰好一个选项，且其 `description` 长度为 0”计算标题状态并隐藏下拉框；直接填写 `is_header = true` 没有这条接入作用。标题使用不同的 `_section_*` 键，不当作功能值读取；标题条目也会进入设置收集。[识别:168](../../../../raw/current/scripts/screens/redux/modconfigurationscreen.lua#L168)、[显示:127](../../../../raw/current/scripts/screens/redux/modconfigurationscreen.lua#L127)、[保存:211](../../../../raw/current/scripts/screens/redux/modconfigurationscreen.lua#L211)

### 读取配置值

在 `modmain.lua` 的代理设置后、消费配置的 `modimport` 之前放入以下片段。`MOD_OPTIONS` 是本例的模组环境表；导入文件可读取它。消费者按 `enabled` 决定是否注册功能，按 `power_multiplier` 使用倍数；此片段只读取，不自动实现效果或支持热更新。[环境](../../../../raw/current/scripts/mods.lua#L298)、[导入](../../../../raw/current/scripts/mods.lua#L301)

```lua
local enabled = GetModConfigData("feature_enabled")
if enabled == nil then
    enabled = true
end

local multiplier = GetModConfigData("power_multiplier")
if multiplier == nil then
    multiplier = 1
end

MOD_OPTIONS = {
    enabled = enabled,
    power_multiplier = multiplier,
}
```

`false` 是合法保存值，不能写 `GetModConfigData("feature_enabled") or true`。读取实现用 `~= nil` 区分缺失；常规表按保存的服务端／客户端值、`saved`、`default` 选择，临时配置表直接按键返回。[modutil.lua:34](../../../../raw/current/scripts/modutil.lua#L34

本地 UI 偏好可用包装器 `GetModConfigData("ui_option", true)`；游戏规则使用正常配置。`require` 的文件不保证处于这个模组环境，优先传入配置表；Prefab／组件等在游戏全局环境调用 `GetModConfigData(key, actual_modname, get_local_config)`。入口内需要绕过包装器时才用 `GLOBAL.GetModConfigData` 的同一全局签名；`actual_modname` 在入口取得并传入，不能填展示名或猜一个工坊 ID。不要把入口包装器的第二参误当 `modname`。

### 本地开发名称与多语言

替换 `modinfo.lua` 的 `name` 赋值；其他基本字段继续保留。将 `mod_starter_dev` 换成自己约定的本地开发目录名；普通发布目录不附加开发后缀。

```lua
local base_name = ChooseTranslationTable({
    "Mod Starter",
    zh = "模组示例",
    zht = "模組示例",
    zhr = "模组示例",
})

name = base_name
if folder_name == "mod_starter_dev" then
    name = name .. " [本地开发]"
end
```

`ChooseTranslationTable` 取当前 `locale` 的项，否则取数组第一项；当前快照 zh／zht／zhr 对应三项中文语言代码。`label`、`description` 等可用同一方法按需翻译。[modindex.lua:603](../../../../raw/current/scripts/modindex.lua#L603)、[loc.lua:15](../../../../raw/current/scripts/languages/loc.lua#L15

`folder_name` 来自实际模组标识；Steam 工坊命名以 `workshop-` 开头，但这只是一种目录命名判断。它不能证明当前联网、正式发布、订阅来源或跨平台身份；优先用明确开发目录白名单。变更 `name` 只改展示文字，不改变文件夹／`modname`；从展示名反查实际标识的接口按字符串相等匹配，不适合当稳定依赖 ID。[注入](../../../../raw/current/scripts/modindex.lua#L601)、[工坊判定](../../../../raw/current/scripts/modindex.lua#L230)、[展示名反查](../../../../raw/current/scripts/modindex.lua#L691)

### 客户端字符串选项

在 `modinfo.lua` 的已有配置表之后加入以下片段，追加一个条目；没有配置表时先建立空表。`client` 仅控制界面筛选。本例保存 UI 风格偏好，消费者需要另行实现样式。

```lua
configuration_options = configuration_options or {}
configuration_options[#configuration_options + 1] = {
    name = "ui_style",
    label = "界面风格",
    hover = "仅用于本地界面",
    client = true,
    options = {
        {description = "简洁", data = "compact", hover = "较少信息"},
        {description = "详细", data = "full", hover = "较多信息"},
    },
    default = "compact",
}
```

在 `modmain.lua` 读取 `local ui_style = GetModConfigData("ui_style", true)`，对 nil 使用 `"compact"`；true 请求本地值，不是发送配置给服务器。字符串／布尔／数值示例共同覆盖常用值类型；消费方仍要检查来自保存文件的类型和取值。[本地读取:34](../../../../raw/current/scripts/modutil.lua#L34)

## 常见陷阱与限制

- **配置没出现**：确认是顺序数组，有 `name`、`options`、非 nil 的默认／保存值，并检查 `client` 与所开的配置界面是否匹配；`default = false` 合法。[界面:21](../../../../raw/current/scripts/screens/redux/modconfigurationscreen.lua#L21)
- **标题显示成下拉框**：单个选项的 `description` 必须是空字符串；一个空格也不满足标题判断。[界面:168](../../../../raw/current/scripts/screens/redux/modconfigurationscreen.lua#L168)
- **改默认值后还是旧值**：已有保存值优先；改 `name` 会改变匹配键，改 `data` 类型不自动迁移旧值。按需要保留键、重置配置或明确处理迁移，不能静默把数值字符串当 number。[modindex.lua:764](../../../../raw/current/scripts/modindex.lua#L764)、[modutil.lua:45](../../../../raw/current/scripts/modutil.lua#L45
- **把配置 UI 当作参数校验**：界面选项不保证磁盘／服务器覆盖值只在列表内；涉及数值范围、类型或执行权限时由实际消费者检查。本页读取 demo 只处理 nil，未提供完整业务校验。[读取实现](../../../../raw/current/scripts/modutil.lua#L34)

## 验证记录

### 版本增量复核 26148036（2026-10-25）
处理范围：锚点重定位 8 处（modindex.lua 全部 +6：初始化环境／保存匹配／读取选择／名称反查等；mods.lua 环境与 modimport -39）；R43 核验：本页未引用 Release ID／`CurrentRelease` 门控，R-11 注册表改版不影响配置读取结论，modconfigurationscreen／modutil／loc 未变。依据：全局枢纽 diff 审阅＋变更清单命中 modindex.lua、mods.lua、util.lua（本版仅 +1 行）。

2026-10-11，Steam buildid 25643519：已审阅元信息环境、配置全部声明字段、界面筛选／说明／标题／保存、按名恢复与读取、工坊命名和语言选择。四个 Lua 片段经 Full Moon 3.0.0／Lua 5.1 语法与原字节往返检查；结构、链接、登记和章节读取另行检查。**未运行 DST**；配置 UI、保存重载、客户端／服务器／分片一致性及跨平台名称表现仍需游戏验证。
