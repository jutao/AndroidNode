# path 常用方法一览
> 为了兼容性(_偷懒_) 本表格中去除了部分API21(即安卓版本5.0)以上才添加的方法。

| 作用          | 相关方法                                     | 备注                                       |
| ----------- | ---------------------------------------- | ---------------------------------------- |
| 移动起点        | moveTo                                   | 移动下一次操作的起点位置                             |
| 设置终点        | setLastPoint                             | 重置当前path中最后一个点位置，如果在绘制之前调用，效果和moveTo相同   |
| 连接直线        | lineTo                                   | 添加上一个点到当前点之间的直线到Path                     |
| 闭合路径        | close                                    | 连接第一个点连接到最后一个点，形成一个闭合区域                  |
| 添加内容        | addRect, addRoundRect,  addOval, addCircle, 	addPath, addArc, arcTo | 添加(矩形， 圆角矩形， 椭圆， 圆， 路径， 圆弧) 到当前Path (注意addArc和arcTo的区别) |
| 是否为空        | isEmpty                                  | 判断Path是否为空                               |
| 是否为矩形       | isRect                                   | 判断path是否是一个矩形                            |
| 替换路径        | set                                      | 用新的路径替换到当前路径所有内容                         |
| 偏移路径        | offset                                   | 对当前路径之前的操作进行偏移(不会影响之后的操作)                |
| 贝塞尔曲线       | quadTo, cubicTo                          | 分别为二次和三次贝塞尔曲线的方法                         |
| rXxx方法      | rMoveTo, rLineTo, rQuadTo, rCubicTo      | **不带r的方法是基于原点的坐标系(偏移量)， rXxx方法是基于当前点坐标系(偏移量)** |
| 填充模式        | setFillType, getFillType, isInverseFillType, toggleInverseFillType | 设置,获取,判断和切换填充模式                          |
| 提示方法        | incReserve                               | 提示Path还有多少个点等待加入**(这个方法貌似会让Path优化存储结构)** |
| 布尔操作(API19) | op                                       | 对两个Path进行布尔运算(即取交集、并集等操作)                |
| 计算边界        | computeBounds                            | 计算Path的边界                                |
| 重置路径        | reset, rewind                            | 清除Path中的内容<br/> **reset不保留内部数据结构，但会保留FillType.**<br/> **rewind会保留内部的数据结构，但不保留FillType** |
| 矩阵操作        | transform                                | 矩阵变换                                     |

# Path含义
Path封装了由直线和曲线(二次，三次贝塞尔曲线)构成的几何路径。你能用Canvas中的drawPath来把这条路径画出来(同样支持Paint的不同绘制模式)，也可以用于剪裁画布和根据路径绘制文字。我们有时会用Path来描述一个图像的轮廓，所以也会称为轮廓线(轮廓线仅是Path的一种使用方法，两者并不等价)

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
## addRect（矩形）

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

## addCircle（圆形）

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


## addRoundRect（圆角矩形）

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

## addOval（椭圆）

    RectF rect = new RectF(100, 100, 800, 500);
    mPath = new Path();
    mPath.addOval(rect, Path.Direction.CW);
    //将矩形变为正方形看效果
    rect = new RectF(100, 600, 300, 800);
    mPath.addOval(rect, Path.Direction.CW);

效果如下：


