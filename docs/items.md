# 物品创建

## 基本格式

在 `items/` 目录下的 `.yml` 文件中定义：

```yaml
legendary_blade:
  material: NETHERITE_SWORD
  display_type: '&6传说武器'
  rank: legendary
  display_name: "&4&l传说之刃"
  lore:
    - "&7传说中的神秘武器"
  attributes:
    ATTACK_DAMAGE: 15
    CRIT_CHANCE: 25
  active_slots:
    - main_hand
```

## 物品身份

每件物品的完整身份由“分类 ID + 物品 ID”组成，格式为：

```text
分类ID:物品ID
```

例如 `melee_weapons:legendary_blade`。物品所属分类由 `categories.yml` 中的
`items_file` 决定，因此不同分类可以使用相同的物品 ID：

```yaml
# items/gems.yml，对应 gems 分类
life_gem:
  material: PAPER
  display_name: "&c生命宝石"

# items/quest_items.yml，对应 quest_items 分类
life_gem:
  material: PAPER
  display_name: "&e任务生命宝石"
```

这两件物品的完整 ID 分别为 `gems:life_gem` 和 `quest_items:life_gem`。
分类 ID 和物品 ID 不区分大小写，但不能包含冒号 `:`。

- 命令固定使用两个参数：`/ic give 玩家 分类ID 物品ID`。
- API 标准写法是分别传入分类 ID 和物品 ID，或使用完整 ID。
- 裸物品 ID 只作为旧附属兼容入口，不参与管理指令解析。
- 新生成物品会在 PDC 中同时保存完整 ID 与本地物品 ID。
- 旧版本物品只有本地 ID 时，会按照旧版最后加载生效的定义自动迁移为完整 ID。

## 完整配置项

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `material` | 材质名 | **必填** | 物品材质 |
| `display_type` | 字符串 | 所在分类 ID | 仅作为类型 Lore 文本；可用 `#type#` 显示，不决定物品行为 |
| `rank` | 品质 ID、随机模板 ID 或旧版文本 | — | 引用 `ranks.yml` 的固定品质或随机模板；可用 `#rank#` 显示 |
| `type_settings` | 字符串、列表或 Map | — | 分类行为系统专属配置；核心仅保存，由对应系统读取 |
| `display_name` | 字符串 | — | 显示名称（支持 `&` 颜色码） |
| `color` | RGB | — | 皮革护甲或药水颜色，例如 `255, 0, 0` |
| `set` | 字符串 | — | 所属套装 ID，对应 `sets.yml` 中的配置 |
| `lore` | 列表 | — | 物品描述 |
| `attributes` | Map | — | 属性配置 |
| `random_attributes` | Map | — | 生成时从属性池中随机抽取额外属性 |
| `enchantments` | Map | — | 附魔：`sharpness: 5` |
| `item_flags` | 列表 | — | ItemFlag：`HIDE_ATTRIBUTES` |
| `unbreakable` | 布尔 | false | 原版不可破坏 |
| `enchantment_glint` | 布尔 | false | 只显示附魔光效，不添加真实附魔 |
| `durability` | 整数 | — | 自定义最大耐久，详见下文 |
| `durability_break` | 布尔 | true | 耐久归零时是否销毁物品 |
| `max_stack` | 整数 | 原版规则 | 最大堆叠 |
| `custom_model_data` | 整数 | — | 自定义模型数据 |
| `active_slots` | 列表 | `any` | 生效装备位；`never` 表示任何槽位都不生效，且优先于其他槽位 |
| `skills` | Map | — | 技能配置（需 ItemCoreMythic） |
| `effects` | Map | — | 装备时给予的药水效果 |
| `keep_on_death` | 布尔 | false | 死亡保留 |
| `droppable` | 布尔 | true | 是否允许丢弃；为 false 时也不能从护甲栏取下 |
| `clickable` | 布尔 | true | 是否允许在背包中点击移动 |
| `right_clickable` | 布尔 | true | 是否允许右键交互（护甲右键穿戴不受限制） |
| `left_clickable` | 布尔 | true | 是否允许左键交互 |
| `disable_anvil` | 布尔 | 全局默认 | 禁止铁砧操作，旧键 `disable_anvil_repair` |
| `disable_enchant` | 布尔 | 全局默认 | 禁止附魔，旧键 `disable_enchanting` |
| `disable_crafting_table` | 布尔 | 全局默认 | 禁止作为原版合成材料，旧键 `block_crafting_ingredient` |
| `permission` | 字符串 | — | 使用权限；玩家缺少该权限时物品的属性、药水效果、技能、套装件数均不生效，也无法点击交互或从 GUI 获取 |
| `bind` | 布尔 | false | 物品绑定，别名 `soulbound`；开启后物品首次进入某个玩家背包时绑定给该玩家，详见下文 |

