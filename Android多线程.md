# Android多线程

## 多线程基础知识



## Android多线程实现方式

### 基础使用

#### 继承Thread类（java）

![示意图](https://tva1.sinaimg.cn/large/008i3skNgy1gva7tun0cpj60yg0h7dhh02.jpg)

```java
//常规使用
// 步骤1：创建线程类 （继承自Thread类）
   class MyThread extends Thread{
// 步骤2：复写run（），内容 = 定义线程行为
    @Override
    public void run(){
    ... // 定义的线程行为
    }
}
// 步骤3：创建线程对象，即 实例化线程类
  MyThread mt=new MyThread(“线程名称”);
// 步骤4：通过 线程对象 控制线程的状态，如 运行、睡眠、挂起  / 停止
// 此处采用 start（）开启线程
  mt.start();

//匿名类
//步骤1：采用匿名类，直接 创建 线程类的实例
 new Thread("线程名称") {
//步骤2：复写run（），内容 = 定义线程行为
      @Override
      public void run() {       
//步骤3：通过线程对象控制线程的状态如运行、睡眠、挂起/停止   
     }.start();
```



#### 实现Runnable接口（java）

![示意图](https://tva1.sinaimg.cn/large/008i3skNgy1gva7u18hekj60yg0g8t9t02.jpg)

```java
//常规步骤
// 步骤1：创建线程辅助类，实现Runnable接口
 class MyThread implements Runnable{
    ....
    @Override
// 步骤2：复写run（），定义线程行为
    public void run(){

    }
}

// 步骤3：创建线程辅助对象，即 实例化 线程辅助类
  MyThread mt=new MyThread();

// 步骤4：创建线程对象，即 实例化线程类；线程类 = Thread类；
// 创建时通过Thread类的构造函数传入线程辅助类对象
// 原因：Runnable接口并没有任何对线程的支持，我们必须创建线程类（Thread类）的实例，从Thread类的一个实例内部运行
  Thread td=new Thread(mt);

// 步骤5：通过 线程对象 控制线程的状态，如 运行、睡眠、挂起  / 停止
// 当调用start（）方法时，线程对象会自动回调线程辅助类对象的run（），从而实现线程操作
  td.start();


//匿名类
// 步骤1：通过匿名类 直接 创建线程辅助对象，即 实例化 线程辅助类
Runnable mt = new Runnable() {
// 步骤2：复写run（），定义线程行为
        @Override
        public void run() {
      }
 };
// 步骤3：创建线程对象，即 实例化线程类；线程类 = Thread类；
Thread mt1 = new Thread(mt, "窗口1");
// 步骤4：通过 线程对象 控制线程的状态，如 运行、睡眠、挂起  / 停止
mt1.start();
```

Thread方法和Runnable方法的比较：

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gvb6ycieotj60o60go75s02.jpg)



### 复合使用

#### AsyncTask

![示意图](https://tva1.sinaimg.cn/large/008i3skNgy1gva7ug9yc9j60yg0c6q4902.jpg)

```java
//定义和解释
public abstract class AsyncTask<Params, Progress, Result> { 
 ... 
}

// 类中参数为3种泛型类型
// 整体作用：控制AsyncTask子类执行线程任务时各个阶段的返回类型
// 具体说明：
    // a. Params：开始异步任务执行时传入的参数类型，对应excute（）中传递的参数
    // b. Progress：异步任务执行过程中，返回下载进度值的类型
    // c. Result：异步任务执行完成后，返回的结果类型，与doInBackground()的返回值类型保持一致
// 注：
    // a. 使用时并不是所有类型都被使用
    // b. 若无被使用，可用java.lang.Void类型代替
    // c. 若有不同业务，需额外再写1个AsyncTask的子类
}

//
```

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gvb7in1pr8j60pz0gngnq02.jpg)

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gvb7j0y8evj60vo08wmxx02.jpg)

使用步骤：

