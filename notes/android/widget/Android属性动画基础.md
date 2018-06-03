# 属性动画
---
### 工作原理
 * 在一定的时间间隔里，通过不断的对值进行改变，并不断地将该值赋值给对象的属性，从而实现该对象在属性上的动画效果。

![属性动画原理.jpg](
https://github.com/WenJunKing/MyNote/blob/master/pics/56d928466744f24d30c8a59bdaa08782_944365-16a162a731f548d8.png)
* 从上述工作原理可以看出属性动画有两个非常重要的类：`ValueAnimator `类  &`ObjectAnimator` 类
* 其实属性动画的使用基本都是依靠这两个类。
### 具体使用
### 1.ValueAnimator类
* 定义：属性动画机制中最核心的一个类
* 实现动画的原理：**通过不断地控制值得变化,再不断地手动的赋值给对象的属性，从而实现动画的效果。**

从上面原理可以看出：`ValueAnimator`类中有3个重要方法：
1. `ValueAnimator.ofInt（int values）`
2. `ValueAnimator.ofFloat（float values）`
3. `ValueAnimator.ofObject（int values）`
### 1.1 ValueAnimator.ofInt(int... Values)
* 作用：将初始值 **以整型数值的形式** 过渡到结束值
>即估值器是整型估值器 - `IntEvaluator`
>
* 实例1：java代码设置
```java
    Button ofIntBtn=findViewById(R.id.btn_ofint);
    private void ofInt(){
        int oldWidth=ofIntBtn.getLayoutParams().width;
        Log.e(TAG,"oldWidth:"+oldWidth);
        ValueAnimator valueAnimator=ValueAnimator.ofInt(oldWidth,500);
        valueAnimator.setDuration(2000);
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                ofIntBtn.getLayoutParams().width= (int) animation.getAnimatedValue();
                ofIntBtn.requestLayout();
            }
        });
        valueAnimator.start();
    }
```
* 实例2：XML设置
  * 设置动画参数
 `set_animation.xml`
```xml
    // ValueAnimator采用<animator>  标签
<animator xmlns:android="http://schemas.android.com/apk/res/android"
    android:valueFrom="0"   // 初始值
    android:valueTo="100"  // 结束值
    android:valueType="intType" // 变化值类型 ：floatType & intType

    android:duration="3000" // 动画持续时间（ms），必须设置，动画才有效果
    android:startOffset ="1000" // 动画延迟开始时间（ms）
    android:fillBefore = “true” // 动画播放完后，视图是否会停留在动画开始的状态，默认为true
    android:fillAfter = “false” // 动画播放完后，视图是否会停留在动画结束的状态，优先于fillBefore值，默认为false
    android:fillEnabled= “true” // 是否应用fillBefore值，对fillAfter值无影响，默认为true
    android:repeatMode= “restart” // 选择重复播放动画模式，restart代表正序重放，reverse代表倒序回放，默认为restart|
    android:repeatCount = “0” // 重放次数（所以动画的播放次数=重放次数+1），为infinite时无限重复
    android:interpolator = @[package:]anim/interpolator_resource // 插值器，即影响动画的播放速度,下面会详细讲
/>
```
  * 在java代码中启动动画
```java
Animator animator = AnimatorInflater.loadAnimator(context, R.animator.set_animation);  
// 载入XML动画
animator.setTarget(view);  
// 设置动画对象
animator.start();  
// 启动动画
```
**`ValueAnimator.ofInt()`与`ValueAnimator.oFloat()`仅仅只是在估值器上的区别：（即如何从初始值 过渡 到结束值)**

* `ValueAnimator.oFloat（）`采用默认的浮点型估值器 (`FloatEvaluator`)
* `ValueAnimator.ofInt（）`采用默认的整型估值器（`IntEvaluator`）

在使用上完全没有区别，此处对`ValueAnimator.oFloat（）`的使用就不作过多描述。

