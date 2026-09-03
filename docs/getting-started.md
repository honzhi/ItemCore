# 安装与配置

## 安装步骤

1. 下载 `ItemCore.jar`，按需下载 `ItemCoreMythic.jar`、`ItemCoreRPG.jar` 等拓展
2. 放入 `plugins/` 目录（需要 Paper 1.21.x 与 Java 21）
3. （可选）安装 PlaceholderAPI（占位符）、BetterHud（法力不足 HUD 提示）
4. 重启服务器
5. 插件首次启动会自动生成配置文件

## 目录结构

```
plugins/ItemCore/
├── config.yml              # 主配置
├── attributes.yml          # 属性全局参数（暴击、适应之力、减伤公式、穿透顺序）
├── messages.yml            # 消息配置
├── categories.yml          # 分类配置
├── ranks.yml               # 品质配置
├── sets.yml                # 套装配置
├── elements.yml            # 元素配置
├── ailments.yml            # 异常配置
├── items/                  # 物品配置目录（文件名由 categories.yml 的 items_file 决定）
│   ├── melee_weapons.yml
│   ├── ranged_weapons.yml
│   ├── armor.yml
│   ├── material.yml
│   ├── consumable.yml
│   ├── blocks.yml
│   └── tools.yml
└── tooltip/
    ├── lore.yml            # Lore 布局
    └── stats.yml           # 属性显示格式
```

## config.yml 主配置

```yaml
language: "zh-CN"          # zh-CN / en-US
debug_mode: false          # 调试日志

combat:
  # 禁用全服原版护甲/护甲韧性减伤；开启后 ARMOR 属性也失效，请改用 PHYSICAL_RESIST
  # 保护类附魔不受影响
  disable_vanilla_armor: false

# 物品全局保护（物品配置中写同名键可覆盖）
Item_global_settings:
  disable_anvil: true            # 禁止铁砧操作（修复/合并/改名）
  disable_enchant: true          # 禁止附魔（附魔台与铁砧附魔书合并）
  disable_crafting_table: true   # 禁止作为原版合成材料

# 法力值系统
mana:
  enabled: false
  base: 100                  # 基础最大法力
  base_regeneration: 1       # 每秒基础恢复
  not_enough_display: chat   # chat / hud / both
  hud:                       # hud 模式需要 BetterHud
    popup: itemcore_mana_warning
    duration: 30             # tick
    gui_x: 50
    gui_y: 60
    scale: 0.8
    message: "<red>法力不足 <gray>(<aqua>{current}</aqua>/<aqua>{cost}</aqua>)</gray></red>"

# 物品绑定系统
bind:
  enabled: true              # 关闭后所有 bind: true 的物品视为普通物品
  notify_on_bind: true       # 绑定时发送聊天提示
  keep_on_death: true        # 绑定物品死亡不掉落

gui:
  name: "物品库"

# Lore 定时刷新（实验性）：让 /ic reload 后玩家背包内物品的 Lore 自动更新
lore_refresh:
  enabled: false
  interval: 100              # tick
```

## attributes.yml 属性参数

```yaml
crit:
  default_crit_damage: 150      # 默认暴击伤害百分比（150 = 1.5 倍）
  cooldown:                     # 暴击冷却，仅对 ATTACK 类型伤害生效
    enabled: false
    mode: FIXED                 # FIXED = 固定冷却 / FULL_CHARGE = 满蓄力才能暴击
    fixed_ticks: 20
    full_charge_threshold: 0.95

adaptive_force:
  attack_conversion: 1.0        # 每点适应之力转化的攻击伤害
  spell_conversion: 1.0         # 每点适应之力转化的法术强度

defense_formulas:               # 可用变量 {damage} {armor}
  physical_resist: '{damage} * (1 - {armor} / ({armor} + 100))'
  spell_resist: '{damage} * (1 - {armor} / ({armor} + 100))'

penetration_order: percent_first  # percent_first / flat_first
```

## 重载配置

修改配置后运行：

```
/ic reload
```

无需重启服务器。已生成物品的随机属性、随机品质、耐久、绑定信息保存在物品数据中，重载不会改变。
