
# 函数式编程：Functional programming
## 函数式编程的一些基本概念
函数式编程，又称函数程序设计、泛函编程，是一种编程范式。

<br><br>

### 函数式编程的定义：

简单来说：函数式编程就是将计算机的程序运算比作数学中的函数运算，函数运算的最终结果只能取决于输入值。当使用相同参数调用函数时，它将始终返回相同的结果。

规范来说：函数式编程有时被视为纯函数式编程的同义词，是将所有函数视为确定性数学函数或纯函数的函数式编程的一个子集。当使用一些给定参数调用纯函数时，它将始终返回相同的结果，并且不受任何可变状态或其他副作用(**side effect**)的影响。

<br><br>

### 程序的副作用：

定义：如果一次操作、一个方法或一个表达式在其本地环境之外修改了一些变量的状态，那么就可以认为程序产生了副作用。

副作用包括修改非局部变量、修改静态局部变量、修改通过引用传递的可变参数、执行I/O或调用其他副作用函数。


**副作用的表现**：

- 在方法变更一个状态后将其从新变更为新的状态
- 在方法执行期间变更本地（非当前方法的局部变量）或全局的变量
- 方法执行未返回前的一次数据库保存操作
- 在方法中打印的日志
- 在方法中执行I/O操作

<br>

### 函数编程与数学函数：

数学中的函数：将函数输入和输出相关联的表达式，函数的输出值始终依赖于函数的输入值。此外，多个函数可以组合成一个新的函数形式。

类似如下例子。表达式：y=x^2 - 1，输入：x=5，输出：y=24，且当x=5时，输出始终为24；

那么函数式编程也是类似的，有表达式：函数体（代码块），输入：方法参数，输出：返回值。

<br>

### 设计函数式编程应该遵循的原则：

- First-Class and Higher-Order Functions
- 纯函数
- 不变性
- 引用透明


当然以上原则并不需要完全遵循，只是作为更为完美的函数设计的参考。下面将对上述的原则一一解释


⭐**First-Class and Higher-Order Functions**

***First-Class Function***：一等函数，即在一门编程语言中，函数与变量并没有实质区别，函数也可以同普通变量一样，作为参数传递给其他函数，或将它们作为其他函数的值返回，并将它们分配给变量或将它们存储在数据结构中。这就意味着，函数名称与普通变量一样，其将被定义为函数类型 + 函数名称，如同 int a一样，只是这里的函数名称实际是由代码块组成的函数运算块。


***Higher-Order Function***：高阶函数有两以下两个特征：
1. 将一个或多个函数作为参数
2. 将函数作为结果返回


***一等函数和高阶函数的区别***：
1. 一等函数针对编程语言：当某种语言具有一等函数特性时，那么该语言将会把函数视为普通变量。更常见的说法是“一种语言是否支持一等函数”。
2. 高阶函数针对函数：高阶函数是对其他函数起作用的函数，当一个函数有N个函数作为参数或将函数作为返回值，那么该函数就是一个高阶函数。
3. 这两件事密切相关，因为很难想象一种具有一流函数的语言不支持高阶函数，相反，很难想象具有高阶函数但没有一流函数支持的语言。


⭐**纯函数**（Pure Functions）：即是没有副作用的函数。


