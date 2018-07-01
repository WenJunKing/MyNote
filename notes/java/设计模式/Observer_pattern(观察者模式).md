### 观察者模式(ObserverPattern):
#### 1.1概述：
  在许多设计中，经常涉及多个对象都对一个特殊对象中的数据变化感兴趣，而且这多个对象都希望跟踪那个特殊对象中的数据变化，在这样的情况下就可以使用观察者模式。
   观察者模式是关于多个对象想知道一个对象中数据变化情况的一种成熟的模式。观察者模式中有一个称作“主题”的对象和若干个称作“观察者”的对象，“主题”和“观察者”间是一种一对多的依赖关系，当“主题”的状态发生变化时，所有“观察者”都得到通知。前面所述的“求职中心”相当于观察者模式的一个具体“主题”；每个“求职者”相当于观察者模式中的一个具体“观察者”。

#### 1.2模式的结构：
* 主题（`Subject`）：主题是一个接口，该接口规定了具体主题需要实现的方法，比如，添加、删除观察者以及通知观察者更新数据的方法。
* 观察者（`Observer`）：观察者是一个接口，该接口规定了具体观察者用来更新数据的方法。
* 具体主题（`ConcreteSubject`）：具体主题是实现主题接口类的一个实例，该实例包含有可以经常发生变化的数据。具体主题需使用一个集合，比如`ArrayList`，存放观察者的引用，以便数据变化时通知具体观察者。
* 具体观察者（`ConcreteObserver`）：具体观察者是实现观察者接口类的一个实例。具体观察者包含有可以存放具体主题引用的主题接口变量，以便具体观察者让具体主题将自己的引用添加到具体主题的集合中，使自己成为它的观察者，或让这个具体主题将自己从具体主题的集合中删除，使自己不再是它的观察者。

观察者的类图如下所示：

