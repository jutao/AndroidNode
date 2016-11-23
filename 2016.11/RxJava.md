# 使用前的准备 #
github地址：

[RxJava](https://github.com/ReactiveX/RxJava) 

[RxAndroid](https://github.com/ReactiveX/RxAndroid )

引入依赖：

  compile 'io.reactivex:rxjava:1.2.1'

  compile 'io.reactivex:rxandroid:1.2.1'

# 基本使用方式输出字符数组 #

      //创建观察者
      //1、正常模式：
      Subscriber light = new Subscriber() {
        @Override public void onCompleted() {
          System.out.println("结束观察...\n");
        }
    
        @Override public void onError(Throwable e) {
    
        }
    
        @Override public void onNext(Object o) {
          System.out.println("handle this---" + o);
        }
          };

      //偷懒模式(非正式写法)
      Action1 light=new Action1() {
        @Override public void call(Object o) {
          System.out.println("handle this---"+o);
        }
      };

      //创建被观察者
      //1、正常模式：
      Observable switcher = Observable.create(new Observable.OnSubscribe<String>() {
    
        @Override public void call(Subscriber<? super String> subscriber) {
          subscriber.onNext("on");
          subscriber.onNext("off");
          subscriber.onNext("on");
          subscriber.onNext("on");
          subscriber.onCompleted();
        }
      });
      //偷懒写法1:
      Observable switcher=Observable.just("on","off","on","on");
      //偷懒写法2：
      String[] kk={"on","off","on","on"};
      Observable switcher=Observable.from(kk);
    
      //将观察者和被观察者联系起来
      switcher.subscribe(light);

一开始感觉到 subscribe 这个方法有点怪：它看起来是observalbe 订阅了 observer / subscriber 而不是observer / subscriber 订阅了 observalbe，这看起来就像杂志订阅了读者一样颠倒了对象关系。这让人读起来有点别扭，不过如果把 API 设计成 observer.subscribe(observable) / subscriber.subscribe(observable) ，虽然更加符合思维逻辑，但对流式 API 的设计就造成影响了，比较起来明显是得不偿失的。

Observable.subscribe(Subscriber) 的内部实现是这样的（仅核心代码）：

    // 注意：这不是 subscribe() 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。
    // 如果需要看源码，可以去 RxJava 的 GitHub 仓库下载。
    public Subscription subscribe(Subscriber subscriber) {
        subscriber.onStart();
        onSubscribe.call(subscriber);
        return subscriber;
    }

可以看到，subscriber() 做了3件事：

* 调用 Subscriber.onStart() 。这个方法在前面已经介绍过，是一个可选的准备方法。
* 调用 Observable 中的 OnSubscribe.call(Subscriber) 。在这里，事件发送的逻辑开始运行。从这也可以看出，在 RxJava 中， Observable 并不是在创建的时候就立即开始发送事件，而是在它被订阅的时候，即当 subscribe() 方法执行的时候。
* 将传入的 Subscriber 作为 Subscription 返回。这是为了方便 unsubscribe().

整个过程中对象间的关系如下图：

![](http://i.imgur.com/J2SD7xO.jpg)

# 不完整定义的回调 #
除了 subscribe(Observer) 和 subscribe(Subscriber) ，subscribe() 还支持不完整定义的回调，RxJava 会自动根据定义创建出 Subscriber 。形式如下：

    Action1<String> onNextAction=new Action1<String>() {
      @Override public void call(String s) {
        System.out.println(s);
      }
    };
    Action1<Throwable> onErrorAction=new Action1<Throwable>() {
      @Override public void call(Throwable throwable) {
        System.out.println(throwable.getMessage());
      }
    };
    Action0 onCompletedAction=new Action0() {
      @Override public void call() {
        System.out.println("completed");
      }
    };

    Observable<String> observable=Observable.create(new Observable.OnSubscribe<String>(){
      @Override public void call(Subscriber<? super String> subscriber) {

      }
    });
    // 自动创建 Subscriber ，并使用 onNextAction 来定义 onNext()
    observable.subscribe(onNextAction);
    // 自动创建 Subscriber ，并使用 onNextAction 和 onErrorAction 来定义 onNext() 和 onError()
    observable.subscribe(onNextAction,onErrorAction);
    // 自动创建 Subscriber ，并使用 onNextAction、 onErrorAction 和 onCompletedAction 来定义 onNext()、 onError() 和 onCompleted()
    observable.subscribe(onNextAction,onErrorAction,onCompletedAction);

    onNextAction.call("call be called");
    onErrorAction.call(new RuntimeException("RunTime Now"));
    onCompletedAction.call();
输出结果如下：

![输出结果.jpg](http://upload-images.jianshu.io/upload_images/3054656-9496b69585399124.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

简单解释一下这段代码中出现的 Action1 和 Action0。 Action0 是 RxJava 的一个接口，它只有一个方法 call()，这个方法是无参无返回值的；由于 onCompleted() 方法也是无参无返回值的，因此 Action0 可以被当成一个包装对象，将 onCompleted() 的内容打包起来将自己作为一个参数传入 subscribe() 以实现不完整定义的回调。这样其实也可以看做将 onCompleted() 方法作为参数传进了 subscribe()，相当于其他某些语言中的『闭包』。 Action1 也是一个接口，它同样只有一个方法 call(T param)，这个方法也无返回值，但有一个参数；与 Action0 同理，由于 onNext(T obj) 和 onError(Throwable error) 也是单参数无返回值的，因此 Action1 可以将 onNext(obj) 和 onError(error) 打包起来传入 subscribe() 以实现不完整定义的回调。事实上，虽然 Action0 和 Action1 在 API 中使用最广泛，但 RxJava 是提供了多个 ActionX 形式的接口 (例如 Action2, Action3) 的，它们可以被用以包装不同的无返回值的方法。

# 场景示例 #

## 求数组之和 ##

将数组中的元素求和，我们用两种方法来实现

### 普通方法求数组之和###

    Integer[] addValue = { 1, 2, 3, 4 };

    Observable<Integer> swither = Observable.from(addValue);

    Subscriber<Integer> observer = new Subscriber<Integer>() {
      @Override public void onCompleted() {
        System.out.println("数组和为" + mSum);
      }

      @Override public void onError(Throwable e) {
        System.out.println(e.getMessage());
      }

      @Override public void onNext(Integer a) {
        mSum+= a.intValue();
      }
    };
    swither.subscribe(observer);

输出结果如下：

![输出结果.jpg](http://i.imgur.com/FwRM5G0.png)

### 不完整定义的回调求数组之和###

    Integer[] addValue = { 5, 6, 7, 8 };
    Action1<Integer> onNextAction=new Action1<Integer>() {
      @Override public void call(Integer a) {
          mSum+= a;
      }
    };
    Action1<Throwable> onError=new Action1<Throwable>() {
      @Override public void call(Throwable throwable) {
        System.out.println(throwable.getMessage());
      }
    };
    Action0 onCompleted=new Action0() {
      @Override public void call() {
        System.out.println("数组和为:"+mSum);
      }
    };

    Observable<Integer> observable=Observable.create(new Observable.OnSubscribe<Integer>() {
      @Override public void call(Subscriber<? super Integer> subscriber) {

      }
    });
    observable.subscribe(onNextAction,onError,onCompleted);
    onNextAction.call(addValue[0]);
    onNextAction.call(addValue[1]);
    onNextAction.call(addValue[2]);
    onNextAction.call(addValue[3]);
    onCompleted.call();

输出结果如下：

![输出结果.jpg](http://i.imgur.com/AAD9uIR.png)

## 由 id 取得图片并显示 ##
由指定的一个 drawable 文件 id drawableRes 取得图片，并显示在 ImageView 中，并在出现异常的时候打印 Toast 报错，代码如下：

       Observable.create(new Observable.OnSubscribe<Drawable>() {
      @Override public void call(Subscriber<? super Drawable> subscriber) {
        //在这里可以加载好drawable对象，通过onNext方法传给观察者
        Drawable drawable= ContextCompat.getDrawable(MainActivity.this,mDrawableRes);
        subscriber.onNext(drawable);
        subscriber.onCompleted();
      }
    }).subscribe(new Observer<Drawable>() {//这样可以直接调用subscribe方法，体现了流式编程之美

      @Override public void onCompleted() {

      }

      @Override public void onError(Throwable e) {
        Toast.makeText(MainActivity.this, "加载图片出错!!"+e.getMessage(), Toast.LENGTH_SHORT).show();
      }

      @Override public void onNext(Drawable drawable) {
          mImageview.setImageDrawable(drawable);
      }
    });
运行结果如下：

RxJava的基本流程就如上面的代码所示，创建出 Observable 和 Subscriber ，再用 subscribe() 将它们串起来，一次 RxJava 的基本使用就完成了。非常简单。

![高圆圆](http://i.imgur.com/qrD38Xf.png)

# 线程控制 Scheduler (一)#
在 RxJava 的默认规则中，事件的发出和消费都是在同一个线程的。也就是说，我们刚才实现出来的只是一个同步的观察者模式。观察者模式本身的目的就是『后台处理，前台回调』的异步机制，因此异步对于 RxJava 是至关重要的。而要实现异步，则需要用到 RxJava 的另一个概念： Scheduler（调度器）。
## Scheduler 的 API (一) ##
在RxJava 中，Scheduler ——调度器，相当于线程控制器，RxJava 通过它来指定每一段代码应该运行在什么样的线程。RxJava 已经内置了几个 Scheduler ，它们已经适合大多数的使用场景：
       
            //直接在当前线程运行，相当于不指定线程，这是默认的 Schedulers
            subscribeOn(Schedulers.immediate())
        
            //启用新线程，并在新线程中执行操作
            subscribeOn(Schedulers.newThread())
        
        /**
         *  I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。
         *  行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个
         *  无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread()
         *  更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。
         */
            subscribeOn(Schedulers.io())
        
        /**
         * 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O
         * 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，
         * 大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作
         * 的等待时间会浪费 CPU
         */
            subscribeOn(Schedulers.computation())
        
            //指定的操作将在 Android 主线程运行
            subscribeOn(AndroidSchedulers.mainThread())
        
            //当其他排队任务完成后，当前线程排队开始执行
            subscribeOn(Schedulers.trampoline());

有了这几个 Scheduler ，就可以使用 subscribeOn() 和 observeOn() 两个方法来对线程进行控制了。 * subscribeOn(): 指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。或者叫做事件产生的线程。 * observeOn(): 指定 Subscriber 所运行在的线程。或者叫做事件消费的线程。

	    Observable.just(1, 2, 3, 4).
        subscribeOn(Schedulers.io()).
        observeOn(AndroidSchedulers.mainThread()).
        subscribe(new Action1<Integer>() {
          @Override public void call(Integer number) {
            System.out.println("number:" + number);
          }
        });

上面这段代码中，由于 subscribeOn(Schedulers.io()) 的指定，被创建的事件的内容 1、2、3、4 将会在 IO 线程发出；而由于 observeOn(AndroidScheculers.mainThread()) 的指定，因此 subscriber 数字的打印将发生在主线程 。事实上，这种在 subscribe() 之前写上两句 subscribeOn(Scheduler.io()) 和 observeOn(AndroidSchedulers.mainThread()) 的使用方式非常常见，它适用于多数的 『后台线程取数据，主线程显示』的程序策略。

这个时候我们再去写刚才写过的加载图片案例可以这么写：

    Observable.create(new Observable.OnSubscribe<Drawable>() {

      @Override public void call(Subscriber<? super Drawable> subscriber) {
        Drawable drawable=ContextCompat.getDrawable(MainActivity.this,R.mipmap.gyy);
        subscriber.onNext(drawable);
        subscriber.onCompleted();
      }
    }).subscribeOn(Schedulers.io())//加载图片操作将在IO线程执行
        .observeOn(AndroidSchedulers.mainThread())//设置图片操作将在主线程执行
        .subscribe(new Subscriber<Drawable>() {
          @Override public void onCompleted() {

          }

          @Override public void onError(Throwable e) {
			Toast.makeText(MainActivity.this, "加载图片出错!!"+e.getMessage(), Toast.LENGTH_SHORT).show();
          }

          @Override public void onNext(Drawable drawable) {
            mImageview.setImageDrawable(drawable);
          }
        });

此时，加载图片将会发生在 IO 线程，而设置图片则被设定在了主线程。这就意味着，即使加载图片耗费了几十甚至几百毫秒的时间，也不会造成丝毫界面的卡顿。

线程切换的用法暂时先介绍到这里。

# 变换 #
RxJava 提供了对事件序列进行变换的支持，这是它的核心功能之一，也是大多数人说『RxJava 真是太好用了』的最大原因。

## map ##
    /**
     * 比如被观察者产生的事件中只有图片文件路径；,但是在观察者这里只想要bitmap,那么就需要类型变换。
     */
    Observable.just(R.mipmap.gyy)
        .observeOn(Schedulers.io())
        .map(new Func1<Integer, Bitmap>() {
          @Override public Bitmap call(Integer res) {
            return BitmapFactory.decodeResource(getResources(), res);
          }
        })
        .subscribeOn(AndroidSchedulers.mainThread())
        .subscribe(new Action1<Bitmap>() {
          @Override public void call(Bitmap bitmap) {
            mImageview.setImageBitmap(bitmap);
          }
        });
由上面的代码可以看到，使用操作符将事件处理逐步分解，通过线程调度为每一步设置不同的线程环境，完全解决了你线程切换的烦恼。可以说线程调度+操作符，才真正展现了RxJava无与伦比的魅力。

## flatMap ##
未完待续...
