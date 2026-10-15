---
title: 新增可拾取物品
created: 2026-10-11
updated: "2026-10-28"
type: cookbook
summary: 在已有世界模组中注册一个可生成、检查与拾取的新物品，复用原版动画和图标，再按需组合能力
aliases: [新道具, 新物品, 可拾取物品骨架, 物品 Prefab, new-item, PrefabFiles, dstwiki_token]
tags: [cookbook, item, prefab, modding, server-client]
dst_build: "steam-buildid:26230496"
sources: [mods.lua, mainfunctions.lua, prefabs.lua, prefabs/flint.lua, prefabs/global.lua, standardcomponents.lua, entityscript.lua, entityreplica.lua, stringutil.lua, components/inventoryitem.lua, components/inventoryitem_replica.lua, simutil.lua, util.lua, consolecommands.lua]
reliability: verified
validation: source-reviewed
status: active
---

# 新增可拾取物品

## 适用条件

在**已有 DST 世界／服务器模组**中新增普通物品：地面可生成和检查，能够拾取、放入人物物品栏或普通容器、再丢落。尚无入口时先从 [[cookbook/index|场景导航]] 读取模组起步；已有项目沿用自己的注册与目录约定。

本 demo 使用独立 Prefab `dstwiki_token`，复用原版燧石的外观与图标，无需制作或复制美术资源；它没有燧石的食用、交易、堆叠等能力。新增玩法按下文链接组合，修改已有物品直接查本地实现与机制页。

## 最小 demo

### 文件与加载

目标模组根目录保留已有 `modinfo.lua`、`modmain.lua`，新增一个 Prefab 文件：

```text
<目标模组>/
├── modinfo.lua
├── modmain.lua
└── scripts/
    └── prefabs/
        └── dstwiki_token.lua
```

在已有 `modinfo.lua` 中采用以下运行端设置，其余元信息保留：

```lua
client_only_mod = false
all_clients_require_mod = true
```

