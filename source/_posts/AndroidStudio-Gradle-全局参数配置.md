---
title: AndroidStudio Gradle 全局参数配置
date: 2019-03-09 13:18:36
tags: [AndroidStudio,Gradle,Tutorial]
categories: 
- Android
---
## 先提出以下两个问题 
* **在开发Android应用时，如何将配置参数应用到所有的AndroidStudio项目中？**
* **有些敏感信息我们不希望包含在工程中，跟随代码提交，比如签名相关信息。**

## 解决方案
我们可以利用Gradle的全局配置文件，为我们所有的项目配置相关参数，解决以上两个问题。
* Windows平台在 `C:\Users\user\.gradle` 目录下
* Linux平台在 `/home/user/.gradle` 目录下
* Mac平台类似，找到`gradle`的目录即可

### 下面以配置签名信息为例：
1. 打开`gradle.properties`文件，在文件末尾增加如下信息：
```
KEYSTOREFILE=../../keystore/android.keystore
KEYALIAS=release
KEYPASSWORD=helloworld
STOREPASSWORD=helloworld
```

* **注意**
  - 这里的配置信息是没有`""`双引号的，有引号会导致异常。
  - `KEYSOTREFILE`使用的是相对目录，相对于`app/build.gradle`文件所在的目录。
  - 比如我这个文件就位于和工程文件夹相同级别的目录
```
MyProject/app/build.gradle
MyProject1/app/build.gradle
MyProject2/app/build.gradle
MyProject3/app/build.gradle
keystore/android.keystore
```

2. 在到对应的工程下面，打开app/build.gradle
  - 增加signingConfigs
  - 在buildTypes release里面增加signConfig配置
```
android {
    compileSdkVersion 28
    defaultConfig {
        applicationId "com.example.test"
        minSdkVersion 21
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    // 增加签名配置
    signingConfigs{
        releaseConfig{
            keyAlias KEYALIAS //引用刚才配置的参数
            keyPassword KEYPASSWORD
            storePassword STOREPASSWORD
            storeFile file(KEYSTOREFILE)
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.releaseConfig //引用配置文件
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

3. 这样配置后，即使代码提交了，也不会将签名信息提交，其他开发者把代码拉下来后，只需要到`gradle.properties`里面配置好对应的参数即可。其他全局配置也可以这么操作。