`item_flags` 常用值：`HIDE_ATTRIBUTES`（隐藏原版属性；武器带此标志时原版武器伤害不计入，伤害完全来自 `ATTACK_DAMAGE`）、`HIDE_ENCHANTS`、`HIDE_UNBREAKABLE`、`HIDE_POTION_EFFECTS`。

`active_slots` 支持 `main_hand`、`off_hand`、`head`、`chest`、`legs`、`feet`、
`trinkets`（需 ItemCoreTrinkets）、`any` 和 `never`。例如以下物品无论放在哪个槽位，都不会向玩家提供
属性、药水效果、技能或套装件数：

```yaml
active_slots:
  - never
```

损坏（耐久为 0）、无使用权限或非绑定主人持有的物品同样不会生效。

## 自定义耐久

```yaml
legendary_blade:
  material: NETHERITE_SWORD
  durability: 250          # 最大耐久
  durability_break: false  # 耐久归零后不销毁，只是失效
```

- 配置 `durability` 后，物品使用独立于原版的耐久系统，每次命中消耗 1 点。
- 耐久为 0 的物品不再提供属性、技能与套装件数；`durability_break: true`（默认）时直接销毁。
- 耐久 10 以下时随机播放警告音效。
- Lore 中通过 `#durability#` 显示，格式由 `stats.yml` 的 `durability` 控制（`{current}` `{max}` `{bar}`）。
- 附属可通过 API 修复、扣除或直接设置耐久。

## 物品全局保护

`config.yml` 的 `Item_global_settings` 提供 `disable_anvil`、`disable_enchant`、`disable_crafting_table` 三项默认值（均默认 `true`），物品配置中可用同名键覆盖：

```yaml
craft_material:
  material: PAPER
  disable_crafting_table: false   # 允许该物品参与合成
  disable_anvil: false            # 允许铁砧操作
  disable_enchant: false          # 允许附魔
```

## 物品绑定

```yaml
legendary_blade:
  material: NETHERITE_SWORD
  bind: true
```

`bind: true` 的物品在**首次进入某个玩家背包时**绑定给该玩家（获得即绑定），
无论物品来自 `/ic give`、物品库 GUI、拾取、任务奖励、商店还是其他附属插件。
绑定信息写入物品 PDC（`itemcore:bind_owner`），因此不同主人的同种物品不会堆叠。

绑定后的规则：

| 行为 | 结果 |
|------|------|
| 丢弃（Q 键、拖出背包） | 取消并提示 |
| 死亡 | 绑定物品保留在背包，不掉落（`config.yml` 的 `bind.keep_on_death`） |
| 其他玩家拾取 | 取消并提示；生物也无法拾取 |
| 放入箱子、末影箱、潜影盒等容器 | **允许** |
| 非主人拿到 | 属性、套装、技能、分类行为、自定义攻速全部失效，只相当于一件普通物品；原版交互不受影响 |

尚未绑定的物品（物品库预览、放在箱子里还没人拿过的物品）Lore 显示 `stats.yml` 的
`bind_pending`；已绑定时显示 `bind` 并代入主人名字。

管理员可用 `/ic bind <持有者> <新主人|clear>` 更换持有者主手中物品的绑定对象，或解除绑定；
解除后物品再进入任何玩家背包时会重新绑定给该玩家。

ItemCore 无法拦截其他插件通过 API 完成的物品转移（交易、仓库等），这类插件应在转移前调用
`ItemCoreAPI.isItemBound(itemStack)` 自行拦截，详见 [附属 API](addon-api.md)。
在 `config.yml` 中把 `bind.enabled` 设为 `false` 可整体关闭绑定系统。

## 随机属性

`attributes` 表示必定存在的基础属性，`random_attributes` 会在每个新物品生成时，
从 `entries` 中按权重抽取若干种不同属性：

