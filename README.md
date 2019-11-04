# JavaBooks-Notes

# Chapter 2 - Creating and Destroying Objects

### Item 1: Consider static factory methods instead of constructors

Сначала разберем 3 паттерна:
1. Factory
2. Factory method
3. Static factory method

#### 1. Factory
```java
class FruitFactory {

  public Apple makeApple() {
    return new Apple();
  }

  public Orange makeOrange() {
    return new Orange();
  }

}
```

**Factory** - это единичный класс, без субклассов. Он используется тогда, когда создавать данные объекты в конструкторе сложно.

#### 2. Factory method
```java
abstract class FruitPicker {

  protected abstract Fruit makeFruit();

  public void pickFruit() {
    private final Fruit f = makeFruit();
    // fruit work on here... 
  }
}

class OrangePicker extends FruitPicker {

  @Override
  protected Fruit makeFruit() {
    return new Orange();
  }
}
``` 

**Factory method** - это интерфейс, который могут реализовывать классы. Каждый класс будет создавать свой инстанс. Используется, когда надо делегировать операцию по созданию объекта другому классу, потому что неизвестно, как создавать данный инстанс.

#### 3. Abstract factory

```java
interface PlantFactory {

  Plant makePlant();

  Picker makePicker(); 

}

public class AppleFactory implements PlantFactory {
  Plant makePlant() {
    return new Apple();
  }

  Picker makePicker() {
    return new ApplePicker();
  }
}

public class OrangeFactory implements PlantFactory {
  Plant makePlant() {
    return new Orange();
  }

  Picker makePicker() {
    return new OrangePicker();
  }
}
```

**Abstract Factory** - это фабрика, которая производит *семейства* объектов.
Благодаря abstract factory мы можем не ошибиться в выборе нужных объектов, пока работаем с одной и той же фабрикой, она произоводит объекты, которые схожи по признаку с этой фабрикой.

#### 4. Static factory method

Static factory method - это не то же самое, что factory method. Это какой-то один метод в утилити классе или даже в самом классе.
Вот пример Boolean:

```java
class Boolean {
    public static Boolean valueOf(boolean b) {
        return b ? Boolean.TRUE : Boolean.FALSE;
    }
}
```
Преимущества static factory method:
1. Они имеют имена, говорящие больше, чем конструктор
2. В отличие от конструкторов, они не обязаны возвращать каждый раз новый объект (тот же пример с Boolean). Еще - ```Collections.emptyList()```
3. Они могут возвразать любой субкласс в зависимости от входных параметров, а не конкретно тот, который указан конструктором.

### Item 21:  Design interfaces for posterity

Нужно продумывать дизайн интерфейса особенно тщательно с учетом того, что дальше может понадобиться добавить новый метод.
Когда добавляешь новый метод с default реализацией, следуем подумать, не нарушит ли это инварианты классов уже реализовавших интерфейс.

---

### Item 22:  Use interfaces only to define types

Интерфейсы следует использовать только для определения типов, на основе которых методы инстансов классов смогут быть использованы. 
Constant interface не следует делать
```java
 // Constant interface antipattern - do not use!
public interface PhysicalConstants {
    // Avogadro's number (1/mol)
    // static final double AVOGADROS_NUMBER   = 6.022_140_857e23;
   
    // Boltzmann constant (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    
    // Mass of the electron (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```

Такой интерфейс состоит полностью из констант и это плохой дизайн. То, что класс использует какие-то константы - это деталь реализации, но из-за того, что класс implements такой интерфейс, эта деталь реализации утекает в предоставляемое классом API.
Хотя это не несет вреда для пользователя, но нарушает принцип инкапсуляции и все-таки выглядит странным для пользователя.

Существует несколько путей предоставления констант:
 1. Добавить их в сам класс или интерфейс
 2. Объявить их как enum
 3. Создать noninstantiable utility class:
 ```java
public class PhysicalConstants {
    private PhysicalConstants() { }  // Prevents instantiation
    
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;
    public static final double ELECTRON_MASS    = 9.109_383_56e-31;
}
```

Не указывание класса для нахождения констант (также, как и использование констант с помощью constant интерфейса) можно осуществить с помощью **import static**

---

### Item 23:  Prefer class hierarchies to tagged classes
Следует использование иерархию классов, если экземпляры класса могут быть нескольких видов. Например, Figure. Может быть rectangle, circle.
В tagged классах вся логика всех типов находится в одном общем классе - Figure. В методах которого куча switch и if. На основе типа (для него, кстати, придется хранить отдельное поле, чтобы узнать какого типа настоящая фигура) у методов класса разная логика. Это очень плохо, так как плохо масштабируется на множество типов. Более того, им приходится хранить лишние поля, необходимые для какого-то конкретного типа.

Выпишу здесь отрывок: `Fields can’t be made final unless constructors initialize irrelevant fields, resulting in more boilerplate. Constructors must set the tag field and initialize the right data fields. If you do add aflavor,  you  must  remember  to  add  a  case  to  every  switch  statement,  or  the  classwill  fail  at  runtime. Finally,  the  data  type  of  an  instance  gives  no  clue  as  to  its flavor`.

Просто следует сделать абстрактный класс, который будет на вершине иерархии и объявить в нем методы, общие для всех классов которые является того же типа, что и абстрактный класс. Пример - ASTNode, Figure.

---

### Item 24:  Favor static member classes over nonstatic
Существует 4 типа вложенных классов. 
Nested (вложенные) классы - классы, которые служат только для целей того класса, в котором они создаются. Если такой класс требуется для еще классов кроме настоящего, то он должен быть top level класс.

1. static  member  classes
2. nonstatic  member  classes
3. anonymous  classes
4. local classes

Все, кроме первого класса известны как **inner classes** (внутренние классы).
Static member classes ведут себя также, как и обычные static переменные.

**Static и nonstatic member classes:**

* **Non-Static:**

`Each  instance  of  a  nonstatic  member  class  is  implicitly  associated  with  an enclosing instance of its containing class. Within instance methods of a nonstatic member class, you can invoke methods on the enclosing instance or obtain a reference to the enclosing instance using the qualified this construct`

* **Static:**

If you declare a member class that does not require access to an enclosing instance, always put the static modifier in its declaration
Если не следовать этому правилу, то каждый не static вложенный класс будет иметь ссылку на enclosing класс, а это занимает время и место при конструкции класса. 

Пример static класса - Builder или public helper класс, который будет хранить какие-то enum, который будет использован только в сочетании с самим классом. Типа Calculator.Operation.PLUS и Calculator.Operation.MINUS. Правда ведь, что делать top-level enum CalculatorOperation или Operation, что-то не то.


**Anonymous classes:**
Анонимные классы имеют ссылку на enclosing класс, когда они объявляются в non-static методе. Иначе они не имеют ссылки на enclosing класс. Anonymous классы использовались раннее для созданиия function objects (так как функции в джаве не являются first class function).

Anonymous классы should be kept short.

Anonymous классы не имеют типа и могут только реализовывать какой-то интерфейс или расширять какой-то класс.

**Local classes:**
Самый наименее часто используемый тип вложенного класса.
Это класс, который объявлен прямо в каком-либо методе.

Local class имеет общие свойства с member классами и anonymous классами: `Like member classes, they have names and can beused repeatedly. Like anonymous classes, they have enclosing instances only if theyare defined in a nonstatic context, and they cannot contain static members. And likeanonymous classes, they should be kept short so as not to harm readability.`

Также local классы имеют ссылку на enclosing класс и могут использовать переменные и методы enclosing класса. В дополнение к этому они имеют доступ к любым final переменным, объявленным в методе.

**Вывод:**

To  recap,  there  are  four  different  kinds  of  nested  classes,  and  each  has  its place. If a nested class needs to be visible outside of a single method or is too long to fit comfortably inside a method, use a member class. If each instance of a mem-ber class needs a reference to its enclosing instance, make it nonstatic; otherwise,make it static. Assuming the class belongs inside a method, if you need to create instances from only one location and there is a preexisting type that characterizes the class, make it an anonymous class; otherwise, make it a local class.

---

### Что такое first class function?
Говорится, что язык программирования имеет first class functions, если функции в нем являются first class objects.

First class objects - это элементы, которые могут быть переданы как параметр, возвращены из функции, присвоены переменной, созданы во время выполнения.

---

### Item 25:  Limit source files to a single top-level class
Для каждого класса следует использовать отдельный файл.

`While the Java compiler lets you define multiple top-level classes in a single sourcefile, there are no benefits associated with doing so, and there are significant risks`.

Риск заключается в том, что если объявить несколько классов в одном файле, и потом объявить еще 2 таких же класса в другом файле, то какая будет использоваться реализация, будет зависеть от того, в каком порядке компилировались классы, что абсолютно недопустимо. То есть, это не круто, что если передать файлы для компиляции в каком-то другом порядке, то программа будет работать вообще по-другому.

# Chapter 5 - Generics

Дженерики были созданы для того, чтобы ввести проверку типов в compile-time, а также добавить дополнительный уровень абстракции над типами - также как и темплейты в C++. Дженерики позволяют оперировать различными типами в классе, интерфейсе и их соотвествующих методах и полях, а не быть привязаным к какому-то одному типу. Естественно, для всех типов должна быть правильная работа, поэтому часто классы пишутся направленно на какое-то множество типов, которые extend какой-то суперкласс или implements какой-то интерфейс. Например, sort или compare.

**Wildcards**
Wildcard - ? - означает неизвестный тип. Хотя wildcards очень похожи на type parameters(T, E и т.д.), между ними есть различия.

