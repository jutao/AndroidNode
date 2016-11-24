# WebView简介 #
为了方便开发者实现在app内展示网页并与网页交互的需求，Android SDK提供了WebView组件。
它继承自AbsoluteLayout，展示网页的同时，也可以在其中放入其他的子View。
现如今，Hybrid应用似乎占据的APP的主流类型，那么关于WebView的使用就变得越发的重要。
从Android 4.4（KitKat）开始，原本基于WebKit的WebView开始基于Chromium内核，这一改动大大提升了WebView组件的性能以及对HTML5,CSS3,JavaScript的支持。不过它的API却没有很大的改动，在兼容低版本的同时只引进了少部分新的API，并不需要你做很大的改动。
在WebView中，有几个地方是我们可以使用来定制我们的WebView各种行为的，分别是：WebSettings、JavaScriptInterface、WebViewClient以及WebChromeClient。

# WebView基本使用 #
首先新建一个工程，在layout文件里放入一个WebView控件（当然也可以通过Java代码动态放入，这里不演示了）

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent">
	
	    <WebView
	        android:id="@+id/web_view"
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"/>
	</LinearLayout>

然后在Activity的onCreate方法里写入如下代码：

	String url = "https://www.baidu.com";
	WebView webView = (WebView) findViewById(R.id.web_view);
	webView.loadUrl(url);

接着在AndroidManifest声明访问网络的权限：

	<uses-permission android:name="android.permission.INTERNET"/>

就，完事了~

这时运行app，它已经可以访问指定地址的网页了。

上面提到了WebView继承自AbsoluteLayout，可以在其中放入一些子View，那也顺手来一下。

Layout文件改为：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	  android:layout_width="match_parent"
	  android:layout_height="match_parent">
	
	  <WebView
	      android:id="@+id/web_view"
	      android:layout_width="match_parent"
	      android:layout_height="match_parent">
	
	      <Button
	          android:id="@+id/button"
	          android:layout_width="wrap_content"
	          android:layout_height="wrap_content"
	          android:layout_x="170dp"
	          android:layout_y="400dp"
	          android:background="@color/colorAccent"
	          android:text="@string/app_name" />
	  </WebView>
	</LinearLayout>

Activity里加上：

    mButton = (Button) findViewById(R.id.button);
    mButton.setOnClickListener(this);

		@Override public void onClick(View v) {
	    switch (v.getId()) {
	      case R.id.button:
	        Toast.makeText(getApplicationContext(), "系好安全带!", Toast.LENGTH_SHORT).show();
	        break;
	    }
	  }

这时，运行app，里面就会多出一个Button~ 但如果你真的运行的话，你就会发现，app会自动跳到浏览器并打开指定的网页，而并非在app内展示网页，那这就与我们的初衷背道而驰了，那么要如何实现网页在App内打开呢?这就引出了下面的章节会提到的东西：WebViewClient。我先将代码贴出，具体实现原理留到下节说明。

最终XML布局就如上面那样，Java代码（最终）如下：


	public class MainActivity extends AppCompatActivity {
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	
	        String url = "https://www.google.com";
	        WebView webView = (WebView) findViewById(R.id.web_view);
	        webView.loadUrl(url);
	
	        webView.setWebViewClient(new WebViewClient() {
	            @Override
	            public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
	                view.loadUrl(request.toString());
	                return true;
	            }
	        });
	
	        Button button = (Button) findViewById(R.id.button);
	        button.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View view) {
	                Toast.makeText(getApplicationContext(), "系好安全带!", Toast.LENGTH_SHORT).show();
	            }
	        });
	    }
	}

## WebView常见方法介绍 ##
接下来再介绍一些WebView的常用方法，具体演示会在后面章节的代码里统一展示。

String getUrl()：获取当前页面的URL。

reload()：重新reload当前的URL，即刷新。

boolean canGoBack()：用来确认WebView里是否还有可回退的历史记录。通常我们会在WebView里重写返回键的点击事件，通过该方法判断WebView里是否还有历史记录，若有则返回上一页。

boolean canGoForward()：用来确认WebView是否还有可向前的历史记录。

boolean canGoBackOrForward(int steps)：以当前的页面为起始点，用来确认WebView的历史记录是否足以后退或前进给定的步数，正数为前进，负数为后退。

goBack()：在WebView历史记录后退到上一项。

goForward()：在WebView历史记录里前进到下一项。

