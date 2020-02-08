---
title: WSL2中编译安卓源码
tags:
  - Android
  - WSL2
  - AOSP
  - Android Pie
categories:
  - Android
  - WSL2
  - AOSP
  - Android Pie
abbrlink: fe9083b4
date: 2020-02-07 13:04:36
---

最新得Windows 10 版本18917或更高版本中，可以安装WSL2子系统，相比于WSL1，WSL2相当于提供了更加完整的Linux支持，因此可以用来编译Android系统源码，也不用再单独安装虚拟机了。

# 系统要求
* Windows 10 版本18917或更高版本中
* 足够的存储空间

# 准备工作
## 启用安装WSL2
参考微软官方：https://docs.microsoft.com/zh-cn/windows/wsl/wsl2-install

## 安装Ubuntu
应用商店中搜索Ubuntu，安装Ubuntu 18.04发行版
## 修改安装源
默认的源更新起来慢，可以修改成阿里云的源
* 备份原始源
```bash
sudo cp /etc/apt/sources.list /etc/apt/sourses.list.bak
```
* 更新阿里源
```bash
sudo vim /etc/apt/sources.list
# 将里面的内容替换成
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

* 更新
```
sudo apt update
sudo apt upgrade
```

## Windows Terminal
Windows Terminal是微软推出的一个现代化Terminal工具，很好用
* 到应用商店搜索Windows Terminal安装

## Git & Vim
* 安装
```
sudo apt install git vim
```
* 配置
```
git config --global user.name "your name"
git config --global user.email "your email"
```
## repo
```
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
## 如果上述 URL 不可访问，可以用下面的：
## curl -sSL  'https://gerrit-googlesource.proxy.ustclug.org/git-repo/+/master/repo?format=TEXT' |base64 -d > ~/bin/repo
chmod a+x ~/bin/repo
```

## JDK
```
sudo apt install openjdk-8-jdk
```

## 设置CCCache
```base
vim ~/.bashrc
# 追加以下内容
export USE_CCACHE=1
export CCACHE_DIR=/mnt/e/workspace/CCACHE/.ccache
```


# 针对源码工作目录，开启大小写敏感（重要）
* Windows系统中文件和文件夹默认是不区分大小写的，但是编译Android源码，需要区分大小的系统。
* 为了给WSL提供更好的支持，微软从Windows10 18917更新开始，为NTFS文件系统新增了一个`SetCaseSensitiveInfo`标志。可以有选择的根据所需的文件夹启用此flag，启用之后，NTFS文件系统就会针对**该文件夹及其子文件**视为区分大小写。
* 此功能不仅可以在WSL中起作用，也可以在Windows下起作用。

## 开启方式
* 以管理员身份打开命令提示符或者Powershell
* 执行以下命令开启
```bat
fsutil file SetCaseSensitiveInfo E:\workspace\AOSP enable
fsutil file SetCaseSensitiveInfo E:\workspace\CCACHE enable
```
* 执行以下命令关闭
```bat
fsutil file SetCaseSensitiveInfo E:\workspace\AOSP disable
fsutil file SetCaseSensitiveInfo E:\workspace\CCACHE disable
```
# 下载代码
## 初始化仓库
这里使用中科大的镜像：https://lug.ustc.edu.cn/wiki/mirrors/help/aosp

```bash
cd /mnt/e/workspace/AOSP
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest
## 如果提示无法连接到 gerrit.googlesource.com，可以编辑 ~/bin/repo，把 REPO_URL 一行替换成下面的：
## REPO_URL = 'https://gerrit-googlesource.proxy.ustclug.org/git-repo'
```
## 初始化特定版本
```bash
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-9.0.0_r40
```

## 同步源码
```bash
repo sync
```

## 也可以参考Google官方教程
* https://source.android.com/source/downloading.html

# 编译
我这里选择编译Pixel版本，因为手里有一个Pixel手机
```bash
source build/envsetup.sh
lunch 45 # aosp_sailfish-userdebug
make 
```

# 编译错误处理
* `dex2oatd F dex2oat did not finish after 2850 seconds`

  修改`build/core/`目录下的两个文件
  ```diff
  diff --git a/core/dex_preopt_libart.mk b/core/  dex_preopt_libart.mk
  index 9c4d55de7..3b1158d9c 100644
  --- a/core/dex_preopt_libart.mk
  +++ b/core/dex_preopt_libart.mk
  @@ -184,7 +184,7 @@ source build/make/core/ verify_uses_libraries.sh "$(1)" && \
   source build/make/core/construct_context.sh "$ (PRIVATE_CONDITIONAL_USES_LIBRARIES_HOST)" "$  (PRIVATE_CONDITIONAL_USES_LIBRARIES_TARGET)" && \
   ,) \
   ANDROID_LOG_TAGS="*:e" $(DEX2OAT) \
  -       --runtime-arg -Xms$(DEX2OAT_XMS) --runtime-arg  -Xmx$(DEX2OAT_XMX) \
  +       --runtime-arg -Xms$(DEX2OAT_XMS) -j1  --runtime-arg -Xmx$(DEX2OAT_XMX) \
          $${class_loader_context_arg} \
          $${stored_class_loader_context_arg} \
          --boot-image=$  (PRIVATE_DEX_PREOPT_IMAGE_LOCATION) \
  diff --git a/core/dex_preopt_libart_boot.mk b/core/ dex_preopt_libart_boot.mk
  index a5e7e881a..fadd6d794 100644
  --- a/core/dex_preopt_libart_boot.mk
  +++ b/core/dex_preopt_libart_boot.mk
  @@ -102,7 +102,7 @@ $($(my_2nd_arch_prefix) DEFAULT_DEX_PREOPT_BUILT_IMAGE_FILENAME) : $(LIBART_TARGE
          @rm -f $(dir $($(PRIVATE_2ND_ARCH_VAR_PREFIX) LIBART_TARGET_BOOT_OAT_UNSTRIPPED))/*.art
          @rm -f $(dir $($(PRIVATE_2ND_ARCH_VAR_PREFIX) LIBART_TARGET_BOOT_OAT_UNSTRIPPED))/*.oat
          @rm -f $(dir $($(PRIVATE_2ND_ARCH_VAR_PREFIX) LIBART_TARGET_BOOT_OAT_UNSTRIPPED))/*.art.rel
  -       $(hide) $(DEX2OAT_BOOT_IMAGE_LOG_TAGS) $  (DEX2OAT) --runtime-arg -Xms$(DEX2OAT_IMAGE_XMS) \
  +       $(hide) $(DEX2OAT_BOOT_IMAGE_LOG_TAGS) $  (DEX2OAT) -j1 --runtime-arg -Xms$(DEX2OAT_IMAGE_XMS) \
                  --runtime-arg -Xmx$(DEX2OAT_IMAGE_XMX) \
                  $(PRIVATE_BOOT_IMAGE_FLAGS) \
                  $(addprefix --dex-file=,$ (LIBART_TARGET_BOOT_DEX_FILES)) \
  ```


