# ItemCore 附属开发 API 汇总

本文汇总 ItemCore `1.0.8` 当前提供给附属插件的公开入口。新附属应优先通过 `ItemCoreAPI` 和 `com.minemart.itemcore.api` 包开发，避免直接依赖 `manager`、`listener`、`loader`、`core` 等内部实现。

## 快速选择

| 开发需求 | 推荐接口 |
|---|---|
| 查询、创建或发放 ItemCore 物品 | `ItemCoreAPI` 物品方法 |
| 给玩家整体增加额外属性 | `AttributeProvider` |
| 给任意生物提供战斗属性 | `EntityAttributeProvider` |
| 修改某一件物品的最终属性 | `ItemAttributeProvider` |
| 修改现有属性 Lore 行 | `AttributeLoreProvider` |
| 在 `lore.yml` 插入新的 Lore 区块 | `LoreProvider` |
| 增加新的物品分类行为 | `ItemTypeSystem` |
| 造成 ItemCore 体系伤害或治疗 | `DamageRequest`、`HealingRequest` |
| 查询、恢复或消耗玩家法力 | `ItemCoreAPI` 法力方法 |
| 监听物品、技能、元素或异常状态 | Bukkit 事件接口 |
| 操作自定义耐久 | `ItemCoreAPI` 耐久方法 |

## 项目依赖

附属插件不能把 ItemCore 本体打包进自己的 JAR，应使用 `provided` 依赖。

```xml
<dependency>
    <groupId>com.minemart</groupId>
    <artifactId>itemcore</artifactId>
    <version>1.0.8</version>
    <scope>provided</scope>
</dependency>
```

本地开发时，可以先在 ItemCore 项目执行：

```powershell
mvn clean install
```

附属的 `plugin.yml` 至少声明：

```yaml
name: ExampleAddon
version: 1.0.0
main: com.example.addon.ExampleAddon
api-version: '1.21'
depend: [ItemCore]
```

只有插件在没有 ItemCore 时也能独立运行，才使用 `softdepend: [ItemCore]`，并在调用 API 前检查 ItemCore 是否启用。

## 生命周期规则

推荐在附属插件的 `onEnable()` 注册扩展，在 `onDisable()` 注销不具备自动清理能力的接口。

| 扩展类型 | 注册方法 | 注销方法 | 附属禁用时自动注销 |
|---|---|---|---|
| 玩家整体属性 | `registerAttributeProvider` | `unregisterAttributeProvider` | 否 |
| 实体战斗属性 | `registerEntityAttributeProvider` | `unregisterEntityAttributeProvider` | 否 |
| 单件物品属性 | `registerItemAttributeProvider` | `unregisterItemAttributeProvider` | 是 |
| 现有属性 Lore | `registerAttributeLoreProvider` | `unregisterAttributeLoreProvider` | 是 |
| 自定义 Lore 区块 | `registerLoreProvider` | `unregisterLoreProvider` | 是 |
| 物品类型系统 | `registerItemTypeSystem` | `unregisterItemTypeSystem` | 否 |

ItemCore 也提供 `unregisterItemAttributeExtensions(plugin)` 和 `unregisterLoreProviders(plugin)` 批量注销方法。正常插件禁用流程会自动处理这两类扩展，批量方法主要用于附属主动重载自己的模块。

`NamespacedKey` 应使用附属插件实例创建，防止不同附属使用相同键名：

```java
NamespacedKey key = new NamespacedKey(this, "enhance_attributes");
```

## ItemCoreAPI 总览

主入口类：

```java
import com.minemart.itemcore.api.ItemCoreAPI;
```

### 物品查询与发放

