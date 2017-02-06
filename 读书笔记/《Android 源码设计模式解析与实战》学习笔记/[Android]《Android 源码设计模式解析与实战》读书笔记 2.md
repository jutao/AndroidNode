# 里氏替换原则 (LSP)#
定义：所有引用父类的地方，必须能使用子类的对象。简单地说就是将父类替换为他的子类是不会出现问题，反之，未必可以。那么里氏替换原则就是依赖于面向对象语言的继承与多态。核心原理是抽象。

这里列举一下继承的优缺点： 

优点： 

（1）代码重用，减少创建类的成本，每个子类都拥有父类的方法与属性。 

（2）子类与父类基本相似，但与父类又有所区别。 

（3）提高代码的可扩展性。 

缺点： 

（1）继承是侵入性的，只要继承就必须拥有父类所有的属性与方法。 

（2）可能造成子类代码冗余、灵活性降低,因为子类必须拥有父类的属性和方法。

开闭原则和里氏替换原则是生死相依的、不离不弃的。他们都强调了抽象这一重要的特性。

##示例代码 ##

	public class Window {
		public void show(View child){
			child.draw();
		}
	}

	public abstract class View {
		public abstract void draw();
		public void measure(int width,int height){
			//测量视图大小
		}
	}

	public class Button extends View {
		@Override
		public void draw() {
			// 绘制按钮
		}
	}

	public class TextView extends View {
		@Override
		public void draw() {
			// 绘制文本
		}
	}

上述示例中，Window依赖于 View，而 View 定义了一个视图抽象，measure 是各个子类共享的方法，子类通过覆写 View 的 draw 方法实现具有各自特色的功能，在这里这个功能就是绘制自身的内容。任何继承自 View 类的子类都可以设置给 show 方法，就是所说的里氏替换。通过里氏替换，就可以自定义各式各样、千变万化的 View，然后传递给 Window，Window 负责组织 View 并将 View 显示到屏幕上。

# 依赖倒置原则（DIP） #
定义：指代一种特定的解耦方式，使得高层次的模块不依赖于低层次的模块的实现细节的目的。他有一下几个关键点： 

（1）高层模块不依赖于低层模块，应该都依赖其抽象。 

（2）抽象不依赖细节。 

（3）细节应依赖抽象。

解释：在Java中，抽象就是指接口或者抽象类，两者都是不能直接被实例化的；细节就是实现类，实现接口或者继承抽象类而产生的就是细节，也就是可以加上一个关键字new产生的对象。高层模块就是调用端，底层模块就是具体实现类。依赖倒置原则在Java中的表现就是：模块间通过抽象发生，实现类之间不发生直接依赖关系，其依赖关系是通过接口或者抽象类产生的。如果类与类直接依赖细节，那么就会直接耦合，那么当修改时，就会同时修改依赖者代码，这样限制了可扩展性。

# 接口隔离原则（ISP） #
定义：类间的依赖关系应该建立在最小的接口上，将庞大、臃肿的接口拆分成更小的、更具体的接口。目的是系统的解耦，从而更容易重构、更改和重新部署。
示例：

	 finally {
	      if (fileOutputStream != null) {
	        try {
	          fileOutputStream.close();
	        } catch (IOException e) {
	          e.printStackTrace();
	        }
	      }
	    }
这段代码我们经常见到，他的可读性非常的差。我们知道 Java 中有一个 Closeable 接口标识了可关闭对象，它只有一个 close 方法，有100多个类实现了这个接口，这意味着在关闭这100多个类的对象时我们都要写出上面那一段难看的代码来关闭它，这你能忍？
书中给出的解决方案如下：
既然都实现了Closeable接口，那我们只需要建一个方法来统一关闭这些对象即可，以下为工具类代码：
	public class Closeutils {
	  private Closeutils() {
	  }
	
	  /**
	   * 关闭Closeable对象
	   */
	  public static void closeQuietly(Closeable closeable) {
	    if (null != closeable) {
	      try {
	        closeable.close();
	      } catch (IOException e) {
	        e.printStackTrace();
	      }
	    }
	  }
	}
此时关闭就只需要这样：

	finally {
      Closeutils.closeQuietly(fileOutputStream);
    }
代码非常简洁，而且只要继承了 Closeable 接口都可以用此方法关闭，我们只需要知道这个对象是可关闭的，其他的一概不关心，也就是接口隔离原则。

# 迪米特原则（LOD） #

定义：一个类应该对自己需要耦合或者调用的类知道的最少，类的内部如何实现与调用者或者依赖者没有关系，调用者或依赖者只需知道他需要的方法，其他可以一概不管。这样使得系统具有更低的耦合与更好的可扩展性。