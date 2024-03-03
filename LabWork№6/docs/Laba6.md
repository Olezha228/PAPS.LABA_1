Лабораторная работа 6

# Порождающие шаблоны:
1. Фабричный метод (Factory Method):
```c#
public abstract class WellLoggingFactory
{
    public abstract Log CreateLog();
}

public class GammaRayLogFactory : WellLoggingFactory
{
    public override Log CreateLog()
    {
        return new GammaRayLog();
    }
}

public class ResistivityLogFactory : WellLoggingFactory
{
    public override Log CreateLog()
    {
        return new ResistivityLog();
    }
}

// Пример использования:
WellLoggingFactory factory = new GammaRayLogFactory();
Log log = factory.CreateLog();
```

Шаблон "Фабричный метод" представляет собой порождающий шаблон проектирования, который предоставляет метод для создания объектов в суперклассе, позволяя подклассам изменять тип создаваемых объектов.

В данном коде:

- WellLoggingFactory - абстрактный класс, определяющий метод CreateLog(), который должен быть реализован в подклассах.
- GammaRayLogFactory и ResistivityLogFactory - конкретные фабрики, реализующие метод CreateLog() и создающие соответствующие объекты.
- Log - абстрактный класс или интерфейс, представляющий создаваемый объект.
- GammaRayLog и ResistivityLog - конкретные классы, реализующие интерфейс Log.

При использовании фабричного метода создается экземпляр конкретной фабрики (GammaRayLogFactory или ResistivityLogFactory), затем вызывается метод CreateLog(), который создает объект (экземпляр GammaRayLog или ResistivityLog).

```sql
          +---------------------+
          |    WellLoggingFactory |
          +---------------------+
          | [abstract]           |
          | +CreateLog()         |
          +---------------------+
                   /_\
                    |
    +-----------------------------+
    |        |                          |
+-----------------+        +------------------+
|GammaRayLogFactory|     |ResistivityLogFactory|
+-----------------+        +------------------+
| +CreateLog()    |        | +CreateLog()     |
+-----------------+        +------------------+
                   /_\
                    |
          +----------------+
          |        Log     |
          +----------------+
                   /_\
                    |
          +----------------+
          | GammaRayLog    |
          +----------------+
          | +Method()      |
          +----------------+
          +----------------+
          |ResistivityLog  |
          +----------------+
          | +Method()      |
          +----------------+

```

2. Абстрактная фабрика (Abstract Factory):
```c#
public abstract class WellLoggingAbstractFactory
{
    public abstract Log CreateGammaRayLog();
    public abstract Log CreateResistivityLog();
}

public class GeophysicalWellLoggingFactory : WellLoggingAbstractFactory
{
    public override Log CreateGammaRayLog()
    {
        return new GammaRayLog();
    }

    public override Log CreateResistivityLog()
    {
        return new ResistivityLog();
    }
}

// Пример использования:
WellLoggingAbstractFactory factory = new GeophysicalWellLoggingFactory();
Log gammaRayLog = factory.CreateGammaRayLog();
Log resistivityLog = factory.CreateResistivityLog();
```

Шаблон "Абстрактная фабрика" представляет собой порождающий шаблон проектирования, который предоставляет интерфейс для создания семейств взаимосвязанных или взаимозависимых объектов без указания их конкретных классов.

В данном коде:

- WellLoggingAbstractFactory - абстрактный класс, определяющий методы CreateGammaRayLog() и CreateResistivityLog(), которые должны быть реализованы в подклассах.
- GeophysicalWellLoggingFactory - конкретная фабрика, реализующая методы создания объектов.
- Log, GammaRayLog и ResistivityLog - классы, представляющие создаваемые объекты.

При использовании абстрактной фабрики создается экземпляр конкретной фабрики (GeophysicalWellLoggingFactory), затем вызываются методы создания объектов (CreateGammaRayLog() и CreateResistivityLog()), которые создают соответствующие объекты.

Вот UML-диаграмма классов для данного примера:
```sql
          +-------------------------+
          |WellLoggingAbstractFactory|
          +-------------------------+
          | [abstract]              |
          | +CreateGammaRayLog()    |
          | +CreateResistivityLog() |
          +-------------------------+
                   /_\
                    |
         +-------------------------+
         |                                 |
+---------------------------------+    +----------------------------------+
|GeophysicalWellLoggingFactory    |    | OtherWellLoggingFactory          |
+---------------------------------+    +----------------------------------+
| +CreateGammaRayLog()            |    | +CreateGammaRayLog()             |
| +CreateResistivityLog()         |    | +CreateResistivityLog()          |
+---------------------------------+    +----------------------------------+
                   /_\                                          /_\
                    |                                               |
          +-----------------+                             +-----------------+
          |       Log       |                             |         Log     |
          +-----------------+                             +-----------------+
          +-----------------+                             +-----------------+
          |  GammaRayLog  |                               | ResistivityLog  |
          +-----------------+                             +-----------------+

```

