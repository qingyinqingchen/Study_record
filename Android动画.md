# Android动画

## 传统动画

帧动画和补间动画的区别：帧动画是一帧一帧的刷新，是所有进行替换。补间动画只有四种方式，是对整体的移动，帧动画做不到

### 帧动画

代码要做的事情就是把一幅幅的图片按顺序显示，造成动画的视觉效果。

#### 帧动画的使用方式

设置动画资源的三种方式：

**第一种：**
 // 设置动画
 imageView.setImageResource(R.drawable.frameanimation);
 // 获取动画对象
 animationDrawable = (AnimationDrawable)imageView.getDrawable();
 animationDrawable.start();

**第二种**
 设置背景：
 imageView.setBackgroundResource(R.drawable.frameanimation);
 animationDrawable = (AnimationDrawable) imageView.getBackground()
 **第三种**
 直接获取然后设置：
 animationDrawable = (AnimationDrawable) getResources().getDrawable(
 R.drawable.frameanimation);
 imageView.setBackground(animationDrawable)

**2.2代码使用方式**
 animationDrawable = new AnimationDrawable();
 // 为AnimationDrawable添加动画帧
 animationDrawable .addFrame(getResources().getDrawable(R.drawable.d1), 50);
 imageView.setBackground(animationDrawable );

#### 使用xml实现帧动画的步骤

1.首先将需要展示的帧放在Drawable文件夹中，设置好drawable属性，也就是资源文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:duration="100" android:drawable="@drawable/frame1"></item>
    <item android:duration="100" android:drawable="@drawable/frame2"></item>
    <item android:duration="100" android:drawable="@drawable/frame3"></item>
    <item android:duration="100" android:drawable="@drawable/frame4"></item>
    <item android:duration="100" android:drawable="@drawable/frame5"></item>
</animation-list>
```

2.在xml布局文件中设置好background属性

```xml
<TextView

    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:id="@+id/hello"
    android:text="Hello World!"
    android:background="@drawable/frameanimatiotest"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

3.在代码中调用开始结束

```java
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        TextView textView = findViewById(R.id.hello);
        AnimationDrawable animationDrawable = (AnimationDrawable)textView.getBackground();
        if(!isShowing){
            animationDrawable.start();
            isShowing=true;
        }else{
            animationDrawable.stop();
            isShowing=false;
        }

    }
});
//手动设置帧的方式二
AnimationDrawable animationDrawable = new AnimationDrawable();
animationDrawable.setOneShot(false);
animationDrawable.addFrame(getResources().getDrawable(R.drawable.compositedst1),1000);
animationDrawable.addFrame(getResources().getDrawable(R.drawable.compositedst2),1000);
animationDrawable.addFrame(getResources().getDrawable(R.drawable.compositedst3),1000);
animationDrawable.addFrame(getResources().getDrawable(R.drawable.compositedst4),1000);
imageView.setImageDrawable(animationDrawable);
animationDrawable.start();
```

### 补间动画

1.通常使用XML文件的形式实现，使用的时候先用AnimationUtils.loadAnimation 然后视图view.startAnimation。

 2.可以使用set将各种动画组合在一起：https://www.jianshu.com/p/420629118c10 

3.Interpolator 主要作用是可以控制动画的变化速率 ，就是动画进行的快慢节奏。 

4.pivot 决定了当前动画执行的参考位置 

5.也可以代码决定

ps：动画文件要存放在res/anim文件夹下，访问时采用R.anim.XXX的方式。默认是没有这个文件夹的需要手动创建（右键res目录-->New-->Android Resource Directory-->确定。）

## 属性动画

顾名思义，通过控制对象的属性，来实现动画效果。官方定义：定义一个随着时间 （注：停个顿）更改任何对象属性的动画，无论其是否绘制到屏幕上。

可以控制对象什么属性呢？什么属性都可以，理论是通过set和get某个属性来达到动画效果。例如常用下面一些属性来实现View对象的一些动画效果。

- **位移**:`translationX`、`translationY`、`translationZ`

- **透明度**:`alpha`,透明度全透明到不透明:`0f`->`1f`

- **旋转**:`rotation`,旋转一圈:`0f`->`360f`

- **缩放**：水平缩放`scaleX`,垂直缩放`scaleY`

### 定义属性动画方法

  可以使用xml和code方式创建

