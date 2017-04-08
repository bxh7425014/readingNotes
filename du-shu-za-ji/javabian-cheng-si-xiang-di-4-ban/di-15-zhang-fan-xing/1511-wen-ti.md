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
### 15.11.3转型和警告
使用带有泛型类型参数的转型或instanceof不会有任何效果。下面的容器在内部将各个值存储为Object，并在获取这些值时，耳将它们转型回T：
```java
//: generics/GenericCast.java
class FixedSizeStack<T> {
  private int index = 0;
  private Object[] storage;
  public FixedSizeStack(int size) {
  storage = new Object[size];
  }
  public void push(T item) { storage[index++] = item; }
  @SuppressWarnings("unchecked")
  public T pop() { return (T)storage[--index]; }
}
public class GenericCast {
  public static final int SIZE = 10;
  public static void main(String[] args) {
  FixedSizeStack<String> strings =
  new FixedSizeStack<String>(SIZE);
  for(String s : "A B C D E F G H I J".split(" "))
  strings.push(s);
  for(int i = 0; i < SIZE; i++) {
  String s = strings.pop();
  System.out.print(s + " ");
  }
  }
} /* Output:
J I H G F E D C B A
*///:~
```
如果没有@SuppressWarnings注解，编译器将对pop()产生“unchecked cast'警告。由于擦除的原因，编译器无法知道这个转型是否是安全的，并且pop()方法实际上并没有执行任何转型。这是因为，T被擦除到它的第一个边界，默认情况下是Object, 因此pop()实际上只是将Object转型为Object。
有时，泛型没有消除对转型的需要，这就会由编译器产生警告，而这个警告是不恰当的。例如：
```java
//: generics/NeedCasting.java
import java.io.*;
import java.util.*;
public class NeedCasting {
  @SuppressWarnings("unchecked")
  public void f(String[] args) throws Exception {
  ObjectInputStream in = new ObjectInputStream(
  new FileInputStream(args[0]));
  List<Widget> shapes = (List<Widget>)in.readObject();
  }
} ///:~
```
正如你将在下一章学到的那样，readObject()无法知道它正在读取的是什么，因此它返的是必须转型的对象。但是当注释掉@SuppressWarnings注解，并编译这个程序时，就会得到下面的警告：
```
Note: NeedCasting.java uses unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
```
如果遵循这条指示，使用-Xliint: unchecked来重新编译：
```
NeedCasting.java:12: warning: [unchecked] unchecked cast found:java.lang.Object
required: java.util.List<Widget>
  List<Shape> shapes = (List<Widget>)in.readObjlect():
```
你会被强制要求转型，但是又被告知不应该转型。为了解决这个问题，必须使用在Java SE5中引入的新的转型形式，既通过泛型类来转型：
```java
//: generics/ClassCasting.java
import java.io.*;
import java.util.*;
public class ClassCasting {
  @SuppressWarnings("unchecked")
  public void f(String[] args) throws Exception {
  ObjectInputStream in = new ObjectInputStream(
  new FileInputStream(args[0]));
  // Won't Compile:
// List<Widget> lw1 =
// List<Widget>.class.cast(in.readObject());
  List<Widget> lw2 = List.class.cast(in.readObject());
  }
} ///:~
```
但是，不能转型到实际类型`(List<Widget>)`。也就是说，不能声明：
```java
  List<Widget>.class.cast(in.readObject())
```
甚至当你添加一个像下面这样的另一个转型时：
```java
  (List<Widget>)List.class.cast(in.readObject())
```
仍旧会得到一个警告。
### 15.11.4重裁
下面的程序是不能编译的，即使编译它是一种合理的尝试：
```java
//: generics/UseList.java
// {CompileTimeError} (Won't compile)
import java.util.*;
public class UseList<W,T> {
  void f(List<T> v) {}
  void f(List<W> v) {}
} ///:~
```
由于擦除的原因，重载方法将产生相同的类型签名。
与此不同的是，当被擦除的参数不能产生唯一的参数列表时。必须提供明显有区别的方法名：
```java
//: generics/UseList2.java
import java.util.*;
public class UseList2<W,T> {
  void f1(List<T> v) {}
  void f2(List<W> v) {}
} ///:~
```
幸运的是，这类问题可以由编译器探测到。
### 15.11.5 基类劫持了接口
假设你有一个Pet类，它可以与其他的Pet对象进行比较(实现了Comparable接口)：
```java
//: generics/ComparablePet.java
public class ComparablePet
implements Comparable<ComparablePet> {
  public int compareTo(ComparablePet arg) { return 0; }
} ///:~
```
对可以与ComparablePet的子类比较的类型进行窄化是有意义的，例如，一个Cat对象就只能与其他的Cat对象比较：
```java
//: generics/HijackedInterface.java
// {CompileTimeError} (Won't compile)
class Cat extends ComparablePet implements Comparable<Cat>{
  // Error: Comparable cannot be inherited with
  // different arguments: <Cat> and <Pet>
  public int compareTo(Cat arg) { return 0; }
} ///:~
```
遗憾的是，这不能工作。一旦为Comparable确定了ComparablePet参数，那么其他任何实现类都不能与ComparablePet之外的任何对象比较：
```java
//: generics/RestrictedComparablePets.java
class Hamster extends ComparablePet
implements Comparable<ComparablePet> {
  public int compareTo(ComparablePet arg) { return 0; }
}
// Or just:
class Gecko extends ComparablePet {
  public int compareTo(ComparablePet arg) { return 0; }
} ///:~
```
Hamster说明再次实现ComparablePet中的相同接口是可能的，只要她们精确地相同，包括参数类型在内。但是，这只是与覆盖基类中的方法相同，就像在Geeko中看到的那样。