1. Wildcard не позволяет указать, что оба аргумента являются одним и тем же типом.
Например:
```java
public static <T extends Number> void copy(List<T> dest, List<T> src);
```
и

```java
public static void copy(List<? extends Number> dest, List<? extends Number> src);
```

Первый задает какой-то тип T, который расширяет Number. Поэтому туда можно передать только или оба аргумента типа Integer, или оба аргумента типа Float.
Во второй метод же можно передать как Integer, так и Float в разные параметры, так как мы не задали определенный тип.

Ну и остальное скопирую с so:
1. If you have only one parameterized type argument, then you can use wildcard, although type parameter will also work.
2. Type parameters support multiple bounds, wildcards don't. - ```TypeParameter extends Class & Interface & Inteface```
3. Wildcards support both upper and lower bounds, type parameters just support upper bounds.

Самое важное, что стоит понять - это то, что Generics - это только compile-time feature. Весь код после компиляции превращается в код без никаких дженериков.

Процесс этот называется в джаве **Type Erasure**. После компиляции все bounded типы заменяются их bound-ами, а unbounded типы заменятся на Object.

Bounded тип - это тип, который extends какой-то суперкласс. Unbounded - без extends.

Пример:

**Unbounded:**
```java
public <T> List<T> genericMethod(List<T> list) {
    return list.stream().collect(Collectors.toList());
}
```

Заменяется на:
```java
// for illustration
public List<Object> withErasure(List<Object> list) {
    return list.stream().collect(Collectors.toList());
}
 
// что в дейстительно является
public List withErasure(List list) {
    return list.stream().collect(Collectors.toList());
}
```

**Bounded:**
```java
public <T extends Building> void genericMethod(T t) {
    ...
}
```

Заменяется на:
```java
public void genericMethod(Building t) {
    ...
}
```

#### Bound

**Нижняя граница:** *super* - ```<E super SomeClass>``` означает, что E - это класс (интерфейс), который является родителем класса SomeClass. То есть, если SomeClass расширяет ParentClass, мы можем передать как параметр ParentClass.
**Верхняя граница:** *extends* - ```<E extends SomeClass``` означает, что E - это класс (интерфейс), который является ребенком класса SomeClass, то есть E расширяет SomeClass.

При чем, _super_ включает и сам класс SomeClass, но *extends* только классы, которые расширяют его.
Можно описать так:

*super*: ```>=```
*extends*: ```<```

Именно поэтому примитивы не могут использовать как типы коллекций и вообще всего, что работает с помощью дженериков: примитивы не расширяют никакой класс и тем более не расширяют Object, как это делают любые объекты в джаве.

#### Памятка по дженерикам

|                    Term 	| Example                          	        |
|------------------------:	|------------------------------------------ |
| Parameterized type 	    | ```List<String>```                     	|
| Actual type parameter   	| ```String```                           	|
| Generic type 	            | ```List<E>```                          	|
| Formal type parameter   	| ```E```                                	|
| Unbounded wildcard type 	| ```List<?>```                          	|
| Raw type                	| ```List```                             	|
| Bounded type parameter  	| ```<E extends Number>```               	|
| Recursive type bound    	| ```<T extends Comparable<T>>```        	|
| Bounded wildcard type   	| ```List<? extends Number>```           	|
| Generic method          	| ```static <E> List<E> asList(E[] a)``` 	|
| Type token              	| ```String.class```                     	|

### Item 26: Don’t use raw types

Raw type - это использование класса без указания типа, с которым работает данный класс. Объясню на примере:
```List<E>``` - это generic type (это обобщение generic классов и generic интерфейсов). E - тип, с которым работает этот класс.
```List<String>``` - это parameterized type. String указывает, с каким типом работает данный класс (параметризует класс).
```List``` - raw type. Тип не указан. Такой тип используется для работы с pre-generic кодом (когда дженериков ещё не было). Ни в коем случае нельзя использовать такой тип.
Строго говоря, raw type - это generic type без указания accompanying type parameters.

**Почему нельзя использовать raw type?**

Ну, ясно что из-за использования такого типа ты просто лишаешься compile-time проверки. Ты не видишь, чем параметризован generic класс или интерфейс.

Класть в List или Collection объекты разного типа МОЖНО, но ты получишь ошибку ClassCastException, когда попытаешься достать объект и скастить его к одному типу.
Ну и да, когда лучше обнаружить ошибку: в runtime или в compile-time?

**Почему вообще raw types разрешены?**

Когда появились дженерики, это уже была Java 5, и крайне важно было сохранить совместимость с старым кодом. Надо было сделать так, чтобы передача параметризованных типов в методы, которые работают с raw types, была возможна и все работало и дальше, и наоборот. Это привело к решению использованию **type erasure**. На сегодняший день это крайне ужасное решение, и все хотят изменить это, но не знают как :)

Между тем, использовать ```List<Object>``` - можно. Это указывает, что пользователь знает о дженериках и он целенаправленно решил использовать не параметризованный тип.
Более того, нельзя передать List<String> в метод, ожидающий List, но нельзя передать в метод, ожидающий List<Object>, так как List<String> субтип List, но не субтип List<Object> из-за существования определенных правил для дженериков. Из-за этого type safety не теряется.

Если дейстительно требуется передавать коллекции любых типов, то следует использовать List<?> list в параметрах. Например:
```java
// Uses unbounded wildcard type - typesafe and flexible
static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }
```
Главная разница между List<?> и List - это то, что в List<?> нельзя вставить любой объект (если лист был передан как параметр). Это даст compile-time ошибку.

Если требуется и передавать коллекции любых типов, и вставлять в них что-то, то требуется использовать *bounded wildcard types* или же *generic methods*.
Сейчас покажу решение с wildcard:
```java
private static void addElem(List<? super Fruit> list) {
    list.add(new Fruit());
}
```

**Оператор instanceof:** 

instanceof можно использовать только на unbounded wildcard, и нельзя на параметризованные, потому что информация о типах стирается в рантайме. Но, можно использовать и на raw type, это не принесет вреда, а wildcard только мешается. Даже больше скажу: это предпочтительный вариант использования *instanceof*:
```java
// Legitimate use of raw type - instanceof operator
if (o instanceof Set) {       // Raw type
    Set<?> s = (Set<?>) o;    // Wildcard type
}
```

### Item 28: Prefer lists to arrays

Самые важные различия массивов и генерик коллекций:

1. Массивы ковариантны, а дженерики - инварианты (контрвариантны)

Это означает то, что если *Sub* - субтип *Super*, то *Sub[]* - субтип *Super[]*. Напомню, почему я выше сказал, что если мы действительно хотим хранить объекты любого типа в коллекции, то следует использовать List<Object>, но не в List. Повторю текст выше: ```Более того, нельзя передать List<String> в метод, ожидающий List, но нельзя передать в метод, ожидающий List<Object>, так как List<String> субтип List, но не субтип List<Object> из-за существования определенных правил для дженериков. Из-за этого type safety не теряется.```

Более строго (из википедии):

> Ковариантностью называется сохранение иерархии наследования исходных типов в производных типах в том же порядке. Так, если класс Cat наследуется от класса Animal, то естественно полагать, что перечисление IEnumerable\<Cat\> будет потомком перечисления IEnumerable\<Animal\>. Действительно, «список из пяти кошек» — это частный случай «списка из пяти животных». В таком случае говорят, что тип (в данном случае обобщённый интерфейс) IEnumerable<T> ковариантен своему параметру-типу T. 

> Контравариантностью называется обращение иерархии исходных типов на противоположную в производных типах. Так, если класс String наследуется от класса Object, а делегат Action<T> определён как метод, принимающий объект типа T, то Action\<Object\> наследуется от делегата Action\<String\>, а не наоборот. Действительно, если «все строки — объекты», то «всякий метод, оперирующий произвольными объектами, может выполнить операцию над строкой», но не наоборот. В таком случае говорят, что тип (в данном случае обобщённый делегат) Action\<T\> контравариантен своему параметру-типу T. 

Пример:

```java
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in"; // Throws ArrayStoreException
```
Такой код компилируется, но упадет в рантайме.

Такой код сильно безопасный:
```java
// Won't compile!
List<Object> ol = new ArrayList<Long>(); // Incompatible types
ol.add("I don't fit in");
```

2. Тип массива конкретно определен в рантайме (reified)
Дженерики же используют type erasure, поэтому их тип неизвестен в рантайме. Это позволяет им работать с pre-generic кодом.

Поэтому, это невозможно создать массив generic типа, создать массив параметризованного типа или же type parameter (тот же дженерик). 
Кодом, соответственно: ```new List<E>[]```, ```newList<String>[]```, ```new E[]```.
Напомню, generic type - это generic класс или generic интерфейс. Type parameter - параметр, который задает тип.

Нельзя создать такой массив потому, что тип теряется в рантайме, но массив должен быть определен конкретно.
n
Единственный хак, как можно создать дженерик массив:
```java
List<?>[] listArray = new List<?>[64];
```

Это потому, что unbounded wildcard - reified тип. Строго говоря, non-reifiable type is one whose runtime representation contains less information than its compile-time representation. Because of erasure, the only parameterized types that are reifiable are unbounded wildcard types 

Подводя итог:
> In summary, arrays and generics have very different type rules. Arrays are covariant and reified; generics are invariant and erased. As a consequence, arrays provide runtime type safety but not compile-time type safety, and vice versa for generics. As a rule, arrays and generics don’t mix well.

---

# Chapter 6 - Enums and Annotations

