# Demo 地址 #
[https://github.com/jutao/aidl](https://github.com/jutao/aidl "AIDL服务端")
[AIDL客户端](https://github.com/jutao/aidlclient)

# 什么是AIDL #
在 Android中，每一个应用程序独自拥有一个虚拟机，这样做虽然保证了进程之内数据的安全性，保证一个应用程序的数据不受其他应用程序的影响，也保证了一个应用程序挂掉了不至于影响其他应用程序。但是这样也造成了一个应用程序和另外一个应用程序没办法直接进行通讯。AIDL 的作用就是使来自不同应用的客户端跨进程通信访问你的 Service。
AIDL 是 Android Interface Definition Language 的缩写，就是安卓内部通信接口描述语言。关于 AIDL的描述和用法我主要参考了Google 的官方API，找到一个中文的，链接：[http://www.android-doc.com/guide/components/aidl.html](http://www.android-doc.com/guide/components/aidl.html "AIDL")。需要补充的是，API里描述AIDL支持Java语言中的所有基本数据类型，但是经过查证和实验，实际上 AIDL 是不支持Short类型的。

# AIDL用法 #
## 创建 AIDL 接口 ##
![](http://i.imgur.com/hIeNjic.png)
如果是 Eclipse 的话需要创建 File 并且不要忘记自己打上后缀 .aidl。

## 写 AIDL 接口 ##
AIDL 的用法基本和写普通 Java 接口相同，需要注意的是包名一定要自己检查一下，还有导包也要自己写一下。

    package com.example.jutao.aidl;
    interface IServiceAidl {
    //计算两个数的和
    int add(int num1,int num2);
    }
写完之后需要注意，如果你写的 AIDL 接口正确，那么 Ecipse 是会自动编译的，而 Android Studio 需要手动编译，编译按钮如下图所示：
![](http://i.imgur.com/NnSnU2a.png)
编译通过后，Android Studio 所生成的文件在
![](http://i.imgur.com/VTAkxIR.png)

## 写 Service ##
      //当客户端绑定到该服务的时候
      @Override public IBinder onBind(Intent intent) {
        //当别人绑定服务的时候，就会得到AIDL接口
        return iBinder;
      }
      IBinder iBinder = new IServiceAidl.Stub() {
        @Override public int add(int num1, int num2) throws RemoteException {
        Log.d("TAG", "收到服务端请求,求出" + num1 + "和" + num2 + "的和");
        return num1 + num2;
    }
    };


 		<!-- exported 是否支持其它应用调用当前组件 -->
 		<!-- enabled 这个属性用于指示该服务是否能够被实例化。如果设置为true，则能够被实例化，否则不能被实例化。默认值是true -->
        <service
            android:name="com.example.aidl_service.RemoteService"
            android:enabled="true"
            android:exported="true" >
        </service>
## 写客户端 ##
客户端的主要功能是用户通过界面输入两个数字，点击远程计算按钮后通过服务端代码计算出结果返回给客户端并显示。
![](http://i.imgur.com/zoqkeKX.png)
点击按钮后
![](http://i.imgur.com/JAs8dhp.png)
需要注意的是，客户端也需要有一模一样的 AIDL 包，连包名都要一模一样！！

    //1、获取服务端
    Intent intent = new Intent();
    //Android 5.0之后不支持隐式意图，必须是显式意图来启动绑定服务
    intent.setComponent(
    new ComponentName("com.example.jutao.aidl", "com.example.jutao.aidl.RemoteService"));
    //第三个参数是一个flag，绑定时自动启动
    bindService(intent, conn, Context.BIND_AUTO_CREATE);
conn的定义:

	ServiceConnection conn = new ServiceConnection() {
    //绑定服务时
    @Override public void onServiceConnected(ComponentName name, IBinder service) {
      //拿到了远程的服务
      iServiceAidl = IServiceAidl.Stub.asInterface(service);
    }

    //当服务断开时
    @Override public void onServiceDisconnected(ComponentName name) {
      //回收资源
      iServiceAidl = null;
    }
      };

# AIDL 自定义类型 #
![](http://i.imgur.com/xnkwiMg.png)
AIDL 默认支持的数据类型如上图所示，虽然支持List类型，但是需要 在List前注明输入List还是输出List，下面的例子会讲到。
首先 person 类要实现 Parcelable 接口，详细代码可以在我开头贴的Demo里看。

    import com.example.jutao.aidl.Person;
    
    interface PersonAidl {
       List<Person> add(in Person person);
    }
    parcelable Person;

    

    public class PersonService extends Service {
      private ArrayList<Person> persons;
    
      public PersonService() {
      }
    
      @Override public IBinder onBind(Intent intent) {
    persons = new ArrayList<Person>();
    
    return iBinder;
      }
    
      private IBinder iBinder = new PersonAidl.Stub() {
    
    @Override public List<Person> add(Person person) throws RemoteException {
      persons.add(person);
      return persons;
    }
      };
    }


![](http://i.imgur.com/78H8w2N.png)
可以看到，我每次输出的都是 persons 这一List，这是通过服务端返回的，说明我传输过去的值已经被服务端接收并存储。