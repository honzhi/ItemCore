# 开发者 API

需要按功能快速查找全部公开入口时，请参阅 [附属开发 API 汇总](addon-api.md)。

## 获取 API 实例

```java
import com.minemart.itemcore.api.ItemCoreAPI;

// 获取玩家属性
AttributeContainer attrs = ItemCoreAPI.getPlayerAttributes(player);
double attackDamage = attrs.getAttribute(CustomAttribute.ATTACK_DAMAGE);

// 获取物品；推荐始终提供分类
CustomItem item = ItemCoreAPI.getCustomItem("melee_weapons", "legendary_blade");
CustomItem sameItem = ItemCoreAPI.getCustomItem("melee_weapons:legendary_blade");

// 读取品质定义与物品实例品质
ItemRank excellent = ItemCoreAPI.getRank("excellent");
String rankId = ItemCoreAPI.getItemRankId(itemStack);
int rankLevel = ItemCoreAPI.getItemRankLevel(itemStack);

// 获取分类
Collection<ItemCategory> categories = ItemCoreAPI.getCategories();

// 处理自定义伤害
DamageRequest request = DamageRequest.builder()
    .attacker(attacker)
    .victim(victim)
    .baseDamage(100)
    .damageType(DamageTag.SPELL)
    .element(ElementType.LIUHUO)
    .canCrit(true)
    .attackType(AttackType.SKILL)
    .build();
ItemCoreAPI.processDamage(request);
```

## 品质 API

`ranks.yml` 中的每个品质都会加载为不可变的 `ItemRank`：

```java
ItemRank rank = ItemCoreAPI.getRank("excellent");
if (rank != null) {
    String id = rank.getId();
    int level = rank.getLevel();
    String display = rank.getDisplay();
}

Collection<ItemRank> ranks = ItemCoreAPI.getRanks();
```

随机模板会加载为不可变的 `RandomRankTemplate`：

```java
RandomRankTemplate template = ItemCoreAPI.getRandomRankTemplate("default_random");
Collection<RandomRankTemplate> templates = ItemCoreAPI.getRandomRankTemplates();
ItemRank previewRoll = ItemCoreAPI.rollRandomRank("default_random");
```

读取具体 `ItemStack` 时应优先使用实例 API，因为物品生成后会固定保存品质 ID：

```java
String rankId = ItemCoreAPI.getItemRankId(itemStack);
ItemRank rank = ItemCoreAPI.getItemRank(itemStack);
int level = ItemCoreAPI.getItemRankLevel(itemStack);
```

读取物品模板时可以使用：

```java
CustomItem item = ItemCoreAPI.getCustomItem("armor", "coal_helmet");
String rankId = item.getRankId();
int level = item.getRankLevel();
ItemRank definition = item.getRankDefinition();

boolean excellent = item.hasRank("excellent");
boolean atLeastRare = item.isRankAtLeast(3);

String randomTemplateId = item.getRandomRankTemplateId();
RandomRankTemplate randomTemplate = item.getRandomRankTemplate();
boolean randomRank = item.hasRandomRank();
```

`CustomItem#getRank()` 为兼容旧附属继续返回展示文本，不是品质 ID。
随机品质模板没有单一的模板品质，因此生成实例前 `getRankId()` 和 `getRankLevel()`
分别返回 `null` 和 `0`。如需主动重抽物品实例品质，可以调用：

```java
ItemRank rolled = ItemCoreAPI.rerollItemRank(itemStack);
ItemRank specified = ItemCoreAPI.rerollItemRank(itemStack, "default_random");
```

## 法力值 API

```java
double currentMana = ItemCoreAPI.getMana(player);
double maxMana = ItemCoreAPI.getMaxMana(player);
double regeneration = ItemCoreAPI.getManaRegeneration(player);

if (ItemCoreAPI.consumeMana(player, 20)) {
    // 执行需要法力的附属逻辑
}

ItemCoreAPI.addMana(player, 10);
ItemCoreAPI.restoreFullMana(player);
```

