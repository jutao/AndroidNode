# 第一步：绘制蜘蛛网络
    private void init() {
        mainPaint=new Paint();
        mainPaint.setColor(Color.BLACK);
        mainPaint.setAntiAlias(true);
        mainPaint.setStrokeWidth(1);
        mainPaint.setStyle(Paint.Style.STROKE);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        radius=Math.min(w,h)/2*0.9f;
        centerX=w/2;
        centerY=h/2;
        //一旦size发生改变，重新绘制
        postInvalidate();
        super.onSizeChanged(w, h, oldw, oldh);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        drawPolygon(canvas);
    }

    /**
     * 绘制多边形
     * @param canvas
     */
    private void drawPolygon(Canvas canvas){
        Path path=new Path();
        //1度=1*PI/180   360度=2*PI   那么我们每旋转一次的角度为2*PI/内角个数
        //中心与相邻两个内角相连的夹角角度
        angle= (float) (2*Math.PI/count);
        //每个蛛丝之间的间距
        float r= radius/(count-1);
        for (int i = 0; i < count; i++) {
            //当前半径
            float curR=r*i;
            path.reset();
            for (int j = 0; j < count; j++) {
                if(j==0){
                    path.moveTo(centerX+curR,centerY);
                }else {
                    //对于直角三角形sin(x)是对边比斜边，cos(x)是底边比斜边，tan(x)是对边比底边
                    //因此可以推导出:底边(x坐标)=斜边(半径)*cos(夹角角度)
                    //               对边(y坐标)=斜边(半径)*sin(夹角角度)
                    float x = (float) (centerX+curR*Math.cos(angle*j));
                    float y = (float) (centerY+curR*Math.sin(angle*j));
                    path.lineTo(x,y);
                }
            }
            path.close();
            canvas.drawPath(path,mainPaint);
        }

绘制蜘蛛网络其实就是绘制指定边数的正多边形，这一步比较简单，比较难的可能就是每个顶点的算法，相关注释我都写了，还有一张来自互联网的图以助于思考，如下：


![多边形夹角示意图](http://upload-images.jianshu.io/upload_images/3054656-f380ea0f04b8619b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

绘制出的多边形成品如下:

![多边形效果.gif](http://upload-images.jianshu.io/upload_images/3054656-35a0029632035330.gif?imageMogr2/auto-orient/strip)


动画效果只是写了 set 方法，用 handler 实现,代码如下：

    //设置数值种类
    public void setCount(int count) {
        this.count = count;
        postInvalidate();
    }

    //设置蜘蛛网颜色
    public void setMainPaint(Paint mainPaint) {
        this.mainPaint = mainPaint;
        postInvalidate();
    }

调用方法:

    mainPaint=new Paint();
    mainPaint.setAntiAlias(true);
    mainPaint.setStrokeWidth(1);
    mainPaint.setStyle(Paint.Style.STROKE);
    Handler handler=new Handler();
    for (int i = 3; i < 20; i++) {
    final int finalI = i;
    handler.postDelayed(new Runnable() {
        @Override
        public void run() {
            mRdv.setCount(finalI);
            mainPaint.setStrokeWidth(finalI);
            mRdv.setMainPaint(mainPaint);
        }
    },i*300);
    }

# 第二步：绘制对角线
    /**
     * 绘制直线
     */
    private void drawLines(Canvas canvas){
        Path path=new Path();
        for (int i = 0; i < count; i++) {
            path.reset();
            path.moveTo(centerX,centerY);
            float x = (float) (centerX+radius*Math.cos(angle*i));
            float y = (float) (centerY+radius*Math.sin(angle*i));
            path.lineTo(x,y);
            canvas.drawPath(path,mainPaint);
        }
    }

这一步比较简单，就是将中心点和各个顶点连接起来，效果如下:


![多边形效果.gif](http://upload-images.jianshu.io/upload_images/3054656-e8349daa51b18b38.gif?imageMogr2/auto-orient/strip)

# 第三步：绘制标题文字
    /**
     * 绘制标题文字
     *
     * @param canvas
     */
    private void drawTitle(Canvas canvas) {
        if (count != titles.size()) {
            return;
        }
        //相关知识点:http://mikewang.blog.51cto.com/3826268/871765/
        Paint.FontMetrics fontMetrics = textPaint.getFontMetrics();
        float fontHeight = fontMetrics.descent - fontMetrics.ascent;
        //绘制文字时不让文字和雷达图形交叉,加大绘制半径
        float textRadius = radius + fontHeight;
        double pi = Math.PI;
        for (int i = 0; i < count; i++) {
            float x = (float) (centerX + textRadius * Math.cos(angle * i));
            float y = (float) (centerY + textRadius * Math.sin(angle * i));
            //当前绘制标题所在顶点角度
            float degrees = angle * i;
            //从右下角开始顺时针画起,与真实坐标系相反
            if (degrees >= 0 && degrees < pi / 2) {//第四象限
                float dis=textPaint.measureText(titles.get(i))/(titles.get(i).length()-1);
                canvas.drawText(titles.get(i), x+dis, y, textPaint);
            } else if (degrees >= pi / 2 && degrees < pi) {//第三象限
                float dis=textPaint.measureText(titles.get(i))/(titles.get(i).length()-1);
                canvas.drawText(titles.get(i), x-dis, y, textPaint);
            } else if (degrees >= pi && degrees < 3 * pi / 2) {//第二象限
                float dis=textPaint.measureText(titles.get(i))/(titles.get(i).length());
                canvas.drawText(titles.get(i), x-dis, y, textPaint);
            } else if (degrees >= 3 * pi / 2 && degrees <= 2 * pi) {//第一象限
                canvas.drawText(titles.get(i), x, y, textPaint);
            }

        }

    }

效果如下：

![image.png](http://upload-images.jianshu.io/upload_images/3054656-c9132f5ff1404eaf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 第四步：绘制覆盖区域
要绘制覆盖区域，首先要指定最大值和每个分类的具体数值,有了这些数值之后，就可以绘制了。
代码如下：

    /**
     * 绘制覆盖区域
     */
    private void drawRegion(Canvas canvas){
        valuePaint.setAlpha(255);
        Path path=new Path();
        for (int i = 0; i < count; i++) {
            //计算该数值与最大值比例
            Double perCenter = data.get(i)/maxValue;
            //小圆点所在位置距离圆心的距离
            double perRadius=perCenter*radius;
            float x = (float) (centerX + perRadius * Math.cos(angle * i));
            float y = (float) (centerY + perRadius * Math.sin(angle * i));
            if(i==0){
                path.moveTo(x,y);
            }else {
                path.lineTo(x,y);
            }
            //绘制小圆点
            canvas.drawCircle(x,y,10,valuePaint);
        }
        //闭合覆盖区域
        path.close();
        valuePaint.setStyle(Paint.Style.STROKE);
        //绘制覆盖区域外的连线
        canvas.drawPath(path, valuePaint);
        //填充覆盖区域
        valuePaint.setAlpha(128);
        valuePaint.setStyle(Paint.Style.FILL);
        canvas.drawPath(path,valuePaint);
    }

看一下效果：


![image.png](http://upload-images.jianshu.io/upload_images/3054656-d62b189d8dce1fac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再来看一下动态的效果吧：


![多边形效果.gif](http://upload-images.jianshu.io/upload_images/3054656-50a0b0225608fc7b.gif?imageMogr2/auto-orient/strip)

# 总结
终于完成了,全部代码在下面：

[Android雷达图全部代码 ](https://github.com/jutao/AndroidNode/blob/master/2017/2017.3/%E9%9B%B7%E8%BE%BE%E5%9B%BE%E5%85%A8%E9%83%A8%E4%BB%A3%E7%A0%81.md)

主要是参考 crazy__chen 大神的博客，链接贴在下面，做了一遍其实还蛮简单的，这个控件还有很多不完善的，如果实际使用需要改善的地方还有很多，如果有不足希望大家可以告诉我，谢谢！！

参考资料

[Android雷达图(蜘蛛网图)绘制 ](http://blog.csdn.net/crazy__chen/article/details/50163693)

[Path之基本操作](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B05%5DPath_Basic.md)