```java
/**
  * 步骤1：创建AsyncTask子类
  * 注： 
  *   a. 继承AsyncTask类
  *   b. 为3个泛型参数指定类型；若不使用，可用java.lang.Void类型代替
  *   c. 根据需求，在AsyncTask子类内实现核心方法
  */
  private class MyTask extends AsyncTask<Params, Progress, Result> {
        ....
      // 方法1：onPreExecute（）
      // 作用：执行 线程任务前的操作
      // 注：根据需求复写
      @Override
      protected void onPreExecute() {
           ...
        }

      // 方法2：doInBackground（）
      // 作用：接收输入参数、执行任务中的耗时操作、返回 线程任务执行的结果
      // 注：必须复写，从而自定义线程任务
      @Override
      protected String doInBackground(String... params) {

            ...// 自定义的线程任务

            // 可调用publishProgress（）显示进度, 之后将执行onProgressUpdate（）
             publishProgress(count);
              
         }

      // 方法3：onProgressUpdate（）
      // 作用：在主线程 显示线程任务执行的进度
      // 注：根据需求复写
      @Override
      protected void onProgressUpdate(Integer... progresses) {
            ...

        }

      // 方法4：onPostExecute（）
      // 作用：接收线程任务执行结果、将执行结果显示到UI组件
      // 注：必须复写，从而自定义UI操作
      @Override
      protected void onPostExecute(String result) {

         ...// UI操作

        }

      // 方法5：onCancelled()
      // 作用：将异步任务设置为：取消状态
      @Override
        protected void onCancelled() {
        ...
        }
  }

/**
  * 步骤2：创建AsyncTask子类的实例对象（即 任务实例）
  * 注：AsyncTask子类的实例必须在UI线程中创建
  */
  MyTask mTask = new MyTask();

/**
  * 步骤3：手动调用execute(Params... params) 从而执行异步线程任务
  * 注：
  *    a. 必须在UI线程中调用
  *    b. 同一个AsyncTask实例对象只能执行1次，若执行第2次将会抛出异常
  *    c. 执行任务中，系统会自动调用AsyncTask的一系列方法：onPreExecute() 、doInBackground()、onProgressUpdate() 、onPostExecute() 
  *    d. 不能手动调用上述方法
  */
  mTask.execute()；
```

ps:

1.在`Activity` 或 `Fragment`中使用 `AsyncTask`时，最好在`Activity` 或 `Fragment`的`onDestory（）`调用 `cancel(boolean)`；

2.`AsyncTask`应被声明为`Activity`的静态内部类

3.当`Activity`重新创建时（屏幕旋转 / `Activity`被意外销毁时后恢复），之前运行的`AsyncTask`（非静态的内部类）持有的之前`Activity`引用已无效，故复写的`onPostExecute()`将不生效，即无法更新UI操作。因此可以在`Activity`恢复时的对应方法 重启 任务线程



#### HandlerThread

![示意图](https://tva1.sinaimg.cn/large/008i3skNgy1gva7umcdn7j60yg0fjmyn02.jpg)

```java
// 步骤1：创建HandlerThread实例对象
// 传入参数 = 线程名字，作用 = 标记该线程
   HandlerThread mHandlerThread = new HandlerThread("handlerThread");

// 步骤2：启动线程
   mHandlerThread.start();

// 步骤3：创建工作线程Handler & 复写handleMessage（）
// 作用：关联HandlerThread的Looper对象、实现消息处理操作 & 与其他线程进行通信
// 注：消息处理操作（HandlerMessage（））的执行线程 = mHandlerThread所创建的工作线程中执行
  Handler workHandler = new Handler( mHandlerThread.getLooper() ) {
            @Override
            public boolean handleMessage(Message msg) {
                ...//消息处理
                return true;
            }
        });

// 步骤4：使用工作线程Handler向工作线程的消息队列发送消息
// 在工作线程中，当消息循环时取出对应消息 & 在工作线程执行相关操作
  // a. 定义要发送的消息
  Message msg = Message.obtain();
  msg.what = 2; //消息的标识
  msg.obj = "B"; // 消息的存放
  // b. 通过Handler发送消息到其绑定的消息队列
  workHandler.sendMessage(msg);

// 步骤5：结束线程，即停止线程的消息循环
  mHandlerThread.quit();
```



#### IntentService

![示意图](https://tva1.sinaimg.cn/large/008i3skNgy1gva7uuahauj60yg0bd3zc02.jpg)

```java
//步骤一：定义IntentService的子类
public class myIntentService extends IntentService {

  /** 
    * 在构造函数中传入线程名字
    **/  
    public myIntentService() {
        // 调用父类的构造函数
        // 参数 = 工作线程的名字
        super("myIntentService");
    }

   /** 
     * 复写onHandleIntent()方法
     * 根据 Intent实现 耗时任务 操作
     **/  
    @Override
    protected void onHandleIntent(Intent intent) {

        // 根据 Intent的不同，进行不同的事务处理
        String taskName = intent.getExtras().getString("taskName");
        switch (taskName) {
            case "task1":
                Log.i("myIntentService", "do task1");
                break;
            case "task2":
                Log.i("myIntentService", "do task2");
                break;
            default:
                break;
        }
    }

    @Override
    public void onCreate() {
        Log.i("myIntentService", "onCreate");
        super.onCreate();
    }
   /** 
     * 复写onStartCommand()方法
     * 默认实现 = 将请求的Intent添加到工作队列里
     **/  
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.i("myIntentService", "onStartCommand");
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        Log.i("myIntentService", "onDestroy");
        super.onDestroy();
    }
}
//步骤二：在Manifest.xml中注册服务
<service android:name=".myIntentService">
            <intent-filter >
                <action android:name="cn.scu.finch"/>
            </intent-filter>
        </service>
//步骤三：在activity中开启Service服务
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

            // 同一服务只会开启1个工作线程
            // 在onHandleIntent（）函数里，依次处理传入的Intent请求
            // 将请求通过Bundle对象传入到Intent，再传入到服务里

            // 请求1
            Intent i = new Intent("cn.scu.finch");
            Bundle bundle = new Bundle();
            bundle.putString("taskName", "task1");
            i.putExtras(bundle);
            startService(i);

            // 请求2
            Intent i2 = new Intent("cn.scu.finch");
            Bundle bundle2 = new Bundle();
            bundle2.putString("taskName", "task2");
            i2.putExtras(bundle2);
            startService(i2);

            startService(i);  //多次启动
        }
    }
```

与service的比较：

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gvb8lwkkqqj60s40800tr02.jpg)

与其他线程的比较：

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gvb8mlrggcj60ns07maaq02.jpg)

