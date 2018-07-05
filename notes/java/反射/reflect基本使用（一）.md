### 反射是框架设计的灵魂

（使用的前提条件：必须先得到代表的字节码的Class，Class类用于表示.class文件（字节码））
#### 1.`Class`对象
每个类都会产生一个对应的`Class`对象，也就是保存在.class文件。所有类都是在对其第一次使用时，动态加载到JVM的，当程序创建一个对类的静态成员的引用时，就会加载这个类。`Class`对象仅在需要的时候才会加载，`static`初始化是在类加载时进行的。

![类加载过程.jpg](https://github.com/WenJunKing/MyNote/blob/master/pics/reflect_class_01)

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
③获取成员变量并调用：
```java
	public static void fieldTest(Class<?> clazz)throws Exception{
		Field nameField=clazz.getDeclaredField("name");
		//获取String 和int类型的这两个参数的构造函数，并初始化值（陈燕,28）。
		Object object=clazz.getConstructor(String.class,int.class).newInstance("陈燕",28);
		System.out.println(object.getClass().getName());
		//暴力反射，解除私有限定
		nameField.setAccessible(true);
		//设置姓名 张三
		nameField.set(object, "张三");
		Person person=(Person) object;
		System.out.println("验证姓名："+person.getName());
	}
```
运行结果：
```java
reflect.Person
验证姓名：张三

```
④获取成员方法并调用：
```java
	public static void methodTest(Class<?> clazz)throws Exception{
		//获取公有的 带String参数的 setName方法
		Method method=clazz.getMethod("setName", String.class);
		//实例化一个私有无参的Person对象
		Constructor constructor=clazz.getDeclaredConstructor();
		constructor.setAccessible(true);
		Object object=constructor.newInstance();
		//调用该方法，设置值
		method.invoke(object, "刘德华");
		Person person=(Person) object;
		System.out.println("验证姓名："+person.getName());
	}
```
运行结果:
```java
验证姓名：刘德华

```
`getDeclaredMethod(String name, Class<?>... parameterTypes);`调用制定方法（所有包括私有的），需要传入两个参数，第一个是调用的方法名称，第二个是方法的形参类型，切记是类型。

### 3.反射方法的其它使用之---通过反射运行配置文件内容
配置文件以txt文件为例子（pro.txt）：
```java
className = cn.fanshe.Student
methodName = show
```
`student`类：
```java

public class Student {
	public void show(){
		System.out.println("is show()");
	}
}
```
测试类：
```java
public class Demo {
	public static void main(String[] args) throws Exception {
		//通过反射获取Class对象
		Class stuClass = Class.forName(getValue("className"));//"cn.fanshe.Student"
		//2获取show()方法
		Method m = stuClass.getMethod(getValue("methodName"));//show
		//3.调用show()方法
		m.invoke(stuClass.getConstructor().newInstance());
		
	}
	
	//此方法接收一个key，在配置文件中获取相应的value
	public static String getValue(String key) throws IOException{
		Properties pro = new Properties();//获取配置文件的对象
		FileReader in = new FileReader("pro.txt");//获取输入流
		pro.load(in);//将流加载到配置文件对象中
		in.close();
		return pro.getProperty(key);//返回根据key获取的value值
	}
}
```
控制台输出：
```java
is show()
```
### 4.反射方法的其它使用之---通过反射越过泛型检查

泛型用在编译期，编译过后泛型擦除（消失掉）。所以是可以通过反射越过泛型检查的。
测试类：
```java
public class Demo {
	public static void main(String[] args) throws Exception{
		ArrayList<String> strList = new ArrayList<>();
		strList.add("aaa");
		strList.add("bbb");
		
	//	strList.add(100);
		//获取ArrayList的Class对象，反向的调用add()方法，添加数据
		Class listClass = strList.getClass(); //得到 strList 对象的字节码 对象
		//获取add()方法
		Method m = listClass.getMethod("add", Object.class);
		//调用add()方法
		m.invoke(strList, 100);
		
		//遍历集合
		for(Object obj : strList){
			System.out.println(obj);
		}
	}
}

```
控制台输出：
```java
aaa
bbb
100
```