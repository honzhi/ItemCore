# 技能系统（ItemCoreMythic）

ItemCoreMythic 依赖 ItemCore 与 MythicMobs，为物品与套装提供技能触发，并向 MythicMobs 注册 `icdamage` / `icheal` 机制。

## 技能配置

```yaml
skills:
  right_click:
    skill: fireball
    provider: mythicmobs     # 默认 mythicmobs，可省略
    chance: 25               # 触发概率 0-100，默认 100
    mana_cost: 20            # 法力消耗，默认 0
  left_click:
    skill: slash
  attack:
    skill: bleed
  damage:
    skill: counter
  drink:
    skill: healing_drink
  eat:
    skill: strength_food
  block_break:
    skill: coal_reward
    chance: 10
    natural_only: true
    blocks:
      - COAL_ORE
      - DEEPSLATE_COAL_ORE
  timer:
    skill: regen
    duration: 20             # 间隔 tick，默认 20
```

触发键不区分大小写，`Right_Click`、`right_click`、`on_right_click` 均可识别。

## 触发类型

| 类型 | 说明 |
|------|------|
| `right_click` | 右键触发 |
| `left_click` | 左键触发 |
| `drink` | `consumable` 完整播放原版饮用动画后触发，不留下空瓶 |
| `eat` | `consumable` 完整播放原版进食动画后触发 |
| `attack` | 使用主手物品攻击实体时触发 |
| `damage` | 玩家受到最终伤害大于 0 的伤害后触发；实体伤害来源会作为技能目标 |
| `block_break` | 使用主手物品成功破坏方块时触发 |
| `timer` | 定时触发，`duration` 单位为 tick |

`drink` 和 `eat` 仅对 `consumable` 类型生效，同一物品只能选择其中一种；同时配置时优先使用 `drink`。动画被中断不会触发技能或消耗物品，技能概率未命中或触发事件被取消时也不会消耗。

技能只在物品处于 `active_slots` 允许的槽位、玩家有权限、物品未损坏且为绑定主人时触发。

## 概率与方块过滤

- `chance` 适用于物品和套装的全部技能，取值为 `0-100`，支持小数，默认 `100`。
- `block_break.natural_only` 默认为 `true`，玩家放置的方块不会触发技能。
- `natural_only: true` 时必须配置至少一个有效的 `blocks` 方块，否则技能不会加载。
- 如需允许玩家放置的方块触发，可显式配置 `natural_only: false`；此时 `blocks` 可省略。
- 被其他插件取消的方块破坏事件不会触发技能。
- 挖掘技能的施法者是玩家，技能位置是被破坏方块的中心。
- 玩家放置记录保存在区块 PDC 中，服务器重启后仍然有效，并会随活塞移动。

## 法力消耗

- `mana_cost` 适用于物品和套装的全部技能触发方式，默认值为 `0`，支持小数。
- 法力系统关闭时忽略技能消耗；开启后，当前法力不足的技能不会触发。
- 槽位、权限、损坏状态、方块过滤、概率和物品类型检查均通过后才会扣除法力。
- 技能触发事件被取消时会退还已经扣除的法力。
- `drink` 和 `eat` 技能法力不足时不会消耗物品；`timer` 技能法力不足时只跳过当前一次触发。

### 法力不足提示

`config.yml` 中可配置提示显示位置：

```yaml
mana:
  not_enough_display: hud # chat / hud / both
  hud:
    popup: itemcore_mana_warning
    duration: 30
```

设置为 `hud` 时需要安装 BetterHud；ItemCore 会自动生成对应 Popup 配置并在法力不足时短暂显示。BetterHud 未安装、连接失败或 Popup 尚未加载时会回退到聊天栏；`both` 会同时显示两种提示。

聊天栏和 HUD 使用独立文案：聊天栏读取 `messages.yml` 的 `mana.not_enough`，HUD 直接读取 `config.yml` 的 `mana.hud.message`。HUD 文案支持 MiniMessage 标签，例如 `<red>`、`<aqua>`，并支持 `{cost}` 与 `{current}` 变量。

## icdamage 机制

在 MythicMobs 技能中使用 ItemCore 伤害计算：

```yaml
Skills:
- icdamage{amount="<ic.attack_damage> * 2";type=physical;element=LIUHUO;penetration=5} @EIR{r=5}
- icdamage{amount="<caster.damage>*1.2";type=physical;attacktype=attack;lifesteal=0.1} @target
```

| 参数 | 别名 | 默认值 | 说明 |
|------|------|--------|------|
| `amount` | `a` | `1` | 伤害公式，支持 MM 占位符、`<ic.xxx>`、四则运算与 `%` 写法 |
| `type` | `t` | `spell` | `physical` / `spell` / `projectile` |
| `element` | `e` | `none` | 元素 ID；`fire` / `ice` / `thunder` 分别是流火 / 寒霜 / 雷蛰的别名 |
| `attacktype` | `at` | `skill` | `skill` / `attack`，决定走哪套伤害规则（见[伤害系统](damage.md)） |
| `crit` | `c` | `true` | 是否允许暴击；只有 `attacktype=attack` 时才会使用玩家暴击属性，`skill` 类型不会暴击 |
| `penetration` | `p` | `0` | 额外穿透，与攻击者自身穿透相加，支持占位符与公式 |
| `lifesteal` | `ls` | `0` | 吸血比例（0.1 = 10%），仅 `attacktype=attack` 生效 |

参数使用 MythicMobs 标准分号 `;` 分隔。

### `<ic.xxx>` 占位符

| 占位符 | 说明 |
|--------|------|
| `<ic.attack_damage>` 等任意属性键 | 施法者的 ItemCore 属性 |
| `<ic.crit_damage>` | 玩家为总暴击伤害（含默认 150） |
| `<ic.health>` | 玩家为最大生命值 |
| `<ic.movement_speed>` | 玩家为总移速 |
| `<ic.element_mastery.LIUHUO>` | 指定元素精通 |
| `<ic.element_resist.LIUHUO>` | 指定元素抗性 |

MythicMobs 怪物作为施法者时，属性来自附属注册的实体属性提供器。

```
<ic.spell_power> * 0.5        # 50% 法术强度
<ic.attack_damage> * 2 + 10   # 攻击力 × 2 + 10
<ic.attack_damage> * 50%      # 50% 攻击力
```

## icheal 机制

```yaml
Skills:
- icheal{amount="<ic.spell_power> * 0.3"} @self
```

`amount` 支持 MM 占位符与单次四则运算，经 ItemCore 治疗流程（`HealingRequest`）处理。

## ItemCoreMythic 配置

```yaml
debug: false
providers:
  mythicmobs:
    enabled: true
cooldown-notification:       # MythicMobs 技能冷却提示
  enabled: true
  type: actionbar            # actionbar / chat
  message: "&c技能 &e{skill} &c正在冷却，还需 &e{remaining}&c 秒"
  decimal-places: 1
  throttle-millis: 250
```

## active_slots 影响

- 属性：装备位决定生效
- 技能：只有符合 `active_slots` 的实际生效物品能够触发
- 药水效果：装备位决定给予
- `never`：任何槽位都不生效，并优先于同一列表中的其他槽位
