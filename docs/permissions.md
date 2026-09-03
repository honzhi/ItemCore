# 权限系统

| 权限节点 | 默认 | 说明 |
|----------|------|------|
| `itemcore.admin` | OP | 管理权限（包含以下全部） |
| `itemcore.command.help` | 所有人 | 查看帮助 |
| `itemcore.command.gui` | OP | 打开物品库 GUI |
| `itemcore.command.give` | OP | 给予物品 |
| `itemcore.command.attribute` | OP | 管理玩家临时属性 |
| `itemcore.command.element_attribute` | OP | 管理玩家临时元素属性 |
| `itemcore.command.bind` | OP | 更换或解除物品绑定对象 |
| `itemcore.command.reload` | OP | 重载配置 |
| `itemcore.gui.obtain` | OP | 从 GUI 获取物品 |

`plugin.yml` 中仍保留 `itemcore.command.list`、`itemcore.command.info`、`itemcore.command.repair` 三个旧权限，对应命令已移除或未注册，这些权限当前没有作用。

物品与分类还可配置各自的 `permission` 字段：缺少权限的玩家无法从 GUI 获取该物品，物品的属性、药水效果、技能、套装件数也不会生效。
