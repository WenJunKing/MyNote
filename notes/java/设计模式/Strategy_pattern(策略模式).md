### 策略（`Strategy` `[ˈstrætədʒɪ]`）模式：
>* **策略模式属于对象的行为模式。其用意是针对一组算法，将每一个算法封装到具有共同接口的独立的类中，从而使得它们可以相互替换。**
>* **策略模式使得算法可以在不影响到客户端的情况下发生变化。**

![示意性UML图.jpg](https://github.com/WenJunKing/MyNote/blob/master/pics/strategy_pattern_uml_1.png)
这个模式涉及到3个角色：
* 1. 环境(Context)角色：持有一个Strategy抽象策略类或策略接口的引用。
* 2. 抽象策略(Strategy)角色：这是一个抽象角色，由一个接口或抽象类实现。此角色声明所有的具体策略类需要重写的方法。
* 3. 具体策略(Concrete Strategy)角色：封装了相关的算法或行为。
