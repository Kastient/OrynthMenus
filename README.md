<div align="center">

# OrynthMenus

### Красивые GUI-меню и Bedrock-формы для NukkitMOT

Создавайте магазины, навигацию, донат-меню, награды и интерактивные интерфейсы — без изменения Java-кода.

![Version](https://img.shields.io/badge/version-1.1.0-7c3aed?style=for-the-badge)
![Java](https://img.shields.io/badge/Java-17+-f97316?style=for-the-badge&logo=openjdk&logoColor=white)
![Platform](https://img.shields.io/badge/Nukkit-MOT-22c55e?style=for-the-badge)
![Config](https://img.shields.io/badge/config-YAML-0ea5e9?style=for-the-badge)

**Inventory GUI** · **Bedrock Forms** · **Экономика** · **Условия** · **Плейсхолдеры** · **Java API**

</div>

---

## Что это за плагин?

**OrynthMenus** — гибкий конструктор серверных меню для NukkitMOT. Плагин умеет открывать настоящие инвентарные интерфейсы через FakeInventories и нативные Bedrock SimpleForm, связывать кнопки с командами и действиями, проверять деньги, опыт, предметы и права игрока.

Меню описываются обычными YAML-файлами. Для базовой настройки не требуется писать собственный плагин, а для сложной логики предусмотрен публичный Java API и регистрация пользовательских условий.

| Возможность | Что даёт серверу |
| :--- | :--- |
| 🧰 **Inventory GUI** | Сундуки, двойные сундуки, воронки, печи и другие контейнеры с собственными названиями |
| 📱 **Bedrock Forms** | Лёгкие нативные формы с многострочным текстом и кнопками |
| ⚡ **Действия** | Команды, сообщения, звуки, предметы, деньги, опыт, эффекты и permissions |
| 🔐 **Условия** | Отображение и нажатие по правам, балансу, опыту, предметам или собственным проверкам |
| 💰 **Экономика** | EconomyAPI и OrynthEconomy, включая обычную и донат-валюту |
| 🧩 **Плейсхолдеры** | Встроенные переменные и интеграция с PlaceholderAPI-nukkit |
| 🎛️ **Редактор** | Bedrock-панель настроек и визуальная перестановка предметов по слотам |
| 🛡️ **Безопасность** | Атомарная перезагрузка, резервные копии, preflight покупок и антиспам кликов |

## Быстрый старт

### Требования

- NukkitMOT;
- Java 17 или новее;
- FakeInventories — обязательная зависимость;
- EconomyAPI или OrynthEconomy — опционально, если меню работают с деньгами;
- PlaceholderAPI — опционально, если плейсхолдеры OrynthMenus нужны другим плагинам.

### Установка

1. Поместите `OrynthMenus-1.1.0.jar` и FakeInventories в папку `plugins/`.
2. Запустите сервер — плагин создаст `config.yml` и примеры в `plugins/OrynthMenus/menus/`.
3. Зарегистрируйте своё меню в `config.yml`.
4. Проверьте YAML командой `/omenus validate`.
5. Примените изменения командой `/omenus reload`.

```yaml
# plugins/OrynthMenus/config.yml
gui_menus:
  shop:
    file: shop.yml
```

```yaml
# plugins/OrynthMenus/menus/shop.yml
type: inventory
inventory-type: CHEST
size: 27
title: "&8Магазин"

open:
  command: shop
  description: "Открыть магазин"

items:
  diamond:
    material: "minecraft:diamond"
    slot: 14
    display_name: "&bАлмаз"
    lore:
      - "&7Цена: &e500 монет"
      - "&aНажмите, чтобы купить"
    click_requirement:
      requirements:
        balance:
          type: "has money"
          amount: 500
      deny_action:
        - "[message] &cНедостаточно монет."
        - "[sound] NOTE_BASS"
    action:
      - type: take_money
        amount: 500
      - type: give_item
        material: "minecraft:diamond"
        amount: 1
      - "[message] &aПокупка выполнена!"
      - "[sound] RANDOM_LEVELUP"
```

После перезагрузки игрок сможет открыть меню командой `/shop`.

## Команды

Основная команда: `/orynthmenus`, короткий алиас: `/omenus`.

| Команда | Назначение | Permission |
| :--- | :--- | :--- |
| `/omenus help` | Локализованная справка | — |
| `/omenus settings` | Bedrock-панель управления | `orynthmenus.admin.settings` |
| `/omenus open <игрок> <menu-id>` | Принудительно открыть меню игроку | `orynthmenus.admin.open` |
| `/omenus validate` | Проверить конфигурацию без применения | `orynthmenus.admin.validate` |
| `/omenus reload` | Безопасно применить конфигурацию | `orynthmenus.admin.reload` |

Все административные права по умолчанию доступны операторам сервера.

## Что можно собрать

- главное меню сервера и навигацию между разделами;
- магазин покупки и продажи ресурсов;
- донат-магазин с постоянной выдачей permissions;
- меню наборов, наград и ежедневных бонусов;
- интерфейс выбора режима, мира или сервера;
- меню, которое меняется в зависимости от ранга, баланса или предметов;
- динамические карточки баланса и уровня опыта;
- многостраничные интерфейсы через действие `open_menu`.

## Документация

| Раздел | Содержание |
| :--- | :--- |
| [Установка и настройка](docs/INSTALLATION.md) | Зависимости, сборка, структура файлов, экономика, обновление |
| [Команды и permissions](docs/COMMANDS_AND_PERMISSIONS.md) | Все команды, права и встроенный редактор |
| [Конфигурация меню](docs/CONFIGURATION.md) | Inventory GUI, формы, предметы, слоты, флаги и плейсхолдеры |
| [Действия и условия](docs/ACTIONS_AND_REQUIREMENTS.md) | Полный список действий, требования, отрицания и сценарии покупок |
| [Java API](docs/API.md) | Открытие меню из кода и собственные типы требований |
| [История изменений](CHANGELOG.md) | Что изменилось в версиях плагина |

## Поддерживаемые контейнеры

`CHEST` (27) · `DOUBLE_CHEST` (54) · `HOPPER` (5) · `FURNACE` (3) · `ENDER_CHEST` · `DISPENSER` · `DROPPER` · `BREWING_STAND` · `SHULKER_BOX` · `BEACON` · `BARREL` · `BLAST_FURNACE` · `SMOKER`

## Плейсхолдеры

| Плейсхолдер | Значение |
| :--- | :--- |
| `%OrynthMenus_balance%` | Текущий баланс игрока |
| `%OrynthMenus_money_missing<5000>%` | Сколько денег не хватает до суммы |
| `%OrynthMenus_experience%` | Текущий уровень опыта |
| `%OrynthMenus_experience_missing<45>%` | Сколько уровней не хватает до значения |

Также доступны `{player}`, `{player_name}`, `{money}`, `{economy_balance}`, `{level}` и `{menu}`. Варианты в `%процентах%` тоже поддерживаются.

## API за 30 секунд

```java
OrynthMenusPlugin plugin = (OrynthMenusPlugin) getServer()
        .getPluginManager().getPlugin("OrynthMenus");

OrynthMenusApi api = plugin.getApi();
api.open(player, "shop");

api.registerRequirement("has tokens", context ->
        tokenService.balance(context.player()) >= context.integer("amount", 0)
);
```

## Надёжность

- ошибочный reload не заменяет уже работающие меню;
- ссылки `open_menu` на отсутствующие меню выявляются при проверке;
- файлы меню не могут выйти за пределы каталога `menus/`;
- редактор создаёт резервные копии перед записью YAML;
- перед покупкой проверяются суммарные списания и свободное место;
- предметы интерфейса нельзя забрать или переложить;
- частые повторные клики блокируются настраиваемым cooldown.

---

<div align="center">

**OrynthMenus 1.1.0** · Автор: **Cryovex**

Создано для серверов Minecraft Bedrock на NukkitMOT.

</div>
