 #属性动画之VauleAnimator
---
####1.工作原理
 * 在一定的时间间隔里，通过不断的对值进行改变，并不断地将该值赋值给对象的属性，从而实现该对象在属性上的动画效果。


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