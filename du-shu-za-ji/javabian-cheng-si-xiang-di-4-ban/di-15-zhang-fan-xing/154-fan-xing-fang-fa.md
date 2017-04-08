## 15.4 泛型方法
到目前为止，我们看到的泛型，都是应用于整个类上。但同样可以在类中包含参数化方法，而这个方法所在的类可以是泛型类，也可以不是泛型类。也就是说，是否拥有泛型方法，与其所在的类是否是泛型没有关系。
泛型方法使得该方法能够独立于类而产生变化。以下是一个基本的指导原则无论何时，只要你能做到，你就应该尽最使用泛型方法。也就是说，如果使用泛型方法可以取代将整个类泛型化，那么就应该只使用泛型方法，因为它可以使事情更清楚明白。另外，对于一个static的方法而言，无法访向泛型类的类型参数，所以，如果static方法需要使用泛型能力，就必须使其成为泛型方法。
要定义泛型方法，只需将泛型参数列表置于返回值之前，就像下面这样:
```java
public class GenericMethods {
  public <T> void f(T x) {
  System.out.println(x.getClass().getName());
  }
  public static void main(String[] args) {
  GenericMethods gm = new GenericMethods();
  gm.f("");
  gm.f(1);
  gm.f(1.0);
  gm.f(1.0F);
  gm.f('c');
  gm.f(gm);
  }
} /* Output:
java.lang.String
java.lang.Integer
java.lang.Double
java.lang.Float
java.lang.Character
GenericMethods
*///:~
```
GenericMethods并不是参数化的，尽管这个类和其内部的方法可以被同时参数化，但是在这个例子中，只有方法f()拥有类型参数。这是由该方法的返回类型前面的类型参数列表指明的。
注意，当使用泛型类时，必须在创建对象的时候指定类型参数的值，而使用泛型方法的时候，通常不必指明参数类型，因为编译器会为我们找出具体的类型。这称为类型参数推断(type argumeut inference)。因此，我们可以像调用普通方法一样调用f(),而且就好像是f()被无限次地重载过。它甚至可以接受GenericMethods作为其类型参数。
如果调用f()时传人基本类型，自动打包机制就会介入其中，将基本类型的值包装为对应的对象。事实上，泛型方法与自动打包避免了许多以前我们不得不自己编写出来的代码。
### 15.4.1杠杆利用类型参数推断
人们对泛型有一个抱怨，使用泛型有时候需要向程序中加人更多的代码。如果要创建一个持有List的Map,就要像下面这样:
```
  Map<Person, List<? extends Pet>> petPeople =
  new HashMap<Person, List<? extends Pet>>}();
```
(本章稍后会介绍表达式中问号与extends的用法。)看到了吧，你在重复自己做过的事情，编译器本来应该能够从泛型参数列表中的一个参数推断出另一个参数。唉，可惜的是，编译器暂时还做不到。然而，在泛型方法中，类型参数推断可以为我们简化一部分工作。例如，我们可以编写一个工目类，它包含各种各样的static方法，专门用来创建各种常用的容器对象：
```java
public class New {
  public static <K,V> Map<K,V> map() {
  return new HashMap<K,V>();
  }
  public static <T> List<T> list() {
  return new ArrayList<T>();
  }
  public static <T> LinkedList<T> lList() {
  return new LinkedList<T>();
  }
  public static <T> Set<T> set() {
  return new HashSet<T>();
  }
  public static <T> Queue<T> queue() {
  return new LinkedList<T>();
  }
  // Examples:
  public static void main(String[] args) {
  Map<String, List<String>> sls = New.map();
  List<String> ls = New.list();
  LinkedList<String> lls = New.lList();
  Set<String> ss = New.set();
  Queue<String> qs = New.queue();
  }
} ///:~
```
main()方法演示了如何使用这个工具类，类型参数推断避免了重复的泛型参数列表。它同样可以应用于holding/MapOfList.java：
```java
//: generics/SimplerPets.java
import typeinfo.pets.*;
import java.util.*;
import net.mindview.util.*;
public class SimplerPets {
  public static void main(String[] args) {
  Map<Person, List<? extends Pet>> petPeople = New.map();
  // Rest of the code is the same...
  }
} ///:~
```
> 对于类型参数推断而言，这是一个有趣的例子。不过，很难说它为我们带来了多少好处。
类型推断只对赋值操作有效，其他时候并不起作用。如果你将一个泛型方法调用的结果(例如New.map())作为参数，传递给另一个方法，这时编译器井不会执行类型推断。在这种情况下，编译器认为：调用泛型方法后，其返回值被赋给一个Object类型的变量。下面的例子证明了这一点：