goBackOrForward(int steps)：以当前页面为起始点，前进或后退历史记录中指定的步数，正数为前进，负数为后退。

clearCache(boolean includeDiskFiles)：清空网页访问留下的缓存数据。需要注意的时，由于缓存是全局的，所以只要是WebView用到的缓存都会被清空，即便其他地方也会使用到。该方法接受一个参数，从命名即可看出作用。若设为false，则只清空内存里的资源缓存，而不清空磁盘里的。

clearHistory()：清除当前webview访问的历史记录。

clearFormData()：清除自动完成填充的表单数据。需要注意的是，该方法仅仅清除当前表单域自动完成填充的表单数据，并不会清除WebView存储到本地的数据。

onPause()：当页面被失去焦点被切换到后台不可见状态，需要执行onPause操作，该操作会通知内核安全地暂停所有动作，比如动画的执行或定位的获取等。需要注意的是该方法并不会暂停JavaScript的执行，若要暂停JavaScript的执行请使用接下来的这个方法。

onResume()：在先前调用onPause()后，我们可以调用该方法来恢复WebView的运行。

pauseTimers()：该方法面向全局整个应用程序的webview，它会暂停所有webview的layout，parsing，JavaScript Timer。当程序进入后台时，该方法的调用可以降低CPU功耗。

resumeTimers()：恢复pauseTimers时的所有操作。

destroy()：销毁WebView。需要注意的是：这个方法的调用应在WebView从父容器中被remove掉之后。我们可以手动地调用

用例如下：

	rootLayout.removeView(webView);
	webView.destroy();

getScrollY()：该方法返回的当前可见区域的顶端距整个页面顶端的距离，也就是当前内容滚动的距离。

getHeight()：方法都返回当前WebView这个容器的高度。其实以上两个方法都属于View。

getContentHeight()：该方法返回整个HTML页面的高度，但该高度值并不等同于当前整个页面的高度，因为WebView有缩放功能， 所以当前整个页面的高度实际上应该是原始HTML的高度再乘上缩放比例。因此，准确的判断方法应该是：

	if (webView.getContentHeight() * webView.getScale() == (webView.getHeight() + webView.getScrollY())) {
	    //已经处于底端
	}
	
	if(webView.getScrollY() == 0){
	    //处于顶端
	}

pageUp(boolean top)：将WebView展示的页面滑动至顶部。

pageDown(boolean bottom)：将WebView展示的页面滑动至底部。

# WebSettings #
WebSettings是用来管理WebView配置的类。当WebView第一次创建时，内部会包含一个默认配置的集合。若我们想更改这些配置，便可以通过WebSettings里的方法来进行设置。

WebSettings对象可以通过WebView.getSettings()获得，它的生命周期是与它的WebView本身息息相关的，如果WebView被销毁了，那么任何由WebSettings调用的方法也同样不能使用。

## 获取WebSettings对象 ##
	WebSettings webSettings = webView.getSettings();

## WebSettings常用方法 ##
（几乎所有的set方法都有相应的get方法，这里就只介绍set了。另，所有未写方法返回值类型的皆为空类型）
setJavaScriptEnabled(boolean flag)：设置WebView是否可以运行JavaScript。

setJavaScriptCanOpenWindowsAutomatically(boolean flag)：设置WebView是否可以由JavaScript自动打开窗口，默认为false，通常与JavaScript的window.open()配合使用。

setAllowFileAccess(boolean allow)：启用或禁用WebView访问文件数据。

setBlockNetworkImage(boolean flag)：禁止或允许WebView从网络上加载图片。需要注意的是，如果设置是从禁止到允许的转变的话，图片数据并不会在设置改变后立刻去获取，而是在WebView调用reload()的时候才会生效。
这个时候，需要确保这个app拥有访问Internet的权限，否则会抛出安全异常。
通常没有禁止图片加载的需求的时候，完全不用管这个方法，因为当我们的app拥有访问Internet的权限时，这个flag的默认值就是false。

setSupportZoom(boolean support)：设置是否支持缩放。

setBuiltInZoomControls(boolean enabled)：显示或不显示缩放按钮（wap网页不支持）。

setSupportMultipleWindows(boolean support)：设置WebView是否支持多窗口。

setLayoutAlgorithm(WebSettings.LayoutAlgorithm l)：指定WebView的页面布局显示形式，调用该方法会引起页面重绘。默认值为LayoutAlgorithm#NARROW_COLUMNS。

