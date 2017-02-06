# 简介 #

本书第四章介绍了原型模式。

# 原型模式介绍 #
用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象。被复制的实例就是“原型”，这个原型是可定制的。

#使用场景#
1.类初始化需要消化非常多的资源，这个资源包括数据、硬件资源等，通过原型拷贝避免这些消耗。 

2.通过 new 产生的一个对象需要非常繁琐的数据准备或者权限，这时可以使用原型模式。 

3.一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用，即保护性拷贝。

# 简单实现 #

	public class WordDocument implements Cloneable {
		// 文本
		private String mText;
	
		// 图片名列表
		private ArrayList<String> mImages = new ArrayList<String>();
	
		public WordDocument() {
			System.out
					.println("-----------------WordDocument构造函数------------------");
		}
	
	
		public String getmText() {
			return mText;
		}
	
		public void setmText(String mText) {
			this.mText = mText;
		}
	
		public ArrayList<String> getmImages() {
			return mImages;
		}
	
		public void addImage(String img) {
			this.mImages.add(img);
		}
	
		/**
		 * 打印文档内容
		 */
		public void showDocument() {
			System.out.println("------------word Content Start--------------");
			System.out.println("Text:" + mText);
			System.out.println("Images List:");
			for (String imgName : mImages) {
				System.out.println("image name:" + imgName);
			}
			System.out.println("------------word Content End--------------");
		}
	
	}


	public class Client {
		public static void main(String[] args) {
			//1.构建文档对象
			WordDocument originDoc=new WordDocument();
			
			//2.编辑文档，添加图片内容
			originDoc.setmText("这是一篇文档");
			originDoc.addImage("图片 1");
			originDoc.addImage("图片 2");
			originDoc.addImage("图片 3");
			
			originDoc.showDocument();
			
			//以原始文档为原型，拷贝一份副本
			WordDocument doc2=originDoc.clone();
			doc2.showDocument();
			
			//修改文档副本
			doc2.setmText("修改");
			doc2.addImage("哈哈.jpg");
			doc2.showDocument();
			
			originDoc.showDocument();
		}
	}

运行结果如下：

![](http://i.imgur.com/N6wXcPC.png)

可以看到，originDoc 和 doc2 在 Text 修改之后输出是不一样的，但是在 doc2 中 doc2.addImage("哈哈.jpg") 之后，originDoc 也输出了 哈哈.jpg，说明这只是简单的浅拷贝，引用类型的的新对象 doc2 的 mImages 只是单纯地指向了 this.mImages 引用,并没有重新构造一个 mImages 对象。所以在原型模式里最需要注意的就是深拷贝和浅拷贝，使用深拷贝可以避免操作副本时影响原始对象的问题。我们在这里可以这么做：

	@Override
	protected WordDocument clone() {
		try {
			WordDocument doc = (WordDocument) super.clone();
			doc.mText = this.mText;
			;
			doc.mImages = (ArrayList<String>) this.mImages.clone();
			return doc;
		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
	}

我们将 doc.mImages 指向 this.mImages 的一份拷贝，而不是 this.mImages 本身，所以在 doc2 添加图片不会影响 originDoc。结果如下：

![](http://i.imgur.com/iLG6DKH.png)


# Android源码中的原型模式 #

        Uri uri = Uri.parse("smsto:110");
        Intent intent = new Intent(Intent.ACTION_SEND,uri);
        intent.putExtra("sms_body", "The SMS text");
        //克隆
        Intent intent2 = (Intent)intent.clone();
        startActivity(intent2);


# 总结 #

优点

1.原型模式是在内存中二进制流的拷贝，要比直接new一个对象性能好很多，特别是要在一个循环体内产生大量对象时，原型模式可能更好的体现其优点。

2.还有一个重要的用途就是保护性拷贝，也就是对某个对象对外可能是只读的，为了防止外部对这个只读对象的修改，通常可以通过返回一个对象拷贝的形式实现只读的限制。

缺点：

1.这既是它的优点也是缺点，直接在内存中拷贝，构造函数是不会执行的，在实际开发中应该注意这个潜在问题。优点是减少了约束，缺点也是减少了约束，需要大家在实际应用时考虑。

2.通过实现Cloneable接口的原型模式在调用clone函数构造实例时并不一定比通过new操作速度快，只有当通过new构造对象较为耗时或者说成本较高时，通过clone方法才能够获得效率上的提升。

# 参考资料 #
《Android 源码设计模式解析与实战 》