###### xml：

  先在animation文件夹中创建xml动画文件，如果没有animation文件需要手动创建

  ```xml
  <objectAnimator   
  android:duration="int"   android:interpolator="@[package:]anim/interpolator_resource"   android:propertyName="string"   
  android:valueType=["intType" | "floatType"]   
  android:valueFrom="float | int | color"   
  android:valueTo="float | int | color"   
  android:startOffset="int"   
  android:repeatCount="int"   
  android:repeatMode=["repeat" | "reverse"]   />
  
  //2. 在代码中导入动画xml
  val objectAnimator = AnimatorInflater.loadAnimator(this, R.animator.object_animator) as ObjectAnimator
  objectAnimator.setTarget(target)//target代表想要设置动画的对象
  objectAnimator.start()
  ```

  | 属性         | 含义                               | 取值范围                                                     |
  | ------------ | ---------------------------------- | ------------------------------------------------------------ |
  | duration     | 动画执行时间                       | 整型，需要大于0。                                            |
  | interpolator | 差值器，改变动画或者数值变化的速率 | 安卓自带，自定义                                             |
  | propertyName | 动画目标对象要改变的属性           | 字符串，填入属性名称                                         |
  | valueType    | 值类型                             | 整点型，浮点型。当属性动画值的类型为颜色值时可以省略         |
  | valueFrom    | 动画值的起始值                     | 浮点数，整型数或者颜色值。当为颜色值时，必须符合颜色的定义方式（# + 六位十六进制数） |
  | valueTo      | 动画值的结束值                     | 浮点数，整型数或者颜色值。当为颜色值时，必须符合颜色的定义方式（# + 六位十六进制数） |
  | startOffset  | 动画开始偏移时间                   | 整型数，默认为 0。当为负数时，效果和默认值一样               |
  | repeatCount  | 动画重复的次数                     | 整型数字，默认为 0。当为负数时，表示无限循环                 |
  | repeatMode   | 下一次动画执行的方式               | 默认：重新开始播放。还有倒播                                 |

###### code：

ValueAnimator

ValueAnimator正如其名字，它是关于数字的计算动画。它不会与View直接交互，而是通过ValueAnimator#addUpdateListener来设置动画效果。比如在TextView上显示从0到50时，就可以用ValueAnimator。这个动画是针对**属性的值进行动画的** ，不会对UI造成改变，不能直接实现动画效果。需要通过对动画的监听去做一些操作，在监听中将这个值设置给对应的属性，对应的属性才会改变。

```java

// 创建Animator，同时传入取值范围 
val animator = ValueAnimator.ofFloat(0F, 5000F)  
// 设置差值器， 这里选择的是线性差值器，如果不设置默认的插值器则默认的是AccelerateDecelerateInterpolator，如果设置的是null，则会使用LinearInterpolator
animator.interpolator = LinearInterpolator() 
// 设置动画执行时间 
animator.setDuration(2000)
  
  
 //当数值发生改变的时候会调用onAnimationUpdate接口，所以需要实现这个listener
 animator.addUpdateListener {
    // 在TextView中更新text
    binding.textView.text = ((it.animatedValue as Float).toInt() / 100).toString()
    // 把值传入自定义贝塞尔View，是其产生动画效果
    binding.bezierView.setValue(it.animatedValue as Float)
}

在需要的节点上开始动画和结束动画
// 播放动画
animator.start()
// 动画暂定
animator.pause()
// 播放结束
animator.end()
```

objectAnimator

```java
// 创建动画， 会对对象直接操作
val objectAnimator = ObjectAnimator.ofFloat(binding.imageView, "rotation", 0F, 360F, 0F) 
// 设置动画执行时间 
objectAnimator.setDuration(2000) 
// 开始播放 
objectAnimator.start()
```

创建动画时的第一个传参是被动画对象View，第二个是`properyName`,第三个是变长参数的起始数值。

其中`propertyName`的类型一共有以下几种：

| 名称         | 作用      |
| ------------ | --------- |
| alpha        | 透明化    |
| rotation     | 旋转      |
| rotationX    | 以X轴旋转 |
| rotationY    | 以Y轴旋转 |
| translationX | 以X轴平移 |
| translationY | 以Y轴平移 |
| scaleX       | 以X轴拉伸 |
| scaleY       | 以Y轴拉伸 |

