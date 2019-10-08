---
title: Android Create Platform Keystore for AndroidStudio
tags:
  - Android
  - Keystore
  - AndroidStudio
categories:
  - Andorid
abbrlink: a94712d2
date: 2019-03-29 20:37:26
---

有时候我们需要在AndroidStudio中开发Android系统应用（带系统签名），为了方便开发，我们一般会生成一个debug.keystore，以下是操作步骤

### 找到系统源码的key
```
cd build/target/product/security/
```

### 生成pem文件
```
openssl pkcs8 -in platform.pk8 -inform DER -outform PEM -out shared.priv.pem -nocrypt
```

### 生成pk12文件
```
openssl pkcs12 -export -in platform.x509.pem -inkey shared.priv.pem -out shared.pk12 -name youralias
```

### 设置密码
* 默认密码是android
* 可以自己修改

### 生成keystore
```
keytool -importkeystore -deststorepass android -destkeypass android -destkeystore debug.keystore -srckeystore shared.pk12 -srcstoretype PKCS12 -srcstorepass android -alias youralias
```

### 总共会生成以下三个文件，其中debug.keystore就是我们需要的
```
shared.pk12
shared.priv.pem
debug.keystore
```

### 部署到AndroidStudio
可以参考之前的文章{% post_link AndroidStudio-Gradle-全局参数配置 %}

