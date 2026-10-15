---
title: "bundlemaker"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "配置包装材料使用的临时容器与成品，并在开始打包时消费材料。"
aliases: ["包装材料", "打包袋改礼物", "开始打包", "bundlemaker"]
tags: [component, item, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/bundlemaker.lua", "components/bundler.lua", "prefabs/bundle.lua", "actions.lua", "components/stackable.lua", "componentactions.lua", "modutil.lua", "prefablist.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# bundlemaker

## 快速判断

- 职责与适用：挂在空包装材料（bundlewrap／giftwrap）上，指定临时容器和成品；人物会话由 [[reference/components/bundler]] 管，成品内容由 [[reference/components/unwrappable]] 管。[配置](../../../raw/current/scripts/components/bundlemaker.lua#L1-L10)／[原版装配](../../../raw/current/scripts/prefabs/bundle.lua#L52-L54)
- 优先使用：`SetBundlingPrefabs(临时容器名, 成品名)`；保留开始回调的单件材料消费，不自行重复扣除。[原版消费](../../../raw/current/scripts/prefabs/bundle.lua#L3-L5)
- 重要副作用：本组件只保存配置和转发回调，不生成容器、不消费物品。人物成功打开容器、登记材料信息后才调用开始回调。[实际调用](../../../raw/current/scripts/components/bundler.lua#L26-L41)
- 修改边界：服务端配置；普通材料操作沿用 BUNDLE 动作与人物 SG，不直接调用 OnStartBundling 当完整打包。[动作](../../../raw/current/scripts/actions.lua#L4347-L4357)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `SetBundlingPrefabs(bundling, bundled)` | 两个 Prefab 名 | 前者需可被人物打开的 container，后者需 unwrappable；两字段默认 nil。[设置](../../../raw/current/scripts/components/bundlemaker.lua#L8-L10)／[消费者](../../../raw/current/scripts/components/bundler.lua#L20-L41)／[成品要求](../../../raw/current/scripts/components/bundler.lua#L114-L124) |
| `SetSkinData(skinname, skin_id)` | 成品皮肤数据 | 赋 bundledskinname／bundledskin_id，供人物复制并生成成品；不等于验证皮肤授权或资源兼容。[设置](../../../raw/current/scripts/components/bundlemaker.lua#L13-L15)／[取用](../../../raw/current/scripts/components/bundler.lua#L35-L37) |
| `SetOnStartBundlingFn(fn)` | `fn(material, doer)` | 单回调槽；此时容器已打开，原版回调取堆叠单件并删除，随后 SG 转入 bundling。回调可能删除 material。[设置](../../../raw/current/scripts/components/bundlemaker.lua#L18-L25)／[调用顺序](../../../raw/current/scripts/components/bundler.lua#L33-L41) |

## 典型接入

### 原版打包袋改为产出礼物

目标模组 `modmain.lua`；使用原版 bundlewrap 的动画／收纳与消费回调，临时容器仍为四格 bundle_container，成品改成 gift。原版同一 bundle 文件注册两套包装物，且属于基础 Prefab 列表，无需新增资源。[注册](../../../raw/current/scripts/prefabs/bundle.lua#L672-L682)／[基础加载](../../../raw/current/scripts/prefablist.lua#L186)／[PostInit](../../../raw/current/scripts/modutil.lua#L593)

```lua
AddPrefabPostInit("bundlewrap", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.bundlemaker ~= nil then
        inst.components.bundlemaker:SetBundlingPrefabs(
            "bundle_container", "gift")
    end
end)
```

不替换开始回调，仍消费一张打包袋；取消时人物按原始材料名返还 bundlewrap，不按 giftwrap 返还。已开始的会话复制了成品名，不随事后改材料字段更新。[复制](../../../raw/current/scripts/components/bundler.lua#L33-L38)／[取消返还](../../../raw/current/scripts/components/bundler.lua#L80-L92)

成品遵循 gift 的行为：默认不允许预览，也不沿用普通 bundle 拆出 waxpaper 的额外掉落；不要把改产物当作只换皮肤。停用后新建／读档材料恢复原配置，既有礼物不转回包裹。[礼物设置](../../../raw/current/scripts/prefabs/bundle.lua#L408-L417)／[两种成品注册](../../../raw/current/scripts/prefabs/bundle.lua#L678-L682)

## 常见陷阱与限制

- **开始回调不扣材料也能免费包装：** 正常取消流程还会重建一份材料；取消返还与开始消费是配套约定，不能只删除消费。原版 `stackable:Get()` 默认取一件，详见 [[reference/components/stackable]]。[单件](../../../raw/current/scripts/components/stackable.lua#L109-L116)／[原版消费](../../../raw/current/scripts/prefabs/bundle.lua#L3-L5)／[返还](../../../raw/current/scripts/components/bundler.lua#L80-L89)
- **任意箱子都能作为临时容器：** 需要 container、允许人物打开，还要布局／过滤／完成按钮配合；原版临时实体不独立持久化，布局见 [[reference/components/container]]。[临时工厂](../../../raw/current/scripts/prefabs/bundle.lua#L76-L105)／[打开验证](../../../raw/current/scripts/components/bundler.lua#L26-L45)
- **添加同名标签代替组件：** BUNDLE 动作来自 bundlemaker 的动作收集，服务端还读真实组件字段；沿用已注册组件与双端实体约定，不凭标签造出完整功能。[动作收集](../../../raw/current/scripts/componentactions.lua#L242-L245)／[服务端](../../../raw/current/scripts/components/bundler.lua#L20-L24)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的漂移文件（actions.lua、componentactions.lua、prefablist.lua）对照 build-26148036 更新报告（R-01/R-04、prefablist 新增）逐条核验：BUNDLE 动作实现与 bundlemaker 候选收集逐行未变，prefablist.lua 仅追加 Rifts8 新 prefab 文件（"bundle" 等既有注册不动，整体下移 6 行）；锚点重定位 2 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖配置／回调、原版包装材料与成品注册、开始消费和取消重建、示例资源与礼物行为差异；未核验全部皮肤、活动包装与模组材料保存。
- 结构／示例检查：本批格式、来源文件、链接／行锚点、导航登记与完整示例 Lua 5.1 语法检查通过。
- 游戏运行：未运行；开始／取消／完成、堆叠取件、皮肤、联网与其他模组覆盖需验证。
