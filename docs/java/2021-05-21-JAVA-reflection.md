
# java反射机制
## 反射原理：
java在编译之后，会将Java代码生成为class源文件，JVM启动时，将会载入所有的源文件，并将**类型信息**存放到**方法区**中，将所有**对象实例**存放在**Java堆**中。

- 对于获取或创建新的类型实例：反射是在运行时，通过读取方法区中的字节码，来动态的找到其反射的类或类的方法和属性等（实际上就是在运行时，根据全类型名在方法区找对应的类），以实现类型的检查或创建该类的实例对象。
- 对于修改或获取存在的实例对象：一般来说，我们不通过反射构建的实例对象，通过编译器后都能预先的知道该实例对象有哪些属性和方法，从而可以直接获取或调用方法或属性。
    而反射则不同，由于是运行时进行操作，它没法知道反射的这个实例对象有哪些属性和方法，因此需要先获取该对象的类型信息，从而通过该类型信息的属性或方法来修改实例对象。


## 反射的用途：
反射功能通常用于检查或修改java虚拟机运行中（runtime）的应用程序的行为。反射是一种强大的技术，可以运行原本不可能的操作。

- 在运行中分析类的能力，可以通过完全限定类名创建类的对象实例。
- 在运行中查看和操作对象，可以遍历类的成员变量。
- 反射允许代码执行非反射代码中非法的操作，可以检索和访问类的私有成员变量，包括私有属性、方法等。

注意：要有选择的使用反射功能，如果可以直接执行操作，那么最好不要使用反射。


## 反射的缺点：

- 额外的性能开销（**Performance Overhead**）：由于反射涉及动态类型的解析，它无法执行某些java虚拟机优化，因此反射操作的性能通常要比非反射操作慢。
- 安全限制（**Security Restrictions**）：反射需要运行时操作权限，此操作可能在一些安全管理器下不被允许。
- 内部泄露（**Exposure of Internals**）：由于反射允许代码执行非反射代码中非法的操作（例如访问私有字段和方法），因此使用反射可能会导致意外的副作用，这可能会使代码无法正常工作并可能破坏可移植性。反射性代码破坏了抽象，因此可能会随着平台的升级而改变行为。


## 获取对象类的方式:

1. **Object.getClass()**。从一个实例对象中获取它的类。这仅适用于继承自Object的引用类型（当然java的类默认继承于Object）。

``` java
Map<String, String> hashMap = new HashMap<>();
Class<? extends Map> aClass = hashMap.getClass();
String text = "text";
Class<? extends String> aClass1 = text.getClass();
```



``` java
// Object类
public final native Class<?> getClass();
```



2. **XXX.class**。直接从未实例化的类获取类。

``` java
Class<Integer> integerClass = int.class;
Class<HashMap> hashMapClass = HashMap.class;
```



3. **Class.forName()**。通过完全限定类名获取类。即包名加类名（java.util.HashMap）。否则会报找不到类错误。

``` java
Class<HashMap> hashMapClass = Class.forName("java.util.HashMap");
```



``` java
// class类
public static Class<?> forName(String className)
            throws ClassNotFoundException {
    Class<?> caller = Reflection.getCallerClass();
    return forName0(className, true, ClassLoader.getClassLoader(caller), caller);
}
```



4. **Integer.TYPE**。基本类型的包装类通过TYPE获取类。都是java早期版本的产物，已过时。

``` java
// Integer
@SuppressWarnings("unchecked")
public static final Class<Integer>  TYPE = (Class<Integer>) Class.getPrimitiveClass("int");


// Double
@SuppressWarnings("unchecked")
public static final Class<Double>   TYPE = (Class<Double>) Class.getPrimitiveClass("double");
```



5. 通过反射类ClassAPI获取类。注意，只有在已经直接或间接获得一个类的情况下，才可以访问这些API。

``` java
try {
  Class<?> className = Class.forName("java.lang.String");
  // 获取父类
  Class<?> superclass = className.getSuperclass();
  // 返回调用类的成员变量，包括所有公共的类、接口和枚举
  Class<?>[] classes = className.getClasses();
  // 返回调用类的依赖，包括所有类、接口和显式声明的枚举
  Class<?>[] declaredClasses = className.getDeclaredClasses();
} catch (ClassNotFoundException e) {
  e.printStackTrace();
}
```



## 获取类的成员变量：

获取字段：

