# PlaceholderAPI 占位符

标识符：`itemcore`。属性占位符可使用任意属性配置键，百分比属性返回带 `%` 的数值。

## 属性类

| 占位符 | 说明 |
|--------|------|
| `%itemcore_attack_damage%` | 攻击伤害 |
| `%itemcore_attack_speed%` | 攻击速度属性值 |
| `%itemcore_attack_range%` | 攻击范围 |
| `%itemcore_health%` | 当前生命值 |
| `%itemcore_max_health%` | 最大生命值 |
| `%itemcore_mana%` | 当前法力值 |
| `%itemcore_max_mana%` | 最大法力值 |
| `%itemcore_mana_percent%` | 当前法力百分比，返回 `0-100` 数值 |
| `%itemcore_mana_regeneration%` | 每秒自然恢复法力值 |
| `%itemcore_movement_speed%` | 总移速 |
| `%itemcore_regeneration%` | 生命恢复 |
| `%itemcore_knockback%` | 击退 |
| `%itemcore_luck%` | 幸运值 |
| `%itemcore_spell_damage%` | 法术加成 |
| `%itemcore_physical_damage%` | 物理加成 |
| `%itemcore_projectile_damage%` | 射弹加成 |
| `%itemcore_spell_power%` | 法术强度 |
| `%itemcore_adaptive_force%` | 适应之力 |
| `%itemcore_crit_chance%` | 暴击几率 |
| `%itemcore_crit_damage%` | 总暴击伤害 |
| `%itemcore_armor%` | 护甲 |
| `%itemcore_physical_resist%` | 物理抗性 |
| `%itemcore_spell_resist%` | 法术抗性 |
| `%itemcore_physical_penetration%` | 物理穿透 |
| `%itemcore_physical_penetration_percent%` | 物理穿透% |
| `%itemcore_spell_penetration%` | 法术穿透 |
| `%itemcore_spell_penetration_percent%` | 法术穿透% |
| `%itemcore_damage_reduction%` | 伤害减免 |

## 装备类

`<槽位>` 可为 `main_hand`、`off_hand`、`head`、`chest`、`legs`、`feet`。

| 占位符 | 说明 |
|--------|------|
| `%itemcore_durability_<槽位>%` | 剩余耐久（自定义耐久优先，否则原版耐久） |
| `%itemcore_max_durability_<槽位>%` | 最大耐久 |
| `%itemcore_durability_percent_<槽位>%` | 耐久百分比 |
| `%itemcore_has_durability_<槽位>%` | 是否有可用耐久（`true` / `false`） |
| `%itemcore_has_<槽位>%` | 该槽位是否有物品 |
| `%itemcore_icon_<槽位>%` | 物品的 BetterHud 图标名（`item_材质名`） |

例如 `%itemcore_durability_main_hand%`、`%itemcore_durability_percent_chest%`、`%itemcore_icon_head%`。
