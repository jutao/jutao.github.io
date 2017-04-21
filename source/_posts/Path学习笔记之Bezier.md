---
title: Path学习笔记之Bezier
tags:
- Path
- 自定义控件
- 学习笔记
- 动画
categories:
- Android
---

# 贝塞尔曲线能干什么？
贝塞尔曲线的运用是十分广泛的，可以说**贝塞尔曲线奠定了计算机绘图的基础(_因为它可以将任何复杂的图形用精确的数学语言进行描述_)**，在你不经意间就已经使用过它了。

你会使用Photoshop的话，你可能会注意到里面有一个**钢笔工具**，这个钢笔工具核心就是贝塞尔曲线。

你说你不会PS？ 没关系，你如果看过前面的文章或者用过2D绘图，肯定绘制过圆，圆弧，圆角矩形等这些东西。这里面的圆弧部分全部都是贝塞尔曲线的运用。

贝塞尔曲线作用十分广泛，简单举几个的栗子:
> * QQ小红点拖拽效果
* 一些炫酷的下拉刷新控件
* 阅读软件的翻书效果
* 一些平滑的折线图的制作
* 很多炫酷的动画效果

# 贝塞尔曲线的原理
贝塞尔曲线是用一系列点来控制曲线状态的，我将这些点简单分为两类：

| 类型   | 作用           |
| ---- | ------------ |
| 数据点  | 确定曲线的起始和结束位置 |
| 控制点  | 确定曲线的弯曲程度    |

> 此处暂时仅作了解概念，接下来就会讲解其中详细的含义。

## 一阶曲线原理

一阶曲线是没有控制点的，仅有两个数据点(A 和 B)，最终效果一个线段。

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f35of045w8j308c0dwq2z.jpg)

> **上图表示的是一阶曲线生成过程中的某一个阶段，动态过程可以参照下图(本文中贝塞尔曲线相关的动态演示图片来自维基百科)。**

![](https://upload.wikimedia.org/wikipedia/commons/0/00/B%C3%A9zier_1_big.gif)

> **PS：一阶曲线其实就是 Path 的 lineTo 方法。**

## 二阶曲线原理
二阶曲线由两个数据点(A 和 C)，一个控制点(B)来描述曲线状态，大致如下：

![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f35p4913k7j308c0dw74d.jpg)

上图中红色曲线部分就是传说中的二阶贝塞尔曲线，那么这条红色曲线是如何生成的呢？接下来我们就以其中的一个状态分析一下：

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f361bjqj2vj308c0dwwem.jpg)

连接AB BC，并在AB上取点D，BC上取点E，使其满足条件：
<img src="http://chart.googleapis.com/chart?cht=tx&chl=%5Cfrac%7BAD%7D%7BAB%7D%20%3D%20%5Cfrac%7BBE%7D%7BBC%7D" style="border:none;" />

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f361oje6h1j308c0dwdg0.jpg)

连接DE，取点F，使得:
<img src="http://chart.googleapis.com/chart?cht=tx&chl=%5Cfrac%7BAD%7D%7BAB%7D%20%3D%20%5Cfrac%7BBE%7D%7BBC%7D%20%3D%20%5Cfrac%7BDF%7D%7BDE%7D" style="border:none;" />

这样获取到的点F就是贝塞尔曲线上的一个点，动态过程如下：

