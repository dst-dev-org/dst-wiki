---
title: "curseditem"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "诅咒物品寻人、追逐／强制归还和拾取施咒；active 标签、已绑定主人与新追踪目标分开。"
aliases: ["猴子代币再次拾取静默", "诅咒代币追人", "诅咒物品绑定主人", "curseditem"]
tags: [component, item, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/curseditem.lua", "components/cursable.lua", "components/stackable.lua", "prefabs/cursed_monkey_token.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# curseditem

## 快速判断

- 职责与适用：原猴子代币物品侧寻找玩家、移动靠近、强制归还已绑定主人，并在 onpickup 首次调用 [[reference/components/cursable]]。不是任意 Buff 物品的轻量标记。[初始化](../../../raw/current/scripts/components/curseditem.lua#L16-L35) [更新](../../../raw/current/scripts/components/curseditem.lua#L120) [拾取](../../../raw/current/scripts/components/curseditem.lua#L166-L178)
- 优先使用：只改确切环节，保留原 Given／玩家计数链；解除通过玩家 RemoveCurse，不把删除组件／清 target 当作净化。[归属检查](../../../raw/current/scripts/components/curseditem.lua#L105-L114) [解除](../../../raw/current/scripts/components/cursable.lua#L26-L53)
- 重要副作用：cursed_target 已存在时可强制回库存、挤掉别的物品；未绑定时周期找附近玩家并逐帧移动。物品原宿主另配漂浮、库存和堆叠，不能把此组件随便挂到静态建筑。[寻找](../../../raw/current/scripts/components/curseditem.lua#L78-L100) [收纳](../../../raw/current/scripts/components/cursable.lua#L85-L120) [宿主](../../../raw/current/scripts/prefabs/cursed_monkey_token.lua#L91-L104)
- 修改边界：服务端；active 在组件内部只维护 cursed_inventory_item 标签，OnUpdate／Given 不以它作门禁，设 false 不等于停追／停止施咒。[属性](../../../raw/current/scripts/components/curseditem.lua#L1-L6) [更新与拾取](../../../raw/current/scripts/components/curseditem.lua#L120)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `curse`／`cursed_target`／`target` | curse 由原代币设 MONKEY；cursed_target 是已绑定主人，target 是待追玩家。玩家 ApplyCurse 写前者，Given 清后者，二者不能互当同义字段。[宿主](../../../raw/current/scripts/prefabs/cursed_monkey_token.lua#L100-L101) [施加](../../../raw/current/scripts/components/cursable.lua#L8-L23) [拾取](../../../raw/current/scripts/components/curseditem.lua#L166-L171) |
| `lookforplayer()` | 原拼写小写；取消旧 inst.findplayertask，再每秒选十单位内最近可见候选，校验空间／cursable／出生保护后记 target。最近对象不合格就置 nil，此次不改选第二名。[方法](../../../raw/current/scripts/components/curseditem.lua#L78-L102) |
| `checkplayersinventoryforspace(player)` | 扫主栏空位，考虑同名手持与未满堆，再考虑可挤掉的直接物品；它不是“背包还有空位”或绝不丢物证明。[方法](../../../raw/current/scripts/components/curseditem.lua#L38-L75) |
| `CheckForOwner()` | 绑定者死亡则清 applied_curse／cursed_target；仍绑定且在场景或直接 owner 不同则 ForceOntoOwner。开头访问既有绑定者，不能拿 nil 或过期宿主调用作查询。[方法](../../../raw/current/scripts/components/curseditem.lua#L105-L114) |
| `OnUpdate(dt)` | 先处理绑定，未绑定才追 target；小于二单位时强制交给玩家，否则按距离／dt 移动并更新 floater。无可用 target 时排后续寻找；不是引擎自动寻路。[方法](../../../raw/current/scripts/components/curseditem.lua#L120-L161) |
| `Given(item,data)` | onpickup 的 data.owner 有 cursable 且无 applied_curse 时 ApplyCurse；已有标签时可排 0.5 秒台词，受 item.skipspeech 和届时 activeitem 位置影响。不自检 active 或完整 IsCursable 资格。[方法](../../../raw/current/scripts/components/curseditem.lua#L166-L178) |
| `CopyCursedFields(to)` | 实例上存函数(from,to)，冒号调用时 self 是 from；只复制 active／cursed_target／target，不含 curse 或 applied_curse 标签。原拆堆另补标签与 skipspeech。[复制](../../../raw/current/scripts/components/curseditem.lua#L9-L22) [拆堆](../../../raw/current/scripts/components/stackable.lua#L126-L130) |

### 更新、保存与宿主职责

构造即 StartUpdatingComponent，entitysleep／entitywake 分别停／启；寻找任务存在 inst.findplayertask 上，不能把停止逐帧更新当作完整取消所有任务。组件无自己的存读和移除清理接口；原代币宿主保存的是图标变体，另设侵蚀 timer，拾取时停侵蚀，超时停更新再 ErodeAway。别仅由这些接口推断所有离屏／持有状态。[组件初始化](../../../raw/current/scripts/components/curseditem.lua#L16-L35) [原保存](../../../raw/current/scripts/prefabs/cursed_monkey_token.lua#L37-L52) [原计时](../../../raw/current/scripts/prefabs/cursed_monkey_token.lua#L103-L116)

## 典型接入

### 原猴子代币再次拾取时静默，保留第一次施咒

目标模组 `modmain.lua`，无新资源；包装实际 Given 方法，仅在调用期间令 skipspeech 为真。保留原 onpickup 监听及 ApplyCurse 门禁，不复制计数、堆叠或寻人逻辑。[原监听](../../../raw/current/scripts/components/curseditem.lua#L24-L31) [台词门禁](../../../raw/current/scripts/components/curseditem.lua#L166-L178)

```lua
AddPrefabPostInit("cursed_monkey_token", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local cursed = inst.components.curseditem
    if cursed == nil then return end
    local old = cursed.Given
    cursed.Given = function(self, item, data)
        local previous = item.skipspeech
        item.skipspeech = true
        old(self, item, data)
        if item:IsValid() then item.skipspeech = previous end
    end
end)
```

它抑制的是已有 applied_curse 分支的新拾取台词任务；首次 MONKEY 工具可能安排的猴化提示不在此范围。原方法没有返回值，包装不新增“成功”协议；恢复原字段，避免把标记永久留给后续堆叠／其他业务。[Given](../../../raw/current/scripts/components/curseditem.lua#L166-L178)

不取消安装前已排台词、不改变原诅咒计数、目标追逐或强制归还；停用并重载后恢复原方法，无新存档字段。它不是免疫诅咒／停止追踪案例，若要改变这些行为应审对应玩家和物品入口。

## 常见陷阱与限制

- **active=false 就不动／不施咒**：当前组件方法不据此拒绝，标签可能由其他系统消费，不能把标签状态当整条更新协议。[属性](../../../raw/current/scripts/components/curseditem.lua#L1-L6) [追逐](../../../raw/current/scripts/components/curseditem.lua#L120-L161) [拾取](../../../raw/current/scripts/components/curseditem.lua#L166-L178)
- **手动清 target 就脱离主人**：cursed_target 仍优先执行归还；删除物品也不自动按数量调用玩家 RemoveCurse。净化见 [[reference/components/cursable#典型接入]]。[更新顺序](../../../raw/current/scripts/components/curseditem.lua#L120-L131)
- **拆堆自动保留任意自定义诅咒字段**：原复制只含三项，且已施咒标签另行处理；新增字段需审实际消费者，不把 copied refs 当完整保存／跨分片恢复。[复制](../../../raw/current/scripts/components/curseditem.lua#L9-L22) [消费者](../../../raw/current/scripts/components/stackable.lua#L126-L130)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖完整 curseditem、玩家 ApplyCurse／强制收纳／解除、原代币图标保存与侵蚀／拾取、拆堆字段和标签复制。猴化工具效果复用同批 cursable 的核验，不重复展开全部变身链。未全面审校物品睡醒任务、堆叠合并、变身 SG、跨分片或坏主人引用。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整静默包装经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；首次／重复拾取、整堆拆分、原台词任务、满库存、主人死亡／离线、掉落追逐和侵蚀待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。