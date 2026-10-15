# 明细：本地化与语音（languages/*、strings.lua、speech_*）

资源文案类，按报告结构一笔带过：不逐条展开，登记变化量与代表样例；条目编号本报告内稳定唯一。

| 编号 | 变化 | 影响接入／条件 | 证据位置 | 核验状态 |
|---|---|---|---|---|
| M-1 | `strings.lua`：新增官方名 `FRANKEN_PIG="Frankenswine"`、`MUMMY_MERM="Mermummy"`、`VAMPIRE_BUNNY="Vamphare"` 与装饰 7–10 名（Frankenswine/Mermummy/Vamphare/Hound Batilisk Decoration）；`HALLOWEEN_ORNAMENT_2` 描述 "Bat Decoration"→"Batilisk Decoration"；`UPSELL_TEXT` 断行调整；`VAULT_ORB_REFINED` 描述 "Telelocater"拼写→"Telelocator"。 | 三怪物与四装饰的英文官方名来源（简中见 M-4）；模组对表用这些键。 | 旧:strings.lua 无；新:strings.lua STRINGS.NAMES 段（+8 行） | 已核验 |
| M-2 | `speech_*.lua` 17 个角色语音各新增 6–7 条 describe：三怪物（FRANKEN_PIG／MUMMY_MERM／VAMPIRE_BUNNY）与四装饰（ORNAMENT_7–10）；wilson 另有少量条目微调（净 7 行）。 | 语音键按 STRINGS.NAMES 同名接入；模组自定义角色不写新键会回退 GENERIC。 | 旧:speech_wilson.lua 无对应键；新:17 个 speech_*.lua DESCRIBE 段（每文件 +7~13 行） | 已核验 |
| M-3 | `languages/strings.pot` 净 +630 行：新怪物／装饰／语音键模板同步（与 M-1/M-2 同源）。 | 翻译工具链模板更新；无运行时影响。 | 旧:languages/strings.pot；新:languages/strings.pot | 已核验 |
| M-4 | `languages/chinese_s.po`／`chinese_t.po` 各净 +630 行：三怪物简中官方名 **科学怪猪／鱼人木乃伊／吸血兔**，装饰 7–10 **科学怪猪装饰／鱼人木乃伊装饰／吸血兔装饰／猎犬蝙蝠装饰**，及各角色台词翻译；繁中同步。 | 简中显示名核验依据（本报告 P-1／P-3／P-7／P-8 引用）。 | 旧:languages/chinese_s.po；新:languages/chinese_s.po（NAMES.FRANKEN_PIG 等 msgstr） | 已核验 |
