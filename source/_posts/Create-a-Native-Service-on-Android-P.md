---
title: Create a Native Service on Android P
tags:
  - Android
  - NDK
  - Service
categories:
  - Android
  - NDK
  - Service
abbrlink: 67bab5c2
date: 2019-10-09 10:53:44
---

在定制自己的Android嵌入式系统时候，有时我们需要创建一个Native Service用来和底层设备通信。
* Native Service其实是一个Linux守护进程，提供一些必要的服务。
* Android提供了Binder进程间通信机制，Native Service就需要遵循Android的规则来实现

## 整体框架
* Server (后台Service服务)
* Client (提供和Service通信的接口)
* JNI (持有一个Client端，封装对应的接口，用于和Service通信)
* Java (JNI层提供给Java层的接口)
* Selinux修改(Android 8以后)
* 预编译到系统中，供Framework调用(如何在framework.jar中调用我们自己生成的java接口)

## 代码实现框架
代码我们以一个LED开关接口为例，添加一个NativeService，用来控制一个独立的LED灯的开关。

* 创建组织代码目录
```bash
# root directory
mkdir LEDService
cd LEDService
# headers
mkdir include
# hardware interface
mkdir hal
# service implement
mkdir service
# client test
mkdir test
# JNI interface
mkdir jni
# java interface
mkdir java
# summary
ls -l
```
## 编译脚本
* 为什么先写编译脚本？
  - 编译脚本可以预先帮助我们组织代码
  - 我们可以提前知道需要编译输出什么

* 我们需要生成哪几个文件
  - libledservice.so 共享库，提供给Server端和Client端调用
  - libledservicemanager_jni.so JNI共享库，供Java接口调用
  - ledservicemanager.jar Jar包，提供给其他java层调用
  - ledservice.rc 服务启动的rc文件
  - com.vectoros.led.xml permission文件

* Android.mk

  我们在LEDService目录下创建一个Android.mk用于编译C/C++ so, 可执行文件以及rc，xml等资源文件的编译输出。
  ```makefile
  ################ LIB_LED_SERVICE ############
  include $(CLEAR_VARS)
  LOCAL_MODULE:= libledservice

  LOCAL_SRC_FILES:= \
      service/LEDService.cpp \
      service/ILEDService.cpp \
      service/LEDServiceManager.cpp

  LOCAL_C_INCLUDES += \
      $(LOCAL_PATH)/include

  LOCAL_SHARED_LIBRARIES := \
      libbinder \
      libutils \
      libcutils \
      liblog

  # Export include dir
  LOCAL_EXPORT_C_INCLUDE_DIRS := $(LOCAL_PATH)/include

  # LOCAL_CFLAGS += -mhard-float
  # LOCAL_LDFLAGS += -Wl,--no-warn-mismatch

  include $(BUILD_SHARED_LIBRARY)

  ################ LED_SERVICE ############
  include $(CLEAR_VARS)
  LOCAL_MODULE:= ledservice

  LOCAL_SRC_FILES:= \
      service/main_ledservice.cpp 

  LOCAL_SHARED_LIBRARIES := \
      libutils \
      liblog \
      libbinder \
      libledservice

  LOCAL_MODULE_CLASS := EXECUTABLES
  LOCAL_INIT_RC := service/ledservice.rc
  include $(BUILD_EXECUTABLE)

  ################ LED_SERVICE_MANAGER_JNI ############
  include $(CLEAR_VARS)
  LOCAL_MODULE:= libledservicemanager_jni

  LOCAL_SRC_FILES:= \
      jni/com_vectoros_ledservice_manager_jni.cpp 

  LOCAL_SHARED_LIBRARIES := \
      libutils  \
      liblog \
      libcutils \
      libledservice \
      libbinder

  LOCAL_CFLAGS += -Wno-unused-parameter
  LOCAL_LDFLAGS += -Wl,--no-warn-mismatch

  include $(BUILD_SHARED_LIBRARY)

  # Using Android.bp to build java library
  # ################# JAVA_LIBRARIES ############
  # include $(CLEAR_VARS)
  # LOCAL_MODULE:= ledservicemanager
  # LOCAL_SRC_FILES:= \
  #     $(call all-java-files-under,java)

  # LOCAL_MODULE_TAGS := optional
  # LOCAL_NO_STANDARD_LIBRARIES := true
  # LOCAL_DX_FLAGS := --core-library

  # include $(BUILD_JAVA_LIBRARY)

  # ################ PERMISION_FILES ############
  include $(CLEAR_VARS)
  LOCAL_MODULE:= com.vectoros.led.xml
  LOCAL_MODULE_TAGS := optional
  LOCAL_MODULE_CLASS:= ETC

  LOCAL_SRC_FILES:= java/$(LOCAL_MODULE)
  LOCAL_MODULE_PATH := $(TARGET_OUT_ETC)/permissions
  include $(BUILD_PREBUILT)

  ################ CMD_TEST_CLIENT ############
  include $(CLEAR_VARS)
  LOCAL_MODULE:= ledserviceclient

  LOCAL_SRC_FILES:= \
      test/main_ledservice_client.cpp

  LOCAL_SHARED_LIBRARIES := \
      libbinder \
      libutils  \
      libcutils \
      liblog \
      libledservice

  LOCAL_LDFLAGS += -Wl,--no-warn-mismatch
  LOCAL_MODULE_CLASS := EXECUTABLES
  include $(BUILD_EXECUTABLE)
  ```