⭐**不变性**（Immutability）：即一个对象被实例化后，其属性不能够被改变。这是语言级别的设计支持的。在 Java 中，我们必须自己创建不可变的数据结构（正如String、基本类型、math类一样）。[Immutables](https://www.baeldung.com/immutables)和[Project Lombok](https://www.baeldung.com/intro-to-project-lombok)提供了现成的框架，用于在 Java 中定义不可变数据结构。


⭐**引用透明**（Referential Transparency）：如果将表达式替换为其相应的值，且没有对程序的行为造成影响时，那么该表达式就是引用透明的。换句话说，引用透明的表达式没有副作用且具有不变性。

下面是baeldung的一个例子：

```java
public class SimpleData {
  private Logger logger = Logger.getGlobal();
  private String data;
  public String getData() {
    logger.log(Level.INFO, "Get data called for SimpleData");
    return data;
  }
  public SimpleData setData(String data) {
    logger.log(Level.INFO, "Set data called for SimpleData");
    this.data = data;
    return this;
  }
}


// 对logger的三个调用在语义上是等效的，但在引用上并不透明
String data = new SimpleData().setData("Baeldung").getData();
// 不是引用透明的，因为它会产生副作用，其将会打印一条日志。如果我们像第三次调用一样用它的值替换这个调用，我们将不会打印该日志。
logger.log(Level.INFO, new SimpleData().setData("Baeldung").getData());
// 不是引用透明的，因为SimpleData是可变的。
logger.log(Level.INFO, data);
logger.log(Level.INFO, "Baeldung");
```

<br><br>

### 函数式编程的作用：

采用函数式编程的最大优势是纯函数和不可变状态。大多数编程的挑战的根源在于副作用和可变状态。去掉它们会使我们的程序更易于阅读、推理、测试和维护。


<br><br><br>

------

## 函数式接口
### 说明：

函数式接口实际上只是函数式编程的定义，它将作为方法参数，被传给其他方法，而实现这些接口的方法一般被表现为lambda表达式（当然你也可以用一般接口的形式来实现函数式接口，也可以通过匿名内部类来实现，但lambda表达式实际上就是为了简化上述情况而产生的）。

<br><br>

### 函数式接口的定义：

要定义一个函数式接口，需要满足以下两种情况

1. 使用@FunctionalInterface注解，标注接口为函数式接口
2. 接口内部必须只有一个或以下的普通接口方法。但可以有任意数量的接口默认方法和接口静态方法，这些都不影响函数式接口的定义。

<br><br>

### JDK中的函数式接口：

**Function**：接受一个参数，并返回一个结果

```java
@FunctionalInterface
public interface Function<T, R> {


    // 接受一个参数，并返回一个结果
    R apply(T t);


    // apply方法的组合实现 表示before.apply(v)的函数输出值r将作为this.apply(r)的输入值
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }


    // apply方法的组合实现 表示this.apply(t)的函数输出值r将作为after.apply(r)的输入值
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
    
    // 函数将永远将输入参数作为输出参数
    static <T> Function<T, T> identity() {
        return t -> t;
    }
```

<br>

**Predicate**：接受一个参数，返回Boolean

```java
@FunctionalInterface
public interface Predicate<T> {


    // 接受一个参数，返回Boolean
    boolean test(T t);


    // 表示两个Predicate的与判断 this.test(t) and a.test(t)
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    
    // 表示非Predicate的判断 !this.test(t)
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }


    // 表示两个Predicate的或判断 this.test(t) or a.test(t)
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }


    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    
```


<br>


**Supplier**：无参返回一个结果。该函数可以接受无参构造方法、无参方法来返回一个结果

```java
@FunctionalInterface
public interface Supplier<T> {


    // 无参返回一个结果
    T get();
}
```

<br>

**Consumer**：接受一个参数，并处理，无返回结果

```java
@FunctionalInterface
public interface Consumer<T> {


    // 接受一个参数，并处理，无返回结果
    void accept(T t);


    // 组合操作 表示消费的先后顺序，当前实例消费完成后，after才开始消费
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
```

<br><br>

### 实例：

使用上面的函数接口实现简单的判断和输出功能：

```java
public class FITest<T, R> {
    public static void main(String[] args) {
        // 判断输入的密码是否正确 正确时计算并返回结果
        FITest<String, String> fiTest = new FITest<>();
        Scanner sc = new Scanner(System.in);
        System.out.print("请输入密码： ");
        String key = sc.next();
        String result = fiTest.getIfTrue(key, // 输入数据
                k1 -> k1 + " Is Right KEY! ", // Predicate判断为真时的回调方法
                k2 -> k2.equals("KEY") // 判断条件
        );
        System.out.println(result);
    }


    // 判断正确时打印输出
    public R getIfTrue(T t, Function<T, R> function, Predicate<T> predicate) {
        if (!predicate.test(t)) {
            System.err.println("密码错误！");
            return null;
        }
        return function.apply(t);
    }
}

// 输入输出结果
请输入密码： NKEY
null
密码错误！
----------------------
请输入密码： KEY
KEY Is Right KEY! 


```


<br><br><br>

------

## Lambda表达式：Lambda Expressions

### 概念：

即是一个**匿名函数**，即没有函数名的函数。Lambda表达式可以让我们不用实现函数式接口，而是通过函数体完成其接口方法。Lambda是实现回调函数（callback function）的一种方式。

<br><br>

#### 结构：

（输入 +） 函数体 （+ 输出）

```java
// 输入(String a)   
// "->" 符号后的部分都可以当作函数体 
// 输出一个String类型的变量upper
(String a) -> {
    String upper = a.toUpperCase();
    System.out.println(upper);
    return upper;
}
```

<br><br>


### 语法规则：

1. 声明类型、不声明类型

```java
(int a, int b) -> a + b

(a, b) -> a - b

// 前面两个表达式含义相同，相当于下面的方法
int sum(int a, int b) {
  return a +b;
```



注意：单个参数需要声明类型时，必须使用“()”圆括号。
<br>
```java
// 一个入参
(List<String> l) -> {
  l.forEach(element -> System.out.println(element));
}
```

<br>



2. 有输入参数、没有输入参数

```java
(String a) -> new String(a)
// 需要一个没有内容的括号表示无参
() -> new String()

// 以上相当于一个是有参方法，一个是无参方法
String newStr(String a) {
  return new String(a);
}

String newStr() {
  return new String();
```

<br>

3. 单条执行语句、多条执行语句

```java
() -> "lambda".toUpperCase()

() -> {
    // other statement
    return "lambda".toUpperCase();
}
```

注意：单条语句可以省去“{}”

<br>

4. 带有输出的Lambda表达式、不带有输出

```java
// 有返回值
(List<String> list) -> {
    return list;
}

(List<String> list) -> list


// 无返回值
(List<String> list) -> {
    list.clear();
}

(List<String> list) -> list.clear()
```

注意：

1. 当函数体只有一条且为输出语句时，需要确保简写表达式语句有返回值
2. 当无返回值的表达式只有一条时，需要确保简写表达式不会有返回值


<br><br><br>

------

## 方法引用：Method Reference

### 定义：

当使用一个Lambda表达式创建一个匿名函数时，如果该函数什么都不做，仅仅只是调用一个现有的方法时，就可以使用方法的名字来代表该方法的调用。

<br>

方法引用即是用现有方法的名字，来紧凑的表达一个易于阅读的Lambda表达式。

<br>

方法引用使用两个冒号 **::** 表示


<br><br>


### 方法引用种类：

| 种类                             | 句法                                 | 例子                                                        |
| -------------------------------- | ------------------------------------ | ----------------------------------------------------------- |
| 引用静态方法                     | ContainingClass::staticMethodName    | Person::compareByAgeMethodReferencesExamples::appendStrings |
| 引用类的实例对象的方法           | containingObject::instanceMethodName | myComparisonProvider::compareByNamemyApp::appendStrings2    |
| 引用特殊类型的任意对象的实例方法 | ContainingType::methodName           | String::compareToIgnoreCaseString::concat                   |
| 引用构造函数                     | ClassName::new                       | HashSet::new                                                |

<br>
下面的代码是Oracle文档的一段代码实例：

```java
import java.util.function.BiFunction;


public class MethodReferencesExamples {
    
    public static <T> T mergeThings(T a, T b, BiFunction<T, T, T> merger) {
        return merger.apply(a, b);
    }
    
    public static String appendStrings(String a, String b) {
        return a + b;
    }
    
    public String appendStrings2(String a, String b) {
        return a + b;
    }


    public static void main(String[] args) {
        
        MethodReferencesExamples myApp = new MethodReferencesExamples();


        // Calling the method mergeThings with a lambda expression
        System.out.println(MethodReferencesExamples.
            mergeThings("Hello ", "World!", (a, b) -> a + b));
        
        // Reference to a static method
        System.out.println(MethodReferencesExamples.
            mergeThings("Hello ", "World!", MethodReferencesExamples::appendStrings));


        // Reference to an instance method of a particular object        
        System.out.println(MethodReferencesExamples.
            mergeThings("Hello ", "World!", myApp::appendStrings2));
        
        // Reference to an instance method of an arbitrary object of a
        // particular type
        System.out.println(MethodReferencesExamples.
            mergeThings("Hello ", "World!", String::concat));
   
```

<br><br>

**结语**：花费了大概一周的空闲时间，终于对函数式编程有了初步的了解，接下来，将会继续学习函数式编程在Java中的实际应用：Stream流式操作。也希望能在实际的编码中能写出优美、可读性强且可维护性高的函数式编程语句，加油！

当然，如果文中有错误，也希望能即时提醒。


<br><br>

------

## 参考

[Lambda Expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)

[Functional Programming in Java](https://www.baeldung.com/java-functional-programming)

[Functional programming](https://en.wikipedia.org/wiki/Functional_programming)
[Method References](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)

[Any difference between First Class Function and High Order Function](https://stackoverflow.com/questions/10141124/any-difference-between-first-class-function-and-high-order-function)

[first-class functions：函数是一等公民](https://blog.csdn.net/weixin_30849591/article/details/95599961)

[Higher-order_function](https://en.wikipedia.org/wiki/Higher-order_function)

[First-class_function](https://en.wikipedia.org/wiki/First-class_function)

[Side effect](https://en.wikipedia.org/wiki/Side_effect_(computer_science))

