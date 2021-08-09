# jetpack-LiveData

## 使用生命周期感知型组件处理生命周期

生命周期感知型组件可执行操作来响应另一个组件（如 Activity 和 Fragment）的生命周期状态的变化。这些组件有助于您编写出更有条理且往往更精简的代码，此类代码更易于维护。也就是将在生命周期不同时期的操作移到组件本身，将代码从Activity或者Fragment中抽离。

androidx.lifecycle软件包提供了可用于构建生命周期感知型组件的类和接口 - 这些组件可以根据 Activity 或 Fragment 的当前生命周期状态自动调整其行为。

### Lifecycle介绍

Lifecycle是一个类，用于存储有关组件（如 Activity 或 Fragment）的生命周期状态的信息，并允许其他对象观察此状态。

主要生命周期状态：

事件：从框架和 [`Lifecycle`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle) 类分派的生命周期事件。这些事件映射到 Activity 和 Fragment 中的回调事件。

状态：由 [`Lifecycle`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle) 对象跟踪的组件的当前状态。

![生命周期状态示意图](https://developer.android.com/images/topic/libraries/architecture/lifecycle-states.svg)

主要使用方法：

```java
public class MyObserver implements LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void connectListener() {
        ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void disconnectListener() {
        ...
    }
}

myLifecycleOwner.getLifecycle().addObserver(new MyObserver());
//owner是可以自定义的具有lifecycle流程的类
//添加Observer则是观察者具有listerner的名称，可以监控当前的lifecycle的结果，然后进行操作
//这样就可以尽量少的在activity或者fragment中更新所需要的代码，
//代码的具体实现官网可看
```



## LiveData

### 介绍和优势

- 使用 LiveData 构建数据对象，**在基础数据库改变时通知视图**。LiveData 是一种可观察的数据存储器类。与常规的可观察类不同，LiveData **具有生命周期感知能力**，意指它遵循其他应用组件（如 Activity、Fragment 或 Service）的生命周期。这种感知能力可确保 LiveData 仅更新处于活跃生命周期状态的应用组件观察者。

- 使用 LiveData构建数据对象，在基础数据库改变时通知视图。

- （介绍）ViewModel存储界面相关的数据，这些数据不会在应用旋转时销毁。

- （介绍）Room是一个 SQLite 对象映射库。它可用来避免样板代码，还可以轻松地将 SQLite 表数据转换为 Java 对象。Room 提供 SQLite 语句的编译时检查，并且可以返回 RxJava、Flowable 和 LiveData 可观察对象。

- 您可以注册与实现 LifecycleOwner接口的对象配对的观察者。有了这种关系，当相应的 Lifecycle对象的状态变为 DESTROYED 时，便可移除此观察者。这对于 Activity 和 Fragment 特别有用，因为它们可以放心地观察 LiveData对象，而不必担心泄露（当 Activity 和 Fragment 的生命周期被销毁时，系统会立即退订它们）。（ 也就是说有一个对象是实现了LifecycleOwner接口的，并且有一个观察者在看这个对象，当数据对象销毁后，相对应的观察者都可以去掉了）

- 优势：

  |                优势                |                           官网解释                           |                             理解                             |
  | :--------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
  |      **确保界面符合数据状态**      | LiveData 遵循观察者模式。当底层数据发生变化时，LiveData 会通知 Observer对象。您可以整合代码以在这些 `Observer` 对象中更新界面。这样一来，您无需在每次应用数据发生变化时更新界面，因为观察者会替您完成更新。 | 这里的通知Observer对象一般指listener对象，因此界面更新发生在listener中，这样就不需要当前类去操作界面，交由listener去做就可以了。 |
  |        **不会发生内存泄漏**        | 观察者会绑定到 Lifecycle对象，并在其关联的生命周期遭到销毁后进行自我清理。 |                                                              |
  | **不会因 Activity 停止而导致崩溃** | 如果观察者的生命周期处于非活跃状态（如返回栈中的 Activity），则它不会接收任何 LiveData 事件。 |                                                              |
  |    **不再需要手动处理生命周期**    | 界面组件只是观察相关数据，不会停止或恢复观察。LiveData 将自动管理所有这些操作，因为它在观察时可以感知相关的生命周期状态变化。 |      将数据和具体的操作类进行分开，不需要每次都进行定义      |
  |      **数据始终保持最新状态**      | 如果生命周期变为非活跃状态，它会在再次变为活跃状态时接收最新的数据。例如，曾经在后台的 Activity 会在返回前台后立即接收最新的数据。 |                                                              |
  |         **适当的配置更改**         | 如果由于配置更改（如设备旋转）而重新创建了 Activity 或 Fragment，它会立即接收最新的可用数据。 |                                                              |
  |            **共享资源**            | 您可以使用单例模式扩展 LiveData 对象以封装系统服务，以便在应用中共享它们。LiveData 对象连接到系统服务一次，然后需要相应资源的任何观察者只需观察 LiveData 对象。如需了解详情，请参阅扩展 LiveData。 |      减少对系统数据的获取操作，全都交给相对应的Livedata      |

  

### LiveData的具体使用

#### 大致使用流程

1.创建LivaData实例，一般在ViewModel中实现

2.观察对象。创建可定义 onChanged() 方法的 Observer 对象，该方法可以控制当 LiveData 对象存储的数据更改时会发生什么。通常情况下，您可以在界面控制器（如 Activity 或 Fragment）中创建 Observer 对象。**这一部分中的observer可以理解为listener。**

3.更新对象。使用 observe()方法将 `Observer` 对象附加到 `LiveData` 对象。**`observe()` 方法会采用 LifecycleOwner对象。**可以理解为是Owner对象可以使用观察者观察

ps：observeForver(Observer)/removeObserve()方法可以创建/删除一个不关联Owner对象的观察者，会始终在活跃状态。

#### 创建LiveData对象

LiveData 是一种可用于任何数据的**封装容器**，其中包括可实现 `Collections` 的对象，如 `List`。使用方式和其他容器类似。

```java
public class NameViewModel extends ViewModel {

// Create a LiveData with a String
private MutableLiveData<String> currentName;

    public MutableLiveData<String> getCurrentName() {
        if (currentName == null) {
            currentName = new MutableLiveData<String>();
        }
        return currentName;
    }

// Rest of the ViewModel...
}
```

#### 观察LivaData对象



#### 更新LivaData对象

#### LivaData和Room一起使用

#### 将协程与LivaData一起使用

### 扩展LivaData

### 转换LivaData

### 合并多个LivaData源
