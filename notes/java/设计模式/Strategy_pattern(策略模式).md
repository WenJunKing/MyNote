### 策略（`Strategy` `[ˈstrætədʒɪ]`）模式：
>* **策略模式属于对象的行为模式。其用意是针对一组算法，将每一个算法封装到具有共同接口的独立的类中，从而使得它们可以相互替换。**
>* **策略模式使得算法可以在不影响到客户端的情况下发生变化。**

![示意性UML图.jpg](https://github.com/WenJunKing/MyNote/blob/master/pics/strategy_pattern_uml_1.png)

这个模式涉及到3个角色：
* 1. 环境(Context)角色：持有一个Strategy抽象策略类或策略接口的引用。
* 2. 抽象策略(Strategy)角色：这是一个抽象角色，由一个接口或抽象类实现。此角色声明所有的具体策略类需要重写的方法。
* 3. 具体策略(Concrete Strategy)角色：封装了相关的算法或行为。
### 实例：
　* 环境角色：
```java
public class Context {
    // 持有一个具体策略的对象
    private Strategy strategy;
    /**
     * 构造函数，传入一个具体策略对象
     * @param strategy    具体策略对象
     */
    public Context(Strategy strategy){
        this.strategy = strategy;
    }
    /**
     * 策略方法
     */
    public void contextInterface(){
        
        strategy.strategyInterface();
    }
    
}
```
* 抽象策略角色
```java
public interface Strategy {
    /**
     * 策略方法
     */
    public void strategyInterface();
}
```
* 具体策略角色
```java
public class ConcreteStrategyA implements Strategy {

    @Override
    public void strategyInterface() {
        // 相关的业务
    }

}
```
```java
public class ConcreteStrategyB implements Strategy {

    @Override
    public void strategyInterface() {
        // 相关的业务
    }

}
```
### 使用场景实例：

　　假设某个网站销售各种书籍，对初级会员没有提供折扣，对中级会员提供每本10%的促销折扣，对高级会员提供每本20%的促销折扣。

　　折扣是根据以下的3个算法中的1个进行的：

　　* 算法1：对初级会员没有提供折扣。

　　* 算法2：对中级会员提供10%的促销折扣。

　　* 算法3：对高级会员提供20%的促销折扣。

该实例的UML图：

![UML图.jpg](https://github.com/WenJunKing/MyNote/blob/master/pics/strategy_pattern_uml_2.png)

折扣接口
```java
public interface MemberStrategy {
    /**
     * 计算图书的价格
     * @param booksPrice    图书的原价
     * @return    计算出打折后的价格
     */
    public double calcPrice(double booksPrice);
}
```
初级会员折扣实现类
```java
public class PrimaryMemberStrategy implements MemberStrategy {

    @Override
    public double calcPrice(double booksPrice) {
        
        System.out.println("对于初级会员的没有折扣");
        return booksPrice;
    }

}
```
中级会员折扣实现类
```java
public class IntermediateMemberStrategy implements MemberStrategy {

    @Override
    public double calcPrice(double booksPrice) {

        System.out.println("对于中级会员的折扣为10%");
        return booksPrice * 0.9;
    }

}
```
高级会员折扣实现类
```java
public class AdvancedMemberStrategy implements MemberStrategy {

    @Override
    public double calcPrice(double booksPrice) {
        
        System.out.println("对于高级会员的折扣为20%");
        return booksPrice * 0.8;
    }
}
```
价格类
```java
public class Price {
    // 持有一个具体的策略对象
    private MemberStrategy strategy;
    /**
     * 构造函数，传入一个具体的策略对象
     * @param strategy    具体的策略对象
     */
    public Price(MemberStrategy strategy){
        this.strategy = strategy;
    }
    
    /**
     * 计算图书的价格
     * @param booksPrice    图书的原价
     * @return    计算出打折后的价格
     */
    public double quote(double booksPrice){
        return this.strategy.calcPrice(booksPrice);
    }
}
 ```
客户端
```java
public class Client {

    public static void main(String[] args) {
        // 选择并创建需要使用的策略对象
        MemberStrategy strategy = new AdvancedMemberStrategy();
        // 创建环境
        Price price = new Price(strategy);
        // 计算价格
        double quote = price.quote(300);
        System.out.println("图书的最终价格为：" + quote);
    }

}
```

* **策略模式的重心不是如何实现算法，而是如何组织、调用这些算法，从而让程序结构更灵活，具有更好的维护性和扩展性。策略算法是相同行为的不同实现。在运行期间，策略模式在每一个时刻只能使用一个具体的策略实现对象。把所有的具体策略实现类的共同公有方法封装到抽象类里面，将代码向继承等级结构的上方集中。**

### 　策略模式优点：
>1.  通过策略类的等级结构来管理算法族。
>2. 避免使用将采用哪个算法的选择与算法本身的实现混合在一起的多重条件(if-else if-else)语句。

### 策略模式缺点：
>1. 客户端必须知道所有的策略类，并自行决定使用哪一个策略类
>2. 由于策略模式把每个算法的具体实现都单独封装成类，针对不同的情况生成的对象就会变得很多。

参考资料
《JAVA与模式》之策略模式
 