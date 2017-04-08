## 15.6 构建复杂模型.
泛型的一个重要好处是能够简单而安全地创建复杂的模型。例如，我们可以很容易地创建List元组：
```java
//: generics/TupleList.java
// Combining generic types to make complex generic types.
import java.util.*;
import net.mindview.util.*;
public class TupleList<A,B,C,D>
extends ArrayList<FourTuple<A,B,C,D>> {
  public static void main(String[] args) {
  TupleList<Vehicle, Amphibian, String, Integer> tl =
  new TupleList<Vehicle, Amphibian, String, Integer>();
  tl.add(TupleTest.h());
  tl.add(TupleTest.h());
  for(FourTuple<Vehicle,Amphibian,String,Integer> i: tl)
  System.out.println(i);
  }
} /* Output: (75% match)
(Vehicle@11b86e7, Amphibian@35ce36, hi, 47)
(Vehicle@757aef, Amphibian@d9f9c3, hi, 47)
*///:~
```
> 尽管这看上去有些冗长(特别是迭代器的创建)，但最终还是没有用过多的代码就得到了一个相当强大的数据结构。

下面是另一个示例，它展示了使用泛型类型来构建复杂模型是多么的简单。即使每个类部是作为一个构建块创建的，但是其整个还是包含许多部分。在本例中，构建的模型是一个零售店，它包含走廊、货架和商品：
```java
//: generics/Store.java
// Building up a complex model using generic containers.
import java.util.*;
import net.mindview.util.*;
class Product {
  private final int id;
  private String description;
  private double price;
  public Product(int IDnumber, String descr, double price){
  id = IDnumber;
  description = descr;
  this.price = price;
  System.out.println(toString());
  }
  public String toString() {
  return id + ": " + description + ", price: $" + price;
  }
  public void priceChange(double change) {
  price += change;
  }
  public static Generator<Product> generator =
  new Generator<Product>() {
  private Random rand = new Random(47);
  public Product next() {
  return new Product(rand.nextInt(1000), "Test",
  Math.round(rand.nextDouble() * 1000.0) + 0.99);
  }
  };
}
class Shelf extends ArrayList<Product> {
  public Shelf(int nProducts) {
  Generators.fill(this, Product.generator, nProducts);
  }
}
class Aisle extends ArrayList<Shelf> {
  public Aisle(int nShelves, int nProducts) {
  for(int i = 0; i < nShelves; i++)
  add(new Shelf(nProducts));
  }
}
class CheckoutStand {}
class Office {}
public class Store extends ArrayList<Aisle> {
  private ArrayList<CheckoutStand> checkouts =
  new ArrayList<CheckoutStand>();
  private Office office = new Office();
  public Store(int nAisles, int nShelves, int nProducts) {
  for(int i = 0; i < nAisles; i++)
  add(new Aisle(nShelves, nProducts));
  }
  public String toString() {
  StringBuilder result = new StringBuilder();
  for(Aisle a : this)
  for(Shelf s : a)
  for(Product p : s) {
  result.append(p);
  result.append("\\n");
  }
  return result.toString();
  }
  public static void main(String[] args) {
  System.out.println(new Store(14, 5, 10));
  }
} /* Output:
258: Test, price: $400.99
861: Test, price: $160.99
868: Test, price: $417.99
207: Test, price: $268.99
551: Test, price: $114.99
278: Test, price: $804.99
520: Test, price: $554.99
140: Test, price: $530.99
...
*///:~
```
> 正如我们在Store.tostring()中看到的，其结果是许多层容器，但是它们是类型安全且可管理的。令人印象深刻之处是组装这个的模型十分容易，并不会成为智力挑战。

泛型的一个重要好处是能够简单而安全地创建复杂的模型。例如，我们可以很容易地创建List元组：
```java
//: generics/TupleList.java
// Combining generic types to make complex generic types.
import java.util.*;
import net.mindview.util.*;
public class TupleList<A,B,C,D>
extends ArrayList<FourTuple<A,B,C,D>> {
  public static void main(String[] args) {
  TupleList<Vehicle, Amphibian, String, Integer> tl =
  new TupleList<Vehicle, Amphibian, String, Integer>();
  tl.add(TupleTest.h());
  tl.add(TupleTest.h());
  for(FourTuple<Vehicle,Amphibian,String,Integer> i: tl)
  System.out.println(i);
  }
} /* Output: (75% match)
(Vehicle@11b86e7, Amphibian@35ce36, hi, 47)
(Vehicle@757aef, Amphibian@d9f9c3, hi, 47)
*///:~
```
> 尽管这看上去有些冗长(特别是迭代器的创建)，但最终还是没有用过多的代码就得到了一个相当强大的数据结构。

下面是另一个示例，它展示了使用泛型类型来构建复杂模型是多么的简单。即使每个类部是作为一个构建块创建的，但是其整个还是包含许多部分。在本例中，构建的模型是一个零售店，它包含走廊、货架和商品：
```java
//: generics/Store.java
// Building up a complex model using generic containers.
import java.util.*;
import net.mindview.util.*;
class Product {
  private final int id;
  private String description;
  private double price;
  public Product(int IDnumber, String descr, double price){
  id = IDnumber;
  description = descr;
  this.price = price;
  System.out.println(toString());
  }
  public String toString() {
  return id + ": " + description + ", price: $" + price;
  }
  public void priceChange(double change) {
  price += change;
  }
  public static Generator<Product> generator =
  new Generator<Product>() {
  private Random rand = new Random(47);
  public Product next() {
  return new Product(rand.nextInt(1000), "Test",
  Math.round(rand.nextDouble() * 1000.0) + 0.99);
  }
  };
}
class Shelf extends ArrayList<Product> {
  public Shelf(int nProducts) {
  Generators.fill(this, Product.generator, nProducts);
  }
}
class Aisle extends ArrayList<Shelf> {
  public Aisle(int nShelves, int nProducts) {
  for(int i = 0; i < nShelves; i++)
  add(new Shelf(nProducts));
  }
}
class CheckoutStand {}
class Office {}
public class Store extends ArrayList<Aisle> {
  private ArrayList<CheckoutStand> checkouts =
  new ArrayList<CheckoutStand>();
  private Office office = new Office();
  public Store(int nAisles, int nShelves, int nProducts) {
  for(int i = 0; i < nAisles; i++)
  add(new Aisle(nShelves, nProducts));
  }
  public String toString() {
  StringBuilder result = new StringBuilder();
  for(Aisle a : this)
  for(Shelf s : a)
  for(Product p : s) {
  result.append(p);
  result.append("\\n");
  }
  return result.toString();
  }
  public static void main(String[] args) {
  System.out.println(new Store(14, 5, 10));
  }
} /* Output:
258: Test, price: $400.99
861: Test, price: $160.99
868: Test, price: $417.99
207: Test, price: $268.99
551: Test, price: $114.99
278: Test, price: $804.99
520: Test, price: $554.99
140: Test, price: $530.99
...
*///:~
```
> 正如我们在Store.tostring()中看到的，其结果是许多层容器，但是它们是类型安全且可管理的。令人印象深刻之处是组装这个的模型十分容易，并不会成为智力挑战。