# Полная конфигурация меню

[← На главную](../README.md) · [Действия и условия →](ACTIONS_AND_REQUIREMENTS.md)

## Общая схема

Каждое меню хранится в отдельном YAML-файле и регистрируется в `config.yml`.

```yaml
gui_menus:
  example:
    file: category/example.yml
```

ID `example` используется в API, `/omenus open` и действиях перехода.

## Поля меню

| Поле | Тип | Обязательно | Описание |
| :--- | :--- | :---: | :--- |
| `type` | строка | нет | `inventory` по умолчанию или `form` |
| `title` | строка | да | Заголовок интерфейса |
| `content` | строка/список | для формы — нет | Основной текст Bedrock Form |
| `inventory-type` | строка | нет | Тип контейнера, по умолчанию `CHEST` |
| `size` | число | зависит от типа | Размер контейнера |
| `update_interval` | число | нет | Интервал динамического обновления в секундах |
| `open` | секция | нет | Команда, permission, описание и действия открытия |
| `items` | секция | да | Кнопки меню; также поддерживается имя `buttons` |

Корневую конфигурацию можно обернуть в секцию `menu:` — загрузчик объединит её с остальными корневыми полями.

## Inventory GUI

```yaml
type: inventory
inventory-type: CHEST
size: 27
title: "&8Магазин"
update_interval: 1

open:
  command: shop
  permission: "server.shop"
  description: "Открыть магазин"
  action:
    - "[sound] random.chestopen"

items:
  background:
    material: "minecraft:black_stained_glass_pane"
    slots: "1-27"
    display_name: " "
    priority: 0
    item_flags:
      - HIDE_ATTRIBUTES

  product:
    material: "minecraft:diamond"
    slot: 14
    amount: 1
    display_name: "&bАлмаз"
    lore:
      - "&7Баланс: &e%OrynthMenus_balance%"
    glow: true
    priority: 10
    action:
      - "[message] &aВы нажали на товар."
```

### Поддерживаемые контейнеры

| Тип | Размер |
| :--- | :---: |
| `CHEST` | 27 |
| `DOUBLE_CHEST` | 54 |
| `HOPPER` | 5 |
| `FURNACE` | 3 |
| `ENDER_CHEST` | стандартный размер Nukkit |
| `DISPENSER` | стандартный размер Nukkit |
| `DROPPER` | стандартный размер Nukkit |
| `BREWING_STAND` | стандартный размер Nukkit |
| `SHULKER_BOX` | стандартный размер Nukkit |
| `BEACON` | стандартный размер Nukkit |
| `BARREL` | стандартный размер Nukkit |
| `BLAST_FURNACE` | стандартный размер Nukkit |
| `SMOKER` | стандартный размер Nukkit |

Для `CHEST` размер должен делиться на 9. Значение `CHEST` + `size: 54` автоматически превращается в `DOUBLE_CHEST`.

## Bedrock Form

```yaml
type: form
title: "&8Главное меню"
content:
  - "&7Выберите нужный раздел"
  - ""
  - "&fИгрок: &a{player}"
content_2:
  - "&fБаланс: &e{money}"

open:
  command: menu

items:
  shop:
    text: "&aМагазин"
    lore:
      - "&7Нажмите, чтобы открыть"
    action:
      - "[open] shop"

  close:
    text: "&cЗакрыть"
    action:
      - "[close]"
```

`content`, `content_2`, `content_3` и последующие нумерованные блоки объединяются через перенос строки. Значением может быть строка или YAML-список.

Кнопка формы поддерживает заголовок и не более двух строк `lore`. Если `text`/`display_name` не указан, первая строка `lore` становится заголовком.

## Поля кнопки

| Поле | Значение |
| :--- | :--- |
| `material` | Материал Nukkit, например `minecraft:diamond` |
| `slot` | Один слот, нумерация начинается с 1 |
| `slots` | Число, диапазон или список слотов |
| `amount` | Размер стака от 1 до 64 |
| `display_name` | Название предмета/кнопки |
| `lore` | YAML-список строк описания |
| `glow` | Добавить скрытое зачарование для свечения |
| `enchantments` | Список настоящих зачарований |
| `clear-enchantments` | Не применять указанные зачарования |
| `item_flags` | Скрытие атрибутов и подсказок предмета |
| `priority` | Приоритет при пересечении кнопок в одном слоте |
| `update` | Динамически обновлять кнопку |
| `view_requirement` | Условия видимости |
| `click_requirement` | Условия нажатия |
| `click-permission` | Простая проверка permission на клик |
| `denial-message` | Сообщение при отказе, если нет `deny_action` |
| `deny_action` | Действия при провале условия |
| `success_action` | Действия после успешной проверки |
| `action` | Основные действия кнопки |

