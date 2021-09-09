# LayoutInflater原理学习

用于加载布局。setContentView内部也是使用LayoutInflater,在使用context的时候会自动存在，不能直接获取实例。

## 基本用法

1）获取实例：

```Java
LayoutInflater layoutInflater = LayoutInflater.from(context);

LayoutInflater layoutInflater = (LayoutInflater)context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
//第一种其实就是第二种的简单写法。
```



2）调用inflate()加载布局

```Java
layoutInflater.inflate(resourceId,root);
//返回的是View，如果需要使用的话要调用layout的addView()方法。
//inflate()方法一般接收两个参数，第一个参数就是要加载的布局id，第二个参数是指给该布局的外部再嵌套一层父布局，如果不需要就直接传null。这样就成功成功创建了一个布局的实例，之后再将它添加到指定的位置就可以显示出来了。

```



## 源码分析

```Java
//老版本代码
public View inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot) {
    synchronized (mConstructorArgs) {
        final AttributeSet attrs = Xml.asAttributeSet(parser);//相当于获取对应的XML文件
        mConstructorArgs[0] = mContext;
        View result = root;//可有可无，有的话需要看是否使用了attachToRoot
        try {
            int type;
            while ((type = parser.next()) != XmlPullParser.START_TAG &&
                    type != XmlPullParser.END_DOCUMENT) {
            }
            if (type != XmlPullParser.START_TAG) {
                throw new InflateException(parser.getPositionDescription()
                        + ": No start tag found!");
            }
            final String name = parser.getName();
            if (TAG_MERGE.equals(name)) {
                if (root == null || !attachToRoot) {
                    throw new InflateException("merge can be used only with a valid "
                            + "ViewGroup root and attachToRoot=true");
                }
                rInflate(parser, root, attrs);
            } else {
                View temp = createViewFromTag(name, attrs);
                ViewGroup.LayoutParams params = null;
                if (root != null) {
                    params = root.generateLayoutParams(attrs);
                    if (!attachToRoot) {
                        temp.setLayoutParams(params);
                    }
                }
                rInflate(parser, temp, attrs);
                if (root != null && attachToRoot) {
                    root.addView(temp, params);
                }
                if (root == null || !attachToRoot) {
                    result = temp;
                }
            }
        } catch (XmlPullParserException e) {
            InflateException ex = new InflateException(e.getMessage());
            ex.initCause(e);
            throw ex;
        } catch (IOException e) {
            InflateException ex = new InflateException(
                    parser.getPositionDescription()
                    + ": " + e.getMessage());
            ex.initCause(e);
            throw ex;
        }
        return result;
    }
}
//Android-30
public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
        synchronized (mConstructorArgs) {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");

            final Context inflaterContext = mContext;
            final AttributeSet attrs = Xml.asAttributeSet(parser);//相当于获取对应的XML文件
            Context lastContext = (Context) mConstructorArgs[0];
            mConstructorArgs[0] = inflaterContext;
            View result = root;//可有可无，有的话需要看是否使用了attachToRoot

            try {
                advanceToRootNode(parser);
                final String name = parser.getName();

                if (DEBUG) {
                    System.out.println("**************************");
                    System.out.println("Creating root view: "
                            + name);
                    System.out.println("**************************");
                }

                if (TAG_MERGE.equals(name)) {
                    if (root == null || !attachToRoot) {
                        throw new InflateException("<merge /> can be used only with a valid "
                                + "ViewGroup root and attachToRoot=true");
                    }

                    rInflate(parser, root, inflaterContext, attrs, false);
                } else {
                    // Temp is the root view that was found in the xml,传入节点名和参数，根据节点创建View对象，在这个方法内部还会调用createView方法，然后使用反射的方式创建出View的实例。
                    final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                    ViewGroup.LayoutParams params = null;

                    if (root != null) {
                        if (DEBUG) {
                            System.out.println("Creating params from root: " +
                                    root);
                        }
                        // Create layout params that match root, if supplied
                        params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
                            // Set the layout params for temp if we are not
                            // attaching. (If we are, we use addView, below)
                            temp.setLayoutParams(params);
                        }
                    }

                    if (DEBUG) {
                        System.out.println("-----> start inflating children");
                    }

                    // Inflate all children under temp against its context.
                  //循环遍历根布局下的子元素，其实就是递归
                    rInflateChildren(parser, temp, attrs, true);

                    if (DEBUG) {
                        System.out.println("-----> done inflating children");
                    }

                    // We are supposed to attach all the views we found (int temp)
                    // to root. Do that now.
                    if (root != null && attachToRoot) {
                        root.addView(temp, params);
                    }

                    // Decide whether to return the root that was passed in or the
                    // top view found in xml.
                    if (root == null || !attachToRoot) {
                        result = temp;
                    }
                }

            } catch (XmlPullParserException e) {
                final InflateException ie = new InflateException(e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } catch (Exception e) {
                final InflateException ie = new InflateException(
                        getParserStateDescription(inflaterContext, attrs)
                        + ": " + e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } finally {
                // Don't retain static reference on context.
                mConstructorArgs[0] = lastContext;
                mConstructorArgs[1] = null;

                Trace.traceEnd(Trace.TRACE_TAG_VIEW);
            }

            return result;
        }
    }
```

