## 15.8 擦除的补偿
正如我们看到的，擦除丢失了在泛型代码中执行某些操作的能力。任何在运行时需要知道确切类型信息的操作都将无法工作:
```java
//: generics/Erased.java
// {CompileTimeError} (Won't compile)
public class Erased<T> {
  private final int SIZE = 100;
  public static void f(Object arg) {
  if(arg instanceof T) {} // Error
  T var = new T(); // Error
  T[] array = new T[SIZE]; // Error
  T[] array = (T)new Object[SIZE]; // Unchecked warning
  }
} ///:~
```
偶尔可以绕过这些问题来编程，但是有时必须通过引入类型标签来对擦除进行补偿。这意味着你需要显式地传递你的类型的Class对象，以便你可以在类型表达式中使用它。
例如，在前面示例中对使用instanceof的尝试最终失败了，因为其类型信息已经被擦除了。如果引入类型标签，就可以转而使用动态的isInstance()：
```java
//: generics/ClassTypeCapture.java
class Building {}
class House extends Building {}
public class ClassTypeCapture<T> {
  Class<T> kind;
  public ClassTypeCapture(Class<T> kind) {
  this.kind = kind;
  }
  public boolean f(Object arg) {
  return kind.isInstance(arg);
  }
  public static void main(String[] args) {
  ClassTypeCapture<Building> ctt1 =
  new ClassTypeCapture<Building>(Building.class);
  System.out.println(ctt1.f(new Building()));
  System.out.println(ctt1.f(new House()));
  ClassTypeCapture<House> ctt2 =
  new ClassTypeCapture<House>(House.class);
  System.out.println(ctt2.f(new Building()));
  System.out.println(ctt2.f(new House()));
  }
} /* Output:
true
true
false
true
*///:~
```
编译器将确保类型标签可以匹配泛型参数。
### 15.8.1 创建类型实例
在Erased.java中对创建一个new T()的尝试将无法实现，部分原因是因为擦除，而另一部分原因是因为编译器不能验证T具有默认(无参)构造器。但是在C++中，这种操作很自然、很直观，并且很安全(它是在编译期受到检查的)：
```cpp
//: generics/InstantiateGenericType.cpp
// C++, not Java!
template<class T> class Foo {
  T x; // Create a field of type T
  T* y; // Pointer to T
public:
  // Initialize the pointer:
  Foo() { y = new T(); }
};
class Bar {};
int main() {
  Foo<Bar> fb;
  Foo<int> fi; // ... and it works with primitives
} ///:~
```
Jaya中的解决方案是传递一个工厂对象，并使用它来创建新的实例。最便利的工厂对象就是Class对象，因此如果使用类型标签，那么你就可以使用newInstance来创建这个类型的新对象：
```java
//: generics/InstantiateGenericType.java
import static net.mindview.util.Print.*;
class ClassAsFactory<T> {
  T x;
  public ClassAsFactory(Class<T> kind) {
  try {
  x = kind.newInstance();
  } catch(Exception e) {
  throw new RuntimeException(e);
  }
  }
}
class Employee {}
public class InstantiateGenericType {
  public static void main(String[] args) {
  ClassAsFactory<Employee> fe =
  new ClassAsFactory<Employee>(Employee.class);
  print("ClassAsFactory<Employee> succeeded");
  try {
  ClassAsFactory<Integer> fi =
  new ClassAsFactory<Integer>(Integer.class);
  } catch(Exception e) {
  print("ClassAsFactory<Integer> failed");
  }
  }
} /* Output:
ClassAsFactory<Employee> succeeded
ClassAsFactory<Integer> failed
*///:~
```
这可以编译，但是会因`ClassAsFactory<Integer>`而失败，因为Integer没有任何默认的构造器。因为这个错误不是在编译期捕获的，所以Sun的伙计们对这种方式井不赞成，他们建议使用显式的工厂，井将限制其类型，使得只能接受实现了这个工厂的类：
```java
//: generics/FactoryConstraint.java
interface FactoryI<T> {
  T create();
}
class Foo2<T> {
  private T x;
  public <F extends FactoryI<T>> Foo2(F factory) {
  x = factory.create();
  }
  // ...
}
class IntegerFactory implements FactoryI<Integer> {
  public Integer create() {
  return new Integer(0);
  }
}
class Widget {
  public static class Factory implements FactoryI<Widget> {
  public Widget create() {
  return new Widget();
  }
  }
}
public class FactoryConstraint {
  public static void main(String[] args) {
  new Foo2<Integer>(new IntegerFactory());
  new Foo2<Widget>(new Widget.Factory());
  }
} ///:~
```
注意，这确实只是传递`Class<T>`的一种变体。两种方式都传递了工厂对象，`Class<T>`碰巧是内建的工厂对象，而上面的方式创建了一个显式的工厂对象，但是你却获得了编译期检查。
另一种方式是模板方法设计模式。在下面的示例中，get()是模板方法，而create()是在子类中定义的、用来产生子类类型的对象:
```java
//: generics/CreatorGeneric.java
abstract class GenericWithCreate<T> {
  final T element;
  GenericWithCreate() { element = create(); }
  abstract T create();
}
class X {}
class Creator extends GenericWithCreate<X> {
  X create() { return new X(); }
  void f() {
  System.out.println(element.getClass().getSimpleName());
  }
}
public class CreatorGeneric {
  public static void main(String[] args) {
  Creator c = new Creator();
  c.f();
  }
} /* Output:
X
*///:~
```
### 15.8.2泛型数组
正如你在Erased.java中所见，不能创建泛型数组。一般的解决方案是在任何想要创建泛型数组的地方都使用ArrayList:
```java
//: generics/ListOfGenerics.java
import java.util.*;
public class ListOfGenerics<T> {
  private List<T> array = new ArrayList<T>();
  public void add(T item) { array.add(item); }
  public T get(int index) { return array.get(index); }
} ///:~
```
这里你将获得数组的行为，以及由泛型提供的编译期的类型安全。
有时，你仍旧希璧创建泛型类型的数组(例如，ArrayList内部使用的是数组)。有趣的是可以按照编译器喜欢的方式来定义一个引用，例如:
```java
//: generics/ArrayOfGenericReference.java
class Generic<T> {}
public class ArrayOfGenericReference {
  static Generic<Integer>[] gia;
} ///:~
```
编译器将接受这个程序，而不会产生任何警告。但是，永远都不能创建这个确切类型的数组(包括类型参数)，因此这有一点令人困惑。既然所有数组无论它们持有的类型如何，都具有相同的结构(每个数组槽位的尺寸和数组的布局)，那么看起来你应该能够创建一个Object数组，并将其转型为所希望的数组类型。事实上这可以编译，但是不能运行，它将产生ClassCaseException:
```java
//: generics/ArrayOfGeneric.java
public class ArrayOfGeneric {
  static final int SIZE = 100;
  static Generic<Integer>[] gia;
  @SuppressWarnings("unchecked")
  public static void main(String[] args) {
  // Compiles; produces ClassCastException:
  //! gia = (Generic<Integer>[])new Object[SIZE];
  // Runtime type is the raw (erased) type:
  gia = (Generic<Integer>[])new Generic[SIZE];
  System.out.println(gia.getClass().getSimpleName());
  gia[0] = new Generic<Integer>();
  //! gia[1] = new Object(); // Compile-time error
  // Discovers type mismatch at compile time:
  //! gia[2] = new Generic<Double>();
  }
} /* Output:
Generic[]
*///:~
```
问题在于数组将跟踪它们的实际类型，而这个类型是在数组被创建时确定的，因此，即使gia已经被转型为`Generic<Integer>[]`，但是这个信息只存在干编译期（如果没有@Suppress Warnings注解，你将得到有关这个转型的警告)。在运行时，它仍将引发问题。成功创建泛型数组的唯一方式就是创建一个被擦除类型的新数组，然后对其转型。
让我们看一个更复杂的示例。考虑一个简单的泛型数组包装器：
```java
//: generics/GenericArray.java
public class GenericArray<T> {
  private T[] array;
  @SuppressWarnings("unchecked")
  public GenericArray(int sz) {
  array = (T[])new Object[sz];
  }
  public void put(int index, T item) {
  array[index] = item;
  }
  public T get(int index) { return array[index]; }
  // Method that exposes the underlying representation:
  public T[] rep() { return array; }
  public static void main(String[] args) {
  GenericArray<Integer> gai =
  new GenericArray<Integer>(10);
  // This causes a ClassCastException:
  //! Integer[] ia = gai.rep();
  // This is OK:
  Object[] oa = gai.rep();
  }
} ///:~
```
与前面相同，我们井不能声明T[] array=new T[sz]，因此我们创建了一个对象数组，然后将其转型。
  rep()方法将返回T[]，它在main()中将用于gai，因此应该是Integer[]，但是如果调用它，并尝试着将结果作为Integer[]引用来捕获，就会得到ClassCastException，这还是因为实际的运行时类型是Objet[]。