setNeedInitialFocus(boolean flag)：通知WebView是否需要设置一个节点获取焦点当WebView#requestFocus(int,android.graphics.Rect)被调用时，默认为true。

setAppCacheEnabled(boolean flag)：启用或禁用应用缓存。

setAppCachePath(String appCachePath)：设置应用缓存路径，这个路径必须是可以让app写入文件的。该方法应该只被调用一次，重复调用会被无视~

setCacheMode(int mode)：用来设置WebView的缓存模式。当我们加载页面或从上一个页面返回的时候，会按照设置的缓存模式去检查并使用（或不使用）缓存。

缓存模式有四种：

1.	LOAD_DEFAULT：默认的缓存使用模式。在进行页面前进或后退的操作时，如果缓存可用并未过期就优先加载缓存，否则从网络上加载数据。这样可以减少页面的网络请求次数。

2.	LOAD_CACHE_ELSE_NETWORK：只要缓存可用就加载缓存，哪怕它们已经过期失效。如果缓存不可用就从网络上加载数据。

3.	LOAD_NO_CACHE：不加载缓存，只从网络加载数据。

4.	LOAD_CACHE_ONLY：不从网络加载数据，只从缓存加载数据。

通常我们可以根据网络情况将这几种模式结合使用，比如有网的时候使用LOAD_DEFAULT，离线时使用LOAD_CACHE_ONLY、LOAD_CACHE_ELSE_NETWORK，让用户不至于在离线时啥都看不到。

setDatabaseEnabled(boolean flag)：启用或禁用数据库缓存。

setDomStorageEnabled(boolean flag)：启用或禁用DOM缓存。

setUserAgentString(String ua)：设置WebView的UserAgent值。

setDefaultEncodingName(String encoding)：设置编码格式，通常都设为“UTF-8”。

setStandardFontFamily(String font)：设置标准的字体族，默认“sans-serif”。

setCursiveFontFamily：设置草书字体族，默认“cursive”。

setFantasyFontFamily：设置CursiveFont字体族，默认“cursive”。

setFixedFontFamily：设置混合字体族，默认“monospace”。

setSansSerifFontFamily：设置梵文字体族，默认“sans-serif”。

setSerifFontFamily：设置衬线字体族，默认“sans-serif”

setDefaultFixedFontSize(int size)：设置默认填充字体大小，默认16，取值区间为[1-72]，超过范围，使用其上限值。

setDefaultFontSize(int size)：设置默认字体大小，默认16，取值区间[1-72]，超过范围，使用其上限值。

setMinimumFontSize：设置最小字体，默认8. 取值区间[1-72]，超过范围，使用其上限值。

setMinimumLogicalFontSize：设置最小逻辑字体，默认8. 取值区间[1-72]，超过范围，使用其上限值。

以上就是一些WebSettings的常用方法，具体的使用以及一些缓存的问题会在接下来的代码以及文章中有更加直观的说明。

# WebViewClient #

从名字上不难理解，这个类就像WebView的委托人一样，是帮助WebView处理各种通知和请求事件的，我们可以称他为WebView的“内政大臣”。

* onLoadResource(WebView view, String url)：该方法在加载页面资源时会回调，每一个资源（比如图片）的加载都会调用一次。

* onPageStarted(WebView view, String url, Bitmap favicon)：该方法在WebView开始加载页面且仅在Main frame loading（即整页加载）时回调，一次Main frame的加载只会回调该方法一次。我们可以在这个方法里设定开启一个加载的动画，告诉用户程序在等待网络的响应。

* onPageFinished(WebView view, String url)：该方法只在WebView完成一个页面加载时调用一次（同样也只在Main frame loading时调用），我们可以可以在此时关闭加载动画，进行其他操作。

* onReceivedError(WebView view, WebResourceRequest request, WebResourceError error)：该方法在web页面加载错误时回调，这些错误通常都是由无法与服务器正常连接引起的，最常见的就是网络问题。 这个方法有两个地方需要注意：

	1. 这个方法只在与服务器无法正常连接时调用，类似于服务器返回错误码的那种错误（即HTTP ERROR），该方法是不会回调的，因为你已经和服务器正常连接上了（全怪官方文档(︶^︶)）；
	
	2.	这个方法是新版本的onReceivedError()方法，从API23开始引进，与旧方法onReceivedError(WebView view,int errorCode,String description,String failingUrl)不同的是，新方法在页面局部加载发生错误时也会被调用（比如页面里两个子Tab或者一张图片）。这就意味着该方法的调用频率可能会更加频繁，所以我们应该在该方法里执行尽量少的操作。