| [Class](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html) API | 是否是列表 | 是否获取父类属性 | 能否能获取私有字段 |
| ------------------------------------------------------------ | ---------- | ---------------- | ------------------ |
| [getDeclaredField()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredField-java.lang.String-) | no         | no               | yes                |
| [getField()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getField-java.lang.String-) | no         | yes              | no                 |
| [getDeclaredFields()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredFields--) | yes        | no               | yes                |
| [getFields()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getFields--) | yes        | yes              | no                 |



获取方法：

| [Class](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html) API | List of members? | Inherited members? | Private members? |
| ------------------------------------------------------------ | ---------------- | ------------------ | ---------------- |
| [getDeclaredMethod()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredMethod-java.lang.String-java.lang.Class...-) | no               | no                 | yes              |
| [getMethod()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getMethod-java.lang.String-java.lang.Class...-) | no               | yes                | no               |
| [getDeclaredMethods()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredMethods--) | yes              | no                 | yes              |
| [getMethods()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getMethods--) | yes              | yes                | no               |



获取构造器：

| [Class](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html) API | List of members? | Inherited members? | Private members? |
| ------------------------------------------------------------ | ---------------- | ------------------ | ---------------- |
| [getDeclaredConstructor()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredConstructor-java.lang.Class...-) | no               | N/A1               | yes              |
| [getConstructor()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getConstructor-java.lang.Class...-) | no               | N/A1               | no               |
| [getDeclaredConstructors()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredConstructors--) | yes              | N/A1               | yes              |
| [getConstructors()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getConstructors--) | yes              | N/A1               | no               |



## java.lang.reflect.Field

Field字段具有类型和值。Field提供访问属性对象类型信息的方法；以及获取和设置字段值的方法。



**获取字段类型：**

字段可以是原始类型或引用类型。

有八种基本类型：boolean，byte，short，int，long，char，float，和double。

引用类型是java.lang.Object类的直接或间接子类，包含接口，数组和枚举类型等 。



**获取字段修饰符：**

- 访问修饰符：public，protected，和private
- 仅用于字段的控制运行时行为的修饰符：transient和volatile
- 限制单实例的修饰符： static
- 禁止值修改的修饰符： final
- 注解



``` java
Class<?> className = Class.forName("java.util.HashMap");
Field table = className.getDeclaredField("table");
// 获取属性的名字
String name = table.getName();
// 获取属性的类型
Class<?> type = table.getType();
// 获取修饰符
int modifiers = table.getModifiers();
System.out.println(Modifier.toString(modifiers));
// 获取注解
Override annotation = table.getDeclaredAnnotation(Override.class);
Annotation[] declaredAnnotations = table.getDeclaredAnnotations();
```



**获取和设置字段值：**

给定一个类的实例，可以使用反射来设置该类中字段的值。通常仅在特殊情况下无法以常规方式设置值时才执行此操作。因为这样的访问通常会违反该类的设计意图，所以应绝对谨慎地使用它。



**注意**：通过反射设置字段的值会有一定的性能开销，因为必须进行各种操作，例如验证访问权限。从运行时的角度来看，效果是相同的，并且操作是原子的，就好像直接在类代码中更改了值一样。除此之外，反射会破坏java原本的设定，列如可以重新设置final属性的值等。



**反射修改final修饰的属性值到JVM对String的优化：**

反射功能强大，能修改private以及final修饰的变量。如下代码中，展示了JVM的优化以及反射的一些劣势。

``` java
@Data
public class FieldReflectDemo {
  // 引用直接指向常量池中的常量值
  private final String constantStr = "FinalConstantStringField";
  // JVM优化了getter方法，直接将对constantStr引用全部替换成了常量
//  public String getConstantStr() {return "FinalConstantStringField";}


  // 在堆中新建了一个对象
  private final String newStr = new String("FinalNewStringField");
  
  public FieldReflectDemo(){}
    
    public static void main(String[] args) {
    FieldReflectDemo fieldReflectDemo = new FieldReflectDemo();
    try {
      Class<?> className = fieldReflectDemo.getClass();
      Field constantStr = className.getDeclaredField("constantStr");
      Field newStr = className.getDeclaredField("newStr");
      // 获取实例对象的字段值
      System.out.println("constantStr原：" + constantStr.get(fieldReflectDemo));
      System.out.println("newStr原：" + newStr.get(fieldReflectDemo));
      constantStr.setAccessible(true);
      newStr.setAccessible(true);
      constantStr.set(fieldReflectDemo, "New Filed Name");
      newStr.set(fieldReflectDemo, "New Filed Name");
      System.out.println("constantStr反射修改：" + constantStr.get(fieldReflectDemo));
      System.out.println("newStr反射修改：" + newStr.get(fieldReflectDemo));
    } catch (NoSuchFieldException | IllegalAccessException e) {
      e.printStackTrace();
    }
    System.out.println("constantStr实例对象值：" + fieldReflectDemo.getConstantStr());
    System.out.println("newStr实例对象值：" + fieldReflectDemo.getNewStr());
  }
  
  /**
   * 输出
   * constantStr原：FinalConstantStringField
   * newStr原：FinalNewStringField
   * constantStr反射修改：New Filed Name
   * newStr反射修改：New Filed Name
   * constantStr实例对象值：FinalConstantStringField
   * newStr实例对象值：New Filed Name
   */
}
```

