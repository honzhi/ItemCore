# 常见问题

**Q: 修改配置后需要重启服务器吗？**
A: `/ic reload` 即可热重载。已生成物品的随机属性、品质、耐久、绑定不会被重载改变；Lore 布局修改可开启 `lore_refresh` 自动同步。

**Q: 旧配置里的 `display-name`、`active-slots` 还能用吗？**
A: 配置键已统一为下划线写法（`display_name`、`active_slots`、`item_flags`、`custom_model_data`、`max_stack`、`keep_on_death`），`active_slots` 内的 `main-hand` 会自动转换，但其他键请迁移。元素配置的 `decay-per-second`、`allow-sources`、`refresh-policy` 也需改为下划线。

**Q: `/ic give` 提示找不到物品？**
A: 现在必须写 `/ic give 玩家 分类ID 物品ID`，分类 ID 就是 `categories.yml` 中的键名。

**Q: 属性为什么不生效？**
A: 依次检查：`active_slots` 是否包含当前槽位；物品是否损坏（耐久 0）；玩家是否有物品 `permission`；物品是否绑定给了别人。

**Q: 多装备属性如何计算？**
A: 同名属性自动累加。如两件装备都有 `ATTACK_DAMAGE: 10`，最终 +20。

**Q: 暴击伤害显示 200%？**
A: `CRIT_DAMAGE` 是额外加成。总暴击 = `attributes.yml` 的 `default_crit_damage`（默认 150%）+ `CRIT_DAMAGE`。

**Q: 攻击速度怎么配置？**
A: 主手武器直接写最终值，`1.6` = 铁剑速度，`4.0` = 极快。护甲等位置写的是加成值。

**Q: 百分比属性怎么写？**
A: 直接填整数，`CRIT_CHANCE: 30` = 30%，不是 0.3。

**Q: 为什么自定义武器的伤害和原版武器攻击力无关？**
A: 带 `HIDE_ATTRIBUTES` 的自定义武器会忽略原版武器伤害，只使用 `ATTACK_DAMAGE`。去掉该标志则两者相加。

**Q: 技能 `icdamage` 为什么不暴击？**
A: `attacktype=skill`（默认）的伤害暴击率为 0。需要暴击请使用 `attacktype=attack`，此时会使用玩家的暴击属性并受暴击冷却限制。

**Q: 适应之力有什么用？**
A: 比较攻击伤害与法术强度，转化为较高者的加成。转化率在 `attributes.yml` 配置。

**Q: 元素伤害怎么计算？**
A: 跳过物抗 / 法抗，只受元素抗性、异常易伤和伤害减免影响。

**Q: 技能怎么触发？**
A: 安装 ItemCoreMythic，在物品配置的 `skills` 节点配置技能，确保 `active_slots` 包含对应槽位，且开启法力系统时法力足够。

**Q: 绑定物品放进箱子被别人拿走了怎么办？**
A: 这是预期行为，别人拿到后物品只相当于普通物品。可用 `/ic bind <持有者> <主人>` 改绑，或 `/ic bind <持有者> clear` 解绑后重新获得。

**Q: 创造模式物品攻速不对？**
A: 创造模式物品栏机制问题，生存模式或指令获取正常。
