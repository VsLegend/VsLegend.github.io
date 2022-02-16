# 字符串相关工具类

## StringJoiner-连接字符串
**使用方法**
```java
StringJoiner joiner = new StringJoiner(",", PREFIX, SUFFIX);
joiner.add("Red")
  .add("Green")
  .add("Blue");

String result = joiner.toString();
```

<br>

**遍历列表或数组对象**
```java
List<String> list = Arrays.asList("Red", "Green", "Blue");
for (String color : list) {
    joiner.add(color);
}
```

<br>

**流式操作**
Collectors.joining()在内部使用StringJoiner来执行加入操作
```java
List<String> list = Arrays.asList("Red", "Green", "Blue");
String result = list.stream()
  .map(color -> color.toString())
  .collect(Collectors.joining(","));
```


<br>

**源码**
```java
public final class StringJoiner {
    private final String prefix;// 前缀
    private final String delimiter;// 分隔符
    private final String suffix;// 后缀

    private StringBuilder value;// 实际使用StringBuilder连接的字符串，因此也不是并发安全的

    private String emptyValue;// 默认值

    public StringJoiner(CharSequence delimiter) {
        this(delimiter, "", "");
    }

    public StringJoiner(CharSequence delimiter,
                        CharSequence prefix,
                        CharSequence suffix) {
        Objects.requireNonNull(prefix, "The prefix must not be null");
        Objects.requireNonNull(delimiter, "The delimiter must not be null");
        Objects.requireNonNull(suffix, "The suffix must not be null");
        // make defensive copies of arguments
        this.prefix = prefix.toString();
        this.delimiter = delimiter.toString();
        this.suffix = suffix.toString();
        this.emptyValue = this.prefix + this.suffix;
    }
    // 实现原理：添加字符串前先添加前缀，再添加字符串，在下次添加字符串时会在其前面加上分隔符，调用toString方法时，加上后缀
    public StringJoiner add(CharSequence newElement) {
        prepareBuilder().append(newElement);
        return this;
    }

    private StringBuilder prepareBuilder() {
        if (value != null) {
            value.append(delimiter);
        } else {
            value = new StringBuilder().append(prefix);
        }
        return value;
    }
}
```

<br><br>

---