| 方法 | 说明 |
|---|---|
| `getCustomItem(String)` | 按完整 ID 或唯一裸 ID 获取物品，不存在或裸 ID 有歧义时返回 `null` |
| `getCustomItem(String, String)` | 按分类 ID、物品 ID 精确获取物品 |
| `getItemStack(String)` | 按完整 ID 或唯一裸 ID 构建一件新物品 |
| `getItemStack(String, int)` | 按数量构建新的物品实例 |
| `getItemStack(String, String, int)` | 按分类 ID、物品 ID 和数量构建物品 |
| `hasItem(String)` | 检查完整 ID 或唯一裸 ID 是否存在 |
| `getItemIds()` | 获取去重后的本地物品 ID |
| `getItemKeys()` | 获取全部 `分类ID:物品ID` 完整键 |
| `getItemsById(String)` | 获取全部使用指定本地 ID 的物品 |
| `isAmbiguousItemId(String)` | 判断裸 ID 是否存在于多个分类 |
| `resolveItemKey(String)` | 将完整 ID 或唯一裸 ID 解析为规范完整键 |
| `getItems()` | 获取全部 `CustomItem` |
| `getItemsByCategory(String)` | 获取分类中的物品，保持 YAML 加载顺序 |
| `giveItem(Player, String)` | 向玩家发放一件物品 |
| `giveItem(Player, String, int)` | 按数量发放物品 |
| `giveItem(Player, String, String, int)` | 按分类 ID、物品 ID 和数量精确发放物品 |

```java
CustomItem definition = ItemCoreAPI.getCustomItem("melee_weapons", "legendary_blade");
ItemStack preview = ItemCoreAPI.getItemStack("melee_weapons:legendary_blade");
boolean success = ItemCoreAPI.giveItem(player, "melee_weapons", "legendary_blade", 1);
```

### 品质定义与实例品质

品质定义来自 `ranks.yml`：

```java
ItemRank rank = ItemCoreAPI.getRank("excellent");
Collection<ItemRank> ranks = ItemCoreAPI.getRanks();

RandomRankTemplate template = ItemCoreAPI.getRandomRankTemplate("default_random");
Collection<RandomRankTemplate> templates = ItemCoreAPI.getRandomRankTemplates();
ItemRank previewRoll = ItemCoreAPI.rollRandomRank("default_random");
```

`ItemRank` 提供 `getId()`、`getLevel()` 和 `getDisplay()`。读取具体物品实例时使用：

```java
String rankId = ItemCoreAPI.getItemRankId(itemStack);
ItemRank rank = ItemCoreAPI.getItemRank(itemStack);
int rankLevel = ItemCoreAPI.getItemRankLevel(itemStack);
```

随机模板物品在真实实例构建时只抽取一次，并把结果品质 ID 写入 PDC。附属如需明确
重抽实例品质，可以使用 `ItemCoreAPI.rerollItemRank(itemStack)`，或通过
`rerollItemRank(itemStack, templateId)` 指定随机模板；调用成功后会自动刷新 Lore。

新生成物品会将品质 ID 写入 PDC，因此物品模板以后改成其它品质时，已有物品仍保持原品质 ID。
品质等级和展示文本实时读取当前 `ranks.yml`；修改同一品质的 `level` 或 `display` 会作用于已有物品。
旧版直接将展示文本写入 `rank` 的物品没有结构化品质，实例 API 返回 `null` 或 `0`。

`getItemStack` 只负责构建物品，不会触发获得事件。`giveItem` 会触发 `ItemObtainedEvent`，背包放不下的剩余物品会掉落在玩家位置。通过 API 发放时，调用方应自行处理自己的业务权限判断。

`CustomItem#getId()` 返回分类内的本地 ID，`CustomItem#getKey()` 返回完整 ID。
`ItemIdentifier#getItemId(ItemStack)` 为旧附属保留本地 ID 语义；需要可靠区分同名物品时应使用
`ItemIdentifier#getItemKey(ItemStack)` 或 `ItemIdentifier#getCustomItem(ItemStack)`。

`ItemObtainedEvent#getItemId()` 返回本地 ID，并新增 `getCategoryId()` 与 `getItemKey()`。
旧物品只有 `itemcore:item_id` 时，核心会按照旧版本最后加载生效的定义解析，并在首次识别时补写
`itemcore:item_key`。附属插件不应自行拼接或直接修改这两个 PDC 字段。

### 分类查询

| 方法 | 说明 |
|---|---|
| `getCategory(String)` | 获取分类对象 |
| `hasCategory(String)` | 检查分类是否存在 |
| `getCategoryIds()` | 获取全部分类 ID |
| `getCategories()` | 获取全部分类 |

### 注册表入口

