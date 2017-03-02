# 介绍 #
AtomicInteger，一个提供原子操作的Integer的类。在Java语言中，++i和i++操作并不是线程安全的，在使用的时候，不可避免的会用到synchronized关键字。而AtomicInteger则通过一种线程安全的加减操作接口。

# 代码演示 #

	public class AtomicIntegerDemo {
		public static void main(String[] args) {
			AtomicInteger mInteger=new AtomicInteger(1);
			 //获取当前的值
			v(mInteger.get());
			
			 //取当前的值，并设置新的值
			v(mInteger.getAndSet(15));
			
			
			 //获取当前的值，并自增
			v(mInteger.getAndIncrement());
			
			 //获取当前的值，并自减
			 v(mInteger.getAndDecrement());
			 
			//获取当前的值，并加上预期的值
			 v(mInteger.getAndAdd(15));
			 
			 
			 v(mInteger.get());
		}
		
		 static void v(int i)
		 {
		  System.out.println("AtomicInteger : "+i);
		 }
		
	}

# 程序运行结果 #

![](http://i.imgur.com/CQHrTpS.png)