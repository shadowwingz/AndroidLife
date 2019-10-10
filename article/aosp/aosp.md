参考 [手把手教你在Mac OS下载、编译及导入Android源码](https://juejin.im/post/5cc5165fe51d456e781f2082#heading-8)

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