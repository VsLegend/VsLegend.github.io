# 序列化(Serializable)
参考：[Serializable Objects](https://docs.oracle.com/javase/tutorial/jndi/objects/serial.html)

## 序列化定义：
序列化对象就是将其状态由Java对象变为二进制流(byte stream)。反序列化就是将对象的序列化形式转换回该对象的副本的过程，即将序列化后的二进制流转为Java对象。

## Java中实现序列化：
Java对象要实现实例化，即这个Java类或其父类实现了接口java.io.Serializable（或其子接口java.io.Externalizable）。

## 序列化的作用：
序列化后可以把byte[]保存到文件中，或者把byte[]通过网络传输到远程。这样，就相当于把Java对象存储到文件或者通过网络传输出去了。 即实现Java对象的数据持久化和网络传输。

>注意：反序列化时，由JVM直接构造出Java对象，不调用构造方法，构造方法内部的代码，在反序列化时根本不可能执行。