##15.10通配符
你已经在第11章中看到了一些使用通配符的示例——在泛型参数表达式中的问号，在第14章中这种示例更多。本节将更深人地探讨这个问题。
我们开始入手的示例要展示数组的一种特殊行为：可以向导出类型的数组赋予基类型的数组引用：
```java
//: generics/CovariantArrays.java
class Fruit {}
class Apple extends Fruit {}
class Jonathan extends Apple {}
class Orange extends Fruit {}
public class CovariantArrays {
  public static void main(String[] args) {
  Fruit[] fruit = new Apple[10];
  fruit[0] = new Apple(); // OK
  fruit[1] = new Jonathan(); // OK
  // Runtime type is Apple[], not Fruit[] or Orange[]:
  try {
  // Compiler allows you to add Fruit:
  fruit[0] = new Fruit(); // ArrayStoreException
  } catch(Exception e) { System.out.println(e); }
  try {
  // Compiler allows you to add Oranges:
  fruit[0] = new Orange(); // ArrayStoreException
  } catch(Exception e) { System.out.println(e); }
  }
} /* Output:
java.lang.ArrayStoreException: Fruit
java.lang.ArrayStoreException: Orange
*///:~
```
main()中的第一行创建了一个Apple数组，井将其赋值给一个Fruit数组引用。这是有意义的，因为Apple也是一种Fruit，因此Apple数组应该也是一个Fruit数组。
但是，如果实际的数组类型是Apple[]，你应该只能在其中放置Apple或Apple的子类型，这在编译期和运行时都可以工作。但是请注意，编译器允许你将Fruit放置到这个数组中，这对于编译器来说是有意义的，因为它有一个Fruit[]引用——它有什么理由不允许将Fruit对象或名任何从Fruit继承出来的对象(例如Orange)，放置到这个数组中呢？因此，在编译期，这是允许的。但是，运行时的数组机制知道它处理的是Apple[]，因此会在向数组中放置异构类型时抛出异常。
实际上，向上转型不合适用在这里。你真正做的是将一个数组赋值给另一个数组。数组的行为应该是它可以持有其他对象，这里只是因为我们能够向上转型而已，所以很明显，数组对象可以保留有关它们包含的对象类型的规则。就好像数组对它们持有的对象是有意识的，因此在编译期检查和运行时检查之间，你不能滥用它们。
对数组的这种赋值并不是那么可怕，因为在运行时可以发现你已经插入了不正确的类型。但是泛型的主要目标之一是将这种错误检测移入到编译期。因此当我们试图使用泛型容器来代替数组时，会发生什么呢？
```java
//: generics/NonCovariantGenerics.java
// {CompileTimeError} (Won't compile)
import java.util.*;
public class NonCovariantGenerics {
  // Compile Error: incompatible types:
  List<Fruit> flist = new ArrayList<Apple>();
} ///:~
```
尽管你在第一次阅读这段代码时会认为：“不能将一个Apple容器赋值给一个Fruit容器”。别忘了，泛型不仅和容器相关。正确的说法是：“不能把一个涉及Apple的泛型赋给一个涉及Fruit的泛型”。如果就像在数组的情况中一样，编译器对代码的了解足够多，可以确定所涉及到的容器，那么它可能会留下一些余地。但是它不知透任何有关这方面的信息，因此它拒绝向上转型。然而实际上这根本不是向上转型——Apple的List不是Fruit的List。Apple的List将持有Apple和Apple的子类型，而Fruit的List将持有任何类型的Fruit，诚然，这包括Apple在内，但是它不是一个Apple的List，它仍旧是Fruit的List。Apple的List在类型上不等价于Fruit的List，即使Apple是一种Fruit类型。
真正的问题是我们在谈论容器的类型。而不是容器持有的类型。与数组不同，泛型没有内建的协变类型。这是因为数组在语言中是完全定义的，因此可以内建了编译期和运行时的检查，但是在使用泛型时，编译器和运行时系统都不知道你想用类型做些什么，以及该采用什么样的规则。
但是，有时你想要在两个类型之间建立某种类型的向上转型关系，这正是通配符所允许的：
```java
//: generics/GenericsAndCovariance.java
import java.util.*;
public class GenericsAndCovariance {
  public static void main(String[] args) {
  // Wildcards allow covariance:
  List<? extends Fruit> flist = new ArrayList<Apple>();
  // Compile Error: can't add any type of object:
  // flist.add(new Apple());
  // flist.add(new Fruit());
  // flist.add(new Object());
  flist.add(null); // Legal but uninteresting
  // We know that it returns at least Fruit:
  Fruit f = flist.get(0);
  }
} ///:~
```
flist类型现在是`List<? extends Fruit>`，你可以将其读作“具有任何从Fruit继承的类型的列表”。但是，这实际上并不意味着这个List将持有任何类型的Fruit。通配符引用的是明确的类型，因此它意味着“某种first引用没有指定的具体类型”。因此这个被赋值的List必须持有诸如Fruit或Apple这样的某种指定类型，但是为了向上转型为flist，这个类型是什么并没有人关心。
如果唯一的限制是这个List要持有某种具体的Fruit或Fruit的子类型，但是你实际上并不关心它是什么，那么你能用这样的List做什么呢?如果不知道List持有什么类型，那么你怎样才能安全地向其中添加对象呢?就像在CovariantArrays.java中向上转型数组一样，你不能，除非编译器而不是运行时系统可以阻止这种操作的发生。你很快就会发现这一问题。
你可能会认为，事情变得有点走极端了，因为现在你甚至不能向刚刚声明过将持有Apple对象的List中放置一个Apple对象了。是的，但是编译器并不知道这一点。`List<? extends Fruit>`可以合法地指向一个`List<Orange>`。一旦执行这种类型的向上转型，你就将丢失掉向其中传递任何对象的能力，甚至是传递Object也不行。
另一方面，如果你调用一个返回Fruit的方法，则是安全的，因为你知道在这个List中的任何对象至少具有Fruit类型，因此编译器将允许这么做。
### 15.10.1 编译器有多聪明
现在，你可能会猜想自己被阻止去调用任何接受参数的方法，但是请考虑下面的程序：
```java
//: generics/CompilerIntelligence.java
import java.util.*;
public class CompilerIntelligence {
  public static void main(String[] args) {
  List<? extends Fruit> flist =
  Arrays.asList(new Apple());
  Apple a = (Apple)flist.get(0); // No warning
  flist.contains(new Apple()); // Argument is 'Object'
  flist.indexOf(new Apple()); // Argument is 'Object'
  }
} ///:~
```
你可以看到，对contains()和indexOf()的调用，这两个方法都接受Apple对象作为参数，而这些调用都可以正常执行。这是否意味着编译器实际上将检查代码，以查着是否有某个特定的方法修改了它的对象？
通过查看ArrsyList的文档，我们可以发现，编译器并没有这么聪明。尽管add()将接受一个具有泛型参数类型的参数，但是contains()和indexOf()将接受Object类型的参数。因此当你指定一个`ArrayList<? extends Fruit>`时，add()的参数就变成了“? Extends Fruit"。从这个描述中，编译器并不能了解这里需要Fruit的哪个具体子类型，因此它不会接受任何类型的Fruit。如果先将Apple向上转型为Fruit,也无关紧要——编译器将直接拒绝对参数列表中涉及通配符的方法(例如add())的调用。
在使用contains()和indexOf()时，参数类型是Object，因此不涉及任何通配符，而编译器也将允许这个调用。这意味着将由泛型类的设计者来决定哪些调用是“安全的”，并使用Object类型作为其参数类型。为了在类型中使用了通配符的情况下禁止这类调用，我们需要在参数列表中使用类型参数。
可以在一个非常简单的Holder类中看到这一点：
```java
//: generics/Holder.java
public class Holder<T> {
  private T value;
  public Holder() {}
  public Holder(T val) { value = val; }
  public void set(T val) { value = val; }
  public T get() { return value; }
  public boolean equals(Object obj) {
  return value.equals(obj);
  }
  public static void main(String[] args) {
  Holder<Apple> Apple = new Holder<Apple>(new Apple());
  Apple d = Apple.get();
  Apple.set(d);
  // Holder<Fruit> Fruit = Apple; // Cannot upcast
  Holder<? extends Fruit> fruit = Apple; // OK
  Fruit p = fruit.get();
  d = (Apple)fruit.get(); // Returns 'Object'
  try {
  Orange c = (Orange)fruit.get(); // No warning
  } catch(Exception e) { System.out.println(e); }
  // fruit.set(new Apple()); // Cannot call set()
  // fruit.set(new Fruit()); // Cannot call set()
  System.out.println(fruit.equals(d)); // OK
  }
} /* Output: (Sample)
java.lang.ClassCastException: Apple cannot be cast to Orange
true
*///:~
```
Holder有一个接受T类型对象的set()方法，一个get()方法，以及一个接受Object对象的equals()方法。正如你已经看到的，如果创建了一个`Holder<Apple>`，不能将其向上转型为`Holder<Fruit>`，但是可以将其向上转型为`Holder<? extends Fruit>`，如果调用get()，它只会返回一个Fruit——这就是在给定“任何扩展自Fruit的对象”这一边界之后，它所能知道的一切了。
如果能够了解更多的信息，那么你可以转型到某种具体的Fruit类型，而这不会导致任何警告，但是你存在着得到ClassCastException的风险。set()方法不能工作于Apple或Fruit,因为set()的参数也是"? Extends Fruit"，这意味着它可以是任何事物，而编译器无法验证“任何事物”的类型安全性。
但是，equals()方法工作良好，因为它将接受Object类型而井非T类型的参数。因此，编译器只关注传递进来和要返回的对象类型，它井不会分析代码，以查看是否执行了任何实际的写入和读取操作。
### 15.10.2 逆变
还可以走另外一条路，即使用`超类型通配符`。这里，可以声明通配符是由某个特定类的任何基类来界定的，方法是指定`<? super MyClass>`，甚至或者使用类型参数：`<? super T>`(尽管你不能对泛型参数给出一个超类型边界；即不能声明`<T super MyClass>`)。这使得你可以安全地传递一个类型对象到泛型类型中。因此，有了超类型通配符，就可以向Collection写入了:
```java
//: generics/SuperTypeWildcards.java
import java.util.*;
public class SuperTypeWildcards {
  static void writeTo(List<? super Apple> apples) {
  apples.add(new Apple());
  apples.add(new Jonathan());
  // apples.add(new Fruit()); // Error
  }
} ///:~
```
参数Apple是Apple的某种基类型的List，这样你就知道向其中添加Apple或Apple的子类型是安全的。但是，既然Apple是下界，那么你可以知道向这样的List中添加Fruit是不安全的，因为这将使这个List敞开口子，从而可以向其中添加非Apple类型的对象，而这是违反静态类型安全的。
因此你可能会根据如何能够向一个泛型类型“写人”(传递给一个方法)，以及如何能够从一个泛型类型中“读取”(从一个方法中返回)，来着手思考子类型和超类型边界。
超类型边界放松了在可以向方法传递的参数上所作的限制：
```java
//: generics/GenericWriting.java
import java.util.*;
public class GenericWriting {
  static <T> void writeExact(List<T> list, T item) {
  list.add(item);
  }
  static List<Apple> apples = new ArrayList<Apple>();
  static List<Fruit> fruit = new ArrayList<Fruit>();
  static void f1() {
  writeExact(apples, new Apple());
  // writeExact(fruit, new Apple()); // Error:
  // Incompatible types: found Fruit, required Apple
  }
  static <T> void
  writeWithWildcard(List<? super T> list, T item) {
  list.add(item);
  }
  static void f2() {
  writeWithWildcard(apples, new Apple());
  writeWithWildcard(fruit, new Apple());
  }
  public static void main(String[] args) { f1(); f2(); }
} ///:~
```
writeExact()方法使用了一个确切参数类型(无通配符)。在f1()中，可以看到这工作良好——只要你只向`List<Apple>`中放置Apple。但是，writeExact()不允许将Apple放置到`List<Fruit>`中，即使知道这应该是可以的。
在writeWithWildcard()中，其参数现在是`List<? super T>`，因此这个List将持有从T导出的某种具体类型，这样就可以安全地将一个T类型的对象或者从T导出的任何对象作为参数传递给List的方法。在f2()中可以看到这一点，在这个方法中我们仍旧可以像前面那样，将Apple放置到`List<Apple>`中，但是现在我们还可以如你所期望的那样，将Apple放置到`List<Fruit>`中。
我们可以执行下面这个相同的类型分析。作为对协变和通配符的一个复习：
```java
//: generics/GenericReading.java
import java.util.*;
public class GenericReading {
  static <T> T readExact(List<T> list) {
  return list.get(0);
  }
  static List<Apple> apples = Arrays.asList(new Apple());
  static List<Fruit> fruit = Arrays.asList(new Fruit());
  // A static method adapts to each call:
  static void f1() {
  Apple a = readExact(apples);
  Fruit f = readExact(fruit);
  f = readExact(apples);
  }
  // If, however, you have a class, then its type is
  // established when the class is instantiated:
  static class Reader<T> {
  T readExact(List<T> list) { return list.get(0); }
  }
  static void f2() {
  Reader<Fruit> fruitReader = new Reader<Fruit>();
  Fruit f = fruitReader.readExact(fruit);
  // Fruit a = fruitReader.readExact(apples); // Error:
  // readExact(List<Fruit>) cannot be
  // applied to (List<Apple>).
  }
  static class CovariantReader<T> {
  T readCovariant(List<? extends T> list) {
  return list.get(0);
  }
  }
  static void f3() {
  CovariantReader<Fruit> fruitReader =
  new CovariantReader<Fruit>();
  Fruit f = fruitReader.readCovariant(fruit);
  Fruit a = fruitReader.readCovariant(apples);
  }
  public static void main(String[] args) {
  f1(); f2(); f3();
  }
} ///:~
```
与前面一样，第一个方法readExact()使用了精确的类型。因此如果使用这个没有任何通配符的精确类型，就可以向List中写人和读取这个精确类型。另外，对于返回值，静态的泛型方法readExact()可以有效地“适应”每个方法调用，并能够从`List<Apple>`中返回一个Apple，从`List<Fruit>`中返回一个Fruit,就像在f1()中看到的那样。因此，如果可以摆脱静态泛型方法，那么当只是读取时，就不需要协变类型了。
但是，如果有一个泛型类，那么当你创建这个类的实例时，要为这个类确定参数。就像在f2()中看到的，fruitReader实例可以从`List<Fruit>`中读取一个Fruit，因为这就是它的确切类型。但是`List<Apple>`还应该产生Fruit对象，而fruitReader不允许这么做。
为了修正这个问题，CovariantReader.readCovcariant()方法将接受`List<? extends T>`，因此，从这个列表中读取一个T是安全的(你知道在这个列表中的所有对象至少是一个T，并且可能是从T导出的某种对象)。在f3()中，你可以看到现在可以从`List<Apple>`中读取Fruit了。
### 15.10.3 无界通配符
无界通配符`<?>`看起来意味着“任何事物”，因此使用无界通配符好像等价于使用原生类型。
事实上，编译器初看起来是支持这种判断的：
```java
//: generics/UnboundedWildcards1.java
import java.util.*;
public class UnboundedWildcards1 {
  static List list1;
  static List<?> list2;
  static List<? extends Object> list3;
  static void assign1(List list) {
  list1 = list;
  list2 = list;
  // list3 = list; // Warning: unchecked conversion
  // Found: List, Required: List<? extends Object>
  }
  static void assign2(List<?> list) {
  list1 = list;
  list2 = list;
  list3 = list;
  }
  static void assign3(List<? extends Object> list) {
  list1 = list;
  list2 = list;
  list3 = list;
  }
  public static void main(String[] args) {
  assign1(new ArrayList());
  assign2(new ArrayList());
  // assign3(new ArrayList()); // Warning:
  // Unchecked conversion. Found: ArrayList
  // Required: List<? extends Object>
  assign1(new ArrayList<String>());
  assign2(new ArrayList<String>());
  assign3(new ArrayList<String>());
  // Both forms are acceptable as List<?>:
  List<?> wildList = new ArrayList();
  wildList = new ArrayList<String>();
  assign1(wildList);
  assign2(wildList);
  assign3(wildList);
  }
} ///:~
```
有很多情况都和你在这里看到的情况类似，即编译器很少关心使用的是原生类型还是`<?>`。在这些情况中，`<?>`可以被认为是一种装饰，但是它仍旧是很有价值的，因为，实际上，它是在声明：“我是想用Java的泛型来编写这段代码，我在这里并不是要用原生类型，但是在当前这种情况下，泛型参数可以持有任何类型。”
第二个示例展示了无界通配符的一个重要应用。当你在处理多个泛型参数时，有时允许一个参数可以是任何类型，同时为其他参数确定某种特定类型的这种能力会显得很重要：
```java
//: generics/UnboundedWildcards2.java
import java.util.*;
public class UnboundedWildcards2 {
  static Map map1;
  static Map<?,?> map2;
  static Map<String,?> map3;
  static void assign1(Map map) { map1 = map; }
  static void assign2(Map<?,?> map) { map2 = map; }
  static void assign3(Map<String,?> map) { map3 = map; }
  public static void main(String[] args) {
  assign1(new HashMap());
  assign2(new HashMap());
  // assign3(new HashMap()); // Warning:
  // Unchecked conversion. Found: HashMap
  // Required: Map<String,?>
  assign1(new HashMap<String,Integer>());
  assign2(new HashMap<String,Integer>());
  assign3(new HashMap<String,Integer>());
  }
} ///:~
```
但是，当你拥有的全都是无界通配符时，就像在`Map<?,?>`中看到的那样，编译器看起来就无法将其与原生Map区分开了。另外，UnboundedWildcards.java展示了编译器处理`List<?>`和`List<? extends Object>`时是不同的。
令人困惑的是，编译器并非总是关注像List和`List<?>`之间的这种差异，因此它们看起来就像是相同的事物。因为，事实上，由干泛型参数将擦除到它的第一个边界，因此`List<?>`看起来等价于`List<Object>`，而List实际上也是`List<Object>`——除非这些语句都不为真。List实际上表示“持有任何Object类型的原生List”，而`List<?>`表示“具有`某种特定类型`的非原生List，只是我们不知道那种类型是什么。”
编译器何时才会关注原生类型和涉及无界通配符的类型之间的差异呢？下面的示例使用了前面定义的`Holder<T>`类，它包含接受Holder作为参数的各种方法，但是它们具有不同的形式作为原生类型，具有具体的类型参数以及具有无界通配符参数：
```java
//: generics/Wildcards.java
// Exploring the meaning of wildcards.
public class Wildcards {
  // Raw argument:
  static void rawArgs(Holder holder, Object arg) {
  // holder.set(arg); // Warning:
  // Unchecked call to set(T) as a
  // member of the raw type Holder
  // holder.set(new Wildcards()); // Same warning
  // Can't do this; don't have any 'T':
  // T t = holder.get();
  // OK, but type information has been lost:
  Object obj = holder.get();
  }
  // Similar to rawArgs(), but errors instead of warnings:
  static void unboundedArg(Holder<?> holder, Object arg) {
  // holder.set(arg); // Error:
  // set(capture of ?) in Holder<capture of ?>
  // cannot be applied to (Object)
  // holder.set(new Wildcards()); // Same error
  // Can't do this; don't have any 'T':
  // T t = holder.get();
  // OK, but type information has been lost:
  Object obj = holder.get();
  }
  static <T> T exact1(Holder<T> holder) {
  T t = holder.get();
  return t;
  }
  static <T> T exact2(Holder<T> holder, T arg) {
  holder.set(arg);
  T t = holder.get();
  return t;
  }
  static <T>
  T wildSubtype(Holder<? extends T> holder, T arg) {
  // holder.set(arg); // Error:
  // set(capture of ? extends T) in
  // Holder<capture of ? extends T>
  // cannot be applied to (T)
  T t = holder.get();
  return t;
  }
  static <T>
  void wildSupertype(Holder<? super T> holder, T arg) {
  holder.set(arg);
  // T t = holder.get(); // Error:
  // Incompatible types: found Object, required T
  // OK, but type information has been lost:
  Object obj = holder.get();
  }
  public static void main(String[] args) {
  Holder raw = new Holder<Long>();
  // Or:
  raw = new Holder();
  Holder<Long> qualified = new Holder<Long>();
  Holder<?> unbounded = new Holder<Long>();
  Holder<? extends Long> bounded = new Holder<Long>();
  Long lng = 1L;
  rawArgs(raw, lng);
  rawArgs(qualified, lng);
  rawArgs(unbounded, lng);
  rawArgs(bounded, lng);
  unboundedArg(raw, lng);
  unboundedArg(qualified, lng);
  unboundedArg(unbounded, lng);
  unboundedArg(bounded, lng);
  // Object r1 = exact1(raw); // Warnings:
  // Unchecked conversion from Holder to Holder<T>
  // Unchecked method invocation: exact1(Holder<T>)
  // is applied to (Holder)
  Long r2 = exact1(qualified);
  Object r3 = exact1(unbounded); // Must return Object
  Long r4 = exact1(bounded);
  // Long r5 = exact2(raw, lng); // Warnings:
  // Unchecked conversion from Holder to Holder<Long>
  // Unchecked method invocation: exact2(Holder<T>,T)
  // is applied to (Holder,Long)
  Long r6 = exact2(qualified, lng);
  // Long r7 = exact2(unbounded, lng); // Error:
  // exact2(Holder<T>,T) cannot be applied to
  // (Holder<capture of ?>,Long)
  // Long r8 = exact2(bounded, lng); // Error:
  // exact2(Holder<T>,T) cannot be applied
  // to (Holder<capture of ? extends Long>,Long)
  // Long r9 = wildSubtype(raw, lng); // Warnings:
  // Unchecked conversion from Holder
  // to Holder<? extends Long>
  // Unchecked method invocation:
  // wildSubtype(Holder<? extends T>,T) is
  // applied to (Holder,Long)
  Long r10 = wildSubtype(qualified, lng);
  // OK, but can only return Object:
  Object r11 = wildSubtype(unbounded, lng);
  Long r12 = wildSubtype(bounded, lng);
  // wildSupertype(raw, lng); // Warnings:
  // Unchecked conversion from Holder
  // to Holder<? super Long>
  // Unchecked method invocation:
  // wildSupertype(Holder<? super T>,T)
  // is applied to (Holder,Long)
  wildSupertype(qualified, lng);
  // wildSupertype(unbounded, lng); // Error:
  // wildSupertype(Holder<? super T>,T) cannot be
  // applied to (Holder<capture of ?>,Long)
  // wildSupertype(bounded, lng); // Error:
  // wildSupertype(Holder<? super T>,T) cannot be
  // applied to (Holder<capture of ? extends Long>,Long)
  }
} ///:~
```
在rawArgs()中，编译器知道Holder是一个泛型类型，因此即使它在这里被表示成一个原生类型，编译器仍旧知道向set()传递一个Object是不安全的。由于它是原生类型，你可以将任何类型的对象传递给set()，而这个对象将被向上转型为Object。因此，无论何时，只要使用了原生类型，都会放弃编译期检查。对get()的调用说明了相同的问题：没有任何T类型的对象，因此结果只能是一个Object。
人们很自然地会开始考虑原生Holder与`Holder<?>`是大致相同的事物。但是unboundedArg()强调它们是不同的——它揭示了相同的问题，但是它将这些问题作为错误而不是警告报告，因为原生Holder将持有任何类型的组合，而`Holder<?>`将持有具有某种具体类型的同构集合，因此不能只是向其中传递Object。
在exact1()和exact2()中，你可以看到使用了确切的泛型参数没有任何通配符。你将看到，exact2()与exact1()具有不同的限制，因为它有额外的参数。
在wildSubtype()中，在Holder类型上的限制被放松为包括持有任何扩展自T的对象的Holder。这还是意味着如果T是Fruit，那么Holder可以是`Holder<Apple>`,这是合法的。为了防止将Orange放置到`Holder<Apple>`中，对set()的调用(或者对任何接受这个类型参数为参数的方法的调用)都是不允许的。但是，你仍旧知道任何来自`Holder<? extends Fruit>`的对象至少是Fruit，因此get()(或者任何将产生具有这个类型参数的返回值的方法)都是允许的。
wildSupertype()展示了超类型通配符，这个方法展示了与wildSubtype()相反的行为：holder可以是持有任何T的基类型的容器。因此，set()可以接受T,因为任何可以工作于基类的对象都可以多态地作用于导出类(这里就是T)。但是，尝试着调用get()是没有用的，因为由holde持有的类型可以是任何超类型，因此唯一安全的类型就是Object。
这个示例还展示了对于在unbounded()中使用无界通配符能够做什么不能做什么所做出的限制。对于迁移兼容性，rawArgs()将接受所有Holder的不同变体，而不会产生警告。unbounded-Args()方法也可以接受相同的所有类型，尽管如前所述，它在方法体内部处理这些类型的方式并不相同。
如果向接受“确切”泛型类型(没有通配符)的方法传递一个原生Holder引用，就会得到一个警告，因为确切的参数期望得到在原生类型中井不存在的信息。如果向exact1()传递一个无界引用，就不会有任何可以确定返回类型的类型信息。
可以看到，exact2()具有最多的限制，因为它希望精确地得到一个`Holder<T>`，以及一个具有类型T的参数，正由于此，它将产生错误或警告，除非提供确切的参数。有时，这样做很好，但是如果它过于受限，那么就可以使用通配符，这取决于是否想要从泛型参数中返回类型确定的返回值(就像在wildSubtype()中看到的那样)，或者是否想要向泛型参数传递类型确定的参数(就像在wildSupertype()中看到的那样)。
因此，使用确切类型来替代通配符类型的好处是，可以用泛型参数来做更多的事，但是使用通配符使得你必须接受范围更宽的参数化类型作为参数。因此，必须逐个情况地权衡利弊，找到更适合你的需求的方法。
### 15.10.4 捕获转换
有一种情况特别需要使用`<?>`而不是原生类型。如果向一个使用`<?>`的方法传递原生类型，那么对编译器来说，可能会推断出实际的类型参数，使得这个方法可以回转并调用另一个使用这个确切类型的方法。下面的示例演示了这种技术，它被称为捕获转换，因为未指定的通配符类型被捕获，并被转换为确切类型。这里，有关警告的注释只有在@SuppressWarnings注解被移除之后才能起作用：
```java
//: generics/CaptureConversion.java
public class CaptureConversion {
  static <T> void f1(Holder<T> holder) {
  T t = holder.get();
  System.out.println(t.getClass().getSimpleName());
  }
  static void f2(Holder<?> holder) {
  f1(holder); // Call with captured type
  }
  @SuppressWarnings("unchecked")
  public static void main(String[] args) {
  Holder raw = new Holder<Integer>(1);
  // f1(raw); // Produces warnings
  f2(raw); // No warnings
  Holder rawBasic = new Holder();
  rawBasic.set(new Object()); // Warning
  f2(rawBasic); // No warnings
  // Upcast to Holder<?>, still figures it out:
  Holder<?> wildcarded = new Holder<Double>(1.0);
  f2(wildcarded);
  }
} /* Output:
Integer
Object
Double
*///:~
```
f1()中的类型参数都是确切的，没有通配符或边界。在f2()中，Holder()参数是一个无界通配符，因此它看起来是未知的。但是，在f2()中，f1()被调用，而f1()需要一个已知参数。这里所发生的是：参数类型在调用f2()的过程中被捕获，因此它可以在对f1()的调用中被使用。
你可能想知道，这项技术是否可以用于写入，但是这要求要在传递`Holder<?>`时同时传递一个具体类型。捕获转换只有在这样的情况下可以工作：即在方法内部，你需要使用确切的类型。注意，不能从f2()中返回T，因为T对于f2()来说是未知的。捕获转换十分有趣，但是非常受限。
## 15.11问题
本节将阐述在使用Java泛型时会出现的各类问题。
### 15.11.1任何基本类型都不能作为类型参数
正如本章早先提到过的，你将在Java泛型中发现的限制之一是，不能将基本类型用作类型参数。因此不能创建`ArrayList<int>`之类的东西。
  解决之道是使用甚本类型的包装器类以及Java SE5的自动包装机制。如果创建一个`ArrayList<Integer>`，并将基本类型int应用于这个容器，那么你将发现自动包装机制将自动地实现int到Integer的双向转换——因此，这几乎就像是有一个`ArrayList<int>`一样：