```java
public class LimitsOfInference {
  static void
  f(Map<Person, List<? extends Pet>> petPeople) {}
  public static void main(String[] args) {
  // f(New.map()); // Does not compile
  }
} ///:~
```
- 显式的类型说明
在泛型方法中，可以显式地指明类型，不过这种语法很少使用。要显式地指明类型，必须在点操作符与方法名之间擂人尖括号，然后把A型置于尖括号内。如果是在定义该方法的类的内部，必须在点操作符之前使用this关键字，如果是使用static的方法，必须在点操作符之前加上类名。使用这种语法，可以解决LimitsOfInference.java中的问题：
```java
public class ExplicitTypeSpecification {
  static void f(Map<Person, List<Pet>> petPeople) {}
  public static void main(String[] args) {
  f(New.<Person, List<Pet>>map());
  }
} ///:~
```
当然，这种语法抵消了New类为我们带来的好处(即省去了大量的类型说明)，不过，只有在编写非赋值语句时，我们才需要这样的额外说明。
### 15.4.2 可变参数与泛型方法
泛型方法与可变参数列表能够很好地共存：
```java
public class GenericVarargs {
  public static <T> List<T> makeList(T... args) {
  List<T> result = new ArrayList<T>();
  for(T item : args)
  result.add(item);
  return result;
  }
  public static void main(String[] args) {
  List<String> ls = makeList("A");
  System.out.println(ls);
  ls = makeList("A", "B", "C");
  System.out.println(ls);
  ls = makeList("ABCDEFFHIJKLMNOPQRSTUVWXYZ".split(""));
  System.out.println(ls);
  }
} /* Output:
[A]
[A, B, C]
[, A, B, C, D, E, F, F, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z]
*///:~
```
makeList()方法展示了与标淮类库中java.util.Arrays.asList()方法相同的功能。
### 15.4.3 用于Generator的泛型方法
利用生成器，我们可以很方便地填充一个Collection,而泛型化这种操作是具有实际意义的:
```java
public class Generators {
  public static <T> Collection<T>
  fill(Collection<T> coll, Generator<T> gen, int n) {
  for(int i = 0; i < n; i++)
  coll.add(gen.next());
  return coll;
  }
  public static void main(String[] args) {
  Collection<Coffee> coffee = fill(
  new ArrayList<Coffee>(), new CoffeeGenerator(), 4);
  for(Coffee c : coffee)
  System.out.println(c);
  Collection<Integer> fnumbers = fill(
  new ArrayList<Integer>(), new Fibonacci(), 12);
  for(int i : fnumbers)
  System.out.print(i + ", ");
  }
} /* Output:
Americano 0
Latte 1
Americano 2
Mocha 3
1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144,
*///:~
```
> 请注意，fill()方法是如何透明地应用于Coffee和lnteger的容器和生成器。

### 15.4.4 一个通用的Generator
下面的程序可以为任何类构造一个Generator，只要该类具有默认的构造器。为了减少类型声明，它提供了一个泛型方法。用以生成BasicGenerator：
```java
public class BasicGenerator<T> implements Generator<T> {
  private Class<T> type;
  public BasicGenerator(Class<T> type){ this.type = type; }
  public T next() {
  try {
  // Assumes type is a public class:
  return type.newInstance();
  } catch(Exception e) {
  throw new RuntimeException(e);
  }
  }
  // Produce a Default generator given a type token:
  public static <T> Generator<T> create(Class<T> type) {
  return new BasicGenerator<T>(type);
  }
}
```
> 这个类提供了一个基本实现，用以生成某个类的对象。这个类必需具备两个特点:
(1)它必须声明为public,(因为BasicGenerator与要处理的类在不同的包中，所以该类必须声明为public，并且不只具有包内访问权限.)
(2)它必须具备默认的构造器（无参数的构造器）。要创建这样的BasicGenerator对象，只需调用create()方法，并传入想要生成的类型。泛型化的create()方法允许执行BasicGenerator.create(MyType.class),而不必执行麻烦的new BasicGenerator<MyType>(MyType.class)。
例如，下面是一个具有默认构造器的简单的类：

