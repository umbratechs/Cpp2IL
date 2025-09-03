# Control Flow Graph (CFG)

## Обзор

Control Flow Graph (CFG) - это представление программы в виде графа, где узлы представляют базовые блоки кода, а ребра - переходы между ними. В Cpp2IL CFG используется для анализа потока управления и является ключевым компонентом новой архитектуры анализа.

## Архитектура CFG

### Основные компоненты

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   ISIL Code     │───►│   CFG Builder   │───►│   CFG Graph      │
│                 │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Basic Blocks    │    │ Edge Detection  │    │ Graph Analysis  │
│                 │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Ключевые классы

### 1. ISILControlFlowGraph

Основной класс для представления графа потока управления:

```csharp
public class ISILControlFlowGraph
{
    public List<Block> Blocks { get; }
    public Dictionary<Block, List<Block>> Edges { get; }
    public Block EntryBlock { get; }
    public List<Block> ExitBlocks { get; }
}
```

### 2. Block

Представляет базовый блок - последовательность инструкций без переходов:

```csharp
public class Block
{
    public List<InstructionSetIndependentInstruction> Instructions { get; }
    public BlockType Type { get; }
    public ulong StartAddress { get; }
    public ulong EndAddress { get; }
}
```

### 3. BlockType

Типы базовых блоков:

- `Normal` - обычный блок
- `Entry` - точка входа
- `Exit` - точка выхода
- `Conditional` - условный переход
- `Unconditional` - безусловный переход

## Текущая реализация

### ✅ Реализовано

1. **Базовая структура графов**
   - Классы для представления блоков и графов
   - Система типов блоков
   - Хранение инструкций в блоках

2. **Построение графов**
   - Разбиение кода на базовые блоки
   - Обнаружение переходов между блоками
   - Создание связей в графе

3. **Базовый анализ**
   - Определение точек входа и выхода
   - Классификация типов блоков
   - Обнаружение простых паттернов

### 🔄 В процессе разработки

1. **Интеграция с ISIL**
   - Автоматическое создание CFG из ISIL кода
   - Обработка сложных переходов
   - Поддержка всех типов инструкций

2. **Продвинутый анализ**
   - Обнаружение циклов
   - Анализ доминирования
   - Поиск путей выполнения

3. **Оптимизация**
   - Удаление недостижимого кода
   - Объединение блоков
   - Упрощение графов

### ❌ Не реализовано

1. **Сложный анализ потока данных**
   - Анализ переменных
   - Отслеживание значений
   - Оптимизация выражений

2. **Восстановление высокоуровневой логики**
   - Восстановление циклов
   - Восстановление условий
   - Восстановление функций

## Примеры использования

### Создание CFG из ISIL кода

```csharp
var isilInstructions = GetIsilInstructions();
var cfg = ISILControlFlowGraph.FromInstructions(isilInstructions);
```

### Анализ графа

```csharp
// Получение всех блоков
var blocks = cfg.Blocks;

// Получение переходов из блока
var successors = cfg.Edges[block];

// Поиск циклов
var cycles = cfg.FindCycles();

// Анализ путей
var paths = cfg.FindPaths(entryBlock, exitBlock);
```

### Визуализация графа

```csharp
// Экспорт в DOT формат
var dotContent = cfg.ToDotFormat();

// Экспорт в JSON
var jsonContent = cfg.ToJson();
```

## Интеграция с плагинами

### Cpp2IL.Plugin.ControlFlowGraph

Плагин для генерации и экспорта графов потока управления:

**Возможности:**
- Создание CFG для каждого метода
- Экспорт в DOT формат
- Интеграция с Graphviz
- Визуализация потоков выполнения

**Использование:**
```bash
Cpp2IL-Win.exe --game-path=C:\Game --output-as=controlflowgraph
```

## Алгоритмы анализа

### 1. Построение базовых блоков

```csharp
public static List<Block> BuildBasicBlocks(List<Instruction> instructions)
{
    var blocks = new List<Block>();
    var currentBlock = new Block();
    
    foreach (var instruction in instructions)
    {
        currentBlock.Instructions.Add(instruction);
        
        if (IsTerminator(instruction))
        {
            blocks.Add(currentBlock);
            currentBlock = new Block();
        }
    }
    
    return blocks;
}
```

### 2. Обнаружение переходов

```csharp
public static Dictionary<Block, List<Block>> BuildEdges(List<Block> blocks)
{
    var edges = new Dictionary<Block, List<Block>>();
    
    foreach (var block in blocks)
    {
        var lastInstruction = block.Instructions.Last();
        var successors = GetSuccessors(lastInstruction, blocks);
        edges[block] = successors;
    }
    
    return edges;
}
```

### 3. Анализ доминирования

```csharp
public static Dictionary<Block, Block> ComputeDominators(ISILControlFlowGraph cfg)
{
    // Алгоритм Lengauer-Tarjan для вычисления доминирования
    // Реализация в разработке
}
```

## Планы развития

### Краткосрочные цели

1. **Завершение базовой функциональности**
   - Стабильное построение графов
   - Поддержка всех типов переходов
   - Базовые алгоритмы анализа

2. **Интеграция с ISIL**
   - Автоматическое создание CFG
   - Обработка сложных инструкций
   - Тестирование на реальных данных

### Долгосрочные цели

1. **Продвинутый анализ**
   - Анализ потока данных
   - Оптимизация кода
   - Восстановление логики

2. **Инструменты визуализации**
   - Интерактивные графы
   - Анимация выполнения
   - Интеграция с IDE

## Заключение

Control Flow Graph представляет собой важный компонент новой архитектуры Cpp2IL для анализа кода. Хотя базовая структура уже реализована, требуется значительная доработка для полной интеграции с ISIL и реализации продвинутых алгоритмов анализа.
