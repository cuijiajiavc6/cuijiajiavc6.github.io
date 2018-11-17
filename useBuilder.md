如果你的类参数超级多，超级长。比如说new A(1,1,1,1,1,1,1,1)如果不是所有都是必填的话，用户面临一个问题，就是我得填一些不关心的值在对应位置，看花眼。怎么解决这个问题。

**第一个想法：**
   
   写多几种构造函数，然后内部去重叠调用来构建（就是我帮你隐藏掉一些参数）。但是这样万一组合很多，那就累死程序员了，而且代码变得很臃肿。

**好一点的想法：**
   我写成javaBean一样的，先创建对象，然后用户一个个set进去。但是这个对象在真正创建完成之前有好多状态（因为这些属性是你一个个set进去的）。那你要考虑线程安全问题。
   
**更好一点的想法：**
   用builder模式。
   
   
```Java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
    
    public static class Builder{
       private final int servingSize;
       private final int servings;
       
       private int calories           = 0;
       private int fat                   = 0;
       private int carbohydrate = 0;
       private int sodium           = 0;
       
       public Builder(int servingSize, int servings){
            this.servingSize = servingSize;
            this.servings      = servings;
       }
       
       public Builder calories(int val){
            calories = val;
            return this;
       }
      public Builder fat(int val) {
           fat = val;
           return this;
      }
      public Builder carbohydrate(int val){
           carbohydrate = val;
           return this;
      }
      public Builder sodium(int val){
          sodium = val;
          return this;
      }
      public NutritionFacts builds(){
           return new NutritionFacts(this);
    }
  }
  
  private NutritionFacts(Builder builder){
     servingSize     = builder.servingSize;
     servings          = builder.servings;
     calories           = builder.calories;
     fat                   = builder.fat;
     sodium           = builder.sodium;
     carbohydrate = builder.carbohydrate;
  }
}
```
创建对象的时候：
new NuritionFacts.Builder(240,8).calories(100),sodium(35).build();

小技巧，把必填写成builder构造函数参数，这样就一定要填啦。选填的就写成可变的，让用户自己set.

优点:
1. 选填参数可以有一个名字，增加可读性。
2. 选填参数可长可短，可自由组合，不用增加代码。
3. 如果你想的话，你也可以给用户传入自己的builder来创建对象。通过泛型通配来限制类型。更灵活。
例如 Tree buildTree(Builder<? extends Node> nodeBuilder){...}
4. 比JavaBeans安全，因为对象状态不会变化。


缺点:
创建对象还得先多创建一个builder，如果对十分注重性能的情况下就是个问题。