```java
//: generics/CountedObject.java
public class CountedObject {
  private static long counter = 0;
  private final long id = counter++;
  public long id() { return id; }
  public String toString() { return "CountedObject " + id;}
} ///:~
```
使用BasicGenerator，你可以很容易地为CountedObject创建一个Generator:
```java
//: generics/BasicGeneratorDemo.java
import net.mindview.util.*;
public class BasicGeneratorDemo {
  public static void main(String[] args) {
  Generator<CountedObject> gen =
  BasicGenerator.create(CountedObject.class);
  for(int i = 0; i < 5; i++)
  System.out.println(gen.next());
  }
} /* Output:
CountedObject 0
CountedObject 1
CountedObject 2
CountedObject 3
CountedObject 4
*///:~
```
可以看到，使用泛型方法创建Generator对象，大大减少了我们要编写的代码。Java泛型要求传人Class对象，以便也可以在create()方法中用它进行类型推断。
CountedObject类能够记录下它创建了多少个CountedObject实例，井通过tostring()方法告诉我们其编号。
### 15.4.5 简化元组的使用
有了类型参数推断。再加上static方法.我们可以重新编写之前看到的元组工具，使其成为更通用的工具类库。在这个类中，我们通过重载static方法创建元组:
```java
//: net/mindview/util/Tuple.java
// Tuple library using type argument inference.
package net.mindview.util;
public class Tuple {
  public static <A,B> TwoTuple<A,B> tuple(A a, B b) {
  return new TwoTuple<A,B>(a, b);
  }
  public static <A,B,C> ThreeTuple<A,B,C>
  tuple(A a, B b, C c) {
  return new ThreeTuple<A,B,C>(a, b, c);
  }
  public static <A,B,C,D> FourTuple<A,B,C,D>
  tuple(A a, B b, C c, D d) {
  return new FourTuple<A,B,C,D>(a, b, c, d);
  }
  public static <A,B,C,D,E>
  FiveTuple<A,B,C,D,E> tuple(A a, B b, C c, D d, E e) {
  return new FiveTuple<A,B,C,D,E>(a, b, c, d, e);
  }
} ///:~
```
下面是修改后的TupleTest.java，用来测试Tuple.java：
```java
//: generics/TupleTest2.java
import net.mindview.util.*;
import static net.mindview.util.Tuple.*;
public class TupleTest2 {
  static TwoTuple<String,Integer> f() {
  return tuple("hi", 47);
  }
  static TwoTuple f2() { return tuple("hi", 47); }
  static ThreeTuple<Amphibian,String,Integer> g() {
  return tuple(new Amphibian(), "hi", 47);
  }
  static
  FourTuple<Vehicle,Amphibian,String,Integer> h() {
  return tuple(new Vehicle(), new Amphibian(), "hi", 47);
  }
  static
  FiveTuple<Vehicle,Amphibian,String,Integer,Double> k() {
  return tuple(new Vehicle(), new Amphibian(),
  "hi", 47, 11.1);
  }
  public static void main(String[] args) {
  TwoTuple<String,Integer> ttsi = f();
  System.out.println(ttsi);
  System.out.println(f2());
  System.out.println(g());
  System.out.println(h());
  System.out.println(k());
  }
} /* Output: (80% match)
(hi, 47)
(hi, 47)
(Amphibian@7d772e, hi, 47)
(Vehicle@757aef, Amphibian@d9f9c3, hi, 47)
(Vehicle@1a46e30, Amphibian@3e25a5, hi, 47, 11.1)
*///:~
```
> 注意，方法f()返回一个参数化的TwoTuple对象，而f2()返回的是非参数化的TwoTuple对象。在这个例子中，编译器井没有关于f2()的警告信息，因为我们井没有将其返回值作为参数化对象使用。在某种意义上，它被“向上转型’为一个非参数化的TwoTuple.然而，如果试图将f2()的返回值转型为参数化的TwoTuple,编译器就会发出警告。

