# Действия и условия

[← На главную](../README.md) · [Конфигурация](CONFIGURATION.md) · [Java API →](API.md)

## Порядок выполнения

Для кнопки логика выполняется так:

1. проверяется `click-permission`;
2. проверяются простые `requirements` и расширенный `click_requirement`;
3. для ресурсных действий выполняется предварительная суммарная проверка;
4. списания и `trade` выполняются до выдачи ресурсов;
5. затем идут визуальные и остальные действия.

При провале проверки выполняется `deny_action`. Если его нет, используется `denial-message`, затем встроенное локализованное сообщение.

После безопасной ресурсной части остальные действия сохраняют порядок YAML. Ошибка останавливает оставшуюся цепочку.

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
  - "[back]"
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
| `[back]` | Вернуться в предыдущее меню |
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

- type: back_menu
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
  currency: regular

- type: give_money
  amount: 250
  currency: donate
```

Алиасы: `withdraw_money` и `deposit_money`. Требуется активный EconomyAPI или OrynthEconomy. Если `currency` не указана, используется `regular`; EconomyAPI поддерживает только обычную валюту.

### Атомарная покупка и продажа

```yaml
- type: trade
  operation: buy
  material: "minecraft:diamond"
  amount: 16
  price: 250
  currency: regular

- type: trade
  operation: sell
  material: "minecraft:emerald"
  amount: 32
  reward: 100
  currency: donate
```

`trade` сам проверяет баланс, предметы и свободное место. При отказе экономики инвентарь восстанавливается. Для количеств 1/16/32/64 на Bedrock создаются отдельные кнопки, а не Java-схема левого/правого клика.

### Задержка и шанс

```yaml
- type: message
  message: "&aГотово!"
  delay-ticks: 20
  chance: 50
```

`delay-ticks` и `chance` (0–100) разрешены только для сообщений, title/actionbar/tip/popup, звуков и переходов по меню. Для денег, предметов, permissions и `trade` модификаторы запрещены валидатором.

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
| `string equals` | `input`, `output` |
| `string equals ignorecase` | `input`, `output` |
| `string contains` | `input`, `output` |
| `regex matches` | `input`, `regex` |
| `>`, `>=`, `==`, `!=`, `<=`, `<` | `input`, `output` |
| `world` | `output` или `value` |
| `gamemode` | `output`: `survival`, `creative`, `adventure`, `spectator` |
| `is near` | `world`, `x`, `y`, `z`, `distance` |
| пользовательский тип | Поля определяет внешний плагин |

## Требования открытия

`open_requirement` использует ту же схему, что `view_requirement` и `click_requirement`:

```yaml
open_requirement:
  requirements:
    spawn:
      type: world
      output: world
    distance:
      type: is near
      world: world
      x: 0
      y: 64
      z: 0
      distance: 100
  deny_action:
    - "[message] &cМеню доступно только на спавне."
  success_action:
    - "[sound] random.click"
```

`/omenus open <игрок> <menu-id>` намеренно обходит условия, так как это административная команда.

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
        not_owned:
          type: "!has permission"
          permission: "server.premium"
      deny_action:
        - "[message] &cПокупка недоступна."
        - "[sound] NOTE_BASS"
    success_action:
      - "[sound] RANDOM_LEVELUP"
    action:
      - type: trade
        operation: buy
        material: "minecraft:nether_star"
        amount: 1
        price: 1000
        currency: regular
      - type: give_permission
        permission: "server.premium"
      - "[message] &aПремиум-доступ активирован!"
      - "[refresh]"
```

## Preflight и атомарные транзакции

Перед цепочкой OrynthMenus суммирует все `take_money`, `take_experience*`, `take_item` и `give_item`. Цепочка не запускается, если:

- недостаточно общего баланса;
- недостаточно уровней;
- недостаточно суммарного количества предметов;
- инвентарь не вместит выдаваемые предметы.

Это уменьшает риск частично выполненных покупок и потери ресурсов.

Для обычных магазинов предпочтителен `trade`: он хранит снимок инвентаря, списывает ресурсы до выдачи и восстанавливает инвентарь, если экономический провайдер отклонил операцию. Искусственного КД нет: быстрые последовательные клики разрешены, а повторный вход в уже идущую операцию блокируется.

---

[← На главную](../README.md) · [Java API →](API.md)
