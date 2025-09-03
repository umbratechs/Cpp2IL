# Поддерживаемые платформы и форматы

## Обзор

Cpp2IL поддерживает широкий спектр платформ и форматов файлов, что позволяет анализировать Unity IL2CPP приложения на различных устройствах и операционных системах.

## Поддерживаемые платформы

### 1. Windows

**Поддерживаемые архитектуры:**
- x86 (32-bit)
- x64 (64-bit)

**Форматы файлов:**
- PE (Portable Executable)
- DLL файлы
- EXE файлы

**Автоопределение:**
- Автоматическое обнаружение Unity версии из файла версии
- Поиск GameAssembly.dll и global-metadata.dat
- Поддержка различных структур папок

### 2. macOS

**Поддерживаемые архитектуры:**
- x64 (Intel)
- ARM64 (Apple Silicon)

**Форматы файлов:**
- Mach-O
- .app bundles
- .dylib файлы

**Структура:**
```
Game.app/
├── Contents/
│   ├── MacOS/
│   │   └── GameExecutable
│   ├── Frameworks/
│   │   └── GameAssembly.dylib
│   └── Resources/
│       └── Data/
│           └── il2cpp_data/
│               └── Metadata/
│                   └── global-metadata.dat
```

### 3. Linux

**Поддерживаемые архитектуры:**
- x64 (64-bit)
- ARM64 (в разработке)

**Форматы файлов:**
- ELF (Executable and Linkable Format)
- .so файлы

**Структура:**
```
Game/
├── GameExecutable
├── GameAssembly.so
└── Game_Data/
    └── il2cpp_data/
        └── Metadata/
            └── global-metadata.dat
```

### 4. Android

**Поддерживаемые архитектуры:**
- ARMv7 (32-bit)
- ARM64 (64-bit)
- x86 (32-bit, эмуляция)

**Форматы файлов:**
- APK (Android Package)
- APKM (APK Mod)
- XAPK (Extended APK)

**Автоизвлечение:**
- Автоматическое извлечение из APK архивов
- Поиск libil2cpp.so и global-metadata.dat
- Поддержка различных структур APK

### 5. iOS

**Поддерживаемые архитектуры:**
- ARM64 (64-bit)

**Форматы файлов:**
- IPA (iOS App Store Package)
- TIPA (Tweaked IPA)

**Структура:**
```
App.ipa/
├── Payload/
│   └── App.app/
│       ├── App (executable)
│       ├── Frameworks/
│       │   └── GameAssembly.framework/
│       │       └── GameAssembly
│       └── Data/
│           └── il2cpp_data/
│               └── Metadata/
│                   └── global-metadata.dat
```

### 6. WebAssembly

**Поддерживаемые архитектуры:**
- WebAssembly (WASM)

**Форматы файлов:**
- .wasm файлы
- .framework.js файлы (для экспортов)

**Особенности:**
- Поддержка обфусцированных экспортов
- Анализ секций имен
- Интеграция с JavaScript фреймворком

### 7. Nintendo Switch

**Поддерживаемые архитектуры:**
- ARM64 (64-bit)

**Форматы файлов:**
- NSO (Nintendo Switch Object)
- NRO (Nintendo Switch Relocatable Object)

**Особенности:**
- Поддержка специфичных форматов Nintendo
- Обработка релокаций
- Анализ метаданных Switch

### 8. PlayStation

**Поддерживаемые архитектуры:**
- x64 (64-bit)

**Форматы файлов:**
- PKG (PlayStation Package)
- SELF (Sony ELF)

**Плагин:** `Cpp2IL.Plugin.OrbisPkg`
- Извлечение файлов из PKG архивов
- Поддержка PlayStation специфичных форматов
- Автоматическое определение Unity версии

## Поддерживаемые Unity версии

### IL2CPP Metadata версии

- **24.0** - Unity 2018.1+
- **24.1** - Unity 2018.2+
- **24.2** - Unity 2018.3+
- **24.3** - Unity 2018.4+
- **24.4** - Unity 2019.1+
- **24.5** - Unity 2019.2+
- **27.0** - Unity 2019.3+
- **27.1** - Unity 2019.4+

### Автоопределение версии

Cpp2IL автоматически определяет Unity версию из:
1. Файла версии исполняемого файла (Windows)
2. globalgamemanagers файла (другие платформы)
3. Метаданных IL2CPP

## Наборы инструкций

### Реализованные Instruction Sets

1. **X86InstructionSet** (x86/x64)
   - Поддержка всех основных инструкций x86
   - Обработка режимов адресации
   - Поддержка SIMD инструкций

2. **NewArmV8InstructionSet** (ARM64)
   - Полная поддержка ARM64
   - Обработка условных инструкций
   - Поддержка векторных операций

3. **ArmV7InstructionSet** (ARMv7)
   - Поддержка ARMv7 архитектуры
   - Обработка Thumb инструкций
   - Совместимость с Android

4. **WasmInstructionSet** (WebAssembly)
   - Поддержка WASM инструкций
   - Обработка стековой архитектуры
   - Анализ экспортов

### Расширяемость

Новые наборы инструкций могут быть добавлены через:
1. Создание класса, наследующего от `Cpp2IlInstructionSet`
2. Регистрация в `InstructionSetRegistry`
3. Реализация необходимых методов дисассемблирования

## Обработка файлов

### Автоматическое извлечение

**APK файлы:**
```bash
Cpp2IL-Win.exe --game-path=game.apk
```

**IPA файлы:**
```bash
Cpp2IL-Win.exe --game-path=game.ipa
```

**PKG файлы:**
```bash
Cpp2IL-Win.exe --game-path=game.pkg
```

### Ручное указание файлов

```bash
Cpp2IL-Win.exe --game-path=C:\Game --exe-name=GameExecutable
```

### Поддержка архивов

- **ZIP** - стандартные архивы
- **APK** - Android пакеты
- **IPA** - iOS пакеты
- **PKG** - PlayStation пакеты (через плагин)

## Специальные возможности

### WebAssembly Framework

Для WASM файлов с обфусцированными экспортами:
```bash
Cpp2IL-Win.exe --game-path=game.wasm --wasm-framework-file=framework.js
```

### Обфусцированные файлы

Поддержка обфусцированных и stripped файлов через:
- `Cpp2IL.Plugin.StrippedCodeRegSupport`
- `DeobfuscationMapProcessingLayer`

### Отладочная информация

Поддержка PDB файлов через:
- `Cpp2IL.Plugin.Pdb`
- Восстановление символов
- Информация о строках кода

## Ограничения

### Текущие ограничения

1. **WebAssembly**
   - Ограниченная поддержка сложных экспортов
   - Требуется framework.js для обфусцированных файлов

2. **Nintendo Switch**
   - Базовая поддержка NSO/NRO форматов
   - Ограниченный анализ метаданных

3. **ARM64**
   - Не все SIMD инструкции поддерживаются
   - Ограниченная поддержка специфичных операций

### Планы развития

1. **Расширение поддержки**
   - Новые архитектуры процессоров
   - Экзотические платформы
   - Специфичные форматы файлов

2. **Улучшение автоопределения**
   - Более точное определение версий
   - Поддержка новых форматов
   - Улучшенная обработка ошибок

## Заключение

Cpp2IL обеспечивает широкую поддержку различных платформ и форматов файлов, что делает его универсальным инструментом для анализа Unity IL2CPP приложений. Система плагинов позволяет расширять поддержку новых платформ и форматов без изменения основной кодовой базы.