| 方法 | 说明 |
|---|---|
| `getItemRegistry()` | 获取物品只读查询注册表 |
| `getCategoryRegistry()` | 获取分类只读查询注册表 |
| `getItemTypeRegistry()` | 获取物品类型系统注册表 |
| `getLoreProviderRegistry()` | 获取 Lore 区块注册表 |
| `getItemAttributeExtensionRegistry()` | 获取物品属性扩展注册表 |

`setItemRegistry` 和 `setCategoryRegistry` 是 ItemCore 初始化使用的一次性替换入口。普通附属不应调用，否则可能替换核心物品库实现。

## 属性 API

### AttributeContainer

`AttributeContainer` 是 ItemCore 通用属性容器。

| 方法 | 说明 |
|---|---|
| `getAttribute(CustomAttribute)` | 获取普通属性 |
| `setAttribute(CustomAttribute, double)` | 设置普通属性 |
| `addAttribute(CustomAttribute, double)` | 累加普通属性 |
| `setAttributeRange(CustomAttribute, min, max)` | 设置随机范围 |
| `getElementMastery(ElementType)` | 获取元素精通 |
| `setElementMastery(ElementType, double)` | 设置元素精通 |
| `getElementResistance(ElementType)` | 获取元素抗性 |
| `setElementResistance(ElementType, double)` | 设置元素抗性 |
| `merge(AttributeContainer)` | 合并另一个属性容器 |

通过配置风格的 Map 创建容器：

```java
AttributeContainer attributes = ItemCoreAPI.createAttributeContainer(Map.of(
    "attack_damage", 10,
    "crit_chance", "15%"
));
```

### 查询玩家最终属性

| 方法 | 说明 |
|---|---|
| `getPlayerAttributes(Player)` | 获取玩家当前全部最终属性 |
| `getAttribute(Player, CustomAttribute)` | 获取指定普通属性 |
| `getAttackDamage(Player)` | 获取攻击伤害 |
| `getSpellPower(Player)` | 获取法术强度 |
| `getAdaptiveForceValue(Player)` | 获取适应之力 |
| `getElementMastery(Player, ElementType)` | 获取元素精通 |
| `getElementResistance(Player, ElementType)` | 获取元素抗性 |
| `refreshPlayerAttributes(Player)` | 立即重算生命、法力上限、移动速度、攻击速度和被动物品效果 |

### 法力值

| 方法 | 说明 |
|---|---|
| `isManaEnabled()` | 查询法力系统是否启用 |
| `getMana(Player)` | 获取当前法力值 |
| `getMaxMana(Player)` | 获取最大法力值 |
| `getManaRegeneration(Player)` | 获取每秒自然恢复量 |
| `getManaPercent(Player)` | 获取 `0-100` 的法力百分比 |
| `setMana(Player, double)` | 设置并限制当前法力值，返回实际值 |
| `addMana(Player, double)` | 增减当前法力值，返回实际值 |
| `consumeMana(Player, double)` | 法力充足时扣除并返回 `true` |
| `restoreFullMana(Player)` | 将当前法力恢复至最大值 |

当前法力保存在玩家 PDC 中。法力系统关闭时，查询方法返回 `0`，修改和消耗方法不生效。Bukkit 玩家数据应只在服务器主线程操作。

### 查询实体最终属性

| 方法 | 说明 |
|---|---|
| `getEntityAttributes(LivingEntity)` | 获取任意生物的合并属性；玩家仍走原有装备、套装和附属计算 |
| `getEntityAttribute(LivingEntity, CustomAttribute)` | 获取实体指定普通属性 |
| `getEntityElementMastery(LivingEntity, ElementType)` | 获取实体元素精通 |
| `getEntityElementResistance(LivingEntity, ElementType)` | 获取实体元素抗性 |

没有匹配 Provider 的非玩家实体返回空容器，各项数值均为 `0`。

### 玩家整体属性提供器

`AttributeProvider` 适合称号、职业、天赋等不属于某一件物品的玩家属性。

```java
AttributeProvider provider = player -> {
    AttributeContainer result = new AttributeContainer();
    result.addAttribute(CustomAttribute.HEALTH, 20);
    return result;
};

ItemCoreAPI.registerAttributeProvider(provider);
```

该接口没有所有者参数，也不会自动注销。附属禁用时必须调用：

