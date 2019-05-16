# JavaBooks-Notes

Item 21:  Design interfaces for posterity

Нужно продумывать дизайн интерфейса особенно тщательно с учетом того, что дальше может понадобиться добавить новый метод.
Когда добавляешь новый метод с default реализацией, следуем подумать, не нарушит ли это инварианты классов уже реализовавших интерфейс.

Item 22:  Use interfaces only to define types

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
