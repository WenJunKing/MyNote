 #���Զ���֮VauleAnimator
---
####1.����ԭ��
 * ��һ����ʱ�����ͨ�����ϵĶ�ֵ���иı䣬�����ϵؽ���ֵ��ֵ����������ԣ��Ӷ�ʵ�ָö����������ϵĶ���Ч����


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
        // ��ʼ������
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