```java
ItemCoreAPI.unregisterAttributeProvider(provider);
```

`getAttributeProviders()` 可以查看当前已注册的玩家整体属性提供器；不要直接修改返回列表，应使用注册和注销方法维护生命周期。

该提供器处于高频属性计算路径，不应执行数据库、文件或网络操作，也不应抛出异常。

### 玩家临时属性

命令与附属可以为在线玩家添加仅在内存中存在的临时属性。固定值先加入玩家当前属性，百分比效果随后逐层乘算。`adaptive_force` 的百分比会在适应之力转换前生效，其余百分比在适应之力和异常属性修改后生效：

```java
ItemCoreAPI.addTemporaryAttribute(
    player,
    CustomAttribute.ATTACK_DAMAGE,
    -20,
    600,
    TemporaryAttributeOperation.PERCENT,
    TemporaryAttributeMode.STACK
);
```

`STACK` 保留同属性的已有临时效果；`REPLACE` 会先移除该属性的全部固定与百分比临时效果。移除接口：

```java
ItemCoreAPI.removeTemporaryAttributes(player, CustomAttribute.ATTACK_DAMAGE);
ItemCoreAPI.clearTemporaryAttributes(player);
```

临时属性不会写入物品、玩家 PDC 或配置文件，服务器重启后清除。负数是否产生实际负值由对应属性的既有计算规则决定；`ARMOR` 会同步为原版护甲修饰符，并遵循 `combat.disable_vanilla_armor`。

元素精通和元素抗性使用独立的元素属性类型：

```java
ItemCoreAPI.addTemporaryElementAttribute(
    player,
    ElementAttributeType.MASTERY,
    ElementType.LIUHUO,
    10,
    600,
    TemporaryAttributeOperation.ADD,
    TemporaryAttributeMode.STACK
);

ItemCoreAPI.removeTemporaryElementAttributes(
    player, ElementAttributeType.MASTERY, ElementType.LIUHUO);
ItemCoreAPI.clearTemporaryElementAttributes(
    player, ElementAttributeType.RESISTANCE);
```

元素临时属性与普通临时属性分别存储。`REPLACE` 只替换同一种元素属性和同一个元素的效果；
`clearTemporaryElementAttributes` 只清除指定的 `MASTERY` 或 `RESISTANCE`。

### 实体战斗属性提供器

`EntityAttributeProvider` 供怪物、NPC 等外部实体系统接入 ItemCore 伤害、抗性和元素积累计算。

```java
EntityAttributeProvider provider = new EntityAttributeProvider() {
    @Override
    public boolean supports(LivingEntity entity) {
        return externalMobManager.isManaged(entity.getUniqueId());
    }

    @Override
    public AttributeContainer getAttributes(LivingEntity entity) {
        AttributeContainer result = new AttributeContainer();
        result.setAttribute(CustomAttribute.PHYSICAL_RESIST, 100);
        result.setElementResistance(ElementType.LIUHUO, 20);
        return result;
    }
};

ItemCoreAPI.registerEntityAttributeProvider(provider);
```

多个匹配 Provider 的结果会相加合并。每次调用应返回新容器，不要复用之后仍会修改的实例；无属性时可返回 `null` 或空容器。单个 Provider 抛出异常时 ItemCore 会忽略该次贡献并继续伤害计算。附属禁用时必须调用 `unregisterEntityAttributeProvider(provider)`，`getEntityAttributeProviders()` 返回当前注册列表的只读快照。

### 单件物品最终属性提供器

`ItemAttributeProvider` 用于强化、镶嵌、品质、词缀等单件物品数据。

```java
public final class EnhanceProvider implements ItemAttributeProvider {
    private final NamespacedKey key;

    public EnhanceProvider(Plugin plugin) {
        this.key = new NamespacedKey(plugin, "enhance_attributes");
    }

    @Override
    public NamespacedKey getKey() {
        return key;
    }

    @Override
    public int getPriority() {
        return 100;
    }

    @Override
    public void modify(ItemAttributeContext context, AttributeContainer attributes) {
        int level = readLevel(context.getItemStack());
        attributes.addAttribute(CustomAttribute.ATTACK_DAMAGE, level * 2.0);
    }
}
```