* onReceivedHttpError(WebView view, WebResourceRequest request, WebResourceResponse errorResponse)：上一个方法提到onReceivedError并不会在服务器返回错误码时被回调，那么当我们需要捕捉HTTP ERROR并进行相应操作时应该怎么办呢？API23便引入了该方法。当服务器返回一个HTTP ERROR并且它的status code>=400时，该方法便会回调。这个方法的作用域并不局限于Main Frame，任何资源的加载引发HTTP ERROR都会引起该方法的回调，所以我们也应该在该方法里执行尽量少的操作，只进行非常必要的错误处理等。

* onReceivedSslError(WebView view, SslErrorHandler handler, SslError error)：当WebView加载某个资源引发SSL错误时会回调该方法，这时WebView要么执行handler.cancel()取消加载，要么执行handler.proceed()方法继续加载（默认为cancel）。需要注意的是，这个决定可能会被保留并在将来再次遇到SSL错误时执行同样的操作。

* WebResourceResponse shouldInterceptRequest(WebView view, WebResourceRequest request)：当WebView需要请求某个数据时，这个方法可以拦截该请求来告知app并且允许app本身返回一个数据来替代我们原本要加载的数据。

比如你对web的某个js做了本地缓存，希望在加载该js时不再去请求服务器而是可以直接读取本地缓存的js，这个方法就可以帮助你完成这个需求。你可以写一些逻辑检测这个request，并返回相应的数据，你返回的数据就会被WebView使用，如果你返回null，WebView会继续向服务器请求。

* boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request)：哈~ 终于到了这个方法，在最开始的基础演示时我们用到了这个方法。从实践中我们知道，当我们没有给WebView提供WebViewClient时，WebView如果要加载一个url会向ActivityManager寻求一个适合的处理者来加载该url（比如系统自带的浏览器），这通常是我们不想看到的。于是我们需要给WebView提供一个WebViewClient，并重写该方法返回true来告知WebView url的加载就在app中进行。这时便可以实现在app内访问网页。

* onScaleChanged(WebView view, float oldScale, float newScale)：当WebView得页面Scale值发生改变时回调。

* boolean shouldOverrideKeyEvent(WebView view, KeyEvent event)：默认值为false，重写此方法并return true可以让我们在WebView内处理按键事件。

# WebChromeClient #
如果说WebViewClient是帮助WebView处理各种通知、请求事件的“内政大臣”的话，那么WebChromeClient就是辅助WebView处理Javascript的对话框，网站图标，网站title，加载进度等偏外部事件的“外交大臣”。

* onProgressChanged(WebView view, int newProgress)：当页面加载的进度发生改变时回调，用来告知主程序当前页面的加载进度。

* onReceivedIcon(WebView view, Bitmap icon)：用来接收web页面的icon，我们可以在这里将该页面的icon设置到Toolbar。

* onReceivedTitle(WebView view, String title)：用来接收web页面的title，我们可以在这里将页面的title设置到Toolbar。

以下两个方法是为了支持web页面进入全屏模式而存在的（比如播放视频），如果不实现这两个方法，该web上的内容便不能进入全屏模式。

* onShowCustomView(View view, WebChromeClient.CustomViewCallback callback)：该方法在当前页面进入全屏模式时回调，主程序必须提供一个包含当前web内容（视频 or Something）的自定义的View。

* onHideCustomView()：该方法在当前页面退出全屏模式时回调，主程序应在这时隐藏之前show出来的View。

* Bitmap getDefaultVideoPoster()：当我们的Web页面包含视频时，我们可以在HTML里为它设置一个预览图，WebView会在绘制页面时根据它的宽高为它布局。而当我们处于弱网状态下时，我们没有比较快的获取该图片，那WebView绘制页面时的gitWidth()方法就会报出空指针异常~ 于是app就crash了。。

<b>这时我们就需要重写该方法，在我们尚未获取web页面上的video预览图时，给予它一个本地的图片，避免空指针的发生。</b>

* View getVideoLoadingProgressView()：重写该方法可以在视频loading时给予一个自定义的View，可以是加载圆环 or something。

* boolean onJsAlert(WebView view, String url, String message, JsResult result)：处理Javascript中的Alert对话框。

* boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result)：处理Javascript中的Prompt对话框。

* boolean onJsConfirm(WebView view, String url, String message, JsResult result)：处理Javascript中的Confirm对话框

