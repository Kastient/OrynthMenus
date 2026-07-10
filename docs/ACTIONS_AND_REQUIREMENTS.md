# Действия и условия

[← На главную](../README.md) · [Конфигурация](CONFIGURATION.md) · [Java API →](API.md)

## Порядок выполнения

Для кнопки логика выполняется так:

1. проверяется `click-permission`;
2. проверяются простые `requirements` и расширенный `click_requirement`;
3. для экономических действий выполняется предварительная суммарная проверка;
4. запускается `success_action`;
5. запускается основной `action`.

При провале проверки выполняется `deny_action`. Если его нет, используется `denial-message`, затем встроенное локализованное сообщение.

Действия идут строго сверху вниз. Ошибка останавливает оставшуюся цепочку.

## Короткий строковый формат

```yaml
action:
  - "[console] give %player_name% diamond 1"
  - "[player] spawn"
  - "[message] &aГотово!"
  - "[title] &6Награда"
  - "[actionbar] &e+1 предмет"
  - "[tip] &7Подсказка"
  - "[popup] &bУведомление"
  - "[sound] RANDOM_LEVELUP"
  - "[open] next_menu"
  - "[refresh]"
  - "[close]"
```

| Запись | Результат |
| :--- | :--- |
| `[console] <команда>` | Команда от имени консоли |
| `[player] <команда>` | Команда от имени игрока |
| `[message] <текст>` | Сообщение в чат |
| `[title] <текст>` | Title |
| `[actionbar] <текст>` | Action bar |
| `[tip] <текст>` | Tip |
| `[popup] <текст>` | Popup |
| `[sound] <звук>` | Звук около игрока |
| `[open] <menu-id>` | Открыть другое меню |
| `[refresh]` | Перерисовать текущее inventory-меню |
| `[close]` | Закрыть активное меню/форму |

## Полный YAML-формат действий

### Команды и интерфейс

```yaml
- type: console_command
  command: "give %player_name% diamond 1"

- type: player_command
  command: "spawn"

- type: message
  message: "&aСообщение"

- type: title
  title: "&6Заголовок"
  subtitle: "&fПодзаголовок"

- type: sound
  sound: "RANDOM_LEVELUP"

- type: open_menu
  menu: "main"

- type: close_menu

- type: refresh_menu
```

Для `message`, `tip`, `popup`, `actionbar` допускается поле `text`; для `title` — `text`; для `sound` — `name`.

### Предметы

```yaml
- type: give_item
  material: "minecraft:diamond"
  amount: 3

- type: take_item
  material: "minecraft:emerald"
  amount: 16
```

`give_item` проверяет свободное место и не выбрасывает остаток на землю. `take_item` заранее проверяет суммарное количество подходящих предметов.

### Экономика

```yaml
- type: take_money
  amount: 500

- type: give_money
  amount: 250
```

Алиасы: `withdraw_money` и `deposit_money`. Требуется активный EconomyAPI или OrynthEconomy.

### Опыт

```yaml
- type: give_experience
  amount: 150

- type: take_experience_levels
  levels: 45
```

`give_experience` выдаёт очки опыта. `take_experience`/`take_experience_levels` списывает уровни.

### Эффекты

```yaml
- type: effect
  effect: "speed"
  level: 2
  seconds: 1800
  visible: true
```

Алиас действия — `give_effect`. Уровень и длительность должны быть положительными.

### Permissions

```yaml
- type: give_permission
  permission: "server.rank.vip"
  temporary: false

- type: remove_permission
  permission: "server.rank.vip"
  temporary: false
```

Без `temporary: true` право сохраняется в `permissions.yml` и восстанавливается при входе.

## Простые требования

Старая компактная секция `requirements` поддерживает одновременную проверку permission, денег, уровней и предмета:

```yaml
requirements:
  permission: "server.shop"
  money: 500
  experience-levels: 10
  item:
    material: "minecraft:emerald"
    amount: 3
```

Для видимости также есть `visible-if` и короткие поля `view-permission`/`visible-permission`.

## Расширенные группы требований

```yaml
click_requirement:
  requirements:
    access:
      type: "has permission"
      permission: "server.shop"
    balance:
      type: "has money"
      amount: 500
    level:
      type: "has experience"
      amount: 10
    key:
      type: "has item"
      material: "minecraft:tripwire_hook"
      amount: 1
```

| Тип | Обязательные поля |
| :--- | :--- |
| `has permission` / `permission` | `permission` |
| `has money` / `money` | `amount` |
| `has experience` / `experience` | `amount` или `levels` |
| `has item` / `item` | `material`, опционально `amount` |
| пользовательский тип | Поля определяет внешний плагин |

## Отрицания

Префикс `!` инвертирует любое условие:

```yaml
view_requirement:
  requirements:
    locked:
      type: "!has permission"
      permission: "server.vip"
```

## Логика «ИЛИ» и минимальное количество

По умолчанию все не-optional требования должны пройти. Для варианта «хотя бы одно»:

```yaml
view_requirement:
  minimum_requirements: 1
  stop_at_success: true
  requirements:
    vip:
      type: "has permission"
      permission: "rank.vip"
      optional: true
    premium:
      type: "has permission"
      permission: "rank.premium"
      optional: true
```

- `minimum_requirements` — сколько успешных проверок достаточно;
- `stop_at_success` — прекратить проверку сразу после достижения минимума;
- `optional` — провал этого условия сам по себе не отклоняет группу.

## Полный безопасный сценарий покупки

```yaml
items:
  premium:
    material: "minecraft:nether_star"
    slot: 14
    display_name: "&6Премиум-доступ"
    lore:
      - "&7Цена: &e1 000 монет"
      - "&7Не хватает: &c%OrynthMenus_money_missing<1000>%"
    click_requirement:
      requirements:
        money:
          type: "has money"
          amount: 1000
        not_owned:
          type: "!has permission"
          permission: "server.premium"
      deny_action:
        - "[message] &cПокупка недоступна."
        - "[sound] NOTE_BASS"
    success_action:
      - "[sound] RANDOM_LEVELUP"
    action:
      - type: take_money
        amount: 1000
      - type: give_permission
        permission: "server.premium"
      - "[message] &aПремиум-доступ активирован!"
      - "[refresh]"
```

## Preflight-защита

Перед цепочкой OrynthMenus суммирует все `take_money`, `take_experience*`, `take_item` и `give_item`. Цепочка не запускается, если:

- недостаточно общего баланса;
- недостаточно уровней;
- недостаточно суммарного количества предметов;
- инвентарь не вместит выдаваемые предметы.

Это уменьшает риск частично выполненных покупок и потери ресурсов.

---

[← На главную](../README.md) · [Java API →](API.md)
