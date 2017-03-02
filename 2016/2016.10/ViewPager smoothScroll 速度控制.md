# 问题 #
大家都知道 ViewPager 可以通过 mViewPager.setCurrentItem(index, true) 来始切换动画时进行平缓的滑动，但是如果我们的需求是控制滑动时间的话，会发现 ViewPager 好像并没有提供这一个方法。我们可以查看ViewPager 的源码：

        public void setCurrentItem(int item, boolean smoothScroll) {
        mPopulatePending = false;
        setCurrentItemInternal(item, smoothScroll, false);
        }
发现 setCurrentItem 走的是一个 setCurrentItemInternal(item, smoothScroll, false) 方法
            
    void setCurrentItemInternal(int item, boolean smoothScroll, boolean always) {
        setCurrentItemInternal(item, smoothScroll, always, 0);
    }
而这个方法所调用的 void setCurrentItemInternal(int item, boolean smoothScroll, boolean always, int velocity)  方法貌似是可以指定速度的，但是先别高兴的太早，这个方法居然不是 public 的。。也就是说我们调用不了。但是这也难不倒我们，用反射就轻松解决了。 有兴趣你可以用反射调用这个方法试试看，反正我是试过了，速度好像并没有什么明显的变化，再仔细看了看源码，我们发现最终走的 smoothScrollTo 方法中有一句 

    duration = Math.min(duration, MAX_SETTLE_DURATION)

然而我们发现 ViewPager 中有一个常量     private static final int MAX_SETTLE_DURATION = 600; // ms

也就是说 duration 无论如何都不会超过 600ms 的，那我们偏要大于 600ms 呢？

毕竟道高一尺魔高一丈，我们发现有了 duration 后，会调用 mScroller.startScroll(sx, sy, dx, dy, duration) 方法，终究还是要走 mScroller 的，那我们用反射修改 mScroller不就行了吗？

# 效果前后展示 #
![](http://i.imgur.com/9rJD03b.gif)


![已经减速.gif](http://upload-images.jianshu.io/upload_images/3054656-8ae1bc06d367e234.gif?imageMogr2/auto-orient/strip)

# 解决方案 #
重写一个 Scroller 类：

    /**
     * 利用这个类来修正ViewPager的滑动速度
     * 我们重写 startScroll方法，忽略传过来的 duration 属性
     * 而是采用我们自己设置的时间
     */
    public class FixedSpeedScroller extends Scroller {
    
      public int mDuration=1500;
      public FixedSpeedScroller(Context context) {
        super(context);
      }
    
      public FixedSpeedScroller(Context context, Interpolator interpolator) {
        super(context, interpolator);
      }
    
      public FixedSpeedScroller(Context context, Interpolator interpolator, boolean flywheel) {
        super(context, interpolator, flywheel);
      }
    
      @Override public void startScroll(int startX, int startY, int dx, int dy) {
        startScroll(startX,startY,dx,dy,mDuration);
      }
    
      @Override public void startScroll(int startX, int startY, int dx, int dy, int duration) {
        //管你 ViewPager 传来什么时间，我完全不鸟你
        super.startScroll(startX, startY, dx, dy, mDuration);
      }
    
      public int getmDuration() {
        return mDuration;
      }
    
      public void setmDuration(int duration) {
        mDuration = duration;
      }
    }

利用反射把我们的 FixedSpeedScroller 类设置给 ViewPager

    /**
     * 通过反射来修改 ViewPager的mScroller属性
     */
    try {
      Class clazz=Class.forName("android.support.v4.view.ViewPager");
      Field f=clazz.getDeclaredField("mScroller");
      FixedSpeedScroller fixedSpeedScroller=new FixedSpeedScroller(this,new LinearOutSlowInInterpolator());
      fixedSpeedScroller.setmDuration(2000);
      f.setAccessible(true);
      f.set(mViewPager,fixedSpeedScroller);
    } catch (Exception e) {
      e.printStackTrace();
    }

这个时候再设置 mViewPager.setCurrentItem(index, true) 的时候应该就可以看到缓慢滑动的效果了。