在 ItemCore 物品库 GUI 中，随机范围会完整显示而不会预先抽取。例如
`ATTACK_DAMAGE` 配置 `min: 10`、`max: 12` 时，Lore 显示为 `10-12`；
玩家实际获得物品时仍只会生成一个精确到一位小数的随机值。

```yaml
attributes:
  ATTACK_DAMAGE: 15

random_attributes:
  count:
    min: 2
    max: 4
  entries:
    CRIT_CHANCE:
      min: 5
      max: 10
      weight: 100
    CRIT_DAMAGE:
      min: 10
      max: 25
      weight: 80
    ATTACK_SPEED:
      min: 0.1
      max: 0.4
      weight: 50
    HEALTH:
      value: 3
      weight: 30
```

`count` 可以直接填写固定数量，例如 `count: 3`；未配置时默认为 `1`。
`weight` 是相对权重，默认为 `100`。同一种属性不会重复抽取，数值精确到一位小数。
如果随机属性与基础属性相同，最终数值会相加。

随机结果保存在物品 PDC 中，只在物品首次生成时抽取。`/ic reload`、
`lore_refresh`、耐久变化和附属刷新都只读取已有结果，不会重新随机。修改配置只影响
之后生成的新物品，已经存在的物品不会自动获得或改变随机属性。

## 分类行为与类型 Lore

- 物品分类由 `categories.yml` 的分类 ID 以及对应 `items_file` 决定。
- 分类可通过单个 `parent` 继承父分类规则；图标、槽位、Lore 和物品文件不会继承。
- 父分类与子分类都注册系统时按父 → 子执行，限制类结果采用 AND。
- `ranged_weapons` 分类中的物品不会用远程武器属性造成近战全额伤害。
- `consumable` 分类中的物品在技能成功触发后消耗一个；若事件被取消则不消耗。
- `consumable` 分类可使用 `skills.drink` 或 `skills.eat`，完整播放对应原版动画后再触发技能。
- `display_type` 是开放的 Lore 文本，不参与分类或功能判断；未配置时默认显示分类 ID。
- Lore 布局中加入 `#type#` 可显示 `display_type`，格式由 `stats.yml` 的 `item_type` 控制。
- Lore 布局中加入 `#rank#` 可显示 `ranks.yml` 中对应品质的 `display`；未配置时该占位符不会产生 Lore。
- 旧版直接填写颜色文本或图片标签的 `rank` 仍可显示，但品质 ID 不存在且等级固定为 `0`，附属不应依赖这种写法。
- 旧配置名 `type` 仍可读取，但仅用于兼容，建议迁移到 `display_type`。
- `type_settings` 由物品所属分类注册的行为系统读取，未知配置不会被核心丢弃。
- 字符串和列表写法会转换为布尔类型标记，可通过 `hasTypeSetting(String)` 判断。
- 核心技能法力消耗应配置在 `skills.<触发方式>.mana_cost`；`type_settings.mana_cost` 只是附属类型系统可以自行读取的示例字段，不会自动扣除法力。

```yaml
arcane_wand:
  material: BLAZE_ROD
  display_type: '&d奥术法杖'
  type_settings:
    mana_cost: 20
    cast_mode: charged
```

宝石等分类只需要类型标记时，可以使用单值或列表简写：

```yaml
life_gem_1:
  material: PAPER
  type_settings: 生命

hybrid_gem:
  material: PAPER
  type_settings:
    - 生命
    - 攻击
```

列表简写在内部等价于：

```yaml
type_settings:
  生命: true
  攻击: true
```

## 工具专属配置

`tools` 分类的物品支持以下 `type_settings` 配置：

| 配置项 | 类型 | 默认 | 说明 |
|--------|------|------|------|
| `mining_speed` | 数值 | 材质原版速度 | 挖掘速度，对标原版工具基础速度值（木=2 石=4 铁=6 钻石=8 下界合金=9 金=12） |
| `vein_range` | 整数 | 不连锁 | 连锁挖掘范围（与被挖方块的最大距离），仅连锁同类型方块，单次上限 64 个 |

```yaml
miner_pickaxe:
  material: IRON_PICKAXE
  display_name: '&9矿工之镐'
  type_settings:
    mining_speed: 8    # 铁镐挖出钻石镐的速度
    vein_range: 3      # 连锁挖掘 3 格范围内的同类方块
```

