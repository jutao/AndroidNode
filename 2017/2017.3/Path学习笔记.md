# 直线与点的操作
## lineTo
    //创建一条从原点到坐标(300,300)的直线
    mPath.lineTo(500,500);

    //创建从(500,500)到(100,200)的一条直线
    mPath.lineTo(100,200);
效果如下：

![lineTo效果.png](http://upload-images.jianshu.io/upload_images/3054656-9fa8ff91af3e6ca6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



所以可知 lineTo 方法的连接的起点是由lineTo方法上一个Path操作决定的。

## moveTo
    //这样改写的话是画一条从(500,500)到(100,200)的线
    mPath.moveTo(500,500);
    mPath.lineTo(100,200);

效果如下:

![moveTo效果.png](http://upload-images.jianshu.io/upload_images/3054656-2a235e192504b31f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以可知 moveTo 方法的作用为将下次画路径起点移动到(x,y)

## setLastPoint
    mPath.lineTo(300,300);
    mPath.setLastPoint(200,200);
    mPath.lineTo(200,400);
效果如下:

![setLastPoint效果.png](http://upload-images.jianshu.io/upload_images/3054656-0ac7b2790c36c336.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以可知 setLastPoint 的作用为改变上一次操作路径的结束坐标点。

因此我们可以总结出 moveTo 和 lineTo 的区别为：
- moveTo 影响上一次操作不影响上一次
- setLastPoint上一次和下一次操作都影响

## close
    mPath.lineTo(300,300);
    mPath.setLastPoint(200,200);
    mPath.lineTo(200,400);
    mPath.close();

效果如下：

![close效果.png](http://upload-images.jianshu.io/upload_images/3054656-76e5e3e3aa79e2fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

观察可知 close 的效果为用线段连接起始点和终点，除非起始点等于终点。

# 基本形状
## addXxx,arcTo

方法一览：

    //矩形
    public void addRect(RectF rect, Direction dir)

    public void addRect(float left, float top, float right, float bottom, Direction dir)

    //圆形
    public void addCircle(float x, float y, float radius, Direction dir)


    //圆角矩形
    public void addRoundRect(RectF rect, float[] radii, Direction dir)

    public void addRoundRect (float left,float top,float right,float bottom,float rx,float ry,Path.Direction dir)

    public void addRoundRect (RectF rect,float[] radii,Path.Direction dir)

    public void addRoundRect (float left,float top,float right,float bottom,float[] radii,Path.Direction dir)

    //椭圆
    public void addOval(RectF oval, Direction dir)

    public void addOval (float left,float top,float right,float bottom,Path.Direction dir)

    //圆弧
    public void addArc (RectF oval, float startAngle, float sweepAngle)
    public void arcTo (RectF oval, float startAngle, float sweepAngle)
    public void arcTo (RectF oval, float startAngle, float sweepAngle, boolean forceMoveTo)

    // 添加Path
    public void addPath (Path src)
    public void addPath (Path src, float dx, float dy)
    public void addPath (Path src, Matrix matrix)

### addRect（矩形）

    mPath.addRect(50,50,150,150,Direction.CW);

    mPath.addRect(50,250,150,350,Direction.CCW);

效果如下：

![矩形.png](http://upload-images.jianshu.io/upload_images/3054656-a63df17a624ec167.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上下两个方法画出来的一模一样，那他们有什么区别呢？我们试着用一下刚学的 setLastPoint 看效果。

    mPath.addRect(50,50,150,150,Direction.CW);
    //改变最后一笔的位置
    mPath.setLastPoint(50,100);

    mPath.addRect(50,250,150,350,Direction.CCW);
    mPath.setLastPoint(150,300);

效果如下：

![矩形顺时逆时对比.png](http://upload-images.jianshu.io/upload_images/3054656-0cde63a4201a51d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以我们可以看出 Direction.CW 的最后一笔是左下角，Direction.CCW 的最后一笔是右上角，而且 Android 源码中给 Direction.CW 的注释为 clockwise(顺时针),Direction.CCW 的注释为 counter-clockwise(逆时针),通过这些我们可以很清楚的知道以上两种矩形画法的区别。

而矩形的另外一种画法 public void addRect(RectF rect, Direction dir) 和上面的方法其实一样，只是把坐标封装到了 RectF 对象中而已。

### addCircle（圆形）

    mPath.addCircle(210,210,200, Path.Direction.CW);

    mPath.addCircle(800,800,200, Path.Direction.CCW);

效果如下：


![圆形.png](http://upload-images.jianshu.io/upload_images/3054656-f25c8134566cb3d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

画圆非常简单，x，y 代表圆心坐标，radius 代表半径，dir 和矩形一样，代表顺时针或者逆时针。

    mPath.addCircle(210,210,200, Path.Direction.CW);
    mPath.setLastPoint(210,210);

    mPath.addCircle(800,800,200, Path.Direction.CCW);
    mPath.setLastPoint(800,800);

效果如下：


![image.png](http://upload-images.jianshu.io/upload_images/3054656-b24fe1e1da6d757f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### addRoundRect（圆角矩形）

圆角矩形主要有两种画法，一种是圆角弧度统一，第二种是定制每一个圆角的弧度。用法如下

    mPath = new Path();
    mPath.addRoundRect(rect1,100,100, Path.Direction.CW);

    RectF rect2=new RectF(100,600,800,1000);

    /*必须传入8个数值，分四组，分别对应每个角所使用的椭圆的横轴半径和纵轴半径，
     如｛x1,y1,x2,y2,x3,y3,x4,y4｝，其中，x1,y1对应第一个角的（左上角）用来产
     生圆角的椭圆的横轴半径和纵轴半径，其它类推……*/
    float[] radii={100,100,150,150,200,200,300,200};
    mPath.addRoundRect(rect2,radii,Path.Direction.CCW);

效果如下：

![圆角矩形](http://upload-images.jianshu.io/upload_images/3054656-7b6e69e14822fa9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我标了一下每个圆角半径，很丑：

![圆角半径](http://upload-images.jianshu.io/upload_images/3054656-571ad0c63463d64d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### addOval（椭圆）

    RectF rect = new RectF(100, 100, 800, 500);
    mPath = new Path();
    mPath.addOval(rect, Path.Direction.CW);
    //将矩形变为正方形看效果
    rect = new RectF(100, 600, 300, 800);
    mPath.addOval(rect, Path.Direction.CW);

效果如下：


![椭圆](http://upload-images.jianshu.io/upload_images/3054656-8305a07ea17626a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### addArc与arcTo（圆弧）

    RectF rect = new RectF(100, 100, 800, 500);
    //开始的角度
    float startAngle=90;
    //扫过的角度
    float sweepAngle=180;
    mPath = new Path();
    mPath.addArc(rect,startAngle,sweepAngle);

效果：

![弧](http://upload-images.jianshu.io/upload_images/3054656-014416205f8d730e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

关于起始角度以及扫过的角度这些是怎么来的，我理解的是实际上我们是在指定矩形内画一个内切椭圆，通过指定角度在这个椭圆上截取一部分，这一部分就是我们所画的弧。示意图如下，不要嫌丑：

![弧](http://upload-images.jianshu.io/upload_images/3054656-2f1bf4d694f95e7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