### AnimatorSet的使用：

###### Code:

```java
//方法一：
//定义两个动画
ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(btnShow,"translationX",0,300);
ObjectAnimator objectAnimator1 = ObjectAnimator.ofFloat(btnShow,"alpha",0.1f,1f);
//整合起来
AnimatorSet animatorSet = new AnimatorSet();
//playSequentially表示顺序执行，playTogether表示同步执行
animatorSet.playSequentially(objectAnimator,objectAnimator1);
animatorSet.setDuration(2000);
animatorSet.setInterpolator(new BounceInterpolator());
animatorSet.start();

//方法二：使用builder方式
ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(btnShow, "translationX", 0, 300);
ObjectAnimator objectAnimator1 = ObjectAnimator.ofFloat(btnShow, "rotation", 0, 360);
AnimatorSet animatorSet = new AnimatorSet();
animatorSet.setDuration(2000);
animatorSet.play(objectAnimator).after(objectAnimator1);
animatorSet.start();

//AnimatorSet有一个内部类AnimatorSet.Builder，AnimatorSet只提供了playSequentially和playTogether两种方式，而AnimatorSet.Builder可以自由组合各种方式。
AnimatorSet.Builder通过调用AnimatorSet的play方法获取。
AnimatorSet.Builder有四个方法：
after(long delay)	在一定时间后执行play里的动画
after(Animator anim)	先执行after里的动画，再执行play里的动画
before(Animator anim)	先执行play里的动画，再执行after里的动画
with(Animator anim)	一起执行
```

###### XML:

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:ordering="together">
    <objectAnimator
        android:valueFrom="0"
        android:valueTo="360"
        android:propertyName="rotation"/>
    <objectAnimator
        android:valueFrom="0"
        android:valueTo="360"
        android:propertyName="translationX"/>
</set>
android:orderin有两个值，sequentially和together，分别对应playSequentially和playTogether两个方法的效果
java
AnimatorSet animatorSet = (AnimatorSet) AnimatorInflater.loadAnimator(context,R.animator.anim_set);
                animatorSet.setTarget(btnShow);
                animatorSet.setDuration(2000);
                animatorSet.start();
```

###### 对animation的监听：

AnimatorSet内部有一个私有的监听AnimatorSetListener，这个用来他自己处理内部动画。

自己加监听一般是Animator.AnimatorListener

```java
animatorSet.addListener(new Animator.AnimatorListener() {
                    @Override
                    public void onAnimationStart(Animator animation) {
                        Log.i(TAG, "onAnimationStart: ");
                    }
                    @Override
                    public void onAnimationEnd(Animator animation) {
                        Log.i(TAG, "onAnimationEnd: ");
                    }
                    @Override
                    public void onAnimationCancel(Animator animation) {
                        Log.i(TAG, "onAnimationCancel: ");
                    }
                    @Override
                    public void onAnimationRepeat(Animator animation) {
                        Log.i(TAG, "onAnimationRepeat: ");
                    }
                });
//这个时候，AnimatorSet中的所有动画成了一个整体，onAnimationEnd是在所有动画完成后才会调用，不管是顺序执行还是同时执行。
```

### ViewPropertyAnimator的使用：最简单的方法

View类的animate()方法，是在Android 3.1系统上新增的一个方法，其作用就是返回ViewPropertyAnimator的实例对象。

特点：
1、专门针对**View对象**动画而操作的类。
2、提供了更简洁的链式调用设置多个属性动画，这些动画可以同时进行的。
3、拥有更好的性能，多个属性动画是一次同时变化，只执行一次UI刷新（也就是只调用一次invalidate,而n个ObjectAnimator就会进行n次属性变化，就有n次invalidate）。
4、每个属性提供两种类型方法设置。
5、该类只能通过View的animate()获取其实例对象的引用

```java
//使用方式
btnShow.animate()
        .setDuration(2000)
        .translationX(300)
        .rotation(360)
        .start();

