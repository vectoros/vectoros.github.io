---
title: NDK JNI中动态注册和静态注册，native和static native区别
tags:
  - Android
  - NDK
  - JNI
categories:
  - Android
  - NDK
  - JNI
abbrlink: '17976623'
date: 2019-10-17 20:11:59
keywords:
description:
---
动态注册和静态注册在JNI开发中是必然会遇到的问题。在最新的AndroidStudio中，默认使用的是静态注册，但是动态注册确实一种更加灵活的方式。

## 静态注册
在AndroidStudio中，创建项目时选择Native C++项目，会帮我们默认创建以下几个文件。

* MainActivity.java Java文件，包含JNI Java层接口
    ```java
    public class MainActivity extends AppCompatActivity {

    // Used to load the 'native-lib' library on application startup.
    static {
        System.loadLibrary("native-lib");
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Example of a call to a native method
        TextView tv = findViewById(R.id.sample_text);
        tv.setText(stringFromJNI());
    }

    /**
     * A native method that is implemented by the 'native-lib' native library,
     * which is packaged with this application.
     */
        public native String stringFromJNI();
    }
    ```

* native-lib.cpp C++文件，包含JNI函数的实现
    ```cpp
    #include <jni.h>
    #include <string>

    extern "C" JNIEXPORT jstring JNICALL
    Java_com_dten_jnimethod_MainActivity_stringFromJNI(
            JNIEnv *env,
            jobject /* this */) {
        std::string hello = "Hello from C++";
        return env->NewStringUTF(hello.c_str());
    }
    ```
* CMakeLists.txt CMake编译脚本
    ```makefile
    cmake_minimum_required(VERSION 3.4.1)
    
    add_library( # Sets the name of the library.
            native-lib
            SHARED
            native-lib.cpp)
    
    find_library( # Sets the name of the path variable.
            log-lib
            log)
    
    target_link_libraries( # Specifies the target library.
            native-lib
            ${log-lib})
    ```

## 动态注册
动态注册需要写的代码会多一些，我们可以参考Framework的很多JNI接口的实现。Java层的代码不需要变化，CMake也不需要修改（如果有引入其他头文件，需要在CMake里面增加对应的库）

* JNINativeMethod
动态注册过程中，需要使用结构体JNINativeMethod来记录java方法和jni函数的对应关系
    ```c
    typedef struct {
        const char* name; //Java方法名
        const char* signature; //方法的参数和返回值，使用字符串记录，格式形如`()V, (I)I`，括号内表示函数参数，括号右侧表示函数返回值
        void*       fnPtr; // 指向JNI函数的函数指针
    } JNINativeMethod;
    ```