### 1.2 ValueAnimator.ofObject(TypeEvaluator evaluator, Object... values)
* 作用：将初始值**以对象的形式过渡到结束值**
>即通过操作 对象 实现动画效果
>
* 具体使用：
```java
// 创建初始动画时的对象  & 结束动画时的对象
myObject object1 = new myObject();  
myObject object2 = new myObject();  

ValueAnimator anim = ValueAnimator.ofObject(new myObjectEvaluator(), object1, object2);  
// 创建动画对象 & 设置参数
// 参数说明
// 参数1：自定义的估值器对象（TypeEvaluator 类型参数） - 下面会详细介绍
// 参数2：初始动画的对象
// 参数3：结束动画的对象
anim.setDuration(5000);  
anim.start();
```
在继续讲解`ValueAnimator.ofObject（）`的使用前，我先讲一下估值器（`TypeEvaluator`）

### 估值器（TypeEvaluator） 介绍
* 作用：设置动画 **如何从初始值 过渡到 结束值 的逻辑**
>1.  插值器（`Interpolator`）决定 值 的变化模式（匀速、加速blabla）
>2. 估值器（`TypeEvaluator`）决定 值 的具体变化数值
>
从`ValueAnimator.ofInt`可以看到：
* `ValueAnimator.ofInt（）`实现了 **将初始值 以整型的形式 过渡到结束值 ** 的逻辑，那么这个过渡逻辑具体是怎么样的呢？
* 其实是系统内置了一个 `IntEvaluator`估值器，内部实现了初始值与结束值 以整型的过渡逻辑
* 我们来看一下 `IntEvaluator`的代码实现：
```java
public class IntEvaluator implements TypeEvaluator<Integer> {
    // 重写evaluate()
    public Integer evaluate(float fraction, Integer startValue, Integer endValue) {
        int startInt = startValue;
        // 参数说明
        // fraction：表示动画完成度（根据它来计算当前动画的值）
        // startValue、endValue：动画的初始值和结束值
        return (int)(startInt + fraction * (endValue - startInt));
        // 初始值 过渡 到结束值 的算法是：
        // 1. 用结束值减去初始值，算出它们之间的差值
        // 2. 用上述差值乘以fraction系数
        // 3. 再加上初始值，就得到当前动画的值
    }
}
```
* `ValueAnimator.ofInt（）` & `ValueAnimator.ofFloat（）`都具备系统内置的估值器，即`FloatEvaluator` & `IntEvaluator`
>即系统已经默认实现了 **如何从初始值 过渡到 结束值 的逻辑**
>
* 但对于`ValueAnimator.ofObject（）`，从上面的工作原理可以看出并没有系统默认实现，因为对对象的动画操作复杂 & 多样，系统无法知道如何从初始对象过度到结束对象
* 因此，对于`ValueAnimator.ofObject（）`，我们需自定义估值器（`TypeEvaluator`）来告知系统如何进行从 初始对象 过渡到 结束对象的逻辑。
### 实例：
* 实现的动画效果：一个圆从一个点 移动到 另外一个点

