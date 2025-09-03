# Система плагинов Cpp2IL

## Обзор

Система плагинов Cpp2IL обеспечивает расширяемость функциональности через динамическую загрузку модулей. Плагины могут добавлять новые форматы вывода, слои обработки, обработчики путей к играм и другие возможности.

## Архитектура плагинов

### Основные компоненты

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Plugin Manager │    │   Plugin Base   │    │  Plugin Types   │
│                 │◄──►│                 │◄──►│                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Load Plugins  │    │   OnLoad()      │    │ Output Formats  │
│                 │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Process Game   │    │   OnFinish()    │    │ Processing      │
│      Paths      │    │                 │    │   Layers       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Типы плагинов

### 1. Output Format Plugins

Плагины, добавляющие новые форматы вывода результатов анализа.

**Интерфейс:** `IOutputFormat`
- `OutputSingle()` - вывод в один файл
- `OutputMultiple()` - вывод в несколько файлов

**Реализованные плагины:**
- `Cpp2IL.Plugin.BuildReport` - генерация отчетов о сборке
- `Cpp2IL.Plugin.ControlFlowGraph` - создание графов потока управления

### 2. Processing Layer Plugins

Плагины, добавляющие новые слои обработки данных.

**Базовый класс:** `ProcessingLayer`
- `Process()` - обработка данных
- `Configuration` - конфигурация слоя

**Встроенные слои:**
- `AttributeInjectorProcessingLayer` - инъекция атрибутов
- `CallAnalysisProcessingLayer` - анализ вызовов
- `StableRenamingProcessingLayer` - стабильное переименование
- `NativeMethodDetectionProcessingLayer` - обнаружение нативных методов
- `DeobfuscationMapProcessingLayer` - деобфускация
- `AttributeAnalysisProcessingLayer` - анализ атрибутов

### 3. Game Path Handler Plugins

Плагины, добавляющие поддержку новых форматов игр и платформ.

**Метод:** `HandleGamePath()`
- Обработка специфичных путей к играм
- Извлечение файлов из архивов
- Поддержка новых платформ

**Реализованные плагины:**
- `Cpp2IL.Plugin.OrbisPkg` - поддержка PlayStation форматов

## Реализованные плагины

### 1. Cpp2IL.Plugin.BuildReport

**Назначение:** Генерация подробных отчетов о процессе анализа.

**Возможности:**
- Статистика по типам, методам, полям
- Информация о времени выполнения
- Детали по обработке данных
- Экспорт в различные форматы

**Использование:**
```bash
Cpp2IL-Win.exe --game-path=C:\Game --output-as=buildreport
```

### 2. Cpp2IL.Plugin.ControlFlowGraph

**Назначение:** Создание графов потока управления для методов.

**Возможности:**
- Визуализация потоков выполнения
- Анализ путей кода
- Экспорт в DOT формат
- Интеграция с Graphviz

**Использование:**
```bash
Cpp2IL-Win.exe --game-path=C:\Game --output-as=controlflowgraph
```

### 3. Cpp2IL.Plugin.OrbisPkg

**Назначение:** Поддержка PlayStation игр в формате PKG.

**Возможности:**
- Извлечение файлов из PKG архивов
- Поддержка PlayStation специфичных форматов
- Автоматическое определение Unity версии

**Зависимости:**
- LibOrbisPkg (LGPL v3)

### 4. Cpp2IL.Plugin.Pdb

**Назначение:** Поддержка PDB файлов для отладочной информации.

**Возможности:**
- Извлечение символов отладки
- Восстановление имен функций
- Информация о строках кода

**Зависимости:**
- AssetRipper.Bindings.MsPdbCore

### 5. Cpp2IL.Plugin.StrippedCodeRegSupport

**Назначение:** Поддержка обфусцированного и stripped кода.

**Возможности:**
- Обработка удаленных символов
- Восстановление информации о регистрах
- Поддержка обфусцированных бинарных файлов

## Создание собственного плагина

### Шаг 1: Создание проекта

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <AssemblyName>Cpp2IL.Plugin.MyPlugin</AssemblyName>
  </PropertyGroup>
  
  <ItemGroup>
    <ProjectReference Include="..\Cpp2IL.Core\Cpp2IL.Core.csproj" />
  </ItemGroup>
</Project>
```

### Шаг 2: Создание класса плагина

```csharp
using Cpp2IL.Core;
using Cpp2IL.Core.Attributes;

[RegisterCpp2IlPlugin]
public class MyPlugin : Cpp2IlPlugin
{
    public override string Name => "My Plugin";
    
    public override void OnLoad()
    {
        // Инициализация плагина
    }
    
    public override void OnFinish()
    {
        // Очистка ресурсов
    }
}
```

### Шаг 3: Реализация функциональности

#### Output Format Plugin

```csharp
public class MyOutputFormat : IOutputFormat
{
    public string Name => "myformat";
    
    public void OutputSingle(ApplicationAnalysisContext context, string outputRoot)
    {
        // Реализация вывода
    }
}
```

#### Processing Layer Plugin

```csharp
public class MyProcessingLayer : ProcessingLayer
{
    public override string Name => "mylayer";
    
    public override void Process(ApplicationAnalysisContext context)
    {
        // Реализация обработки
    }
}
```

### Шаг 4: Регистрация плагина

Плагин автоматически регистрируется через атрибут `[RegisterCpp2IlPlugin]`.

## Жизненный цикл плагина

### 1. Загрузка
- Сканирование директории плагинов
- Загрузка DLL файлов
- Создание экземпляров плагинов
- Вызов `OnLoad()`

### 2. Выполнение
- Обработка путей к играм
- Выполнение слоев обработки
- Генерация выходных форматов

### 3. Завершение
- Вызов `OnFinish()`
- Очистка ресурсов
- Выгрузка плагинов

## Конфигурация плагинов

### Параметры командной строки

```bash
# Использование плагина
Cpp2IL-Win.exe --game-path=C:\Game --use-processor=mylayer

# Конфигурация плагина
Cpp2IL-Win.exe --game-path=C:\Game --use-processor=mylayer --processor-config=key=value
```

### Конфигурация в коде

```csharp
public class MyProcessingLayer : ProcessingLayer
{
    public override ProcessingLayerConfiguration Configuration => new()
    {
        ["key"] = "value",
        ["option"] = "setting"
    };
}
```

## Отладка плагинов

### Логирование

```csharp
using Cpp2IL.Core.Logging;

Logger.InfoNewline("Plugin message");
Logger.VerboseNewline("Debug information");
Logger.WarnNewline("Warning message");
Logger.ErrorNewline("Error message");
```

### Обработка ошибок

```csharp
try
{
    // Код плагина
}
catch (Exception e)
{
    Logger.ErrorNewline($"Plugin error: {e.Message}");
    throw;
}
```

## Заключение

Система плагинов Cpp2IL обеспечивает мощную расширяемость функциональности. Существующие плагины демонстрируют различные возможности системы, а архитектура позволяет легко создавать новые плагины для специфических задач.