* Andorid.bp

  我们在java目录下，创建一个Android.bp文件，用于生成jar包。至于为什么必须要用Android.bp，会留在集成到framework里面的时候讲。
  ```json
  java_library {
      name: "ledservicemanager",
      srcs: [
          "com/vectoros/led/*.java",
      ],

      hostdex: true,
      java_version: "1.7",
      no_framework_libs: true,
  }
  ```
* 总结

  通过以上的编译脚本，我们知道了自己需要写哪些文件，会生成什么样的文件，接下来就可以往下写实现部分的代码了。

## 头文件实现
* 主要有三个头文件
    - LEDService.h
    - LEDServiceManager.h
    - ILEDService.h

* ILEDService.h
```cpp
#ifndef ANDROID_ILED_NATIVE_SERVICE_H
#define ANDROID_ILED_NATIVE_SERVICE_H

#include <stdint.h>
#include <sys/types.h>


#include <utils/RefBase.h>
#include <utils/Singleton.h>
#include <utils/Errors.h>
#include <utils/String16.h>

#include <binder/IInterface.h>

namespace android
{

    class ILEDService : public IInterface
    {
        public:
            DECLARE_META_INTERFACE(LEDService);

            virtual int initHardware(void) = 0;
            virtual void releaseHardware(void) = 0;
            virtual void on(void) = 0;
            virtual void off(void) = 0;
    };

    class BnLEDService : public BnInterface<ILEDService>{
        public:
            virtual status_t onTransact(uint32_t code, const Parcel& data,
                 Parcel* reply, uint32_t flags = 0);
    };

    enum LED_SERVICE_TYPE{
        BASE_START,
        ON,
        OFF,
    };
    
} // namespace android
#endif //ANDROID_ILED_NATIVE_SERVICE_H
```

* LEDService.h
```cpp
#ifndef ANDROID_LED_NATIVE_SERVICE_H
#define ANDROID_LED_NATIVE_SERVICE_H

#include <map>
#include <stdint.h>
#include <sys/types.h>

#include <cutils/compiler.h>

#include <utils/Atomic.h>
#include <utils/Errors.h>
#include <utils/KeyedVector.h>
#include <utils/RefBase.h>
#include <utils/SortedVector.h>
#include <utils/threads.h>

#include <binder/BinderService.h>

#include "ILEDService.h"


namespace android {

    class LEDService : public BinderService<LEDService>, public BnLEDService
    {
    public:
        static char const* getServiceName() {
            return "LEDService";
        }

        LEDService();
        ~LEDService();

    private:
        virtual int initHardware(void);
        virtual void releaseHardware(void);
        virtual void on(void);
        virtual void off(void);
    };
}; // namespace android

#endif // ANDROID_LED_NATIVE_SERVICE_H
```

