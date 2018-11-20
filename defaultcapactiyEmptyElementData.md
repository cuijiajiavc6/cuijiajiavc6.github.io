

ArrayList源代码里面有两个数组，一个是EMPTY_ELEMENTDATA代表空数组，另外一个是DEFAULTCAPACITY_EMPTY_ELEMENTDATA。一看看去，这两个东西怎么都初始化成{}了。有啥区别。

```java
private static final int DEFAULT_CAPACITY = 10;
private static final Object[] EMPTY_ELEMENTDATA = {}; 
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```

其实DEFAULTCAPACITY_EMPTY_ELEMENTDATA是默认数组来的，里面幻想它的长度是10，然后真正加入第一个元素的时候，才把ArrayList里面的数组扩容到10，懒初始化。

关于ArrayList 扩容机制，可以参考：

https://github.com/Snailclimb/JavaGuide/blob/master/Java%E7%9B%B8%E5%85%B3/ArrayList-Grow.md


另外为什么 ArrayList的最大数组大小是Integer.MAX_VALUE - 8
因为数组最大空间是Integer.MAX_VALUE，而保存数组除了元素外，还要保存size,size占8个字节。