![椭圆](http://upload-images.jianshu.io/upload_images/3054656-8305a07ea17626a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## addArc与arcTo（圆弧）

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

知道了addArc的用法之后，我们来看一下arcTo这个方法，这个方法也是用来画圆弧的，但是与addArc有些不同:
- addArc	直接添加一段圆弧
- arcTo	  添加一段圆弧，如果圆弧的起点与上一次Path操作的终点不一样的话，就会在这两个点连成一条直线

      RectF rect = new RectF(100, 100, 800, 500);
      //开始的角度
      float startAngle=90;
      //扫过的角度
      float sweepAngle=180;
      mPath = new Path();
      mPath.lineTo(100,100);
      mPath.arcTo(rect,startAngle,sweepAngle);

效果如下：


![image.png](http://upload-images.jianshu.io/upload_images/3054656-785a72beeca6dac4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，圆弧的起点并不是直线的终点，于是他们连接在了一起。如果你不想让他们连接怎么办？

    mPath.arcTo(rect,startAngle,sweepAngle,true);

arcTo 最后一个属性填 true 就可以了。效果如下：


![image.png](http://upload-images.jianshu.io/upload_images/3054656-255b070700818d89.png)

## addPath（添加Path）

先看最普通的 public void addPath (Path src)

    Path circle=new Path();
    RectF rect = new RectF(100, 100, 800, 500);
    mPath = new Path();
    mPath.addRect(rect, Path.Direction.CW);
    circle.addCircle(450,300,200, Path.Direction.CW);
    mPath.addPath(circle);

它将两个path合并在了一起，效果如下：


![image.png](http://upload-images.jianshu.io/upload_images/3054656-2d0a81593436197c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

addPath的第二个方法的 dx 和 dy 两个参数是什么意思呢？

    mPath.addPath(circle,350,-100);

其实它们是代表添加path后的位移值

效果如下：


![image.png](http://upload-images.jianshu.io/upload_images/3054656-4d28b8f8a6b14b86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 设置方法
## set()
    Path circle=new Path();
    RectF rect = new RectF(100, 100, 800, 500);
    mPath = new Path();
    mPath.addRect(rect, Path.Direction.CW);
    circle.addCircle(450,300,200, Path.Direction.CW);
    mPath.set(circle);

这个方法就是将path之前的矩形变成圆形。

效果如下：


![image.png](http://upload-images.jianshu.io/upload_images/3054656-546c79bf325f031e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## offset()
    RectF rect = new RectF(100, 100, 800, 500);
    mPath = new Path();
    mPath.addRect(rect, Path.Direction.CW);
    mPath.offset(150,150);


![image.png](http://upload-images.jianshu.io/upload_images/3054656-55ae6162b44b82ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

它的作用就是将path进行平移

offset() 还有一个参数 dst 是起什么作用呢？

    mPath=new Path();
    mPath.addCircle(50,50,200, Path.Direction.CW);
    Path temp=new Path();
    RectF rect = new RectF(100, 100, 800, 500);
    temp.addRect(rect, Path.Direction.CW);
    //相当于用set方法将 temp set 给了mPath，覆盖 mPath 原有的图案
    temp.offset(150,150,mPath);

效果如下：

![image.png](http://upload-images.jianshu.io/upload_images/3054656-55ae6162b44b82ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到 mPath 之前画的圆已经被覆盖

## reset()
这个方法很简单，就是将path的所有操作都清空掉。

# 判断方法
## isConvex()（这个方法在API21之后才有）

判断path是否为凸多边形，如果是就为true，反之为false。

要理解这个方法首先，我们要知道什么是凸多边形。

凸多边形的概念:

1. 每个内角小于180度
2. 任何两个顶点间的线段位于多边形的内部或边界上。

也就是说矩形，三角形，直线都是凸多边形，但是五角星那种形状就不是。现在我们用代码验证一下：

    mPath=new Path();
    mPath.moveTo(100,100);
    mPath.lineTo(200,200);
    mPath.lineTo(100,400);
    mPath.lineTo(300,50);
    mPath.close();
    System.out.println(mPath.isConvex());

效果如下：

![image.png](http://upload-images.jianshu.io/upload_images/3054656-b99b4c9a0f651743.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

很明显，这不是一个凸多边形，查看输出：

    I/System.out: false

## isEmpty
这个方法依然很简单，就是判断 path 中是否包含内容

    mPath=new Path();
    System.out.println(mPath.isEmpty());
    mPath.lineTo(300,50);
    System.out.println(mPath.isEmpty());

查看输出：

    I/System.out: true
    I/System.out: false

##  isRect
    mPath=new Path();
    RectF rect=new RectF();
    mPath.addRect(100,100,50,50,Path.Direction.CW);
    System.out.println(mPath.isRect(rect));
    System.out.println(rect);

  判断path是否是一个矩形，如果是一个矩形的话，将矩形的信息存到参数rect中。

  输出如下：

     I/System.out: true
     I/System.out: RectF(50.0, 50.0, 100.0, 100.0)