Алиасы старой схемы: `buttons`, `name`, `weight`, `hide-flags`, `actions`, `inventory_type`, `update-interval`, `command-permission`.

## Слоты и диапазоны

Слоты в YAML считаются от `1`, а не от `0`.

```yaml
slot: 14
```

```yaml
slots: "1-9"
```

```yaml
slots:
  - 1
  - "3-7"
  - 9
```

Повторяющиеся номера удаляются с сохранением порядка. Нулевые, отрицательные, выходящие за размер, нисходящие (`9-1`) и некорректные диапазоны отклоняются.

## Приоритеты и перекрытия

Несколько кнопок могут занимать один слот. Кнопки отрисовываются по `priority`; более высокий приоритет перекрывает низкий. Это удобно для состояний «закрыто/открыто»:

```yaml
items:
  locked:
    material: BARRIER
    slot: 14
    priority: 10
    display_name: "&cНедоступно"
    view_requirement:
      requirements:
        no_access:
          type: "!has permission"
          permission: "server.vip"

  unlocked:
    material: DIAMOND
    slot: 14
    priority: 100
    display_name: "&aДоступно"
    view_requirement:
      requirements:
        access:
          type: "has permission"
          permission: "server.vip"
```

## Зачарования и item flags

```yaml
enchantments:
  - id: sharpness
    level: 5
  - id: 17
    level: 1

item_flags:
  - HIDE_ENCHANTS
  - HIDE_ATTRIBUTES
  - HIDE_UNBREAKABLE
  - HIDE_CAN_DESTROY
  - HIDE_CAN_PLACE_ON
  - HIDE_POTION_EFFECTS
  - HIDE_DYE
```

Поддерживается `ALL`, а также числовая битовая маска от `0` до `127`. Имена нечувствительны к регистру, `_` и `-` взаимозаменяемы.

## Динамическое обновление

```yaml
update_interval: 1

items:
  balance:
    material: EMERALD
    slot: 5
    display_name: "&aБаланс"
    lore:
      - "&fМонеты: &e%OrynthMenus_balance%"
    update: true
```

Плагин раз в секунду проверяет открытые меню. Перерисовываются только кнопки с `update: true`, когда прошёл заданный `update_interval`. Значение `0` отключает обновления.

## Цвета и центрирование

Поддерживаются стандартные цветовые коды `&`. Каждая строка автоматически начинается со сброса форматирования.

```yaml
title: "&8Меню"
content:
  - "<center>&6Добро пожаловать</center>"
```

Тег `<center>...</center>` рассчитывает отступ по видимой длине строки без цветовых кодов.

## Встроенные переменные

| Переменная | Значение |
| :--- | :--- |
| `{player}` / `%player%` | Имя игрока |
| `{player_name}` / `%player_name%` | Имя игрока |
| `{level}` / `%level%` | Уровень опыта |
| `{money}` / `%money%` | Форматированный баланс |
| `{economy_balance}` | Форматированный баланс |
| `{menu}` | ID текущего меню |

## Плейсхолдеры OrynthMenus

| Плейсхолдер | Значение |
| :--- | :--- |
| `%OrynthMenus_balance%` | Баланс |
| `%OrynthMenus_money%` | Алиас баланса |
| `%OrynthMenus_experience%` | Уровень опыта |
| `%OrynthMenus_level%` | Алиас уровня |
| `%OrynthMenus_money_missing<5000>%` | Недостающая сумма |
| `%OrynthMenus_experience_missing<45>%` | Недостающие уровни |

Поддерживаются варианты `orynthmenus_*` в нижнем регистре. Внутри меню они работают всегда; при наличии PlaceholderAPI регистрируются глобально.

---

[← На главную](../README.md) · [Действия и условия →](ACTIONS_AND_REQUIREMENTS.md)
