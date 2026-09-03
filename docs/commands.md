# 命令系统

## 主命令 /ic

主命令 `/itemcore`，别名 `/ic`。不带参数时显示帮助。

| 子命令 | 用途 | 权限 |
|--------|------|------|
| `help [页码]` | 显示帮助（每页 5 条） | `itemcore.command.help` |
| `open` / `gui` | 打开物品库 GUI | `itemcore.command.gui` |
| `give <玩家> <分类> <物品> [数量] [-s]` | 给予物品，数量 1-64，`-s` 不通知目标玩家 | `itemcore.command.give` |
| `attribute add <玩家> <属性> <数值> <持续时间tick> <-s\|-r>` | 添加固定临时属性 | `itemcore.command.attribute` |
| `attribute percent <玩家> <属性> <数值> <持续时间tick> <-s\|-r>` | 添加百分比临时属性 | `itemcore.command.attribute` |
| `attribute remove <玩家> <属性>` | 移除指定属性的全部临时效果 | `itemcore.command.attribute` |
| `attribute clear <玩家>` | 移除玩家的全部临时属性效果 | `itemcore.command.attribute` |
| `element_mastery <add\|percent\|remove\|clear> ...` | 管理临时元素精通 | `itemcore.command.element_attribute` |
| `element_resist <add\|percent\|remove\|clear> ...` | 管理临时元素抗性 | `itemcore.command.element_attribute` |
| `bind <持有者> <新主人\|clear>` | 更换持有者主手物品的绑定对象，`clear` 解除绑定 | `itemcore.command.bind` |
| `reload` | 热重载配置 | `itemcore.command.reload` |

### 示例

```
/ic open
/ic give Is_Lianhua melee_weapons legendary_blade 1
/ic give Is_Lianhua melee_weapons legendary_blade 1 -s
/ic attribute add Is_Lianhua attack_damage 10 600 -s
/ic attribute percent Is_Lianhua attack_damage -20 200 -r
/ic attribute remove Is_Lianhua attack_damage
/ic element_mastery add Is_Lianhua LIUHUO 10 600 -s
/ic element_resist percent Is_Lianhua HANSHUANG 20 600 -r
/ic element_mastery remove Is_Lianhua LIUHUO
/ic bind Is_Lianhua Steve
/ic bind Is_Lianhua clear
/ic reload
```

`give` 指令始终要求分别填写分类 ID 和物品 ID。Tab 补全会先列出分类，再只列出
该分类中的物品，不会尝试按全局唯一裸 ID 匹配。背包已满时剩余物品掉落在玩家脚下。

`bind` 指令操作的是持有者**主手**中的物品，且该物品必须在配置中开启了 `bind: true`。
新主人可以是离线玩家，但必须是服务器曾经见过的玩家名。`clear` 解除绑定后，物品再进入任何玩家背包时会重新绑定给该玩家。

## 临时属性规则

- 属性名使用小写配置键，如 `attack_damage`、`crit_chance`，大写也可识别。
- `-s` 为叠加：保留已有同属性效果，每条效果独立计时。
- `-r` 为替换：先移除该属性已有的全部固定与百分比临时效果，再添加新效果。
- 固定效果先相加，百分比效果随后逐层乘算。例如 `+10` 与 `-20%` 的结果为 `(原值 + 10) × 0.8`。
- `percent` 乘算 ItemCore 属性容器中的值，不包含基础生命 `20`、基础移速 `0.1`、无武器基础攻速 `4` 或基础法力；`armor` 会另外映射到原版护甲运算。
- 正数表示增益，负数表示减益；最终值是否允许为负数遵循对应属性及其现有计算规则。例如，负数 `damage_reduction` 会增伤，`attack_damage` 可降为负数但最终伤害不会低于 0。
- `remove` 和 `clear` 只影响临时效果，不会修改装备、套装或外部 Provider 属性。
- 临时效果仅保存在内存中，玩家下线后仍按服务器 Tick 计时，服务器重启后清除。

## 临时元素属性

元素精通和元素抗性使用独立命令，不与 `attribute` 的基础属性名称混用：

```text
/ic element_mastery add <玩家> <元素> <数值> <持续时间tick> <-s|-r>
/ic element_mastery percent <玩家> <元素> <数值> <持续时间tick> <-s|-r>
/ic element_mastery remove <玩家> <元素>
/ic element_mastery clear <玩家>

/ic element_resist add <玩家> <元素> <数值> <持续时间tick> <-s|-r>
/ic element_resist percent <玩家> <元素> <数值> <持续时间tick> <-s|-r>
/ic element_resist remove <玩家> <元素>
/ic element_resist clear <玩家>
```

元素参数会补全当前注册的元素，例如 `LIUHUO`、`HANSHUANG`、`LEIZHE`。
`clear` 只清除当前命令对应的元素属性，不会影响基础临时属性或另一种元素属性。

## RPG 命令 /itemcorerpg

别名 `/icrpg`，由 ItemCoreRPG 提供。

| 子命令 | 用途 | 权限 |
|--------|------|------|
| `info [玩家]` | 打开玩家属性信息 GUI | `itemcorerpg.command.info`（查看他人需 `itemcorerpg.command.info.other`） |
| `reload` | 重载配置 | `itemcorerpg.command.reload` |
| `help` | 显示帮助 | `itemcorerpg.command.help` |
