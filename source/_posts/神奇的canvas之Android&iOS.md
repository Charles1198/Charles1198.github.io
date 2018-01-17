---
title: 神奇的 Canvas 之 Android&iOS
date: 2018-1-11 18:00:00
tags: js
---

最近公司事不多，自从前些天[学习使用 canvas](http://jiayueji.cn/2018/01/08/%E7%A5%9E%E5%A5%87%E7%9A%84canvas/) 之后这几天又抽空实现了几个类似的效果：

![](https://github.com/Charles1198/Charles1198.github.io/blob/master/image/round.gif?raw=true)
![](https://github.com/Charles1198/Charles1198.github.io/blob/master/image/segment.gif?raw=true)
![](https://github.com/Charles1198/Charles1198.github.io/blob/master/image/circle.gif?raw=true)
![](https://github.com/Charles1198/Charles1198.github.io/blob/master/image/line.gif?raw=true)

是用 Vue 实现的，[代码在这里](https://github.com/Charles1198/VueDemo/tree/master/src/components/canvas)，实际效果还是很流畅的，只是 .gif 图片看起来有些掉帧。

今天闲来无事就想着将上述效果实现在 Android 和 iOS 上试试看。

# Android

在 Android 中首先想到的就是自定义 View 来实现。首先定义圆点的属性。
```
public class Point {
    /**
     * 圆点坐标及半径、透明度（黑色）
     */
    private float x;
    private float y;
    private float r;
    private int alpha;

    /**
     * 圆点横向和纵向移动速度
     */
    private float vx = (float) (Math.random() - 0.5);
    private float vy = (float) Math.random();

    public Point(float x, float y, float r, int alpha) {
        this.x = x;
        this.y = y;
        this.r = r;
        this.alpha = alpha;
    }

    // get/set 方法
}
```
自定义 View MovingPointView 继承自 View
```
public class MovingPointView extends View {
    /**
     * 画布长宽
     */
    private int width;
    private int height;

    /**
     * 画笔
     */
    private Paint paint;

    /**
     * 圆点数量
     */
    private int pointCount = 1000;

    /**
     * 圆点数组
     */
    private List<Point> pointList = new ArrayList<>();
}
```
创建画笔
```
private void initPatin() {
    paint = new Paint(Paint.ANTI_ALIAS_FLAG);
    initTimer();
}
```
在构造方法里调用
```
public MovingPointView(Context context, @Nullable AttributeSet attrs) {
    super(context, attrs);
    initPatin();
}
```
接下来创建圆点数组
```
private void initPoint() {
    if (pointList.size() > 0) {
        return;
    }
    for (int i = 0; i < pointCount; i++) {
        Point p = new Point((float) (Math.random() * width), (float) (Math.random() * height),
                (float) (Math.random() * 15), (int) (Math.random() * 120 + 20));
        pointList.add(p);
    }
}
```
由于要保证圆点的坐标在 View 范围之内，所以要等到 onMeasure 方法执行后得到 View 的宽高再创建圆点数组。因为 onMeasure 方法会被执行多次，所以在 initPoint 方法中要判断 pointList 如果已经构造过就不再构造。
```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    width = this.getMeasuredWidth();
    height = this.getMeasuredHeight();

    initPoint();
}
```
此时调用 onDraw 方法就可以画出点点啦。
```
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    for (Point p : pointList) {
        paint.setARGB(p.getAlpha(), 0, 0, 0);
        paint.setStrokeWidth(p.getR());
        canvas.drawCircle(p.getX(), p.getY(), p.getR(), paint);
    }
}
```
不过这时圆点还是静态的不会动，要让圆点动起来还需要为每一个圆点加上动画......扯淡，并不是。跟[《神奇的 Canvas》](http://jiayueji.cn/2018/01/08/%E7%A5%9E%E5%A5%87%E7%9A%84canvas/)这篇文章中一样，我们通过定时改变圆点的坐标并刷新 View 的方法来实现动画。

向 Point 类中加入移动方法：
```
/**
* 移动圆点，范围不超过画布
*
* @param width 画布宽
* @param height 画布高
*/
public void move(int width, int height) {
    if (x <= 0 || x >= width) {
        vx = 0 - vx;
    }
    if (y <= 0) {
        y += height;
    }
    this.x += vx;
    this.y -= vy;
}
```
定时使用 Timer
```
/**
* 定时器
*/
private Timer timer;

private void initTimer() {
    @SuppressLint("HandlerLeak")
    final Handler mainHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            //刷新 View
            invalidate();
        }
    };

    timer = new Timer();
    timer.schedule(new TimerTask() {
        @Override
        public void run() {
            //run() 方法执行在非主线程，要刷新 view 要借助 Handler
            mainHandler.sendMessage(new Message());
        }
    }, 60, 60);//延时 60 毫秒，每 60 毫秒执行一次
}
```
记得在 View 被销毁的时候释放定时器
```
@Override
protected void onDetachedFromWindow() {
    super.onDetachedFromWindow();

    if (timer != null) {
        timer.cancel();
        timer = null;
    }
}
```
最后在每次画圆点的时候移动一下圆点的位置，这样在下一次画圆点的时候就会有动画效果
```
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    for (Point p : pointList) {
        paint.setARGB(p.getAlpha(), 0, 0, 0);
        paint.setStrokeWidth(p.getR());
        canvas.drawCircle(p.getX(), p.getY(), p.getR(), paint);

        //每次画完就移动圆点
        p.move(width, height);
    }
}
```
最终效果如下：

![](https://github.com/Charles1198/Charles1198.github.io/blob/master/image/view.gif?raw=true)

由于是 .gif 图片，看起来会不流畅，其实模拟器上还是很流畅的。

此时圆点的数量只有 60 个，当我把圆点的数量调整到 1000 个的时候，动画就有些许的卡顿了，这时我想到了 SurfaceView。将 MovingPointView 的父类换成 SurfaceView 会流畅很多，但是这不是本文的重点就不展开了，代码在[这里](https://github.com/Charles1198/AppForLearn-android/blob/master/app/src/main/java/com/bqteam/appforlearn/function/canvas/MovingPointSurfaceView.java)。


# iOS

由于在 iOS 上的实现方式跟 Android 很类似，代码我就不贴了，在[这里](https://github.com/Charles1198/AppForLearn-iOS/blob/master/AppForLearn/functions/MovingPointView.swift)就可以找到。

最后实现效果

![](https://github.com/Charles1198/Charles1198.github.io/blob/master/image/ios.gif?raw=true)
