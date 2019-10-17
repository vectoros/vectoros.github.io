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

* native-lib.cpp
    ```cpp
    #include <jni.h>
    #include <string>

    // 定义java层对应的package名称
    #define JNIREG_CLASS "com/example/MainActivity"

    // Native方法实现
    static jstring stringFromJNI(JNIEnv *env, jobject obj)
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

## native方法

## static native方法
