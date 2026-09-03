# 元素系统

## 默认元素

| 元素 ID | 显示名 | 图标 | 颜色 | 积累方式 | 异常效果 |
|---------|--------|------|------|----------|---------|
| `LIUHUO` | 流火 | `🔥` | `&c` | 伤害 × 50% | 灼烧：8 秒，每秒 2% 当前生命的流火伤害 |
| `HANSHUANG` | 寒霜 | `❄` | `&b` | 每次固定 25 | 虚弱：4 秒，物理抗性 / 法抗 -30% |
| `LEIZHE` | 雷蛰 | `⚡` | `&e` | 法术强度 × 0.2 | 易伤：6 秒，受到的技能伤害 +25% |

## 元素配置

```yaml
# elements.yml
elements:
  LIUHUO:
    display: "&c流火"
    icon: '🔥'              # 伤害飘字图标（默认 ✦）
    color: '&c'             # 伤害飘字颜色（默认 &f）
    threshold: 30           # 积累阈值，达到后触发异常（默认 100）
    decay_per_second: 1     # 每秒衰减量（默认 5）
    accumulation:
      mode: DAMAGE_PERCENT  # DAMAGE_PERCENT / FIXED / ATTRIBUTE
      value: 0.5            # 每次伤害 × 50% 计入积累
      allow_sources:        # 允许积累的来源，留空 = 全部
        - ATTACK
        - SKILL
    ailment: LIUHUO_DOT     # 达到阈值时触发的异常 ID
```

## 积累模式

| 模式 | 积累值 |
|------|--------|
| `DAMAGE_PERCENT` | 元素伤害 × `value` |
| `FIXED` | 每次固定 `value` |
| `ATTRIBUTE` | 攻击者 `attribute` × `multiplier`，`attribute` 支持 `SPELL_POWER` 或 `ATTACK_DAMAGE` |

积累规则：

- 只有带元素的伤害才会积累；来源为 `ATTACK`（普攻或 `attacktype=attack`）或 `SKILL`（其他 DamageRequest 伤害，包括异常 DOT）。
- 目标的元素抗性会降低积累量，最多降低 75%。
- 达到阈值后触发异常并清零该元素积累。
- 没有元素伤害输入时每秒按 `decay_per_second` 衰减。

## 异常配置

```yaml
# ailments.yml
ailments:
  LIUHUO_DOT:
    display: "&c流火"
    duration: 160           # tick，160 = 8 秒
    refresh_policy: RESET   # RESET / STACK / IGNORE / REPLACE
    max_stacks: 1           # 仅 STACK 策略使用
    triggers:
      - type: DAMAGE_PERCENT
        value: 0.02         # 2% 当前生命
        interval: 20        # 每 20 tick 触发一次

  HANSHUANG_WEAKEN:
    display: "&b寒霜"
    duration: 80
    refresh_policy: RESET
    triggers:
      - type: ATTRIBUTE_MOD
        attribute: PHYSICAL_RESIST
        value: -0.3         # 属性 × (1 - 0.3)
      - type: ATTRIBUTE_MOD
        attribute: SPELL_RESIST
        value: -0.3

  LEIZHE_BREAK:
    display: "&e雷蛰"
    duration: 120
    refresh_policy: RESET
    triggers:
      - type: RESISTANCE_REDUCTION
        value: -0.25        # 受到的技能伤害 +25%
```

### 刷新策略

| 策略 | 行为 |
|------|------|
| `RESET` | 已存在时重新计时（默认） |
| `STACK` | 叠加层数，每层独立计时，上限 `max_stacks` |
| `IGNORE` | 已存在时忽略新触发 |
| `REPLACE` | 移除旧的，应用新的 |

### 触发器类型

| 类型 | 说明 |
|------|------|
| `DAMAGE_PERCENT` | 每 `interval` tick 造成 `value` × 当前生命的伤害；伤害带该元素、按法术类型走 ItemCore 伤害流程，不暴击 |
| `DAMAGE_FIXED` | 每 `interval` tick 造成固定 `value` 点原版伤害（不经过 ItemCore 伤害计算） |
| `ATTRIBUTE_MOD` | 持续期间将目标 `attribute` 乘以 `(1 + value)`，可用于任意属性 |
| `RESISTANCE_REDUCTION` | 持续期间提高目标受到的 DamageRequest 伤害（技能、`icdamage`、DOT），`value` 为负数；原版普攻不受影响 |
| `POTION_EFFECT` | 尚未实现 |

三种默认异常有内置粒子效果（火焰 / 雪花 / 紫色尘埃），自定义异常没有粒子。

## 拓展自定义元素

```java
ItemCore.getElementRegistry().register(new ElementType("ARCANE", "奥术"));
```

注册后在 `elements.yml` 与 `ailments.yml` 中配置积累与异常规则，并可在物品 `attributes.element_mastery` / `element_resist` 与技能 `element=ARCANE` 中使用。自定义元素应在物品配置加载前注册。