新增网络 Prefab 需要服务器和客户端加载对应工厂定义；复用原版贴图不取消代码依赖。运行端标志含义见 [[reference/api/modinfo/fields#运行端与用户配置]]；本地开发测试时各端需安装同一版本。

### modmain.lua 注册

空功能入口可以使用以下完整内容；已有入口只合并新条目与字符串，保留原有代码：

```lua
PrefabFiles = {
    "dstwiki_token",
}

GLOBAL.STRINGS.NAMES.DSTWIKI_TOKEN = "示例石片"
GLOBAL.STRINGS.CHARACTERS.GENERIC.DESCRIBE.DSTWIKI_TOKEN =
    "一块可以随身携带的石片。"
```

`PrefabFiles` 填 `scripts/prefabs/` 下的**文件名，不带 `.lua`**；加载器读取文件，再登记返回的 `Prefab`。[mods.lua:654](../../raw/current/scripts/mods.lua#L654)、[mainfunctions.lua:171](../../raw/current/scripts/mainfunctions.lua#L171)。`modmain` 的 `GLOBAL` 指向游戏全局；这里显式访问，不要求重复添加全局读取代理。[mods.lua:289](../../raw/current/scripts/mods.lua#L289

展示名称与检查语使用注册名的大写键；`GENERIC` 是普通角色缺少专属台词时的回退。[entityscript.lua:840](../../raw/current/scripts/entityscript.lua#L840)、[stringutil.lua:396](../../raw/current/scripts/stringutil.lua#L396

### 物品 Prefab

目标文件 `scripts/prefabs/dstwiki_token.lua`，完整内容：

```lua
local assets = {
    Asset("ANIM", "anim/flint.zip"),
}

local function fn()
    local inst = CreateEntity()

    inst.entity:AddTransform()
    inst.entity:AddAnimState()
    inst.entity:AddSoundEmitter()
    inst.entity:AddNetwork()

    MakeInventoryPhysics(inst)

    inst.AnimState:SetRayTestOnBB(true)
    inst.AnimState:SetBank("flint")
    inst.AnimState:SetBuild("flint")
    inst.AnimState:PlayAnimation("idle")
    inst.pickupsound = "rock"

    inst.entity:SetPristine()

    if not TheWorld.ismastersim then
        return inst
    end

    inst:AddComponent("inspectable")
    inst:AddComponent("inventoryitem")
    inst.components.inventoryitem:ChangeImageName("flint")

    return inst
end

return Prefab("dstwiki_token", fn, assets)
```

共同的实体、物理与动画初始化放 `SetPristine()` 前；保持原版 `SetPristine → 客户端返回 → 服务端组件` 顺序。`inspectable` 提供检查，`inventoryitem` 接入拾取／收纳，客户端由游戏复制体系建立相应 replica。[flint.lua:15](../../raw/current/scripts/prefabs/flint.lua#L15)、[standardcomponents.lua:371](../../raw/current/scripts/standardcomponents.lua#L371)、[entityscript.lua:653](../../raw/current/scripts/entityscript.lua#L653)、[entityreplica.lua:34](../../raw/current/scripts/entityreplica.lua#L34

**资源与替换点**：

- `anim/flint.zip` 由当前 DST 安装提供；bank、build 为 `flint`，动画为 `idle`，沿用原版配置。Prefab 的 `assets` 声明该动画，解析资源路径时可回退游戏目录。[flint.lua:1](../../raw/current/scripts/prefabs/flint.lua#L1)、[util.lua:587](../../raw/current/scripts/util.lua#L587
- `ChangeImageName("flint")` 不带 `.tex`；本例保留默认 `atlasname = nil`，replica 根据 `flint.tex` 查原版图集。图集／纹理由游戏全局资源加载，不需在模组里增加 XML／TEX。[inventoryitem.lua:470](../../raw/current/scripts/components/inventoryitem.lua#L470)、[inventoryitem_replica.lua:131](../../raw/current/scripts/components/inventoryitem_replica.lua#L131)、[simutil.lua:675](../../raw/current/scripts/simutil.lua#L675)、[global.lua:476](../../raw/current/scripts/prefabs/global.lua#L476
- 更换 Prefab 标识时，统一替换文件名、`PrefabFiles` 条目、`Prefab("dstwiki_token", ...)` 与字符串大写键；仅改展示名称则只改 `STRINGS.NAMES` 的文字。文件名与注册名是两种概念，本例刻意同名。`Prefab` 第三个参数是资源表，第四个才是依赖的 Prefab 名列表。[prefabs.lua:6](../../raw/current/scripts/prefabs.lua#L6
- 换原创外观时需实际取得动画 ZIP 及正确 bank／build／动画；换原创图标需匹配 XML、TEX 与图元名，并按 [[reference/components/inventoryitem#关键接口|图标字段]] 配置。美术制作和格式转换不在本骨架中展开。

### 验证物品

在隔离的测试世界启用该模组，重新加载；服务端控制台（远端玩家需具备权限并使用远程执行）生成：

```lua
c_spawn("dstwiki_token")
```

预期检查：地面可见燧石外观与“示例石片”名称；普通角色可检查并读取本例描述；拾取后显示燧石图标，能放入普通箱子再取出、丢落。多个实例各占一格，本例没有堆叠组件。再验证保存／重载和第二位客户端看到、拾取同一对象；这些是**待执行步骤**。[consolecommands.lua:200](../../raw/current/scripts/consolecommands.lua#L200)、[:208](../../raw/current/scripts/consolecommands.lua#L208)、[util.lua:20](../../raw/current/scripts/util.lua#L20

## 按需求扩展

只读取当前功能所需章节；能力代码加入服务端分支时，仍按对应页核对共同初始化、网络数据与清理要求。

| 新需求 | 读取与组装方向 |
|---|---|
| 堆叠与拆分 | [[reference/components/stackable#关键接口|stackable]]；数量状态与拆分继承按该页处理 |
| 特殊收纳或随身效果 | [[reference/components/inventoryitem#关键接口|限制／持有者]]、[[reference/components/inventoryitem#生命周期|归属顺序]]；需要人物主栏限定时再读 [[reference/components/inventory#携带范围的选择|携带范围]] |
| 可装备、工具或武器／护甲 | 新增手持物用 [[cookbook/equipment/hand#最小-demo|手部骨架]]，其他装备从 [[cookbook/equipment/index|装备导航]] 判断覆盖；已有外观骨架只补 [[reference/components/equippable#关键接口|装备入口]]，按功能读 [[reference/components/tool#关键接口|工具动作]]、[[reference/components/weapon#关键接口|攻击]] 或 [[reference/components/armor#关键接口|减伤]] |
| 自带容器格子 | [[reference/components/container#新容器的客户端布局|双端布局]]、[[reference/components/container#可拾取且未装备的容器|便携容器]]；物品自身可收纳与内部存物分开 |
| 可食用与会腐败 | [[reference/components/edible#关键接口|食物类型／收益]]；需要腐败再读 [[reference/components/perishable#关键接口|保质期／变质]] |

本例的取得方式是控制台生成；注册 Prefab 不会自动添加制作配方、世界刷新或掉落来源。添加这些来源时另查对应接口／原版调用，不把全部取得方式塞进物品工厂。

## 边界与验证

- **已有工程直接覆盖 `PrefabFiles`**：会丢失原注册条目；合并数组并保留已有入口。加载器只遍历该数组中的文件；外观资源名也不会替你注册 Prefab。[mods.lua:654](../../raw/current/scripts/mods.lua#L654
- **复制燧石外观后默认继承功能**：工厂独立，只拥有本例加入的组件；如需堆叠、食用或交易再显式接入。原版这些能力由各自组件建立。[flint.lua:42](../../raw/current/scripts/prefabs/flint.lua#L42
- **注册名改了，图标仍用默认回退**：新名字会查 `<prefab>.tex`，游戏没有该图元；保留本例 `ChangeImageName("flint")` 或提供匹配原创图标。[inventoryitem_replica.lua:151](../../raw/current/scripts/components/inventoryitem_replica.lua#L151

### 版本增量复核 26148036（2026-10-25）
处理范围：锚点重定位 7 处（mods.lua -39 两处、mainfunctions +26、entityscript SetPrefabName +4、inventoryitem ChangeImageName +7、simutil +5、global.lua 图集段 +7）；无结论修订，仅锚点迁移——entityscript 本版改动在湿润查询／距离函数，不涉 SetPrefabName。依据：全局枢纽 diff 审阅＋变更清单命中 mods.lua、mainfunctions.lua、entityscript.lua、inventoryitem.lua、simutil.lua、prefabs/global.lua、util.lua（本版仅 +1 行）。

验证记录（2026-10-11，Steam buildid 25643519）：已审阅 Prefab 注册／生成、原版燧石初始化、物理辅助、名称／检查回退、拾取图标的组件／replica 与资源解析。只读检查安装资源确认 `anim/flint.zip` 存在，当前 `images/inventoryimages2.xml` 含 `flint.tex` 且对应纹理存在；示例通过查找函数选图集，不硬编码其编号。

四个 Lua 代码块经 Full Moon 3.0.0／Lua 5.1 语法与原字节往返检查；页面、来源路径、链接／锚点、大小、所属导航登记及实际检索／章节读取检查通过。**未启动 DST**：生成、拾取／容器交互、远端显示、存档重载及模组兼容未实测；资源解析与 `SetPristine` 的引擎内部仅按原版调用模式使用。本骨架限定普通地面物品，海上漂浮／沉没和新增效果的清理另需核验。
