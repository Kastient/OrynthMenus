---
icon: folder-open
---

# Структура конфигурации

## 🗂 Структура конфигурации

OrynthMenus хранит регистрацию меню отдельно от их содержимого. Это позволяет удобно распределять файлы по папкам и поддерживать порядок даже на большом сервере.

### Основная папка

После первого запуска создаётся:

```
plugins/OrynthMenus/
```

Пример структуры:

```
plugins/OrynthMenus/
├── config.yml
├── permissions.yml
├── language/
└── menus/
    ├── main.yml
    ├── Buyer/
    │   ├── ores.yml
    │   └── vegetables.yml
    └── donshop/
        ├── main.yml
        └── effects.yml
```

### Файл `config.yml`

Все меню должны быть зарегистрированы в:

```
plugins/OrynthMenus/config.yml
```

Пример:

```yaml
gui_menus:
  main:
    file: main.yml

  buyer_ores:
    file: Buyer/ores.yml

  buyer_vegetables:
    file: Buyer/vegetables.yml

  donshop:
    file: donshop/main.yml
```

### Как работает путь

Путь всегда указывается относительно папки:

```
plugins/OrynthMenus/menus/
```

Например:

```yaml
file: shops/main.yml
```

означает файл:

```
plugins/OrynthMenus/menus/shops/main.yml
```

> ⚠️ Не указывайте полный путь диска и не добавляйте `plugins/OrynthMenus/menus/` перед именем файла.

### ID меню

В примере:

```yaml
gui_menus:
  shop:
    file: shops/main.yml
```

`shop` — это ID меню.

ID используется для:

* открытия меню через `[open]`;
* вызова меню через Java API;
* проверки существования меню;
* связи между несколькими интерфейсами.

Пример открытия другого меню:

```yaml
action:
  - "[open] shop"
```

### Правила для ID

Рекомендуется использовать:

```
main
shop
buyer_ores
donate_info
server_navigation
```

Не рекомендуется:

```
Главное меню
Shop Menu
донат магазин
```

Лучше использовать:

* латиницу;
* нижний регистр;
* символ `_`;
* короткие понятные названия.

### Структура файла меню

Базовый инвентарный файл выглядит так:

```yaml
type: inventory
inventory-type: CHEST
size: 27
title: "&8Меню"

open:
  command: menu
  description: "Открыть меню"

items:
  example:
    material: "minecraft:diamond"
    slot: 14
    display_name: "&bАлмаз"
    lore:
      - "&7Пример элемента"
    action:
      - "[message] &aВы нажали на предмет."
```

Базовая форма:

```yaml
type: form
title: "&8Информация"

content:
  - "&fОсновной текст формы"
  - "&7Выберите действие ниже"

open:
  command: info

items:
  close:
    lore:
      - "&cЗакрыть"
      - "&7Вернуться в игру"
    action:
      - "[close]"
```

### Основные секции

| Секция            | Назначение           |
| ----------------- | -------------------- |
| `type`            | Тип меню             |
| `inventory-type`  | Тип контейнера       |
| `size`            | Количество слотов    |
| `title`           | Заголовок            |
| `update_interval` | Интервал обновления  |
| `open`            | Команда открытия     |
| `content`         | Основной текст формы |
| `items`           | Элементы и кнопки    |

### Организация файлов

Для большого проекта удобно использовать отдельные папки:

```
menus/
├── main/
├── shops/
├── buyer/
├── donate/
├── navigation/
└── information/
```

Пример регистрации:

```yaml
gui_menus:
  main:
    file: main/index.yml

  shop_blocks:
    file: shops/blocks.yml

  shop_food:
    file: shops/food.yml

  donate_main:
    file: donate/main.yml

  rules:
    file: information/rules.yml
```

### Проверка YAML

Перед перезагрузкой убедитесь:

* отступы сделаны пробелами;
* кавычки закрыты;
* после ключей стоит `:`;
* список начинается с `-`;
* ID не повторяются;
* путь к файлу существует.

> 💡 Если одно меню не загружается, сначала проверьте путь в `config.yml`, затем сам YAML-файл.
