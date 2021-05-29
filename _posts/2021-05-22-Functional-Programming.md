---
title:  "函数式编程"
author: WangJwi
categories:
- Java
tags:
- Java
- Lambda
- Stream
- functional programming
---

导读：函数式编程，或称函数程序设计、泛函编程，是一种编程范式，它将电脑运算视为函数运算，并且避免使用程序状态以及易变对象。其中，λ演算为该语言最重要的基础。

------

## 函数式编程：Functional programming

### 什么是函数式编程

函数式编程，或称函数程序设计、泛函编程，是一种编程范式，它将电脑运算视为函数运算，并且避免使用程序状态以及易变对象。其中，λ演算为该语言最重要的基础。而且，λ演算的函数可以接受函数作为输入参数和输出返回值。



比起指令式编程，函数式编程更加强调程序执行的结果而非执行的过程，倡导利用若干简单的执行单元让计算结果不断渐进，逐层推导复杂的运算，而不是设计一个复杂的执行过程。

在函数式编程中，函数是第一类对象，意思是说一个函数，既可以作为其它函数的输入参数值，也可以从函数中返回值，被修改或者被分配给一个变量。



### 函数式接口

**定义：**

要定义一个函数式接口，需要满足以下两种情况

1. 使用@FunctionalInterface注解，标注接口为函数式接口
2. 接口内部必须只有一个或以下的普通接口方法。可以有任意数量的接口默认方法和接口静态方法，这些都不影响函数式接口的定义。



**说明：**

函数式接口实际上只是函数式编程的定义，它将作为方法参数，被传给其他方法，而实现这些接口的方法一般被表现为lambda表达式（当然你也可以用一般接口的形式来实现函数式接口，也可以通过匿名内部类来实现，但lambda表达式实际上就是为了简化上述情况而产生的）。



常用的函数式接口：

**Function**：接受一个参数，并返回一个结果

``` java
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



**Predicate**：接受一个参数，返回Boolean

``` java
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





**Supplier**：无参返回一个结果。该函数可以接受无参构造方法、无参方法来返回一个结果

``` java
@FunctionalInterface
public interface Supplier<T> {


    // 无参返回一个结果
    T get();
}
```



**Consumer**：接受一个参数，并处理，无返回结果

``` java
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



实例：

这一段内容只做简单介绍，在后文讲解过lambda表达式后，将使用这些函数式接口来实现一些功能。

``` java
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
        if (predicate.test(t)) {
            return function.apply(t);
        } else {
            System.err.println("密码错误！");
        }
        return null;
    }
}
```





### Lambda表达式：Lambda Expressions

概念：

即是一个**匿名函数**，即没有函数名的函数。Lambda表达式可以让我们不用实现函数式接口，而是通过函数体完成其接口方法。Lambda是实现回调函数（callback function）的一种方式。



结构：

（输入 +） 函数体 （+ 输出）

``` java
// 输入(String a)   
// "->" 符号后的部分都可以当作函数体 
// 输出一个String类型的变量upper
(String a) -> {
    String upper = a.toUpperCase();
    System.out.println(upper);
    return upper;
}
```



语法规则：

1. 声明类型、不声明类型

``` java
(int a, int b) -> a + b


(a, b) -> a - b


// 前面两个表达式含义相同，相当于下面的方法
int sum(int a, int b) {
  return a +b;
```



注意：单个参数需要声明类型时，必须使用“(” “ )”圆括号。

``` java
// 一个入参
(List<String> l) -> {
  l.forEach(element -> System.out.println(element));
}
```





1. 有输入参数、没有输入参数

``` java
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



1. 单条执行语句、多条执行语句

``` java
() -> "lambda".toUpperCase()


() -> {
    // other statement
    return "lambda".toUpperCase();
}
```

注意：单个语句可以省去 { }



1. 带有输出的Lambda表达式、不带有输出

``` java
// 有返回值
(List<String> list) -> {
    return list;
}


(List<String> list) -> list




// 无返回值
(List<String> list) -> {
    list.clear();
}


(List<String> list) -> list.clea
```

注意：

1. 当函数体只有一条且为输出语句时，需要确保简写表达式语句有返回值
2. 当无返回值的表达式只有一条时，需要确保简写表达式不会有返回值



### 方法引用：Method Reference

定义：

当使用一个Lambda表达式创建一个匿名函数时，如果该函数什么都不做，仅仅只是调用一个现有的方法时，就可以使用方法的名字来代表该方法的调用会让程序更为清晰，方法引用便可以让你实现此功能。



方法引用即使用现有方法的名字，来紧凑的表达一个易于阅读的Lambda表达式。



方法引用使用两个冒号 :: 表示





方法引用种类：

| 种类                             | 句法                                 | 例子                                                        |
| -------------------------------- | ------------------------------------ | ----------------------------------------------------------- |
| 引用静态方法                     | ContainingClass::staticMethodName    | Person::compareByAgeMethodReferencesExamples::appendStrings |
| 引用类的实例对象的方法           | containingObject::instanceMethodName | myComparisonProvider::compareByNamemyApp::appendStrings2    |
| 引用特殊类型的任意对象的实例方法 | ContainingType::methodName           | String::compareToIgnoreCaseString::concat                   |
| 引用构造函数                     | ClassName::new                       | HashSet::new                                                |

下面的代码是Oracle的一段代码实例：

``` java
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



### 流式操作：Stream



参考：

[java 8 Stream](https://www.logicbig.com/how-to/code-snippets/jcode-java-8-streams-stream-collect.html)

[Lambda Expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)

[Functional Programming in java](https://www.baeldung.com/java-functional-programming)

[Functional programming](https://en.wikipedia.org/wiki/Functional_programming)