# Архитектурные принципы и шаблоны проектирования

Этот документ содержит детальное описание и примеры ключевых принципов программирования и проектирования архитектуры.

***

## 1. Базовые принципы (SOLID, DRY, KISS, YAGNI)

### SOLID

Акроним пяти принципов объектно-ориентированного программирования и дизайна.

#### **S - Single Responsibility Principle (Принцип единственной ответственности)**

*Класс должен иметь только одну причину для изменения.*

- **Суть:** Каждый объект/модуль должен отвечать за одну конкретную функциональность.
- **Пример:**
  ```typescript
  // Плохо: класс делает всё (логику и сохранение)
  class User {
    constructor(public name: string) {}
    saveToDatabase() { /* ... */ }
  }

  // Хорошо: разделение ответственности
  class User {
    constructor(public name: string) {}
  }
  class UserRepository {
    save(user: User) { /* ... */ }
  }
  ```

#### **O - Open/Closed Principle (Принцип открытости/закрытости)**

*Программные сущности должны быть открыты для расширения, но закрыты для модификации.*

- **Суть:** Мы должны добавлять новый функционал, не меняя старый код.
- **Пример:**
  ```typescript
  // Вместо switch/if по типам, используем полиморфизм
  interface Shape {
    getArea(): number;
  }
  class Rectangle implements Shape {
    constructor(public w: number, public h: number) {}
    getArea() { return this.w * this.h; }
  }
  class Circle implements Shape {
    constructor(public r: number) {}
    getArea() { return Math.PI * this.r ** 2; }
  }
  ```

#### **L - Liskov Substitution Principle (Принцип подстановки Барбары Лисков)**
*Объекты в программе должны быть заменяемыми на экземпляры их подтипов без изменения правильности работы программы.*
- **Суть:** Наследник не должен ломать логику родителя. Если функция работает с базовым классом, она должна так же успешно работать и с его наследником.
- **Пример:**
  ```typescript
  // Плохо: Нарушение LSP через классическую проблему Квадрата и Прямоугольника
  class Rectangle {
    constructor(protected width: number, protected height: number) {}
    setWidth(w: number) { this.width = w; }
    setHeight(h: number) { this.height = h; }
    getArea() { return this.width * this.height; }
  }

  class Square extends Rectangle {
    // Квадрат вынужден менять обе стороны, чтобы оставаться квадратом
    setWidth(w: number) {
      this.width = w;
      this.height = w;
    }
    setHeight(h: number) {
      this.width = h;
      this.height = h;
    }
  }

  // Функция, которая ожидает Прямоугольник, сломается на Квадрате
  function resize(rect: Rectangle) {
    rect.setWidth(10);
    rect.setHeight(5);
    // Для прямоугольника площадь должна быть 50, но для квадрата будет 25
    console.log(rect.getArea()); 
  }

  // Хорошо: Используем общую абстракцию без ложных предположений о поведении сторон
  interface Shape {
    getArea(): number;
  }

  class Rectangle implements Shape {
    constructor(private width: number, private height: number) {}
    getArea() { return this.width * this.height; }
  }

  class Square implements Shape {
    constructor(private size: number) {}
    getArea() { return this.size * this.size; }
  }
  ```

#### **I - Interface Segregation Principle (Принцип разделения интерфейса)**

*Много специализированных интерфейсов лучше, чем один универсальный.*

- **Суть:** Клиенты не должны зависеть от методов, которые они не используют.
- **Пример:**
  ```typescript
  // Плохо: интерфейс заставляет реализовывать лишние методы
  interface Bird {
    fly(): void;
    swim(): void;
  }

  class Penguin implements Bird {
    fly() { throw new Error("Penguins can't fly"); }
    swim() { console.log("Swimming"); }
  }

  // Хорошо: разделяем интерфейсы
  interface Flyable {
    fly(): void;
  }

  interface Swimmable {
    swim(): void;
  }

  class Parrot implements Flyable {
    fly() { console.log("Flying"); }
  }

  class Penguin implements Swimmable {
    swim() { console.log("Swimming"); }
  }
  ```

