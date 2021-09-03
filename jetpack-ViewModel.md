# jetpack-ViewModel

ViewModel类旨在以注重生命周期的方式存储和管理界面相关的数据。ViewModel类让数据可在发生屏幕旋转等配置更改后继续留存。

当某个界面有信息需要存储的时候随着界面的变化也会消失，虽然可以使用onSaveInstanceState（），但这个方法仅适合可以序列化和反序列化的少量数据，不适合较大的数据。

## ViewModel的生命周期

ViewModel的对象存在时间是获取Viewmodel时传递给ViewModelProvider的lifecycle存在时间。

![说明 ViewModel 随着 Activity 状态的改变而经历的生命周期。](https://developer.android.com/images/topic/libraries/architecture/viewmodel-lifecycle.png)

## 在Fragment之间共享数据（小例子）

```java
public class SharedViewModel extends ViewModel {
    private final MutableLiveData<Item> selected = new MutableLiveData<Item>();

    public void select(Item item) {
        selected.setValue(item);
    }

    public LiveData<Item> getSelected() {
        return selected;
    }
}

public class MasterFragment extends Fragment {
    private SharedViewModel model;

    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        model = new ViewModelProvider(requireActivity()).get(SharedViewModel.class);
        itemSelector.setOnClickListener(item -> {
            model.select(item);
        });
    }
}

public class DetailFragment extends Fragment {

    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        SharedViewModel model = new ViewModelProvider(requireActivity()).get(SharedViewModel.class);
        model.getSelected().observe(getViewLifecycleOwner(), item -> {
           // Update the UI.
        });
    }
}
//这一部分是写了两个fragment监控同一个数据，是在同一个viewmodel中的数据，MasterFragment是改变数据用的，而detail是设置数据的。
```

### ViewModel

1. 缓存在内存中的，相当于本地缓存和网络缓存读写比较快
2. 可以存较大的值，比如bitmap、大对象等等
3. 数据绑定了Activity或者Fragment的生命周期，当Activity正常关闭的时候，都会清除ViewModel中的数据.比如
   - 调用finish()
   - 按返回键退出，
   - 用户直接杀进程或者是放后台内存不足，被回收了
4. 在Activity旋转或者其他配置改变的时候，ViewModel里面是还有值得，不会销毁，
5. ViewModel也可以使用本地存储,通过SavedStateHandle实现的，使用SavedStateViewModelFactory这个工厂，引用是`implementation "androidx.lifecycle:lifecycle-viewmodel-savedstate:2.2.0`

6，也可以new ViewModle的，new出来的 基本上就没有ViewModle的特性了。相当于一个真正的Present层了

###  onSaveInstanceState

1. 仅适合可以序列化再反序列化的少量数据
2. 不适合数量可能较大的数据，如用数组或Bitmap。可以存一些简单的基本类型和简单的小对象、例如字符串，和一些id
3. 会把数据存储到磁盘上

