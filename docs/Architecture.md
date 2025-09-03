# Архитектура Cpp2IL

## Общая архитектура

Cpp2IL построен по модульной архитектуре с четким разделением ответственности между компонентами:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Cpp2IL CLI    │    │  Cpp2IL.Core    │    │   LibCpp2IL     │
│   (Main App)    │◄──►│   (Core API)     │◄──►│ (Low-level Lib) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│    Plugins      │    │ Processing      │    │ Instruction     │
│   (Extensions)  │    │   Layers        │    │    Sets         │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Основные компоненты

### 1. Cpp2IL (CLI Application)

Главное консольное приложение, которое:
- Обрабатывает аргументы командной строки
- Координирует работу всех компонентов
- Управляет жизненным циклом анализа

**Ключевые файлы:**
- `Program.cs` - точка входа и основная логика
- `CommandLineArgs.cs` - обработка аргументов командной строки
- `ConsoleLogger.cs` - логирование в консоль

### 2. Cpp2IL.Core

Основная библиотека, предоставляющая API для работы с IL2CPP:

**Ключевые классы:**
- `Cpp2IlApi` - основной API для инициализации и работы
- `ApplicationAnalysisContext` - контекст анализа всего приложения
- `AssemblyAnalysisContext` - контекст анализа сборки
- `TypeAnalysisContext` - контекст анализа типа
- `MethodAnalysisContext` - контекст анализа метода

**Основные директории:**
- `Api/` - публичные API интерфейсы
- `Model/` - модели данных и контексты анализа
- `ProcessingLayers/` - слои обработки данных
- `OutputFormats/` - форматы вывода
- `InstructionSets/` - поддержка различных наборов инструкций
- `ISIL/` - промежуточный язык
- `Graphs/` - графы потока управления

### 3. LibCpp2IL

Низкоуровневая библиотека для работы с IL2CPP бинарными файлами:

**Основные возможности:**
- Парсинг IL2CPP метаданных
- Чтение бинарных файлов различных платформ
- Дисассемблирование кода
- Извлечение структур данных

**Поддерживаемые форматы:**
- PE (Windows)
- ELF (Linux)
- Mach-O (macOS)
- WebAssembly
- Nintendo Switch

### 4. Система плагинов

Архитектура плагинов позволяет расширять функциональность:

**Типы плагинов:**
- Output Formats - форматы вывода
- Processing Layers - слои обработки
- Game Path Handlers - обработчики путей к играм

**Реализованные плагины:**
- `Cpp2IL.Plugin.BuildReport` - отчеты о сборке
- `Cpp2IL.Plugin.ControlFlowGraph` - графы потока управления
- `Cpp2IL.Plugin.OrbisPkg` - поддержка PlayStation
- `Cpp2IL.Plugin.Pdb` - поддержка PDB файлов
- `Cpp2IL.Plugin.StrippedCodeRegSupport` - поддержка обфусцированного кода

## Поток данных

### 1. Инициализация
```
CLI Args → Cpp2IlApi.InitializeLibCpp2Il() → LibCpp2IL → ApplicationAnalysisContext
```

### 2. Обработка
```
ApplicationAnalysisContext → Processing Layers → Output Formats
```

### 3. Анализ (в разработке)
```
Disassembly → ISIL → Control Flow Graph → Analysis
```

## Модель данных

### Контексты анализа

Проект использует иерархическую систему контекстов:

```
ApplicationAnalysisContext
├── AssemblyAnalysisContext[]
│   ├── TypeAnalysisContext[]
│   │   ├── MethodAnalysisContext[]
│   │   ├── FieldAnalysisContext[]
│   │   └── PropertyAnalysisContext[]
│   └── ...
└── ...
```

### Обработка данных

Данные проходят через слои обработки:

1. **Raw Data** - сырые данные из IL2CPP
2. **Processing Layers** - преобразование и анализ
3. **Output Formats** - финальный вывод

## Расширяемость

### Создание плагинов

Плагины создаются путем:
1. Создания класса, наследующего от `Cpp2IlPlugin`
2. Добавления атрибута `[RegisterCpp2IlPlugin]`
3. Реализации необходимых интерфейсов

### Добавление новых форматов вывода

Новые форматы вывода реализуют интерфейс `IOutputFormat`:
- `OutputSingle(ApplicationAnalysisContext context, string outputRoot)`
- `OutputMultiple(ApplicationAnalysisContext context, string outputRoot)`

### Добавление слоев обработки

Слои обработки наследуются от `ProcessingLayer`:
- `Process(ApplicationAnalysisContext context)`
- Конфигурация через `ProcessingLayerConfiguration`