Джава имеет два специальных типа ссылок: Enum тип класса и Annotation тип интерфейса. Здесь рассматриваются best practices для этих типов.

### Item 34: Use enums instead of int constants

Почему следует использовать enum вместе int enum паттерна:

1. Int enum паттерн не дает возможности тайп чека. Это означает нарушение инвариантности в некоторых функциях, например если метод ожидает 3 значения интов - 0, 1, 2, то ничего не затруднит передать просто 4, так как типа параметра int. Или например мы можем сравнивать разные семейства констант с помощью == оператора, что точно нельзя делать. То есть, compile time safety.
2. Более того, enum класс - это полноценный класс с полями и методами, и это дает намного больше возможностей. Например, хотя бы мы можем менять вывод с помощью переопределения метода toString().
3. Нет неймспейса специального под константы. Приходится писать через префикс. Например: APPLE_FUJI и ORANGE_TEMPLE.

Еще в енамах можно создавать методы и даже существует способ определния поведения метода в зависимости от енама. Это делается легко - определяется абстрактный метод в классе енама, а инстансы енама реализуют этот метод. 

Вот пример:
```java
// Enum type with constant-specific method implementations
public enum Operation {
    PLUS  {public double apply(double x, double y){return x + y;}},
    MINUS {public double apply(double x, double y){return x - y;}},
    TIMES {public double apply(double x, double y){return x * y;}},
    DIVIDE{public double apply(double x, double y){return x / y;}};
    
    public abstract double apply(double x, double y);
}
```

A  disadvantage  of  constant-specific  method  implementations  is  that  theymake  it  harder  to  share  code  among  enum  constants.
Поэтому можно сделать по другому: определить еще один nested enum, в котором будут как инстансы с тем же методом определены инстансы поведений, а далее эти инстансы поведений передавать в конструктор top level енама, и при необходимости вызова метода делегировать вызов nested енаму:
```java
// The strategy enum pattern
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
    SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }
    PayrollDay() { this(PayType.WEEKDAY); }  // Default

    int pay(int minutesWorked, int payRate) {return payType.pay(minutesWorked, payRate);}

    // The strategy enum type
    private enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {return minsWorked <= MINS_PER_SHIFT ? 0 :(minsWorked - MINS_PER_SHIFT) * payRate / 2;}
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {return minsWorked * payRate / 2;}
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
        
        int pay(int minsWorked, int payRate) {int basePay = minsWorked * payRate;return basePay + overtimePay(minsWorked, payRate);}
    }
}
```

То есть кратко, enum-ы более понятны и читабельнее, безопаснее и мощнее.

---

### Item 35: Use instance fields instead of ordinals

Каждый enum имеет метод *ordinal()*. Он возвращает числовое значение enum-а. И данный итем заключается в том, что если требуется числовое значение завязанное на конкретном енаме, то это число следует передавать в конструктор и хранить его как instance field данного енама. Причины такие:

- Порядок может меняться, и значит ordinal() тоже меняет свое значение, и приложение сломается в рантайме
- Добавлять новые енамы сложно, тем более с значениями большими чем количество енамов (так как ordinal() берет значение по порядку)

---

### Item 36:  Use EnumSet instead of bit fields

Если элементы enum используются в сетах констант (наборах констант), то обычно применяется int enum pattern, присваивая каждой константе различную степень двойки:
```java
public class Text {
    public static final int STYLE_BOLD          = 1 << 0;  // 1
    public static final int STYLE_ITALIC        = 1 << 1;  // 2

    public void applyStyles(int styles) { ... }
}
```
и используется это так:
```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

Этот сет констант называется *bit field*.

Недостатки такого подхода - это все недостатки int enum pattern, но имеются и другие:
- Это даже сложнее интерпретировать результат вывода bit field, чем просто int
- Нет легкого способа пройтись по всем элементам представляемым данным bit полем. Например, как сверху, обойти STYLE_BOLD | STYLE_ITALIC.
- И наконец, однажды выбрав количество битов (int и long - 32 и 64 бита соответственно), будет сложно поменять API, так как типы отличаются и будет ошибка компиляции.

Но в джаве есть лучшее решение - **EnumSet**.

Внутри EnumSet представлен бит вектором, единичным long полем, поэтому он эффективен.
Более того, всю работу с наборот констант класс делает сам, и более того, делает тоже с помощью битовой арифметики, то есть избавляет вас от тяжелой работы где легко ошибиться (error-proneness работа).

С использованием EnumSet код становится намного понятнее и легче:
```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    public void applyStyles(Set<Style> styles) { ... }
}
```

А использовать можно так:
```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

---

### Item 37: Use EnumMap instead of ordinal indexing

Часто необходимо мапить элементы по енамам, но нельзя для этого использовать метод ordinal() для мапинга как индекс элемента массива или листа. Например так:
 ```java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
    
    final String name;
    final LifeCycle lifeCycle;
    
    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override 
    public String toString() {
        return name;
    }
};

Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

for (int i = 0; i < plantsByLifeCycle.length; i++) {
    plantsByLifeCycle[i] = new HashSet<>();
}

for (Plant p : garden) {
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
}
```

по следующим причинам:
- Так как лист или массив не знает, какой индекс представляет какой енам, то маппинг придется делать самим (момент с `plantsByLifeCycle[p.lifeCycle.ordinal()]`)
- Приходится проверять корректность int значения, нет compile time safety енамов

Есть способ лучше сделать такую логику - использовать *Map*, где ключ - сам enum, а значения то что требуется. Однако, существует специальная реализации мапы для енамов, более быстрая и эффективная - **EnumMap**.

Вот как будет выглядеть прежний код с использованием EnumMap:
```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle 
    = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
    plantsByLifeCycle.put(lc, new HashSet<>());
}

for (Plant p : garden) {
    plantsByLifeCycle.get(p.lifeCycle).add(p);
}

System.out.println(plantsByLifeCycle);
```

Этот код короче, чище и понятнее, и есть тайп чек. По быстроте эта мапа сравнима с реализацией через массив выше, так как на самом деле внутренне EnumMap представлен массивом, таким образом мы берем быстроту работы массива и type safety от мапы.

---

### Item 38: Emulate extensible enums with interfaces

Иногда небходимо расширять енамы, но так как *enum* в Джаве - это static final класс, то его нельзя экстендить.

Да и вообще, это не требуется обычно, но если все таки требуется, то делается это так: создается интерфейс, и его имплементируют множество енамов, а в методах принимается и отдается объект типа интерфейса, а не конкретного енама.

Недостаток такого подхода понятен - невозможно наследовать имплементацию от одного енама к другому. Однако, если имплементация не зависит от состояния, то ее можно поместить в default метод интерфейса.

Однако, как мы увидели, эмулировать расширение енамов все таки можно.

---

### Item 39: Prefer annotations to naming patterns

Что-то про использование аннотаций, не буду писать.

---

### Item 40: Consistently use the Override annotation

Следует использовать аннотацию @Override при переопределении метода super класса по следующим причинам:
- данная аннотация позволяет показать явно, что метод переопределен - читабельность и ясность кода
- помощь компилятора: проверяет, правда ли переопределяем такой метод. Например, метод может не существовать, или мы вообще перегружаем вместо переопределения.

Можно не использовать аннотацию при расширении абстрактного класса с абстрактным методом, так как компилятор все равно выдаст ошибку, если мы не переопределим абстрактный метод.

Однако, аннотацию можно и даже нужно использовать на всех методах абстратного класса или интерфейса, если они расширяют другой супер класс или суперинтерфейс соответственно, и мы хотим убедиться, что мы не добавили никаких новых методов кроме тех, что были в супер классе или интерфейсе.

Например, интерфейс *Set* не добавляет никаких новых методов к интерфейсу *Collection*, но расширяет его и поэтому там методы могли бы быть помечены как @Override (там это не делается, но просто как пример).

---

### Item 41: Use marker interfaces to define types

A **marker interface** is an interface that contains no method declarations but merely designates  (or  “marks”) a class that implements the interface as having some property. For example, consider the *Serializable* interface (Chapter 12). By implementing this interface, a class indicates that its instances can be written to an `ObjectOutputStream` (or “serialized”).

Так, зачем же нужен *marker interface*, когда есть *marker annotation*?

**Marker interfaces have two advantages over  marker  annotations.**

1. First  and  foremost,  marker  interfaces  define  a  type that is implemented by instances of the marked class; marker annotations do not. The existence of a marker interface type allows you to catch errors at compile time that you couldn’t catch until runtime if you used a marker annotation. То есть, в каком то методе можно применять объекты, реализующие этот интерфейс, и это проверка в компил тайме.
2. Another advantage of marker interfaces over marker annotations is that they can be targeted more precisely. Мы можем marker interface экстендить от другого интерфейса, и применяя к классам, мы убеждаемся, что помеченные типы будут также подтипами sole interface, который мы расширили.

**The chief advantage of marker annotations over marker interfaces is that they  are  part  of  the  larger  annotation  facility.**  Therefore,  marker  annotations allow for consistency in annotation-based frameworks.

**Как выбрать, что использовать, *marker annotation* или *marker interface*?**

