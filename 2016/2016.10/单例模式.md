# 简介 #
这周继续写《Android源码设计模式解析与实战》读书笔记。本书的第二章介绍了单例模式的各种实现方式，以及在 Android 源码中的应用。

# 单例模式介绍 #
确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例。它的作用是避免产生多个对象消耗过多的资源，或者某种类型的对象只应该有且只有一个。比如创建一个对象需要消耗的资源过多，如要访问 IO 和数据库等资源。

# 单例模式使用要点 #
单例模式 UML 类图如下：

![单例模式 UML 类图](http://i.imgur.com/BO1J9Ma.png)

实现单例模式主要有如下几个关键点：

1.构造函数不对外开放，一般为 Private；

2.通过一个静态方法或者枚举返回单例对象；

3.确保单例类的对象只有一个，尤其是在多线程环境下（难点）；

4.确保单例类对象在反序列化时不会重新构建对象。

# 单例模式用法 #
## 饿汉模式 ##
	public class Singleton {  
	     private static Singleton instance = new Singleton();  
	     private Singleton (){
	     }
	     public static Singleton getInstance() {  
	     return instance;  
	     }  
	 }  
饿汉模式在装载类时就创建对象实例，是典型的空间换时间。

## 懒汉模式 ##
	public class Singleton {
		private static Singleton instance;
		private Singleton(){};
		
		public static synchronized Singleton getInstance(){
			if(instance==null){
				instance=new Singleton();
			}
			return instance;
		}
	}
懒汉模式在每次获取实例时都会进行判断，是典型的时间换空间。 getInstance() 方法中添加了 synchronized关键字，也就是上面所说的在多线程情况下保证单例对象唯一性的手段。但是即使 instance 已经被初始化，每次调用 getInstance() 方法都会进行同步，浪费不必要的资源，这也就是懒汉模式的最大问题。因此这种模式一般不建议使用。

## 双重检查锁定（Double Check Lock）实现单例 ##
	public class DCLSingleton {
		// JDK1.5后的版本才可使用volatile关键字，保证sInstance对象每次都从主内存中读取
		private volatile static DCLSingleton sInstance = null;
	
		private DCLSingleton() {
		}
		
		public static DCLSingleton getInstance(){
			if(sInstance==null){
				synchronized(DCLSingleton.class){
					if(sInstance==null){
						sInstance=new DCLSingleton();
					}
				}
			}
			return sInstance;
		}
	}
这个写法的特别之处在于对 instance 进行了两次判空：第一层主要是为了避免不必要的同步，第二层则是为了在 null 的情况下创建实例。
我们会发现上面代码有一个volatile关键字，因为在这里会有DCL失效问题。

DCL 失效问题：假设线程 A 执行到sInstance=new DCLSingleton()语句，这看上去像是一句代码，实际上它并不是一个原子操作，这句代码最终会被编译为多条汇编指令，它大致做了三件事：

1.给 sInstance 的实例分配内存；

2.调用 DCLSingleton 的构造函数，初始化成员字段；

3.将 sInstance 对象指向分配的内存空间（此时 sInstance 就不是 null了）。

但是由于 Java 编译器允许处理器乱序执行。因此执行顺序可能是 1-2-3 也可能是 1-3-2，如果是后者，并且在 3 执行完毕、2 未执行之前被切换到 B 线程上，这时的 sInstance 因为已经在线程 A 内执行过了第三点，sInstance 已经是非空了，所以线程 B 直接取走 sInstance，再使用就会出错，这就是 DCL 失效问题。

JDK 1.5 之后的版本具体化了 volatile 关键字，用它可以保证 sInstance 对象每次都从主内存中读取，虽然会影响性能，这种方式第一次加载时会稍慢，在高并发环境会有缺陷，但是一般能够满足需求。

## 静态内部类单例模式 ##

	public class Singleton implements  Serializable{
		private Singleton(){}
		
		public static Singleton getInstance(){
			return SingletonHolder.sInstance;
		}
		/**
		 * 静态内部类
		 */
		private static class SingletonHolder{
			private static final Singleton sInstance=new Singleton();
		}
		
		/**
		 * 为了杜绝对象在反序列化时重新生成对象，则重写Serializable的私有方法
		 * @return
		 * @throws ObjectStreamException
		 */
		private Object readResolve() throws ObjectStreamException{
			return SingletonHolder.sInstance;
		}
		
	}

这种是推荐使用的单例模式实现方式。当第一次加载Singleton类时并不会初始化INSTANCE，只有在第一次调用getInstance方法时才会导致INSTANCE被初始化。这种方式不仅能够保证线程安全，也能保证单例对象的唯一性，同时也延长了单例的实例化。
上面的代码重写了 readResolve() 方法，这是因为通过序列化可以将一个单例的实例对象写到磁盘，然后读回来，从而获得一个实例。即使构造函数是私有的，反序列化时依然可以通过特殊的途径去创建类的一个新的实例，相当于调用该类的构造函数。反序列化操作提供了一个特别的钩子函数，类中具有一个私有的、被实例化的方法 readResolve()，这个方法可以让开发人员控制对象的反序列化。重写该方法返回 SingletonHolder.sInstance ，而不是默认的生成新的实例，从而保持单例。

## 枚举单例 ##
	public enum SingletonEnum {
		INSTANCE;
		public void doSomething(){
			System.out.println("do sth.");
		}
	}
这种方式是Effective Java作者Josh Bloch 提倡的方式，它不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象。

## 容器实现单例 ##
	public class SingletonManager {
		private static Map<String,Object> objMap=new HashMap<String,Object>();
		
		private SingletonManager(){};
		
		public static void registerService(String key,Object instance){
			if(!objMap.containsKey(key)){
				objMap.put(key, instance);
			}
		}
		
		public static Object getService(String key){
			return objMap.get(key);
		}
	}
将多种单例类型注入到一个统一的管理类中，在使用时根据key获取对象对应类型的对象。这种方式使得我们可以管理多种类型的单例，并且在使用时可以通过统一的接口进行获取操作，降低了用户的使用成本，也对用户隐藏了具体实现，降低了耦合度。

# 单例模式运用场景 #
> 1. Windows 的 Task Manager （任务管理器）就是很典型的单例模式（这个很熟悉吧），想想看，是不是呢，你能打开两个 windows task manager 吗？ 不信你自己试试看哦~ 
> 
> 2. windows的Recycle Bin（回收站）也是典型的单例应用。在整个系统运行过程中，回收站一直维护着仅有的一个实例。
>  
> 3. 网站的计数器，一般也是采用单例模式实现，否则难以同步。
>  
> 4. 应用程序的日志应用，一般都何用单例模式实现，这一般是由于共享的日志文件一直处于打开状态，因为只能有一个实例去操作，否则内容不好追加。
> 
> 5. Web 应用的配置对象的读取，一般也应用单例模式，这个是由于配置文件是共享的资源。
> 
> 6. 数据库连接池的设计一般也是采用单例模式，因为数据库连接是一种数据库资源。数据库软件系统中使用数据库连接池，主要是节省打开或者关闭数据库连接所引起的效率损耗，这种效率上的损耗还是非常昂贵的，因为何用单例模式来维护，就可以大大降低这种损耗。
> 
> 7. 多线程的线程池的设计一般也是采用单例模式，这是由于线程池要方便对池中的线程进行控制。
> 
> 8. 操作系统的文件系统，也是大的单例模式实现的具体例子，一个操作系统只能有一个文件系统。
> 
> 9. HttpApplication 也是单位例的典型应用。熟悉 ASP.Net(IIS) 的整个请求生命周期的人应该知道 HttpApplication 也是单例模式，所有的 HttpModule 都共享一个 HttpApplication 实例.
>  
> 总结以上，不难看出：
> 
> 　　单例模式应用的场景一般发现在以下条件下：
> 
> 　　（1）资源共享的情况下，避免由于资源操作时导致的性能或损耗等。如上述中的日志文件，应用配置。
> 
> 　　（2）控制资源的情况下，方便资源之间的互相通信。如线程池等。


## Android源码中的单例模式 ##
在 Android 系统中，我们经常会通过 Context 获取系统级别的服务，如 WindowsManagerService、ActivityManagerService 等，更常用的是一个 LayoutInflater 的类，这些服务会在合适的时候以单例的形式注册在系统中，在我们需要的时候就通过 Context 的 getSystemService(String name) 获取。

# 总结 #
优点：

1.由于单例模式在内存中只有一个实例，减少了内存开支，特别是一个对象需要频繁的创建、销毁时，而且创建或销毁时性能又无法优化，单例模式的优势就非常明显。 

2.单例模式可以避免对资源的多重占用，例如一个文件操作，由于只有一个实例存在内存中，避免对同一资源文件的同时操作。 

3.单例模式可以在系统设置全局的访问点，优化和共享资源访问，例如，可以设计一个单例类，负责所有数据表的映射处理。

缺点：

1.单例模式一般没有接口，扩展很困难，若要扩展，只能修改代码来实现。 

2.单例对象如果持有 Context，那么很容易引发内存泄露。此时需要注意传递给单例对象的 Context 最好是 Application Context。

# 参考资料 #
[设计模式之——单例模式(Singleton)的常见应用场景](http://blog.csdn.net/likika2012/article/details/11483167 "设计模式之——单例模式(Singleton)的常见应用场景")

《Android 源码设计模式解析与实战 》