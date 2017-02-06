# 简介 #

这周继续写《Android源码设计模式解析与实战》读书笔记。本书的第三章介绍了 Builder(建造者)模式的使用方式以及在 Android 源码中的应用。

# Builder 模式介绍 #
将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。对于复杂的对象，为了在构建过程中对外部隐藏实现细节，可以使用 Builder 模式将部件和组件过程分离，使得构建过程和部件都可以自由扩展，两者之间的耦合度也降到最低。

# 使用场景 #
1.相同的方法，不同的执行顺序，产生不同的事件结果时。 

2.多个部件或零件，都可以装配到一个对象中，但是产生的运行结果又不相同时。 

3.产品类非常复杂，或者产品类中的调用顺序不同产生了不同的作用，这个使用建造者模式非常适合。 

4.当初始化一个对象特别复杂时，如参数多，且很多参数有默认值。

# Builder 使用方法 #
我仿照 Android 源码中的 AlertDialog.Builder ，将书中的源码作了修改，更加符合日常使用习惯。代码如下：

	public class Computer {
		private ComputerConfig mConfig;
	
		private Computer() {
		}
	
		public static Computer getInstance() {
			return ComputerHolder.sInstance;
		}
	
		public void init(ComputerConfig config) {
			this.mConfig = config;
		}
	
		@Override
		public String toString() {
			return "主机:" + mConfig.mBoard + "  显示器:" + mConfig.mDisplay + "  操作系统:"
					+ mConfig.mOS;
		}
	
		private static class ComputerHolder {
			private final static Computer sInstance = new Computer();
		}
	}


Computer 类的属性全部放在 ComputerConfig 类中

	public class ComputerConfig {
		 String mBoard;
		 String mDisplay;
		 String mOS;
		 
		 private ComputerConfig(){}
		 
		 public static class Builder{
			 String mBoard;
			 String mDisplay;
			 String mOS;
			 
			public Builder setmBoard(String mBoard) {
				this.mBoard = mBoard;
				return this;
			}
			
			public Builder setmDisplay(String mDisplay) {
				this.mDisplay = mDisplay;
				return this;
			}
			
			public Builder setmOS(String mOS) {
				this.mOS = mOS;
				return this;
			}
			 
			 private void apply(ComputerConfig config){
				 config.mBoard=this.mBoard;
				 config.mDisplay=this.mDisplay;
				 config.mOS=this.mOS;
			 }
			 
			 public ComputerConfig create(){
				 ComputerConfig config=new ComputerConfig();
				 this.apply(config);
				 return config;
			 }
		 }
	}



	public class Demo {
		public static void main(String[] args) {
			ComputerConfig config = new ComputerConfig.Builder().setmBoard("机甲战神")
					.setmDisplay("清华同方").setmOS("windows 10").create();
			Computer computer = Computer.getInstance();
			computer.init(config);
			System.out.println(computer.toString());
		}
	}

运行结果：

![](http://i.imgur.com/NEb3D6i.png)

通过将 ComputerConfig 的构造函数私有化，用户只有通过 Builder 对象的 create 方法才能获取 ComputerConfig 实例并设置属性，这就是构建和表示相分离。Android 中最经典的 Builder 模式应用也就是 AlertDialog.Builder 了，我写的这个例子也是参照了源码中的写法，因此在本文中就不列举源码了。


# 总结 #
优点：

1.良好的封装性，使用建造者模式可以使客户端不必知道产品内部组成细节。 

2.建造者独立，容易扩展。

缺点：

1.会产生多余的Builder对象及Director对象，消耗内存。


# 参考资料 #

《Android 源码设计模式解析与实战 》