# 属性动画之VauleAnimator
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