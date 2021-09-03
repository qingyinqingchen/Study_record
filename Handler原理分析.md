# Handler原理浅析

## 一、Handler介绍

官翻：Handler是一个可以通过关联一个消息队列来发送和处理消息，发送或处理Runnable对象的一个**处理程序**，每个Handler都关联一个单个的线程和消息队列，当你创建一个新的Handler的时候它就将绑定到一个线程或线程上的消息队列，从那时起，这个Handler就将为这个消息队列提供消息或Runnable对象，处理消息队列释放出来的消息或Runnable对象。

Handler有两个主要的用途

- 1 安排和处理消息和Runnable对象在某个时间
- 2 将一个动作放在不同的线程上执行



## 二、Handler常见用法

### 子线程到主线程

![img](http://www.gaohaiyan.com/guide_north/content/4_activity/4_2.files/4_2-1285.png)

上图是子线程到主线程的消息流动路线，在主线程中保留handler实例，子线程在创建的过程中接受主线程的实例，通过在handler中添加消息进行传递。

```java
//UI线程代码
public class MainActivity extends Activity{
  private Handler handler=new Handelr();
  public void handleMessage(Message msg){
    super.handleMessage(msg);
    if(msg.what ==99){
      doSomething...
    }
  }
}
//子线程代码
public class ChildThread1 extends Thread{
  private Handler handler;
  public ChildThread1(Handler handler){
    this.handler = handler;
  }
  @Override
  public void run(){
    super.run();
    Message msg = handler.obtainMessage();
    msg.what == 99;
    msg.obj = ".....";
    handler.sendMessage(msg);
  }
}

//使用
new ChildThread1(handler).start();
```

### 主线程到子线程

这种情况比较少，但也需要了解。关键：子线程要有自己的Looper。

![img](http://www.gaohaiyan.com/guide_north/content/4_activity/4_2.files/4_2-2301.png)

```java
public class MainActivity extends Activity {
  /** 在子线程内部类里实例化的消息处理器，测试主UI向子线程发消息 **/
  private Handler handler3;
  @Override
  protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
   new InnerThread().start();  // 在UI创建时（onCreate）启动子线程，否则NullPointException
  }
// 通过事件触发，如点击按钮
  public void onTest(View v) {
   Message msg1 = handler3.obtainMessage();
   msg1.what = 100;
   handler3.sendMessage(msg1);
  }
  /** 内部类，一个子线程 **/
  class InnerThread extends Thread {
    @Override
    public void run() {
     super.run();
      // 子线程仍然须要有自己的消息泵
     Looper.prepare();
    // 主线程声明，子线程初始化
      handler3 = **new** Handler() {
       @Override
       public void handleMessage(Message msg) {
         super.handleMessage(msg);
          if (msg.what == 100) {
         Log.e("ard", "主线程->内部类消息");
         }
        }
      };
     Looper.*loop*();
    }
  }
}
```

### 进程间通信-使用Messenger

使用步骤：

Messenger使用步骤：

1. 服务端创建一个Handler，用来接收客户端每个调用的回调
2. Handler用于创建Messenger对象（对Handler的引用），在Handler中的 handleMessage()方法中处理客户端传过来的Message。
3. Messenger创建一个IBinder，服务端通过onBind()方法使其返回客户端
4. 客户端使用IBinder将Messenger（引用服务的Handler）实例化，然后使用后者将Message对象发送给服务端。
5. 如果客户端想接收服务器的响应，则需要客户端也创建一个Messenger，当客户端收到ServiceConnection中的onServiceConnected()回调时，会向服务端发送一条Message，**将客户端的Messenger赋值给Message的replyTo参数**。

```java
//客户端
//先创建Massenger,里面有Handler
Messenger messenger = new Messenger(new ClientHandler());
private class ClientHandler extends Handler {
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        switch (msg.what) {
             //根据msg.what来区分服务器返回的不同信息
            case ServerThinking:
                showMessage("\n服务器正在绞尽脑汁的思考...");
                break;
            case MESS_FROM_SERVER:
                Bundle bundle = msg.getData();
                String responseInfo = bundle.getString(RESPONSE_KEY);
                if (!TextUtils.isEmpty(responseInfo)) {
                    showMessage("\n" + responseInfo, R.color.red_f);
                }
                break;
        }
    }
}
//再创建和服务端链接的Connection，用来接受链接成功的消息
private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            //连接成功
            showMessage("\n连接远程服务端成功!", R.color.blue_color);
            isBound = true;
            mService = new Messenger(service);
            String question = "\n客户端：程序猿最讨厌皇帝的第几个儿子？";
            Message message = Message.obtain(null, MESS_FROM_CLIENT);
           //Messenger用来接收服务端发来的信息
            message.replyTo = messenger;
            try {
                //将Message发送到服务端
                mService.send(message);
                showMessage(question, R.color.red_f);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
        @Override
        public void onServiceDisconnected(ComponentName name) {
            //连接失败
            mService = null;
            isBound = false;
        }
    };
//绑定Service并且设置动作
    Intent intent = new Intent();
intent.setAction("android.mq.messenger.service");
intent.setPackage("org.ninetripods.mq.multiprocess_sever");
bindService(intent, connection, BIND_AUTO_CREATE);

//服务端-service-先声明
private HandlerThread thread;
private Handler serverHandler;
private Messenger messenger;
@Override
public void onCreate() {
    super.onCreate();
    //创建一个名字为my_thread的线程
    thread = new HandlerThread("my_thread");
    //启动一个线程
    thread.start();
    //在这个线程中创建一个handler
    serverHandler = new Handler(thread.getLooper()) {
       @Override
       public void handleMessage(Message msgFromClient) {
           super.handleMessage(msgFromClient);
           Message replyToClient = Message.obtain(msgFromClient);
             switch (msgFromClient.what) {
             //根据Message.what来判断执行服务端的哪段代码
                case MESS_FROM_CLIENT:
                   replyToClient.what = ServerThinking;
                     try {
                         Thread.sleep(1000);
                         //向客户端发送Message
                         msgFromClient.replyTo.send(replyToClient);
                         Thread.sleep(3 * 1000);
                        } catch (Exception e) {
                          e.printStackTrace();
                        }
                        replyToClient.what = MESS_FROM_SERVER;
                        Bundle bundle = new Bundle();
                        bundle.putString(RESPONSE_KEY, "服务器：我知道答案");
                        replyToClient.setData(bundle);
                        try {
                            //通过Bundle 封装数据并发送到客户端
                            msgFromClient.replyTo.send(replyToClient);
                        } catch (RemoteException e) {
                            e.printStackTrace();
                        }
                        break;
                }
            }
        };
        //构建一个Messenger
        messenger = new Messenger(serverHandler);
    }
    @Override
    public IBinder onBind(Intent intent) {
        //通过onBind()返回Binder实例供客户端调用
        return messenger.getBinder();
    }
```

## 三、Handler的消息机制



![img](https://upload-images.jianshu.io/upload_images/2846669-2c542831e231b9d7.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

### Handler

```java
//Handler的初始化，这里面包括队列的绑定和使用当前线程的Looper做轮询
public Handler(Callback callback, boolean async) {
        ......
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()"); // 说明了如果Handler在没有Looper的线程中创建则会抛出异常
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }

//Handler执行插入数据的操作
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;//在这里处理数据的绑定，在looper的时候通过这个值进行确认handler
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }

//Handler处理消息的顺序：自定义的CallBack-》mCallBack-〉handleMessage方法
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
    
    private static void handleCallback(Message message) {
        message.callback.run();
    }
```

Handler中处理消息的顺序是：自定义的CallBack-》mCallBack-〉handleMessage方法。发送方法post和sentmessage方法没有不同，只是用法不一样。

### 消息队列

消息队列主要包含两个操作：插入（enqueueMessage）和读取（next）

1.读取本身会伴随着删除操作，next方法是一个无限循环的方法，如果消息队列中没有消息，那么Next方法会一直阻塞，当有新消息到来时，next会返回这条消息并移除

2.插入：单链表维护消息队列，插入也就是单链表插入操作。插入过程中是按照时间戳的顺序，也就是如果是延时传送的消息会按照实际执行时间排序。一般是插入到队列头。

### Looper

不停的从MessageQueue中查看是否有新消息。looper创建消息队列。然后保存当前的线程的对象。

Handler的工作需要Looper，没有就会报错，使用prepare（）创建。给主线程创建就是prepareMainLooper方法。Looper的looper方法就是不断的读取queue的next方法并处理。prepareMainLooper方法是由系统自行调用，一般不需要自己调用。主要的操作如下注释：

```Java
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    if (me.mInLoop) {
        Slog.w(TAG, "Loop again would have the queued messages be executed"
                + " before this one completed.");
    }

    me.mInLoop = true;
    final MessageQueue queue = me.mQueue;
    Binder.clearCallingIdentity();//设置调用当前looper的身份，并跟踪
    final long ident = Binder.clearCallingIdentity();
    final int thresholdOverride =//设置超时的阈值，用来设定slowDispatchThresholdMs和slowDeliveryThresholdMs
            SystemProperties.getInt("log.looper."
                    + Process.myUid() + "."
                    + Thread.currentThread().getName()
                    + ".slow", 0);

    boolean slowDeliveryDetected = false;

    for (;;) {
        Message msg = queue.next(); // 可能会在这个地方阻塞，因为有消息的时候会发送，没有消息的时候就阻塞
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

        // This must be in a local variable, in case a UI event sets the logger
        final Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }
        // 设置观察者，防止更改
        final Observer observer = sObserver;
				。。。。
        。。。。
        if (observer != null) {
            token = observer.messageDispatchStarting();
        }
        long origWorkSource = ThreadLocalWorkSource.setUid(msg.workSourceUid);
        try {
            msg.target.dispatchMessage(msg);//主要就是在这里处理消息
            if (observer != null) {
                observer.messageDispatched(token, msg);
            }
            dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
        } catch (Exception exception) {
            if (observer != null) {
                observer.dispatchingThrewException(token, msg, exception);
            }
            throw exception;
        } finally {
            ThreadLocalWorkSource.restore(origWorkSource);
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }
      。。。。
        。。。。
        msg.recycleUnchecked();//消息使用完之后回收
    }
}
```

### **解释下在单线程模型中Message、Handler、MessageQueue、Looper之间的关系。**

- 1.`Handler`
- 发送消息到消息队列，并且执行从消息队列中取出的消息。
- 2.`Looper`
- 每个线程只有一个`Looper`，并且Looper中包含了消息队列，Looper中会进行`loop()`消息循环，不断地从消息队列中取出消息来给Handler进行处理。
- 3.`MessageQueue`
- 消息队列用来存放Handler发送过来的消息，一个消息队列中可包含多个消息Message。当Looper创建的时候会自动创建消息队列存储在Looper中。
- 4.`Message`
- 消息对象。建议通过`Message.obtain()`从对象池中取出数据，并且在使用完消息之后会通过`recycleUnchecked`进行将消息回收到对象池中。

### Looper和Handler的对应关系：

一个线程只能创建一个Looper，但是可以对应无数个Handler，使用的消息队列也是同一个。不同的Handler是怎么做到处理自己发出的消息的，看如下代码：

```Java
private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
        long uptimeMillis) {
    msg.target = this;//这个地方给消息加上Handler赋值，在Looper处理消息的过程中就可以区分开不同的Handler了，方法在Looper.loop中
    msg.workSourceUid = ThreadLocalWorkSource.getUid();

    if (mAsynchronous) {//设置为强制异步的方法处理消息，省的处理UI线程容易阻塞
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```



参考链接：

https://blog.csdn.net/u011240877/article/details/72892321

https://peterxiaosa.github.io/2019/05/29/Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6/

https://peterxiaosa.github.io/2020/07/14/IdleHandler%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90/