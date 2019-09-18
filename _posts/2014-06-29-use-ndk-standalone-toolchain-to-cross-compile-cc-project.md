---
layout: post
title: "使用 Android NDK 的交叉编译工具链移植 C/C++ 项目到安卓平台"
description: ""
category: [Android, NDK, C++]
---

## 什么是 NDK？

[Android NDK][1] 是一套可以让开发者在安卓应用开发中使用 C/C++ 实现特定模块的工具集，不是所有应用都需要用到，但是正确地使用可以有效提高应用运行效率和安全性。

## 为什么要在安卓开发中使用 NDK？

* 游戏引擎使用 Native 的 C/C++ 库，便于跨平台移植，开发游戏应用，使用NDK可以直接调用这些库
* 通用应用开发中，像加密、解密以及其他一些核心算法等等都可以用 C/C++ 实现，使用NDK编译为动态库后，Java 可以通过 `Jni` 调用。提高性能和安全性。
* 移植一些通用的 C/C++ 工具到安卓平台，以便我们的应用能使用这些 Utilities， 比如 Curl 等。
* NDK 支持用纯 C++ 开发安卓应用，使用 OpenGL 来创建交互界面。

## 如何安装NDK

在安装好 [安卓开发环境][2] 后，参考官方说明进行 [NDK的下载配置][3]。本文着重讲如何用 NDK 移植 C/C++ 的项目到安卓系统，至于在安卓应用项目中使用 NDK 进行开发，包括使用纯 C++ 进行开发和通过 Jni 和 Java 代码配合的开发网上已经有很多朋友的总结。

## NDK 移植 C/C++ 项目到安卓平台

安卓系统也是一个基于 ARM 架构的嵌入式系统，其内核是 Linux 操作系统，而且 Google 支持从 NDK 中导出独立的交叉编译工具链。如此我们就可以参考嵌入式开发和移植的经验。这篇文章移植 curl 库到安卓平台，然后我们用一个使用 curl 的二进制可执行文件在安卓系统中进行测试。

- 从 NDK 中导出 ``standalone` 的交叉编译工具链

    * `/usr/local/android-ndk-r9d` 需要被替换为 NDK 的实际路径。
    * `make-standalone-toolchain.sh` 是 Google 官方提供的导出交叉编译工具链的脚本
    * `--platform=android-19` 指定导出的 API，19 代表安卓系统版本 4.4.2
    * 默认情况下，导出的是 ARM 架构的工具链。如果需要 X86 后者 Mips 架构的，需要加上 `--arch=x86` 或者 `--arch=mips`
    * 导出的压缩包将会位于 `/tmp/ndk/TOOL_CHAIN_NAME.tar.bz2`

{% highlight bash %}

    /usr/local/android-ndk-r9d/build/tools/make-standalone-toolchain.sh --platform=android-19

{% endhighlight %}

- 找到导出的压缩包，解压到自己定义的目录下

{% highlight bash %}

mkdir Android19NDK48
tar -xjvf /tmp/ndk-rwang/arm-linux-androideabi-4.8.tar.bz2 -C ~/Android19NDK48/

{% endhighlight %}

- 将解压后 `toolchain` 所在的路径添加到环境变量中并重新加载环境配置文件。

- 下载 [CurL][4] 的源代码压缩包，并解压缩，然后进入 src 目录：

{% highlight bash %}

tar -xzvf curl-7.36.0.tar.gz -C CURLSrc/
cd CURLSrc/

{% endhighlight %}

- 运行 `configure` 生成 makefile 文件，配置参数说明参见 [NDK交叉编译需要注意的地方][5]

{% highlight bash %}

./configure CC=arm-linux-androideabi-gcc --host=arm-linux-androideabi CFLAGS='-march=armv7-a -mfloat-abi=softfp' LDFLAGS='-Wl,--fix-cortex-a8'

{% endhighlight %}

- 创建一个 C++ 源文件，添加 curl 的测试代码

{% highlight C++ %}

#include <stdio.h>
#include <curl/curl.h>

int main()
{
    CURL *curl;
    CURLcode res;
    FILE *fp;
    if ((fp = fopen("test.txt", "w")) == NULL)
    return 1;
    struct curl_slist *headers = NULL;
    headers = curl_slist_append(headers, "Accept: Agent-007");
    curl = curl_easy_init();
    if (curl)
    {
        curl_easy_setopt(curl, CURLOPT_HTTPHEADER, headers);
        curl_easy_setopt(curl, CURLOPT_URL,"http://www.baidu.com");
        curl_easy_setopt(curl, CURLOPT_WRITEDATA, fp); 
        curl_easy_setopt(curl, CURLOPT_HEADERDATA, fp); 
        res = curl_easy_perform(curl); 
        if (res != 0) {

            curl_slist_free_all(headers);
            curl_easy_cleanup(curl);
        }
        fclose(fp);
        return 0;
    }
}

{% endhighlight %}

- 创建 makefile 文件，测试项目只有一个源文件，使用 makefile 主要是为了管理编译参数方便
    
{% highlight bash %}

CC= arm-linux-androideabi-g++
CFLAGS='-march=armv7-a -mfloat-abi=softfp'
LDFLAGS='-Wl,--fix-cortex-a8'

INCLUDES+= -I.
INCLUDES+= -I/home/rwang/Android19NDK48/sysroot/usr/include
LIBS+= -L/home/rwang/Android19NDK48/sysroot/usr/lib -lcurl -lz

src := $(shell ls *.cpp)
objs := $(patsubst %.cpp,%.o,$(src))

curlTestOnAndroid : $(objs)
$(CC) -o $@ $^ $(LIBS)

%.o: %.cpp
$(CC) -c $< -o $@ $(INCLUDES)

clean :
    -rm curlTestOnAndroid $(objs)

{% endhighlight %}

- 执行 `make` 命令进行编译之前，我们需要把之前移植的 curl 静态库放到交叉编译工具链的 lib 目录下，头文件拷贝到相应的 include 目录下，具体可以参考上面 makefile 里面的信息。拷贝后，运行 `make`

- 将编译好的二进制文件通过 `adb push` 命令上传到 ARM 架构的安卓设备，比如安卓手机，通过 `adb shell` 进入安卓系统的终端，赋权后运行查看结果。

至此，我们完成了使用基于NDK的交叉编译器进行安卓平台 C/C++ 开发的流程。NDK 对 C++ STL 的支持是有限的，在安卓应用开发中使用 C++ 源代码的时候，Application.mk 的设置需要参考 [这篇文章][6]

[1]: https://developer.android.com/tools/sdk/ndk/index.html
[2]: https://developer.android.com/tools/index.html
[3]: https://developer.android.com/tools/sdk/ndk/index.html#Installing
[4]: http://curl.haxx.se/download.html
[5]: http://www.kandroid.org/ndk/docs/STANDALONE-TOOLCHAIN.html
[6]: http://www.kandroid.org/ndk/docs/CPLUSPLUS-SUPPORT.html


