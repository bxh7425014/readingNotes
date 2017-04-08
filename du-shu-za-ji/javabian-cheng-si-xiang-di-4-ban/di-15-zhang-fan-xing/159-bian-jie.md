## 15.9 边界
本章前面简单地介绍过边界。边界使得你可以在用于泛型的参数类型上设置限制条件。尽管这使得你可以强制规定泛型可以应用的类型，但是其潜在的一个更重要的效果是你可以按照自己的边界类型来调用方法。
因为擦出移除了类型信息，所以，可以用无界泛型参数调用的方法只是哪些可以用Object调用的方法。但是，如果能够将这个参数限制为某个类型子集，那么你就可以用这些类型子集来调用方法。为了执行这种限制，Java泛型重用了extends关键字。对你来说有一点很重要，即要理解extends关键字在泛型边界上下文环境中和在普通情况下所具有的意义是完全不同的。下面的示例展示了边界的基本要素：
```java
//: generics/BasicBounds.java
interface HasColor { java.awt.Color getColor(); }
class Colored<T extends HasColor> {
  T item;
  Colored(T item) { this.item = item; }
  T getItem() { return item; }
  // The bound allows you to call a method:
  java.awt.Color color() { return item.getColor(); }
}
class Dimension { public int x, y, z; }
// This won't work -- class must be first, then interfaces:
// class ColoredDimension<T extends HasColor & Dimension> {
// Multiple bounds:
class ColoredDimension<T extends Dimension & HasColor> {
  T item;
  ColoredDimension(T item) { this.item = item; }
  T getItem() { return item; }
  java.awt.Color color() { return item.getColor(); }
  int getX() { return item.x; }
  int getY() { return item.y; }
  int getZ() { return item.z; }
}
interface Weight { int weight(); }
// As with inheritance, you can have only one
// concrete class but multiple interfaces:
class Solid<T extends Dimension & HasColor & Weight> {
  T item;
  Solid(T item) { this.item = item; }
  T getItem() { return item; }
  java.awt.Color color() { return item.getColor(); }
  int getX() { return item.x; }
  int getY() { return item.y; }
  int getZ() { return item.z; }
  int weight() { return item.weight(); }
}
class Bounded
extends Dimension implements HasColor, Weight {
  public java.awt.Color getColor() { return null; }
  public int weight() { return 0; }
}
public class BasicBounds {
  public static void main(String[] args) {
  Solid<Bounded> solid =
  new Solid<Bounded>(new Bounded());
  solid.color();
  solid.getY();
  solid.weight();
  }
} ///:~
```
你可能已经观察到了，BasiicBounds.java看上去包含可以通过继承消除的冗余。下面，可以看到如何在继承的每个层次上添加边界限制：
```java
//: generics/InheritBounds.java
class HoldItem<T> {
  T item;
  HoldItem(T item) { this.item = item; }
  T getItem() { return item; }
}
class Colored2<T extends HasColor> extends HoldItem<T> {
  Colored2(T item) { super(item); }
  java.awt.Color color() { return item.getColor(); }
}
class ColoredDimension2<T extends Dimension & HasColor>
extends Colored2<T> {
  ColoredDimension2(T item) { super(item); }
  int getX() { return item.x; }
  int getY() { return item.y; }
  int getZ() { return item.z; }
}
class Solid2<T extends Dimension & HasColor & Weight>
extends ColoredDimension2<T> {
  Solid2(T item) { super(item); }
  int weight() { return item.weight(); }
}
public class InheritBounds {
  public static void main(String[] args) {
  Solid2<Bounded> solid2 =
  new Solid2<Bounded>(new Bounded());
  solid2.color();
  solid2.getY();
  solid2.weight();
  }
} ///:~
```
HoldItem直接持有一个对象，因此这种行为被继承到了Colored2中，它也要求其参数与HasColor一致。ColoredDimension2和Solid2进一步扩展了这个层次结构，并在每个层次上都添加了边界。现在这些方法被继承，因而不必在每个类中重复。
下面是具有更多层次的示例：
```java
//: generics/EpicBattle.java
// Demonstrating bounds in Java generics.
import java.util.*;
interface SuperPower {}
interface XRayVision extends SuperPower {
  void seeThroughWalls();
}
interface SuperHearing extends SuperPower {
  void hearSubtleNoises();
}
interface SuperSmell extends SuperPower {
  void trackBySmell();
}
class SuperHero<POWER extends SuperPower> {
  POWER power;
  SuperHero(POWER power) { this.power = power; }
  POWER getPower() { return power; }
}
class SuperSleuth<POWER extends XRayVision>
extends SuperHero<POWER> {
  SuperSleuth(POWER power) { super(power); }
  void see() { power.seeThroughWalls(); }
}
class CanineHero<POWER extends SuperHearing & SuperSmell>
extends SuperHero<POWER> {
  CanineHero(POWER power) { super(power); }
  void hear() { power.hearSubtleNoises(); }
  void smell() { power.trackBySmell(); }
}
class SuperHearSmell implements SuperHearing, SuperSmell {
  public void hearSubtleNoises() {}
  public void trackBySmell() {}
}
class DogBoy extends CanineHero<SuperHearSmell> {
  DogBoy() { super(new SuperHearSmell()); }
}
public class EpicBattle {
  // Bounds in generic methods:
  static <POWER extends SuperHearing>
  void useSuperHearing(SuperHero<POWER> hero) {
  hero.getPower().hearSubtleNoises();
  }
  static <POWER extends SuperHearing & SuperSmell>
  void superFind(SuperHero<POWER> hero) {
  hero.getPower().hearSubtleNoises();
  hero.getPower().trackBySmell();
  }
  public static void main(String[] args) {
  DogBoy dogBoy = new DogBoy();
  useSuperHearing(dogBoy);
  superFind(dogBoy);
  // You can do this:
  List<? extends SuperHearing> audioBoys;
  // But you can't do this:
  // List<? extends SuperHearing & SuperSmell> dogBoys;
  }
} ///:~
```
注意，通配符〔我们下面将要学习)被限制为单一边界。