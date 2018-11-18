下面介绍3种比较好的单例模式实现：

**公有静态成员**
```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  
  private Elvis() {...}
  
  public void leaveTheBuilding(){
  }
}
```

这种AccessibleObject.setAccessible方法，可以通过反射调用到构造方法，可以通过在构造方法内判断INSTANCE是否存在，存在抛出异常来防止被破坏单例模式。

**静态工厂**

````java
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis() {...}
  public static Elvis getInstance(){ return INSTANCE; }
  
  public void leaveTheBuilding() { ... }
}
````
这种把内部实例对象private 就灵活一点，哪天不想单例了，可以灵活变更，不会影响到用户。但是这种要考虑序列化，反序列化的时候可能会对类造成破坏。可以通过提供readResolve方法来防止反序列化的破坏。
````java
private Object readResolve(){
   return INSTANCE;
}
````

**枚举**
````java
public enum Elvis{
    INSTANCE;
    
    public void leaveTheBuilding(){
    }
}
````

1. 枚举不怕反射攻击：
因为Constructor类的newInstance方法里面判断
`clazz.getModifiers() & Modifier.ENUM) != 0`
如果类是个枚举，就会抛异常，不能调用构造函数

2. 枚举不怕反序列化：
**Java对象序列化规范**
(Java Object Serialization Specification)中规定:
在序列化的时候Java仅仅是将枚举对象的name属性输出到结果中，反序列化的时候则是通过java.lang.Enum的valueOf方法来根据名字查找枚举对象。同时，编译器是不允许任何对这种序列化机制的定制的，因此禁用了writeObject、readObject等方法

之所以推荐枚举是，枚举都帮你做好了那些反射啊、反序列化啊的保护了，你就不用花那么多力气去自己再实现一遍了。所以用起来比较简洁一点。