### 高级使用

#### 线程池

![示意图](https://tva1.sinaimg.cn/large/008i3skNgy1gva7v8sf3yj60yg0dqmyf02.jpg)

核心参数：

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gvb8px95rej60qy0iw75o02.jpg)

```java
//线程池使用方法
// 1. 创建线程池
   // 创建时，通过配置线程池的参数，从而实现自己所需的线程池
   Executor threadPool = new ThreadPoolExecutor(
                                              CORE_POOL_SIZE,
                                              MAXIMUM_POOL_SIZE,
                                              KEEP_ALIVE,
                                              TimeUnit.SECONDS,
                                              sPoolWorkQueue,
                                              sThreadFactory
                                              );
    // 注：在Java中，已内置4种常见线程池，下面会详细说明

// 2. 向线程池提交任务：execute（）
    // 说明：传入 Runnable对象
       threadPool.execute(new Runnable() {
            @Override
            public void run() {
                ... // 线程执行任务
            }
        });

// 3. 关闭线程池shutdown() 
  threadPool.shutdown();
  
  // 关闭线程的原理
  // a. 遍历线程池中的所有工作线程
  // b. 逐个调用线程的interrupt（）中断线程（注：无法响应中断的任务可能永远无法终止）

  // 也可调用shutdownNow（）关闭线程：threadPool.shutdownNow（）
  // 二者区别：
  // shutdown：设置 线程池的状态 为 SHUTDOWN，然后中断所有没有正在执行任务的线程
  // shutdownNow：设置 线程池的状态 为 STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表
  // 使用建议：一般调用shutdown（）关闭线程池；若任务不一定要执行完，则调用shutdownNow（）
```

常见的功能线程池：

定长线程池（`FixedThreadPool`）

- 特点：只有核心线程 & 不会被回收、线程数量固定、任务队列无大小限制（超出的线程任务会在队列中等待）
- 应用场景：控制线程最大并发数

- 具体使用：通过 *Executors.newFixedThreadPool()* 创建

- 示例：

```java
// 1. 创建定长线程池对象 & 设置线程池线程数量固定为3
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);

// 2. 创建好Runnable类线程对象 & 需执行的任务
Runnable task =new Runnable(){
  public void run(){
    System.out.println("执行任务啦");
     }
    };
        
// 3. 向线程池提交任务：execute（）
fixedThreadPool.execute(task);
        
// 4. 关闭线程池
fixedThreadPool.shutdown();
```

定时线程池（`ScheduledThreadPool` ）

- 特点：核心线程数量固定、非核心线程数量无限制（闲置时马上回收）
- 应用场景：执行定时 / 周期性 任务
- 使用：通过*Executors.newScheduledThreadPool()*创建
- 示例：

```csharp
// 1. 创建 定时线程池对象 & 设置线程池线程数量固定为5
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);

// 2. 创建好Runnable类线程对象 & 需执行的任务
Runnable task =new Runnable(){
       public void run(){
              System.out.println("执行任务啦");
          }
    };
// 3. 向线程池提交任务：schedule（）
scheduledThreadPool.schedule(task, 1, TimeUnit.SECONDS); // 延迟1s后执行任务
scheduledThreadPool.scheduleAtFixedRate(task,10,1000,TimeUnit.MILLISECONDS);// 延迟10ms后、每隔1000ms执行任务

// 4. 关闭线程池
scheduledThreadPool.shutdown();
```