1. 如果root为null，attachToRoot将失去作用，设置任何值都没有意义。

2. 如果root不为null，attachToRoot设为true，则会给加载的布局文件的指定一个父布局，即root。

3. 如果root不为null，attachToRoot设为false，则会将**布局文件最外层的所有layout属性**进行设置，当该view被添加到父view当中时，这些layout属性会自动生效。

4. 在不设置attachToRoot参数的情况下，如果root不为null，attachToRoot参数默认为true。

  

  这个和setContentView里面的不一样，这个函数中会自动添加一层layout。虽然setContentView()方法大家都会用，但实际上Android界面显示的原理要比我们所看到的东西复杂得多。任何一个Activity中显示的界面其实主要都由两部分组成，**标题栏和内容布局**。标题栏就是在很多界面顶部显示的那部分内容，比如刚刚我们的那个例子当中就有标题栏，可以在代码中控制让它是否显示。而内容布局就是一个FrameLayout，这个布局的id叫作content，我们调用setContentView()方法时所传入的布局其实就是放到这个FrameLayout中的，这也是为什么这个方法名叫作setContentView()，而不是叫setView()。

  ![](https://tva1.sinaimg.cn/large/008i3skNgy1gu6z8c7975j60fc0cijrm02.jpg)

  ## adroid中提供的XML解析器

  |                              | 介绍                                                         | 优点                                                         | 缺点                                                         |
  | ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | SAX（simple API for XML）    | 一种XML解析的替代方法。相比于DOM，SAX是一种速度更快，更有效的方法。它逐行扫描文档，一边扫描一边解析。而且相比于DOM，SAX可以在解析文档的任意时刻停止解析，但任何事物都有其相反的一面，对于SAX来说就是操作复杂。SAX是事件驱动型的，SAX的工作原理简单地说就是对文档进行顺序扫描，当扫描到文档（document）开始与结束、元素（element）开始与结束、文档（document）结束等地方时通知事件处理函数，由事件处理函数做相应动作，然后继续同样的扫描，直至文档结束。 | 解析速度快，占用内存少，SAX解析器非常适合在Android移动设备中使用。 | **只能读取XML**，无法修改。无法知道当前解析标签的上层标签及嵌套结构，**仅仅知道当前解析的标签的名字和属性，要知道其他信息需要自己实现。无法随机访问某个标签。** |
  | DOM（Document Object Model） | 是W3C制定的一套规范标准，即规定了解析文件的接口。各种语言可以按照DOM规范去实现这些接口，给出解析文件的解析器。DOM是基于树形结构的的节点或信息片段的集合，允许开发人员使用DOM API遍历XML树、检索所需数据。分析该结构通常需要加载整个文档和构造树形结构，然后才可以检索和更新节点信息。 | 易用性强，使用简单，DOM在内存中以树形结构存放，因此检索和更新效率会更高，适于修改XML结构 | 效率低，解析速度慢，内存占用量过高，不适合大型文件           |
  | PULL解析器                   | 是一个开源的java项目，既可以用在Android上，也可以用在java EE上，运行方式与SAX相似，都是基于事件的模式。不同的是，在PULL解析过程中，我们需要自己获取产生的事件然后做相应的操作，而不像SAX那样由**处理器触发一种事件**的方法，执行我们的代码。 | 小巧轻便，解析速度快，简单易用，非常适合在Android移动设备中使用，Android系统内部在解析各种XML时也是用PULL解析器 | 同SAX。但比SAX更容易实现。                                   |





# View绘制流程解析

主要经过onMeasure()、onLayout()和onDraw()

## 初识ViewRoot和DecorView

ViewRoot对应于ViewRootImpl类，它是连接WindowManager和DecorView的纽带，三个流程都是通过ViewRoot完成的。当Activity对象创建完毕后，会将DecorView添加到Window中，同时创建ViewRootImpl对象，并将ViewRootImpl和DecorView进行关联。

**Measure**

View系统的**绘制流程**会从**ViewRoot**的performTraversals()方法开始，内部调用measure并传入widthMeasureSpec和heightMeasureSpec两个参数，用于确定试图的宽度和高度的规格和大小。performTraversals()方法如下。Measure方法是会传递下去的，跟另外两个相同，只是draw是通过dispatchDraw传递，但没有什么不同。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gu7u1k708uj60ym0n2wga02.jpg)