```java
ItemCoreAPI.registerItemAttributeProvider(this, new EnhanceProvider(this));
```

计算顺序为：物品固定值或随机 PDC 值 → 低优先级提供器 → 高优先级提供器。提供器修改的是新的工作副本，不会覆盖原始随机值。

`ItemAttributeContext` 提供：

- `getItem()`：物品配置对象。
- `getItemStack()`：物品克隆，可能为 `null`。
- `getViewer()`：相关玩家，可能为 `null`。
- `getSlot()`：生效槽位，非装备计算时可能为 `null`。

查询单件物品最终属性：

```java
AttributeContainer finalAttributes = ItemCoreAPI.getItemAttributes(itemStack, player);
```

只读取物品生成时抽取的随机属性，不包含基础属性和附属修改：

```java
AttributeContainer randomAttributes =
        ItemCoreAPI.getRolledRandomAttributes(itemStack);
```

按物品配置中的指定随机属性范围重新抽取一个数值：

```java
OptionalDouble rerolled = customItem.getRandomAttributes()
        .rollValue(CustomAttribute.ATTACK_DAMAGE);
if (rerolled.isPresent()) {
    double value = rerolled.getAsDouble();
}
```

`rollValue(CustomAttribute)` 不参与权重选择，也不受 `count` 影响，只读取指定属性的
`min/max` 并生成一次精确到一位小数的结果。指定属性未配置时返回
`OptionalDouble.empty()`；该方法只返回新数值，不会自动覆盖物品 PDC 或刷新 Lore。

附属 PDC 发生变化后调用：

```java
ItemCoreAPI.refreshItemAttributes(itemStack, player);
```

该方法会同步原版护甲修饰符、属性 Lore，并立即刷新玩家属性。

### 现有属性 Lore 提供器

`AttributeLoreProvider` 用于覆盖 `#attack_damage#` 等现有属性占位符生成的行。

```java
public String render(AttributeLoreContext context) {
    if (context.getAttribute() != CustomAttribute.ATTACK_DAMAGE) {
        return null;
    }
    return context.getDefaultLine() + " &8(强化后)";
}
```

```java
ItemCoreAPI.registerAttributeLoreProvider(this, provider);
```

返回规则：

- 返回 `null`：当前提供器不处理该属性。
- 返回空字符串：隐藏该属性行。
- 返回文本：替换默认属性行。
- 高优先级提供器优先获得覆盖权。

`AttributeLoreContext#getValue()` 是经过全部物品属性提供器处理后的最终值。

## Lore 区块 API

`LoreProvider` 用于增加独立 Lore 区块，不负责替换现有属性行。

```java
public final class LevelLoreProvider implements LoreProvider {
    private final NamespacedKey key;

    public LevelLoreProvider(Plugin plugin) {
        this.key = new NamespacedKey(plugin, "level");
    }

    @Override
    public NamespacedKey getKey() {
        return key;
    }

    @Override
    public List<String> render(LoreContext context) {
        return List.of("&7等级: &f10");
    }
}
```

```java
ItemCoreAPI.registerLoreProvider(this, new LevelLoreProvider(this));
```

服务器管理员在 `tooltip/lore.yml` 放置：

```yaml
lore_format:
  - '{bar}&7等级信息'
  - '#ext:exampleaddon:level#'
```

提供器返回 `null` 或空列表时隐藏区块，并且不会触发前面的条件 `{bar}`。

`LoreContext` 提供物品配置、物品克隆、查看者以及渲染原因：

| `LoreRenderReason` | 含义 |
|---|---|
| `ITEM_BUILD` | 初次构建物品 |
| `DURABILITY_CHANGE` | 耐久发生变化 |
| `CONFIG_RELOAD` | 配置版本刷新 |
| `ADDON_UPDATE` | 附属主动刷新 |
| `GUI_PREVIEW` | ItemCore GUI 预览 |

`GUI_PREVIEW` 下，ItemCore 内置属性 Lore 的 `defaultLine` 会将配置范围完整显示为
`min-max`；`AttributeLoreContext#getValue()` 表示最终预览范围的中点值。

刷新接口：

