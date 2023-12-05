## Лабораторная работа 3.

# Диаграмма компонентов

# Диаграмма последовательностей

# Модель БД

# Применение основных принципов разработки
- DRY
  В данном случае выделена фабричная статичная функция, чтобы на основе любого HugeDoubleArray можно было создать HugeDoubleArrayReader.
  ```c#
  public static HugeDoubleArrayReader Create(HugeDoubleArray hugeArray)
        {
            Contract.Requires(hugeArray != null);

            using var lockObject = new ReaderWriterLockSlim(LockRecursionPolicy.SupportsRecursion);
            lockObject.EnterWriteLock();

            try
            {
                var isOpen = hugeArray.OpenRead();

                return isOpen
                    ? new HugeDoubleArrayReader(hugeArray)
                    : null;
            }
            finally
            {
                lockObject.ExitWriteLock();
            }
        }
  ```

  - YAGNI
    В данном случае матрица только 2Д, и не рассматривается ситуация для создания трехмерной матрицы, так как в отрисовке будет только 2Д. We are not gonna need it.
    'public class Matrix<T>
    {
        private readonly T[] matrix;

        /// <summary>
        /// Конструктор с заданным кол-вом строк и столбцов
        /// </summary>
        /// <param name="rowCount">Кол-во строк</param>
        /// <param name="columnCount">Кол-во столбцов</param>
        public Matrix(int rowCount, int columnCount)
        {
            Contract.Requires(rowCount >= 0);
            Contract.Requires(columnCount >= 0);

            RowCount = rowCount;
            ColumnCount = columnCount;
            this.matrix = new T[rowCount * columnCount];
        }'
    
  - KISS
    В данном случае не создается отдельный объект, который будет отвечать за reverse массива. Также не создается иерархия классов для работы с массивом, а просто создается расширения для массива, которое будет возвращать reversed массив.
    'public static HugeDoubleArray Reverse(this HugeDoubleArray sourceArray)
        {
            Contract.Requires(sourceArray != null);
            Contract.Ensures(Contract.Result<HugeDoubleArray>() != null);

            using var reader = HugeDoubleArrayReader.Create(sourceArray);
            using var writer = new HugeDoubleArrayWriter(sourceArray);

            return ReverseValues(reader, writer);
        }'

  
# SOLID:
- Single Responsibility Principle (Принцип единственной обязанности)
  В следующем кусочке кода, выполняется требования о том, что класс должен иметь лишь одну причину для изменения, так как его придется менять только тогда, когда изменится логика операции Undo для расчета времени по Dt.
  'internal class ComputeTimeByDtUndo : CanalSetChangedUndo
    {
        /// <summary>
        /// Конструктор
        /// </summary>
        /// <param name="process"></param>
        internal ComputeTimeByDtUndo(ComputingTimeByDtProcess process) : base(Resource.ComputeT, GetUndoCanals(process))
        {
            Contract.Requires(process != null);
        }

        private static Canal[] GetUndoCanals(ComputingTimeByDtProcess process)
        {
            return process.GetChangedAssociations().Select(a => a.TimeCurve).OfType<Canal>().ToArray();
        }
    }'

- Open/Closed Principle (Принцип открытости/закрытости)
  Применен принцип: открыт для расширения, закрыт для изменения. Открыто для написания наследников FilteredStructure, и закрыто для изменения и класса ExplorerPropertyModelFactory, и всех наследников FilteredStructure. Нужно написать свой фильтр (FilteredStructure) и передать его в конструктор ExplorerPropertyModelFactory, например, фильтр по типу канала или по его размерности.
  'public class ExplorerPropertyModelFactory : PropertyModelFactoryBase<ExplorerPropertyModel>
    {
        private readonly FilteredStructure structure;

        /// <summary>
        /// Конструктор с фильтрованной структурой
        /// </summary>
        /// <param name="structure">элементы эксплорера</param>
        public ExplorerPropertyModelFactory(FilteredStructure structure)
        {
            Contract.Requires(structure != null);
            this.structure = structure;
        }

        /// <inheritdoc />
        protected override ExplorerPropertyModel DoCreate(object dataInstance, string dataPropertyName)
        {
            var model = new ExplorerPropertyModel(this.structure, dataInstance, dataPropertyName) { CanDelete = CanDelete };

            return model;
        }

        /// <summary>
        /// Выбранный элемент может или нет быть удален пользователем (убран из выделения)
        /// </summary>
        public bool CanDelete { get; set; }
    }'
  пример использования:
  - 'var factory = new ExplorerPropertyModelFactory(new Canal1DFilteredStructure(loggingData));'
  - 'var factory = new ExplorerPropertyModelFactory(new FilteredCanalKindStructure(task.LoggingData, Filter));'

- Liskov Substitution Principle (Принцип подстановки Лисков)
  Должна быть возможность вместо базового типа подставить любой его подтип. Если какому-либу полю типа ProcessingTaskForCanal присвоить объект типа WaveSelectionTask, то при вызове метода DoDispose у этого поля, программа не упадет, так как метод DoDispose реализован и самое главное - нужен.
  'public class ProcessingTaskForCanal<T> : ProcessingTask
        where T : Canal
    {
        /// <summary>
        /// Создает задачу
        /// </summary>
        /// <param name="taskName">Название задачи</param>
        /// <param name="source">Исходный канала</param>
        protected ProcessingTaskForCanal(string taskName, T source)
            : base(taskName, source.GetParentNode<LoggingData>())
        {
            Contract.Requires(source.IsPartLoggingData());

            Source = source;
            Subscribe();

            ProcessingInterval.DataCanal = Source;
            InputData.ItemCollection.Add(Source);
        }
  
        /// <inheritdoc />
        protected override void DoDispose()
        {
            base.DoDispose();

            Unsubscribe();
        }

public class WaveSelectionTask : ProcessingTaskForCanal<Canal3D>
    {
        /// <summary>
        /// Создает задачу с заданным именем
        /// </summary>
        /// <param name="taskName">Имя задачи</param>
        /// <param name="waveform">Исходный волновой сигнал</param>
        public WaveSelectionTask(string taskName, Canal3D waveform)
            : base(taskName, waveform)
        {
            Contract.Requires(!string.IsNullOrEmpty(taskName));
            Contract.Requires(waveform != null);

            Parameters = new WaveSelectionTaskParameters(waveform);
            Parameters.Disposed += OnParametersDisposed;

            Subscribe();
        }
        
        protected override void DoDispose()
        {
            base.DoDispose();
            Unsubscribe();
            Parameters.Disposed -= OnParametersDisposed;
            Parameters.Dispose();
        }
    }
  '

- Interface Segregation Principle (Принцип разделения интерфейсов)
  Клиенты не должны вынужденно зависеть от методов, которыми не пользуются. Классу канала не нужно реализовывать или определять методы для обработки многомерных каналов, поэтому часть методов интерфейса ICanal была перенесена в IMultiCanal, чтобы одномерным каналам не нужно было реализовывать бессмысленную функциональность или оставлять методы пустыми.
  'public abstract class Canal : DisposableBase, ICanal { ... }'

- Dependency Inversion Principle (Принцип инверсии зависимостей)
  Модули верхнего уровня не должны зависеть от модулей нижнего уровня. И те и другие должны зависеть от абстракций. В это примере класс PlanshetCanal не зависит от реализации свойства Canal.
  'public class PlanshetCanal : PlanshetCanalBase
    {
        /// <summary>
        /// Канал данных.
        /// </summary>
        ICanal Canal { get; }
  }'

# Дополнительные принципы разработки

- BDUF (Big Design Up Front - «Масштабное проектирование прежде всего»):
--Применимость: В контексте разрабатываемого приложения, BDUF может быть частично применим, чтобы определить общую архитектуру системы, особенности интеграции и основные компоненты. Однако, из-за того, что очень часто появляются новые подходы, то сделать это будет очень сложно.
--Отказ: Полное предварительное проектирование может быть нереалистичным из-за нестабильности и изменчивости подходов к анализу данных скважины. Предварительное проектирование может быть ориентировано на общую структуру, но гибкость и адаптивность также должны быть учтены.

-SoC (Separation of Concerns - принцип разделения ответственности):
--Применимость: Принцип SoC играет важную роль в разработке ГИС (так называются приложения для обработки и анализа геофизических данных). Разделение данных, обработки, визуализации, и обмена данными (например, данные о скважинах, их обработка и визуализация) позволяет создать более гибкую, понятную и легко модифицируемую архитектуру.
--Обоснование: Разделение обязанностей позволяет четко определить ответственность каждой части системы, что упрощает поддержку, тестирование и модификацию.

-MVP (Minimum Viable Product - минимально жизнеспособный продукт):
--Применимость: MVP особенно полезен в данном случае, поскольку он позволяет быстро протестировать базовые функции. Например, начать с минимальных возможностей визуализации данных о скважинах для проверки их правильности.
--Обоснование: Создание минимального жизнеспособного продукта позволяет быстрее получить обратную связь от пользователей и выявить основные требования.

-PoC (Proof of Concept - доказательство концепции):
--Применимость: PoC может быть полезным для тестирования новых алгоритмов обработки геофизических данных или разработки новых методов визуализации, прежде чем они будут полностью интегрированы в систему.
--Обоснование: Позволяет провести эксперименты, проверить новые идеи и технологии, минимизируя затраты времени и ресурсов на ранней стадии разработки.