**DecorView是顶级View，一般分为两栏，一个标题栏，一个内容栏，setContentView则是把内容放在内容栏**。

而MeasureSpec由specSize和specMode组成，mode记录的是规格。

1. EXACTLY

表示父视图希望子视图的大小应该是由specSize的值来决定的，系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

2. AT_MOST

表示子视图最多只能是specSize中指定的大小，开发人员应该尽可能小得去设置这个视图，并且保证不会超过specSize。系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

3. UNSPECIFIED

表示开发人员可以将视图按照自己的意愿设置成任意的大小，没有任何限制。这种情况比较少见，不太会用到。

widthMeasureSpec和heightMeasureSpec这两个值都是由**父视图**经过计算后传递给子视图的，说明父视图会在一定程度上决定子视图的大小。

LayoutParams：在View测量的时候，系统会将LayoutParas在父容器的约束下转换成对应的MeasureSpec。对于DecorView，其MesureSpec由窗口的尺寸和自身的LayoutParas约定，对于普通的View，由父容器的MeasureSpec和自身的LayoutParams来共同决定。下图是普通Vie。w如果根据父容器提供的SpecMode确定值

![](https://tva1.sinaimg.cn/large/008i3skNgy1gu7uxtf62wj61720dqq4i02.jpg)

ViewGroup并没有实现Measure过程，需要他的子类自行实现。

**layout**

layout首先会通过setFrame方法来设定View的四个顶点的位置，即初始化。接着调用onLayout方法，确定子元素的位置，View和ViewGroup都没有实现具体的onLayout方法。

**draw**

绘制流程如下

![](https://tva1.sinaimg.cn/large/008i3skNgy1gu7x96lmgtj60sq09saai02.jpg)

## 自定义View

### 自定义View的分类：标准不唯一，参考书分四类

1.继承View重写onDraw

​		主要用于实现一些不规则的效果，需要实现者自己支持wrap_content，需要自己处理padding。支持wrap_content如果没有特殊要求就是固定大小，具体的情况具体分析，可以参考textView等组件的处理方式。对于padding的处理，可以在onDraw的时候对于绘制的大小添加padding参数。

2.继承ViewGroup派生特殊的layout

​		实现自定义的布局，需要合适的处理ViewGroup的**测量、布局**这两个过程，同时处理子元素的测量和布局过程。

3.继承特定的View（TextView）

​		扩展View的功能，不需要自己写wrap_content和padding

4.继承特定的ViewGroup（linearlayout）

​		跟2实现的效果差不多，只是方法2更接近View的底层



### 自定义属性

1.在value目录下创建自定义属性的XML，在文件中自定义属性集合，在集合内可以定义多自定义属性

2.在View的构造方法中解析自定义属性的值并做相应处理

3.在布局文件中使用自定义属性

在使用的时候需要在布局文件中添加schemas声明：xmlns:app=http://schems.android.com/apk/res-auto



### 自定义View的注意事项

1. 让View支持wrap_content

   会达不到效果

2. 如果有必要，让View支持padding

   直接继承View的空间如果不在draw中处理padding，则无法起作用。继承自ViewGroup的控件需要在onMeasure和onLayout中考虑padding和子元素margin对其造成的影响，不然后者会失效。

3. 尽量不要在View中使用Handler，没必要

   View提供了Post系列方法。

4. View中要是有线程或则动画，需要及时停止，参考View。onDtachedFromWindow

5. View带有滑动嵌套的时候需要处理好滑动冲突

# View的事件体系-点击事件

### MotionEvent和TouchSlop

MotionEvent典型的事件类型有：

ACTION_DOWN：**手指刚接触屏幕，按下去的那一瞬间产生该事件**

ACTION_MOVE：**手指在屏幕上移动时候产生该事件**

ACTION_UP：**手指从屏幕上松开的瞬间产生该事件**

一个动作会触发一系列点击事件。getRawX/getRawY返回的是相对于手机屏幕左上角的x和y坐标。

TouchSlop是系统所能识别出的被认为是滑动的最小的距离。在不同的设备上这个值是不同的：ViewConfiguration.get(getContext()).getScaledTouchSlop()获取。

### VelocityTracker、GestureDetector和Scroller

1.VelocityTracker

速度追踪，包括水平速度和竖直方向的速度

2.GestureDector

3.Scroller

弹性滑动对象，用于实现View的弹性滑动，实现有过渡效果的滑动

## View的事件分发机制

### 点击事件的传递规则

​	主要是是对MotionEvent的传递，主要由三个方法完成

1.dispatchTouchEvent:用来进行事件的分发，如果事件能够传递给当前的View，那么此方法一定会被调用。返回结果受当前View的onTouchEvent和下级的View的dispatchTouchEvent方法的影响，表示是否消耗当前事件。如果一个MotionEvent传递给了View，那么dispatchTouchEvent方法一定会被调用！返回值：表示是否消费了当前事件。可能是View本身的onTouchEvent方法消费，也可能是子View的dispatchTouchEvent方法中消费。返回true表示事件被消费，本次的事件终止。返回false表示View以及子View均没有消费事件，将调用父View的onTouchEvent方法

2.onInterceptTouchEvent：在上述方法内部调用，用来判断是否拦截某个事件，如果拦截了，在同一个时间序列中不会再调用。View没有这个方法。

3.onTouchEvent：真正对MotionEvent进行处理或者说消费的方法。在dispatchTouchEvent进行调用。返回值：返回true表示事件被消费，本次的事件终止。返回false表示事件没有被消费，将调用父View的onTouchEvent方法在第一个方法中被调用，用来处理点击事件。可以使用不同的listener判断

当一个根ViewGroup产生点击事件，首先调用的是1，如果2表示拦截，那么会调用3，如果2不拦截，则会传递给子元素处理。

### 图示

假设有如下的view结构

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gu99o1767ej60pu0g3abe02.jpg)

事件分发的过程如下：

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gu99nkbcufj60i50z5dhd02.jpg)

