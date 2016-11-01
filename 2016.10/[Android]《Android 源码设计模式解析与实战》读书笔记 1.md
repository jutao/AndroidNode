# 简介 #
这周入手了《Android源码设计模式解析与实战》，将花一段时间去阅读并做上读书笔记。本书的第一章介绍了面向对象的六大原则，这篇文章先介绍前两条：单一职责原则和开闭原则，并观察书中所举的例子，一个写的不错的图片加载器，来看看作者是怎么用代码诠释着两大原则的。

# 单一职责原则（SRP） #

> 单一职责所表达出的用意就是 "单一" 二字。如何划分一个类、一个函数的职责，每个人都有自己的看法，这需要根据个人经验、具体的业务逻辑而定。但是，它也有一些基本的指导原则，例如，两个完全不一样的功能就不应该放在一个类中。一个类中应该是一组相关性很高的函数、数据的封装。

试想一下，如果所有的功能写在一个类里，那么这个类会越来越大，越来越复杂，越不易修改维护。那么根据功能，各自独立拆分出来，岂不是逻辑会清晰些。比如作者给的例子是一个 ImageLoder 类，和一个 ImageCache 类。

	public class ImageCache{
		//只负责图片缓存逻辑
	}

	public class ImageLoader {
		//只负责图片的加载逻辑
	}

# 开闭原则 （OCP）#
> 软件中的对象（类、模块、函数等）应该对于扩展是开放的，但是对于修改是封闭的。当软件需要变化时，我们应该尽量通过扩展的方式实现变化，而不是通过修改原有的代码来实现。因为直接的修改，可能会影响已有的正常代码。不利于出现错误时排除问题。当然实际开发中，修改原有代码与扩展代码是同时存在的。但应尽量以扩展为主。

为了使程序更利于扩展，作者将之前的 ImageCache改造成了一个接口

	/**
	 * 图片缓存接口
	 */
	public interface ImageCache {
	  public Bitmap get(String url);
	
	  public void put(String url, Bitmap bmp);
	}
定义了这样一个接口后，无论是想用内存缓存、SD卡缓存还是双缓存都只需要实现该接口即可。我们看看这几个缓存的实现：

	/**
	 * 内存缓存MemoryCache类
	 */
	public class MemoryCache implements ImageCache {
	
	  private LruCache<String, Bitmap> mMemeryCache;
	
	  public MemoryCache() {
	    initLruCache();
	  }
	
	  //初始化LRU缓存
	  private void initLruCache() {
	    //计算可使用的最大内存
	    final int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);
	
	    //取出1/4的内存作为缓存
	    final int cacheSize = maxMemory / 4;
	
	    mMemeryCache = new LruCache<String, Bitmap>(cacheSize) {
	      @Override protected int sizeOf(String key, Bitmap value) {
	        return value.getRowBytes() * value.getHeight() / 1024;
	      }
	    };
	  }
	
	  @Override public Bitmap get(String url) {
	    return mMemeryCache.get(url);
	  }
	
	  @Override public void put(String url, Bitmap bmp) {
	    mMemeryCache.put(url, bmp);
	  }
	}


	/**
	 * 将图片缓存到SD卡中
	 */
	public class DiskCache implements ImageCache {
	
	  static String cacheDir = "sdcard/cache/";
	
	  //从缓存中获取图片
	  public Bitmap get(String url) {
	    return BitmapFactory.decodeFile(cacheDir + url);
	  }
	
	  //将图片存到内存中
	  public void put(String url, Bitmap bmp) {
	    FileOutputStream fileOutputStream = null;
	
	    try {
	      fileOutputStream = new FileOutputStream(cacheDir + url);
	
	      //30 是压缩率，表示压缩70%; 如果不压缩是100，表示压缩率为0
	      bmp.compress(Bitmap.CompressFormat.PNG, 100, fileOutputStream);
	    } catch (FileNotFoundException e) {
	      e.printStackTrace();
	    } finally {
	      if (fileOutputStream != null) {
	        try {
	          fileOutputStream.close();
	        } catch (IOException e) {
	          e.printStackTrace();
	        }
	      }
	    }
	  }
	}


	/**
	 * 双缓存
	 */
	public class DoubleCache implements ImageCache {
	  ImageCache mMemoryCache = new MemoryCache();
	  DiskCache mDiskCache = new DiskCache();
	
	  //先从内存缓存中获取图片，如果没有就从SD卡中获取
	  public Bitmap get(String url) {
	    Bitmap bitmap = mMemoryCache.get(url);
	    if (bitmap == null) {
	      bitmap = mDiskCache.get(url);
	    }
	    return bitmap;
	  }
	
	  public void put(String url, Bitmap bmp) {
	    mMemoryCache.put(url, bmp);
	    mDiskCache.put(url, bmp);
	  }
	}


在 ImageLoder 中只需要做如下一个小小的改动

	 public void setmImageCache(ImageCache cache) {
	    mImageCache = cache;
	  }

用户可以通过 setmImageCache(ImageCache cache) 方法注入不同的缓存实现，这样不仅能够使 ImageLoder 更简单、健壮，也使得 ImageLoder 的可扩展性、灵活性更高。三个缓存图片的具体实现完全不同，但他们都实现了 ImageCache 接口，就都可以通过 setmImageCache 方法注入到 ImageLoder 中，这样 ImageLoder 就实现了千变万化的缓存策略。