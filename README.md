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

**Factory** - это единичиный класс, без субклассов. Он используется тогда, когда создавать данные объекты в конструкторе сложно.

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
Анонимные классы имеют ссылку на enclosing класс, когда они объявляются в non-static методе. Иначе они не имеют ссылки на enclosing класс. Anonymous классы использовались раннее для созданиия function objects.

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

Единственный хак, как можно создать дженерик массив:
```java
List<?>[] listArray = new List<?>[64];
```

Это потому, что unbounded wildcard - reified тип. Строго говоря, non-reifiable type is one whose runtime representation contains less information than its compile-time representation. Because of erasure, the only parameterized types that are reifiable are unbounded wildcard types 

Подводя итог:
> In summary, arrays and generics have very different type rules. Arrays are covariant and reified; generics are invariant and erased. As a consequence, arrays provide runtime type safety but not compile-time type safety, and vice versa for generics. As a rule, arrays and generics don’t mix well.
