## 15.3 泛型接口
泛型也可以应用于接口。例如生成器(generator)，这是一种专门负责创建对象的类。实际上，这是工厂方法设计模式的一种应用。不过，当使用生成器创建新的对象时，它不需要任何参数，而工厂方法一般需要参数。也鼓是说，生成器无需额外的信息就知道如何创建新对象。
一般而言，一个生成器只定义一个方法，该方法用以产生新的对象.在这里，就是next方法。
```java
// A generic interface.
public interface Generator<T> { T next(); }
```
> 方法next()的返回类型是参数化的T。正如你所见到的，接口使用泛型与类使用泛型没什么区别。

为了演示如何实现Generator接口，我们还需要一些别的类。例如，Coffee类层次结构如下：
```java
public class Coffee {
  private static long counter = 0;
  private final long id = counter++;
  public String toString() {
  return getClass().getSimpleName() + " " + id;
  }
}
public class Latte extends Coffee {} // 拿铁
public class Mocha extends Coffee {} // 摩卡
public class Cappuccino extends Coffee {} // 卡布奇诺
public class Americano extends Coffee {} // 美式咖啡
public class Breve extends Coffee {}
```
现在，我们可以编写一个类，实现`Generator<Coffee>`接口，它能够随机生成不同类型的Coffee对象。
```java
public class CoffeeGenerator implements Generator<Coffee>, Iterable<Coffee> {
  private Class[] types = { Latte.class, Mocha.class,
  Cappuccino.class, Americano.class, Breve.class, };
  private static Random rand = new Random(47);
  public CoffeeGenerator() {}
  // For iteration:
  private int size = 0;
  public CoffeeGenerator(int sz) { size = sz; }
  public Coffee next() {
  try {
  return (Coffee)
  types[rand.nextInt(types.length)].newInstance();
  // Report programmer errors at run time:
  } catch(Exception e) {
  throw new RuntimeException(e);
  }
  }
  class CoffeeIterator implements Iterator<Coffee> {
  int count = size;
  public boolean hasNext() { return count > 0; }
  public Coffee next() {
  count--;
  return CoffeeGenerator.this.next();
  }
  public void remove() { // Not implemented
  throw new UnsupportedOperationException();
  }
  };
  public Iterator<Coffee> iterator() {
  return new CoffeeIterator();
  }
  public static void main(String[] args) {
  CoffeeGenerator gen = new CoffeeGenerator();
  for(int i = 0; i < 5; i++)
  System.out.println(gen.next());
  for(Coffee c : new CoffeeGenerator(5))
  System.out.println(c);
  }
} /* Output:
Americano 0
Latte 1
Americano 2
Mocha 3
Mocha 4
Breve 5
Americano 6
Latte 7
Cappuccino 8
Cappuccino 9
*///:~
```
> 参数化的Generator接口确保next()的返回值是参数的类型。CofeeGenerator同时还实现了Iterable接口，所以它可以在循环语句中使用。不过，它还需要一个“末端哨兵”来判断何时停止，这正是第二个构造器的功能。

下面的类是`Generetore<T>`接口的另一个实现，它负责生成Fibonacci(斐波那契)数列：
```java
public class Fibonacci implements Generator<Integer> {
  private int count = 0;
  public Integer next() { return fib(count++); }
  private int fib(int n) {
  if(n < 2) return 1;
  return fib(n-2) + fib(n-1);
  }
  public static void main(String[] args) {
  Fibonacci gen = new Fibonacci();
  for(int i = 0; i < 18; i++)
  System.out.print(gen.next() + " ");
  }
}
/* Output:
1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584
*///:~
```
> 虽然我们在Fibonacci类的里里外外使用的都是int类型，但是其类型参数却是Integer,这个例子引出了Java泛型的一个局限性：基本类型无法作为类型参数。不过，Java 5具备了自动打包和自动拆包的功能，可以很方便地在基本类型和其相应的包装器类型之间进行转换。通过这个例子中Fibonacci类对int的使用，我们已经看到了这种效果。
如果还想更进一步，编写一个实现了Iterable的Fibonacci生成器。我们的一个选择是重写这个类，令其实现Iterable接口。不过，你并不是总能拥有源代码的控制权，井且，除非必须这么做，否则，我们也不愿意重写一个类。而且我们还有另一种选择，就是创建个过配器(adapter)来实现所需的接口，我们在前面介绍过这个设计模式。
有多种方法可以实现适配器。例如，可以通过继承来创建适配器类：

```java
public class IterableFibonacci extends Fibonacci implements Iterable<Integer> {
  private int n;
  public IterableFibonacci(int count) { n = count; }
  public Iterator<Integer> iterator() {
  return new Iterator<Integer>() {
  public boolean hasNext() { return n > 0; }
  public Integer next() {
  n--;
  return IterableFibonacci.this.next();
  }
  public void remove() { // Not implemented
  throw new UnsupportedOperationException();
  }
  };
  }
  public static void main(String[] args) {
  for(int i : new IterableFibonacci(18))
  System.out.print(i + " ");
  }
}
/* Output:
1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584
*///:~
```
> 如果要在循环语句中使用IterableFibonacci，必须向IterableFibonacci的构造器提供一个边界值，然后hasNext()方法才能知道何时应该返回false。