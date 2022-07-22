`instanceof`这个关键词，主要用来判断某个对象是不是某个类的实例。

比如，有时候我们要处理一个类似这样的数据集：

```java
Map<String, Object> data = new HashMap<>();
data.put("key1", "aaa");
data.put("key2", 111);
```

这个Map中的Value值因为可能是不同的对象，所以定义的是Object。这个时候，当我们get出来的时候，就需要去判断和转换之后再去处理。

比如，我们取出`key1`的`value`，然后截取一段字符串的操作，就需要这样写：

```java
Object value  =data.get("key1");
if (value instanceof String) {
  String s = (String) value;
  System.out.println(s.substring(1));
}
```

先判断获取的value是否是String，再做强制类型转换，然后再对字符串进行操作。这是传统的写法，而在Java 16的增强之后，对于`instanceof`的判断以及类型转换可以合二为一了，所以改进后的写法可以如下：

```java
Object value  =data.get("key1");
if (value instanceof String s) {
  System.out.println(s.substring(1));
}
```

是不是简单不少呢？如果没用过的话，赶紧操作试试看吧