* boolean onShowFileChooser(WebView webView, ValueCallback filePathCallback, WebChromeClient.FileChooserParams fileChooserParams)：该方法在用户进行了web上某个需要上传文件的操作时回调。我们应该在这里打开一个文件选择器，如果要取消这个请求我们可以调用filePathCallback.onReceiveValue(null)并返回true。

* onPermissionRequest(PermissionRequest request)：该方法在web页面请求某个尚未被允许或拒绝的权限时回调，主程序在此时调用grant(String [])或deny()方法。如果该方法没有被重写，则默认拒绝web页面请求的权限。

* onPermissionRequestCanceled(PermissionRequest request)：该方法在web权限申请权限被取消时回调，这时应该隐藏任何与之相关的UI界面。

# Js与WebView交互 #

既然嗨鸟应用大行其道，那么毫无疑问Android与JavaScript的交互我们也必须了解清楚，下面来介绍一下JavaScript与Android是如何互相调用的。

利用WebView调用网页上的JavaScript代码

在WebView中调用Js的基本格式为webView.loadUrl("javascript:methodName(parameterValues)");

现有以下这段JavaScript代码

	  function readyToGo() {
	      alert("Hello")
	  }
	
	  function alertMessage(message) {
	      alert(message)
	  }
	
	  function getYourCar(){
	      return "Car";
	  }


1. WebView调用JavaScript无参无返回值函数

		String call = "javascript:readyToGo()";
		webView.loadUrl(call);

2. WebView调用JavScript有参无返回值函数
	
		String call = "javascript:alertMessage(\"" + "content" + "\")";
		webView.loadUrl(call);

3. WebView调用JavaScript有参数有返回值的函数

		  @TargetApi(Build.VERSION_CODES.KITKAT) private void evaluateJavaScript(WebView webView) {
		    webView.evaluateJavascript("getYourCar()", new ValueCallback<String>() {
		      @Override public void onReceiveValue(String s) {
		        Log.d("findCar", s);
		      }
		    });
		  }

## JavaScript通过WebView调用Java代码 ##

从API19开始，Android提供了 @JavascriptInterface 对象注解的方式来建立起 Javascript 对象和 Android 原生对象的绑定，提供给JavScript调用的函数必须带有 @JavascriptInterface。

## 演示一 JavaScript 调用 Android Toast 方法 ##

1. 编写Java原生方法并用使用 @JavascriptInterface 注解

		@JavascriptInterface
		public void show(String s){
		    Toast.makeText(getApplication(), s, Toast.LENGTH_SHORT).show();
		}

2. 注册JavaScriptInterface

		webView.addJavascriptInterface(this, "android");

	addJavascriptInterface 的作用是把 this 所代表的类映射为 JavaScript 中的 android 对象。

3. 编写JavaScript代码

		function toastClick(){
		    window.android.show("JavaScript called~!");
		}

## 演示二 JavaScript调用有返回值的Java方法 ##

1. 定义一个带返回值的Java方法，并使用 @JavaInterface：

		  @JavaInterface
	 	  public String getMessage(){
	    	return "Hello,boy~";
	  	  }
2.	添加JavaScript的映射

		webView.addJavaScriptInterface(this,"Android");

3.	通过JavaScript调用Java方法

		function showHello(){
	    	var str=window.Android.getMessage();
	    	console.log(str);
		}

以上就是Js与WebView交互的一些介绍，希望能对你有帮助。

# WebView加载优化 #
当WebView的使用频率变得频繁的时候，对于其各方面的优化就变得逐渐重要了起来。可以知道的是，我们每加载一个 H5页面，都会有很多的请求。除了HTML主URL自身的请求外，HTML外部引用的 JS、CSS、字体文件、图片都是一个个独立的HTTP 请求，虽然请求是并发的，但当网页整体数量达到一定程度的时候，再加上浏览器解析、渲染的时间，Web整体的加载时间变得很长。同时请求文件越多，消耗的流量也会越多。那么对于加载的优化就变得非常重要，这方面的经验我也没有什么别的，大概三个方面：

<b>一个，就是资源本地化的问题</b>

首先可以明确的是，以目前的网络条件，通过网络去服务器获取资源的速度是远远比不上从本地读取的。**谈论各种优化策略其实恰恰忽略了“需要加载”才是阻挡速度提升的最大绊脚石**。所以我们的思路一，就是将一些较重的资源比如js、css、图片甚至HTML本身进行本地化处理，在每次加载到这些资源的时候，从本地读取进行加载，可以简单记忆为“存·取·更”。

