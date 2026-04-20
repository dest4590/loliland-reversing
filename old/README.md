# [LoliLand](https://loliland.ru/) Reversing

### [README-EN](README-EN.md)


### Разбор проекта [LoliLand](https://loliland.ru/) для Minecraft

## Структура проекта

Проект LoliLand использует собственный лаунчер для запуска модифицированных сборок Minecraft и включает защиту от установки сторонних модификаций.

Каждая сборка содержит `LoliMod`, обеспечивающий ее функциональность. `LoliMod` является общим для всех сборок.

## Анализ лаунчера

Лаунчер представляет собой JAR-файл (`launcher.jar`), расположенный по пути `%appdata%\.loliland\launcher.jar`.

Он обфусцирован, но строки кода остаются читаемыми.

При запуске лаунчер отправляет данные о конфигурации ПК на API сервер [(список серверов)](servers-ru.md).

```json
{"operatingSystem":{"name":"Windows 11","arch":"amd64","isX64":true},"processor":{"id":"*********","name":"AMD Ryzen 5 5600X 6-Core Processor ","physicalCount":6,"logicalCount":12,"freq":3701000000},"baseboard":{"manufacturer":"********","name":"Micro-Star International Co., Ltd.","serialNumber":"**************","hardwareUUID":"**************"},"graphicsCards":[{"name":"Meta Virtual Monitor","vendor":"Meta Inc.","virtualMemory":0},{"name":"NVIDIA GeForce GTX 1080 Ti","vendor":"NVIDIA","virtualMemory":4293918720}],"displays":["24G2WG3-: 52 x 29 cm (20,5 x 11,4 in)","HP ZR22w: 48 x 27 cm (18,9 x 10,6 in)"],"totalMemory":34282242048}
```

После авторизации лаунчер выполняет следующие запросы:

-   Авторизация: `https://api.loliland.ru/launcher.auth` Результат: `{"data":{"votes":0},"type":"success"}`
-   Параметры запуска: `https://launcher.loliland.pro/sync` Результат: [JSON формат](api/sync-output.json)
-   IP серверов: `https://api.loliland.ru/launcher.address` Результат: [JSON формат](api/address-output.json)

Процесс запуска клиента лаунчером:

1.  **Проверка файлов:** `updateVerify: [libraries, natives, mods, coremods, client.jar]` - лаунчер определяет список файлов и их хеши для проверки обновлений.
2.  **Проверка версий конфигураций:** `updateVersion: {config/_=3.1, optionsof.txt=2.0, lang/_=1.1, options.txt=2.0}` - проверяются версии конфигурационных файлов.
3.  **Локальные хеши:** `updateVersion local hashes: {}` - вывод локальных хешей файлов.
4.  **Удаление файлов:** `toRemove: []` - определение файлов для удаления (используется для защиты).

Исходный код этапов проверки файлов находится в файле [launcher/file-security.java](launcher/file-security.java).

## Анализ клиента (LoliMod)

`LoliMod` отвечает за основные функции клиента, включая:

-   Магазин
-   Кастомный чат
-   GUI
-   Обработку сетевых пакетов сервера

При запуске, `LoliMod` загружает модули обработки пакетов сервера:

```
[18:49:43] [Client thread/INFO] [LoliLand]: On Server PreInit load module: l0lIIOLAND
[18:49:43] [Client thread/INFO] [LoliLand]: On Server PreInit loaded module: l0lIIOLAND
```

## Анализ клиента (client.jar)

Клиент (`client.jar`) при запуске загружает моды и проверяет их хеши (если этого файла нету то мод не считается и не загружается).

Каждый мод содержит файл `META-INF\mc_class_data_hash.bin` со списком хешей классов. Пример содержимого:

```
00000000 E1 88 43 78:68 2F 09 A7|84 1C 8F 9D:50 2F 28 D3
00000010 A1 E4 E1 BB:18 13 B5 76|45 15 4E 11:E3 99 84 B4
00000020 8B BF 00 7E:D6 AD 87 F9|90 EC AE E9:D1 D7 4A CE
00000030 C2 40 3E 05:FC 8A 05 92|51 19 00 56:BA 7F 7C EE
```

## Обход защиты лаунчера и клиента

**Обход проверки хешей лаунчера:**

1.  Откройте `launcher.jar`.
2.  Найдите в коде строку `toRemove: `.
3.  Удалите код, отвечающий за данный этап проверки файлов.

**Обход верификации модов клиента:**

1.  Откройте `%userprofile%\loliland\updates\clients\techno_magic\client.jar` (путь зависит от сборки).
2.  Найдите строку `Examining file %s for potential mods`.
3.  Замените условие `if (ClassDataDumpParser.HAS_DUMP) {` на `if (false) {` для принудительной загрузки модов без хешей.

**Запуск клиента без лаунчера (рекомендуется):**

Используйте скрипт [start.py](scripts/start.py).

**Вывод:**

Лаунчер и клиент используют простые методы защиты, которые можно обойти. Однако, использование читов на проекте LoliLand вероятно не принесет желаемого результата.

## Байпасы

### Обход ваниша (отображение всех игроков в TAB-листе, включая в ванише):

1.  Найдите строку `loliland/gui/modules/tablist/vanish.png`.
2.  Удалите условие, в котором находиться эта строка.
3.  Повторите для всех иконок (`fly.png`, `god.png`, `vanish.png`).