对于View（注意！ViewGroup也是View）而言，如果设置了onTouchListener，那么OnTouchListener方法中的onTouch方法会被回调。onTouch方法返回true，则onTouchEvent方法不会被调用（onClick事件是在onTouchEvent中调用）所以三者优先级是onTouch->onTouchEvent->onClick

在Activity产生事件之后传递顺序是：Activity->Window->View,然后再由View进行分发。

## View的滑动冲突

### 常见的滑动冲突场景：

1.外部滑动方向和内部滑动方向不一致

ViewPager不会出现这种情况 ，如果是ScrollView就需要处理

2.外部滑动方向和内部滑动方向一致

3.上面两种的嵌套

解决方法：

外部拦截法：重写父容器的onInterceptTouchEvent

```java
 public boolean onInterceptTouchEvent(MotionEvent event) {
        boolean intercepted = false;
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN: {
                intercepted = false;
                break;
            }
            case MotionEvent.ACTION_MOVE: {
                if (满足父容器的拦截要求) {
                    intercepted = true;
                } else {
                    intercepted = false;
                }
                break;
            }
            case MotionEvent.ACTION_UP: {
                intercepted = false;
                break;
            }
            default:
                break;
        }
        mLastXIntercept = x;
        mLastYIntercept = y;
        return intercepted;
    }
```

内部拦截法：所有父View都不拦截任何事件，都传递给子View处理，需要子View使用requestDisallowIntercepttouchEvent才能正常工作。

```java
//子View的方法
public boolean dispatchTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN: {
                parent.requestDisallowInterceptTouchEvent(true);
                break;
            }
            case MotionEvent.ACTION_MOVE: {
                int deltaX = x - mLastX;
                int deltaY = y - mLastY;
                if (父容器需要此类点击事件) {
                    parent.requestDisallowInterceptTouchEvent(false);
                }
                break;
            }
            case MotionEvent.ACTION_UP: {
                break;
            }
            default:
                break;
        }

        mLastX = x;
        mLastY = y;
        return super.dispatchTouchEvent(event);
    }
//父VIew重写
 public boolean onInterceptTouchEvent(MotionEvent event) {

        int action = event.getAction();
        if (action == MotionEvent.ACTION_DOWN) {
            return false;
        } else {
            return true;
        }
    }
//在action_down的地方处理逻辑
//requestDisallowInterceptTouchEvent 方法其实就是在父View的如下方法中走if的第一个分支，让父VIew去处理
final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
```



