# AnimatorEvents 效果 #
AnimatorEvents 是 ApiDemos 弹球动画效果之一，他的效果为绘制小球并移动。效果如下图所示：

起始：

![](http://i.imgur.com/TX0SeM5.png)

结束：

![](http://i.imgur.com/YrFSwuT.png)

下面来一步步分析如何从创建小球到实现小球的移动的。

# 创建小球 #
首先创建一个存储小球属性的类 ShapeHolder，然后开始绘制小球，绘制代码如下：

            private ShapeHolder createBall(float x, float y) {
            OvalShape circle = new OvalShape();
            circle.resize(50f, 50f);
            ShapeDrawable drawable = new ShapeDrawable(circle);
            ShapeHolder shapeHolder = new ShapeHolder(drawable);
            shapeHolder.setX(x - 25f);
            shapeHolder.setY(y - 25f);
            //下面是通过随机的方法去生成红绿蓝三个值.,从而组合成ARGB的值,设置为该圆的颜色
            int red = (int)(Math.random() * 255);
            int green = (int)(Math.random() * 255);
            int blue = (int)(Math.random() * 255);
            int color = 0xff000000 | red << 16 | green << 8 | blue;
            //每个ShapeDrawable对象都有自己的paint,直接getPaint()就能获取了
            Paint paint = drawable.getPaint(); //new Paint(Paint.ANTI_ALIAS_FLAG);
            int darkColor = 0xff000000 | red/4 << 16 | green/4 << 8 | blue/4;
            //给上面生成的ARGB值,添加圆的中心到边缘颜色从深到浅的渐变效果
            RadialGradient gradient = new RadialGradient(37.5f, 12.5f,
                    50f, color, darkColor, Shader.TileMode.CLAMP);
            paint.setShader(gradient);
            //到了这一步,圆的颜色效果已经确定了
            shapeHolder.setPaint(paint);
            return shapeHolder;
        }

# 小球移动 #
设置小球移动的代码如下：

            private void createAnimation() {
            if (animation == null) {
            	//创建小球纵向移动属性动画
                ObjectAnimator yAnim = ObjectAnimator.ofFloat(ball, "y",
                        ball.getY(), getHeight() - 50f).setDuration(1500);
                yAnim.setRepeatCount(0);
                yAnim.setRepeatMode(ValueAnimator.REVERSE);
                yAnim.setInterpolator(new AccelerateInterpolator(2f));
                yAnim.addUpdateListener(this);
                yAnim.addListener(this);

                //创建横向移动属性动画
                ObjectAnimator xAnim = ObjectAnimator.ofFloat(ball, "x",
                        ball.getX(), ball.getX() + 300).setDuration(1000);
                xAnim.setStartDelay(0);
                xAnim.setRepeatCount(0);
                xAnim.setRepeatMode(ValueAnimator.REVERSE);
                xAnim.setInterpolator(new AccelerateInterpolator(2f));

                ObjectAnimator alphaAnim = ObjectAnimator.ofFloat(ball, "alpha", 1f, .5f).
                        setDuration(1000);
                AnimatorSet alphaSeq = new AnimatorSet();
                alphaSeq.play(alphaAnim);

                animation = new AnimatorSet();
                ((AnimatorSet) animation).playTogether(yAnim, xAnim);
                animation.addListener(this);
            }
        }

# 整体逻辑 #
上面的两段代码为 AnimatorEvents 的核心代码，接下来只需要在点击对应的按钮时调用动画开始、暂停等方法，并调整对应状态的透明度即可。