![](https://upload.wikimedia.org/wikipedia/commons/3/3d/B%C3%A9zier_2_big.gif)

> **PS: 二阶曲线对应的方法是quadTo**

## 三阶曲线原理
三阶曲线由两个数据点(A 和 D)，两个控制点(B 和 C)来描述曲线状态，如下：

![](http://ww2.sinaimg.cn/large/005Xtdi2gw1f36myeqcu5j308c0dwdg2.jpg)

三阶曲线计算过程与二阶类似，具体可以见下图动态效果：

![](https://upload.wikimedia.org/wikipedia/commons/d/db/B%C3%A9zier_3_big.gif)

> **PS: 三阶曲线对应的方法是cubicTo**

## 其他
> ** [贝塞尔曲线速查表](https://github.com/GcsSloop/AndroidNote/blob/master/QuickChart/Bezier.md)**

> ** 强烈推荐[点击这里](http://bezier.method.ac/)练习贝塞尔曲线，可以加深对贝塞尔曲线的理解程度。**

# 贝塞尔曲线基本用法
## 一阶曲线
一阶曲线是一条线段，非常简单，可以参见上一篇文章
[Path之基本操作](https://github.com/jutao/AndroidNode/blob/master/2017/2017.3/Path%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%E4%B9%8B%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C.md)，此处就不详细讲解了。

## 二阶曲线
代码如下：

```java
public class Bezier extends View{
    private Paint mPaint;
    private PointF start,end,control;
    private int centerX,centerY;

    public Bezier(Context context) {
        this(context,null);
    }

    public Bezier(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs,0);
    }

    public Bezier(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private void init() {
        start=new PointF(0,0);
        end=new PointF(0,0);
        control=new PointF(0,0);

        mPaint=new Paint();
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setColor(Color.BLACK);
        mPaint.setStrokeWidth(8);
        mPaint.setTextSize(60);
        mPaint.setAntiAlias(true);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);

        centerX=w/2;
        centerY=h/2;

        start.x=centerX-200;
        start.y=centerY;

        end.x=centerX+200;
        end.y=centerY;

        control.x=centerX;
        control.y=centerY-100;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        drawPoint(canvas);
        drawAuxiliaryLine(canvas);
        drawBezierPath(canvas);
    }

    /**
     * 绘制数据点和控制点
     */
    private void drawPoint(Canvas canvas) {
        mPaint.setColor(Color.GRAY);
        canvas.drawCircle(start.x,start.y,4,mPaint);
        canvas.drawCircle(end.x,end.y,4,mPaint);
        canvas.drawCircle(control.x,control.y,4,mPaint);
    }

    /**
     * 绘制辅助线
     */
    private void drawAuxiliaryLine(Canvas canvas) {
        mPaint.setStrokeWidth(4);
        canvas.drawLine(start.x,start.y,control.x,control.y,mPaint);
        canvas.drawLine(end.x,end.y,control.x,control.y,mPaint);
    }

    /**
     * 绘制二阶曲线
     */
    private void drawBezierPath(Canvas canvas) {
        mPaint.setColor(Color.RED);
        mPaint.setStrokeWidth(8);
        Path path=new Path();
        path.moveTo(start.x,start.y);
        path.quadTo(control.x,control.y,end.x,end.y);
        canvas.drawPath(path,mPaint);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        control.x=event.getX();
        control.y=event.getY();
        postInvalidate();
        return true;
    }
}
```

效果如下：

![Bezier.gif](http://upload-images.jianshu.io/upload_images/3054656-a7f7504b98e1a36b.gif?imageMogr2/auto-orient/strip)

## 三阶曲线
代码如下：
```java
public class Bezier2 extends View{
    private Paint mPaint;
    private PointF start,end,control1,control2;
    private int centerX,centerY;
    private boolean isControl1=true;

    public Bezier2(Context context) {
        this(context,null);
    }

    public Bezier2(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs,0);
    }

    public Bezier2(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private void init() {
        start=new PointF(0,0);
        end=new PointF(0,0);
        control1=new PointF(0,0);
        control2=new PointF(0,0);

        mPaint=new Paint();
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setColor(Color.BLACK);
        mPaint.setStrokeWidth(8);
        mPaint.setTextSize(60);
        mPaint.setAntiAlias(true);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);

        centerX=w/2;
        centerY=h/2;

        start.x=centerX-300;
        start.y=centerY;

        end.x=centerX+300;
        end.y=centerY;

        control1.x=centerX-100;
        control1.y=centerY-200;

        control2.x=centerX+100;
        control2.y=centerY-200;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        drawPoint(canvas);
        drawAuxiliaryLine(canvas);
        drawBezierPath(canvas);
    }

    /**
     * 绘制数据点和控制点
     */
    private void drawPoint(Canvas canvas) {
        mPaint.setColor(Color.GRAY);
        canvas.drawCircle(start.x,start.y,4,mPaint);
        canvas.drawCircle(end.x,end.y,4,mPaint);
        canvas.drawCircle(control1.x,control1.y,4,mPaint);
        canvas.drawCircle(control2.x,control2.y,4,mPaint);
    }

    /**
     * 绘制辅助线
     */
    private void drawAuxiliaryLine(Canvas canvas) {
        mPaint.setStrokeWidth(4);
        canvas.drawLine(start.x,start.y,control1.x,control1.y,mPaint);
        canvas.drawLine(end.x,end.y,control2.x,control2.y,mPaint);
        canvas.drawLine(control1.x,control1.y,control2.x,control2.y,mPaint);

    }

    /**
     * 绘制三阶曲线
     */
    private void drawBezierPath(Canvas canvas) {
        mPaint.setColor(Color.RED);
        mPaint.setStrokeWidth(8);
        Path path=new Path();
        path.moveTo(start.x,start.y);
        path.cubicTo(control1.x,control1.y,control2.x,control2.y,end.x,end.y);
        canvas.drawPath(path,mPaint);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if(isControl1){
            control1.x=event.getX();
            control1.y=event.getY();
        }else {
            control2.x=event.getX();
            control2.y=event.getY();
        }

        postInvalidate();
        return true;
    }

    public void isControl1(boolean control1) {
        isControl1 = control1;
    }
}
```

主要的区别就是多了一个控制点，效果如下：


![Bezier.gif](http://upload-images.jianshu.io/upload_images/3054656-0e9407c333224757.gif?imageMogr2/auto-orient/strip)

## 降阶与升阶
> 三阶曲线相比于二阶曲线可以制作更加复杂的形状，但是对于高阶的曲线，用低阶的曲线组合也可达到相同的效果，就是传说中的**降阶**。因此我们对贝塞尔曲线的封装方法一般最高只到三阶曲线。

| 类型   | 释义                               | 变化                         |
| ---- | -------------------------------- | -------------------------- |
| 降阶   | 在保持曲线形状与方向不变的情况下，减少控制点数量，即降低曲线阶数 | 方法变得简单，数据点变多，控制点可能减少，灵活性变弱 |
| 升阶   | 在保持曲线形状与方向不变的情况下，增加控制点数量，即升高曲线阶数 | 方法更加复杂，数据点不变，控制点增加，灵活性变强   |

# 贝塞尔曲线使用实例
在制作这个实例之前，首先要明确一个内容，就是在什么情况下需要使用贝塞尔曲线？

> 需要绘制不规则图形时？ 当然不是！目前来说，使用贝塞尔曲线主要有以下几个方面(摘抄)

| 序号   | 内容                           | 用例             |
| ---- | ---------------------------- | -------------- |
| 1    | 事先不知道曲线状态，需要实时计算时            | 天气预报气温变化的平滑折线图 |
| 2    | 显示状态会根据用户操作改变时               | QQ小红点，仿真翻书效果   |
| 3    | 一些比较复杂的运动状态(配合PathMeasure使用) | 复杂运动状态的动画效果    |
至于只需要一个静态的曲线图形的情况，用图片岂不是更好，大量的计算会很不划算。

如果是显示SVG矢量图的话，已经有相关的解析工具了(内部依旧运用的有贝塞尔曲线)，不需要手动计算。

**贝塞尔曲线的主要优点是可以实时控制曲线状态，并可以通过改变控制点的状态实时让曲线进行平滑的状态变化。**

我们要实现的效果是使一个圆渐变为心形

代码:

```java
public class BezierHeartView extends View {

  // 一个常量，用来计算绘制圆形贝塞尔曲线控制点的位置
  private static final float C = 0.551915024494f;

  private int mCenterX;
  private int mCenterY;
  //圆的半径
  private float mCircleRadius = 200;
  // 圆形的控制点与数据点的差值
  private float mDifference = mCircleRadius * C;
  private Paint mPaint;

  // 顺时针记录绘制圆形的四个数据点
  private float[] mData = new float[8];

  // 顺时针记录绘制圆形的八个控制点
  private float[] mCtrl = new float[16];
  private Paint mFuZhuPaint;

  // 变化总时长
  private float mDuration = 1000;
  // 当前已进行时长
  private float mCurrent = 0;
  // 将时长总共划分多少份
  private float mCount = 100;
  // 每一份的时长
  private float mPiece = mDuration / mCount;

  public BezierHeartView(Context context) {
    this(context, null);
  }

  public BezierHeartView(Context context, @Nullable AttributeSet attrs) {
    this(context, attrs, 0);
  }

  public BezierHeartView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);
    init();
  }

  private void init() {
    mPaint = new Paint();
    mPaint.setColor(Color.BLACK);
    mPaint.setStrokeWidth(8);
    mPaint.setStyle(Paint.Style.STROKE);
    mPaint.setTextSize(60);

    mFuZhuPaint = new Paint();
    mFuZhuPaint.setStrokeWidth(5);
    mFuZhuPaint.setColor(Color.RED);
    mFuZhuPaint.setStyle(Paint.Style.STROKE);

    //初始化数据点
    mData[0] = 0;
    mData[1] = mCircleRadius;

    mData[2] = mCircleRadius;
    mData[3] = 0;

    mData[4] = 0;
    mData[5] = -mCircleRadius;

    mData[6] = -mCircleRadius;
    mData[7] = 0;

    //初始化控制点
    mCtrl[0] = mData[0] + mDifference;
    mCtrl[1] = mData[1];

    mCtrl[2] = mData[2];
    mCtrl[3] = mData[3] + mDifference;

    mCtrl[4] = mData[2];
    mCtrl[5] = mData[3] - mDifference;

    mCtrl[6] = mData[4] + mDifference;
    mCtrl[7] = mData[5];

    mCtrl[8] = mData[4] - mDifference;
    mCtrl[9] = mData[5];

    mCtrl[10] = mData[6];
    mCtrl[11] = mData[7] - mDifference;

    mCtrl[12] = mData[6];
    mCtrl[13] = mData[7] + mDifference;

    mCtrl[14] = mData[0] - mDifference;
    mCtrl[15] = mData[1];
  }

  @Override protected void onSizeChanged(int w, int h, int oldw, int oldh) {
    super.onSizeChanged(w, h, oldw, oldh);
    mCenterX = w / 2;
    mCenterY = h / 2;
  }

  @Override protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    // 绘制坐标系
    drawCoordinateSystem(canvas);
    //绘制辅助线
    drawAuxiliaryLine(canvas);
    // 绘制贝塞尔曲线
    drawBezier(canvas);
    //幻化为心
    changeBezier(canvas);
  }

  private void changeBezier(Canvas canvas) {
    mCurrent+=mPiece;
    if(mCurrent<=mDuration){
      mData[1]+=120/mCount;
      mData[5]+=150/mCount;

      mCtrl[0]-=80/mCount;
      mCtrl[14]+=80/mCount;

      mCtrl[4]-=20/mCount;
      mCtrl[10]+=20/mCount;

      postInvalidateDelayed((long) mPiece);
    }
  }

  private void drawBezier(Canvas canvas) {
    mPaint.setColor(Color.RED);
    mPaint.setStrokeWidth(8);
    mPaint.setStyle(Paint.Style.STROKE);
    Path path = new Path();
    path.moveTo(mData[0], mData[1]);
    for (int i = 2, j = 0; j < mCtrl.length; i += 2, j += 4) {
      path.cubicTo(mCtrl[j], mCtrl[j + 1], mCtrl[j + 2], mCtrl[j + 3], mData[i % mData.length],
          mData[(i + 1) % mData.length]);
    }

    canvas.drawPath(path, mPaint);
  }

  private void drawAuxiliaryLine(Canvas canvas) {
    mPaint.setColor(Color.GRAY);
    mPaint.setStrokeWidth(2);
    mPaint.setStyle(Paint.Style.FILL);
    for (int i = 0, j = 0; i < mData.length; i += 2, j += 4) {
      canvas.drawCircle(mData[i], mData[i + 1], 4, mPaint);
      canvas.drawCircle(mCtrl[j], mCtrl[j + 1], 4, mPaint);
      canvas.drawCircle(mCtrl[j + 2], mCtrl[j + 3], 4, mPaint);
    }

    for (int i = 0, j = 0; j < mCtrl.length; i += 2, j += 4) {
      canvas.drawLine(mData[i], mData[i + 1], mCtrl[j], mCtrl[j + 1], mPaint);
      canvas.drawLine(mCtrl[j], mCtrl[j + 1], mCtrl[j + 2], mCtrl[j + 3], mPaint);
      canvas.drawLine(mCtrl[j + 2], mCtrl[j + 3], mData[(i + 2) % mData.length],
          mData[(i + 3) % mData.length], mPaint);
    }
  }

  private void drawCoordinateSystem(Canvas canvas) {
    canvas.drawLine(0, mCenterY, mCenterX * 2, mCenterY, mFuZhuPaint);
    canvas.drawLine(mCenterX, 0, mCenterX, mCenterY * 2, mFuZhuPaint);
    canvas.translate(mCenterX, mCenterY);
  }
}
```

预览效果：


![贝塞尔.gif](http://upload-images.jianshu.io/upload_images/3054656-6126e83d6fb071f6.gif?imageMogr2/auto-orient/strip)


# 参考资料
[Path之贝塞尔曲线](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B06%5DPath_Bezier.md)