可缓存线程池（`CachedThreadPool`）

- 特点：只有非核心线程、线程数量不固定（可无限大）、灵活回收空闲线程（具备超时机制，全部回收时几乎不占系统资源）、新建线程（无线程可用时）

> 任何线程任务到来都会立刻执行，不需要等待

- 应用场景：执行大量、耗时少的线程任务
- 使用：通过*Executors.newCachedThreadPool()*创建
- 示例：

```csharp
// 1. 创建可缓存线程池对象
ExecutorService cachedThreadPool = Executors.newCachedThreadPool();

// 2. 创建好Runnable类线程对象 & 需执行的任务
Runnable task =new Runnable(){
  public void run(){
        System.out.println("执行任务啦");
            }
    };

// 3. 向线程池提交任务：execute（）
cachedThreadPool.execute(task);

// 4. 关闭线程池
cachedThreadPool.shutdown();

//当执行第二个任务时第一个任务已经完成
//那么会复用执行第一个任务的线程，而不用每次新建线程。
```

单线程化线程池（`SingleThreadExecutor`）

- 特点：只有一个核心线程（保证所有任务按照指定顺序在一个线程中执行，不需要处理线程同步的问题）
- 应用场景：不适合并发但可能引起IO阻塞性及影响UI线程响应的操作，如数据库操作，文件操作等
- 使用：通过*Executors.newSingleThreadExecutor()*创建
- 示例：

```csharp
// 1. 创建单线程化线程池
ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();

// 2. 创建好Runnable类线程对象 & 需执行的任务
Runnable task =new Runnable(){
  public void run(){
        System.out.println("执行任务啦");
            }
    };

// 3. 向线程池提交任务：execute（）
singleThreadExecutor.execute(task);

// 4. 关闭线程池
singleThreadExecutor.shutdown();
```

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gvb92f6r9zj610w0goq6602.jpg)

### 对比