![效果图.gif](https://github.com/WenJunKing/MyNote/blob/master/pics/efd2325c2319bc9b9c37bc314349dbda_944365-45b817bd4ca8c119.gif)

### 步骤1：定义对象类
* 因为`ValueAnimator.ofObject（）`是面向对象操作的，所以需要自定义对象类。
* 本例需要操作的对象是**圆的点坐标**
Point.java
```java
public class Point {
    // 设置两个变量用于记录坐标的位置
    private float x;
    private float y;
    public Point(float x,float y){
        this.x=x;
        this.y=y;
    }
    public float getX() {
        return x;
    }
    public void setX(float x) {
        this.x = x;
    }
    public float getY() {
        return y;
    }
    public void setY(float y) {
        this.y = y;
    }
}
```
### 步骤2：根据需求实现TypeEvaluator接口
* 实现`TypeEvaluator`接口的目的是自定义如何 从初始点坐标 过渡 到结束点坐标；
* 本例实现的是一个从左上角到右下角的坐标过渡逻辑。
PointEvaluator.java
```java

/**
 * Author:wenjundu on 2018/5/16
 * Email: 179451678@qq.com
 * Description:
 */

public class PointEvaluator implements TypeEvaluator<Point> {

    @Override
    public Point evaluate(float fraction, Point startValue, Point endValue) {
        float x=startValue.getX()+fraction*(endValue.getX()-startValue.getX());
        float y=startValue.getY()+fraction*(endValue.getY()-startValue.getY());
        return new Point(x,y);
    }
}
```
### 步骤三：自定义View
PointView.java
```java
/**
 * Author:wenjundu on 2018/5/16
 * Email: 179451678@qq.com
 * Description:
 */

public class PointView extends View {
    private final float RADIUS=70f;
    private Point currentPoint;
    private Paint mPaint;
    public PointView(Context context) {
        this(context,null);
    }
    public PointView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        init();
    }
    private void init() {
        // 初始化画笔
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.BLUE);
        currentPoint=new Point(RADIUS,RADIUS);
    }
    //设置当前的Point
    public void setCurrentPoint(Point point){
        currentPoint=point;
        requestLayout();
    }
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.drawCircle(currentPoint.getX(),currentPoint.getY(),RADIUS,mPaint);
    }
}
```
### 步骤4：将属性动画作用到自定义View当中
```java
 /**
     * VauleAnimator ofObject 实例
     */
    private void ofObject(){
        Point startPoint=new Point(70,70);
        Point endPoint=new Point(700,1000);
        ValueAnimator valueAnimator=ValueAnimator.ofObject(new PointEvaluator(),startPoint,endPoint);
        valueAnimator.setDuration(2000);
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                //调用自定义View的设置当前点的方法
                pointView.setCurrentPoint((Point) animation.getAnimatedValue());
            }
        });
        valueAnimator.start();
    }
```

### 特别注意
* 从上面可以看出，其实`ValueAnimator.ofObject（）`的本质还是操作 ** 值 **，只是是采用将 多个值 封装到一个对象里的方式 同时对多个值一起操作而已.
>就像上面的例子，本质还是操作坐标中的x，y两个值，只是将其封装到Point对象里，方便同时操作x，y两个值而已.
>
* 至此，关于属性动画中最核心的 `ValueAnimator`类已经讲解完毕.

### 2.ObjectAnimator类
* 实现动画的原理：直接对对象的属性值进行改变操作，从而实现动画效果。
>1. 如直接改变 View的 alpha 属性 从而实现透明度的动画效果
>2. 继承自`ValueAnimator`类，即底层的动画实现机制是基于`ValueAnimator`类
* 本质原理： 通过不断控制 值 的变化，再不断 **自动** 赋给对象的属性，从而实现动画效果。如下图：
![ObjectAnimator原理图.jpg](https://github.com/WenJunKing/MyNote/blob/master/pics/4efdd88a2ea9cf42c5e46ddae5100d0c_944365-36c8b1b8066b0623.png)

从上面的工作原理可以看出：`ObjectAnimator`与 `ValueAnimator`类的区别：
* `ValueAnimator `类是先改变值，然后 **手动赋值** 给对象的属性从而实现动画；是 **间接** 对对象属性进行操作；
* `ObjectAnimator` 类是先改变值，然后 **自动赋值** 给对象的属性从而实现动画；是 **直接** 对对象属性进行操作；
### 2.1具体使用
由于是继承了`ValueAnimator`类，所以使用的方法十分类似：`XML `设置 /  `Java`设置

**设置方式1：`Java `设置**
```java
ObjectAnimator animator = ObjectAnimator.ofFloat(Object object, String property, float ....values);  

// ofFloat()作用有两个
// 1. 创建动画实例
// 2. 参数设置：参数说明如下
// Object object：需要操作的对象
// String property：需要操作的对象的属性
// float ....values：动画初始值 & 结束值（不固定长度）
// 若是两个参数a,b，则动画效果则是从属性的a值到b值
// 若是三个参数a,b,c，则则动画效果则是从属性的a值到b值再到c值
// 以此类推
// 至于如何从初始值 过渡到 结束值，同样是由估值器决定，此处ObjectAnimator.ofFloat（）是有系统内置的浮点型估值器FloatEvaluator，同ValueAnimator讲解
 // 设置动画运行的时长
anim.setDuration(500);
// 设置动画延迟播放时间
anim.setStartDelay(500);  
// 设置动画重复播放次数 = 重放次数+1
// 动画播放次数 = infinite时,动画无限重复
 anim.setRepeatCount(0);
 // 设置重复播放动画模式
 // ValueAnimator.RESTART(默认):正序重放
 // ValueAnimator.REVERSE:倒序回放
 anim.setRepeatMode(ValueAnimator.RESTART);
// 启动动画
animator.start();  

```
* 使用实例
此处先展示四种基本变换：平移、旋转、缩放 & 透明度
### a.透明度
```java
    //透明度
    private void runAlpha(){
        ObjectAnimator objectAnimator=ObjectAnimator.ofFloat(button,"alpha",1f,0f);
        objectAnimator.setDuration(2000);
        objectAnimator.start();
    }
```
### b.平移
```java
    //平移动画
    private void runTranslation(){
        ObjectAnimator objectAnimator=ObjectAnimator.ofFloat(button,"translationX",100f,300f);
        objectAnimator.setDuration(2000);
        objectAnimator.start();
    }
```
### c.缩放
```java
    //缩放
    private void runScale(){
        ObjectAnimator objectAnimator=ObjectAnimator.ofFloat(button,"scaleX",1f,3f,1f);
        objectAnimator.setDuration(2000);
        objectAnimator.start();
    }
```
### d.旋转
```java
    //旋转
    private void runRotation(){
        ObjectAnimator objectAnimator=ObjectAnimator.ofFloat(button,"rotation",0f,360f);
        objectAnimator.setDuration(2000);
        objectAnimator.start();
    }
```
* 在上面的讲解，我们使用了属性动画最基本的四种动画效果：透明度、平移、旋转 & 缩放
>即在`ObjectAnimator.ofFloat（）`的第二个参数`String property`传入`alpha`、`rotation`、`translationX` 和 `scaleY` 等属性。

| 属性 | 作用 | 数值类型 |
| ----   | :---   |  :---         |
| Alpha | 控制View的透明度 | float |
| TranslationX | 控制X方向的位移 | float |
| TranslationY| 控制Y方向的位移 | float | 
| ScaleX | 控制X方向的缩放倍数 | float | 
| ScaleY | 控制Y方向的缩放倍数 | float | 
| Rotation | 控制以屏幕方向为轴的旋转度数 | float | 
| RotationX | 控制以X轴为轴的旋转度数 | float | 
| RotationY | 控制以Y轴为轴的旋转度数 | float | 

**问题：那么ofFloat()的第二个参数还能传入什么属性值呢？**

答案：**任意属性值。**因为：

* `ObjectAnimator `类 对 对象属性值 进行改变从而实现动画效果的本质是：通过不断控制 值 的变化，再不断 自动 赋给对象的属性，从而实现动画效果.
* 而自动赋给对象的属性的本质是调用该对象属性的`set（）` & `get（）`方法进行赋值.
* 所以，`ObjectAnimator.ofFloat(Object object, String property, float ....values)`的第二个参数传入值的作用是：让`ObjectAnimator`类根据传入的属性名 去寻找 该对象对应属性名的` set（`） & `get（）`方法，从而进行对象属性值的赋值，如上面的例子：
```java
ObjectAnimator animator = ObjectAnimator.ofFloat(mButton, "rotation", 0f, 360f);
// 其实Button对象中并没有rotation这个属性值
// ObjectAnimator并不是直接对我们传入的属性名进行操作
// 而是根据传入的属性值"rotation" 去寻找对象对应属性名对应的get和set方法，从而通过set（） &  get（）对属性进行赋值

// 因为Button对象中有rotation属性所对应的get & set方法
// 所以传入的rotation属性是有效的
// 所以才能对rotation这个属性进行操作赋值
public void setRotation(float value);  
public float getRotation();  

// 实际上，这两个方法是由View对象提供的，所以任何继承自View的对象都具备这个属性
```
自动赋值的逻辑：

1. 初始化时，如果属性的初始值没有提供，则调用属性的 `get（）`进行取值；
2. 当 值 变化时，用对象该属性的` set（）`方法，从而从而将新的属性值设置给对象属性。

所以：
* `ObjectAnimator` 类针对的是**任意对象** & **任意属性值**，并不是单单针对于View对象.
* 如果需要采用`ObjectAnimator `类实现动画效果，那么需要操作的对象就必须有该属性的`set（）` &` get（）`.

### 3.额外的使用方法
#### 3.1 组合动画（`AnimatorSet` 类）
* 单一动画实现的效果相当有限，更多的使用场景是同时使用多种动画效果，即组合动画。
* 具体使用：
```java
AnimatorSet.play(Animator anim)   ：播放当前动画
AnimatorSet.after(long delay)   ：将现有动画延迟x毫秒后执行
AnimatorSet.with(Animator anim)   ：将现有动画和传入的动画同时执行
AnimatorSet.after(Animator anim)   ：将现有动画插入到传入的动画之后执行
AnimatorSet.before(Animator anim) ：  将现有动画插入到传入的动画之前执行
```
* 实例
主要动画是平移，平移过程中伴随旋转动画，平移完后进行透明度变化
```java
    //组合动画
    private void runSetAnimation(){
        // 步骤1：设置需要组合的动画效果
        // 平移动画
        ObjectAnimator translation = ObjectAnimator.ofFloat(button, "translationX", 0, 300,0);
        // 旋转动画
        ObjectAnimator rotate = ObjectAnimator.ofFloat(button, "rotation", 0f, 360f);
        // 透明度动画
        ObjectAnimator alpha = ObjectAnimator.ofFloat(button, "alpha", 1f, 0f, 1f);
        // 步骤2：创建组合动画的对象
        AnimatorSet animSet = new AnimatorSet();
        // 步骤3：根据需求组合动画
        animSet.play(translation).with(rotate).before(alpha);
        animSet.setDuration(5000);
        // 步骤4：启动动画
        animSet.start();
    }
```
#### 3.2监听动画
* `Animation`类通过监听动画开始 / 结束 / 重复 / 取消时刻来进行一系列操作，如跳转页面等等。
* 通过在`Java`代码里`addListener（）`设置。
```java
Animation.addListener(new AnimatorListener() {
          @Override
          public void onAnimationStart(Animation animation) {
              //动画开始时执行
          }
      
           @Override
          public void onAnimationRepeat(Animation animation) {
              //动画重复时执行
          }

         @Override
          public void onAnimationCancel()(Animation animation) {
              //动画取消时执行
          }
    
          @Override
          public void onAnimationEnd(Animation animation) {
              //动画结束时执行
          }
      });

// 特别注意：每次监听必须4个方法都重写。
```
* 因`Animator`类、`AnimatorSet`类、`ValueAnimator`、`ObjectAnimator`类存在以下继承关系:
![各类继承关系.jpg](https://github.com/WenJunKing/MyNote/blob/master/pics/1d60fc359f4bb73b26f31fd78c4b9e3e_944365-56dfb73edfed1293.png)