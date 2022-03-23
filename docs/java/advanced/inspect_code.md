# FindBugs代码检测与优化

[FindBugs Bug Descriptions](http://findbugs.sourceforge.net/bugDescriptions.html)

## Efficiency


### WMI
>WMI: XXX makes inefficient use of keySet iterator instead of entrySet iterator

You are retrieving all the keys (accessing the whole map) and then for some keys, you access the map again to get the value.

You can iterate over the map to get map entries (Map.Entry) (couples of keys and values) and access the map only once.

Map.entrySet() delivers a set of Map.Entrys each one with the key and corresponding value.

```java
for ( Map.Entry< String, LIMSGridCell > entry : cellsMap.entrySet() ) {
    if ( entry.getKey().startsWith( columnIndex ) ) {
        cells.add( entry.getValue() );
    }
}
```

Note: I doubt that this will be much of an improvement since if you use map entries you will instantiate an object for each entry. 
I don't know if this is really faster than calling get() and retrieving the needed reference directly.



### SS: Unread field
>SS: Unread field: XXX; should this field be static?

They must be static if they belong to the class, and not be static if they belong to the instance of the class:
```java
public class ImmutablePerson {
    private static final int MAX_LAST_NAME_LENGTH = 255; // belongs to the type
    private final String firstName; // belongs to the instance
    private final String lastName; // belongs to the instance

    public ImmutablePerson(String firstName, String lastName) {
        if (lastName.length() > MAX_LAST_NAME_LENGTH) {
            throw new IllegalArgumentException("last name too large");
        }
        this.firstName = firstName;
        this.lastName = lastName;
    }

    // getters omitted for brevity
}
```

- static and final are entirely different things. static means that the field relates to the type rather than any particular instance of the type. 

- final means that the field can't change value after initial assignment (which must occur during type/instance initialization).

- static final fields are usually for constants - whereas instance fields which are final are usually used when creating immutable types.



### SBSC
>SBSC: XXX concatenates strings using + in a loop

You can create StringBuilder outside the for loop and reuse it instead of String +.
```java
StringBuilder sb=new StringBuilder();
for (final FieldError fieldError : result.getFieldErrors()) {
      sb.append(fieldError.getField())
           .append(" - ")
           .append(getErrorMessageFromProperties(fieldError.getCode()))
           .append("*");
 } 
```


### UrF: Unread field
>UrF: Unread field: XXX

字段未使用，建议删除或注释



### Performance - Method concatenates strings using + in a loop

不能用for循环拼接字符串，由于字符串类String实例是一个不可变对象，每次拼接都会生成新对象，浪费内存。使用StringBuffer (or StringBuilder in Java 1.5)代替。

```java
// This is bad
  String s = "";
  for (int i = 0; i < field.length; ++i) {
    s = s + field[i];
  }

  // This is better
  StringBuffer buf = new StringBuffer();
  for (int i = 0; i < field.length; ++i) {
    buf.append(field[i]);
  }
  String s = buf.toString();

```


### Performance - Private method is never called

私有方法未被调用


<br><br>

---

## Maintainability
### CN 
>CN: Class XXX implements Cloneable but does not define or use clone method

实现了Cloneable，但是没有重载Object.clone()方法，或没有在实例中使用该方法，建议删除Cloneable


### Nm 
>Nm: The class name XXX shadows the simple name of the superclass XXX

类名名称掩盖了超类的简单名称，建议修改子类名称


### ES 
>ES: Comparison of String objects using == or != in XXX

比较字符串时，使用了== 或 !=，需要使用equals方法



### RV 
>RV: Exceptional return value of java.io.File.mkdirs() ignored in XXX

File.mkdirs()仅当创建了目录以及所有必要的父目录时才为true，否则false。因此必须处理false时的情况，以保证稳定性



### RV 
>RV: Exceptional return value of java.io.File.mkdirs()/File.delete() ignored in XXX

File.mkdirs()仅当创建了目录以及所有必要的父目录时才为true，否则false。因此必须处理false时的情况，以保证稳定性








<br><br>

---

## Reliability

### Eq
>Eq: XXX overrides equals in XXX and may not be symmetric

当类定义了一个 equals 方法，该方法覆盖了超类中的 equals 方法。 如果两种equals方法方法都使用instanceof来判断两个对象是否相等， 那么这将充满了危险。
因为 equals 方法是对称的（换句话说，a.equals(b) == b.equals(a)）很重要。 
如果 B 是 A 的子类型，并且 A 的 equals 方法检查参数是否为 A 的 instanceof，而 B 的 equals 方法检查参数是否为 B 的 instanceof，则很可能这些方法定义的等价关系不是对称的。

Lombok中可以通过@EqualsAndHashCode(callSuper = true)设置关联超类判断


### ICAST
>ICAST: Integral value cast to double and then passed to Math.ceil/Math.round  in XXX

此代码将整数值（例如 int 或 long）转换为双精度浮点数，然后将结果传递给 Math.ceil() 函数，该函数将 double 舍入为下一个更高的整数值。 

此操作应始终为无操作，因为将整数转换为双精度应给出没有小数部分的数字。 生成要传递给 Math.ceil 的值的操作很可能打算使用双精度浮点运算来执行。


### USELESS_STRING: Invocation of toString on args
> USELESS_STRING: Invocation of toString on args in XXX

需要注意的是，数组直接使用toString方法，将会返回数组的内存地址，而非数组内容，考虑使用Arrays.toString(args)来获取数组内容


### NP: Non-virtual method call
> NP: Non-virtual method call in XXX

可能为 null 的值被传递给非 null 方法参数。 要么将参数注释为应始终为非空的参数，要么分析表明它将始终被取消引用。


### MS: mutable array
> MS: XXX is a mutable array

当你想让数组无法修改时，final static这两个修饰词并不能完成该目的，即使数组被他们修饰，但是仍然可以被恶意代码或意外从另一个包中访问。 因为反射或别的原因，使得仍然能自由修改数组的内容。

可以使用如下方法：
```java
List<Integer> items = Collections.unmodifiableList(Arrays.asList(0,1,2,3));
```

### MS: isn't final but should be
> MS: XXX.xxx isn't final but should be



### MS: should be package protected
> MS: XXX.xxx should be package protected

可变静态字段可能被恶意代码或意外更改。 可以对该字段进行包保护以避免此漏洞。




### EI
> EI: com.fanmei.custom.domain.dto.excel.AdmissionImportDTO.getBirthday() may expose internal representation by returning AdmissionImportDTO.birthday

返回对存储在对象字段之一中的可变对象值的引用会公开对象的内部表示。 
如果实例由不受信任的代码访问，并且对可变对象的未经检查的更改会危及安全性或其他重要属性，则您将需要做一些不同的事情。 
在许多情况下，返回对象的新副本是更好的方法。


### EI2
> EI2: XXX may expose internal representation by storing an externally mutable object into XXX

此代码将对外部可变对象的引用存储到对象的内部表示中。 
如果实例由不受信任的代码访问，并且对可变对象的未经检查的更改会危及安全性或其他重要属性，则您将需要做一些不同的事情。 
在许多情况下，存储对象的副本是更好的方法。









<br><br>

---

## Usability
### DLS
> DLS: Dead store to e in XXX 

该指令为局部变量赋值，但在任何后续指令中都不会读取或使用该值。 
这其实是一种错误的使用，因为从未使用过计算的值。

请注意，Sun 的 javac 编译器通常会为最终局部变量生成死存储。 
因为 FindBugs 是一个基于字节码的工具，所以没有简单的方法可以消除这些误报。

考虑真正使用时才设置这些局部变量。


### DB
> DB: com.fanmei.common.utils.file.FileUtils.setFileDownloadHeader(HttpServletRequest, String) uses the same code for two branches 

该方法中的两个分支使用了相同的代码

考虑缩减分支


### RCN
> RCN: Redundant nullcheck of XXX, which is known to be non-null in XXX

此方法包含对已知非空值与常量空值的冗余检查。



# 阿里代码检测


## Critical

### Object的equals方法容易抛空指针异常，应使用常量或确定有值的对象来调用

如下，str作为入参，可能为null
```java
    public void f(String str){
        String inner = "hi";
        if (inner.equals(str)) {
            System.out.println("hello world");
        }
    }
```


## Major

### 不允许任何魔法值（即未经定义的常量）直接出现在代码中

代码首先是给人读的，只是恰好可以执行。

而魔法值随意出现，主要是影响可读性和代码可维护性。


### 集合初始化时，指定集合的初始值大小

此规范的主要目的完全是出于性能考虑，查看 HashMap 的源码也就可以发现此规范的原因，如果我们能为集合设置合理的大小就可以避免 HashMap 的扩容操作，而 HashMap 的扩容方法 resize 有很多逻辑判断和业务操作，如果设置了合理的大小就可以避免执行更多的代码，因此就可以更大限度的提高集合的执行效率。

在初始化集合时，如果已知集合的数量，那么一定要在初始化时设置集合的容量大小，这样就可以有效的提高集合的性能，但需要注意的是 HashMap 的实际存储量是“元素个数*负载因子”，而负载因子默认是 0.75，因此在设置大小时，要使用“(存储元素个数/负载因子)+1”的公式计算出正确的值再进行设置。


