# Руководство по использованию Cpp2IL

## Установка

### Скачивание

Cpp2IL доступен в нескольких вариантах:

1. **Релизные сборки** - стабильные версии с GitHub Releases
2. **CI сборки** - последние изменения с GitHub Actions
3. **NuGet пакет** - для использования в собственных проектах

### Ссылки на сборки

**Windows:**
- [Native Build (.NET 9)](https://nightly.link/SamboyCoding/Cpp2IL/workflows/dotnet-core/development/Cpp2IL-net9-win-x64.zip)
- [Framework Build (.NET 4.7.2)](https://nightly.link/SamboyCoding/Cpp2IL/workflows/dotnet-core/development/Cpp2IL-Netframework472-Windows.zip)

**Linux:**
- [Native Build](https://nightly.link/SamboyCoding/Cpp2IL/workflows/dotnet-core/development/Cpp2IL-net9-linux-x64.zip)

**macOS:**
- [Native Build](https://nightly.link/SamboyCoding/Cpp2IL/workflows/dotnet-core/development/Cpp2IL-net9-osx-x64.zip)

### Системные требования

- **Windows:** .NET 9.0 Runtime или .NET Framework 4.7.2+
- **Linux:** .NET 9.0 Runtime
- **macOS:** .NET 9.0 Runtime

## Базовое использование

### Простой случай (Windows)

Для большинства Unity игр на Windows достаточно указать путь к папке игры:

```bash
Cpp2IL-Win.exe --game-path=C:\Path\To\Your\Game
```

Cpp2IL автоматически:
- Найдет исполняемый файл игры
- Определит Unity версию
- Найдет GameAssembly.dll и global-metadata.dat
- Создаст папку cpp2il_out с результатами

### Указание имени исполняемого файла

Если в папке несколько .exe файлов:

```bash
Cpp2IL-Win.exe --game-path=C:\Game --exe-name=MyGame
```

### Подробное логирование

Для отладки и получения дополнительной информации:

```bash
Cpp2IL-Win.exe --game-path=C:\Game --verbose
```

## Поддерживаемые форматы

### APK файлы (Android)

```bash
Cpp2IL-Win.exe --game-path=game.apk
```

Cpp2IL автоматически извлечет файлы из APK и найдет необходимые компоненты.

### IPA файлы (iOS)

```bash
Cpp2IL-Win.exe --game-path=game.ipa
```

### WebAssembly

```bash
Cpp2IL-Win.exe --game-path=game.wasm
```

Для обфусцированных WASM файлов может потребоваться framework.js:

```bash
Cpp2IL-Win.exe --game-path=game.wasm --wasm-framework-file=framework.js
```

## Слои обработки

### Просмотр доступных слоев

```bash
Cpp2IL-Win.exe --list-processors
```

### Использование слоев

```bash
# Инъекция атрибутов
Cpp2IL-Win.exe --game-path=C:\Game --use-processor=attributeinjector

# Анализ вызовов
Cpp2IL-Win.exe --game-path=C:\Game --use-processor=callanalysis

# Стабильное переименование
Cpp2IL-Win.exe --game-path=C:\Game --use-processor=stablerenaming

# Несколько слоев одновременно
Cpp2IL-Win.exe --game-path=C:\Game --use-processor=attributeinjector --use-processor=callanalysis
```

### Конфигурация слоев

```bash
Cpp2IL-Win.exe --game-path=C:\Game --use-processor=mylayer --processor-config=key=value
```

## Форматы вывода

### Просмотр доступных форматов

```bash
Cpp2IL-Win.exe --list-output-formats
```

### Использование форматов

```bash
# Dummy DLL (по умолчанию)
Cpp2IL-Win.exe --game-path=C:\Game --output-as=dummydll

# C# код для сравнения
Cpp2IL-Win.exe --game-path=C:\Game --output-as=diffablecs

# ISIL дамп
Cpp2IL-Win.exe --game-path=C:\Game --output-as=isildump

# Отчет о сборке (требует плагин)
Cpp2IL-Win.exe --game-path=C:\Game --output-as=buildreport

# Граф потока управления (требует плагин)
Cpp2IL-Win.exe --game-path=C:\Game --output-as=controlflowgraph
```

### Указание папки вывода

```bash
Cpp2IL-Win.exe --game-path=C:\Game --output-to=my_output_folder
```

## Плагины

### Установка плагинов

1. Скачайте плагин
2. Поместите .dll файл в папку `Plugins` рядом с Cpp2IL
3. Запустите Cpp2IL - плагин загрузится автоматически

### Доступные плагины

1. **BuildReport** - генерация отчетов о сборке
2. **ControlFlowGraph** - создание графов потока управления
3. **OrbisPkg** - поддержка PlayStation PKG файлов
4. **Pdb** - поддержка PDB файлов отладки
5. **StrippedCodeRegSupport** - поддержка обфусцированного кода

## Примеры использования

### Анализ Android игры

```bash
# Скачивание APK
wget https://example.com/game.apk

# Анализ с подробным отчетом
Cpp2IL-Win.exe --game-path=game.apk --output-as=buildreport --verbose
```

### Анализ WebAssembly игры

```bash
# Скачивание WASM и framework.js
wget https://example.com/game.wasm
wget https://example.com/framework.js

# Анализ с поддержкой обфусцированных экспортов
Cpp2IL-Win.exe --game-path=game.wasm --wasm-framework-file=framework.js --output-as=isildump
```

### Создание dummy DLL для моддинга

```bash
# Базовая генерация
Cpp2IL-Win.exe --game-path=C:\Game --output-as=dummydll

# С инъекцией атрибутов
Cpp2IL-Win.exe --game-path=C:\Game --output-as=dummydll --use-processor=attributeinjector
```

### Анализ потока управления

```bash
# Создание графов для всех методов
Cpp2IL-Win.exe --game-path=C:\Game --output-as=controlflowgraph

# С анализом вызовов
Cpp2IL-Win.exe --game-path=C:\Game --output-as=controlflowgraph --use-processor=callanalysis
```

## Устранение неполадок

### Ошибки инициализации

**Проблема:** "Failed to initialize LibCpp2IL"
**Решение:** Проверьте, что файлы не повреждены и Unity версия поддерживается

### Ошибки определения версии

**Проблема:** "Could not determine unity version"
**Решение:** Укажите версию вручную или проверьте структуру файлов

### Ошибки плагинов

**Проблема:** "Plugin failed to load"
**Решение:** Проверьте зависимости плагина и совместимость версий

### Проблемы с памятью

**Проблема:** "Out of memory"
**Решение:** Используйте флаг `--low-memory-mode` для больших игр

## Цвета в терминале

Cpp2IL использует цветное логирование:

- **Серый** - VERB (подробная информация)
- **Синий** - INFO (обычная информация)
- **Желтый** - WARN (предупреждения)
- **Красный** - FAIL (ошибки)

### Отключение цветов

```bash
# Установка переменной окружения
set NO_COLOR=true

# Или в PowerShell
$env:NO_COLOR="true"
```

## Продвинутое использование

### Использование Core API

Для интеграции в собственные проекты:

```csharp
using Cpp2IL.Core;
using AssetRipper.Primitives;

// Инициализация
Cpp2IL.Core.Cpp2IlApi.InitializeLibCpp2Il(
    "GameAssembly.dll",
    "global-metadata.dat",
    new UnityVersion(2020, 3, 1)
);

// Создание dummy DLL
var assemblies = Cpp2IL.Core.Cpp2IlApi.MakeDummyDLLs();
```

### Создание собственных плагинов

См. документацию в `docs/PluginSystem.md` для подробной информации о создании плагинов.

## Заключение

Cpp2IL предоставляет мощный и гибкий инструмент для анализа Unity IL2CPP приложений. Система плагинов и слоев обработки позволяет адаптировать инструмент под конкретные задачи, а поддержка множественных платформ делает его универсальным решением для реверс-инжиниринга Unity игр.