So  when  should  you  use  a  marker  annotation  and  when  should  you  use  a marker interface? Clearly you must use an annotation if the marker applies to any program  element  other  than  a  class  or  interface,  because  only  classes  and  inter-faces can be made to implement or extend an interface. If the marker applies only to classes and interfaces, ask yourself the question “Might I want to write one or more methods that accept only objects that have this marking?” If so, you should use a marker interface in preference to an annotation. This will make it possible for you to use the interface as a parameter type for the methods in question, which will result in the benefit of compile-time type checking. If you can convince your-self  that  you’ll  never  want  to  write  a  method  that  accepts  only  objects  with  the marking, then you’re probably better off using a marker annotation. If, additionally, the marking is part of a framework that makes heavy use of annotations, then a marker annotation is the clear choice.

---

# Chapter 7 - Lambdas and Streams

В джава 8 были добавлены функциональные интерфейсы, лямбды и ссылки на методы (method references), стримы. Вообще, стримы нужны для обработки последовательностей каких-то, более лаконичный вариант цикла.

### Item 42: Prefer lambdas to anonymous classes

Давно для function objects использовались интерфейсы с единственным абстрактным методом, и передавать функцию можно было передавать в метод используя анонимный класс, реализующий этот интерфейс.

Но для функционального программирования, где часто передаются функции, это выглядело ужасно. Поэтому в джаве 8 такие интерфейсы удостоились специального отношения и были названы как functional interfaces, а создавать их стало возможно с помощью *лямбда выражений (lambda expressions)*, или *лямбд (lambdas)* кратко.

Лямбды намного кратче, понятнее, и позволяют сфокусироваться на бизнес логике, чем на бойлерплейте. Тип, возвращаемый лямбдой, не указывается явно часто и выводится компилятором, это называется *type inference*. Это крайне сложный процесс, и занимает целую главу в JLS (Java Language Specification). Но это ок, тип не надо указывать явно, пока компилятор сам не сможет вывести его сам или вы уверены, что код станет понятнее если явно указать тип возвращаемого значения.

Однако, не следует использовать лямбды везде: unlike  methods  and  classes,  lambdas  lack  names  and  documentation;  if  a computation  isn’t  self-explanatory,  or  exceeds  a  few  lines, **don’t put it in a lambda**. Поэтому constant-specific тела все еще хороший way to go.

Также не следует выкидывать анонимные классы из обращения, есть вещи, где они все еще могут потребоваться. Лямбды ограничены функциональными интерфейсами, поэтому с их помощью не получится реализовать абстрактный класс. Также, анонимные классы позволяют реализовывать абстрактные классы с несколькими абстрактными методами. И наконец, лямбда не может получать ссылку на саму себя с помощью `this`, а только на enclosing instance. В анонимном классе же `this` ведет на сам класс, поэтому если требуется использовать function object из его тела, то надо использовать анонимный класс.

---

### Item 43: Prefer method references to lambdas

Итак. лямбды интересны тем что они кратки и не позволяют фокусироваться на бизнес логике. Однако, джава имеет способ генерировать function objects еще более кратким способом: **method references**.

Однако, иногда даже лямбды не привносят много пользы, и приходится писать бойлерплейт код. Пример:
```java
(n1, n2) -> n1 + n2
```

или

```java
s -> System.out.println(s)
```

Да, выглядит красиво и читается легко (по сравнению с анонимными классами), но такой код бойлерплейт, и все эти параметры и действия не привносят много пользы в понимание логики. Вместо этого мы могли бы использовать следующее: ```Integer::sum``` и ```System.out::println```. Method reference позволяет теперь полностью сконцентрироваться на семантике, на логике.

В method reference нет ничего особенного, то есть все что можно сделать с помощью method reference можно сделать и лямбдой. Method reference просто позволяет еще более сократить код и еще более сосредоточиться на бизнес логике.

Итак, где method references более кратки и ясны, следует использовать их, а иначе лямбды.

---

### Item 44: Favor the use of standard functional interfaces

Теперь, когда имеются лямбды, лучшие практики для написания API поменялись. Теперь все больше конструкторы или static factory методы будут принимать function objects как параметры. Раньше для определения конкретного поведения использовался паттерн *шаблонный метод*.

Основной пойнт данного итема - так как functional interfaces, становятся более важны, следует использовать верные, более подходящие интерфейсы.

Например, *java.util.function* содержит множество стандартных функциональных интерфейсов, и для большинства кейсом они подходят. Не стоит строить свои функциональные интерфейсы для задачи, если стандартный справляется с ней и так.

