# ApiDemos 中的 ListFlipper #
ListFlipper 是 ApiDemos 中的一个简单的动画特效示例，它直接继承了 Activity,使用起来还是比较简单的。他的作用是让一个界面旋转并且切换到另外一个界面上，是一个比较常见的切换动画。

# XML 布局 #
ApiDemos 中的 ListFlipper 只是一个简单的示例，它的 XML 布局也非常简单。就是一个线性布局， Button 用来控制触发动画效果，两个 ListView ,其中一个 ListView 的 visibility 属性为 gone.
    
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:orientation="vertical" >
	
	    <Button
	        android:id="@+id/button"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:text="Flip" />
	
	    <ListView
	        android:id="@+id/list_en"
	        android:layout_width="match_parent"
	        android:layout_height="0dip"
	        android:layout_weight="1.0" />
	
	    <ListView
	        android:id="@+id/list_fr"
	        android:layout_width="match_parent"
	        android:layout_height="0dip"
	        android:layout_weight="1.0"
	        android:visibility="gone" />
	</LinearLayout>
# ListFlipper 源码 #
源码原理很简单，我在代码里做了一些简单的注释。


	public class ListFlipper extends Activity {
	
		private static final String[] LIST_STRINGS_EN = new String[] { "One",
				"Two", "Three", "Four", "Five", "Six" };
		private static final String[] LIST_STRINGS_FR = new String[] { "Un",
				"Deux", "Trois", "Quatre", "Le Five", "Six" };
	
		ListView mEnglishList;
		ListView mFrenchList;
	
		/** Called when the activity is first created. */
		@Override
		public void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			// FrameLayout container = (LinearLayout) findViewById(R.id.container);
			mEnglishList = (ListView) findViewById(R.id.list_en);
			mFrenchList = (ListView) findViewById(R.id.list_fr);
	
			// Prepare the ListView
			final ArrayAdapter<String> adapterEn = new ArrayAdapter<String>(this,
					android.R.layout.simple_list_item_1, LIST_STRINGS_EN);
			// Prepare the ListView
			final ArrayAdapter<String> adapterFr = new ArrayAdapter<String>(this,
					android.R.layout.simple_list_item_1, LIST_STRINGS_FR);
	
			mEnglishList.setAdapter(adapterEn);
			mFrenchList.setAdapter(adapterFr);
			// mFrenchList.setRotationY(-90f);
	
			Button starter = (Button) findViewById(R.id.button);
			starter.setOnClickListener(new View.OnClickListener() {
				public void onClick(View v) {
					flipit();
				}
			});
		}
	
		private Interpolator accelerator = new AccelerateInterpolator();
		private Interpolator decelerator = new DecelerateInterpolator();
	
		private void flipit() {
			// 定义两个ListView，作用是充当容器
			final ListView visibleList;
			final ListView invisibleList;
			// 如果英语List为不可见的
			if (mEnglishList.getVisibility() == View.GONE) {
				// 可见List存储法语List
				visibleList = mFrenchList;
				// 不可见List存储英语List
				invisibleList = mEnglishList;
			} else {// 相反
				invisibleList = mFrenchList;
				visibleList = mEnglishList;
			}
			// 绕Y轴0°到90°旋转
			ObjectAnimator visToInvis = ObjectAnimator.ofFloat(visibleList,
					"rotationY", 0f, 90f);
			visToInvis.setDuration(500);
			// 在动画开始的地方速率改变比较慢，然后开始加速
			visToInvis.setInterpolator(accelerator);
			// 不可见List绕Y轴逆时针旋转90°
			final ObjectAnimator invisToVis = ObjectAnimator.ofFloat(invisibleList,
					"rotationY", -90f, 0f);
			invisToVis.setDuration(500);
			// 在动画开始的地方快然后慢
			invisToVis.setInterpolator(decelerator);
			// 给可见List设置一个动画监听
			visToInvis.addListener(new AnimatorListenerAdapter() {
				@TargetApi(Build.VERSION_CODES.HONEYCOMB)
				@Override
				public void onAnimationEnd(Animator anim) {
					// 当动画执行结束时，将可见List设置为不可见
					visibleList.setVisibility(View.GONE);
					// 启动不可见List动画
					invisToVis.start();
					// 将不可见List设置为可见的
					invisibleList.setVisibility(View.VISIBLE);
				}
			});
			// 启动可见List的动画
			visToInvis.start();
		}
	}

# Interpolator 调查 #

以前做动画没有接触过 Interpolator，去调查了一下，说明如下。

> Android:interpolator
> 
>  
> 
>    Interpolator 被用来修饰动画效果，定义动画的变化率，可以使存在的动画效果accelerated(加速)，decelerated(减速),repeated(重复),bounced(弹跳)等。
> 
>   android中的文档内容如下：
> 
>  
> 
> 
> 
>   AccelerateDecelerateInterpolator 在动画开始与结束的地方速率改变比较慢，在中间的时候加速
> 
>   AccelerateInterpolator  在动画开始的地方速率改变比较慢，然后开始加速
> 
>   AnticipateInterpolator 开始的时候向后然后向前甩
> 
>   AnticipateOvershootInterpolator 开始的时候向后然后向前甩一定值后返回最后的值
> 
>   BounceInterpolator   动画结束的时候弹起
> 
>   CycleInterpolator 动画循环播放特定的次数，速率改变沿着正弦曲线
> 
>   DecelerateInterpolator 在动画开始的地方快然后慢
> 
>   LinearInterpolator   以常量速率改变
> 
>   OvershootInterpolator    向前甩一定值后再回到原来位置
> 
> 
> 
> 如果android定义的interpolators不符合你的效果也可以自定义interpolators
> 
> 出处：http://blog.csdn.net/jason0539