因为JVM在编译时期, 就把final类型的**直接赋值的String**进行了优化, 在编译时期就会把String处理成常量。反射成功将其值修改成功了，但是在它的get方法中，返回的不是当前变量，而是返回JVM优化好的一个常量值。



## java.lang.reflect.Method

Method方法具有参数和返回值，并且方法可能抛出异常。Method提供获取参数信息、返回值的方法；它也可以调用（invoke）给定对象的方法。



**获取方法类型的信息：**

方法声明包含了方法名、修饰符、参数、返回类型以及抛出的多个异常。

``` java
public class MethodReflectDemo {


public MethodReflectDemo() {
  
private void getNothing(String name) {
  
public int getNumByName(String name) throws NullPointerException {
  if (StringUtils.isEmpty(name))
    throw new NullPointerException("名字为空");
  return name.length();
}


  public static void main(String[] args) {
    MethodReflectDemo methodReflectDemo = new MethodReflectDemo();
    try {
      Class<? extends MethodReflectDemo> demoClass = methodReflectDemo.getClass();
      Method method = demoClass.getDeclaredMethod("getNumByName", String.class);
      String name = method.getName();
      System.out.println("方法名：" + name);
      // 修饰符
      int modifiers = method.getModifiers();
      System.out.println("所有修饰符：" + Modifier.toString(modifiers));
      // 参数
      Parameter[] parameters = method.getParameters();
      // 返回类型
      Class<?> returnType = method.getReturnType();
      System.out.println("返回类型：" + returnType.getTypeName());
      // 异常
      Class<?>[] exceptionTypes = method.getExceptionTypes();
      System.out.println("");
      // 实例对象调用方法
      Object invoke = method.invoke(methodReflectDemo, "名称");
      System.out.println(invoke);
    } catch (NoSuchMethodException e) {
      e.printStackTrace();
    }
  }
```



## java.lang.reflect.Constructor

Constructor与Method相似，但有几点不同：
- 构造函数没有返回值
- 构造函数无法被实例对象执行，它的调用只能为给定的类创建对象的新实例。


``` java

public class ConstructorReflectDemo {

  public ConstructorReflectDemo() {}

  private void getNothing(String name) { }

  public int getNumByName(String name) throws NullPointerException {
    if (StringUtils.isEmpty(name))
      throw new NullPointerException("名字为空");
    return name.length();
  }

  public static void main(String[] args) {
    ConstructorReflectDemo methodReflectDemo = new ConstructorReflectDemo();
    try {
      Class<? extends ConstructorReflectDemo> demoClass = methodReflectDemo.getClass();
      Constructor<? extends ConstructorReflectDemo> constructor = demoClass.getConstructor();
      String name = constructor.getName();
      System.out.println("构造方法名：" + name);
      // 修饰符
      int modifiers = constructor.getModifiers();
      System.out.println("所有修饰符：" + Modifier.toString(modifiers));
      // 参数
      Parameter[] parameters = constructor.getParameters();
      // 异常
      Class<?>[] exceptionTypes = constructor.getExceptionTypes();
      System.out.println("");
      // 构造方法无法被调用，只可以创建新实例
      ConstructorReflectDemo constructorReflectDemo = constructor.newInstance();
    } catch (NoSuchMethodException | IllegalAccessException | InstantiationException | InvocationTargetException e) {
      e.printStackTrace();
    }
  }

}
```



参考：

[The Reflection API](https://docs.oracle.com/javase/tutorial/reflect/index.html)
