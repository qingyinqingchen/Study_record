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

书

主要经过onMeasure()、onLayout()和onDraw()

## onMeasure

View系统的绘制流程会从ViewRoot的perform Traversals()方法开始，内部调用measure并传入widthMeasureSpec和heightMeasureSpec两个参数，用于确定试图的宽度和高度的规格和大小。

而MeasureSpec由specSize和specMode组成，mode记录的是规格。

1. EXACTLY

表示父视图希望子视图的大小应该是由specSize的值来决定的，系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

2. AT_MOST

表示子视图最多只能是specSize中指定的大小，开发人员应该尽可能小得去设置这个视图，并且保证不会超过specSize。系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

3. UNSPECIFIED

表示开发人员可以将视图按照自己的意愿设置成任意的大小，没有任何限制。这种情况比较少见，不太会用到。

widthMeasureSpec和heightMeasureSpec这两个值都是由父视图经过计算后传递给子视图的，说明父视图会在一定程度上决定子视图的大小。

## 重绘过程

guolin blog 三

## 自定义View

书

# View的事件体系-点击事件

书