| 方法 | 说明 |
|---|---|
| `refreshItemLore(ItemStack)` | 刷新单件物品，不提供玩家上下文 |
| `refreshItemLore(ItemStack, Player)` | 刷新单件物品并提供查看者 |
| `refreshPlayerLore(Player)` | 强制刷新玩家背包中的 ItemCore 物品 |
| `invalidateAllLore()` | 提升全局 Lore 版本，交由定时刷新逐步更新 |

## 物品分类行为系统 API

`ItemTypeSystem` 用于为 `categories.yml` 中的分类 ID 增加交互、攻击或技能规则。
接口名称为兼容现有附属而保留；物品配置中的 `display_type` 只用于 Lore 展示。

```java
public final class WandTypeSystem implements ItemTypeSystem {
    @Override
    public String getTypeId() {
        return "arcane_wand";
    }

    @Override
    public void onInteract(ItemTypeInteractContext context) {
        Object manaCost = context.getItem().getTypeSetting("mana_cost");
        boolean lifeGem = context.getItem().hasTypeSetting("生命");
    }
}
```

```java
ItemCoreAPI.registerItemTypeSystem(new WandTypeSystem());
```

| 回调 | 用途 |
|---|---|
| `onInteract` | 处理左右键交互 |
| `beforeAttack` | ItemCore 计算攻击结果前处理 |
| `contributesAttackAttributes` | 决定某槽位物品是否参与本次攻击属性 |
| `afterAttack` | 攻击结果生成后处理 |
| `canTriggerSkill` | 决定该分类是否允许技能触发 |
| `afterSkillTrigger` | 技能成功通过前置检查后的后处理 |

分类 ID 重复注册时返回 `false`。分类可在 `categories.yml` 使用 `parent` 继承父分类系统；父系统先执行，再执行子系统，展示配置不会继承。`contributesAttackAttributes` 和 `canTriggerSkill` 对完整继承链采用 AND。该接口不会自动注销：

```java
ItemCoreAPI.unregisterItemTypeSystem("arcane_wand");
```

分类关系可通过 `getParentCategory`、`getChildCategories` 和 `getCategoryAncestors` 查询；`ItemTypeRegistry#getSystemChain` 返回实际按执行顺序解析出的系统链。

`ItemTypeSkillContext#consumeOne()` 可以从玩家背包中消耗一个对应 ItemCore 物品。

## 伤害与治疗 API

### 自定义伤害

```java
DamageRequest request = DamageRequest.builder()
    .attacker(attacker)
    .victim(victim)
    .baseDamage(100)
    .damageType(DamageTag.SPELL)
    .element(ElementType.LIUHUO)
    .canCrit(true)
    .critChance(20)
    .critDamage(180)
    .penetration(10)
    .attackType(AttackType.SKILL)
    .lifesteal(5)
    .castId(UUID.randomUUID())
    .build();

ItemCoreAPI.processDamage(request);
```

`DamageRequest` 默认值：物理伤害、无元素、允许暴击、暴击率 `0`、暴击伤害 `150`、穿透 `0`、技能攻击、吸血 `0`。

`AttackType.ATTACK` 表示普通攻击规则，`AttackType.SKILL` 表示技能传入的伤害规则。

### 自定义治疗

```java
HealingRequest request = HealingRequest.builder()
    .healer(caster)
    .target(target)
    .amount(20)
    .castId(castId)
    .build();

ItemCoreAPI.processHeal(request);
```

相同 `castId` 可供伤害、治疗或附属逻辑标识同一次技能释放。

## 元素与异常状态 API

| 方法 | 说明 |
|---|---|
| `getElementIcon(String)` | 获取元素图标，缺失时返回默认符号 |
| `getElementColor(String)` | 获取元素颜色，缺失时返回白色 |
| `getElementProgress(LivingEntity, ElementType)` | 获取元素积累快照 |
| `addElementProgress(LivingEntity, ElementType, double)` | 增加元素积累 |
| `clearElementProgress(LivingEntity, ElementType)` | 清除指定元素积累 |
| `hasAilment(LivingEntity, String)` | 检查异常状态 |
| `applyElementDamage(LivingEntity, DamageContext)` | 将一次元素伤害计入积累系统 |

注册自定义元素：

```java
ItemCore.getElementRegistry()
    .register(new ElementType("ARCANE", "奥术"));
```

