# Cloneable


## 官方描述

A class implements the Cloneable interface to indicate to the Object.clone() method that it is legal for that method to make a field-for-field copy of instances of that class.
Invoking Object's clone method on an instance that does not implement the Cloneable interface results in the exception CloneNotSupportedException being thrown.

By convention, classes that implement this interface should override Object.clone (which is protected) with a public method. See Object.clone() for details on overriding this method.

Note that this interface does not contain the clone method. Therefore, it is not possible to clone an object merely by virtue of the fact that it implements this interface. 
Even if the clone method is invoked reflectively, there is no guarantee that it will succeed.

[Interface Cloneable](https://docs.oracle.com/javase/8/docs/api/java/lang/Cloneable.html)

<br>

一个类通过实现Cloneable接口，使得该类在调用[Object.clone()](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#clone--)方法生成实例副本时，不会抛出异常。
在没有实现 Cloneable接口的实例上调用Object.clone()方法会抛出**CloneNotSupportedException**异常。

按照惯例，实现此接口的类应该重载Object.clone()方法（protected）,并将方法改为public。

请注意，此接口不包含clone方法。因此，不可能仅凭借实现该接口的事实来克隆对象。即使以反射方式调用 clone 方法，也不能保证它会成功。


<br>

## stackoverflow

The first thing you should know about Cloneable is - don't use it.

It is very hard to implement cloning with Cloneable right, and the effort is not worth it.

Instead of that use some other options, like apache-commons [SerializationUtils](https://commons.apache.org/proper/commons-lang/javadocs/api-3.4/org/apache/commons/lang3/SerializationUtils.html) (deep-clone) 
or [BeanUtils](https://commons.apache.org/proper/commons-beanutils/) (shallow-clone), or simply use a copy-constructor.

[How does Cloneable work in Java and how do I use it?](https://stackoverflow.com/questions/4081858/how-does-cloneable-work-in-java-and-how-do-i-use-it)


<br>




不推荐使用Cloneable，它的利用效率和开发成本并不匹配，推荐使用SerializationUtils（深拷贝）类或BeanUtils（浅拷贝）替代，或者直接为类添加一个用于复制字段的构造器。


<br>

## Josh Bloch on Design

Copy Constructor versus Cloning
Bill Venners: In your book you recommend using a copy constructor instead of implementing Cloneable and writing clone. Could you elaborate on that?

Josh Bloch: If you've read the item about cloning in my book, especially if you read between the lines, you will know that I think clone is deeply broken. 
There are a few design flaws, the biggest of which is that the Cloneable interface does not have a clone method. 
And that means it simply doesn't work: making something Cloneable doesn't say anything about what you can do with it. Instead, it says something about what it can do internally. 
It says that if by calling super.clone repeatedly it ends up calling Object's clone method, this method will return a field copy of the original.

But it doesn't say anything about what you can do with an object that implements the Cloneable interface, which means that you can't do a polymorphic clone operation. 
If I have an array of Cloneable, you would think that I could run down that array and clone every element to make a deep copy of the array, but I can't. 
You cannot cast something to Cloneable and call the clone method, because Cloneable doesn't have a public clone method and neither does Object. 
If you try to cast to Cloneable and call the clone method, the compiler will say you are trying to call the protected clone method on object.

The truth of the matter is that you don't provide any capability to your clients by implementing Cloneable and providing a public clone method other than the ability to copy. 
This is no better than what you get if you provide a copy operation with a different name and you don't implement Cloneable. That's basically what you're doing with a copy constructor. 
The copy constructor approach has several advantages, which I discuss in the book. One big advantage is that the copy can be made to have a different representation from the original. 
For example, you can copy a LinkedList into an ArrayList.

Object's clone method is very tricky. It's based on field copies, and it's "extra-linguistic." It creates an object without calling a constructor. 
There are no guarantees that it preserves the invariants established by the constructors. There have been lots of bugs over the years, 
both in and outside Sun, stemming from the fact that if you just call super.clone repeatedly up the chain until you have cloned an object, 
you have a shallow copy of the object. The clone generally shares state with the object being cloned. If that state is mutable, you don't have two independent objects. 
If you modify one, the other changes as well. And all of a sudden, you get random behavior.

There are very few things for which I use Cloneable anymore. I often provide a public clone method on concrete classes because people expect it. 
I don't have abstract classes implement Cloneable, nor do I have interfaces extend it, 
because I won't place the burden of implementing Cloneable on all the classes that extend (or implement) the abstract class (or interface). It's a real burden, with few benefits.

Doug Lea goes even further. He told me that he doesn't use clone anymore except to copy arrays. 
You should use clone to copy arrays, because that's generally the fastest way to do it. But Doug's types simply don't implement Cloneable anymore. 
He's given up on it. And I think that's not unreasonable.

It's a shame that Cloneable is broken, but it happens. The original Java APIs were done very quickly under a tight deadline to meet a closing market window. 
The original Java team did an incredible job, but not all of the APIs are perfect. Cloneable is a weak spot, and I think people should be aware of its limitations.

[A Conversation with Effective Java Author, Josh Bloch](https://www.artima.com/articles/josh-bloch-on-design#part13)