#### **D - Dependency Inversion Principle (Принцип инверсии зависимостей)**
*Зависимости должны строиться на абстракциях, а не на конкретике.*
- **Суть:** Модули верхнего уровня не должны зависеть от модулей нижнего уровня. Оба должны зависеть от абстракций.
- **Пример:**
  ```typescript
  // Плохо: Высокоуровневый класс жестко зависит от реализации (Низкоуровневого класса)
  class FetchData {
    get() { /* получение данных по сети */ }
  }

  class MyPage {
    private service = new FetchData(); // Жесткая привязка к реализации
    show() { this.service.get(); }
  }

  // Хорошо: Оба зависят от абстракции (Интерфейса)
  interface DataReader {
    get(): void;
  }

  class FetchData implements DataReader {
    get() { console.log("Data from API"); }
  }

  class LocalData implements DataReader {
    get() { console.log("Data from LocalStorage"); }
  }

  class MyPage {
    // MyPage не знает, КАК именно получаются данные
    constructor(private service: DataReader) {} 
    show() { this.service.get(); }
  }

  // Теперь мы можем легко подменить источник данных
  const page = new MyPage(new FetchData());
  const offlinePage = new MyPage(new LocalData());
  ```

***

### DRY (Don't Repeat Yourself)

*Не повторяйся.*

- **Суть:** Каждое знание должно иметь единственное, непротиворечивое и авторитетное представление в системе.
- **Пример:** Вынос общей логики в хелперы или хуки вместо копипаста.

### KISS (Keep It Simple, Stupid)

*Делай проще, тупица.*

- **Суть:** Система работает лучше всего, если она остается простой. Избегайте ненужной сложности.
- **Пример:** Использование понятных циклов вместо сложных однострочников на регулярках, если это не дает профита.

### YAGNI (You Ain't Gonna Need It)

*Тебе это не понадобится.*

- **Суть:** Не добавляйте функциональность, пока она действительно не нужна.
- **Пример:** Не пишите "универсальный движок плагинов", если вам нужно просто добавить одну кнопку.

***

## 2. Шаблоны проектирования (GoF)

Паттерны делятся на три основные группы: порождающие, структурные и поведенческие.

### Порождающие (Creational)
*Отвечают за механизмы создания объектов.*

- **Singleton (Одиночка):** Гарантирует, что у класса есть только один экземпляр.
  ```typescript
  class Database {
    private static instance: Database;
    private constructor() {} // Закрываем конструктор
    public static getInstance(): Database {
      if (!Database.instance) Database.instance = new Database();
      return Database.instance;
    }
  }
  ```
- **Factory Method (Фабричный метод):** Позволяет подклассам решать, какой класс инстанцировать.
  ```typescript
  abstract class Dialog {
    abstract createButton(): Button;
    render() { const btn = this.createButton(); btn.onClick(); }
  }
  class WindowsDialog extends Dialog {
    createButton() { return new WindowsButton(); }
  }
  ```
- **Abstract Factory (Абстрактная фабрика):** Создает семейства связанных объектов.
  ```typescript
  interface GUIFactory { createButton(): Button; createCheckbox(): Checkbox; }
  class WinFactory implements GUIFactory {
    createButton() { return new WinButton(); }
    createCheckbox() { return new WinCheckbox(); }
  }
  ```
- **Builder (Строитель):** Позволяет создавать сложные объекты пошагово.
  ```typescript
  class QueryBuilder {
    private query: string = "";
    select(fields: string) { this.query += `SELECT ${fields} `; return this; }
    from(table: string) { this.query += `FROM ${table} `; return this; }
    build() { return this.query; }
  }
  const sql = new QueryBuilder().select("*").from("users").build();
  ```

### Структурные (Structural)
*Отвечают за построение удобных иерархий классов.*

- **Adapter (Адаптер):** Позволяет несовместимым интерфейсам работать вместе.
  ```typescript
  // Наш интерфейс ожидает JSON, а сервис отдает XML
  class XMLToJSONAdapter extends MyService {
    private legacyService: LegacyService;
    getData() { 
      const xml = this.legacyService.getXML();
      return convertXMLToJSON(xml);
    }
  }
  ```
- **Decorator (Декоратор):** Динамически добавляет новую функциональность.
  ```typescript
  class NotifierDecorator implements Notifier {
    constructor(protected wrapper: Notifier) {}
    send(msg: string) { this.wrapper.send(msg); }
  }
  class SMSDecorator extends NotifierDecorator {
    send(msg: string) { super.send(msg); this.sendSMS(msg); }
  }
  ```
