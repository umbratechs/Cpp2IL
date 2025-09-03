# План разработки ISIL и Control Flow Graph

## Обзор

Этот документ описывает текущее состояние кода ISIL и Control Flow Graph, а также план разработки для их доработки и интеграции.

## Текущее состояние кода

### ISIL (Instruction-Set-Independent Language)

#### Основные файлы:

1. **`Cpp2IL.Core/ISIL/`** - основная директория ISIL
   - `IsilBuilder.cs` - построитель ISIL инструкций
   - `InstructionSetIndependentInstruction.cs` - основная инструкция ISIL
   - `InstructionSetIndependentOpCode.cs` - коды операций
   - `InstructionSetIndependentOperand.cs` - операнды
   - `IsilMnemonic.cs` - мнемоники
   - `IsilFlowControl.cs` - управление потоком
   - `IsilCondition.cs` - условия

2. **`Cpp2IL.Core/InstructionSets/`** - наборы инструкций
   - `X86InstructionSet.cs` - реализация для x86/x64
   - `NewArmV8InstructionSet.cs` - реализация для ARM64
   - `ArmV7InstructionSet.cs` - реализация для ARMv7
   - `WasmInstructionSet.cs` - реализация для WebAssembly

#### Ключевые методы:

```csharp
// В каждом InstructionSet
public override List<InstructionSetIndependentInstruction> GetIsilFromMethod(MethodAnalysisContext context)
{
    var builder = new IsilBuilder();
    // Конвертация инструкций в ISIL
    return builder.BackingStatementList;
}
```

### Control Flow Graph (CFG)

#### Основные файлы:

1. **`Cpp2IL.Core/Graphs/`** - основная директория CFG
   - `ISILControlFlowGraph.cs` - основной класс графа
   - `Block.cs` - блок базовых блоков
   - `BlockType.cs` - типы блоков

2. **`Cpp2IL.Core/Graphs/Processors/`** - процессоры графов
   - (пустая директория - требует реализации)

#### Ключевые методы:

```csharp
// В MethodAnalysisContext.cs
public ISILControlFlowGraph? ControlFlowGraph;

// Создание CFG из ISIL
ControlFlowGraph = new ISILControlFlowGraph();
ControlFlowGraph.Build(ConvertedIsil);
```

## План разработки

### Этап 1: Завершение ISIL (1-2 недели)

#### 1.1 Расширение IsilBuilder

**Файл:** `Cpp2IL.Core/ISIL/IsilBuilder.cs`

**Задачи:**
- Добавить недостающие операции (SIMD, FPU)
- Улучшить обработку ошибок
- Добавить валидацию инструкций

**Пример кода:**
```csharp
// Добавить в IsilBuilder.cs
public void SimdAdd(ulong instructionAddress, InstructionSetIndependentOperand dest, 
    InstructionSetIndependentOperand left, InstructionSetIndependentOperand right)
{
    AddInstruction(new(InstructionSetIndependentOpCode.SimdAdd, instructionAddress, 
        IsilFlowControl.Continue, dest, left, right));
}

public void FpuOperation(ulong instructionAddress, InstructionSetIndependentOpCode opCode,
    InstructionSetIndependentOperand dest, InstructionSetIndependentOperand src)
{
    AddInstruction(new(opCode, instructionAddress, IsilFlowControl.Continue, dest, src));
}
```

#### 1.2 Расширение InstructionSetIndependentOpCode

**Файл:** `Cpp2IL.Core/ISIL/InstructionSetIndependentOpCode.cs`

**Задачи:**
- Добавить SIMD операции
- Добавить FPU операции
- Добавить специфичные операции архитектур

**Пример кода:**
```csharp
// Добавить в перечисление
SimdAdd,
SimdSub,
SimdMul,
SimdDiv,
FpuAdd,
FpuSub,
FpuMul,
FpuDiv,
FpuSqrt,
FpuSin,
FpuCos,
```

#### 1.3 Улучшение конвертации в InstructionSets

**Файлы:** 
- `Cpp2IL.Core/InstructionSets/X86InstructionSet.cs`
- `Cpp2IL.Core/InstructionSets/NewArmV8InstructionSet.cs`
- `Cpp2IL.Core/InstructionSets/ArmV7InstructionSet.cs`

**Задачи:**
- Добавить поддержку SIMD инструкций
- Добавить поддержку FPU инструкций
- Улучшить обработку сложных инструкций