![ObserverPatternUML.jpg](https://github.com/WenJunKing/MyNote/blob/master/pics/observer_pattern_uml01.jpg)

#### 1.3适合使用观察者模式的情景:
* 当一个对象的数据更新时需要通知其他对象，但这个对象又不希望和被通知的那些对象形成紧耦合。
* 当一个对象的数据更新时，这个对象需要让其他对象也各自更新自己的数据，但这个对象不知道具体有多少对象需要更新数据。

#### 1.4观察者模式的应用：
模拟一个新闻服务号和一些订阅者：

首先开始写主题接口和观察者接口
```java
/**
 * Created by StephenDu on 2018/6/30.
 * Email:179451678@qq.com
 * Description:主题接口
 */
public interface Subject {
    //添加一个观察者
    void registerObserver(Observer observer);

    //移除一个观察者
    void removeObserver(Observer observer);

    //通知所有观察者
    void notifyObservers();
}

```
```java
/**
 * Created by StephenDu on 2018/6/30.
 * Email:179451678@qq.com
 * Description:
 */
public interface Observer<T> {
    void updateMsg(T t);
}

```
新闻服务号的实现类：

```java
**
 * Created by StephenDu on 2018/6/30.
 * Email:179451678@qq.com
 * Description:
 */
public class NewsSubject implements Subject {
    private String news;
    List<Observer<String>> observers=new ArrayList<>();
    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void removeObserver(Observer observer) {
        int index=observers.indexOf(observer);
        if(index>=0){
            observers.remove(index);
        }
    }

    @Override
    public void notifyObservers() {
        for(Observer observer:observers){
            observer.updateMsg(news);
        }
    }
    public void setNews(String news){
        this.news=news;
        notifyObservers();
    }
}
```
模拟两个观察者：
```java
/**
 * Created by StephenDu on 2018/6/30.
 * Email:179451678@qq.com
 * Description:
 */
public class NewsObserver1 implements Observer<String> {
    final String TAG="NewsObserver1";
    private Subject subject;
    public NewsObserver1(Subject subject){
        this.subject=subject;
        this.subject.registerObserver(this);
    }
    public void unRegister(){
        subject.removeObserver(this);
    }
    @Override
    public void updateMsg(String s) {
        Log.e(TAG,"newsObserver1收到的News:"+s);
    }

}
```
```java
/**
 * Created by StephenDu on 2018/6/30.
 * Email:179451678@qq.com
 * Description:
 */
public class NewsObserver2 implements Observer<String> {
    final  String TAG="NewsObserver2";
    private Subject subject;
    public NewsObserver2(Subject subject){
        this.subject=subject;
        this.subject.registerObserver(this);
    }
    public void unRegister(){
        subject.removeObserver(this);
    }
    @Override
    public void updateMsg(String s) {
        Log.e(TAG, "newsObserver2收到的News:"+s);
    }
}
```
**可以看出：服务号中维护了所有向它订阅消息的使用者，当服务号有新消息时，通知所有的使用者。整个架构是一种松耦合，主题的实现不依赖与使用者，当增加新的使用者时，主题的代码不需要改变；使用者如何处理得到的数据与主题无关；**

测试代码：
```java
        NewsSubject newsSubject=new NewsSubject();
        NewsObserver1 newsObserver1=new NewsObserver1(newsSubject);
        NewsObserver2 newsObserver2=new NewsObserver2(newsSubject);
        newsSubject.setNews("大冷门，德国0:2憾负韩国，无缘淘汰赛！");
        newsObserver1.unRegister();
        newsSubject.setNews("今晚10点淘汰赛第一场法国VS阿根廷，看梅西如何斗群星！");
```
运行结果
```java
NewsObserver1:--> newsObserver1收到的News:大冷门，德国0:2憾负韩国，无缘淘汰赛！
NewsObserver2:--> newsObserver2收到的News:大冷门，德国0:2憾负韩国，无缘淘汰赛！
NewsObserver2:--> newsObserver2收到的News:今晚10点淘汰赛第一场法国VS阿根廷，看梅西如何斗群星！
```
#### 2.1 使用Java内置的类实现观察者模式
* 借助于`java.util.Observable`和`java.util.Observer`。

#### `Observable`类
**`Observable`是一个抽象的主题对象。**
一个被观测的对象必须服从下面的两个简单规则：
* 第一，如果它被改变了，它必须调用`setChanged()`方法。
* 第二，当它准备通知观测程序它的改变时，它必须调用`notifyObservers()`方法，这导致了在观测对象中对`update()`方法的调用。
>注意：如果在调用`notifyObservers()`方法之前没有调用
>  `setChanged()`方法，就不会有什么动作发生。
> `notifyObservers()`方法中包含`clearChanged()`方法，将标志变量置回原值。
> `notifyObservers()`方法采用的是从后向前的遍历方式，即最后加入的观察者最先被调用`update()`方法。

#### `Observer`接口
此接口中只有一个方法：
`void update(Observable o, Object arg) `
这个方法在被观察对象（`Observable`类）的`notifyObservers()`方法中被调用。

首先是一个新闻订阅号主题：
```java
public class NewsInnerSubject extends Observable{
    private String msg;
    //主题更新消息
    public void setMsg(String mgs){
        this.msg=mgs;
        setChanged();
        notifyObservers(msg);
    }
}
```
定义两个观察者:
```java
public class NewsInnerObserver1 implements Observer{
    private final String TAG="NewsInnerObserver1";
    public NewsInnerObserver1(Observable observable){
        observable.addObserver(this);
    }
    @Override
    public void update(Observable o, Object arg) {
        if(o instanceof NewsInnerSubject){
            String msg= (String) arg;
            Log.e(TAG, "NewsInnerObserver1:"+msg);
        }
    }
}
```
```java
public class NewsInnerObserver2 implements Observer{
    private final String TAG="NewsInnerObserver2";
    public NewsInnerObserver2(Observable observable){
        observable.addObserver(this);
    }
    @Override
    public void update(Observable o, Object arg) {
        if(o instanceof NewsInnerSubject){
            String msg= (String) arg;
            Log.e(TAG,"NewsInnerObserver2:"+msg);
        }
    }
}
```
测试调用：
```java
        NewsInnerSubject newsInnerSubject=new NewsInnerSubject();
        NewsInnerObserver1 newsInnerObserver1=new NewsInnerObserver1(newsInnerSubject);
        NewsInnerObserver2 newsInnerObserver2=new NewsInnerObserver2(newsInnerSubject);
        newsInnerSubject.setMsg("使用java内置观察者模式！！！");
```
运行结果：
```java
NewsInnerObserver2---> NewsInnerObserver2:使用java内置观察者模式！！！
NewsInnerObserver1---> NewsInnerObserver1:使用java内置观察者模式！！！
```
可以看出，使用Java内置的类实现观察者模式，代码非常简洁，对了`addObserver`,`removeObserver`,`notifyObservers`都已经为我们实现了，所有可以看出`Observable`（主题）是一个类，而不是一个接口，基本上书上都对于`Java`的如此设计抱有反面的态度，觉得`Java`内置的观察者模式，违法了面向接口编程这个原则，但是如果转念想一想，的确你拿一个主题在这写观察者模式（我们自己的实现），接口的思想很好，但是如果现在继续添加很多个主题，每个主题的`ddObserver`,`removeObserver`,`notifyObservers`代码基本都是相同的吧，接口是无法实现代码复用的，而且也没有办法使用组合的模式实现这三个方法的复用，所以我觉得这里把这三个方法在类中实现是合理的。

