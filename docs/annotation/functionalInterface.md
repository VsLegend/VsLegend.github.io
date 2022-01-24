# @FunctionalInterface

**作用**：只是一种信息标注，将标注接口为函数式接口，函数式接口的实例可以使用 lambda 表达式、方法引用或构造函数引用来创建。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FunctionalInterface {}
```

**使用条件**：

- 该类型是接口类型，而不是注释类型、枚举或类。

- 带注释的类型满足功能接口的要求，即只有一个抽象方法。


**使用方法**：
以Runnable接口为例：
```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

通过Lambda表达式实现匿名函数的使用。
```java
new Thread(() -> {
    System.out.println(Thread.currentThread().getName());
})
```