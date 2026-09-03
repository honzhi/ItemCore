# 属性系统

配置键不区分大小写，`ATTACK_DAMAGE` 与 `attack_damage` 等价。百分比属性直接填整数（`30` = 30%），也可写 `"30%"`。

## 攻击类

| 配置键 | 显示名 | 百分比 | 说明 |
|--------|--------|--------|------|
| `ATTACK_DAMAGE` | 攻击伤害 | — | 基础攻击力，直接加到伤害上 |
| `ATTACK_SPEED` | 攻击速度 | — | 主手武器上写最终值（`1.6` = 铁剑）；护甲等其他位置上写加成值（基础 4.0） |
| `ATTACK_RANGE` | 攻击范围 | — | 攻击距离加成 |
| `KNOCKBACK` | 击退 | — | 暴击时额外击退倍率 |
| `CRIT_CHANCE` | 暴击几率 | % | 暴击概率 |
| `CRIT_DAMAGE` | 暴击伤害 | % | 额外暴击倍率 |

## 法术/伤害类型

| 配置键 | 显示名 | 百分比 | 说明 |
|--------|--------|--------|------|
| `SPELL_POWER` | 法术强度 | — | 法术伤害基础值，供技能公式 `<ic.spell_power>` 使用 |
| `PHYSICAL_DAMAGE` | 物理加成 | % | 物理伤害百分比 |
| `SPELL_DAMAGE` | 法术加成 | % | 法术伤害百分比 |
| `PROJECTILE_DAMAGE` | 射弹加成 | % | 射弹伤害百分比 |
| `ADAPTIVE_FORCE` | 适应之力 | — | 自动转为攻击或法强 |

## 防御类

| 配置键 | 显示名 | 百分比 | 说明 |
|--------|--------|--------|------|
| `ARMOR` | 护甲 | — | 原版护甲值 |
| `PHYSICAL_RESIST` | 物理抗性 | — | 物理减伤，公式可配置 |
| `SPELL_RESIST` | 法术抗性 | — | 法术减伤，公式可配置 |
| `DAMAGE_REDUCTION` | 伤害减免 | % | 最终减伤，可为负数（增伤） |

当 `config.yml` 中的 `combat.disable_vanilla_armor` 设置为 `true` 时，原版护甲值、护甲韧性以及 ItemCore 的 `ARMOR` 属性均不再参与减伤。此模式下请使用 `PHYSICAL_RESIST` 配置物理防御；保护类附魔仍然生效。

## 穿透类

| 配置键 | 显示名 | 百分比 | 说明 |
|--------|--------|--------|------|
| `PHYSICAL_PENETRATION` | 物理穿透 | — | 固定值穿透 |
| `PHYSICAL_PENETRATION_PERCENT` | 物理穿透% | % | 百分比穿透 |
| `SPELL_PENETRATION` | 法术穿透 | — | 固定值穿透 |
| `SPELL_PENETRATION_PERCENT` | 法术穿透% | % | 百分比穿透 |

## 生存类

| 配置键 | 显示名 | 百分比 | 说明 |
|--------|--------|--------|------|
| `HEALTH` | 生命值 | — | 额外生命（基础20） |
| `MANA` | 法力值 | — | 额外最大法力值 |
| `MOVEMENT_SPEED` | 移动速度 | — | 额外移速（基础0.1） |
| `REGENERATION` | 生命恢复 | — | 每秒恢复 |
| `MANA_REGENERATION` | 法力恢复 | — | 每秒额外恢复法力值 |
| `LUCK` | 幸运值 | — | 幸运加成 |

最大法力值为 `config.yml` 中的 `mana.base` 加上玩家全部 `MANA` 属性；每秒自然恢复量为 `mana.base_regeneration` 加上全部 `MANA_REGENERATION` 属性。法力系统由 `mana.enabled` 控制。

## 元素属性

在物品 `attributes` 中通过 `element_mastery` / `element_resist` 子节点配置：

```yaml
attributes:
  element_mastery:
    LIUHUO: 20
  element_resist:
    HANSHUANG: 10
```

| 配置键 | 显示名 | 说明 |
|--------|--------|------|
| `element_mastery.<元素ID>` | 元素精通 | 目前核心伤害计算不直接使用，可通过 API 与技能占位符 `<ic.element_mastery.LIUHUO>` 读取 |
| `element_resist.<元素ID>` | 元素抗性 | 降低对应元素伤害与元素积累量，负数为易伤 |

## 属性叠加顺序

玩家最终属性按以下顺序计算，同名属性自动累加：

1. 各生效槽位物品属性（含随机属性、附属物品属性提供器）
2. 套装激活属性
3. 附属注册的玩家属性提供器（饰品、称号、等级等）
4. 临时属性固定值
5. 适应之力转换
6. 异常状态的 `ATTRIBUTE_MOD`
7. 临时属性百分比

## 适应之力

```yaml
# attributes.yml
adaptive_force:
  attack_conversion: 0.5   # 每1点 → 0.5攻击
  spell_conversion: 1.0    # 每1点 → 1.0法强
```

比较 `ATTACK_DAMAGE` 和 `SPELL_POWER`，攻击伤害大于等于法术强度时转为攻击伤害，否则转为法术强度。

## 暴击机制

`CRIT_DAMAGE` 为额外加成。总暴击伤害 = 默认值（`attributes.yml` 的 `default_crit_damage`，默认 150%）+ `CRIT_DAMAGE`。

暴击冷却（`attributes.yml` 的 `crit.cooldown`）只限制 ATTACK 类型伤害（原版普攻、`icdamage` 中 `attacktype=attack`），SKILL 类型不受限制：

- `FIXED` 模式：两次暴击间隔至少 `fixed_ticks`。
- `FULL_CHARGE` 模式：近战蓄力达到 `full_charge_threshold` 才能暴击；射弹与技能攻击不受蓄力限制。

## 攻击速度

- 主手自定义武器配置了非 0 的 `ATTACK_SPEED` 时，玩家攻速 = 所有生效物品 `ATTACK_SPEED` 之和，武器上直接写最终值：`1.6` = 铁剑速度，`4.0` = 极快。
- 主手没有自定义武器攻速时，玩家攻速 = 4.0 + 其他物品的 `ATTACK_SPEED` 加成。
- 近战蓄力不足时，`ATTACK_DAMAGE` 会按蓄力比例（最低 20%）折算。

## 防御公式

```yaml
# 默认：百分比减伤，可用变量 {damage} {armor}
defense_formulas:
  physical_resist: '{damage} * (1 - {armor} / ({armor} + 100))'
  spell_resist: '{damage} * (1 - {armor} / ({armor} + 100))'

penetration_order: percent_first  # percent_first / flat_first
```

## 百分比属性

直接填整数：`CRIT_CHANCE: 30` = 30%
