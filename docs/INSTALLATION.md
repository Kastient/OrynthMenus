# Установка и настройка OrynthMenus

[← На главную](../README.md) · [Конфигурация →](CONFIGURATION.md)

## Системные требования

| Компонент | Требование |
| :--- | :--- |
| Сервер | NukkitMOT |
| Java | 17 или новее |
| OrynthMenus | 1.1.0 |
| FakeInventories | Обязательно |
| EconomyAPI | Опционально |
| OrynthEconomy | Опционально |
| PlaceholderAPI-nukkit | Опционально |

`FakeInventories` указан как жёсткая зависимость: без него OrynthMenus не запустится. Экономический плагин нужен только для условий и действий с деньгами. PlaceholderAPI нужен только для публикации плейсхолдеров OrynthMenus другим плагинам — внутри меню встроенные переменные работают самостоятельно.

## Установка готового JAR

1. Остановите сервер.
2. Скопируйте OrynthMenus и FakeInventories в `plugins/`.
3. При необходимости установите EconomyAPI, OrynthEconomy и PlaceholderAPI.
4. Запустите сервер.
5. Убедитесь, что в консоли появилось сообщение о загрузке OrynthMenus.

После первого запуска структура будет выглядеть примерно так:

```text
plugins/
├─ OrynthMenus.jar
├─ FakeInventories.jar
└─ OrynthMenus/
   ├─ config.yml
   ├─ permissions.yml          # создаётся при постоянной выдаче прав
   ├─ menus/
   │  ├─ menu.yml
   │  ├─ advanced.yml
   │  ├─ donate.yml
   │  ├─ Buyer/
   │  └─ donshop/
   └─ backups/                # резервные копии миграций и редактора
```

## Сборка из исходников

Рядом с проектом должен находиться совместимый `core.jar` NukkitMOT.

```powershell
.\gradlew.bat clean test build
```

Результат:

```text
build/libs/OrynthMenus-1.1.0.jar
```

## Основной config.yml

```yaml
config-version: 2
language: RU

security:
  click-cooldown-ms: 300

economy:
  provider: EconomyAPI
  orynth-currency: regular

gui_menus:
  main:
    file: menu.yml
  shop:
    file: shops/main.yml
```

### `language`

Поддерживаются `RU` и `EN`. Настройка меняет встроенные сообщения, справку и административную панель.

### `security.click-cooldown-ms`

Минимальная пауза между кликами игрока по кнопкам меню. Значение `0` отключает ограничение. По умолчанию — `300` мс.

### `economy.provider`

| Значение | Поведение |
| :--- | :--- |
| `EconomyAPI` | Использовать EconomyAPI |
| `OrynthEconomy` | Использовать OrynthEconomy |
| `AUTO` | Сначала искать OrynthEconomy, затем EconomyAPI |
| `NONE` | Отключить денежные операции |

Для OrynthEconomy параметр `economy.orynth-currency` принимает `regular` или `donate`.

## Регистрация меню

Каждое меню должно быть зарегистрировано в `gui_menus`. ID — ключ секции, путь считается относительно `plugins/OrynthMenus/menus/`.

```yaml
gui_menus:
  weapons_shop:
    file: shops/weapons.yml
```

В действиях и API это меню будет называться `weapons_shop`. Регистр ID при открытии не важен.

Плагин нормализует путь и запрещает выход за каталог `menus/`, поэтому значения вроде `../secret.yml` отклоняются.

## Проверка и применение

```text
/omenus validate
/omenus reload
```

`validate` загружает и проверяет YAML, но не заменяет активные меню. `reload` применяет конфигурацию только тогда, когда все зарегистрированные меню прошли проверку. При ошибке текущий рабочий набор остаётся активным.

## Обновление и миграции

1. Остановите сервер.
2. Сделайте резервную копию `plugins/OrynthMenus/`.
3. Замените JAR.
4. Запустите сервер и изучите консоль.
5. Выполните `/omenus validate`.

Схема конфигурации имеет поле `config-version`. Миграция до версии 2 выполняется один раз и сохраняет оригиналы в `backups/migration-<дата>/`. Изменения визуального редактора сохраняются атомарно, а первая правка файла за сессию получает копию в `backups/editor-<дата>/`.

## Частые проблемы

### Плагин не запускается

Проверьте Java 17, наличие FakeInventories и совместимость `core.jar`/NukkitMOT.

### Денежные действия всегда отклоняются

Проверьте `economy.provider`, название установленного экономического плагина и сообщение `Economy provider:` в консоли.

### Команда меню не появилась

Имя должно состоять из `a-z`, `0-9`, `_` или `-`. Если команда уже занята другим плагином, OrynthMenus оставит меню загруженным, но не зарегистрирует конфликтующую команду.

### Reload не применился

Запустите `/omenus validate` и прочитайте точную ошибку в консоли. Рабочие меню намеренно остаются без изменений.

### Меню ссылается на несуществующий ID

Любой `open_menu`/`[open]` должен указывать на ID из `config.yml`. Меню с битой ссылкой исключается из набора при проверке.

---

[← На главную](../README.md) · [Команды и права](COMMANDS_AND_PERMISSIONS.md) · [Конфигурация →](CONFIGURATION.md)