3. Одиночка (Singleton):
```c#
public class LogFile
{
    private static LogFile instance;
    private LogFile() { }

    public static LogFile Instance
    {
        get
        {
            if (instance == null)
            {
                instance = new LogFile();
            }
            return instance;
        }
    }

    // Другие члены класса...
}

// Пример использования:
LogFile logFile = LogFile.Instance;
```

# Структурные шаблоны:
1. Декоратор (Decorator):
```c#
public interface Log
{
    void LogData();
}

public class BasicLog : Log
{
    public void LogData()
    {
        Console.WriteLine("Logging basic data...");
    }
}

public abstract class LogDecorator : Log
{
    protected Log log;

    public LogDecorator(Log log)
    {
        this.log = log;
    }

    public virtual void LogData()
    {
        log.LogData();
    }
}

public class TimestampedLogDecorator : LogDecorator
{
    public TimestampedLogDecorator(Log log) : base(log) { }

    public override void LogData()
    {
        base.LogData();
        Console.WriteLine("Adding timestamp...");
    }
}

// Пример использования:
Log basicLog = new BasicLog();
Log timestampedLog = new TimestampedLogDecorator(basicLog);
timestampedLog.LogData();
```

2. Адаптер (Adapter):
```c#
public interface LogAdapter
{
    void LogData();
}

public class ThirdPartyLog
{
    public void StoreLog()
    {
        Console.WriteLine("Storing log data using third-party library...");
    }
}

public class ThirdPartyLogAdapter : LogAdapter
{
    private ThirdPartyLog thirdPartyLog;

    public ThirdPartyLogAdapter(ThirdPartyLog thirdPartyLog)
    {
        this.thirdPartyLog = thirdPartyLog;
    }

    public void LogData()
    {
        thirdPartyLog.StoreLog();
    }
}

// Пример использования:
ThirdPartyLog thirdPartyLog = new ThirdPartyLog();
LogAdapter adapter = new ThirdPartyLogAdapter(thirdPartyLog);
adapter.LogData();
```

3. Компоновщик (Composite):
```c#
public interface LogComponent
{
    void LogData();
}

public class CompositeLog : LogComponent
{
    private List<LogComponent> logComponents = new List<LogComponent>();

    public void Add(LogComponent logComponent)
    {
        logComponents.Add(logComponent);
    }

    public void LogData()
    {
        foreach (var logComponent in logComponents)
        {
            logComponent.LogData();
        }
    }
}

// Пример использования:
CompositeLog compositeLog = new CompositeLog();
compositeLog.Add(new BasicLog());
compositeLog.Add(new TimestampedLogDecorator(new BasicLog()));
compositeLog.LogData();
```

4. Прокси (Proxy):
```c#
public interface Log
{
    void LogData();
}

public class RemoteLog : Log
{
    public void LogData()
    {
        Console.WriteLine("Logging data remotely...");
    }
}

public class LogProxy : Log
{
    private RemoteLog remoteLog;

    public void LogData()
    {
        if (remoteLog == null)
        {
            remoteLog = new RemoteLog();
        }
        remoteLog.LogData();
    }
}

// Пример использования:
Log log = new LogProxy();
log.LogData();
```

# Поведенческие шаблоны:
1. Наблюдатель (Observer):
```c#
using System;
using System.Collections.Generic;

public interface IObserver
{
    void Update(string data);
}

public class ConcreteObserver : IObserver
{
    private string name;

    public ConcreteObserver(string name)
    {
        this.name = name;
    }

    public void Update(string data)
    {
        Console.WriteLine($"{name} received data: {data}");
    }
}

public interface ISubject
{
    void Attach(IObserver observer);
    void Detach(IObserver observer);
    void Notify(string data);
}

public class ConcreteSubject : ISubject
{
    private List<IObserver> observers = new List<IObserver>();

    public void Attach(IObserver observer)
    {
        observers.Add(observer);
    }

    public void Detach(IObserver observer)
    {
        observers.Remove(observer);
    }

    public void Notify(string data)
    {
        foreach (var observer in observers)
        {
            observer.Update(data);
        }
    }
}

// Пример использования:
ConcreteSubject subject = new ConcreteSubject();
IObserver observer1 = new ConcreteObserver("Observer 1");
IObserver observer2 = new ConcreteObserver("Observer 2");

subject.Attach(observer1);
subject.Attach(observer2);

subject.Notify("New data available!");

```

