# ItemCore

> 版本: v1.0.8 | Paper 1.21.x | Java 21 | 可选依赖: PlaceholderAPI、BetterHud

---

ItemCore 是一套面向 Paper 1.21.x 的自定义物品系统插件。核心只负责物品定义、属性计算、伤害系统、元素系统、套装、Lore 展示与物品库 GUI，其余玩法通过拓展插件接入：

| 插件 | 用途 |
|------|------|
| **ItemCore** | 核心插件：物品管理、属性计算、伤害系统、元素系统、套装、GUI |
| **ItemCoreMythic** | MythicMobs 桥接：物品技能触发、`icdamage` / `icheal` 技能机制 |
| **ItemCoreRPG** | 伤害飘字、玩家属性面板（`/icrpg info`） |
| **ItemCoreForge** | 锻造 / 合成拓展 |
| **ItemCoreTrinkets** | 饰品栏拓展 |
| **ItemCoreLevel** | 玩家等级拓展 |
| **ItemCoreEnhance** | 强化 / 镶嵌拓展 |
| **ItemCoreTitles** | 称号拓展 |
| **ItemCoreQuest** | 任务拓展 |

## 核心特性

- 25 种自定义属性（攻击、防御、暴击、穿透、法力等），支持固定值、随机范围与随机词条
- 品质（固定 / 随机权重）、套装、自定义耐久、物品绑定
- 3 种默认元素类型（流火 / 寒霜 / 雷蛰），框架级可扩展
- 自定义伤害计算系统（物理 / 法术 / 射弹 + 元素混合伤害）
- 元素积累 / 异常机制（灼烧 DOT、寒霜减双抗、雷蛰易伤）
- 法力值系统（可选开启，支持 BetterHud 法力不足提示）
- 自动 Lore 生成（可配置布局、条件区块、附属扩展区块）
- 物品库 GUI（分类浏览、获取物品、随机属性 / 品质预览）
- PlaceholderAPI 集成（属性、法力、耐久、装备占位符）
- 完整的附属开发 API（属性提供器、Lore 提供器、分类行为系统等）
- 热重载：修改配置后 `/ic reload` 即可生效

## 开始使用

- [安装与配置](getting-started.md)
- [命令系统](commands.md)
- [物品创建](items.md)
- [属性系统](attributes.md)
- [元素系统](elements.md)

## 项目地址

[GitHub - honzhi/ItemCore](https://github.com/honzhi/ItemCore)