### 15.4.6 一个Set实用工具
作为泛型方法的另一个示例，我们看看如何用Set来表达数学中的关系式。通过使用泛型方法，可以很方便地做到这一点，而且可以应用于多种类型:
```java
//: net/mindview/util/Sets.java
package net.mindview.util;
import java.util.*;
public class Sets {
  public static <T> Set<T> union(Set<T> a, Set<T> b) {
  Set<T> result = new HashSet<T>(a);
  result.addAll(b);
  return result;
  }
  public static <T>
  Set<T> intersection(Set<T> a, Set<T> b) {
  Set<T> result = new HashSet<T>(a);
  result.retainAll(b);
  return result;
  }
  // Subtract subset from superset:
  public static <T> Set<T>
  difference(Set<T> superset, Set<T> subset) {
  Set<T> result = new HashSet<T>(superset);
  result.removeAll(subset);
  return result;
  }
  // Reflexive--everything not in the intersection:
  public static <T> Set<T> complement(Set<T> a, Set<T> b) {
  return difference(union(a, b), intersection(a, b));
  }
} ///:~
```
> 在前三个方法中，都将第一个参数Set复制了一份，将Set中的所有引用都存人一个新的HashSet对象中，因此，我们并未直接修改参数中的Set,返回的值是一个全新的set对象。
这四个方法表达了如下的数学集合操作:union()返回一个Set,它将两个参数合并在一起；intersection()返回的Set只包含两个参数共有的部分。difference()方法从superset中移除subset包含的元素；complement()返回的Set包含除了交集之外的所有元素。下面提供了一个enum，它包
含各种水彩画的颜色。我们将用它来演示以上这些方法的功能和效果。