- `mining_speed` 通过原版 `BLOCK_BREAK_SPEED` 属性实现，客户端挖掘进度条原生同步；仅主手生效。
- `mining_speed` 只改变挖掘速度，不改变掉落判定：能否掉落仍由工具材质等级决定。
- 挖掘速度与效率附魔叠乘：带附魔时表现为等比例加速，而非严格绝对值。
- 连锁挖掘对每个方块逐一走 `BlockBreakEvent`，兼容领地/保护插件，被保护的方块不会被连锁破坏。
- 连锁破坏的方块正常掉落、正常消耗工具耐久。

## 方块专属配置

`blocks` 分类的物品支持以下 `type_settings` 标记：

| 配置项 | 说明 |
|--------|------|
| `no_place` | 禁止放置，右键时提示 `messages.yml` 的 `blocks.no_place` |
| `infinite_place` | 放置后物品不消耗 |

两者同时配置时以 `no_place` 优先。

```yaml
decorative_stone:
  material: CHISELED_STONE_BRICKS
  type_settings:
    - no_place

builder_stone:
  material: STONE_BRICKS
  type_settings:
    - infinite_place
```

## 品质配置

固定品质与随机模板都定义在插件目录的 `ranks.yml`：

```yaml
ranks:
  common:
    level: 1
    display: '&7普通'

  excellent:
    level: 2
    display: '<image:sky_block:rank_b>'

random:
  default_random:
    common: 80
    excellent: 20
```

物品始终只使用一个 `rank` 配置项。填写固定品质 ID 时品质固定：

```yaml
coal_helmet:
  material: LEATHER_HELMET
  rank: excellent
```

填写随机模板 ID 时，在每次创建新物品实例时按权重抽取一次：

```yaml
random_helmet:
  material: LEATHER_HELMET
  rank: default_random
```

- `level` 用于附属比较品质高低，必须大于 `0`。
- `display` 是品质的展示值，支持颜色代码与图片标签；`#rank#` 会把它代入 `stats.yml` 的 `rank` 格式。
- 随机模板中的数字是相对权重，不要求总和为 `100`。
- 固定品质 ID 与随机模板 ID 共享命名空间，不允许重名。
- ItemCore 不根据品质等级限制强化次数或执行其它玩法规则。
- 随机品质只在真实物品生成时抽取，结果品质 ID 会保存到 PDC；Lore 刷新和配置重载不会重新抽取。
- 物品库预览不会固化随机结果，`#rank#` 会逐行显示模板内品质及其实际概率。
- 修改随机模板只影响以后生成的物品；修改物品模板的 `rank` 不会改变已有物品的品质。
- 修改某个品质的 `level` 或 `display` 会影响所有引用该品质 ID 的物品。
- 未找到固定品质或随机模板时，`rank` 仍按旧版展示文本处理，但品质等级为 `0`。

```yaml
# categories.yml
categories:
  melee_weapons:
    name: 近战武器
    items_file: melee_weapons.yml

  swords:
    parent: melee_weapons
    name: 单手剑
    items_file: swords.yml
```

## 物品染色

`color` 使用红、绿、蓝三个 `0-255` 的整数，支持皮革护甲、普通药水、喷溅药水和滞留药水：

```yaml
煤炭头盔:
  material: LEATHER_HELMET
  display_name: "煤炭头盔"
  color: 255, 0, 0
```

```yaml
赤红药水:
  material: POTION
  display_name: "&c赤红药水"
  color: 255, 0, 0
```

## 完整示例

```yaml
fire_sword:
  material: DIAMOND_SWORD
  display_type: '&c近战武器'
  display_name: "&c烈焰之剑"
  lore:
    - "&7燃烧一切的烈焰之剑"
  enchantments:
    sharpness: 3
    fire_aspect: 2
  item_flags:
    - HIDE_ATTRIBUTES
  enchantment_glint: true
  unbreakable: true
  active_slots:
    - main_hand
  attributes:
    ATTACK_DAMAGE: 12
    ATTACK_SPEED: 1.6
    CRIT_CHANCE: 15
    CRIT_DAMAGE: 30
    PHYSICAL_PENETRATION: 5
  skills:
    right_click:
      skill: fireball
      provider: mythicmobs
```
