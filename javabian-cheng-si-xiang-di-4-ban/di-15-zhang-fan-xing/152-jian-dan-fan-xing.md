## 15.2 简单泛型
有许多原因促成了泛型的出现，而最引人往目的一个原因，就是为了创造容器类。容器，就是存放要使用的对象的地方.数组也是如此，不过与简单的数组相比，容器类更加灵活，具备更多不同的功能。事实上，所有的程序，在运行时都要求你持有一大堆对象，所以，容器类算得上最具重用性的类库之一。
我们先来看看一个只能持有单个对象的类。当然了，这个类可以明确指定其持有的对象的类型：
```java
//: generics/Holder1.java
class Automobile {}
public class Holder1 {
  private Automobile a;
  public Holder1(Automobile a) { this.a = a; }
  Automobile get() { return a; }
} ///:~
```
不过，这个类的可重用性就不怎么样了，它无法特有其他类型的任何对象。我们可不希望为碰到的每个类型都编写一个新的类。
在Javaa SE5之前，我们可以让这个类直接持有Object类型的对象：
```java
//: generics/Holder2.java
public class Holder2 {
  private Object a;
  public Holder2(Object a) { this.a = a; }
  public void set(Object a) { this.a = a; }
  public Object get() { return a; }
  public static void main(String[] args) {
  Holder2 h2 = new Holder2(new Automobile());
  Automobile a = (Automobile)h2.get();
  h2.set("Not an Automobile");
  String s = (String)h2.get();
  h2.set(1); // Autoboxes to Integer
  Integer x = (Integer)h2.get();
  }
} ///:~
```
现在，Holder2可以存储任何类型的对象，在这个例子中，只用了一个Hlolder2对象，却先后三次存储了三种不同类型的对象。
有些情况下，我们确实希望容器能够同时持有多种类型的对象。但是，通常而言，我们只会使用容器来存储一种类型的对象。泛型的主要目的之一就是用来指定容器要持有什么类型的对象。而且由编译器来保证类型的正确性。
因此，与其使用Object，我们更喜欢暂时不指定类型，而是稍后再决定具体使用什么类型。要达到这个目的，需要使用类型参数，用尖括号括住，放在类名后面。然后在使用这个类的时候，再用实际的类型替换此类型参数。在下面的例子中，T就是类型参数：
```java
//: generics/Holder3.java
public class Holder3<T> {
  private T a;
  public Holder3(T a) { this.a = a; }
  public void set(T a) { this.a = a; }
  public T get() { return a; }
  public static void main(String[] args) {
  Holder3<Automobile> h3 =
  new Holder3<Automobile>(new Automobile());
  Automobile a = h3.get(); // No cast needed
  // h3.set("Not an Automobile"); // Error
  // h3.set(1); // Error
  }
} ///:~
```
现在，当你创建Holder3对象时，必须指明想持有什么类型的对象，将其置于尖括号内。就像main()中那样。然后，你就只能在Holder3中存入该类型(或其子类，因为多态与泛型不冲突)的对象了。并且，在你从Holder3中取出它持有的对象时，自动地就是正确的类型。
这就是Java泛型的核心概念：告诉编译器想使用什么类型，然后编译器帮你处理一切细节。
一般而言，你可以认为泛型与其他的类型差不多，只不过它们碰巧有类型参数罢了。稍后我们会看到，在使用泛型时，我们只需指定它们的名称以及类型参数列表即可。
### 15.2.1 一个元组类库
仅一次方法调用就能返回多个对象，你应该经常需要这样的功能吧。可是return语句只允许返回单个对象，因此，解决办法就是创建一个对象，用它来持有想要返回的多个对象。当然，可以在每次需要的时候，专门创建一个类来完成这样的工作。可是有了泛型。我们就能够一次性地解决该问题，以后再也不用在这个问题上浪费时间了。同时，我们在编译期就能确保类型安全。
这个概念称为元组(tuple),它是将一组对象直接打包存储干其中的一个单一对象。这个容器对象允许谈取其中元素，但是不允许向其中存放新的对象。(这个概念也称为数据传这对象或信使。)
通常，元组可以具有任意长度，同时，元组中的对象可以是任意不同的类型。不过，我们希望能够为每一个对象指明其类型，并且从容器中读取出来时，能够得到正确的类型。要处理不同长度的问题，我们需要创建多个不同的元组。下面的程序是一个2维元组，它能够持有两个对象：
```java
//: net/mindview/util/TwoTuple.java
package net.mindview.util;
public class TwoTuple<A,B> {
  public final A first;
  public final B second;
  public TwoTuple(A a, B b) { first = a; second = b; }
  public String toString() {
  return "(" + first + ", " + second + ")";
  }
} ///:~
```
构造器捕获了要存储的对象，而toString()是一个便利函数，用来显示列表中的值。注意，元组隐含地保持了其中元素的次序。
第一次阅读上面的代码时，你也许会想，这不是违反了Java编程的安全性原则吗？first和second应该声明为private，然后提供getFirst()和getSecond()之类的访问方法才对呀？让我们仔细看看这个例子中的安全性：客户端程序可以读取first和second对象，然后可以随心所欲地使用这两个对象。但是，它们却无法将其他值赋予first或second。因为final声明为你买了相同的安全保险，而且这种格式更简洁明了。
还有另一种设计考虑，即你确实希望允许客户端程序员改变first办second所引用的对象。然而，采用以上的形式无疑是更安全的做法，这样的话，如果程序员想要使用具有不同元素的元组，就强制要求他们另外创建一个新的TwoTuple对象。
我们可以利用继承机制实现长度更长的元组。从下面的例子中可以看到，增加类型参数是件很简单的事情：
```java
//: net/mindview/util/ThreeTuple.java
package net.mindview.util;
public class ThreeTuple<A,B,C> extends TwoTuple<A,B> {
  public final C third;
  public ThreeTuple(A a, B b, C c) {
  super(a, b);
  third = c;
  }
  public String toString() {
  return "(" + first + ", " + second + ", " + third +")";
  }
} ///:~
```
为了使用元组，你只需定义一个长度适合的元组，将其作为方法的返回值，然后在return语句中创建该元组，井返回即可。
```java
//: generics/TupleTest.java
import net.mindview.util.*;
class Amphibian {}
class Vehicle {}
public class TupleTest {
  static TwoTuple<String,Integer> f() {
  // Autoboxing converts the int to Integer:
  return new TwoTuple<String,Integer>("hi", 47);
  }
  static ThreeTuple<Amphibian,String,Integer> g() {
  return new ThreeTuple<Amphibian, String, Integer>(
  new Amphibian(), "hi", 47);
  }
  static
  FourTuple<Vehicle,Amphibian,String,Integer> h() {
  return
  new FourTuple<Vehicle,Amphibian,String,Integer>(
  new Vehicle(), new Amphibian(), "hi", 47);
  }
  static
  FiveTuple<Vehicle,Amphibian,String,Integer,Double> k() {
  return new
  FiveTuple<Vehicle,Amphibian,String,Integer,Double>(
  new Vehicle(), new Amphibian(), "hi", 47, 11.1);
  }
  public static void main(String[] args) {
  TwoTuple<String,Integer> ttsi = f();
  System.out.println(ttsi);
  // ttsi.first = "there"; // Compile error: final
  System.out.println(g());
  System.out.println(h());
  System.out.println(k());
  }
} /* Output: (80% match)
(hi, 47)
(Amphibian@1f6a7b9, hi, 47)
(Vehicle@35ce36, Amphibian@757aef, hi, 47)
(Vehicle@9cab16, Amphibian@1a46e30, hi, 47, 11.1)
*///:~
```
由于有了泛型，你可以很容易地创建元组，令其返回一组任意类型的对象。而你所要做的，只是编写表达式而已。
通过ttsi.first=“there”语句的错误，我们可以看出，final明确实能够保护public元素，在对象被构造出来之后，声明为final的元素便不能被再赋予其他值了。
在上面的程序中，new表达式确实有点罗嗦。本章稍后会介绍，如何利用泛型方法简化这样的表达式。
### 15.2.2 一个堆栈类
接下来我们看一个稍微复杂一点的例子：传统的下推堆栈。在第11章中，我们看到，这个堆栈是作为net.mindview.util.Stack类，用一个LinkedList实现的。在那个例子中，LinkedList本身已经具备了创建堆栈所必需的方法，而Stack也可以通过两个泛型的类`Stack<T>`和`LinkedList<T>`的组合来创建。在那个示例中，我们可以看出，泛型类型也就是另一种类型罢了(稍候我们会一些例外的情况)。
现在我们不用LinkedList,来实现自己的内部链式存储机制：
```java
public class LinkedStack<T> {
  private static class Node<U> {
  U item;
  Node<U> next;
  Node() { item = null; next = null; }
  Node(U item, Node<U> next) {
  this.item = item;
  this.next = next;
  }
  boolean end() { return item == null && next == null; }
  }
  private Node<T> top = new Node<T>(); // End sentinel
  public void push(T item) {
  top = new Node<T>(item, top);
  }
  public T pop() {
  T result = top.item;
  if(!top.end())
  top = top.next;
  return result;
  }
  public static void main(String[] args) {
  LinkedStack<String> lss = new LinkedStack<String>();
  for(String s : "Phasers on stun!".split(" "))
  lss.push(s);
  String s;
  while((s = lss.pop()) != null)
  System.out.println(s);
  }
}
/* Output:
stun!
on
Phasers
*/
```
内部类Node也是一个泛型，它拥有自己的类型参数。
> 这个例子使用了一个末端哨兵(end sentinel)来判断堆栈何时为空。这个末端哨兵是在构造LinkedStack时创建的。然后，每调用一次push()方法，就会创建一个`Node<T>`对象，并将其链接到前一个`Node<T>`对象。当你调用pop()方法时，总是返top.item,然后丢弃当前top所指的`Node<T>`，并将top转移到下一个`Node<T>`,除非你已经碰到了末端哨兵，这时候就不再移动top了。如果已经到了末端，客户端程序还继续调用pop()方法，它只能得到null，说明堆栈已经空了。
### 15.2.3 RandomList
作为容器的另一个例子，假设我们需要一个持有特定类型对象的列表，每次调用其上的select()方法时，它可以随机地选取一个元素。如果我们希望以此构建一个可以应用干各种类型的对象的工具，就需要使用泛型：
```java
//: generics/RandomList.java
import java.util.*;
public class RandomList<T> {
  private ArrayList<T> storage = new ArrayList<T>();
  private Random rand = new Random(47);
  public void add(T item) { storage.add(item); }
  public T select() {
  return storage.get(rand.nextInt(storage.size()));
  }
  public static void main(String[] args) {
  RandomList<String> rs = new RandomList<String>();
  for(String s: ("The quick brown fox jumped over " +
  "the lazy brown dog").split(" "))
  rs.add(s);
  for(int i = 0; i < 11; i++)
  System.out.print(rs.select() + " ");
  }
} /* Output:
brown over fox quick quick dog brown The brown lazy brown
*///:~
```