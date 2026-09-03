# Lore 展示系统

## 布局配置

```yaml
# tooltip/lore.yml
lore_format:
  - '#item_lore#'
  - '#rank#'
  - '{bar}   &6基础属性'
  - '#attack_damage#'
  - '#attack_speed#'
  - '#health#'
  - '{bar}'
  - '#set_lore#'
  ...
```

## 占位符

| 占位符 | 说明 |
|--------|------|
| `#属性名#` | 显示属性值（为0自动隐藏）；`#element_mastery#` / `#element_resist#` 逐元素显示 |
| `#item_lore#` | 物品描述文本 |
| `#type#` | 物品配置中的 `display_type`，格式由 `stats.yml` 的 `item_type` 控制 |
| `#rank#` | 显示物品品质在 `ranks.yml` 中配置的 `display`，未配置时隐藏 |
| `#durability#` | 自定义耐久，格式由 `stats.yml` 的 `durability` 控制 |
| `#bind#` | 开启了 `bind: true` 的物品显示绑定状态：已绑定用 `stats.yml` 的 `bind`（`{owner}` 为主人名字），未绑定用 `bind_pending`；未开启绑定的物品隐藏 |
| `#set_lore#` | 物品所属套装的描述文本 |
| `#ext:命名空间:键#` | 显示附属插件注册的动态 Lore 区块 |
| `{bar}` | 条件区块边界；两个分隔符之间有内容时才显示 |
| `{bar}文本` | 条件区块标题；标题本身不参与内容判断 |
| `{sbar}` | 无条件区块边界；即使区块为空也显示 |
| 普通文本 | 始终显示的固定文本，支持颜色代码 |

`{bar}   &6基础属性` 中 `{bar}` 后的空格与文本会原样保留。ItemCore 会先渲染当前 `{bar}` 到下一个 `{bar}` 或 `{sbar}` 之间的整个区块；其中没有任何可见内容时，分隔空行、标题和区块都会隐藏。文件结尾视为区块的结束边界。

内容判断不限于属性，也包括物品描述、类型、套装 Lore、耐久、附属 Lore 和普通固定文本。

附属插件占位符必须单独占一行。提供器返回空列表时，该区块会自动隐藏，并且不会触发前面的 `{bar}` 条件文本。

```yaml
lore_format:
  - '{bar}&7等级信息'
  - '#ext:itemcorelevel:level#'
```

## 属性显示格式

```yaml
# tooltip/stats.yml
attack_damage: '&f攻击伤害: &6<plus>{value}'
crit_chance: '&f暴击几率: &6<plus>{value}%'
```

变量：`{value}` 数值、`<plus>` 自动 +/- 号；元素属性格式可用 `{display}` 代入元素显示名。属性为范围时 GUI 预览显示 `min-max`。

物品配置中的类型通过 `item_type` 格式显示：

```yaml
# tooltip/stats.yml
item_type: '&f{value}'
rank: '&6● &7品质: &r{value}'
```

`item_type` 的 `{value}` 表示物品配置中的 `display_type`。未配置时使用物品所属分类 ID；旧字段 `type` 仅作为兼容别名。将 `item_type` 设置为空字符串可以隐藏物品类型。

`#rank#` 推荐搭配结构化品质使用。物品中的 `rank: excellent` 会读取
`ranks.yml` 中 `ranks.excellent.display` 的文本；旧版直接填写展示文本的写法仍可显示，
并通过 `stats.yml` 的 `rank` 格式替换 `{value}`；将 `rank` 设置为空字符串可以隐藏品质。
旧版文本不会获得可供附属读取的品质 ID 和等级。物品引用 `random` 下的随机模板时，真实物品
显示已经抽中的品质；物品库预览则逐行显示模板内每个品质及其实际概率。

## 配色

- 未使用颜色字符时，Lore 默认显示为白色。
- 词条名：`&f`（白色）
- 数值：`&6`（橙色）

## 自动刷新

```yaml
# config.yml
lore_refresh:
  enabled: true
  interval: 100  # tick
```

开启后会按 `interval` 定期扫描在线玩家背包并刷新 Lore，使 `/ic reload` 后的布局修改自动同步到已有物品。玩家较多时请适当加大间隔。