![示意图](https://tva1.sinaimg.cn/large/008i3skNgy1gva7vfvl3sj60yg0g3q5b02.jpg)

### 其他

#### 线程同步：Synchronized关键字

![示意图](https://tva1.sinaimg.cn/large/008i3skNgy1gva7vpv31hj60yg0dwwft02.jpg)

#### 线程变量：ThreadLocal

![示意图](https://tva1.sinaimg.cn/large/008i3skNgy1gva7vx7vhuj60yg0c0ab902.jpg)

## java的多线程实现方式补充

### Callable和future创建线程：

- 1. 创建 Callable 接口的实现类，并实现 call() 方法，该 call() 方法将作为线程执行体，并且有返回值。
- 2. 创建 Callable 实现类的实例，使用 FutureTask 类来包装 Callable 对象，该 FutureTask 对象封装了该 Callable 对象的 call() 方法的返回值。
- 3. 使用 FutureTask 对象作为 Thread 对象的 target 创建并启动新线程。
- 4. 调用 FutureTask 对象的 get() 方法来获得子线程执行结束后的返回值。

```java
public class CallableThreadTest implements Callable<Integer> {
    public static void main(String[] args)  
    {  
        CallableThreadTest ctt = new CallableThreadTest();  
        FutureTask<Integer> ft = new FutureTask<>(ctt);  
        for(int i = 0;i < 100;i++)  
        {  
            System.out.println(Thread.currentThread().getName()+" 的循环变量i的值"+i);  
            if(i==20)  
            {  
                new Thread(ft,"有返回值的线程").start();  
            }  
        }  
        try  
        {  
            System.out.println("子线程的返回值："+ft.get());  
        } catch (InterruptedException e)  
        {  
            e.printStackTrace();  
        } catch (ExecutionException e)  
        {  
            e.printStackTrace();  
        }  
  
    }
    @Override  
    public Integer call() throws Exception  
    {  
        int i = 0;  
        for(;i<100;i++)  
        {  
            System.out.println(Thread.currentThread().getName()+" "+i);  
        }  
        return i;  
    }  
}
```

## 原子类

原子类的主要作用是为了解决 synchronized 处理原子操作 **资源消耗过高** 的问题 ,原子类的CAS 基于 [Unsafe](https://juejin.cn/post/6939052022977003551) 实现 .Unsafe 是 CAS 的核心类 , 他提供了硬件级别得原子操作 (其他情况下Java 需要通过本地 Native 方法访问底层操作系统)

使用原子类的原因：需要获得比使用Sychronized更高的效率，又要获得更好的原子性。

- #### CAS操作：CompareAndSwap

  - CAS有3个操作数：

    > - 内存值V
    > - 旧的预期值A
    > - 要修改的新值B

  - 当多个线程尝试使用CAS同时更新同一个变量时，**只有其中一个线程能更新变量的值**(A和内存值V相同时，将内存值V修改为B)，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，**并可以再次尝试(或者什么都不做)**。

  - 我们可以发现CAS有两种情况：

    > - 如果内存值V和我们的预期值A**相等**，则将内存值修改为B，操作成功！
    > - 如果内存值V和我们的预期值A**不相等**，一般也有两种情况：
    >   - 重试(自旋)
    >   - 什么都不做

  - 理解CAS的核心就是：

    > **CAS是原子性的**，虽然你可能看到比较后再修改(compare and swap)觉得会有两个操作，但终究是原子性的！

- #### 原子变量类在`java.util.concurrent.atomic`包下，总体来看有这么多个

  > - 基本类型：
  >   - AtomicBoolean：布尔型
  >   - AtomicInteger：整型
  >   - AtomicLong：长整型
  > - 数组：
  >   - AtomicIntegerArray：数组里的整型
  >   - AtomicLongArray：数组里的长整型
  >   - AtomicReferenceArray：数组里的引用类型
  > - 引用类型：
  >   - AtomicReference：引用类型
  >   - AtomicStampedReference：带有版本号的引用类型
  >   - AtomicMarkableReference：带有标记位的引用类型
  > - 对象的属性
  >   - AtomicIntegerFieldUpdater：对象的属性是整型
  >   - AtomicLongFieldUpdater：对象的属性是长整型
  >   - AtomicReferenceFieldUpdater：对象的属性是引用类型
  > - JDK8新增DoubleAccumulator、LongAccumulator、DoubleAdder、LongAdder
  >   - 是对AtomicLong等类的改进。比如LongAccumulator与LongAdder在高并发环境下比AtomicLong更高效。

  - Atomic包里的类基本都是使用**Unsafe**实现的包装类

  - Unsafe里边有几个方法(CAS)：

    ```java
    // 第一和第二个参数代表对象的实例以及地址，第三个参数代表期望值，第四个参数代表更新值
    public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);
    
    public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
    
    public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);  
    ```

- #### 原子变量类使用

  ```java
  class Count{
      // 共享变量(使用AtomicInteger来替代Synchronized锁)
      private AtomicInteger count = new AtomicInteger(0);
      public Integer getCount() {
          return count.get();
      }
      public void increase() {
          count.incrementAndGet();
      }
  }
  ```

- #### 原子类的ABA问题

  - 下面的操作都可以正常执行完的，这样会发生什么问题呢？？线程C无法得知线程A和线程B修改过的count值，这样是有**风险**的。

    > 1. 现在我有一个变量`count=10`，现在有三个线程，分别为A、B、C
    > 2. 线程A和线程C同时读到count变量，所以线程A和线程C的内存值和预期值都为10
    > 3. 此时线程A使用CAS将count值修改成100
    > 4. 修改完后，就在这时，线程B进来了，读取得到count的值为100(内存值和预期值都是100)，将count值修改成10
    > 5. 线程C拿到执行权，发现内存值是10，预期值也是10，将count值修改成11

- #### 解决ABA问题

  > 要解决ABA的问题，我们可以使用JDK给我们提供的AtomicStampedReference和AtomicMarkableReference类。
  >
  > 简单来说就是在给为这个对象提供了一个**版本**，并且这个版本如果被修改了，是自动更新的。
  >
  > 原理大概就是：维护了一个Pair对象，Pair对象存储我们的对象引用和一个stamp值。每次CAS比较的是两个Pair对象

- #### LongAdder 性能比 AtomicLong 要好

  - 使用AtomicLong时，在高并发下大量线程会同时去竞争更新**同一个原子变量**，但是由于同时只有一个线程的CAS会成功，所以其他线程会不断尝试自旋尝试CAS操作，这会浪费不少的CPU资源。
  - 而LongAdder可以概括成这样：内部核心数据value**分离**成一个数组(Cell)，每个线程访问时,通过哈希等算法映射到其中一个数字进行计数，而最终的计数结果，则为这个数组的**求和累加**。
    - 简单来说就是将一个值分散成多个值，在并发的时候就可以**分散压力**，性能有所提高。

参考链接：https://www.cnblogs.com/jingmoxukong/p/12109049.html