如果在注释掉@Suppresswarniugs注解之后再编译GenericArray.java ,编译器就会产生警告:
```
Note: GenericArray.java uses unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
```
在这种情况下，我们将只获得单个的警告，并且相信这事关转型。但是如果真的想要确定是否是这么回事，就应该用.Xlint:unchecked来编译：
```
GenerieArray.java:7: warning:[unchecked] unchecked cast
found : java.lang.Object[]
required: T[]
array = (T[])new Object[sz];
  ^
1 warning
```
这确实是对转型的抱怨。因为警告会变得令人迷惑，所以一旦我们验证某个特定警告是可预期的，那么我们的上策就是用@SuppressWarnings关闭它。通过这种方式，当警告确实出现时，我们就可以真正地展开对它的调查了。
因为有了擦除，数组的运行时类型就只能是Object[]。如果我们立即将其转型为T[]，那么在编译期该数组的实际类型就将丢失，而编译器可能会错过某些潜在的错误检查。正因为这样，最好是在集合内部使用Object[]，然后当你使用数组元素时，添加一个对T的转型。让我们看着这是如何作用于GenericArray.java示例的：
```java
//: generics/GenericArray2.java
public class GenericArray2<T> {
  private Object[] array;
  public GenericArray2(int sz) {
  array = new Object[sz];
  }
  public void put(int index, T item) {
  array[index] = item;
  }
  @SuppressWarnings("unchecked")
  public T get(int index) { return (T)array[index]; }
  @SuppressWarnings("unchecked")
  public T[] rep() {
  return (T[])array; // Warning: unchecked cast
  }
  public static void main(String[] args) {
  GenericArray2<Integer> gai =
  new GenericArray2<Integer>(10);
  for(int i = 0; i < 10; i ++)
  gai.put(i, i);
  for(int i = 0; i < 10; i ++)
  System.out.print(gai.get(i) + " ");
  System.out.println();
  try {
  Integer[] ia = gai.rep();
  } catch(Exception e) { System.out.println(e); }
  }
} /* Output: (Sample)
0 1 2 3 4 5 6 7 8 9
java.lang.ClassCastException: [Ljava.lang.Object; cannot be cast to [Ljava.lang.Integer;
*///:~
```
初看起来，这好像没多大变化，只是转型挪了地方。如果没有@Suppresswarnings注解，你仍旧会得到unchecked告。但是，现在的内部表示是Object[]而不是T[]。当get()被调用时，它将对象转型为T，这实际上是正确的类型，因此这是安全的。然而，如果你调用rep() ,它还是尝试着将Object[]转型为T[]，这仍旧是不正确的，将在编译期产生警告，在运行时产生异常。因此，没有任何方式可以推翻底层的数组类型，它只能是Object[]。在内部将array当作Object[]而不是T[]处理的优势是：我们不太可能忘记这个数组的运行时类型，从而意外地引入缺陷(尽管大多数也可能是所有这类缺陷都可以在运行时快速地探测到)。
对于新代码，应该传递一个类型标记。在这种情况下，GenericArray看起来会像下面这样：
```java
//: generics/GenericArrayWithTypeToken.java
import java.lang.reflect.*;
public class GenericArrayWithTypeToken<T> {
  private T[] array;
  @SuppressWarnings("unchecked")
  public GenericArrayWithTypeToken(Class<T> type, int sz) {
  array = (T[])Array.newInstance(type, sz);
  }
  public void put(int index, T item) {
  array[index] = item;
  }
  public T get(int index) { return array[index]; }
  // Expose the underlying representation:
  public T[] rep() { return array; }
  public static void main(String[] args) {
  GenericArrayWithTypeToken<Integer> gai =
  new GenericArrayWithTypeToken<Integer>(
  Integer.class, 10);
  // This now works:
  Integer[] ia = gai.rep();
  }
} ///:~
```
类型标记`Class<T>`被传递到构造器中，以便从擦除中恢复，使得我们可以创建需要的实际类型的数组，尽管从转型中产生的警告必须用@Suppresswarnings压制住。一旦我们获得了实际类型。就可以返回它，并获得想要的结果，就像在main()中看到的那样。该数组的运行时类型是确切类型T[]。
遗憾的是，如果查看java SE5标准类库中的源代码，你就会看到从Object数组到参数化类型的转型遍及各处。例如，下面是经过整理和简化之后的从Collection中复制ArrayList的构造器：
```java
public ArrayList(Collection c) {
  size = c.size();
  elementData = (E[])new Object[size];
  c.toArray(elementData);
}
```
如果你通读ArrayList.java，就会发现它充满了这种转型。如果我们编译它，又会发生什么呢？
```
Note: ArrayList.java uses unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
```
可以十分肯定，标准类库会产生大量的警告。如果你曾经用过C++，特别是ANSI C之前的版本，你就会记得警告的特殊效果：当你发现可以忽略它们时，你就可以忽略。正是因为这个原因，最好是从编译器中不要发出任何消息，除非程序员必须对其进行响应。
Neal Gafter (Java SE5的领导开发者之一)在他的博客中指出，在重写Java库时，他十分懒散，而我们不应该像他那样。Neal还指出，在不破坏现有接口的情况下，他将无法修改某些Java类库代码。因此，即使在Java类库源代码中出现了某些惯用法，也不能表示这就是正确的解决之道。当查看类库代码时，你不能认为它就是应该在自己的代码中遵循的示例。