Вот самые важные и нужные из них, остальные можно вспомнить из них:
![Standart functional interfaces](https://i.imgur.com/T2fra1Y.png)

Здесь также существует 3 варианта каждого из этих 6 базовых стандартных функциональных интерфейсов - для int, long и double. Их имена начинаются с префикса Int, Long или Double. Например, `IntPredicate` или `LongBinaryOperator`. Никакой из них не параметризован дженериками (принимают или возвращают примтивные значения), кроме *Function*, так как функция принимает и возвращает значения разных типов. Например, *LongFunction<int[]>* принимает long и возвращает int[].

Однако, и для Function есть 6 вариантов для принимаемого и возвращемого значений примитивных типов. Они имеют префикс *Src*To*Result*. Например, `LongToIntFunction`. А также существует 3 варианта для случаев, когда принимаемое значение примитивного типа, а возвращаемого - ссылка (объект). Такие стандартные функциональные интерфейсы имеют префикс *\<Src\>*To*Obj*. Например, `DoubleToObjFunction`.

Есть еще варианты с *Bi* для Predicate, Function и Consumer - то есть принимают два значения, а в остальном такие же по поведению. Конкретно это: `BiPredicate<T,U>`, `BiFunction<T,U,R>`, и `BiConsumer<T,U>`. Также существуют эти варианты для возвращения примитивных типов для Function: `ToIntBiFunction<T,U>`, `ToLongBiFunction<T,U>`, `andToDoubleBiFunction<T,U>`.

В общем, много всякого. Также следует стараться использовать варианты для примитивных типов, а не использовать boxed типы. 

Вот эти 6 стандартных функциональных интерфейсов выше стоит запомнить - а там смотреть по ситуации, можно ли найти по семантике схожий, но более специфичный по параметрам и типам параметров.

Однако, существуют случаи когда надо будет написать все таки свой функциональный интерфейс. Первое - это, очевидно, когда нет подходящего стандартного интерфейса. 

Второе - это отдельный случай, когда даже если имеется нужная структура интерфейса, но все равно лучше написать свой. Например, *Comparator\<T\>* по структуре схож с *ToIntBiFunction\<T,T\>*, но все равно заслуживает существования по следующим причинам:
- Имя интерфейса более специфично для задач, и код становится понятнее, когда там используется имеется Comparator, а не ToIntBiFunction\<T,T\>, тем более данный интерфейс встречается в коде часто.
- Второе - контракт интерфейса. Написав отдельный специфичный интерфейс, можно написать под него контракт, который необходимо соблюдать при реализации данного интерфейса. И Comparator имеет такой контракт (возвращение -1, 0, 1).
- И наконец, интерфейс Comparator хоть и является функциональным интерфейсом с одним абстрактным методом, но имеет полезные дефолтные методы.

И так, причины для написания своего функционального интерфейса при существовании схожего по структуре стандартного, следующие:
- It will be commonly used and could benefit from a descriptive name
- It has a strong contract associated with it
- It would benefit from custom default methods

Если выполняется хотя бы одно из этих правил, то следует писать свой функциональный интерфейс.

Следует упомянуть такую аннотацию, как *@FunctionalInterface*. Она схожа с аннотацией @Override, и служит для следующих целей:
- Дает читателю класса подсказку, что данный интерфейс следует использовать как функциональный интерфейс, то есть использовать лямбды
- Помощь со стороны компилятора - интерфейс не скомпилируется пока не будет иметь ровно 1 абстрактный метод
- Не дает расширять этот метод добавляя в него новые абстрактные методы (компилятор выдаст ошибку).

Всегда следует использовать эту аннотацию при написании функционального интерфейса.

---

### Item 45:  Use streams judiciously

Итем говорит: используйте стримы с умом.

Stream API было добавлено в Java 8 для совершения bulk операций, последовательно или параллельно. Поговорим немного об этом API.

Stream API имеет 2 составляющие: *streams*, которые являются конечной или бесконечной последовательностью данных, и *stream pipeline*, которое представляет multistage выполнение на этих элементах, производимых стримом.

Stream pipeline состоит из исходного стрима и 0 или более *промежуточных операций (intermediate operations)* и одной *завершающей операции (terminal operation)*.

Каждая промежуточная операция каким-либо образом изменяет стрим. Более формально, все промежуточные операции трансформируют один стрим в другой, элементы которого могут быть того же типа, что и на предыдущем шаге, или другого типа. Терминальная операция производит финальные вычисления на результирующем стриме с последней операции, например - сохранение элементов в коллекцию, возвращая определенный элемент или печатая все элементы стрима.

Stream pipeline выполняются **lazily**: вычисления не начинаются пока завершаюшая операция не началась выполняться, и элементы которые не требуются для завершения терминальной операции, никогда не будут вычисляться.

Да, стримы делают код короче и понятнее, но это не всегда не так. Не следует использовать стрим просто потому что можешь. Если прмиенять неправильно, то код становится тяжело понимаемым и тяжело читаемым.

Как вообще стрим пайплайн делает вычисления? С помощью лямбд. Итеративный код, то есть цикл, использует для вычисления блоки кода. Блоки кода позволяют делать некоторые вещи которые не возможны в стримах:
- Из блока кода можно изменять локальные переменные. В лямбде можно только читать final или effectively final переменные
- Из блока кода можно использовать `return`, `break`, `continue`, бросать checked исключения. Из лямбды нельзя делать ни одно из этих вещей

Однако, стримы имеют следующие преимущества:
- Единообразно трансформировать последовательности элементов
- Фильтровать последовательности элементов
- Собирать последовательности элементов в коллекцию, при необходимости группируя как-то
- Искать в последовательности какой-то элемент удовлетворяющий условиям
- Комбинировать несколько последовательностей элементов (стримов), используя одну операцию (Stream.concat)

---

### Item 46: Prefer side-effect-free functions in streams

Самая важная часть парадигмы стримов это структруирование вычислений как такую последовательность трансформаций, где результат каждого шага близок к результату возможной *чистой функции (pure function)*. Pure function - это такая функция, результат которой зависит только от входа, а не какого то изменяемого состояния, и она не изменяет состояние. Для того, чтобы достичь этого, любые function objects которые мы передаем в операции стримов должны быть избавлены от сайд эффектов (side-effects).

**Немного о чистых функциях**.
Есть такое понятие, как побочные эффекты функции (side effects). Из вики:
> В императивных языках некоторые функции в процессе выполнения своих вычислений могут модифицировать значения глобальных переменных, осуществлять операции ввода-вывода, реагировать на исключительные ситуации, вызывая их обработчики. Такие функции называются функциями с побочными эффектами. Другим видом побочных эффектов является модификация переданных в функцию параметров (переменных), когда в процессе вычисления выходного значения функции изменяется и значение входного параметра. 
> Описывать функции без побочных эффектов позволяет практически любой язык программирования. Однако некоторые языки поощряют или даже требуют от некоторых видов функций использования побочных эффектов. Например, во многих объектно-ориентированных языках в функцию-член класса передаётся скрытый параметр — указатель на экземпляр класса, от имени которого вызывается соответствующая функция (например, в C++ этот параметр называется this), который эта функция неявно модифицирует. Тем не менее, в языке C++ можно указать для метода класса модификатор const, тем самым сообщив компилятору о том, что метод не модифицирует данные класса. 

**Побочный эффект функции** — возможность в процессе выполнения своих вычислений: читать и модифицировать значения глобальных переменных, осуществлять операции ввода-вывода, реагировать на исключительные ситуации, вызывать их обработчики. Если вызвать функцию с побочным эффектом дважды с одним и тем же набором значений входных аргументов, может случиться так, что в качестве результата будут возвращены разные значения.

Важное - это **не использовать *forEach* кроме того случая, когда надо представить результат вычислений - вывести или сохранить его куда-то.**

Мутный айтем какой-то получился, да, но хоть вспомнил о чистых функциях.

---

### Item 47: Prefer Collection to Stream as a return type

Множество методов возвращают какие-то последовательности элементов. До Java 8 это были *Collection*, *List*, *Set*, *Iterable* и массивы.
Если возвращаемая последовательность использовалась только для прохождения по ней, то это был очевидный *Iterable*. Если же стоял вопрос производительности, или надо было использовать примитивные типы, то использовался массив.

С появлением стримов кажется что надо возвращать именно их, но это совсем не так. Как уже обсуждали раннее, обычный итеративный проход по коллекциям совсем не стал obsolete, и наиболее удачное решение это вообще совмещение обоих подходов.

Поэтому нельзя возвращать только стримы. Можно легко перейти от обычной коллекции или Iterable к стриму, но не наоборот, потому что Stream не расширяет интерфейс Iterable. Вообще, это можно сделать, используя каст к итератору (потому что type inference не сработает без этого) + метод iterator() у интерфейса BaseStream, но такой код слишком надоедливый и непрозрачный на практике. Вот это тот код:
```java
// Hideous workaround to iterate over a stream
for (ProcessHandle ph : (Iterable<ProcessHandle> ProcessHandle.allProcesses()::iterator)) {
    ...
}
```

Легче для этого использовать метод-адаптер для превращения из Stream в Iterable (не стандартный, самописный):
```java
// Adapter from  Stream<E> to Iterable<E>
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}
```

Теперь его можно использовать так:
```java
for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
    ...
}
```

И наоборот, если API возвращает только Iterable, то тот кто захочет использовать Stream будет расстроен. Однако, точно также можно написать метод:
```java
// Adapter from Iterable<E> to Stream<E>
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterator(), false);
}
```

Конечно, если вы уверены, что возвращаемая последовательность будет использоваться только в стрим пайплайнах, то можно без сомнений возвращать стрим, и наоборот, если вы уверены, что возвращаемая последовательность будет использовать только для итерации по ней, то можно возвращать Iterable.

Но если есть такая возможность, что пользователи захотят использовать оба пути, то следует давать им такой выбор в API, пока вы точно не уверены, что пользователи точно предпочтут только один механизм.

Следует упомянуть *Collection*, который наследуется от Iterable, а также предоставляет метод *stream()*, поэтому Collection может использоваться дл обоих механизмов, и поэтому в общем Collection это наилучший тип для возвращения какой-то последовательности из API.

Массивы также могут использоваться обоими путями с помощью методов *Arrays.asList* и *Stream.of*.

Однако, для того чтобы написать имплементацию *Collection* поверх *AbstractCollection*, необходимо реализовать два метода, необходимых для Iterable: *contains* и *size*. Если это невозможно сделать (например, по причине того что коллекция не предопределена перед итерацией, а элементы вычисляются по ходу), то следует возвращать все таки Iterable или Stream, или вообще оба, если дизайн API такой.

Итак, подведем итог: когда мы пишем метод, возвращающий какую-то последовательность элементов, следует помнить что разные пользователи захотят использовать оба метода. Следует рассмотрим обе группы пользователей. Если можно вернуть Collection, то очевидно лучше возвращать её. Если же вернуть коллекцию не осуществимо, и вы уверены, что более подходяще всегда использовать только 1 механизм, то смело возвращайте только 1 тип - Stream или Iterable. Если же понадобятся оба - можно возвращать оба типа. Если в будущих релизах Java стрим станет экстендить Iterable - то можно будет возвращать всегда Stream.

---

### Item 48: Use caution when making streams parallel

Хоть Джава делает многое для облегчения написания многопоточных приложений, параллельные стримы все также должны использовать с осторожностью, как и любой конкуррентный код в принципе.

Стримы легко могут быть распалелены с помощью метода *parallel()*. Но следует быть осторожным.

Первое - распалеливание стрима вряд ли улучшит производительность, если соурс стрим из *Stream.iterate*, или использована промежуточная операция *limit*.

Второе - производительность повышается от распалеливания на инстансах ArrayList, HashMap, HashSet, and ConcurrentHashMap; массивах; int ranges и long ranges. Что делает эти структура данных особенными в этом моменте, так это то, что их элементы могут быть аккуратно и дешево разделены (splitted) по subranges любого размера, что делает легким разделение работы по нескольким тредам. Для выполнения этого разделения используется абстракция под названием `spliterator`, который возвращается методом *spliterator()* на Stream или Iterable.
Другой плюс этих структур данных, что они имеют good-to-excellent *locality of reference*. Без этого треды проводили бы много времени в ожидании загрузки данных из памяти в кэш процессора. Структура данных с самым лучшим locality of reference - это примитивные массивы, так как там сами данные, а даже не ссылки, расположены последовательно в памяти.

Также стоит рассмотреть роль терминирующей операции. Лучший тип терминирующей операции для параллелизма это *reduction*, где все элементы комбинируются с помощью методов Stream *reduce()* или таких редукций как *min*, *max*, *count*, *sum*. Метод стрима *collect()* также может быть использован в параллелизме.

Немного отвлечемся. Чем отличаются *reduce* метод и *Stream.collect*? Дело в том, что *reduce* операция каждый раз, при обработке нового элемента, возвращает новое значение. И *reduce* точно нельзя использовать для добавления элементов в коллекцию, иначе каждый раз, с добавлением нового элемента, создавалась бы новая коллекция. 

Метод *collect* работает по-другому: он изменяет одну коллекцию, и не создает новых, именно поэтому называется *mutable reduce*.

Сейчас мы говорили только о проблемах с производительностью. Однако параллелизм может привести также и к непредсказуемому поведению и некорректным результатам. Дело в том, что на различные mappers, filters и другие кастомные промежуточные функции накладываются определенные строгие ограничения, и их нарушение может привести к некорректной работе программы.

Однако, даже предпологая, что все советы соблюдены, распаралеливание стримов все равно может не привести к какому-то выигрышу. Дело в том, что выполняемая работа могут быть маленькой, тогда затраты потраченные на распаралеливание будут совместимы с выигрышем от распаралеливания, и выигрыша в чем-то не будет.

Самое главное, помнить, что параллелизм - это исключительно о производительности. Поэтому необходимо тестить производительного этого места, где применяется параллелизм, до и после распаралеливания.

Также следует помнить, что параллельные задачи стримов выполняются в общем джава fork-join пуле (standard fork-join pool), и плохое применение парраллелизма в одном месте программы может повлиять на другие, даже не связанные места программы.

---

# Chapter 8 - Methods

Эта часть покрывает такие аспекты дизайна: как относиться к параметрам и возвращаемым значениям, как дизайнить API (параметры метода) и как документировать методы. Большинство материала здесь применяется также и к конструкторам.

---

## Item 49: Check parameters for validity

Кратко: следует валидировать все параметры. Это принцип fail-fast: следует упасть как можно раньше чтобы потом не произошла еще более херовая херня и вообще ошибку было легко отследить и исправить этот момент, а не пытаться найти ее потом, когда она затронет уже много моментом программы из этого метода.

Итак, что конкретно может пойти не так:
- Выбросить непонятное, неуместное исключение посреди выполнения
- Хуже - вернуть значение, тихо вычисленное не смотря на ошибку, и конечно же оно будет не верным
- И самое худшее - вернуться нормально, но затронуть какой-то момент программы так, что ошибка связанная с этим появится в неопределенный момент времени в будущем и неопределенном месте кода

Для документации следует использовать аннотацию `@throws`, чтобы показать что при нарушении валидации может выброситься такой метод. Обычно, это:
- `IllegalArgumentException`
- `IndexOutOfBoundsException`
- `NullPointerException`

Не надо описывать только `NullPointerException` для всех параметров. Дело в том, что это исключение уже *может быть* задокументировано для класса параметра. Например, BigInteger:
>  All methods and constructors in this class throw
   {@code NullPointerException} when passed
   a null object reference for any input parameter.

Если же все таки надо проверить на null, то следует использовать метод `Objects.requireNonNull`. Он удобен тем, что позволяет сразу присваивать и проверять значение. Но его можно использовать и только для проверки. Например:
```java
// Inline use of Java's null-checking facility
this.strategy = Objects.requireNonNull(strategy, "strategy")
```

Для неэкспортированных методов, не доступных публично, при разработке можно использовать `assert`. В отличие от обычных валидаций с помощью исключений, assert-ы не взимают дополнительных трат. Но этот подход больше подходит для каких-то либ и совсем уже закрытых кусков. Обычно такое не практикуется в продакшене в проектах.

Также важно проверять на валидность параметры, которые не используются в метода прямо сейчас, но сохраняются для дальнейшего использования. Это важно, потому что такие параметры могут вызвать ошибку далеко во времени в приложении, и отследить ее будет также трудно.
Например, представь. Есть метод для создания листа из передаваемых int-ов. Если бы он не проверял параметры на null, то при возвращении листа клиент при попытке воспользоваться контентом листа получил бы NPE, но к тому времени источник листа уже было бы сложно определить. То есть здесь сохранение параметров не значит только сохранение их в инстансе класса, в котором определен метод (в какую то мапу например и т.д.), но и этот кейс тоже, да.

То же самое относится и к конструктору - там это даже важнее, так как инстансом класса будут пользоваться, и инстансом может быть много, а значит больше мест для ошибок.

Однако, существуют и исключения для случаев валидации - это когда выполняются оба (и только оба!) следующих условия: проверка слишком дорога и в методе существует неявная валидация (implicit validation). Например, в методе `Collections.sort()` все элементы массива должны быть попарно сравнимы. Если бы мы проверяли все элементы на входе, это было бы слишком дорого. Наоборот, они проверяют в ходе сравнения. Если элементы не могут быть сравнены - то бросается `ClassCastException`. Однако, следует помнить что это может нарушать **failure atomicity** (то есть параметр после бросания исключения должен быть снова возможным для использования, при чем в том же виде, в котором он был до передачи в метод). Здесь этот принцип нарушается потому, что некоторая часть массива уже будет отсортирована после бросания исключения.

Не следует полагаться на конкретные ситуации, всегда следует учитывать ограничения думая в целом, более обще. Не стоит думать: "зачем вообще я бы стал передавать такой параметр" - произойти может все, особенно когда это большая кодовая база, и параметр проходит дооолгий путь от одной точки программы к другой. А на этом пути может встречаться множество багов и ошибок в вычислениях.

---

## Item 50: Make defensive copies when needed

Это невозможно изменить внутреннее состояние объекта без помощи самого объекта, но иногда мы можем предоставить такую помощь, даже если честно не хотели этого. Так можно ошибиться, написав "иммутабельный" класс, но на самом деле он не будет таковым. Вот пример:
```java
// Broken "immutable" time period class
public final class Period {
    private final Date start;
    private final Date end;
    
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)throw new IllegalArgumentException(start + " after " + end);
        this.start = start;
        this.end   = end;
    }

    // getters & other methods
    // ...
}
```

```java
// Attack the internals of a Period instance
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78);  // Modifies internals of p!
```

Проблема возникает здесь из-за того, что мы не можем изменить внутренние состояние Period меняя ссылку на объект Date (т.к. поле final), да, это верно, но можем изменять внутреннее состояние Period через ссылку на объект Date, доступную вне, из-за того, что в объекте по ссылке имеются возможности для модификации внутреннего состояния. В случае Java 8, эту проблему можно было бы решить с помощью `Instant` (или `LocalDateTime` или `ZonedDateTime`). 

Однако, что же делать если нет возможности поменять класс и мы имеем дело с такими объектами?

Для того чтобы защитить внутренние объекты Period от этого типа атаки, мы должны делать **defensive copy** каждого такого параметра в конструкторе и использовать их вместо оригинальных объектов.

Вот как следует изменить конструктор:
```java
// Repaired constructor - makes defensive copies of parameters
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end   = new Date(end.getTime());

    if (this.start.compareTo(this.end) > 0) {
        throw new IllegalArgumentException(this.start + " after " + this.end);
    }
}
```

Следует заметить, что **валидация осуществляется на defensive copy объекта, а не оригинале**. Это защищает нас от случая, когда появляется окно между моментами, когда осуществлялась проверка и установились ссылки на объекты в самом инстансе, который мы строим в конструкторе. То есть, валидировать мы могли одно, а установить другое совсем потом (хотя и осуществилась defensive copy). То есть, тред переключился в момент валидации, ссылка поменялась, валидация завершается успешно и устанавливается ссылка на тот же объект, но теперь он измененный, и валидация на нем не осуществлялась. Таким образом, мы имеем (можем иметь) невалидный объект в инстансе, который построили. В компьютерной безопаности это называется *time-of-check/time-of-use* или *TOCTOU* атака.

Также **не следует использовать метод `clone`** для создания defensive copy. Этот метод может возвращать инстанс совсем не того класса, а специально измененный для совершения атак инстанс.

Теперь еще одна важная вещь. Мы сделали defensive copy в конструкторе, но геттеры все равно предоставляют возможность получить скопированный объект и изменить его! **Мы должны не только сделать копию, но и не дать возможности изменить эту копию!**. Сделаем геттеры такими:
```java
// Repaired accessors - make defensive copies of internal fields
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}
```

Все, теперь можем выдохнуть. С этими новыми конструктором и геттерами наш класс Period стал по настоящему иммутабельным.

Следует понять, что defensive copy может использоваться не только для дизайна иммутабельных классов, но и просто для того, чтобы поддерживать правильное состояние любого класса, который опирается на какой-то объект, который потенциально изменяем (mutable). Если любые изменения с этим объектом воспрещены, то следует делать его defensive copy при отдаче наружу. Если же изменения не запрещены, или класс сможем с этим смириться и работать правильно дальше, то все конечно же ок.

То есть, **не важно, иммутабельный класс или нет, всегда нужно подумать дважды прежде чем возвращать ссылку на изменяемый объект**.

Конечно, defensive copy требует определенных трат, и иногда это ОК довериться и отдать изменяемый класс. Больше всего это подходит, если метод package-private, то есть классы находятся в одном пакете. Ну и конечно, такие ограничения на изменение следует задокументировать.

Даже если происходит взаимодействие между пакетами, в таких случаях тоже может не требоваться defensive copy. Это тот случай, когда контроль над объектом передается классу, и объект больше не используется, а взаимодействие с ним будет происходит только через класс. То есть, когда вызывается такой метод, клиент обещает, что больше не будет изменять объект. Следует сказать, что этот момент должен быть четко описан в документации, чтобы клиент знал, что метод или конструктор ожидает передачи контроля и изменять напрямую сам объект после отдачи в метод или конструктор.

Однако, такие методы и конструкторы не могут защитить себя от атак, когда объект будет все таки изменяться, нарушая гарантии. Такие классы все еще возможны, НО **если происходит нарушение гарантии, то это нарушение должно вредить только клиенту, а не всему приложению**! Пример такого класса - Wrapper pattern. Если мы заврапили какой-то объект и позже используем сам объект напрямую, то это может навредить только нам самим.

**То есть, здесь ситуация точно такая же как и с валидацией**. Если выполняются 2 следующих условия (и только оба!), то можно не делать defensive copy: затраты слишком большие И клиент и класс доверяют друг другу. Но в документации запрещение изменений конечно же должно быть описано.

---

## Item 51: Design method signatures carefully

Здесь говорится о том, как дизайнить методы.
Тут выделю моменты которые мне приглянулись, большинство сам знал.

1. Использовать тип интерфейса вместо конкретного класса для параметров и возвращаемых значений. 
Почему это важно:
- Используя класс вместо интерфейса, мы ограничиваем конкретной имплементацией и заставляем при необходимости делать копирование данных, если потребуется хранить данные в другой форме чем мы ожидаем (параметры) или отдаем (возвращаемое значение).

2. Использовать two-element enum вместо boolean
Следует делать это, пока boolean параметр метода смысл. Например, такой класс намного лучше для шкалы термометра:
```java
public enum TemperatureScale { FAHRENHEIT, CELSIUS }
```

чем boolean тип, потому что такое поле явно отражает смысл. Параметры типа `isCelsius` сложны для восприятия и чтения кода.
Например, сравни:
```java
Thermometer.newInstance(TemperatureScale.CELSIUS)
```

И

```java
Thermometer.newInstance(true)
```

По первому понятно, что шкала термометра в цельсиях, по второму - непонятно нихрена, пока не заглянешь в доки метода. Неприкольно постоянно смотреть на доки.

Более того, такие **enum-ы позволят добавить больше вариативности**, если потребуется. Например, шкала может быть и в Кельвинах.

Кроме того, мы можем переместить методы, опирающиеся на конкретные enum константы, в сам класса enum-а, и сделать класс более легким и ненагроможденнным методами.

---

## Item 52: Use overloading judiciously

Первое, что следует запомнить, это то, что **выбор перегруженного метода для вызова происходит в компил тайме**. Не следует путать с overriden методом (@Override). Да, нужный overriden метод определяется в ран тайме на основе типа субкласса, метод которого вызываем, но перегруженные - в компил тайме.

Пример ошибочного использования перегрузки. Пусть есть класс, который определяет тип коллекции:
```java
// Broken! - What does this program print?
public class CollectionClassifier {
    public static String classify(Set<?> s) {return "Set";}
    
    public static String classify(List<?> lst) {return "List";}
    
    public static String classify(Collection<?> c) {return "Unknown Collection";}

    public static void main(String[] args) {
        Collection<?>[] collections = {
            new HashSet<String>(),
            new ArrayList<BigInteger>(),
            new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections) {
            System.out.println(classify(c));
        }    
    }
}
```

Мы могли ожидать вывода `Set`, `List`, `Unknown Collection` поочередно, но этот кусок кода выведет 3 раза `Unknown Collection`, потому что в компил тайме каждый инстанс, храниящийся в коллекции `collections`, при прохождении в forEach цикле имеет тип `Collection<?>`.

**Лучший способ - это избегать перегрузки** и использовать методы с разными названиями. Но есть и исключение из правила - например, когда два параметра имеют абсолютно разные типы и невозможно скастить один параметр к другому.

Но вообще да, перегрузка - это плохо. Хотя бы просто потому, что алгоритм перегрузки просто очень сложен.

---

## Item 53: Use varargs judiciously

**Varargs method** - это метод, который принимает 0 или более аргументов указанного типа. 

Varargs работают следующим образом:
1. Создается массив такого размера, сколько аргументов было передано в метод
2. Аргументы кладутся в массив
3. Массив передается в метод

Первая проблема varargs в том, что если метод ожидает передачи гарантированно 1 и более аргументов (а не 0 и более), то это можно проверить только в рантайме и бросить какое-нибудь исключение. Конечно же лучше проверка в компил тайме. Да и вообще, такая проверка очень некрасива.

Но здесь есть способ намного лучше достичь этого эффекта: поменять сигнатуру метода таким образом, чтобы первым параметром был обычный тип, а вторым - varargs того же типа.

Однако, здесь стоит помнить о том, что varargs - это всегда создание и заполнение массива. Поэтому, если производительность сильно важна, то можно создать методы с разным набором параметров, а на непокрытые случаи - оставить varargs. Например, если вы определили, что 95% вызовов метода происходят с передачей 3 аргументов, то объявите 3 метода с разным набором параметров, а оставшийся сделайте с varargs. Вот пример:
```java
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```

Как вы помните, *EnumSet* задумывался как схожая эффективности замена bit fields. Поэтому EnumSet старается следовать этому максимально и в нем определены конструкторы для разного количества параметров, почти как выше.

---

## Item 54: Return empty collections or arrays, not nulls

Итем прост: следует возвращать из методов не `null`, а просто пустую коллекцию. Причины:
1. Пустая коллекция подходит лучше по семантике.
2. Для проверки на `null` требуется дополнительный и абсолютно ненужный код в любом месте, где используется коллекция.
3. Этот подход error-prone, потому что легко можно словить NPE при неосторожном обращении с коллекцией и при отсутствии проверки на `null`, когда использование пустой коллекции не дало бы исключения.

То же самое относится и к массивам - вместо null возвращайте массив размера 0.

Не стоит здесь думать много о производительности. Вообще, бизнес код мало задумывается о производительности, главное все должно быть понятно и хорошо работать. И пока измерения не покажут, что проблема действительно в этом, не стоит заниматься оптимизациями.

---

## Item 55: Return optionals judiciously

До Java 8 в случае, когда необходимо было показать отсутствие по какой-либо причине вычисленного значения при возврате из метода бросалось исключение или возвращался `null`. Ни один из этих методов не был хорош. Первый тем, что исключения должны использоваться правильно, для исключительных событий, а также тем, что они довольны затратны, так как требуют снятия всего стэк трейса. Второй способ плох тем, что он error-prone (можно легко словить NPE в каком-то месте программы, и особенно плох тем, что при отсутствии проверок и сохранении null значения на будущее, ошибка может возникнуть внезано в неопределенный момент времени) и требует проверки на `null` до тех пор, пока программист (клиент метода) не докажет, что возвращение `null` невозможно.

В Java 8 были добавлены опшенелы (по факту, это монада Maybe).

Почему возвращение Optional\<T\> лучше, чем возвращение null?
1. Семантика - опшенел явно показывает, что значения нет
2. Другой тип - вы не сохраните опшенел куда-то или что-то сможете с ним делать, пока не анврапнете контейнер
3. Fail-fast - если значения нет, но get() вызывается, то бросается NoSuchElementException

Стоит запомнить, что **нельзя возвращать null из метода, который задекларирован возвращать Optional\<T\>**.

Вообще, можно заметить, что опшенелы похожи на **Checked Exception**. Похожи тем, что ставят клиента метода перед фактом, что здесь может отсутствовать значение и не обработать этот кейс нельзя никак. Бросание unchecked exception или null позволяет клиенту игнорировать возможность не получения значения.

Однако, **не все типы хороши для обертывания в опшенел**. Такие контейнерные типы, как коллекции, мапы, стримы, массивы и опшенелы (!!) не должны оборачиваться в опшенелы. Например, вместо возвращения Optional\<List\<T>> лучше просто вернуть пустой List\<T> и все.

Также, **опшенел не должен использоватья как ключ или значение в любой коллекции**.

Вообще, опшенел используется в основном именно как возвращаемое значение, а другие кейсы не подходят.

---

## Item 56: Write doc comments for all exposed API elements

Тут про важность документации и правильную документацию. Ну, это и так понятно, что доки важны, так что здесь писать ничего не буду.

---

# Chapter 9 - General Programming

Здесь говорится об общих моментах программирования, просто почитаю это без конспектирования.

---

# Chapter 10 - Exceptions

Эта глава дает рекомендации по правильному использованию исключений.

---

## Item 69: Use exceptions only for exceptional conditions

Мораль этого итема такая: использовать исключения для исключительных ситуаций, а не контроля флоу программы.

Например, если метод зависит от какого-то состояния, то следует иметь дополнительный метод для проверки состояния, а не обрабатывать состояние с помощью исключения. Например, пусть Iterator не имел бы метода *hasNext*, тогда код выглядел бы следующим образом:
```java
// Do not use this hideous code for iteration over a collection!
try {
    Iterator<Foo> i = collection.iterator();
    while(true) {
        Foo foo = i.next();
        ...
    }
} catch (NoSuchElementException e) {
}
```

Этот код очень плох. Во-первых, исключение используется здесь только для контроля флоу программы, а не обработки исключительной ситуации. Во-вторых, внутррений код while-блока может тоже выбросить такое исключение, и тогда цикл завершится, не пройдя всю коллекцию, а сильно раньше, что является багом.
Такие методы называются state-dependent (например, *iterator.next()*) и state-testing (например, *iterator.hasNext()*) соответственно.
State-dependent же должен бросать какое-то исключение при невалидном состоянии.

Алтернатива предоставлению state-testing метода это сделать state-dependent таким, чтобы он возвращал empty Optional или какое то раздельное значение (например, *null*) для обозначения невалидного состояния. Тогда он не должен бросать исключение.

Но лучше использовать все таки state-testing метод по следующим причинам:
1. Если забыть вызвать state-testing метод, то state-dependent метод бросит исключение, и баг будет явно виден. Если забыть проверить возвращаемое значение или опшенел, то баг может быть не обнаружен сразу, а это плохо.
2. Код становится читаемее и правильнее

Отдельное возвращаемое значение может понадобиться заместо state-testing метода по следующим причинам:
1. Если метод используется конкуррентно, тогда между вызовом state-testing метода и state-dependent метода появляется временное окно, во время которого тред может свитчнуться и состояние может измениться вызовами из другого треда.

Итак, подытожим:
1. Не использовать исключения для контроля флоу программы
2. Не писать API, которое заставит других делать это

---

## Item 70: Use checked exceptions for recoverable conditions and runtime exceptions for programming errors

Просто пару советов:
- Использовать checked exceptions для случаев, когда восстановление из такого состояния гарантированно возможно, и unchecked для случаев, когда восстановление невозможно. Если не уверены, возможно ли восстановление, также лучше использовать unchecked exception.
- Использовать unchecked exceptions для обозначения программных ошибок. Большинство таких ошибок - это нарушение preconditions (условия которые должны быть соблюдены клиентом перед вызовом API). Например, unchecked исключение `ArrayIndexOutOfBoundsException` говорит о том, что был нарушен контракт на то, что индекс массива должен быть от 0 до (array.size - 1).

**Errors** же зарезервированы для JVM (хоть это JLS не требует этого, но это просто соглашение), поэтому все unchecked exceptions должны расширять класс *Runtime Exception*, а не *Error*.

Также не стоит создавать классы-исключения, которые не расширяют ни *Exception*, ни *RuntimeException*, ни *Error*, а расширяют только *Throwable*. Джавой подразумевается, что такое исключение - это checked exception. Но таких случаев, когда понадобится такой класс, просто нет, и если делать так, то это только запутает клиента API.

---

## Item 71: Avoid unnecessary use of checked exceptions

Много программистов не любят checked исключения, но используя их правильно, можно улучшить API и программы. В отличии от кодов ошибок или unchecked исключений, они заставляют клиента API обработать исключительную ситуацию, улучшая надежность программы. Но нельзя злоупотреблять checked исключениями - обработка исключений требует дополнительного кода и может сделать API сильно меньше привлекательным для использования.

Итак, этот burden с checked исключениями может быть оправдан в следующих случаях:
- Исключительная ситуация не может быть гарантированно предовтращена правильным использованием API (соблюдением preconditions)
- После того, как поймали checked exception, можно принять какие-нибудь полезные действия

Прежде чем бросать checked исключение, всегда следует думать, можно ли обработать вообще такое исключение нормальным путем?

И еще, если метод бросает *только одно checked исключение*, стоит задуматься, а можно ли избавиться от такого исключения? Sole исключение плохо тем, что в отличие от методов где уже есть исключение, потребует try catch блока, загромождая код, а во вторых, методы с checked исключениями не могут быть нормально использованы в стримах. Как пример, можно избавиться, возвращая пустой опшенел вместо checked исключения.

Итак, checked исключения стоит бросать, только если можно сделать recover из исключительной ситуации И если вы хотите поставить клиента API перед фактом возможного исключения, т.е. хотите чтобы он обработал это явно. Однако, сначала надо спросить, не достаточно ли будет просто вернуть пустой опшенел? Если это дает достаточно информации, то лучше вернуть пустой опшенел - явная обработка все равно остается. Если же информации недостаточно, только тогда следует бросить checked исключение - так как исключения тем и хороши, что позволяют нести внутри себя какую-то дополнительную информацию, так как это полноценный класс.

---

## Item 72: Favor the use of standard exceptions

Мораль айтема такая: изучить стандартную библиотеку джавы на наличие подходящего под ситуацию исключения и не строить свое исключение до.

Переиспользование стандартных исключений объяснимо легко: API становится более понятным и легким для изучения, так как программисты уже привыкли к стандартным исключениям и им не придется изучать новые, вами построенные исключения

---

## Item 73: Throw exceptions appropriate to the abstraction

Айтем говорит, что на верхнем уровне абстракции следует отдавать исключения, соответствующие этому уровню абстракции, и не следует отдавать исключения нижнего уровня, так как таким образом мы раскрываем implementation details. Процесс перевода одного типа исключения в другой называется **exception translation**. 

Exception translate может применяться и для перевода исключения нижнего уровня в исключение верхнего уровня.

**То есть, на верхнем уровне абстракции следует отдавать исключение, которое может быть объяснено в терминах этого уровня**.

Пример exception translation:
```java
// Exception Translation
try {... // Use lower-level abstraction to do our bidding
} catch (LowerLevelException e) {
    throw new HigherLevelException(...);
}
```

Однако, иногда для дебагга ошибки было бы хорошо получать доступ к исключению нижнего уровня. Тогда исключение нижнего уровня следует положить в исключение верхнего как *cause* и получать с помощью вызова метода класса Throwable *getCause()*. Этот процесс называется **exception chaining**.

Пример exception chaining:
```java
// Exception Chaining
try {... // Use lower-level abstraction to do our bidding
} catch (LowerLevelException cause) {
    throw new HigherLevelException(cause);
}
```

Также стоит знать, что *exception chaining* включает стек трейс исключения нижнего уровня в стек трейс исключение верхнего уровня, что также удобно.

Однако, для конструкции такого исключения в классе исключения должен быть *chaining-aware конструктор*:
```java
// Exception with chaining-aware constructor
class HigherLevelException extends Exception {
    HigherLevelException(Throwable cause) {
        super(cause);
    }
}
```

Многие стандартные исключения также содержат такой конструктор, но если его нет, то можно установить cause исключение с помощью вызова метода Throwable *initCause()*.

Но не стоит злоупотреблять этим всем - иногда лучше просто тихо обработать исключения нижних уровней, не сообщая об исключении верхним уровням. **Но логировать исключения нижнего уровня все-таки надо!**

Итак, подведем итог. Если не получается обработать исключенин нижнего уровня тихо (+залогировав), то следует использовать exception translation. А еще лучше делать, где уместно, exception chaining - таким образом клиент метода верхнего уровня сможет понять, чем было вызвано исключение.

---

## Item 74: Document all exceptions thrown by each method

**Следует документить все возможные исключения, которые могут броситься этим методом, а также надо документить все возможные условия, при которых может броситься это исключение**. И не надо документить какой-то супер класс вместо отдельных исключений - от этого нет смысла. Например, `@throws Exception` - мало информации тут, точнее ее вообще нет.

**Не следует забывать и об unchecked исключениях!** Потому что они представляют собой программные ошибки, и посмотреть на условия, при которых они вызываются, будет полезно каждому программисту, работающему с API.

Вообще, хорошо задокументированные unchecked исключения составляют *список preconditions*, при которых метод гарантированно выполнится успешно. А так как это важно писать preconditions, при которых метод выполняется успешно, то список хорошо задокументированных unchecked исключений как раз делает эту работу.

Также важно писать список возможных unchecked исключений для методов интерфейсов - это позволит сохранять общее поведение при работе с различными имплементациями интерфейса.

Однако, тут важно заметить, что да, @throws аннотацию надо использовать для документации unchecked исключений, **но не надо использовать keyword *throws* в декларации метода**. Наличие исключения в документации, но отсутствие в списке исключений помеченных keyword *throws* дает сразу понять, что это исключение - unchecked, а отделять исключения важно.

---

## Item 75: Include failure-capture information in detail messages

Когда программа падает из-за какого-то исключения, в вывод автоматически печатается стек трейс исключения. Поэтому очень важно включить в исключение любую полезную информацию о том, чем было вызвано исключение.

Когда выводится стек трейс, этот вывод включает также *string representation* исключения, результат вызова метода *toString()*. Это string representation обычно состоит из названия класса исключения, за которым следует *detail message*.

Поэтому важно предоставить в *detail message* как можно больше информации, чтобы инспектировать ошибку, когда она возникнет.

Также стоит понимать, что detail message пишется для программистов и site reliability инженеров, а не для юзеров, поэтому контент более важен, чем читабельность.

---

## Item 76: Strive for failure atomicity

**Failure atomicity** - это принцип, по которому объект после бросания исключения должен быть в юзабельном состоянии и пригодном для повторного использования. В общем говоря, объект после бросания исключения должен остаться в том же состоянии, в каком он был до вызова метода, который бросил исключение. Метод, который обладает этим свойством, называется **failure-atomic**.

Если что, здесь говорится об объекте, метод которого вызывается, а не о параметрах.

Этот принцип особенно должен соблюдаться для checked исключений, так как клиент ожидает восстановиться после их обработки.

Есть несколько способов достичь этого эффекта. 

1. Первый способ - использование immutable объектов. Если объект иммутабелен, то failure atomicity обеспечивается бесплатно.

2. Второй способ - перед внесением изменений проверять параметры на валидность. Тогда исключение бросится раньше, чем объект изменится.

Давай рассмотрим такой пример:
```java
public Object pop() {
    if (size == 0) {
        throw new EmptyStackException();
    }
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete referencereturn result;}
}
```

Если бы здесь не было проверки на size, то исключение бросилось бы все равно. Однако, это оставило бы поле *size* в неконсистентном состоянии, заставляя все следующие вызовы *pop()* упасть также с исключением. Также, исключение *ArrayIndexOutOfBoundsException* брошенное бы в случае отсутствия проверки size было бы не подходящим для этого уровня абстракции. И в самом деле, стек может быть реализован по разному, и то что мы отдали бы ошибку ArrayIndexOutOfBoundsException, означало бы, что мы раскрыли детали имплементации.

3. Третий способ - создавать копию объекта и делать изменения на ней, а перед успешным возвратом из метода заменить объект копией. Этот способ обычно применяется, когда делать прямые изменения на объекте с последующей заменой намного эффективнее, чем делать постоянные проверки.

4. Последний способ - написать *recovery code*, который ловит ошибку посреди выполнения и возвращает объект к состоянию до начала операции.

Хотя failure atomicity возможно обеспечить почти всегда, но это может быть нежелательно из-за сложности или стоимости.

Если failure atomicity не может быть обеспечено, то документация должна явно об этом говорить.

---

## Item 77: Don’t ignore exceptions

Это вообще сказка. Но это я и сам прекрасно знаю, и знаю как важно не игнорить исключения, так что писать тут много не буду.

Важное скажу - если исключение все таки игнорируется, то надо обязательно оставить коммент почему оно игнорируется. Скорее всего, мы гарантированно знаем, что оно не произойдет (хотя checked исключения обычно используются, когда даже соблюдение preconditions не освобождает от возможности исключения), но это должно быть явно указано с комменте.

Также, если исключение игнорируется, то переменная исключения должна быть обозвана `ignored`.

Этот айтем применяется в равной степени к checked и unchecked исключениям.

---

# Chapter 11: Concurrency

Эта глава о конкуррентности. Но я здесь писать много не буду, просто буду читать, так как в будущем я планирую изучить книгу Java Concurrency in Practice.

Выделю один момент здесь. **Item 83: Use lazy initialization judiciously** очень важен, он показывает правильные техники делания lazy initialization - для static полей - про holder class, для instance полей - про double-check idiom и single-check idiom. Также там сказано про использование volatile.

---

Все, это все заметки по книге. Пытался собрать самое важное.