**Пример кода:**
```csharp
// В X86InstructionSet.cs добавить в ConvertInstructionStatement
case Mnemonic.Addps:
case Mnemonic.Addpd:
    builder.SimdAdd(instruction.IP, ConvertOperand(instruction, 0), 
        ConvertOperand(instruction, 1), ConvertOperand(instruction, 2));
    break;
case Mnemonic.Addss:
case Mnemonic.Addsd:
    builder.FpuOperation(instruction.IP, InstructionSetIndependentOpCode.FpuAdd,
        ConvertOperand(instruction, 0), ConvertOperand(instruction, 1));
    break;
```

### Этап 2: Интеграция ISIL с CFG (1-2 недели)

#### 2.1 Улучшение ISILControlFlowGraph

**Файл:** `Cpp2IL.Core/Graphs/ISILControlFlowGraph.cs`

**Задачи:**
- Исправить логику построения графов
- Добавить обработку сложных переходов
- Улучшить обнаружение циклов

**Пример кода:**
```csharp
// Добавить методы анализа
public List<Block> FindCycles()
{
    var cycles = new List<Block>();
    var visited = new HashSet<Block>();
    var recursionStack = new HashSet<Block>();
    
    foreach (var block in blockSet)
    {
        if (!visited.Contains(block))
        {
            FindCyclesDFS(block, visited, recursionStack, cycles);
        }
    }
    
    return cycles;
}

private void FindCyclesDFS(Block block, HashSet<Block> visited, 
    HashSet<Block> recursionStack, List<Block> cycles)
{
    visited.Add(block);
    recursionStack.Add(block);
    
    foreach (var successor in GetSuccessors(block))
    {
        if (!visited.Contains(successor))
        {
            FindCyclesDFS(successor, visited, recursionStack, cycles);
        }
        else if (recursionStack.Contains(successor))
        {
            cycles.Add(block);
        }
    }
    
    recursionStack.Remove(block);
}
```

#### 2.2 Создание процессоров графов

**Директория:** `Cpp2IL.Core/Graphs/Processors/`

**Задачи:**
- Создать базовый класс процессора
- Реализовать процессоры анализа
- Добавить оптимизации

**Файлы для создания:**

1. **`Cpp2IL.Core/Graphs/Processors/IGraphProcessor.cs`**
```csharp
public interface IGraphProcessor
{
    void Process(ISILControlFlowGraph graph);
    string Name { get; }
}
```

2. **`Cpp2IL.Core/Graphs/Processors/DeadCodeEliminationProcessor.cs`**
```csharp
public class DeadCodeEliminationProcessor : IGraphProcessor
{
    public string Name => "DeadCodeElimination";
    
    public void Process(ISILControlFlowGraph graph)
    {
        // Удаление недостижимого кода
        var reachableBlocks = FindReachableBlocks(graph);
        RemoveUnreachableBlocks(graph, reachableBlocks);
    }
}
```

3. **`Cpp2IL.Core/Graphs/Processors/BlockMergingProcessor.cs`**
```csharp
public class BlockMergingProcessor : IGraphProcessor
{
    public string Name => "BlockMerging";
    
    public void Process(ISILControlFlowGraph graph)
    {
        // Объединение последовательных блоков
        MergeSequentialBlocks(graph);
    }
}
```

#### 2.3 Интеграция в MethodAnalysisContext

**Файл:** `Cpp2IL.Core/Model/Contexts/MethodAnalysisContext.cs`

**Задачи:**
- Добавить автоматическое создание CFG
- Добавить применение процессоров
- Добавить кэширование результатов

**Пример кода:**
```csharp
// Добавить в MethodAnalysisContext
public void BuildControlFlowGraph()
{
    if (ConvertedIsil == null)
    {
        ConvertedIsil = AppContext.InstructionSet.GetIsilFromMethod(this);
    }
    
    ControlFlowGraph = new ISILControlFlowGraph();
    ControlFlowGraph.Build(ConvertedIsil);
    
    // Применить процессоры
    ApplyGraphProcessors();
}

private void ApplyGraphProcessors()
{
    if (ControlFlowGraph == null) return;
    
    var processors = new IGraphProcessor[]
    {
        new DeadCodeEliminationProcessor(),
        new BlockMergingProcessor()
    };
    
    foreach (var processor in processors)
    {
        processor.Process(ControlFlowGraph);
    }
}
```

### Этап 3: Анализ потока данных (2-3 недели)

#### 3.1 Создание анализатора потока данных

**Директория:** `Cpp2IL.Core/Analysis/`

**Файлы для создания:**