- **Facade (Фасад):** Простой интерфейс к сложной системе.
  ```typescript
  class VideoConverter {
    convert(file: string) {
      const source = new VideoFile(file);
      const codec = new CodecFactory().extract(source);
      const buffer = BitrateReader.read(file, codec);
      return new AudioMixer().fix(buffer);
    }
  }
  ```
- **Proxy (Заместитель):** Контролирует доступ к объекту.
  ```typescript
  class CachedYouTubeProxy implements YouTubeLib {
    private service: YouTubeService;
    private cache: any;
    getVideo(id: string) {
      if (!this.cache) this.cache = this.service.getVideo(id);
      return this.cache;
    }
  }
  ```

### Поведенческие (Behavioral)
*Отвечают за эффективную коммуникацию между объектами.*

- **Observer (Наблюдатель):** Механизм подписки на события.
  ```typescript
  class Store {
    private observers: Observer[] = [];
    subscribe(o: Observer) { this.observers.push(o); }
    notify(data: any) { this.observers.forEach(o => o.update(data)); }
  }
  ```
- **Strategy (Стратегия):** Смена алгоритмов внутри объекта.
  ```typescript
  class Navigator {
    private strategy: RouteStrategy;
    setStrategy(s: RouteStrategy) { this.strategy = s; }
    buildRoute(A, B) { return this.strategy.build(A, B); }
  }
  ```
- **Command (Команда):** Превращает запросы в объекты.
  ```typescript
  class Invoker {
    private history: Command[] = [];
    execute(cmd: Command) {
      cmd.do();
      this.history.push(cmd);
    }
    undo() { const cmd = this.history.pop(); cmd.undo(); }
  }
  ```
- **State (Состояние):** Смена поведения при изменении состояния.
  ```typescript
  class AudioPlayer {
    private state: State;
    changeState(s: State) { this.state = s; }
    clickPlay() { this.state.clickPlay(); }
  }
  // State будет либо PlayingState, либо PausedState
  ```

***

## 3. Принципы GRASP

GRASP (General Responsibility Assignment Software Patterns) — шаблоны распределения обязанностей.

- **Information Expert (Информационный эксперт):** Обязанность должна быть назначена тому, кто владеет максимумом информации для её выполнения.
- **Creator (Создатель):** Определяет, кто должен создавать новый экземпляр класса. Обычно тот, кто использует или агрегирует этот объект.
- **Controller (Контроллер):** Первый объект за пределами UI, который получает и обрабатывает системное событие.
- **Low Coupling (Слабая связанность):** Проектируйте так, чтобы зависимости между классами были минимальными. Это упрощает поддержку.
- **High Cohesion (Высокое зацепление):** Обязанности класса должны быть максимально сфокусированы на одной задаче.
- **Polymorphism (Полиморфизм):** Используйте полиморфные операции вместо проверки типов (if/switch).
- **Pure Fabrication (Чистая выдумка):** Создание искусственного класса, не отражающего предметную область, для обеспечения Low Coupling и High Cohesion (например, Service классы).
- **Indirection (Посредник):** Слабая связанность между объектами через введение промежуточного объекта.
- **Protected Variations (Защищенные изменения):** Защита системы от изменений в её частях путем оборачивания их в интерфейс.

***

## 4. Антипаттерны

Ошибочные подходы, которые часто встречаются на практике.

- **Spaghetti Code (Спагетти-код):** Программа с запутанной структурой, где потоки управления сложно отследить.
- **God Object (Божественный объект):** Класс, который делает слишком много и знает о слишком многих частях системы. Нарушает SRP.
- **Golden Hammer (Золотой молоток):** Использование одной и той же технологии/подхода для всех задач, даже если она не подходит.
- **Cargo Cult Programming (Программирование культа карго):** Копирование кода или подходов без понимания того, как и почему они работают.
- **Premature Optimization (Преждевременная оптимизация):** Попытка оптимизировать код до того, как выявлены реальные узкие места.
- **Hard Coding (Хардкодинг):** Вшивание данных прямо в код вместо использования конфигураций или переменных окружения.
- **Copy-Paste Programming:** Решение задач через массовое копирование кода, что приводит к нарушению DRY.

