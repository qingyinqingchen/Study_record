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

