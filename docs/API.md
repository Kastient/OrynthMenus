# Java API OrynthMenus

[← На главную](../README.md) · [Действия и условия](ACTIONS_AND_REQUIREMENTS.md)

## Подключение

Добавьте OrynthMenus в зависимости своего `plugin.yml`:

```yaml
depend:
  - OrynthMenus
```

Получите экземпляр плагина и API после включения зависимостей:

```java
import ru.cryovex.orynthmenus.OrynthMenusPlugin;
import ru.cryovex.orynthmenus.api.OrynthMenusApi;

OrynthMenusPlugin plugin = (OrynthMenusPlugin) getServer()
        .getPluginManager()
        .getPlugin("OrynthMenus");

if (plugin == null || !plugin.isEnabled()) {
    throw new IllegalStateException("OrynthMenus is unavailable");
}

OrynthMenusApi menus = plugin.getApi();
```

`getApi()` выбрасывает `IllegalStateException`, если OrynthMenus ещё не включён.

## Методы API

| Метод | Назначение |
| :--- | :--- |
| `open(Player, String)` | Открыть меню по ID |
| `refresh(Player)` | Перерисовать текущее inventory-меню игрока |
| `close(Player)` | Закрыть меню, форму или редактор игрока |
| `hasMenu(String)` | Проверить существование загруженного меню |
| `menuIds()` | Получить неизменяемый набор ID меню |
| `registerRequirement(String, CustomRequirement)` | Зарегистрировать пользовательское условие |
| `unregisterRequirement(String)` | Удалить пользовательское условие |
| `hasRequirement(String)` | Проверить регистрацию типа условия |

### Открытие меню

```java
if (menus.hasMenu("shop")) {
    menus.open(player, "shop");
}
```

Обычный `open` соблюдает `open.permission` меню. Административный обход разрешения не вынесен в публичный API.

### Обновление и закрытие

```java
menus.refresh(player);
menus.close(player);
```

`refresh` действует только для открытой inventory-сессии. Для формы вызов ничего не делает.

## Пользовательские требования

Внешний плагин может добавить собственный `type` для `view_requirement` или `click_requirement`.

```java
menus.registerRequirement("has tokens", context -> {
    int required = context.integer("amount", 0);
    return tokenService.balance(context.player()) >= required;
});
```

Использование в YAML:

```yaml
click_requirement:
  requirements:
    tokens:
      type: "has tokens"
      amount: 100
  deny_action:
    - "[message] &cНедостаточно жетонов."
```

Неизвестные встроенному загрузчику типы автоматически считаются пользовательскими. Имя нормализуется: регистр игнорируется, `_`, `-` и повторяющиеся пробелы приводятся к единому виду.

## RequirementContext

```java
public record RequirementContext(
        Player player,
        String id,
        Map<String, Object> values
) { }
```

| Метод | Описание |
| :--- | :--- |
| `player()` | Игрок, для которого выполняется проверка |
| `id()` | ID требования из YAML |
| `values()` | Неизменяемые исходные значения секции |
| `string(key, fallback)` | Безопасно прочитать строку |
| `integer(key, fallback)` | Безопасно прочитать целое число |
| `decimal(key, fallback)` | Безопасно прочитать дробное число |
| `bool(key, fallback)` | Безопасно прочитать boolean |

Пример условия с несколькими параметрами:

```java
menus.registerRequirement("world and reputation", context -> {
    String world = context.string("world", "world");
    int reputation = context.integer("reputation", 0);

    return context.player().getLevelName().equalsIgnoreCase(world)
            && reputationService.get(context.player()) >= reputation;
});
```

```yaml
view_requirement:
  requirements:
    access:
      type: "world and reputation"
      world: "spawn"
      reputation: 25
```

## Жизненный цикл регистрации

Регистрируйте требования после включения OrynthMenus и до того, как игроки начнут использовать соответствующие меню. Регистрация с тем же нормализованным именем заменяет предыдущий обработчик.

```java
menus.unregisterRequirement("has tokens");
```

Если обработчик выбросит `RuntimeException`, требование безопасно вернёт `false`, а действие не будет выполнено.

## Минимальный интеграционный класс

```java
public final class TokenMenusIntegration {
    private final OrynthMenusApi menus;
    private final TokenService tokens;

    public TokenMenusIntegration(OrynthMenusApi menus, TokenService tokens) {
        this.menus = menus;
        this.tokens = tokens;
    }

    public void register() {
        menus.registerRequirement("has tokens", context ->
                tokens.balance(context.player())
                        >= context.integer("amount", 0)
        );
    }

    public void unregister() {
        menus.unregisterRequirement("has tokens");
    }
}
```

---

[← На главную](../README.md) · [Действия и условия](ACTIONS_AND_REQUIREMENTS.md)
