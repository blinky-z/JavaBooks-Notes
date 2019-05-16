# JavaBooks-Notes

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

Такой интерфейс состоит полностью из констант и это плохой дизайн. То, что класс использует какие-то константы - это деталь реализации и из-за того, что класс implements такой интерфейс, эта деталь реализации утекает в предоставляемое классом API.
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

Риск заключается в том, что если объявить несколько классов в одном файле, и потом объявить еще 2 таких же класса в другом файле, то какая будет использоваться реализация, будет зависеть от того, в каком порядке компилировались классы, что абсолютно недопустимо.