具体实现思路为：

1. “**存**”——将上述重量级资源打包进apk文件，每次加载相应文件时时从本地取即可。也可不打包，在第一次加载时以及接下来的若干间隔时间里动态下载存储，将所有的资源文件都存在Android的asset目录下；

2. “**取**”——重写WebViewClient的WebResourceResponse shouldInterceptRequest(WebView view, WebResourceRequest request)方法，通过一定的判别方法（例如正则表达式）拦截相应的请求，从本地读取相应资源并返回；

3. “**更**”——建立起Cache Control机制，定期或使用API通知的形式控制本地资源的更新，保证本地资源是最新和可用的。

**第二个，就是缓存的问题**

倘若你不采用或不完全采用第一条资源本地化的思路，那么你的WebView缓存是必须要开启的（虽然这一思路和第一条有重合的地方）。

	WebSettings settings = webView.getSettings();
	settings.setAppCacheEnabled(true);
	settings.setDatabaseEnabled(true);
	settings.setDomStorageEnabled(true);//开启DOM缓存
	settings.setCacheMode(WebSettings.LOAD_DEFAULT);

在网络正常时，采用默认缓存策略，在缓存可获取并且没有过期的情况下加载缓存，否则通过网络获取资源以减少页面的网络请求次数。

这里值得提起的是，我们经常在app里用WebView展示页面时，并不想让用户觉得他是在访问一个网页。因为倘若我们的app里网页非常多，而我们给用户的感觉又都像在访问网页的话，我们的app便失去了意义。（我的意思是为什么用户不直接使用浏览器呢？）

所以这时，离线缓存的问题就值得我们注意。我们需要让用户在没有网的时候，依然能够操作我们的app，而不是面对一个和浏览器里的网络错误一样的页面，哪怕他能进行的操作十分有限。

这里我的思路是，在开启缓存的前提下，WebView在加载页面时检测网络变化，倘若在加载页面时用户的网络突然断掉，我们应当更改WebView的缓存策略。

	ConnectivityManager connectivityManager = (ConnectivityManager)getSystemService(Context.CONNECTIVITY_SERVICE);
	NetworkInfo networkInfo = connectivityManager.getActiveNetworkInfo();
	if(networkInfo.isAvailable()) {
	    settings.setCacheMode(WebSettings.LOAD_DEFAULT);//网络正常时使用默认缓存策略
	} else {
	    settings.setCacheMode(WebSettings.LOAD_CACHE_ONLY);//网络不可用时只使用缓存
	}

既然有缓存，就要有缓存控制，与一相似的是我们也要建立缓存控制机制，定期或接受服务器通知来进行缓存的清空或更新。

**第三个，就是延迟加载和执行js**

在WebView中，onPageFinished()的回调意味着页面加载的完成。但该方法会在JavScript脚本执行完成后才会触发，倘若我们要加载的页面使用了JQuery，会在处理完DOM对象，执行完$(document).ready(function() {})后才会渲染并显示页面。这是不可接受的，所以我们需要对Js进行延迟加载，当然这部分是Web前端的工作。

**如果说还有什么**

那就是 JsBridge 一律不得滥用，这个对页面加载的完成速度是有很大影响的，倘若一个页面很多操作都通过 JSbridge 来控制，再怎么优化也无济于事（因为毕竟有那么多操作要实际执行）。同时要注意的是，不管你是否对资源进行缓存，都请将资源在服务器端进行压缩。因为无论是资源的获取和更新，都是要从服务器获取的，所以对于资源文件的压缩其实是最直接也最应该做的事情之一，但是一般服务器端都会做好，所以主要就是上面这三件事。


# 实战 #

介绍了这么多，希望能对你有点帮助。接下来时纯实战时间，我会将上面所介绍的很多知识点在接下来的代码里实际应用一遍，希望能够带给你更加直观的使用感受。