2. Состояние (State):
```c#
using System;

public interface IState
{
    void Handle(Context context);
}

public class ConcreteStateA : IState
{
    public void Handle(Context context)
    {
        Console.WriteLine("Handling in State A");
        context.State = new ConcreteStateB();
    }
}

public class ConcreteStateB : IState
{
    public void Handle(Context context)
    {
        Console.WriteLine("Handling in State B");
        context.State = new ConcreteStateA();
    }
}

public class Context
{
    public IState State { get; set; }

    public Context(IState state)
    {
        State = state;
    }

    public void Request()
    {
        State.Handle(this);
    }
}

// Пример использования:
Context context = new Context(new ConcreteStateA());
context.Request();
context.Request();

```

3. Итератор (Iterator):
```c#
using System;
using System.Collections;

public interface IIterator
{
    bool HasNext();
    object Next();
}

public class ConcreteIterator : IIterator
{
    private ArrayList items;
    private int position = 0;

    public ConcreteIterator(ArrayList items)
    {
        this.items = items;
    }

    public bool HasNext()
    {
        return position < items.Count;
    }

    public object Next()
    {
        if (!HasNext())
        {
            throw new InvalidOperationException();
        }
        object item = items[position];
        position++;
        return item;
    }
}

public interface IAggregate
{
    IIterator CreateIterator();
}

public class ConcreteAggregate : IAggregate
{
    private ArrayList items = new ArrayList();

    public void AddItem(object item)
    {
        items.Add(item);
    }

    public IIterator CreateIterator()
    {
        return new ConcreteIterator(items);
    }
}

// Пример использования:
ConcreteAggregate aggregate = new ConcreteAggregate();
aggregate.AddItem("Item 1");
aggregate.AddItem("Item 2");
aggregate.AddItem("Item 3");

IIterator iterator = aggregate.CreateIterator();
while (iterator.HasNext())
{
    Console.WriteLine(iterator.Next());
}

```

4. Стратегия (Strategy):
```c#
using System;

public interface IStrategy
{
    void Execute();
}

public class ConcreteStrategyA : IStrategy
{
    public void Execute()
    {
        Console.WriteLine("Executing strategy A");
    }
}

public class ConcreteStrategyB : IStrategy
{
    public void Execute()
    {
        Console.WriteLine("Executing strategy B");
    }
}

public class Context
{
    private IStrategy strategy;

    public Context(IStrategy strategy)
    {
        this.strategy = strategy;
    }

    public void SetStrategy(IStrategy strategy)
    {
        this.strategy = strategy;
    }

    public void ExecuteStrategy()
    {
        strategy.Execute();
    }
}

// Пример использования:
Context context = new Context(new ConcreteStrategyA());
context.ExecuteStrategy();

context.SetStrategy(new ConcreteStrategyB());
context.ExecuteStrategy();

```

5. Команда
```c#
using System;
using System.Collections.Generic;

// Интерфейс команды
public interface ICommand
{
    void Execute();
}

// Конкретная команда - запуск каротажа
public class StartLoggingCommand : ICommand
{
    private readonly LoggingSystem loggingSystem;

    public StartLoggingCommand(LoggingSystem loggingSystem)
    {
        this.loggingSystem = loggingSystem;
    }

    public void Execute()
    {
        loggingSystem.StartLogging();
    }
}

// Конкретная команда - остановка каротажа
public class StopLoggingCommand : ICommand
{
    private readonly LoggingSystem loggingSystem;

    public StopLoggingCommand(LoggingSystem loggingSystem)
    {
        this.loggingSystem = loggingSystem;
    }

    public void Execute()
    {
        loggingSystem.StopLogging();
    }
}

// Получатель команды
public class LoggingSystem
{
    public void StartLogging()
    {
        Console.WriteLine("Каротаж начат");
    }

    public void StopLogging()
    {
        Console.WriteLine("Каротаж остановлен");
    }
}

// Исполнитель команды
public class LoggingInvoker
{
    private readonly List<ICommand> commands = new List<ICommand>();

    public void AddCommand(ICommand command)
    {
        commands.Add(command);
    }

    public void ExecuteCommands()
    {
        foreach (var command in commands)
        {
            command.Execute();
        }
    }
}

class Program
{
    static void Main(string[] args)
    {
        // Создание получателя команды
        LoggingSystem loggingSystem = new LoggingSystem();

        // Создание конкретных команд
        ICommand startLogging = new StartLoggingCommand(loggingSystem);
        ICommand stopLogging = new StopLoggingCommand(loggingSystem);

        // Создание исполнителя команд
        LoggingInvoker invoker = new LoggingInvoker();

        // Добавление команд в исполнителя
        invoker.AddCommand(startLogging);
        invoker.AddCommand(stopLogging);

        // Выполнение всех команд
        invoker.ExecuteCommands();
    }
}

```