* 数据类型映射
    - 基本数据类型
        Java类型 | Native类型 | 域描述符 | 补充
        -|-|-|-
        boolean | jboolean | Z |
        byte    | jbyte    | B |
        char    | jchar    | C |
        short   | jshort   | S |
        int     | jint     | I |
        long    | jlong    | J |
        float   | jfloat   | F |
        double  | jdouble  | D |
        void    | void     | V |

    - 数组引用类型
        Java类型 | Native类型 | 域描述符 | 补充
        -|-|-|-
        boolean[] | jbooleanArray | [Z |
        byte[]    | jbyteArray    | [B |
        char[]    | jcharArray    | [C |
        short[]   | jshortArray   | [S |
        int[]     | jintArray     | [I |
        long[]    | jlongArray    | [J |
        float[]   | jfloatArray   | [F |
        double[]  | jdoubleArray  | [D |
    - 对象引用类型
        Java类型 | Native类型 | 域描述符 | 补充
        -|-|-|-
        Class | jobject | Lcom.example.Class; | 以`"L"`开头，以`";"`结尾，内部类使用`"$"`连接，String除外
        String | jstring | Ljava/lang/String; | 唯一的例外
    - 对象数组引用类型
        Java类型 | Native类型 | 域描述符 | 补充
        -|-|-|-
        Class | jobject | [Lcom.example.Class; | 以`"[L"`开头，以`";"`结尾，内部类使用`"$"`连接，String除外
        String | jstring | [Ljava/lang/String; | 唯一的例外

* JNI函数默认参数

    - native方法默认参数
        普通的native方法，是Java类的成员方法，默认的参数有以下两个
        ```
        JNIEnv *env, jobject thiz
        ```
        * `JNIEnv *env`

            指代当前的java环境，可以利用JNIEnv操作Java层代码
        * `jobject thiz`

            指代JNI函数对应的java native方法对应的类的实例
    - static native方法默认参数
        static native方法，是Java类的static方法，默认的参数有以下两个
        ```
        JNIEnv *env, jclass classz
        ```
        * `JNIEnv *env`

            指代当前的java环境，可以利用JNIEnv操作Java层代码
        * `jclass classz`
        
            指代JNI函数对应的java static native方法对应的class对象

* native-lib.cpp
    ```cpp
    #include <jni.h>
    #include <string>

    // 定义java层对应的package名称
    #define JNIREG_CLASS "com/example/MainActivity"

    // Native方法实现
    static jstring stringFromJNI(JNIEnv *env, jclass classz)
    {
        std::string hello = "Hello from C++";
        return env->NewStringUTF(hello.c_str());
    }
    
    // Java层和Native层的方法签名
    static JNINativeMethod nativeMethods[] = {
        {"stringFromJNI", "()Ljava/lang/String;", (jstring *)stringFromJNI},
    };

    // 注册native方法
    int register_native_methods(JNIEnv *env)
    {
        jclass clazz = env->FindClass(JNIREG_CLASS);
        if (clazz == NULL)
        {
            ALOGE("Native registeration unable to find class '%s'",     JNIREG_CLASS);
            return JNI_FALSE;
        }

        if (env->RegisterNatives(clazz, nativeMethods, sizeof   (nativeMethods) / sizeof(nativeMethods[0])) < 0)
        {
            env->DeleteLocalRef(clazz);
            ALOGE("RegisterNatives failed for '%s'", JNIREG_CLASS);
            return JNI_FALSE;
        }

        return JNI_TRUE;
    }

    // 重写JNI_OnLoad方法
    extern "C" JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reserved)
    {
        JNIEnv *env = NULL;
        if (vm->GetEnv((void **)&env, JNI_VERSION_1_6) != JNI_OK)
        {
            return JNI_ERR;
        }

        if (JNI_TRUE != register_native_methods(env))
        {
            return JNI_ERR;
        }

        return JNI_VERSION_1_6;
    }


    ```


## 两种注册方法的对比

注册方法 | 优点 | 缺点
-|-|-
静态注册 | 支持自动生成的IDE，可以比较方便的编写代码 | 在没有IDE自动生成的情况下，编写不方便 |
静态注册 |  | 程序运行效率低，每次调用native函数时，需要根据函数名在JNI层搜索对应的本地函数，建立对应关系，比较耗时 |
动态注册 | Native层方法和Java层实现简单的代码分离，降低耦合度 | 需要手动实现注册方法 |
动态注册 | 大量的Native方法，注册起来更加方便 |  |


## native方法和static native方法的区别

在上面的默认参数一节中，我们已经提到了关于native方法和static native方法的一些区别

* 静态注册时，IDE帮我们生成对应的方法，我们可以省去编写函数签名的操作。
* 动态注册时，函数签名需要我们自己编写，就需要知道native方法和static native方法的一些区别和差异，才能使得我们的函数能够正常运行。

## 常用的JNI方法

* jclass获取jobject
```cpp
jmethod methodId = env->GetMethodID(classz, "<init>", "()V");
jobject jobj = env->NewObject(classz, methodId);
```
* jobject获取jclass
```cpp
jclass classz = env->GetObjectClass(thiz);
```
* 获取object的非静态字段
```cpp
// 获取jclass，静态方法则不需要这步
jclass classz = env->GetObjectClass(thiz);
// 获取field id，
jfieldID fid = env->GetFieldID(classz, "mName", "Ljava/lang/String;");
// 获取field 值
jstring mName = env->GetObjectField(thiz, fid);
// 设置值
std::string name = "myname";
env->SetObjectField(thiz, fid, env->NewStringUTF(name.c_str()))
```
* 获取class的静态字段
```cpp
// 获取jclass，静态方法则不需要这步
jclass classz = env->GetObjectClass(thiz);
// 获取field id，
jfieldID fid = env->GetStaticFieldID(classz, "sName", "Ljava/lang/String;");
// 获取field 值
jstring mName = env->GetStaticObjectField(thiz, fid);
// 设置值
std::string name = "myname";
env->SetStaticObjectField(thiz, fid, env->NewStringUTF(name.c_str()))
```
* 获取class的非静态方法
```cpp
// 获取jclass，静态方法则不需要这步
jclass classz = env->GetObjectClass(thiz);
// 获取method id，
jmethodID methodId = env->GetMethodID(classz, "getName", "()Ljava/lang/String;");
// 调用java方法
jstring name = env->CallObjectMethod(thiz, methodId)
```
* 获取class的静态方法
```cpp
// 获取jclass，静态方法则不需要这步
jclass classz = env->GetObjectClass(thiz);
// 获取method id，
jmethodID methodId = env->GetStaticMethodID(classz, "getCount", "()I");
// 调用java方法
jint count = env->CallStaticIntMethod(thiz, methodId)
```
* 访问构造方法
```cpp
// 获取jclass，静态方法则不需要这步
jclass classz = env->FindClass("java/util/Date");
jmethod methodId = env->GetMethodID(classz, "<init>", "()V");
// 实例化对象
jobject date = env->NewObject(classz, methodId);
// 获取调用方法ID
jmethod getTimeId = env->GetMethodID(classz, "getTime", "()J");
// 调用方法
jlong time = env->CallLongMethod(date, getTimeId);
std::out<<"current time: " << time <<endl;
```


