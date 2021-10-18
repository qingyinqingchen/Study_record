# Android-JNI

## JNI介绍

定义：`Java Native Interface`，即 `Java`本地接口

作用： 使得Java与 本地其他类型语言（如C、C++）交互即在Java代码里调用 C、C++等语言的代码 或 C、C++代码调用 Java 代码

ps:JNI是属于java范畴，与Android无直接关系

为什么要使用JNI：java属于跨平台的，与本地代码交互的能力非常弱，但是实际使用过程中需要和本地代码进行交互，因此采用JNI特性增强与本地代码的交互能力

### 实现步骤：

1. 在`Java`中声明`Native`方法（即需要调用的本地方法）
2. 编译上述 `Java`源文件javac（得到 `.class`文件）
3. 通过 `javah` 命令导出`JNI`的头文件（`.h`文件）
4. 使用 `Java`需要交互的本地代码 实现在 `Java`中声明的`Native`方法

> 如 `Java` 需要与 `C++` 交互，那么就用`C++`实现 `Java`的`Native`方法

1. 编译`.so`库文件
2. 通过`Java`命令执行 `Java`程序，最终实现`Java`调用本地代码

## NDK介绍

- 定义：`Native Development Kit`，是 `Android`的一个工具开发包，NDK是属于 Android的，与java并无直接关系
- 快速开发`C`、 `C++`的动态库，并自动将`so`和应用一起打包成 `APK`

> 即可通过 `NDK`在 `Android`中 使用 `JNI`与本地代码（如C、C++）交互

### 实现步骤：

1. 配置 `Android NDK`环境（Android studio 2.2以上都自带，后面的都不需要）
2. 创建 `Android` 项目，并与 `NDK`进行关联
3. 在 `Android` 项目中声明所需要调用的 `Native`方法
4. 使用 `Android`需要交互的本地代码 实现在`Android`中声明的`Native`方法

> 比如 `Android` 需要与 `C++` 交互，那么就用`C++` 实现 `Java`的`Native`方法

1. 通过 `ndk - bulid` 命令编译产生`.so`库文件
2. 编译 `Android Studio` 工程，从而实现 `Android` 调用本地代码

## JNI和NDK的关系

![示意图](https://tva1.sinaimg.cn/large/008i3skNgy1gvcbu4a12tj60yg0g0gmi02.jpg)

## JNI的具体使用



Android studio 2.2以上

步骤一：创建工程时配置NDK，可以直接创建NDK C++项目

步骤二：Android studio会自动生成c++文件并设置好调用的代码，只需要自行更改需要的代码

# JNI基础

## JNI命名规则：

JNI方法名规范 :

```
返回值 + Java前缀 + 全路径类名 + 方法名 + 参数① JNIEnv + 参数② jobject + 其它参数
```

简单的一个例子，返回一个字符串

```java
extern "C" 
JNIEXPORT jstring JNICALL Java_com_yeungeek_jnisample_NativeHelper_stringFromJNI(JNIEnv *env, jclass jclass1) {
    LOGD("##### from c");

    return env->NewStringUTF("Hello JNI");
}
```

- 返回值：jstring

- 全路径类名：com_yeungeek_jnisample_NativeHelper

- 方法名：stringFromJNI

- 如果本地代码是`C++`（`.cpp`或者`.cc`），要使用`extern "C" { }`把本地方法括进去

- `JNIEXPORT jstring JNICALL`中的`JNIEXPORT` 和 `JNICALL`不能省

- 关于方法名

  ```
  Java_scut_carson_1ho_ndk_1demo_MainActivity_getFromJNI
  ```

  1. 格式 = `Java _包名 _ 类名_Java需要调用的方法名`
  2. `Java`必须大写
  3. 对于包名，包名里的`.`要改成`_`，`_`要改成`_1`

  > 如包名是：`scut.carson_ho.ndk_demo`，则需要改成`scut_carson_1ho_ndk_1demo`

## 数据类型

### 基本数据类型

| Signature | Java    | Native   |
| --------- | ------- | -------- |
| B         | byte    | jbyte    |
| C         | char    | jchar    |
| D         | double  | jdouble  |
| F         | float   | jfloat   |
| I         | int     | jint     |
| S         | short   | jshort   |
| J         | long    | jlong    |
| Z         | boolean | jboolean |
| V         | void    | jvoid    |

### 引用数据类型

| Signature             | Java      | Native        |
| --------------------- | --------- | ------------- |
| L+classname +;        | Object    | jobject       |
| Ljava/lang/String;    | String    | jstring       |
| [L+classname +;       | Object[]  | jobjectArray  |
| Ljava.lang.Class;     | Class     | jclass        |
| Ljava.lang.Throwable; | Throwable | jthrowable    |
| [B                    | byte[]    | jbyteArray    |
| [C                    | char[]    | jcharArray    |
| [D                    | double[]  | jdoubleArray  |
| [F                    | float[]   | jfloatArray   |
| [I                    | int[]     | jintArray     |
| [S                    | short[]   | jshortArray   |
| [J                    | long[]    | jlongArray    |
| [Z                    | boolean[] | jbooleanArray |






