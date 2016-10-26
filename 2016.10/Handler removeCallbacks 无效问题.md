#  问题起因	 #
> 点击按钮就调用 handler.post(runnable); 就能启动定时器,这里是每隔1s打印线程名字，从打印中我们可以知道,他并没有另开线程，而是运行在 UI 线程当中，当你要取消定时器的时候，只需要调用 handler.removeCallbacks(runnable) 就可以了。
> 上面中有一个问题，有时候你会发现removeCallbacks有时候会失效，不能从消息队列中移除，看下面的代码
> 
> 	public class TimerActivity extends Activity {
> 	  Handler handler = new Handler();
> 	  Runnable runnable = new Runnable() {
> 	
> 	    @Override
> 	    public void run() {
> 	      System.out.println("update...");
> 	      handler.postDelayed(runnable, 1000);
> 	    }
> 	  };
> 	
> 	  @Override
> 	  protected void onCreate(Bundle savedInstanceState) {
> 	    super.onCreate(savedInstanceState);
> 	    setContentView(R.layout.layout_roll_view);
> 	
> 	    Button mButtonStart = (Button) findViewById(R.id.button1);
> 	    Button mButtonStop = (Button) findViewById(R.id.button2);
> 	
> 	    mButtonStart.setOnClickListener(new OnClickListener() {
> 	
> 	      @Override
> 	      public void onClick(View v) {
> 	        handler.post(runnable);
> 	      }
> 	    });
> 	
> 	    mButtonStop.setOnClickListener(new OnClickListener() {
> 	
> 	      @Override
> 	      public void onClick(View v) {
> 	        handler.removeCallbacks(runnable);
> 	      }
> 	    });
> 	  }
> 	
> 	}

# Handler removeCallbacks 无效问题#
> 当Activity进入后台运行后再转入前台运行，removeCallbacks 无法将 updateThread 从 message queue 中移除。
> 
> 这是为什么呢？
> 
> 在 Activity 由前台转后台过程中，线程是一直在运行的，但是当 Activity 转入前台时会重新定义 Runnable runnable；也就是说此时从message queue 移除的 runnable 与原先加入 message queue中的 runnable 并非是同一个对象。如果把 runnable 定义为静态的则removeCallbacks 不会失效,对于静态变量在内存中只有一个拷贝（节省内存），JVM只为静态分配一次内存，在加载类的过程中完成静态变量的内存分配。

也就是说 removeCallbacks 有时会出现移除失效的问题主要原因出在 runnable 对象上。

但是尽管我用了 removeCallbacksAndMessages 方法，依然有时会出现失效的现象，输出内存地址比较过后，发现 handler 对象也会发生变化。

# 解决方案 #
在 Application中建立容器存储 handler 和 runnable 对象，关闭时使用容器来关闭。

	  public static Map<Integer,Handler> sHandlerMap=new HashMap<>();
  	  public static Map<Integer,Runnable> sRunnableMap=new HashMap<>();

	    public static void stop() {
    		for (int i=0;i<NewsApplication.sHandlerMap.size();i++){
      			if (NewsApplication.sHandlerMap.get(i)!=null){
        			//NewsApplication.sHandlerMap.get(i).removeCallbacksAndMessages(null);
        			NewsApplication.sHandlerMap.get(i).removeCallbacks(NewsApplication.sRunnableMap.get(i));
     	 		}
  		  }
 	 }


	  /**
	   * Handler 容易造成内存泄漏，需要在application中全局保存
	   */
 	  public void start() {
	
	    if(NewsApplication.sRunnableMap.get(mIndex)==null){
	      RollRunable mRollRunable=new RollRunable();
	      NewsApplication.sRunnableMap.put(mIndex,mRollRunable);
	    }
	    if(NewsApplication.sHandlerMap.get(mIndex)==null){
	      Handler handler=new Handler();
	      NewsApplication.sHandlerMap.put(mIndex,handler);
	    }
	
	
	    NewsApplication.sHandlerMap.get(mIndex).postDelayed(NewsApplication.sRunnableMap.get(mIndex),2000);
	  }

以上解决方案适用于任意 handler 的任务移除。


# 引用 #
部分内容转自博客 

[Android 定时器实现的几种方式和removeCallbacks失效问题详解](http://blog.csdn.net/xiaanming/article/details/9011193 )