## xml代码 ##

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:app="http://schemas.android.com/apk/res-auto"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:orientation="vertical"
	    >
	  <android.support.design.widget.AppBarLayout
	      android:layout_width="match_parent"
	      android:layout_height="wrap_content"
	      >
	    <android.support.v7.widget.Toolbar
	        android:id="@+id/toolbar"
	        android:layout_width="match_parent"
	        android:layout_height="?attr/actionBarSize"
	        android:background="?attr/colorPrimary"
	        app:theme="@style/ThemeOverlay.AppCompat.Light"
	        >
	    </android.support.v7.widget.Toolbar>
	  </android.support.design.widget.AppBarLayout>
	
	  <LinearLayout
	      android:layout_width="match_parent"
	      android:layout_height="wrap_content"
	      android:orientation="horizontal"
	      >
	    <Button
	        android:id="@+id/btn_up"
	        android:text="向上"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        />
	    <Button
	        android:id="@+id/btn_down"
	        android:text="向下"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        />
	    <Button
	        android:id="@+id/btn_refresh"
	        android:text="刷新"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        />
	  </LinearLayout>
	
	  <FrameLayout
	      android:layout_width="match_parent"
	      android:layout_height="match_parent"
	      >
	    <WebView
	        android:id="@+id/web_view"
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        >
	    </WebView>
	    <ProgressBar
	        android:id="@+id/progress_bar"
	        android:layout_width="50dp"
	        android:layout_height="50dp"
	        android:layout_gravity="center"
	        android:visibility="gone"
	        />
	  </FrameLayout>
	
	</LinearLayout>

