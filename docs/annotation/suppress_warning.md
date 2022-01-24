# @SuppressWarnings

**作用**：抑制编译器警告信息。当我们面对一些无法处理或不能处理的警告时，我们可以取消该警告。

**使用对象**:TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE

```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```

@SuppressWarnings注解中的属性value用于设置要抑制的类型。警告类型可能因编译器供应商而异

**常见警告类型作用如下**：
- deprecation：编译器忽略使用了@Deprecation注解的类或方法
- unchecked：编译器忽略原始类型（raw types）
>unchecked警告：
>
>当编译器并不能保证类型安全时，就会发出unchecked警告。
>是指编译器和运行时系统没有足够的类型信息来执行确保类型安全所必需的所有类型检查。
>unchecked警告的最常见来源是使用原始类型。当通过原始类型变量访问对象时会发出“unchecked”警告，因为原始类型没有提供足够的类型信息来执行所有必要的类型检查。 


**使用方式**：
```java
@SuppressWarnings("unchecked")
public void method() {}
```
```java
@SuppressWarnings({"unchecked", "deprecated"})
public void method() {}
```


<br>

---

[Java @SuppressWarnings Annotation](https://www.baeldung.com/java-suppresswarnings)

[What is SuppressWarnings ("unchecked") in Java?](https://stackoverflow.com/questions/1129795/what-is-suppresswarnings-unchecked-in-java)

[What is an "unchecked" warning?](http://www.angelikalanger.com/GenericsFAQ/FAQSections/TechnicalDetails.html#FAQ001)

[What is a raw type and why shouldn't we use it?](https://stackoverflow.com/questions/2770321/what-is-a-raw-type-and-why-shouldnt-we-use-it)