自定义元素应在依赖它的物品、属性或技能配置加载前注册。

## 耐久 API

| 方法 | 说明 |
|---|---|
| `hasDurability(ItemStack)` | 是否启用了 ItemCore 自定义耐久 |
| `getDurability(ItemStack)` | 获取当前耐久 |
| `getMaxDurability(ItemStack)` | 获取最大耐久 |
| `setDurability(ItemStack, int)` | 直接设置当前耐久 |
| `damageItem(Player, ItemStack, int)` | 扣除耐久并执行损坏、销毁和 Lore 刷新规则 |
| `repairItem(ItemStack, int)` | 修复指定耐久 |

附属不要直接修改 ItemCore 的耐久 PDC，应通过这些方法保证耐久条、Lore 和损坏状态同步。

## 物品绑定 API

物品配置 `bind: true` 时，物品首次进入玩家背包即绑定给该玩家；非主人拿到后物品只相当于普通物品。
ItemCore 只拦截丢弃、死亡掉落和拾取，**不会**拦截附属插件通过 API 完成的转移，
交易、仓库、邮件类附属必须在转移前自行校验。

| 方法 | 说明 |
|---|---|
| `isBindableItem(ItemStack)` | 物品定义是否开启了绑定且绑定系统已启用 |
| `isItemBound(ItemStack)` | 物品实例是否已经绑定 |
| `getItemBindOwner(ItemStack)` | 绑定主人 UUID，未绑定为 `null` |
| `getItemBindOwnerName(ItemStack)` | 绑定主人名字，用于显示 |
| `isItemBindOwner(Player, ItemStack)` | 玩家是否为该物品主人 |
| `canUseBoundItem(Player, ItemStack)` | 玩家能否把物品当作 ItemCore 物品使用；未开启绑定或未绑定返回 `true` |
| `bindItem(ItemStack, Player)` | 绑定给在线玩家 |
| `bindItem(ItemStack, UUID, String)` | 绑定给指定 UUID（可离线） |
| `unbindItem(ItemStack)` | 解除绑定，物品再进入玩家背包时会重新绑定 |
| `bindPlayerInventory(Player)` | 扫描背包并绑定所有未绑定的可绑定物品 |

交易插件推荐写法：

```java
if (ItemCoreAPI.isItemBound(itemStack)) {
    player.sendMessage("绑定物品无法交易");
    return;
}
```

绑定、换绑和解绑前都会触发可取消的 `ItemBindEvent`，`getReason()` 区分
`OBTAIN`、`PICKUP`、`INVENTORY_SCAN`、`COMMAND`、`API`。

## Bukkit 事件

所有事件均通过 Bukkit `PluginManager` 监听。

| 事件 | 可取消 | 可修改内容 | 用途 |
|---|---|---|---|
| `ItemObtainedEvent` | 是 | 数量、物品实例 | GUI、命令或 API 发放物品前 |
| `ItemBindEvent` | 是 | 取消绑定/解绑 | 物品即将绑定、换绑或解绑时 |
| `ItemSkillTriggerEvent` | 是 | 目标、位置 | ItemCore 物品技能触发时 |
| `ItemCoreDamageEvent` | 是 | `DamageResult` 中的伤害 | ItemCore 攻击结果生成后 |
| `ItemCoreDamageAppliedEvent` | 否 | 只读 | 正数伤害通过最终取消检查并已交由 Bukkit 应用后 |
| `ElementAccumulationEvent` | 否 | 只读 | 元素积累数值变化 |
| `AilmentTriggerEvent` | 是 | 取消触发 | 异常状态即将触发 |
| `AilmentExpireEvent` | 否 | 只读 | 异常状态结束 |

`ItemCoreDamageEvent#getAttackType()` 用于区分 `ATTACK` 与 `SKILL`；
`ItemCoreDamageEvent#getDamageOrigin()` 用于区分伤害来源。其中通过 Paper 玩家攻击意图校验的
直接近战击打为 `ATTACK + VANILLA_MELEE`；普通插件构造的玩家直伤为
`ATTACK + BUKKIT_DIRECT`，玩家射弹与技能伤害为 `SKILL`。需要实现“成功普攻后触发”的
附属应监听 `ItemCoreDamageAppliedEvent`，并同时判断：

