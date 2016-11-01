# 简介#
Handler 在 Android 开发中非常常见，它的常见用法相信只要稍微学过一些 Android 基础的朋友都已经烂熟于心，但是他背后的原理对于初学者来说比较复杂，这篇文章梳理了 Handler 的调用流程，通过源码观察 Hanler 背后的原理。

# 子线程创建 Handler #

    new Thread(new Runnable() {
      @Override public void run() {
        handler1 = new Handler();
      }
    }).start();


如果按照上面的代码来创建一个 Handler，运行程序，发现程序崩溃了，错误提示信息如下：

	java.lang.RuntimeException: Can't create handler inside thread that has not called Looper.prepare()
也就是说，不能在没有调用 Looper.prepare() 方法的线程中创建 Handler。那么我们就先调用一下 Looper.prepare()。
	
	new Thread(new Runnable() {
      @Override public void run() {
        Looper.prepare();
        handler1 = new Handler();
      }
    }).start();
这样程序果然不报错了，这是为什么呢，我们观察源码寻找答案。我们一开始出错是在创建 Handler 的时候，所以很有可能是 Handler 在构造函数里做了些什么，所以我选择首先观察 Handler 构造函数。

	 public Handler(Callback callback, boolean async) {
       	//省略部分源码
		
        mLooper = Looper.myLooper();
        if ( mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }

可以看到，源码中先是通过 Looper.myLooper() 方法获取一个 Looper 对象，然后判断这个对象是否为空，如果是空的就抛出一个异常，这个异常就是我们刚才看到的那个。为什么不调用 Looper.prepare() 方法 Looper.myLooper() 获取的对象就为空呢？先看 Looper.myLooper()方法：

	
	public static Looper myLooper() {
        return sThreadLocal.get();
    }
原来是把 Looper 存储到了一个线程存储器中，如果没有 Looper 对象，返回自然为空。这个 sThreadLocal 我们是操作不了的，所以想一想就可以知道，一定是 Looper.prepare() 中进行了存储。看 prepare() 方法源码：
	
	public static void prepare() {
        prepare(true);
    }	

再往下追：

	private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
	
可以看到，首先判断 sThreadLocal 中是否已经存在Looper，如果还没有则创建一个新的 Looper 设置进去。这样也就完全解释了为什么我们要先调用 Looper.prepare() 方法，才能创建 Handler 对象。同时也可以看出每个线程中最多只会有一个 Looper 对象。
那为什么我们在主线程可以直接创建 Handler 呢？一定是主线程已经替我们调用了 Looper.prepare() 方法。查看ActivityThread中的main()方法验证我们的猜想,源码如下：

	public static void main(String[] args) {
        SamplingProfilerIntegration.start();

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Set the reporter for event logging in libcore
        EventLogger.setReporter(new EventLoggingReporter());

        Security.addProvider(new AndroidKeyStoreProvider());

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();

		//省略
	}

可以看到上面的代码调用了一个  Looper.prepareMainLooper() 方法，我一开始以为只要我在子线程也调用 Looper.prepareMainLooper() 方法，就可以在子线程修改 UI 了，但是报错如下：

	 java.lang.IllegalStateException: The main Looper has already been prepared.
也就是说，我的想法还不算太荒谬， Looper.prepareMainLooper() 方法就是区分子线程和主线程的关键所在。我们进去看看它都做了些什么：

	 public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
原来如此，虽然主线程最终也调用了 prepare() 方法,但是给的值是 false，我们之前调用 prepare() 方法，默认值为 true，是不是感觉恍然大悟呢，不得不赞叹源码写的真是巧妙啊！！

到这里，我相信大家都和我一样，明白了为什么要先调用 Looper.prepare() 方法才可以创建 Handler 对象。我想过自己重写一个Handler 对象试试看能不能跳过这一步，但是你会发现，没有 Looper对象，你创建了也是白搭呀，至于为什么白搭，继续看下面的分析。
	
# Handler 消息发送 #
		
    new Thread(new Runnable() {
      @Override public void run() {
        Message message = new Message();
        message.what=1;
        Bundle bundle = new Bundle();
        bundle.putString("data", "data");
        message.setData(bundle);
        handler.sendMessage(message);
      }
    }).start();
这段代码相信大家都非常熟悉了，那它到底把 Message 发到哪里去了呢？Handler 给我们提供了很多方法来发送消息，有 post 的，也有 send 的。通过观察源码，你会发现，除了 enqueueMessage() 方法，其他所有发送消息的方法最后都会走到 sendMessageAtTime() 方法中，但是他们最终都会调用 MessageQueue 类中的 enqueueMessage() 方法，这肯定就是入队方法了：

	 boolean enqueueMessage(Message msg, long when) {
        if (msg.isInUse()) {
            throw new AndroidRuntimeException(msg + " This message is already in use.");
        }
        if (msg.target == null) {
            throw new AndroidRuntimeException("Message must have a target.");
        }

        synchronized (this) {
            if (mQuitting) {
                RuntimeException e = new RuntimeException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w("MessageQueue", e.getMessage(), e);
                return false;
            }

            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
	
通过源码可知，这个消息队列实际上是按照发送延时时间，也就是 when 来降序排序的，这样我们发送的消息就按照发送时间排好队了，但是他们排好队要去哪里呢，也就是出队操作在哪执行呢？我们再来分析 ActivityThread 中的 main 方法：

	public static void main(String[] args) {

      	//省略部分源码

        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        AsyncTask.init();

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
你会发现，Looper.prepareMainLooper() 或者  Looper.prepare() 方法总是和 Looper.loop() 方法对应，有你必有它，那么我可以合理的怀疑这个 Looper.loop() 方法很有可能就是执行出队操作的方法：

	public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            msg.target.dispatchMessage(msg);

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycle();
        }
    }
可以看到，这段代码从 13 行 开始进入了一个死循环，Message msg = queue.next() 就是我们要找的出队方法，而且还是一个阻塞方法，它的简单逻辑就是如果当前 MessageQueue 中存在 mMessages(即待处理消息)，就将这个消息出队，然后让下一条消息成为 mMessages，否则就进入一个阻塞状态，一直等到有新的消息入队。接下来比较重要的代码就是 msg.target.dispatchMessage(msg)，这个 target 其实就是 Handler 发送消息的 Handler 对象，观察 handler 调用入队方法的必经之路 enqueueMessage() 方法可知：

	 private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }

看上面代码的第二行。

所以我们可以知道，loop() 方法把最新出队的 message 又传给了 Handler 对象 的 dispatchMessage() 方法，所以我们肯定要观察 dispatchMessage() 方法了。

	 public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }

dispatchMessage() 方法的逻辑为：如果 msg.callback 不为空(callback 一般是通过 Handler 的 post 系列方法设置的，是一个 Runnable 对象)，则执行 message.callback.run() 方法。否则判断 mCallback 如果不为空，则调用 mCallback 的 handleMessage()方法，否则直接调用 Handler 的 handleMessage() 方法，并将消息对象作为参数传递过去。

# 总结 #
上面就是 Handler 的一个完整的从信息发送到执行的流程。流程图如下：
![](http://i.imgur.com/SGuBgaN.png)