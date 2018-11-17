##### 考虑用静态工厂方法替代构造器
   创建对象在下面的情况下，可以用静态方法来代替new 对象。
   
   1. **想要给对象名称的时候。**
   比如构造器BigInteger(int, int, Random)，返回的可能是素数。如果通过构造函数直接new 其实很难看出来你创建的是一个素数。如果用BigInteger.probablePrime这个静态方法来返回。就会变得很直观。
   
   2. **不想每次获取实例都创建一个新对象的时候。**
   如果通过构造函数来获取对象，每次都是新的对象。但是如果用静态方法，其实你可以缓存起创建过的对象，当用户要获取对象的时候，从缓存拿出来返回。
   
   3. **想要返回多种类型实例的时候（子类型）**
   可以内嵌子类型，不用公开，然后获取的时候根据参数来决定返回哪种实例。这样更简洁。
   这种设计适合于框架设计，因为设计框架的时候不需要考虑未来所有的子类实现，我只要给出一个静态获取实例的方法，然后继续编写框架逻辑，后期可以动态去添加不同的子类。  比如说JDBC提供一个获取对象的静态方法。用户自由组装需要哪种实现，可以让框架和实现耦合性降低。
   
 **框架**:  
```java
public class A{
   public static A getInstance(){}
}
```
**第三方**:
```java
 public class SubA extends A{
   public static A getInstance(){
        return new SubA();
   }
 }
```


**用户**：通过框架来做另外一个工具。用户在不存在SubA这个类的时候就已经可以做出一些可以应用于SubA的工具啦。
```java
A k = A.getInstance();
//工具类用k 实例做一些处理
k ...  
//假如不使用静态工厂的话，就只能调用构造方法，子类就不能使用这个工具
A k = new A();
//当子类要复用工具的时候，需要在工具里给子类重新写获取实例的代码
A k = new SubA();
```
这样用户写的代码，跟服务提供SubA就没有耦合在一起。



   4. **创建泛型类型实例的时候代码简洁一点。**
   `Map<String, List<String>> m = new HashMap<String,List<String>>()`
   替换成
`   Map<String, List<String>> m = HashMap.newInstance();`
   我常常都是这样写的：
 `  Map<String, List<String>> m = new HashMap(); `
 所以觉得这个例子没打动我。
   
   5. **工厂方法命名规范：**
   valueOf, of, getInstance, newInstance,getType, newType
