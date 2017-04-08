## 15.7 擦除的神秘之处
当你开始更深人地钻研泛型时，会发现有大量的东西初看起来是没有意义的。例如，尽管可以声明ArrayList.class，但是不能声明`ArrayList<Integer>.class`，请考虑下面的情况：
```java
//: generics/ErasedTypeEquivalence.java
import java.util.*;
public class ErasedTypeEquivalence {
  public static void main(String[] args) {
  Class c1 = new ArrayList<String>().getClass();
  Class c2 = new ArrayList<Integer>().getClass();
  System.out.println(c1 == c2);
  }
} /* Output:
true
*///:~
```
`ArrayList<String>`和`ArrayList<Integer>`很容易被认为是不同的类型。不同的类型在行为方面肯定不同，例如，如果尝试着将一个Integer放入`ArrayList<String>`，所得到的行为(将失败)与把一个Integer放入`ArrayList<Integer>`(将成功)所得到的行为完全不同。但是上面的程序会认为它们是相同的类型。
下面是的示例是对这个谜题的一个补充：
```java
//: generics/LostInformation.java
import java.util.*;
class Frob {}
class Fnorkle {}
class Quark<Q> {}
class Particle<POSITION,MOMENTUM> {}
public class LostInformation {
  public static void main(String[] args) {
  List<Frob> list = new ArrayList<Frob>();
  Map<Frob,Fnorkle> map = new HashMap<Frob,Fnorkle>();
  Quark<Fnorkle> quark = new Quark<Fnorkle>();
  Particle<Long,Double> p = new Particle<Long,Double>();
  System.out.println(Arrays.toString(
  list.getClass().getTypeParameters()));
  System.out.println(Arrays.toString(
  map.getClass().getTypeParameters()));
  System.out.println(Arrays.toString(
  quark.getClass().getTypeParameters()));
  System.out.println(Arrays.toString(
  p.getClass().getTypeParameters()));
  }
} /* Output:
[E]
[K, V]
[Q]
[POSITION, MOMENTUM]
*///:~
```
根据JDK文档的描述，Class.getTypeParameters()将“返回一个TypeVariable对象数组，表示有泛型声明所声明的类型参数...”。这好像是在暗示你可能发现参数类型的信息，但是，正如你从输出中所看到的，你能够发现的只是用作参数占位符的标识符，这并非有用的信息。
因此，残酷的现实是：
在泛型代码内部，无法获得任何有关泛型参数类型的信息。
因此，你可以知道诸如类型参数标识符和泛型类型边界这类的信息——你却无法知道用来创建某个特定实例的实际的类型参数。如果你曾经是C++程序员，那么这个事实肯定让你觉得很沮丧，在使用Java泛型工作时它是必须处理的最基本的间题。
Java泛型是使用擦除来实现的，这意味着当你在使用泛型时，任何具体的类型信息都被擦除了，你唯一知遒的就是你在使用一个对象。因此`List<String>`和`List<interger>`在运行时事实上是相同的类型。这两种形式都被擦除成它们的“原生”类型，即List。理解擦除以及应该如何处理它，是你在学习Java泛型时面临的最大障碍。这也是我们在本节将要探讨的内容。
### 15.7.1 C++的方式
下面是使用模版的C++示例，你将注意到用于参数化类型的语法十分相似，因为Java是受C++的启发：
```cpp
//: generics/Templates.cpp
#include <iostream>
using namespace std;
template<class T> class Manipulator {
  T obj;
public:
  Manipulator(T x) { obj = x; }
  void manipulate() { obj.f(); }
};
class HasF {
public:
  void f() { cout << "HasF::f()" << endl; }
};
int main() {
  HasF hf;
  Manipulator<HasF> manipulator(hf);
  manipulator.manipulate();
} /* Output:
HasF::f()
///:~
```
Manipulator类存储了一个类型T的对象，有意思的地方是manipulate()方法，它在obj上调用方f()。它怎么能知道f()方法是为类型参数T而存在的呢？当你实例化这个模版时，C++编译器将进行检查，因此在`Manipulalor<HasF>`被实例化的这一刻，它看到HasF拥有一个方法f()。如果情况井非如此，就会得到一个编译期错误，这样类型安全就得到了保障。
用C++编写这种代码很简单，因为当模版被实例化时，模版代码知道其模版参数的类型。Java泛型就不同了。下面是HasF的Java版本：
```java
//: generics/HasF.java
public class HasF {
  public void f() { System.out.println("HasF.f()"); }
} ///:~
```
如果我们将这个示例的其余代码都翻译成Java，那么这些代码将不能编译：
```java
//: generics/Manipulation.java
// {CompileTimeError} (Won't compile)
class Manipulator<T> {
  private T obj;
  public Manipulator(T x) { obj = x; }
  // Error: cannot find symbol: method f():
  public void manipulate() { obj.f(); }
}
public class Manipulation {
  public static void main(String[] args) {
  HasF hf = new HasF();
  Manipulator<HasF> manipulator =
  new Manipulator<HasF>(hf);
  manipulator.manipulate();
  }
} ///:~
```
由于有了擦除，Java编译器无法将maaipulate()必须能够在obj上调用f()这一需求映射到HasF有f()这一事实上。为了调用f()，我们必须协助泛型类，给定泛型类的边界，以此告知编译器只能接受遵循这个边界的类型。这里重用了extends关键字。由于有了边界，下面的代码就可以编译了:
```java
//: generics/Manipulator2.java
class Manipulator2<T extends HasF> {
  private T obj;
  public Manipulator2(T x) { obj = x; }
  public void manipulate() { obj.f(); }
} ///:~
```
边界`<T extends HasF>`声明T必须具有类型HasF或者从HasF导出的类型。如果情况确实如此，那么就可以安全地在obj上调用f()了。
我们说泛型类型参数将擦除到它的第一个边界(它可能会有多个边界，稍候你就会看到)，我们还捉到了`类型参数`的擦除。编译器实际上会把类型参数替换为它的擦除，就像上面的示例一样。T擦除到了HasF，就好像在类的声明中用HasF替换了T一样。
你可能已经正确地观察到，在Manipulation2.java中，泛型没有贡献任何好处。只需很容易地自己去执行擦除，就可以创建出没有泛型的类：
```java
//: generics/Manipulator3.java
class Manipulator3 {
  private HasF obj;
  public Manipulator3(HasF x) { obj = x; }
  public void manipulate() { obj.f(); }
} ///:~
```
这提出了很重要的一点：只有当你希望使用的类型参数比某个具体类型(以及它的所有子类型)更加“泛化”时——也就是说，当你希望代码能够跨多个类工作时，使用泛型才有所帮助。因此，类型参数和它们在有用的泛型代码中的应用，通常比简单的类替换要更复杂。但是，不能因此而认为`<T extends HasF>`形式的任何东西而都是有缺陷的。例如，如果某个类有一个返回T的方法，那么泛型就有所帮助，因为它们之后将返回确切的类型：
```java
//: generics/ReturnGenericType.java
class ReturnGenericType<T extends HasF> {
  private T obj;
  public ReturnGenericType(T x) { obj = x; }
  public T get() { return obj; }
} ///:~
```
必须查看所有的代码，并确定它是否“足够复杂”到必须作用泛型的程度。
  我们将在本章稍后介绍有关边界的更多细节。
