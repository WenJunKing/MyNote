### 反射是框架设计的灵魂

（使用的前提条件：必须先得到代表的字节码的Class，Class类用于表示.class文件（字节码））
#### 1.`Class`对象
每个类都会产生一个对应的`Class`对象，也就是保存在.class文件。所有类都是在对其第一次使用时，动态加载到JVM的，当程序创建一个对类的静态成员的引用时，就会加载这个类。`Class`对象仅在需要的时候才会加载，`static`初始化是在类加载时进行的。

![类加载过程.jpg]()

  类加载器首先会检查这个类的`Class`对象是否已被加载过，如果尚未加载，默认的类加载器就会根据类名查找对应的.class文件。

  想在运行时使用类型信息，必须获取对象(比如类Base对象)的Class对象的引用，使用功能`Class.forName(“Base”)`可以实现该目的，或者使用`base.class`。注意，有一点很有趣，使用功能`”.class”`来创建`Class`对象的引用时，不会自动初始化该`Class`对象，使用`forName()`会自动初始化该`Class`对象。为了使用类而做的准备工作一般有以下3个步骤：

* 加载：由类加载器完成，找到对应的字节码，创建一个`Class`对象
* 链接：验证类中的字节码，为静态域分配空间
* 初始化：如果该类有超类，则对其初始化，执行静态初始化器和静态初始化块.
```java
public class TestMain {
    public static void main(String[] args) {
        System.out.println(XYZ.name);
    }
}

class XYZ {
    public static String name = "luoxn28";

    static {
        System.out.println("xyz静态块");
    }

    public XYZ() {
        System.out.println("xyz构造了");
    }
}
```
```java
public class Base {
    static int num = 1;
    
    static {
        System.out.println("Base " + num);
    }
}
public class Main {
    public static void main(String[] args) {
        // 不会初始化静态块
        Class clazz1 = Base.class;
        System.out.println("------");
        // 会初始化
        Class clazz2 = Class.forName("zzz.Base");
    }
}
```
```java
输出结果：
-------
xyz静态代码块
xyz构造了
```
#### 2.基本使用
① 创建对应的`Class`实例:
 `java.lang.Class` 反射的源头，反射涉及到的类都在  `java.lang.reflect`目录下，如`Field`，`Method`，`ConstructorType`等等。 
实例化`Class`的方法(4种方法)：
* 调用运行时类的`.class`属性:
```java
Class c = Person.class;
System.out.println("方法一 : 调用运行时类的.class属性: "+c.getName());
```
* 通过运行时类的对象，调用`getClass()`方法:
```java
Person person=new Person();
Class c = person.getClass();
System.out.println("方法二 : 通过运行时类的对象，调用getClass()方法: "+c.getName());
```
* 调用`Class`的静态方法`forName(String className)`:
```java
Class c = Class.forName(Person.class.getName().toString());
System.out.println("方法三 : 调用Class的静态方法forName(String className): "+c.getName());
```
* 通过类的加载器:
```java
ClassLoader classLoader=Thread.currentThread().getContextClassLoader();		
Class c=classLoader.loadClass(Person.class.getName());
System.out.println("方法四：通过类的加载器: "+c.getName());
```
②通过反射获取构造方法并使用：
`Person`类：
```java
package reflect;

public class Person {

	private String name;
	public int age;

	public Person(String name,int age){
		this.name=name;
		this.age=age;
	}
	private Person(){
		
	}
	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

}
```
测试类：
```java
	public static void main(String[] agrs)throws Exception {
		
		Class clazz= Class.forName("reflect.Person");// 注意此字符串必须是真实路径，就是带包名的类路径，包名.类名
		
//		ClassLoader classLoader=Thread.currentThread().getContextClassLoader();
//		Class class4=classLoader.loadClass(Person.class.getName());
		
				constructorTest(clazz);
	}
	public static void constructorTest(Class<?> clazz)throws Exception{
		Constructor[] pubCons=clazz.getConstructors();
		for(Constructor constructor:pubCons){
			System.out.println("所有的公用的构造函数："+constructor);
		}
		Constructor con=clazz.getDeclaredConstructor(null);
		System.out.println("私有无参数的构造函数："+con);
		Constructor[] constructors=clazz.getDeclaredConstructors();
		for(Constructor constructor:constructors){
			System.out.println("所有的构造函数："+constructor);
		}
		
	}
		
```
运行结果：
```java
所有的公用的构造函数：public reflect.Person(java.lang.String,int)
私有无参数的构造函数：private reflect.Person()
所有的构造函数：public reflect.Person(java.lang.String,int)
所有的构造函数：private reflect.Person()

```