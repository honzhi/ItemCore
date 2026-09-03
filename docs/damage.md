# 伤害系统

## 伤害类型

| 类型 | 标签 | 说明 |
|------|------|------|
| 物理 | `PHYSICAL` | 普攻默认类型，吃 `PHYSICAL_DAMAGE` / 物理抗性 |
| 法术 | `SPELL` | 技能默认类型，吃 `SPELL_DAMAGE` / 法抗 |
| 射弹 | `PROJECTILE` | 弓弩等射弹，吃 `PROJECTILE_DAMAGE` |

## 原版攻击流程（近战与射弹）

```
基础伤害（武器带 HIDE_ATTRIBUTES 时按 1 计，原版武器伤害不计入）
  → + ATTACK_DAMAGE（近战按蓄力比例折算，最低 20%）
  → × 伤害类型加成（PHYSICAL_DAMAGE / PROJECTILE_DAMAGE / SPELL_DAMAGE）
  → 暴击判定（CRIT_CHANCE，受暴击冷却限制）× 总暴击伤害
  → 物理抗性减免（扣除固定 / 百分比穿透后套用 defense_formulas.physical_resist）
  → × (1 - DAMAGE_REDUCTION%)
  → ItemCoreDamageEvent（附属可修改或取消）
  → 扣除主手自定义耐久 1 点
```

- 主手 `ranged_weapons` 物品在直接近战时不贡献属性。
- 非绑定主人或无权限持有的自定义武器按普通物品处理。
- 暴击时根据 `KNOCKBACK` 增强击退。

## DamageRequest 流程（技能、icdamage、异常 DOT）

```
基础伤害
  → attacktype=attack：+ ATTACK_DAMAGE，× 伤害类型加成，使用玩家暴击属性
    attacktype=skill：伤害由技能公式自行决定，不额外加属性，暴击率按请求（icdamage 为 0）
  → 暴击判定
  → 穿透 = 请求穿透 + 攻击者对应类型的固定 / 百分比穿透
  → 无元素：按类型套用物理抗性或法抗减免
    有元素：跳过物理 / 法抗
  → 元素抗性、异常 RESISTANCE_REDUCTION、DAMAGE_REDUCTION 加法叠加
    （减伤上限 100%，增伤无上限）
  → ItemCoreDamageEvent
  → 吸血（仅 attacktype=attack）
  → 应用伤害 → 元素积累 → 暴击击退
```

## 元素伤害规则

- 带元素的伤害**跳过**物理 / 法术抗性，只受元素抗性、异常易伤与 `DAMAGE_REDUCTION` 影响。
- 元素抗性为负数时变为易伤。
- 元素伤害造成后才计入元素积累。

## 伤害减免

`DAMAGE_REDUCTION` 对所有伤害生效（包括元素伤害），为最终减伤，可为负数（增伤）。