### 15.7.2 迁移兼容性
为了减少潜在的关于擦除的混淆，你必须清楚地认识到这不是一个语言特性。它是Java的泛型实现中的一种折中，因为泛型不是Java语言出现时就有的组成部分，所以这种折中是必需的。这种折中会使你痛苦，因此你需要习惯它并了解为什么它会是这样。
如果泛型在Java 1.0中就已经是其一部分了，那么这个特性将不会使用擦除来实现——它将使用县体化，使类型参数保持为第一类实体，因此你就能够在类型参数上执行基于类型的语言操作和反射操作。你将在本章稍后看到，擦除减少了泛型的泛化性。泛型在Java中仍归是有用的，只是不如它们本来设想的那么有用，而原因就是擦除。
在基于擦除的实现中，泛型类型被当作第二类类型处理，即不能在某些重要的上下文环境中使用的类型。泛型类型只有在静态类型检查期间才出现，在此之后，程序中的所有泛型类型都将被擦除，替换为它们的非泛型上界。例如，诸如`List<T>`这样的类型注解将被擦除为List，而普通的类型变量在未指定边界的情况下将被擦除为Object。
擦除的核心动机是它使得泛化的客户端可以用非泛化的类库来使用，反之亦然，这经常被称为“迁移兼容性”。在理想情况下，当所有事物都可以同时被泛化时，我们就可以专注于此。在现实中，即使程序员只编写泛型代码，他们也必须处理在Java SE5之前编写的非泛型类库。那些类库的作者可能从没有想过要泛化它们的代码，或者可能刚刚开始接触泛型。
因此Java泛型不仅必须支特向后兼容性，即现有的代码和类文件仍旧合法，井且继续保持其之前的含义而且还要支持迁移兼容性，使得类库接照它们自己的步调变为泛型的，并且当某个类库变为泛型时，不会破坏依赖于它的代码和应用程序。在决定这就是目标之后，Java设计者们和从事此问题相关工作的各个团队决策认为擦除是唯一可行的解决方案。通过允许非泛型代码与泛型代码共存，擦除使得这种向着泛型的迁移成为可能。
例如，假设某个应用程序具有两个类库X和Y，并且Y还要使用类库Z。随着Java SE5的出现，这个应用程序和这些类库的创建者最终可能希望迁移到泛型上。但是，当进行这种迁移时，他们有着不同动机和限制。为了实现迁移兼容性，每个类库和应用程序都必须与其他所有的部分是否使用了泛型无关。这样，它们必须不具备探测其他类库是否使用了泛型的能力。因此，某个特定的类库使用了泛型这样的证据必须被“擦除”。
如果没有某种类型的迁移途径，所有已经构建了很长时间的类库就需要与希望迁移到Java泛型上的开发者们说再见了。但是，类库是编程语言无可争议的一部分，它们对生产效率会产生最重要的影响，因此这不是一种可以接受的代价。擦除是否是最佳的或者唯一的迁移途径，还需要时间来证明。
### 15.7.3 擦除的问题
因此，擦除主要的正当理由是从非泛化代码到泛化代码的转变过程，以及在不破坏现有类库的情况下，将泛型融入Java语言。擦除使得现有的非泛型客户端代码能够在不改变的情况下继续使用，直至客户端准备好用泛型重写这些代码。这是一个崇高的动机，因为它不会突然间破坏所有现有的代码。
擦除的代价是显著的。泛型不能用于显式地引用运行时类型的操作之中，例如转型、instanceof操作和new表达式。因为所有关于参数的类型信息都丢失了。无论何时，当你在编写泛型代码时，必须时刻提醒自己，你只是`看起来好像`拥有有关参数的类型信息而已。因此，如果你编写了下面这样的代码段：
```java
class Foo<T> {
  T var;
}
```
那么，看起来当你在创建Foo实例时：
```java
Foo<Cat> f = new Foo<Cat>();
```
class Foo中的代码应该知道现在工作于Cat之上，而泛型语法也在强烈暗示：在整个类中的各个地方，类型T都在被替换。但是事实并非如此，无论何时，当你在编写这个类的代码时，必须提醒自己：“不，它只是一个Object。”
另外，擦除和迁移兼容性意味着，使用泛型并不是强制的，尽管你可能希望这样：
```java
//: generics/ErasureAndInheritance.java
class GenericBase<T> {
  private T element;
  public void set(T arg) { arg = element; }
  public T get() { return element; }
}
class Derived1<T> extends GenericBase<T> {}
class Derived2 extends GenericBase {} // No warning
// class Derived3 extends GenericBase<?> {}
// Strange error:
// unexpected type found : ?
// required: class or interface without bounds
public class ErasureAndInheritance {
  @SuppressWarnings("unchecked")
  public static void main(String[] args) {
  Derived2 d2 = new Derived2();
  Object obj = d2.get();
  d2.set(obj); // Warning here!
  }
} ///:~
```
Derived2继承自GenericBase，但是没有任何泛型参数，而编译器不会发出任何警告。警告在Set()被调用时才会出现。
为了关闭警告，Java提供了一个注解，我们可以在列表中看到它(这个注解在Java SE5之前的版本中不支持):
```java
  @suppressWarnings("unchecked")
```
注意，这个往解被放置在可以产生这类警告的方法之上，而不是整个类上。当你要关闭警告时，最好是尽量地“聚焦”，这样就不会因为过于宽泛地关闭警告，而导致意外地遮蔽掉真正的问题。
可以推断，Derived3产生的错误意味着编译器期望得到一个原生基类。
当你希望将类型参数不要仅仅当作Object处理时，就需要付出额外努力来管理边界，并且与在C++, Ada和Eiffel这样的语言中获得参数化类型相比，你需要付出多得多的努力来获得少得多的回报。这并不是说，对于大多数的编程问题而言，这些语言通常都会比Java更得心应手，这只是说，它们的参数化类型机制比Java的更灵活、更强大。
### 15.7.4 边界处的动作
正是因为有了擦除，我发现泛型最令人困惑的方面源自这样一个事实，即可以表示没有任何意义的事物。例如：
```java
//: generics/ArrayMaker.java
import java.lang.reflect.*;
import java.util.*;
public class ArrayMaker<T> {
  private Class<T> kind;
  public ArrayMaker(Class<T> kind) { this.kind = kind; }
  @SuppressWarnings("unchecked")
  T[] create(int size) {
  return (T[])Array.newInstance(kind, size);
  }
  public static void main(String[] args) {
  ArrayMaker<String> stringMaker =
  new ArrayMaker<String>(String.class);
  String[] stringArray = stringMaker.create(9);
  System.out.println(Arrays.toString(stringArray));
  }
} /* Output:
[null, null, null, null, null, null, null, null, null]
*///:~
```
即使kind被存储为`Class<T>`，擦除也意味着它实际将被存储为Class，没有任何参数。因此，当你在使用它时，例如在创建数组时，Array.newInstance()实际上并未拥有kind所蕴含的类型信息，因此这不会产生具体的结果。所以必须转型，这将产生一条令你无法满意的警告。
注意，对于在泛型中创建数组，使用Array.newInstance()是推荐的方式。
如果我们要创建一个容器而不是数组，情况就有些不同了：
```java
//: generics/ListMaker.java
import java.util.*;
public class ListMaker<T> {
  List<T> create() { return new ArrayList<T>(); }
  public static void main(String[] args) {
  ListMaker<String> stringMaker= new ListMaker<String>();
  List<String> stringList = stringMaker.create();
  }
} ///:~
```
编译器不会给出任何警告，尽管我们(从擦除中)知道在create()内部的`new ArrayList<T>`中的`<T>`被移除了——在运行时，这个类的内部没有任何`<T>`，因此这看起来毫无意义。但是如果你遵从这种思路，并将这个表达式改为new ArrayList()，编译器就会给出警告。
在本例中，这是否真的毫无意义呢？如果返回list之前，将某些对象放入其中。就像下面这样，情况又会如何呢？
```java
//: generics/FilledListMaker.java
import java.util.*;
public class FilledListMaker<T> {
  List<T> create(T t, int n) {
  List<T> result = new ArrayList<T>();
  for(int i = 0; i < n; i++)
  result.add(t);
  return result;
  }
  public static void main(String[] args) {
  FilledListMaker<String> stringMaker =
  new FilledListMaker<String>();
  List<String> list = stringMaker.create("Hello", 4);
  System.out.println(list);
  }
} /* Output:
[Hello, Hello, Hello, Hello]
*///:~
```
即使编译器无法知道有关create()中的T的任何信息，但是它仍旧可以在编译期确保你放置到result中的对象具有T类型，使共适合`ArrayList<T>`。因此，即使擦除在方法或类内部移除了有关实际类型的信息，编译器仍旧可以确保在方法或类中使用的类型的内部一致性。
因为擦除在方法体中移除了类型信息，所以在运行时的问题就是边界：即对象进入和离开方法的地点。这些正是编译器在编译期执行类型检查并插入转型代码的地点。请考虑下面的非泛型示例：
```java
//: generics/SimpleHolder.java
public class SimpleHolder {
  private Object obj;
  public void set(Object obj) { this.obj = obj; }
  public Object get() { return obj; }
  public static void main(String[] args) {
  SimpleHolder holder = new SimpleHolder();
  holder.set("Item");
  String s = (String)holder.get();
  }
} ///:~
```
如果用Javap -c SimpleHolder反编译这个类，就可以得到下面的(经过编辑的)内容：
```
public void set (java.lang.Object):
0: aload_0
1: aload_1
2: putfield #2; //Field obj:Object
5: return
public java.lang.Object get():
0: aload_0
1: getfield #2; //Field obj:Object
4: areturn
public static void main(java.lang.String[]):
0: new #3; //class SimpleHolder
3: dup
4: invokespecial #4; //Method "<init>:()v
7: a-store_1
8: aload_ 1
9: ldc #5; //String Item;
11: invokevitrual #6; //Method set:(Object;)V
14: aload_1
15: invokevirtual #7: //Method get:()Object;
18: checkcast #8; //class java/lang/String
21: astore_2
22: return
```
get()和set()方法将直接存储和产生值，而转型是在调用get()的时候接受检查的。
现在将泛型合并到上面的代码中：
```java
//: generics/GenericHolder.java
public class GenericHolder<T> {
  private T obj;
  public void set(T obj) { this.obj = obj; }
  public T get() { return obj; }
  public static void main(String[] args) {
  GenericHolder<String> holder =
  new GenericHolder<String>();
  holder.set("Item");
  String s = holder.get();
  }
} ///:~
```
从get()返回之后的转型消失了，但是我们还知道传递给set()的值在编译期会接受检查。下面是相关的字节码：
```
public void set (java.lang.Object):
0: aload_0
1: aload_1
2: putfield #2; //Field obj:Object
5: return
public java.lang.Object get():
0: aload_0
1: getfield #2; //Field obj:Object
4: areturn
public static void main(java.lang.String[]):
0: new #3; //class GenericHolder
3: dup
4: invokespecial #4; //Method "<init>:()v
7: a-store_1
8: aload_ 1
9: ldc #5; //String Item;
11: invokevitrual #6; //Method set:(Object;)V
14: aload_1
15: invokevirtual #7: //Method get:()Object;
18: checkcast #8; //class java/lang/String
21: astore_2
22: return
```
所产生的字节码是相同的。对进入set()的类型进行检查是不需要的，因为这将由编译器执行。而对从get()返回的值进行转型仍旧是需要的，但这与你自己必须执行的操作是一样的——此处它将由编译器自动插入，因此你写人(和读取)的代码的噪声将更小，
由于所产生的get()和set()的字节码相同，所以在泛型中的所有动作都发生在边界处——对传递进来的值进行额外的编译期检查，并插入对传递出去的值的转型。这有助于澄清对擦除的混淆，记住，“边界就是发生动作的地方”。