* LEDServiceManager.h
```cpp
#ifndef ANDROID_LED_NATIVE_MANAGER_H
#define ANDROID_LED_NATIVE_MANAGER_H

#include <stdint.h>
#include <sys/types.h>

#include <binder/IBinder.h>

#include <utils/RefBase.h>
#include <utils/Singleton.h>
#include <utils/threads.h>

#include "ILEDService.h"

namespace android
{
    class LEDServiceManager : public Singleton<LEDServiceManager>
    {
    private:
        /* data */
        bool isDied;
        void ledServiceDied();

        mutable sp<ILEDService> mLEDService;
        mutable sp<IBinder::DeathRecipient> mDeathObserver;

    public:
        LEDServiceManager(/* args */);
        ~LEDServiceManager();

        int initHardware(void);
        void releaseHardware(void);
        void on(void);
        void off(void);

        status_t assertState();
        bool checkService() const;
        void resetServiceStatus();
    };
} // namespace android
#endif //ANDROID_LED_NATIVE_MANAGER_H
```

## 客户端和服务端实现

* ILEDService.cpp
```cpp
#include <stdio.h>
#include <stdint.h>
#include <malloc.h>
#include <sys/types.h>

#include <binder/Parcel.h>
#include <binder/IMemory.h>
#include <binder/IPCThreadState.h>
#include <binder/IServiceManager.h>
#include "ILEDService.h"

#ifdef __ANDROID__
#define LOG_TAG "ILEDService"
#if ANDROID_API_LEVEL >= 26
#include <log/log.h>
#else
#include <cutils/log.h>
#endif
#endif

namespace android
{
    enum{
        INIT_HARDWARE = IBinder::FIRST_CALL_TRANSACTION,
        RELEASE_HARDWARE,
        ON,
        OFF,
    };


    class BpLEDService : public BpInterface<ILEDService>
    {
        public:
            BpLEDService(const sp<IBinder>& impl) : BpInterface<ILEDService>(impl)
            {

            }

            int initHardware(void)
            {
                Parcel data, reply;
                data.writeInterfaceToken(ILEDService::getInterfaceDescriptor());
                remote()->transact(INIT_HARDWARE, data, &reply);
                return (int)reply.readInt32();
            }

            void releaseHardware(void)
            {
                Parcel data, reply;
                data.writeInterfaceToken(ILEDService::getInterfaceDescriptor());
                remote()->transact(RELEASE_HARDWARE, data, &reply);
            }

            int on(void)
            {
                Parcel data, reply;
                data.writeInterfaceToken(ILEDService::getInterfaceDescriptor());
                remote()->transact(ON, data, &reply);
                return (int) reply.readInt32();
            }

            int off(void)
            {
                Parcel data, reply;
                data.writeInterfaceToken(ILEDService::getInterfaceDescriptor());
                remote()->transact(OFF, data, &reply);
                return (int) reply.readInt32();
            }
    };

    IMPLEMENT_META_INTERFACE(LEDService, "android.vectoros.LEDService");

    status_t BnLEDService::onTransact(uint32_t code, const Parcel& data, Parcel* reply, uint32_t flags)
    {
        char *buff;
        int len, retval;
        status_t status;
    
        switch(code) {
            case INIT_HARDWARE:{
                CHECK_INTERFACE(ILEDService, data, reply);
                retval = initHardware();
                reply->writeInt32(retval); 
                return NO_ERROR;
            } break;
                
    
            case RELEASE_HARDWARE:{
                CHECK_INTERFACE(ILEDService, data, reply); 
                releaseHardware();
                return NO_ERROR;
            } break;

            case ON:{
                CHECK_INTERFACE(ILEDService, data, reply);
                on();
                return NO_ERROR;
            } break;
                
            case OFF:{
                CHECK_INTERFACE(ILEDService, data, reply);
                off();
                return NO_ERROR;
            } break;

            default:
              return BBinder::onTransact(code, data, reply, flags);
        }
    }
} // namespace android
```
* LEDService.cpp
```cpp
#include <stdint.h>
#include <stdio.h>
#include <math.h>
#include <sys/types.h>

#include <utils/Errors.h>
#include <utils/RefBase.h>
#include <utils/Singleton.h>
#include <utils/String16.h>

#include <binder/BinderService.h>
#include <binder/IServiceManager.h>

#include "LEDService.h"

#define BUFFER_SIZE 512
#ifdef __ANDROID__
#define LOG_TAG "LEDService"
#if ANDROID_API_LEVEL >= 26
#include <log/log.h>
#else
#include <cutils/log.h>
#endif
#endif

namespace android {

LEDService::LEDService()
{
    ALOGI("LEDService()"); 
}

LEDService::~LEDService(){
    ALOGI("~LEDService()");
}

int LEDService::initHardware(void)
{
    ALOGI("initHardware(), check camera env.");
}

void LEDService::releaseHardware(void)
{
    ALOGI("releaseHardware()");
}

int LEDService::on(void)
{
    ALOGI("on()");
    return 0;
}

int LEDService::off(void)
{
    ALOGI("off()");
    return 0;
}

}; // namespace android

```
* LEDServiceManager.cpp
```cpp
#include <stdint.h>
#include <sys/types.h>

#include <utils/Errors.h>
#include <utils/RefBase.h>
#include <utils/Singleton.h>

#ifdef __ANDROID__
#define LOG_TAG "LEDServiceManager"
#if ANDROID_API_LEVEL >= 26
#include <log/log.h>
#else
#include <cutils/log.h>
#endif
#endif

#include <binder/IBinder.h>
#include <binder/IServiceManager.h>

#include "ILEDService.h"
#include "LEDServiceManager.h"

namespace android
{
    LEDServiceManager::LEDServiceManager() : isDied(false)
    {

    }

    LEDServiceManager::~LEDServiceManager()
    {

    }

    void LEDServiceManager::LEDServiceDied()
    {
        isDied = true;
        mLEDService.clear();
    }

    status_t LEDServiceManager::assertState() {
        if (mLEDService == NULL) {
            // try for one second
            const String16 name("LEDService");
            for (int i=0 ; i<4 ; i++) {
                status_t err = getService(name, &mLEDService);
                if (err == NAME_NOT_FOUND) {
                    usleep(250000);
                    continue;
                }
                if (err != NO_ERROR) {
                    ALOGE("LEDService not found");
                    return err;
                }
                break;
            }
            
            initLEDServiceNative();
    
    
            class DeathObserver : public IBinder::DeathRecipient {
                LEDServiceManager& mLEDServiceManager;
                virtual void binderDied(const wp<IBinder>& who) {
                    ALOGW("LEDService died [%p]", who.unsafe_get());
                    mLEDServiceManager.LEDServiceDied();
                }
            public:
                DeathObserver(LEDServiceManager& mgr) : mLEDServiceManager(mgr) { }
            };
    
            mDeathObserver = new DeathObserver(*const_cast<LEDServiceManager *>(this));
            mLEDService->asBinder(mLEDService)->linkToDeath(mDeathObserver);
        }
    
        return NO_ERROR;
    }
    
    bool LEDServiceManager::checkService() const 
    {
        return isDied? true:false;
    }
    
    void LEDServiceManager::resetServiceStatus() 
    {
        isDied = false; 
    }
    
    int LEDServiceManager::initHardware(void)
    {
        return mLEDService->initHardware(); 
    }
    
    void LEDServiceManager::releaseHardware(void)
    {
        mLEDService->releaseHardware();
    }

    int LEDServiceManager::on()
    {
        return mLEDService->on();
    }
    
    int LEDServiceManager::off()
    {
        return mLEDService->off();
    }

} // namespace android

```
* main_LED_service.cpp
```cpp
#include <binder/BinderService.h>
#include <LEDService.h>
#include <binder/IPCThreadState.h>
#include <binder/ProcessState.h>
#include <binder/IServiceManager.h>

#include "ILEDService.h"

using namespace android;

int main(int argc, char** argv) {
#if 1
    LEDService::publishAndJoinThreadPool(true);
    // Like the SurfaceFlinger, limit the number of binder threads to 4.
    ProcessState::self()->setThreadPoolMaxThreadCount(4);
#else
    
    sp<ProcessState> proc(ProcessState::self());

    sp<IServiceManager> sm = defaultServiceManager();

    sm->addService(String16("LEDService"), new LEDService());

    ProcessState::self()->startThreadPool();
    ProcessState::self()->giveThreadPoolName();
    IPCThreadState::self()->joinThreadPool();
    ProcessState::self()->setThreadPoolMaxThreadCount(4);
#endif
    return 0;
}
```

## 客户端服务端测试程序

## JNI接口

## Java接口