//常用设置
view.animate()//获取ViewPropertyAnimator对象
                        //动画持续时间
                        .setDuration(5000)

                        //透明度
                        .alpha(0)
                        .alphaBy(0)

                        //旋转
                        .rotation(360)
                        .rotationBy(360)
                        .rotationX(360)
                        .rotationXBy(360)
                        .rotationY(360)
                        .rotationYBy(360)

                        //缩放
                        .scaleX(1)
                        .scaleXBy(1)
                        .scaleY(1)
                        .scaleYBy(1)

                        //平移
                        .translationX(100)
                        .translationXBy(100)
                        .translationY(100)
                        .translationYBy(100)
                        .translationZ(100)
                        .translationZBy(100)

                        //更改在屏幕上的坐标
                        .x(10)
                        .xBy(10)
                        .y(10)
                        .yBy(10)
                        .z(10)
                        .zBy(10)

                        //插值器
                        .setInterpolator(new BounceInterpolator())//回弹
                        .setInterpolator(new AccelerateDecelerateInterpolator())//加速再减速
                        .setInterpolator(new AccelerateInterpolator())//加速
                        .setInterpolator(new DecelerateInterpolator())//减速
                        .setInterpolator(new LinearInterpolator())//线性

                        //动画延迟
                        .setStartDelay(1000)

                        //是否开启硬件加速
                        .withLayer()

                        //监听
                        .setListener(new Animator.AnimatorListener() {
                            @Override
                            public void onAnimationStart(Animator animation) {
                                Log.i("MainActivity", "run: onAnimationStart");
                            }

                            @Override
                            public void onAnimationEnd(Animator animation) {
                                Log.i("MainActivity", "run: onAnimationEnd");
                            }

                            @Override
                            public void onAnimationCancel(Animator animation) {
                                Log.i("MainActivity", "run: onAnimationCancel");
                            }

                            @Override
                            public void onAnimationRepeat(Animator animation) {
                                Log.i("MainActivity", "run: onAnimationRepeat");
                            }
                        })

                        .setUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                            @Override
                            public void onAnimationUpdate(ValueAnimator animation) {
                                Log.i("MainActivity", "run: onAnimationUpdate==");
                            }
                        })

                        .withEndAction(new Runnable() {
                            @Override
                            public void run() {
                                Log.i("MainActivity", "run: end");
                            }
                        })
                        .withStartAction(new Runnable() {
                            @Override
                            public void run() {
                                Log.i("MainActivity", "run: start");
                            }
                        })

                        .start();

//执行顺序
withStartAction（Runnable runnable）
→
onAnimationStart(Animator animation)
→
onAnimationUpdate（ValueAnimator animation）//动画过程中一直调用
→
onAnimationEnd(Animator animation)
→
withEndAction（Runnable runnable）
  
  
有By结尾和没By结尾的区别：
每个动画都有一个By的后缀的方法。加上By的意思是，继续动画这么多数值。不加By的意思是动画到这个数值。

有By：变化偏移

无By：变化到
```

常用类如下：

 ![img](https://api2.mubu.com/v3/document_image/cf2355eb-04a6-48f7-809e-f9c24cc42cae-4119341.jpg)

### 属性动画的listener

#### AnimationListener

主要监听属性动画的开始，结束，取消和重复

```java
public static interface AnimatorListener{    
	void onAnimationStart(Animator animation, boolean isReverse) {}        
	void onAnimationEnd(Animator animation, boolean isReverse) {}        
	void onAnimationCancel(Animator animation, boolean isReverse) {}        
	void onAnimationRepeat(Animator animation, boolean isReverse) {} 
	}
```



#### AnimationPauseListener

主要监听属性动画的暂停，恢复状态

```java
public static interface AnimatorPauseListener {    
	void onAnimationPause(Animator animation);        
	void onAnimationResume(Animator animation); 
}
```



#### AnimationUpdateListener

主要监听值的变化

```java
public static interface AnimatorUpdateListener {    
	void onAnimationUpdate(ValueAnimator animation); 
	}