`setMana`、`addMana` 和 `restoreFullMana` 返回限制到 `0` 与最大法力值之间的实际结果。法力系统关闭时，查询返回 `0`，消耗返回 `false`。

## 注册自定义元素

```java
ItemCore.getInstance().getElementRegistry()
    .register(new ElementType("ARCANE", "奥术"));
```

## 注册物品类型系统

外部插件应在 `plugin.yml` 中声明 `depend: [ItemCore]`，然后为 `categories.yml`
中的分类 ID 注册行为系统。接口名称保留 `ItemTypeSystem` 以兼容现有附属，但物品
配置中的 `display_type` 只用于 Lore，不参与系统分派：

```java
public final class ArcaneWandSystem implements ItemTypeSystem {
    @Override
    public String getTypeId() {
        return "arcane_wand";
    }

    @Override
    public void onInteract(ItemTypeInteractContext context) {
        Object manaCost = context.getItem().getTypeSetting("mana_cost");
        boolean lifeGem = context.getItem().hasTypeSetting("生命");
        // 读取 type_settings 并执行该分类自己的交互逻辑
    }

    @Override
    public boolean contributesAttackAttributes(ItemTypeAttackContext context,
                                               CustomItem item,
                                               ItemSlot slot) {
        return true;
    }
}

boolean registered = ItemCoreAPI.registerItemTypeSystem(new ArcaneWandSystem());
```

每个分类 ID 同时只对应一个系统。子分类通过 `categories.yml` 的 `parent` 继承父分类系统，父系统先于子系统执行；图标、槽位、Lore 和 `items_file` 不继承。注册重复 ID 会返回 `false`；插件卸载时可调用：

```java
ItemCoreAPI.unregisterItemTypeSystem("arcane_wand");
```

可扩展入口包括物品交互、攻击前后、攻击属性贡献判断，以及技能触发前后。`contributesAttackAttributes` 和 `canTriggerSkill` 在整条继承链上采用 AND。处理器异常会被 ItemCore 隔离并记录，不会继续影响其他分类系统。

## 扩展物品 Lore

附属插件可以注册一个带命名空间的 Lore 提供器，再由服务器管理员决定它在 `tooltip/lore.yml` 中的位置。

```java
import com.minemart.itemcore.api.ItemCoreAPI;
import com.minemart.itemcore.api.lore.LoreContext;
import com.minemart.itemcore.api.lore.LoreProvider;
import org.bukkit.NamespacedKey;
import org.bukkit.plugin.Plugin;

import java.util.List;

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
        if (context.getViewer() == null) {
            return List.of("&7等级: &f未知");
        }
        return List.of("&7等级: &f10");
    }
}
```

在附属插件的 `onEnable()` 中注册：

```java
ItemCoreAPI.registerLoreProvider(this, new LevelLoreProvider(this));
```

在 `tooltip/lore.yml` 中放置对应区块：

```yaml
lore_format:
  - '{bar}&7等级信息'
  - '#ext:itemcorelevel:level#'
```

- 键格式为 `#ext:命名空间:键#`，并且必须单独占一行。
- `render` 返回空列表或 `null` 时不显示该区块。
- `LoreContext#getItemStack()` 返回克隆，不能直接修改真实物品。
- `LoreContext#getViewer()` 在初次构建物品时可能为 `null`，玩家背包刷新时会提供玩家。
- 提供器抛出异常时只会跳过当前区块，不会阻止物品生成。
- 附属插件禁用时，ItemCore 会自动注销它注册的全部提供器。

附属数据变化后，可以主动刷新单件物品或玩家背包：

```java
ItemCoreAPI.refreshItemLore(itemStack);
ItemCoreAPI.refreshItemLore(itemStack, player); // 玩家相关 Lore
ItemCoreAPI.refreshPlayerLore(player);
```

