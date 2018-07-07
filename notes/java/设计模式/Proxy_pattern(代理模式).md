### 代理模式（ProxyPattern）
#### [转自：Gonjian](https://www.cnblogs.com/gonjan-blog/p/6685611.html)
#### 1.简述：
 代理模式是常用的java设计模式，他的特征是代理类与委托类有同样的接口，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。简单的说就是，我们在访问实际对象时，是通过代理对象来访问的，代理模式就是在访问实际对象时引入一定程度的间接性，因为这种间接性，可以附加多种用途。

![代理模式结构图](https://github.com/WenJunKing/MyNote/blob/master/pics/proxy_pattern_uml_01.jpg)

#### 2.静态代理
静态代理：由程序员创建或特定工具自动生成源代码，也就是在编译时就已经将接口，被代理类，代理类等确定下来。在程序运行之前，代理类的.class文件就已经生成。

##### 2.1静态代理简单实现：
创建一个Star(明星接口),这个接口是被代理类(Stephen) 和代理类(经纪人)的共同接口，这样他们 都有共同的行为，经纪人就可以代理Stephen的行为。
```java
//明星类
public interface Star {
    //接通告
    void jieTongGao();
    //拍电影
    void makeFilm();
}
```
StephenChow类实现`Star` 接口。StephenChow可以具体实施接通告和拍电影的工作。
```java
public class StephenChow implements Star {
    final String TAG=StephenChow.class.getSimpleName();
    private String name;
    private int age;
    @Override
    public void jieTongGao() {
        Log.e(TAG,"StephenChow接通告了！！！");
    }

    @Override
    public void makeFilm() {
        Log.e(TAG,"StephenChow拍电影了！！！");
    }

}
```
`StephenChowsProxy`类，这个类也实现了`Star`接口，但是还另外持有`StephenChow`对象，由于实现了`Star`接口，同时持有`StephenChow`对象，那么他可以代理StephenChow执行接通告和拍电影的行为。
```java
public class StephenChowsProxy implements Star{
    StephenChow star;
    public StephenChowsProxy (Star star){
          this.star= (StephenChow) star;
    }
    @Override
    public void jieTongGao() {
        star.jieTongGao();
    }

    @Override
    public void makeFilm() {
        star.makeFilm();
    }
}

```
测试：
```java
    /**
     * 运行静态代理
     */
    private void runStaticProxy(){
        StephenChowsProxy staticProxy=new StephenChowsProxy(new StephenChow());
        staticProxy.jieTongGao();
        staticProxy.makeFilm();
    }

```
结果：
```java
    StephenChow接通告了！！！
    StephenChow拍电影了！！！
```
这里并没有直接通过StephenChow（被代理对象）来执行相关行为，而是通过经纪人（代理对象）来代理执行了。这就是代理模式。

代理模式最主要的就是有一个**公共接口**，一个**具体的类**、一个**代理类**、**代理类持有具体类的实例**，代为执行具体类实例方法。上面说到，代理模式就是在访问实际对象时引入一定程度的间接性，因为这种间接性，可以附加多种用途。这里的间接性就是指不直接调用实际对象的方法，那么我们在代理过程中就可以加上一些其他用途。就这个例子来说，假如经纪人在帮接通告前要先收1000W的通告费，通过代理模式很轻松就能办到：
```java
public class StephenChowsProxy implements Star{
    StephenChow star;
    public StephenChowsProxy (Star star){
          this.star= (StephenChow) star;
    }
    @Override
    public void jieTongGao() {
        System.out.println("先给我1000W的通告费");
        star.jieTongGao();
    }

    @Override
    public void makeFilm() {
        star.makeFilm();
    }
}
```
结果:
```java
    先给我1000W的通告费.
    StephenChow接通告了！！！
    StephenChow拍电影了！！！
```
可以看到，只需要在代理类中帮StephenChow接通告之前，执行其他操作就可以了。这种操作，也是使用代理模式的一个很大的优点。最直白的就是在`Spring`中的面向切面编程（`AOP`），我们能在一个切点之前执行一些操作，在一个切点之后执行一些操作，这个切点就是一个个方法。这些方法所在类肯定就是被代理了，在代理过程中切入了一些其他操作。

静态代理的缺点：

* 1.代理的方法如果很多，那么就要为每个方法都要代理，规模大的程序受不了。
* 2.如果真实类中新添加一个方法或功能，那么代理类中就一一对应的写出来，这样不利于扩展并且增加代码维护成本。
* 3.一个代理类只能代理一个真实的对象。


### 2.动态代理
动态代理有别于静态代理，是根据代理的对象，动态创建代理类。这样，就可以避免静态代理中代理类接口过多的问题。动态代理是实现方式，是通过反射来实现的，借助Java自带的`java.lang.reflect.Proxy`,通过固定的规则生成。

步骤如下：

* 1.接口：编写一个委托类的接口.

* 2.被代理类：实现一个真正的委托类。

* 3.创建一个动态代理类：实现`InvocationHandler`接口，并重写该`invoke`方法.
* 4.在测试类中，生成动态代理的对象。

实例：

第一二步骤，和静态代理一样。第三步，代码如下：
```java
public class DynamicProxy implements InvocationHandler {
    private Object object;
    public DynamicProxy(Object object) {
        this.object = object;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result = method.invoke(object, args);
        return result;
    }
}
```
第四步，创建动态代理的对象
```java
  Star star=new StephenChow();
  DynamicProxy proxy = new DynamicProxy(star);
  ClassLoader classLoader = star.getClass().getClassLoader();
  Star proxyStar= (Star ) Proxy.newProxyInstance(classLoader, new Class[]{Star.class}, proxy);
  proxyStar.jieTongGao();
```
也可以将三四步合二为一：
```java
public class StephenChowProxy {
    final String TAG=StephenChowProxy.class.getSimpleName();
    private Star star=new StephenChow();
    public Star getProxy(){

       Star msStar= (Star) Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(),
                                             star.getClass().getInterfaces(),
                                             new InvocationHandler() {
                                   @Override
                                   public Object invoke(Object proxy, Method method, Object[] args)
                                           throws Throwable {
                                       if("jieTongGao".equals(method.getName())){
                                           Log.e(TAG,"我是他经纪人，找他接通告先给我100W！！");
                                           return method.invoke(star,args);
                                       }
                                       if("makeFilm".equals(method.getName())){
                                           Log.e(TAG,"我是他经纪人，找他拍电影先给我1亿！！");
                                           return method.invoke(star,args);
                                       }
                                       return null;
                                   }
                               });
        return msStar;
    }
}

```
测试：
```java
    /**
     * 运行动态代理
     */
    private void runDynamicProxy(){
        StephenChowProxy stephenChowProxy=new StephenChowProxy();
        Star star=stephenChowProxy.getProxy();
        star.makeFilm();
        star.jieTongGao();
    }
```
结果：
```java
我是他经纪人，找他拍电影先给我1亿！！
StephenChow拍电影了！！！
我是他经纪人，找他接通告先给我100W！！
StephenChow接通告了！！！
```
创建动态代理的对象，需要借助`Proxy.newProxyInstance`。该方法的三个参数分别是：
* `ClassLoader loader`表示当前使用到的appClassloader。
* `Class<?>[] interfaces`表示目标对象实现的一组接口。
* `InvocationHandler h`表示当前的`InvocationHandler`实现实例对象。

### 3.动态代理的原理分析:
#### 3.1、Java动态代理创建出来的动态代理类
上面我们利用`Proxy`类的`newProxyInstance`方法创建了一个动态代理对象，查看该方法的源码，发现它只是封装了创建动态代理类的步骤：
```java
public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        Objects.requireNonNull(h);

        final Class<?>[] intfs = interfaces.clone();
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        /*
         * Look up or generate the designated proxy class.
         */
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * Invoke its constructor with the designated invocation handler.
         */
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }

            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }
```
我们最应该关注的是` Class<?> cl = getProxyClass0(loader, intfs);`这句，这里产生了代理类，后面代码中的构造器也是通过这里产生的类来获得，可以看出，这个类的产生就是整个动态代理的关键，由于是动态生成的类文件，我这里不具体进入分析如何产生的这个类文件，只需要知道这个类文件时缓存在java虚拟机中的，我们可以通过下面的方法将其打印到文件里面，一睹真容：
```java
byte[] classFile = ProxyGenerator.generateProxyClass("$Proxy0", Student.class.getInterfaces());
        String path = "G:/javacode/javase/Test/bin/proxy/StuProxy.class";
        try(FileOutputStream fos = new FileOutputStream(path)) {
            fos.write(classFile);
            fos.flush();
            System.out.println("代理类class文件写入成功");
        } catch (Exception e) {
           System.out.println("写文件错误");
        }
```
上述用到的委托类`Student`和`Person`接口：
```java
public interface Person {
    //上交班费
    void giveMoney();
}
```
```java
public class Student implements Person {
    private String name;
    public Student(String name) {
        this.name = name;
    }
    
    @Override
    public void giveMoney() {
        try {
          //假设数钱花了一秒时间
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
       System.out.println(name + "上交班费50元");
    }
}
```
对这个class文件进行反编译，我们看看jdk为我们生成了什么样的内容：
```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;
import proxy.Person;

public final class $Proxy0 extends Proxy implements Person
{
  private static Method m1;
  private static Method m2;
  private static Method m3;
  private static Method m0;
  
  /**
  *注意这里是生成代理类的构造方法，方法参数为InvocationHandler类型，看到这，是不是就有点明白
  *为何代理对象调用方法都是执行InvocationHandler中的invoke方法，而InvocationHandler又持有一个
  *被代理对象的实例，不禁会想难道是....？ 没错，就是你想的那样。
  *
  *super(paramInvocationHandler)，是调用父类Proxy的构造方法。
  *父类持有：protected InvocationHandler h;
  *Proxy构造方法：
  *    protected Proxy(InvocationHandler h) {
  *         Objects.requireNonNull(h);
  *         this.h = h;
  *     }
  *
  */
  public $Proxy0(InvocationHandler paramInvocationHandler)
    throws 
  {
    super(paramInvocationHandler);
  }
  
  //这个静态块本来是在最后的，我把它拿到前面来，方便描述
   static
  {
    try
    {
      //看看这儿静态块儿里面有什么，是不是找到了giveMoney方法。请记住giveMoney通过反射得到的名字m3，其他的先不管
      m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] { Class.forName("java.lang.Object") });
      m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
      m3 = Class.forName("proxy.Person").getMethod("giveMoney", new Class[0]);
      m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
      return;
    }
    catch (NoSuchMethodException localNoSuchMethodException)
    {
      throw new NoSuchMethodError(localNoSuchMethodException.getMessage());
    }
    catch (ClassNotFoundException localClassNotFoundException)
    {
      throw new NoClassDefFoundError(localClassNotFoundException.getMessage());
    }
  }
 
  /**
  * 
  *这里调用代理对象的giveMoney方法，直接就调用了InvocationHandler中的invoke方法，并把m3传了进去。
  *this.h.invoke(this, m3, null);这里简单，明了。
  *来，再想想，代理对象持有一个InvocationHandler对象，InvocationHandler对象持有一个被代理的对象，
  *再联系到InvacationHandler中的invoke方法。嗯，就是这样。
  */
  public final void giveMoney()
    throws 
  {
    try
    {
      this.h.invoke(this, m3, null);
      return;
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }

  //注意，这里为了节省篇幅，省去了toString，hashCode、equals方法的内容。原理和giveMoney方法一毛一样。

}
```
jdk为我们的生成了一个叫`$Proxy0`（这个名字后面的0是编号，有多个代理类会一次递增）的代理类，这个类文件时放在内存中的，我们在创建代理对象时，就是通过反射获得这个类的构造方法，然后创建的代理实例。通过对这个生成的代理类源码的查看，我们很容易能看出，动态代理实现的具体过程。

我们可以对`InvocationHandler`看做一个中介类，中介类持有一个被代理对象，在`invoke`方法中调用了被代理对象的相应方法。通过聚合方式持有被代理对象的引用，把外部对`invoke`的调用最终都转为对被代理对象的调用。

代理类调用自己方法时，通过自身持有的中介类对象来调用中介类对象的`invoke`方法，从而达到代理执行被代理对象的方法。也就是说，动态代理通过中介类实现了具体的代理功能。

### 总结：
生成的代理类：`$Proxy0 extends Proxy implements Person`，我们看到代理类继承了`Proxy`类，所以也就决定了java动态代理只能对接口进行代理，Java的继承机制注定了这些动态代理类们无法实现对class的动态代理。
上面的动态代理的例子，其实就是`AOP`的一个简单实现了，在目标对象的方法执行之前和执行之后进行了处理，对方法耗时统计。`Spring`的`AOP`实现其实也是用了`Proxy`和`InvocationHandler`这两个东西的。