```java
//: generics/ListOfInt.java
// Autoboxing compensates for the inability to use
// primitives in generics.
import java.util.*;
public class ListOfInt {
  public static void main(String[] args) {
  List<Integer> li = new ArrayList<Integer>();
  for(int i = 0; i < 5; i++)
  li.add(i);
  for(int i : li)
  System.out.print(i + " ");
  }
} /* Output:
0 1 2 3 4
*///:~
```
注意，自动包装机制甚至允许用foreach语法来产生int。
通常，这种解决方案工作得很好—能够成功地存储和读取int，有一些转换碰巧在发生的同时会对你屏蔽掉。但是，如果性能成为了问题，就需要使用专门适配基本类型的容器版本。
```java
//: generics/ByteSet.java
import java.util.*;
public class ByteSet {
  Byte[] possibles = { 1,2,3,4,5,6,7,8,9 };
  Set<Byte> mySet =
  new HashSet<Byte>(Arrays.asList(possibles));
  // But you can't do this:
  // Set<Byte> mySet2 = new HashSet<Byte>(
  // Arrays.<Byte>asList(1,2,3,4,5,6,7,8,9));
} ///:~
```
注意，自动包装机制解决了一些问题，但并不是解决了所有问题。下面的示例展示了一个泛型的Generator接口，它指定next()方法返回一个具有其参数类型的对象。FArray类包含一个泛型方法，它通过使用生成器在数组中填充对象(这使得类泛型在本例中无法工作，因为这个方法是静态的)。Generator实现来自第16章，并且在main()中，可以看到FArray.fill()使用它在数组中填充对象：
```java
//: generics/PrimitiveGenericTest.java
import net.mindview.util.*;
// Fill an array using a generator:
class FArray {
  public static <T> T[] fill(T[] a, Generator<T> gen) {
  for(int i = 0; i < a.length; i++)
  a[i] = gen.next();
  return a;
  }
}
public class PrimitiveGenericTest {
  public static void main(String[] args) {
  String[] strings = FArray.fill(
  new String[7], new RandomGenerator.String(10));
  for(String s : strings)
  System.out.println(s);
  Integer[] integers = FArray.fill(
  new Integer[7], new RandomGenerator.Integer());
  for(int i: integers)
  System.out.println(i);
  // Autoboxing won't save you here. This won't compile:
  // int[] b =
  // FArray.fill(new int[7], new RandIntGenerator());
  }
} /* Output:
YNzbrnyGcF
OWZnTcQrGs
eGZMmJMRoE
suEcUOneOE
dLsmwHLGEa
hKcxrEqUCB
bkInaMesbt
7052
6665
2654
3909
5202
2209
5458
*///:~
```
由于RandomGenerator.Integer实现了`Generator<Integer>`，所以我的希望是自动包装机制可以自动地将next()的值从Integer转换为int。但是，自动包装机制不能应用于数组，因此这无法工作。
### 15.11.2实现参数化接口
一个类不能实现同一个泛型接口的两种变体，由于擦除的原因，这两个变体会成为相同的接口。下面是产生这种冲突的情况:
```java
//: generics/MultipleInterfaceVariants.java
// {CompileTimeError} (Won't compile)
interface Payable<T> {}
class Employee implements Payable<Employee> {}
class Hourly extends Employee
  implements Payable<Hourly> {} ///:~
```
Hourly不能编译，因为擦除会将`Payable<Employee>`和`Payable<Hourly>`简化为相同的类Payable，这样，上面的代码就意味着在重复两次地实现相同的接口。十分有趣的是，如果从Payable的两种用法中都移除掉泛型参数〔就像编译器在擦出阶段所做的那样)这段代码就可以编译。
在使用某些更基本的Java接口，例如Comparable<T>时，这个问题可能会变得十分令人恼火，就像你在本节稍后就会看到的那样。