如果大量玩家物品都需要刷新，可提升全局 Lore 版本，让已启用的定时刷新任务逐步处理：

```java
ItemCoreAPI.invalidateAllLore();
```

## 扩展物品最终属性

`ItemAttributeProvider` 用于修改单件物品的最终属性。ItemCore 会先读取物品的固定值或已随机出的 PDC 数值，再依次调用提供器，因此不会覆盖或丢失原始随机属性。

```java
import com.minemart.itemcore.api.ItemCoreAPI;
import com.minemart.itemcore.api.attribute.ItemAttributeContext;
import com.minemart.itemcore.api.attribute.ItemAttributeProvider;
import com.minemart.itemcore.item.attribute.AttributeContainer;
import com.minemart.itemcore.item.attribute.CustomAttribute;
import org.bukkit.NamespacedKey;

public final class EnhanceAttributeProvider implements ItemAttributeProvider {
    private final NamespacedKey key;

    public EnhanceAttributeProvider(Plugin plugin) {
        this.key = new NamespacedKey(plugin, "enhance_attributes");
    }

    @Override
    public NamespacedKey getKey() {
        return key;
    }

    @Override
    public void modify(ItemAttributeContext context, AttributeContainer attributes) {
        int level = readEnhanceLevel(context.getItemStack());
        attributes.addAttribute(CustomAttribute.ATTACK_DAMAGE, level * 2.0);
    }
}
```

注册提供器：

```java
ItemCoreAPI.registerItemAttributeProvider(
    this,
    new EnhanceAttributeProvider(this)
);
```

例如物品原始攻击力为 `10`，强化等级提供 `+2`，Lore、战斗计算和 `ItemCoreAPI.getItemAttributes(itemStack)` 得到的最终攻击力都会是 `12`。

- 提供器收到的是新的可修改属性副本，不会直接修改物品配置。
- `ItemAttributeContext#getItemStack()` 返回物品克隆，可安全读取附属插件自己的 PDC。
- 数字越大的优先级越晚执行，适合在其他提供器之后覆盖最终结果。
- 可以修改普通属性、元素精通和元素抗性。
- 提供器异常会被隔离，不会中断其他属性计算。

## 扩展现有属性 Lore

`AttributeLoreProvider` 专门用于覆盖 `#attack_damage#` 等现有属性占位符生成的 Lore 行。

```java
import com.minemart.itemcore.api.attribute.AttributeLoreContext;
import com.minemart.itemcore.api.attribute.AttributeLoreProvider;

public final class EnhanceAttributeLoreProvider implements AttributeLoreProvider {
    private final NamespacedKey key;

    public EnhanceAttributeLoreProvider(Plugin plugin) {
        this.key = new NamespacedKey(plugin, "enhance_attribute_lore");
    }

    @Override
    public NamespacedKey getKey() {
        return key;
    }

    @Override
    public String render(AttributeLoreContext context) {
        if (context.getAttribute() != CustomAttribute.ATTACK_DAMAGE) {
            return null;
        }
        return context.getDefaultLine() + " &8(已强化)";
    }
}
```

注册 Lore 提供器：

```java
ItemCoreAPI.registerAttributeLoreProvider(
    this,
    new EnhanceAttributeLoreProvider(this)
);
```

- `context.getValue()` 是经过所有物品属性提供器处理后的最终数值。
- 返回 `null` 表示不处理当前属性，继续使用其他提供器或默认 Lore。
- 返回空字符串表示隐藏当前属性行。
- 数字越大的优先级越先获得覆盖权。

强化等级或附属 PDC 发生变化后，调用以下方法同步原版护甲修饰符、属性 Lore 和玩家即时属性：

```java
ItemCoreAPI.refreshItemAttributes(itemStack, player);
```

附属插件禁用时，ItemCore 会自动注销它注册的物品属性与属性 Lore 提供器。
