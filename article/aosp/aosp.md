#### 编译步骤

初始化编译环境，包括后面的 lunch 和 make 指令

```
source setenv.sh
```

只编译 framwork

```
make framework
```

生成 idegen.jar 文件

```
mmm development/tools/idegen/
```

生成 android.iml 和 android.ipr 这两个文件

```
development/tools/idegen/idegen.sh
```

打开 android.iml 文件，搜下excludeFolder，在后面加入如下代码：

```
<excludeFolder url="file://$MODULE_DIR$/art" />
<excludeFolder url="file://$MODULE_DIR$/bionic" />
<excludeFolder url="file://$MODULE_DIR$/bootable" />
<excludeFolder url="file://$MODULE_DIR$/build" />
<excludeFolder url="file://$MODULE_DIR$/cts" />
<excludeFolder url="file://$MODULE_DIR$/dalvik" />
<excludeFolder url="file://$MODULE_DIR$/developers" />
<excludeFolder url="file://$MODULE_DIR$/development" />
<excludeFolder url="file://$MODULE_DIR$/device" />
<excludeFolder url="file://$MODULE_DIR$/docs" />
<excludeFolder url="file://$MODULE_DIR$/external" />
<excludeFolder url="file://$MODULE_DIR$/hardware" />
<excludeFolder url="file://$MODULE_DIR$/kernel" />
<excludeFolder url="file://$MODULE_DIR$/libcore" />
<excludeFolder url="file://$MODULE_DIR$/libnativehelper" />
<excludeFolder url="file://$MODULE_DIR$/out" />
<excludeFolder url="file://$MODULE_DIR$/pdk" />
<excludeFolder url="file://$MODULE_DIR$/platform_testing" />
<excludeFolder url="file://$MODULE_DIR$/prebuilts" />
<excludeFolder url="file://$MODULE_DIR$/sdk" />
<excludeFolder url="file://$MODULE_DIR$/system" />
<excludeFolder url="file://$MODULE_DIR$/test" />
<excludeFolder url="file://$MODULE_DIR$/toolchain" />
<excludeFolder url="file://$MODULE_DIR$/tools" />
<excludeFolder url="file://$MODULE_DIR$/.repo" />
```

参考 
[手把手教你在Mac OS下载、编译及导入Android源码](https://juejin.im/post/5cc5165fe51d456e781f2082#heading-8)
[Android Studio 导入 AOSP 源码](http://wuxiaolong.me/2018/08/15/AOSP3/)

编译源码遇到的一些问题：

#### make framework

FAILED: out/soong/.intermediates/bionic/libc/generated_android_ids/gen/generated_android_ids.h
out/soong/host/darwin-x86/bin/sbox --sandbox-path out/soong/.temp --output-root out/soong -c "bionic/libc/fs_config_generator.py aidarray system/core/include/private/android_filesystem_config.h > __SBOX_OUT_FILES__" out/soong/.intermediates/bionic/libc/generated_android_ids/gen/generated_android_ids.h
  File "bionic/libc/fs_config_generator.py", line 1021
    print FSConfigGen._FILE_COMMENT % fname
                    ^
SyntaxError: invalid syntax
sbox command (bionic/libc/fs_config_generator.py aidarray system/core/include/private/android_filesystem_config.h > out/soong/.temp/sbox201042838/.intermediates/bionic/libc/generated_android_ids/gen/generated_android_ids.h) failed with err "exit status 1"

ninja: build stopped: subcommand failed.
12:59:54 ninja failed with: exit status 1
make: *** [run_soong_ui] Error 1

#### 解决：

从报错信息看，是 `fs_config_generator.py` 中有语法错误，猜测是 python 版本问题，使用 `python -V` 命令查看 Mac 中的版本，是 3.4，谷歌要求是 2.6 或 2.7，所以使用 `pyenv global system`

========== 分割线 ==========

#### 类型转换异常

#### 解决：
编译只能用 `1.8.x` 版本的 jdk 版本，所以要用 jenv 切换 jdk 版本，切换版本之后如果还是有问题，使用 `make clean` 清除以前的编译信息，再 `make framework`。


========== 分割线 ==========

Android 模拟器无法调试进程

#### 解决：
模拟器使用的 image 是 Google Play 版的，要使用 Google api 版的，在 SDK manager 里。

========== 分割线 ==========

classes-full-debug.jar', missing and no known rule to make it

#### 解决：

[Android 8.0整体编译成功后使用mmm进行编译失败处理。](https://blog.csdn.net/m0_37039448/article/details/86654742)