1. **`Cpp2IL.Core/Analysis/DataFlowAnalyzer.cs`**
```csharp
public class DataFlowAnalyzer
{
    public Dictionary<Block, Dictionary<string, ValueSet>> Analyze(ISILControlFlowGraph graph)
    {
        var result = new Dictionary<Block, Dictionary<string, ValueSet>>();
        
        // Инициализация
        foreach (var block in graph.Blocks)
        {
            result[block] = new Dictionary<string, ValueSet>();
        }
        
        // Анализ потока данных
        AnalyzeDataFlow(graph, result);
        
        return result;
    }
    
    private void AnalyzeDataFlow(ISILControlFlowGraph graph, 
        Dictionary<Block, Dictionary<string, ValueSet>> dataFlow)
    {
        // Реализация алгоритма анализа потока данных
    }
}
```

2. **`Cpp2IL.Core/Analysis/ValueSet.cs`**
```csharp
public class ValueSet
{
    public HashSet<object> Values { get; } = new();
    public bool IsConstant => Values.Count == 1;
    public bool IsUnknown => Values.Count == 0;
    
    public void AddValue(object value)
    {
        Values.Add(value);
    }
    
    public void Merge(ValueSet other)
    {
        foreach (var value in other.Values)
        {
            Values.Add(value);
        }
    }
}
```

#### 3.2 Интеграция с ISIL

**Файл:** `Cpp2IL.Core/ISIL/InstructionSetIndependentInstruction.cs`

**Задачи:**
- Добавить анализ эффектов инструкций
- Добавить отслеживание переменных
- Добавить оптимизации

**Пример кода:**
```csharp
// Добавить в InstructionSetIndependentInstruction
public class InstructionEffect
{
    public HashSet<string> ReadVariables { get; } = new();
    public HashSet<string> WrittenVariables { get; } = new();
    public bool HasSideEffects { get; set; }
}

public InstructionEffect AnalyzeEffect()
{
    var effect = new InstructionEffect();
    
    switch (OpCode)
    {
        case InstructionSetIndependentOpCode.Move:
            AnalyzeMoveEffect(effect);
            break;
        case InstructionSetIndependentOpCode.Add:
            AnalyzeAddEffect(effect);
            break;
        // ... другие случаи
    }
    
    return effect;
}
```

### Этап 4: Оптимизации и восстановление логики (2-3 недели)

#### 4.1 Оптимизации кода

**Директория:** `Cpp2IL.Core/Optimizations/`

**Файлы для создания:**

1. **`Cpp2IL.Core/Optimizations/ConstantFoldingOptimizer.cs`**
```csharp
public class ConstantFoldingOptimizer
{
    public void Optimize(List<InstructionSetIndependentInstruction> instructions)
    {
        for (int i = 0; i < instructions.Count; i++)
        {
            if (CanFoldConstant(instructions[i]))
            {
                var folded = FoldConstant(instructions[i]);
                instructions[i] = folded;
            }
        }
    }
    
    private bool CanFoldConstant(InstructionSetIndependentInstruction instruction)
    {
        // Проверка возможности свертки констант
        return instruction.OpCode == InstructionSetIndependentOpCode.Add &&
               instruction.Operands[1].IsConstant &&
               instruction.Operands[2].IsConstant;
    }
}
```

2. **`Cpp2IL.Core/Optimizations/DeadCodeOptimizer.cs`**
```csharp
public class DeadCodeOptimizer
{
    public void Optimize(List<InstructionSetIndependentInstruction> instructions)
    {
        var liveVariables = new HashSet<string>();
        
        // Обратный проход для поиска живых переменных
        for (int i = instructions.Count - 1; i >= 0; i--)
        {
            var effect = instructions[i].AnalyzeEffect();
            
            if (effect.WrittenVariables.Any(v => !liveVariables.Contains(v)))
            {
                // Удалить мертвый код
                instructions.RemoveAt(i);
            }
            else
            {
                liveVariables.UnionWith(effect.ReadVariables);
            }
        }
    }
}
```

#### 4.2 Восстановление высокоуровневой логики

**Директория:** `Cpp2IL.Core/Reconstruction/`

**Файлы для создания:**

1. **`Cpp2IL.Core/Reconstruction/LoopReconstructor.cs`**
```csharp
public class LoopReconstructor
{
    public List<LoopStructure> ReconstructLoops(ISILControlFlowGraph graph)
    {
        var loops = new List<LoopStructure>();
        var cycles = graph.FindCycles();
        
        foreach (var cycle in cycles)
        {
            var loop = AnalyzeLoopStructure(graph, cycle);
            loops.Add(loop);
        }
        
        return loops;
    }
    
    private LoopStructure AnalyzeLoopStructure(ISILControlFlowGraph graph, Block cycle)
    {
        // Анализ структуры цикла
        return new LoopStructure
        {
            Header = FindLoopHeader(graph, cycle),
            Body = FindLoopBody(graph, cycle),
            Condition = AnalyzeLoopCondition(cycle)
        };
    }
}
```