```java
//: generics/watercolors/Watercolors.java
package generics.watercolors;
public enum Watercolors {
  ZINC, LEMON_YELLOW, MEDIUM_YELLOW, DEEP_YELLOW, ORANGE,
  BRILLIANT_RED, CRIMSON, MAGENTA, ROSE_MADDER, VIOLET,
  CERULEAN_BLUE_HUE, PHTHALO_BLUE, ULTRAMARINE,
  COBALT_BLUE_HUE, PERMANENT_GREEN, VIRIDIAN_HUE,
  SAP_GREEN, YELLOW_OCHRE, BURNT_SIENNA, RAW_UMBER,
  BURNT_UMBER, PAYNES_GRAY, IVORY_BLACK
} ///:~
```
为了方便起见(可以直接使用enum中的元素名)，下面的示例以static的方式引人Watercolors，这是Java SE5中的新工具，用来从enum直接创建Set。在这里，我们向static方法EnumSet.range()传入某个范围的第一个元素与最后一个元素，然后它将返回一个Set，其中包含该范围内的所有元素：
```java
//: generics/WatercolorSets.java
import generics.watercolors.*;
import java.util.*;
import static net.mindview.util.Print.*;
import static net.mindview.util.Sets.*;
import static generics.watercolors.Watercolors.*;
public class WatercolorSets {
  public static void main(String[] args) {
  Set<Watercolors> set1 =
  EnumSet.range(BRILLIANT_RED, VIRIDIAN_HUE);
  Set<Watercolors> set2 =
  EnumSet.range(CERULEAN_BLUE_HUE, BURNT_UMBER);
  print("set1: " + set1);
  print("set2: " + set2);
  print("union(set1, set2): " + union(set1, set2));
  Set<Watercolors> subset = intersection(set1, set2);
  print("intersection(set1, set2): " + subset);
  print("difference(set1, subset): " +
  difference(set1, subset));
  print("difference(set2, subset): " +
  difference(set2, subset));
  print("complement(set1, set2): " +
  complement(set1, set2));
  }
} /* Output: (Sample)
set1: [BRILLIANT_RED, CRIMSON, MAGENTA, ROSE_MADDER, VIOLET, CERULEAN_BLUE_HUE, PHTHALO_BLUE, ULTRAMARINE, COBALT_BLUE_HUE, PERMANENT_GREEN, VIRIDIAN_HUE]
set2: [CERULEAN_BLUE_HUE, PHTHALO_BLUE, ULTRAMARINE, COBALT_BLUE_HUE, PERMANENT_GREEN, VIRIDIAN_HUE, SAP_GREEN, YELLOW_OCHRE, BURNT_SIENNA, RAW_UMBER, BURNT_UMBER]
union(set1, set2): [SAP_GREEN, ROSE_MADDER, YELLOW_OCHRE, PERMANENT_GREEN, BURNT_UMBER, COBALT_BLUE_HUE, VIOLET, BRILLIANT_RED, RAW_UMBER, ULTRAMARINE, BURNT_SIENNA, CRIMSON, CERULEAN_BLUE_HUE, PHTHALO_BLUE, MAGENTA, VIRIDIAN_HUE]
intersection(set1, set2): [ULTRAMARINE, PERMANENT_GREEN, COBALT_BLUE_HUE, PHTHALO_BLUE, CERULEAN_BLUE_HUE, VIRIDIAN_HUE]
difference(set1, subset): [ROSE_MADDER, CRIMSON, VIOLET, MAGENTA, BRILLIANT_RED]
difference(set2, subset): [RAW_UMBER, SAP_GREEN, YELLOW_OCHRE, BURNT_SIENNA, BURNT_UMBER]
complement(set1, set2): [SAP_GREEN, ROSE_MADDER, YELLOW_OCHRE, BURNT_UMBER, VIOLET, BRILLIANT_RED, RAW_UMBER, BURNT_SIENNA, CRIMSON, MAGENTA]
*///:~
```
我们可以从输出中看到各种关系运算的结果。
下面的示例使用Sets.difference()打印出java.util包中各种Collection类与Map类之间的方法差异：
```java
//: net/mindview/util/ContainerMethodDifferences.java
package net.mindview.util;
import java.lang.reflect.*;
import java.util.*;
public class ContainerMethodDifferences {
  static Set<String> methodSet(Class<?> type) {
  Set<String> result = new TreeSet<String>();
  for(Method m : type.getMethods())
  result.add(m.getName());
  return result;
  }
  static void interfaces(Class<?> type) {
  System.out.print("Interfaces in " +
  type.getSimpleName() + ": ");
  List<String> result = new ArrayList<String>();
  for(Class<?> c : type.getInterfaces())
  result.add(c.getSimpleName());
  System.out.println(result);
  }
  static Set<String> object = methodSet(Object.class);
  static { object.add("clone"); }
  static void
  difference(Class<?> superset, Class<?> subset) {
  System.out.print(superset.getSimpleName() +
  " extends " + subset.getSimpleName() + ", adds: ");
  Set<String> comp = Sets.difference(
  methodSet(superset), methodSet(subset));
  comp.removeAll(object); // Don't show 'Object' methods
  System.out.println(comp);
  interfaces(superset);
  }
  public static void main(String[] args) {
  System.out.println("Collection: " +
  methodSet(Collection.class));
  interfaces(Collection.class);
  difference(Set.class, Collection.class);
  difference(HashSet.class, Set.class);
  difference(LinkedHashSet.class, HashSet.class);
  difference(TreeSet.class, Set.class);
  difference(List.class, Collection.class);
  difference(ArrayList.class, List.class);
  difference(LinkedList.class, List.class);
  difference(Queue.class, Collection.class);
  difference(PriorityQueue.class, Queue.class);
  System.out.println("Map: " + methodSet(Map.class));
  difference(HashMap.class, Map.class);
  difference(LinkedHashMap.class, HashMap.class);
  difference(SortedMap.class, Map.class);
  difference(TreeMap.class, Map.class);
  }
} ///:~
```
在第11章的“总结”中，我们使用了这个程序的输出结果。