```java
event.getAttackType() == AttackType.ATTACK
    && event.getDamageOrigin() == DamageOrigin.VANILLA_MELEE
    && event.getFinalDamage() > 0.0
```

属性提供者需要根据其他属性来源判断条件时，使用
`ItemCoreAPI.getPlayerAttributesExcludingProvider(player, provider)` 排除自身，避免递归调用。随后可以将快照
传给 `ItemCoreAPI.getManaPercent(player, attributes)`，该重载不会再次计算玩家属性。需要从快照读取最大法力时，使用
`ItemCoreAPI.getMaxManaFromAttributes(attributes)`。

```java
@EventHandler
public void onItemObtained(ItemObtainedEvent event) {
    if (event.getSource() == ItemObtainedEvent.ObtainSource.API) {
        event.setAmount(Math.max(1, event.getAmount()));
    }
}
```

`ItemObtainedEvent.ObtainSource` 当前包含 `GUI`、`COMMAND`、`API`。

## CustomItem 常用读取接口

`CustomItem` 是加载后的物品定义，不代表玩家背包中的某一个实例。

| 分类 | 常用方法 |
|---|---|
| 身份 | `getId()`、`getDisplayType()`、`getCategoryId()`、`getSetId()` |
| 显示 | `getMaterial()`、`getDisplayName()`、`getRank()`、`hasRank()`、`getLore()`、`getColor()` |
| 品质 | `getRankId()`、`getRankLevel()`、`getRankDefinition()`、`getRandomRankTemplateId()`、`getRandomRankTemplate()`、`hasRandomRank()`、`hasRank(String)`、`isRankAtLeast(int)` |
| 属性 | `getAttributes()`、`getRandomAttributes()`、`hasRandomAttributes()`、`getActiveSlots()`、`canSlot(ItemSlot)` |
| 耐久 | `hasDurability()`、`getDurability()`、`isDurabilityBreak()` |
| 限制 | `getPermission()`、`hasPermission()`、`isDisableAnvilRepair()`、`isDisableEnchanting()`、`isEnchantmentGlint()`、`isBindable()` |
| 技能 | `getSkills()`、`hasSkills()`、`getConsumeTrigger()` |
| 类型参数 | `getTypeSettings()`、`getTypeSetting(String)`、`hasTypeSetting(String)` |
| 构建实例 | `toItemStack()`、`toItemStack(int)` |

随机品质、随机属性、当前耐久、强化等级和镶嵌数据属于 `ItemStack` 实例数据，不能只读取 `CustomItem` 判断最终结果。

## 开发约定与性能

1. Bukkit 实体、背包和 `ItemStack` 修改应在服务器主线程执行。
2. `LoreContext`、`ItemAttributeContext` 返回的 `ItemStack` 是克隆，只用于安全读取。
3. 属性提供器和 Lore 提供器可能被频繁调用，不要在回调中同步访问数据库、文件或网络。
4. 附属数据应保存在附属自己的 `NamespacedKey` PDC 中，不要覆盖 ItemCore 的内部键。
5. 单件物品数据变化后优先调用 `refreshItemAttributes`；仅改变额外 Lore 时调用 `refreshItemLore`。
6. 大规模配置变化时调用 `invalidateAllLore`，避免一次性同步刷新所有在线玩家。
7. 新增扩展接口时优先使用 `ItemCoreAPI` 注册，不要直接修改 ItemCore 的内部管理器集合。

## 强化插件推荐流程

1. 在物品 PDC 中保存强化等级和附属自己的随机数据。
2. 使用 `ItemAttributeProvider` 根据 PDC 修改最终属性副本。
3. 如需显示“已强化”等附加信息，注册 `AttributeLoreProvider`。
4. 如需增加独立强化区块，注册 `LoreProvider` 并在 `lore.yml` 放置 `#ext:命名空间:键#`。
5. 强化、降级或拆卸强化石后调用 `ItemCoreAPI.refreshItemAttributes(itemStack, player)`。
6. 不要把最终属性覆盖回 ItemCore 的原始随机属性 PDC，否则会丢失基础值并增加降级难度。

更完整的扩展实现示例参见 [开发者 API](api.md)。