## Java代码 ##

	public class MainActivity extends AppCompatActivity implements View.OnClickListener {
	
	  private WebView mWebView;
	  private Toolbar mToolbar;
	  private ProgressBar mProgressBar;
	  private Button mBtnUp;
	  private Button mBtnDown;
	  private Button mBtnRefresh;
	
	  @Override protected void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
	    setContentView(R.layout.activity_main);
	    initView();
	
	    //初始化Toolbar
	    initAppBar();
	
	    //初始化WebView
	    initWebView();
	
	    //初始化WebSettings
	    initWebSettings();
	
	    //初始化WebViewClient
	    initWebViewClient();
	
	    //初始化WebChromeClient
	    initWebChromeClient();
	  }
	
	  private void initAppBar() {
	    mToolbar = (Toolbar) findViewById(R.id.toolbar);
	    mToolbar.setTitle("载入中...");
	    mToolbar.setTitleTextColor(getResources().getColor(R.color.colorWhite));
	    setSupportActionBar(mToolbar);
	    if (getSupportActionBar() != null) {
	      getSupportActionBar().setDisplayHomeAsUpEnabled(false);
	    }
	  }
	
	  private void initWebView() {
	    mWebView = (WebView) findViewById(R.id.web_view);
	    mProgressBar = (ProgressBar) findViewById(R.id.progress_bar);
	    String url = "https://www.baidu.com";
	    mWebView.loadUrl(url);
	  }
	
	  //果你的应用没有在WebView内直接使用JavaScript，不要调用setJavaScriptEnabled()
	  //要忽略警告及加上以下注解
	  @SuppressLint("SetJavaScriptEnabled") private void initWebSettings() {
	    WebSettings settings = mWebView.getSettings();
	    //支持获取手势焦点
	    mWebView.requestFocusFromTouch();
	    //支持JS
	    settings.setJavaScriptEnabled(true);
	    //支持插件
	    settings.setPluginState(WebSettings.PluginState.ON);
	    //设置适应屏幕
	    settings.setUseWideViewPort(true);
	    settings.setLoadWithOverviewMode(true);
	    //支持缩放
	    settings.setSupportZoom(false);
	    //隐藏原生缩放控件
	    settings.setDisplayZoomControls(false);
	    //支持内容重新布局
	    settings.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.SINGLE_COLUMN);
	    //设置WebView是否支持多窗口
	    settings.supportMultipleWindows();
	    settings.setSupportMultipleWindows(true);
	    //设置缓存模式
	    settings.setDomStorageEnabled(true);
	    settings.setDatabaseEnabled(true);
	    settings.setCacheMode(WebSettings.LOAD_DEFAULT);
	    settings.setAppCacheEnabled(true);
	    settings.setAppCachePath(mWebView.getContext().getCacheDir().getAbsolutePath());
	    //设置可访问文件
	    settings.setAllowContentAccess(true);
	    //当webview调用requestFocus时为webview设置节点
	    settings.setNeedInitialFocus(true);
	    //支持自动加载图片
	    if (Build.VERSION.SDK_INT >= 19) {
	      settings.setLoadsImagesAutomatically(true);
	    } else {
	      settings.setLoadsImagesAutomatically(false);
	    }
	    settings.setNeedInitialFocus(true);
	    //设置编码格式
	    settings.setDefaultTextEncodingName("UTF-8");
	  }
	
	  private void initWebViewClient() {
	    mWebView.setWebViewClient(new WebViewClient() {
	      //页面开始加载时
	      @Override public void onPageStarted(WebView view, String url, Bitmap favicon) {
	        super.onPageStarted(view, url, favicon);
	        mProgressBar.setVisibility(View.VISIBLE);
	      }
	
	      //页面完成加载时
	      @Override public void onPageFinished(WebView view, String url) {
	        super.onPageFinished(view, url);
	        mProgressBar.setVisibility(View.GONE);
	      }
	
	      //是否在WebView内加载新页面
	      @Override public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
	        view.loadUrl(request.toString());
	        return true;
	      }
	
	      //网络错误时回调方法
	      @Override public void onReceivedError(WebView view, WebResourceRequest request,
	          WebResourceError error) {
	        /**
	         * 在这里写网络错误时的逻辑,比如显示一个错误页面
	         *
	         * 这里我偷个懒不写了
	         */
	        view.loadUrl("http://www.jianshu.com/users/d251eadadd37/latest_articles");
	      }
	
	      //捕捉HTTP ERROR并进行相应操作
	      @Override public void onReceivedHttpError(WebView view, WebResourceRequest request,
	          WebResourceResponse errorResponse) {
	        view.loadUrl("http://www.jianshu.com/");
	      }
	    });
	  }
	
	  private void initWebChromeClient() {
	    mWebView.setWebChromeClient(new WebChromeClient() {
	      //默认的视频展示图
	      private Bitmap mDefaultVideoPoster;
	
	      @Override public void onReceivedTitle(WebView view, String title) {
	        super.onReceivedTitle(view, title);
	        setToolbarTitle(title);
	      }
	
	      @Override public Bitmap getDefaultVideoPoster() {
	        if (mDefaultVideoPoster == null) {
	          mDefaultVideoPoster = BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher);
	          return mDefaultVideoPoster;
	        }
	        return super.getDefaultVideoPoster();
	      }
	    });
	  }
	
	  /**
	   * 设置Toolbar标题
	   */
	  public void setToolbarTitle(final String title) {
	    Log.d("setToolbarTitle", " WebDetailActivity " + title);
	    if (mToolbar != null) {
	      mToolbar.post(new Runnable() {
	        @Override public void run() {
	          mToolbar.setTitle(TextUtils.isEmpty(title) ? getString(R.string.loading) : title);
	        }
	      });
	    }
	  }
	
	  private void initView() {
	    mBtnUp = (Button) findViewById(R.id.btn_up);
	    mBtnDown = (Button) findViewById(R.id.btn_down);
	    mBtnRefresh = (Button) findViewById(R.id.btn_refresh);
	
	    mBtnUp.setOnClickListener(this);
	    mBtnDown.setOnClickListener(this);
	    mBtnRefresh.setOnClickListener(this);
	  }
	
	  @Override public void onClick(View v) {
	    switch (v.getId()) {
	      case R.id.btn_up:
	        Toast.makeText(getApplicationContext(), "页面向上", Toast.LENGTH_SHORT).show();
	        mWebView.pageUp(true);
	        break;
	      case R.id.btn_down:
	        Toast.makeText(getApplicationContext(), "页面向下", Toast.LENGTH_SHORT).show();
	        mWebView.pageDown(true);
	        break;
	      case R.id.btn_refresh:
	        Toast.makeText(getApplicationContext(), "刷新", Toast.LENGTH_SHORT).show();
	        mWebView.reload();
	        break;
	    }
	  }
	
	  @Override public boolean onKeyDown(int keyCode, KeyEvent event) {
	    //如果按下的是回退键且历史记录里确实还有页面
	    if((keyCode==KeyEvent.KEYCODE_BACK)&&mWebView.canGoBack()){
	      mWebView.goBack();
	      return true;
	    }else {
	      Toast.makeText(getApplicationContext(), "考试结束,恭喜您考试合格!", Toast.LENGTH_LONG).show();
	    }
	    return super.onKeyDown(keyCode, event);
	  }
	}

# 感谢 #
[WebView·开车指南](https://jiandanxinli.github.io/2016-08-31.html?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

[史上最全webview详解](https://yq.aliyun.com/articles/32559 )

[H5 缓存机制浅析 移动端 Web 加载性能优化 ](http://bugly.qq.com/bbs/forum.php?mod=viewthread&tid=267)

[WebView加载速度优化](http://www.jianshu.com/p/427600ca2107)