```



## 传统动画VS属性动画

1.补间动画进行照片移动之后还可以在原位置进行点击，但属性动画不能，这说明属性动画才是真正的实现了view的移动而补间动画只是在不同的地方绘制了一个影子，实际对象还在原来的地方 

2.属性动画需要在activity退出时在onstop方法中将动画停止

## 插值器

**插值器**：根据时间流逝的百分比计算出当前属性值改变的百分比。

**估值器**：根据当前属性改变的百分比来计算改变后的属性值。

插值器的类型：

属性动画和补间动画的插值器是一样的

| 名称                             | 作用                     |
| -------------------------------- | ------------------------ |
| AccelerateDecelerateInterpolator | 先加速，后减速           |
| AccelerateInterpolator           | 一直加速                 |
| AnticipateInterpolator           | 迂回，加速               |
| OvershootInterpolator            | 加速超出，返回终点       |
| AnticipateOvershootInterpolator  | 迂回，加速超出，返回终点 |
| BounceInterpolator               | 弹簧效果                 |
| CycleInterpolator                | 正弦曲线                 |
| DecelerateInterpolator           | 一直减速                 |
| LinearInterpolator               | 线性速度                 |

自定义插值器：如果上面的插值器不符合要求可以自定义一个新的插值器。 自定义插值器时需要继承`BaseInterpolator`。 重写`public float getInterpolation(float input)`。

```java
//线性插值器的代码，参考
public class LinearInterpolator extends BaseInterpolator implements NativeInterpolatorFactory {

    public LinearInterpolator() {
    }

    public LinearInterpolator(Context context, AttributeSet attrs) {
    }

    public float getInterpolation(float input) {
        return input;
    }

    /** @hide */
    @Override
    public long createNativeInterpolator() {
        return NativeInterpolatorFactoryHelper.createLinearInterpolator();
    }
}

```

## 估值器

```java
// 在第3个参数中传入对应估值器类的对象
ObjectAnimator anim = ObjectAnimator.ofObject(button, "height", new Evaluator()，1，3);


```



系统内置的估值器

> IntEvaluator：以整型的形式从初始值 - 结束值 进行过渡
>  FloatEvaluator：以浮点型的形式从初始值 - 结束值 进行过渡
>  ArgbEvaluator：以Argb类型的形式从初始值 - 结束值 进行过渡

自定义估值器：

估值器是用于计算`ValueAnimator`中的数值变化。`ValueAnimator`的默认的估值器是线性的。 如果想要自定义估值器需要继承`TypeEvaluator`，以及重写`public T evaluate(float fraction, T startValue, T endValue);`。

```java
public interface TypeEvaluator {  
    // 参数说明
    // fraction：插值器getInterpolation()方法的返回值
    // startValue：动画的初始值
    // endValue：动画的结束值
    public Object evaluate(float fraction, Object startValue, Object endValue) {  
        ....// 估值器的计算逻辑
        return xxx；
        // 赋给动画属性具体数值，使用反射机制改变属性变化
    }  
}

// FloatEvaluator实现了TypeEvaluator接口
public class FloatEvaluator implements TypeEvaluator {  
    // 重写evaluate()
    // 参数说明
    // fraction：表示动画完成度（根据它来计算当前动画的值）
    // startValue、endValue：动画的初始值和结束值
    public Object evaluate(float fraction, Object startValue, Object endValue) {  
        float startFloat = ((Number) startValue).floatValue();  
        return startFloat + fraction * (((Number) endValue).floatValue() - startFloat);  
        // 初始值过渡到结束值的算法是：
        // 1. 用结束值减去初始值，算出它们之间的差值
        // 2. 用上述差值乘以fraction系数
        // 3. 再加上初始值，就得到当前动画的值
    }  
}
```

## KeyFrame

如果不喜欢插值器和估值器的操作，可以使用关键帧。`Keyframe`让我们可以指定**某个属性百分比**时对象的**属性值**。`Keyframe`同样支持`ofFloat`、`ofInt`、`ofObject`。使用关键帧，至少需要有两个关键帧，不然会奔溃。`PropertyValuesHolder`对象是用来保存动画过程所操作的属性和对应的值。

```java
tvText.setOnClickListener {
        val start = Keyframe.ofFloat(0f, 0f)
        val middle = Keyframe.ofFloat(0.3f, 400f)
        val end = Keyframe.ofFloat(1f, 700f)
        val holder=PropertyValuesHolder.ofKeyframe("translationX",start,middle,end)
        ObjectAnimator.ofPropertyValuesHolder(tvText,holder).apply {
            duration=2000
            start()
        }
    }
```



## 参考链接

学习指南：https://juejin.cn/post/6844903601190486030 

https://juejin.cn/post/6846687601118691341 

插值估值：https://www.jianshu.com/p/676bb3b4d71d

https://blog.csdn.net/sinat_31057219/article/details/77717187

源码分析：https://blog.csdn.net/y874961524/article/details/53984282

高级UI：https://juejin.cn/post/6844903774226481159