2. **`Cpp2IL.Core/Reconstruction/ConditionReconstructor.cs`**
```csharp
public class ConditionReconstructor
{
    public List<ConditionStructure> ReconstructConditions(ISILControlFlowGraph graph)
    {
        var conditions = new List<ConditionStructure>();
        
        foreach (var block in graph.Blocks)
        {
            if (block.BlockType == BlockType.Conditional)
            {
                var condition = AnalyzeCondition(block);
                conditions.Add(condition);
            }
        }
        
        return conditions;
    }
}
```

## Практические шаги для начала разработки

### Шаг 1: Настройка окружения

1. **Клонируйте репозиторий:**
```bash
git clone https://github.com/SamboyCoding/Cpp2IL.git
cd Cpp2IL
```

2. **Откройте решение в IDE:**
```bash
# Visual Studio
Cpp2IL.sln

# VS Code
code .
```

3. **Восстановите зависимости:**
```bash
dotnet restore
```

### Шаг 2: Изучение существующего кода

1. **Изучите ISIL:**
   - `Cpp2IL.Core/ISIL/IsilBuilder.cs` - как создаются инструкции
   - `Cpp2IL.Core/ISIL/InstructionSetIndependentOpCode.cs` - доступные операции
   - `Cpp2IL.Core/InstructionSets/X86InstructionSet.cs` - пример конвертации

2. **Изучите CFG:**
   - `Cpp2IL.Core/Graphs/ISILControlFlowGraph.cs` - построение графов
   - `Cpp2IL.Core/Graphs/Block.cs` - структура блоков
   - `Cpp2IL.Core/Model/Contexts/MethodAnalysisContext.cs` - интеграция

### Шаг 3: Начните с простых улучшений

1. **Добавьте недостающие операции в IsilBuilder:**
```csharp
// В Cpp2IL.Core/ISIL/IsilBuilder.cs
public void SimdMultiply(ulong instructionAddress, InstructionSetIndependentOperand dest, 
    InstructionSetIndependentOperand left, InstructionSetIndependentOperand right)
{
    AddInstruction(new(InstructionSetIndependentOpCode.SimdMultiply, instructionAddress, 
        IsilFlowControl.Continue, dest, left, right));
}
```

2. **Добавьте поддержку в X86InstructionSet:**
```csharp
// В Cpp2IL.Core/InstructionSets/X86InstructionSet.cs
case Mnemonic.Mulps:
case Mnemonic.Mulpd:
    builder.SimdMultiply(instruction.IP, ConvertOperand(instruction, 0), 
        ConvertOperand(instruction, 1), ConvertOperand(instruction, 2));
    break;
```

3. **Создайте простой процессор графа:**
```csharp
// Создайте Cpp2IL.Core/Graphs/Processors/SimpleOptimizer.cs
public class SimpleOptimizer : IGraphProcessor
{
    public string Name => "SimpleOptimizer";
    
    public void Process(ISILControlFlowGraph graph)
    {
        // Простая оптимизация - удаление пустых блоков
        RemoveEmptyBlocks(graph);
    }
    
    private void RemoveEmptyBlocks(ISILControlFlowGraph graph)
    {
        // Реализация
    }
}
```

### Шаг 4: Тестирование

1. **Создайте тесты:**
```csharp
// В Cpp2IL.Core.Tests/ создайте новый тест
[Fact]
public void TestSimdOperation()
{
    // Тест новой SIMD операции
}
```

2. **Запустите существующие тесты:**
```bash
dotnet test
```

## Рекомендации по разработке

### 1. Постепенное развитие
- Начинайте с малых изменений
- Тестируйте каждое изменение
- Документируйте новые возможности

### 2. Следование архитектуре
- Используйте существующие паттерны
- Следуйте соглашениям именования
- Добавляйте комментарии к сложному коду

### 3. Обратная совместимость
- Не ломайте существующий функционал
- Добавляйте новые возможности как опциональные
- Поддерживайте старые форматы вывода

### 4. Производительность
- Оптимизируйте критичные пути
- Используйте кэширование где возможно
- Избегайте избыточных вычислений

## Заключение

Этот план предоставляет структурированный подход к разработке ISIL и Control Flow Graph. Начните с простых улучшений и постепенно переходите к более сложным задачам. Каждый этап строится на предыдущем, что обеспечивает стабильное развитие функциональности.
