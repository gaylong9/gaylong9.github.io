---
title: Windows10下配置Dlib
date: 2021-08-11 14:52:28
tags: 杂项
---



[toc]

记录Windows10下用Python安装dlib的过程。

当然，安装前先要装好[Anaconda](https://www.anaconda.com/products/individual)并创建环境，方便后续包的管理。

[conda环境管理命令](https://www.cnblogs.com/liaohuiqiang/p/9380417.html)



# 1. wheel安装

此法不支持GPU。

通过wheel可以直接安装dlib包，不需要编译等操作。

只需在wheel文件的目录中cmd执行`pip install xxx.whl`。

以下记录几种wheel获取途径：

* dlib的[19.8.1版本](https://pypi.org/project/dlib/19.8.1/#files)是最后一个官方提供wheel的版本，不过**最高只支持到Python 3.6版本**，Python版本不符合时会报错提示

* [这里](https://github.com/shashankx86/dlib_compiled)有好心人分享的Python 3.9版本的wheel，安装时如提示`is not a supported wheel on this platform`，可参考[此博客](https://blog.csdn.net/happywlg123/article/details/107281936)，使用`pip debug --verbose`，查看`Compatible tags:`后的内容，将文件名进行对应修改，再次`pip install xxxx.whl`

* 第三种不算是wheel方法，但结果一致，故记录在此；

	先添加清华源：

	```bash
	conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
	conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge 
	conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
	```

	随后`conda install -c menpo dlib=19.20`（此法对Python和dlib版本要求不严，但19.19版本测试有误，故使用19.20）



# 2. pip安装

可先配置CUDA环境再pip，可支持GPU。（dlib安装时会自动寻找cuda环境，在笔记本上pip测试安装时支持CUDA，待后续验证）

pip安装时，会自动安装较新的版本，但由于没有wheel，需要Visual Studio中的C++编译器，即时构建wheel。

1. 安装[Visual Studio](https://visualstudio.microsoft.com/zh-hans/downloads/)，community社区版即可；与多数博客所讲的要求2015 update3不同，我在配置过程中使用2019版本的VS也能成功安装
2. 安装VS的工作负荷和单个组件，以下列出dlib安装成功时的VS组件安装情况，并不保证每项都是必需的：
	1. 工作负荷：
		1. C++桌面开发
		2. 通用桌面开发
	2. 单个组件：
		1. C++核心功能
		2. MSBuild
		3. MSVC v142 - VS 2019 c++ arm生成工具 v14.25（此项选择 最新 也可）
		4. MSVC v142 - VS 2019 c++ arm64生成工具 v14.25（此项选择 最新 也可）
		5. MSVC v142 - VS 2019 c++ x64/86生成工具 v14.25（此项选择 最新 也可）
		6. 用于 v142 生成工具的C++通用windows平台（ARM64）
		7. 用于Windows的C++ Cmake工具
3. `pip install cmake`：直接命令行输入`cmake`测试安装是否成功，能识别此命令即可
4. `pip install boost`：有些地方提到较新的dlib不需此包，但又小又安装快捷，所以还是装上了
5. `pip install dlib -v`：**重点：-v是为了观察verbose，安装失败的话寻找verbose中的error信息并解决即可（有很多warning是正常现象，且不加-v的话安装失败也会显示verbose）**



# 3. 手动编译，setup.py安装

手动编译dlib，能使其支持GPU。（推荐优先尝试pip安装的方法，环境要求一致，并且会自动判断GPU是否可用）

[参考1](https://blog.csdn.net/weixin_42134521/article/details/82903218)

[参考2](https://blog.csdn.net/qq_29168809/article/details/102655115)

1. 如[2. pip安装](# 2. pip安装)，安装Visual Studio和相关组件，pip安装`cmake`和`boost`

2. 下载[dlib项目](https://github.com/davisking/dlib)

3. 在该项目根目录`dlib-master`下新建`build`文件夹（也可命令行`mkdir build`），**并进入**

4. **在build目录下**，

	命令行`cmake .. -DDLIB_USE_CUDA=1 -DUSE_AVX_INSTRUCTIONS=1`，参数意为是否使用CUDA，是否使用CPU加速，根据实际情况设置；执行结束显示：

	```bash
	-- Found Threads: TRUE
	-- C++11 activated.
	-- Configuring done
	-- Generating done
	-- Build files have been written to: D:/Download/dlib-master/build
	```

5. 命令行`cmake --build .`（最后有`.`，一篇参考博客遗漏了），显示：

	```bash
	dlib.vcxproj -> D:\Download\dlib-master\build\dlib\Debug\dlib19.22.99_debug_64bit_msvc1925.lib
	Building Custom Rule D:/Download/dlib-master/CMakeLists.txt
	```

6. **返回上级目录**，命令行`python setup.py install`，如失败，同样寻找error并解决；成功则显示：

	```bash
	Processing dlib-19.22.99-py3.7-win-amd64.egg
	creating d:\software\anaconda\envs\temp\lib\site-packages\dlib-19.22.99-py3.7-win-amd64.egg
	Extracting dlib-19.22.99-py3.7-win-amd64.egg to d:\software\anaconda\envs\temp\lib\site-packages
	Adding dlib 19.22.99 to easy-install.pth file
	
	Installed d:\software\anaconda\envs\temp\lib\site-packages\dlib-19.22.99-py3.7-win-amd64.egg
	Processing dependencies for dlib==19.22.99
	Finished processing dependencies for dlib==19.22.99
	```

	  

# 4. 验证安装成功

安装成功后，可进入python测试:

```
(test) D:\Download\dlib-master>python
>>> import dlib
>>> dlib.__version__
'19.22.0'
>>> dlib.DLIB_USE_CUDA
False
>>>
```

其中dlib.DLIB_USE_CUDA输出False表示当前dlib不支持CUDA。



# 5. 遇到的问题

1. conda新建环境后，环境目录下没有包，只有一个meta文件夹：新建命令加上`python`或指出Python版本，如`conda create -n new_env python` `conda create -n new_env python=3.9`
2. 安装dlib时遇到的一个error信息：`error C2734: “GifAsciiTable8x8”: 如果不是外部的，则必须初始化常量对象`，该问题出在`Anaconda\Library\include\gif_lib.h`文件的第286行，添加前缀`extern`表示它是外部变量即可；也有博客提到使用`python setup.py install --no DLIB_GIF_SUPPORT`方法，这就需要下载[dlib项目](https://github.com/davisking/dlib)，用setup.py安装（这个出问题的文件在另一台电脑上根本就不存在，很是神秘）



# 6. 总结与完整verbose

核心就是要安装Visual Studio及其各个组件、cmake和boost，**并在安装dlib时查看报错信息，找到error加以解决**。

有博客提到不需要下载完整Visual Studio，只需安装visual cpp build tools即可，但是我一直安装失败，所以放弃这种方法了。

另，不支持GPU的wheel可以直接拿来安装（Python与平台等条件都适配的情况下），支持GPU的wheel中已写入了该电脑的CUDA配置，无法直接使用。

最后贴出完整的pip安装时的verbose，以供对照（例中没有CUDA环境）。

```shell
(test) D:\Download\dlib-master>pip install dlib -v
Using pip 21.2.3 from D:\Software\Anaconda\envs\test\lib\site-packages\pip (python 3.9)
Collecting dlib
  Using cached dlib-19.22.0.tar.gz (7.4 MB)
    Running command python setup.py egg_info
    running egg_info
    creating C:\Users\22366_gu\AppData\Local\Temp\pip-pip-egg-info-hjnlvfnk\dlib.egg-info
    writing C:\Users\22366_gu\AppData\Local\Temp\pip-pip-egg-info-hjnlvfnk\dlib.egg-info\PKG-INFO
    writing dependency_links to C:\Users\22366_gu\AppData\Local\Temp\pip-pip-egg-info-hjnlvfnk\dlib.egg-info\dependency_links.txt
    writing top-level names to C:\Users\22366_gu\AppData\Local\Temp\pip-pip-egg-info-hjnlvfnk\dlib.egg-info\top_level.txt
    writing manifest file 'C:\Users\22366_gu\AppData\Local\Temp\pip-pip-egg-info-hjnlvfnk\dlib.egg-info\SOURCES.txt'
    package init file 'tools\python\dlib\__init__.py' not found (or not a regular file)
    reading manifest file 'C:\Users\22366_gu\AppData\Local\Temp\pip-pip-egg-info-hjnlvfnk\dlib.egg-info\SOURCES.txt'
    reading manifest template 'MANIFEST.in'
    no previously-included directories found matching 'tools\python\build*'
    no previously-included directories found matching 'dlib\test'
    writing manifest file 'C:\Users\22366_gu\AppData\Local\Temp\pip-pip-egg-info-hjnlvfnk\dlib.egg-info\SOURCES.txt'
Building wheels for collected packages: dlib
  Running command 'D:\Software\Anaconda\envs\test\python.exe' -u -c 'import io, os, sys, setuptools, tokenize; sys.argv[0] = '"'"'C:\\Users\\22366_gu\\AppData\\Local\\Temp\\pip-install-hsjixceq\\dlib_680b8efad36243a0b84232cc3915f74b\\setup.py'"'"'; __file__='"'"'C:\\Users\\22366_gu\\AppData\\Local\\Temp\\pip-install-hsjixceq\\dlib_680b8efad36243a0b84232cc3915f74b\\setup.py'"'"';f = getattr(tokenize, '"'"'open'"'"', open)(__file__) if os.path.exists(__file__) else io.StringIO('"'"'from setuptools import setup; setup()'"'"');code = f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' bdist_wheel -d 'C:\Users\22366_gu\AppData\Local\Temp\pip-wheel-l92n3vf8'
  running bdist_wheel
  running build
  running build_py
  package init file 'tools\python\dlib\__init__.py' not found (or not a regular file)
  running build_ext
  Building extension for Python 3.9.6 | packaged by conda-forge | (default, Jul 11 2021, 03:37:25) [MSC v.1916 64 bit (AMD64)]
  Invoking CMake setup: 'cmake C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python -DCMAKE_LIBRARY_OUTPUT_DIRECTORY=C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\lib.win-amd64-3.9 -DPYTHON_EXECUTABLE=D:\Software\Anaconda\envs\test\python.exe -DCMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE=C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\lib.win-amd64-3.9 -A x64'
  -- Building for: Visual Studio 16 2019
  -- Selecting Windows SDK version 10.0.19041.0 to target Windows 10.0.19042.
  -- The C compiler identification is MSVC 19.29.30040.0
  -- The CXX compiler identification is MSVC 19.29.30040.0
  -- Detecting C compiler ABI info
  -- Detecting C compiler ABI info - done
  -- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2019/Community/VC/Tools/MSVC/14.29.30037/bin/Hostx64/x64/cl.exe - skipped
  -- Detecting C compile features
  -- Detecting C compile features - done
  -- Detecting CXX compiler ABI info
  -- Detecting CXX compiler ABI info - done
  -- Check for working CXX compiler: C:/Program Files (x86)/Microsoft Visual Studio/2019/Community/VC/Tools/MSVC/14.29.30037/bin/Hostx64/x64/cl.exe - skipped
  -- Detecting CXX compile features
  -- Detecting CXX compile features - done
  -- Found PythonInterp: D:/Software/Anaconda/envs/test/python.exe (found version "3.9.6")
  -- Found PythonLibs: D:/Software/Anaconda/envs/test/libs/Python39.lib
  -- pybind11 v2.2.4
  -- Using CMake version: 3.20.21032501-MSVC_2
  -- Compiling dlib version: 19.22.0
  -- Looking for sys/types.h
  -- Looking for sys/types.h - found
  -- Looking for stdint.h
  -- Looking for stdint.h - found
  -- Looking for stddef.h
  -- Looking for stddef.h - found
  -- Check size of void*
  -- Check size of void* - done
  -- Enabling SSE2 instructions
  -- Searching for BLAS and LAPACK
  -- Searching for BLAS and LAPACK
  -- Looking for pthread.h
  -- Looking for pthread.h - not found
  -- Found Threads: TRUE
  CUDA_TOOLKIT_ROOT_DIR not found or specified
  -- Could NOT find CUDA (missing: CUDA_TOOLKIT_ROOT_DIR CUDA_NVCC_EXECUTABLE CUDA_INCLUDE_DIRS CUDA_CUDART_LIBRARY) (Required is at least version "7.5")
  -- Found CUDA, but CMake was unable to find the cuBLAS libraries that should be part of every basic CUDA install. Your CUDA install is somehow broken or incomplete. Since cuBLAS is required for dlib to use CUDA we won't use CUDA.
  -- DID NOT FIND CUDA
  -- Disabling CUDA support for dlib.  DLIB WILL NOT USE CUDA
  -- C++11 activated.
  -- Configuring done
  -- Generating done
  -- Build files have been written to: C:/Users/22366_gu/AppData/Local/Temp/pip-install-hsjixceq/dlib_680b8efad36243a0b84232cc3915f74b/build/temp.win-amd64-3.9/Release
  Invoking CMake build: 'cmake --build . --config Release -- /m'
  用于 .NET Framework 的 Microsoft (R) 生成引擎版本 16.10.2+857e5a733
  版权所有(C) Microsoft Corporation。保留所有权利。

  C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Microsoft\VC\v160\Microsoft.CppBuild.targets(517,5): warning MSB8029: 中间目录或输出目录无法驻留在临时目录下，因为这可能会导致增量生成出现问题。 [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\ZERO_CHECK.vcxproj]
    Checking Build System
  C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Microsoft\VC\v160\Microsoft.CppBuild.targets(517,5): warning MSB8029: 中间目录或输出目录无法驻留在临时目录下，因为这可能会导致增量生成出现问题。 [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\dlib_build\dlib.vcxproj]
    Building Custom Rule C:/Users/22366_gu/AppData/Local/Temp/pip-install-hsjixceq/dlib_680b8efad36243a0b84232cc3915f74b/dlib/CMakeLists.txt
    base64_kernel_1.cpp
    bigint_kernel_1.cpp
    bigint_kernel_2.cpp
    bit_stream_kernel_1.cpp
    entropy_decoder_kernel_1.cpp
    entropy_decoder_kernel_2.cpp
    entropy_encoder_kernel_1.cpp
    entropy_encoder_kernel_2.cpp
    fonts.cpp
    md5_kernel_1.cpp
    tokenizer_kernel_1.cpp
    unicode.cpp
    test_for_odr_violations.cpp
    sockets_kernel_1.cpp
    bsp.cpp
    dir_nav_kernel_1.cpp
    dir_nav_kernel_2.cpp
    dir_nav_extensions.cpp
    linker_kernel_1.cpp
    extra_logger_headers.cpp
    logger_kernel_1.cpp
    logger_config_file.cpp
    misc_api_kernel_1.cpp
    misc_api_kernel_2.cpp
    sockets_extensions.cpp
    sockets_kernel_2.cpp
    sockstreambuf.cpp
    sockstreambuf_unbuffered.cpp
    server_kernel.cpp
    server_iostream.cpp
    server_http.cpp
    multithreaded_object_extension.cpp
    threaded_object_extension.cpp
    threads_kernel_1.cpp
    threads_kernel_2.cpp
    threads_kernel_shared.cpp
    thread_pool_extension.cpp
    async.cpp
    timer.cpp
    stack_trace.cpp
    cpu_dlib.cpp
    tensor_tools.cpp
    image_dataset_metadata.cpp
    mnist.cpp
    cifar.cpp
    global_function_search.cpp
    kalman_filter.cpp
    auto.cpp
    widgets.cpp
    drawable.cpp
    canvas_drawing.cpp
    style.cpp
    base_widgets.cpp
    gui_core_kernel_1.cpp
    gui_core_kernel_2.cpp
    png_loader.cpp
    save_png.cpp
    jpeg_loader.cpp
    save_jpeg.cpp
    arm_init.c
    filter_neon_intrinsics.c
    png.c
    pngerror.c
    pngget.c
    pngmem.c
    pngpread.c
    pngread.c
    pngrio.c
    pngrtran.c
    pngrutil.c
    pngset.c
    pngtrans.c
    pngwio.c
    pngwrite.c
    pngwtran.c
    pngwutil.c
    adler32.c
    compress.c
    crc32.c
    deflate.c
    gzclose.c
    gzlib.c
    gzread.c
    gzwrite.c
    infback.c
    inffast.c
    inflate.c
    inftrees.c
    trees.c
    uncompr.c
    zutil.c
    jaricom.c
    jcapimin.c
    jcapistd.c
    jcarith.c
    jccoefct.c
    jccolor.c
    jcdctmgr.c
    jchuff.c
    jcinit.c
    jcmainct.c
    jcmarker.c
    jcmaster.c
    jcomapi.c
    jcparam.c
    jcprepct.c
    jcsample.c
    jdapimin.c
    jdapistd.c
    jdarith.c
    jdatadst.c
    jdatasrc.c
    jdcoefct.c
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\external\libjpeg\jdatadst.c(185,60): warning C4267: “=”: 从“size_t”转换到“unsigned long”，可能丢失数据 [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\dlib_build\dlib.vcxproj]
    jdcolor.c
    jddctmgr.c
    jdhuff.c
    jdinput.c
    jdmainct.c
    jdmarker.c
    jdmaster.c
    jdmerge.c
    jdpostct.c
    jdsample.c
    jerror.c
    jfdctflt.c
    jfdctfst.c
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\external\libjpeg\jerror.c(193,5): warning C4996: 'sprintf': This function or variable may be unsafe. Consider using sprintf_s instead. To disable deprecation, use _CRT_SECURE_NO_WARNINGS. See online help for details. [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\dlib_build\dlib.vcxproj]
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\external\libjpeg\jerror.c(195,5): warning C4996: 'sprintf': This function or variable may be unsafe. Consider using sprintf_s instead. To disable deprecation, use _CRT_SECURE_NO_WARNINGS. See online help for details. [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\dlib_build\dlib.vcxproj]
    jfdctint.c
    jidctflt.c
    jidctfst.c
    jidctint.c
    jmemmgr.c
    jmemnobs.c
    jquant1.c
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\external\libjpeg\jmemmgr.c(307,53): warning C4267: “+=”: 从“size_t”转换到“long”，可能丢失数据 [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\dlib_build\dlib.vcxproj]
    jquant2.c
    jutils.c
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\external\libjpeg\jmemmgr.c(367,70): warning C4267: “+=”: 从“size_t”转换到“long”，可能丢失数据 [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\dlib_build\dlib.vcxproj]
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\external\libjpeg\jmemmgr.c(977,46): warning C4267: “-=”: 从“size_t”转换到“long”，可能丢失数据 [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\dlib_build\dlib.vcxproj]
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\external\libjpeg\jmemmgr.c(991,46): warning C4267: “-=”: 从“size_t”转换到“long”，可能丢失数据 [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\dlib_build\dlib.vcxproj]
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\external\libjpeg\jmemmgr.c(1107,19): warning C4996: 'getenv': This function or variable may be unsafe. Consider using _dupenv_s instead. To disable deprecation, use _CRT_SECURE_NO_WARNINGS. See online help for details. [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\dlib_build\dlib.vcxproj]
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\external\libjpeg\jmemmgr.c(1110,11): warning C4996: 'sscanf': This function or variable may be unsafe. Consider using sscanf_s instead. To disable deprecation, use _CRT_SECURE_NO_WARNINGS. See online help for details. [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\dlib_build\dlib.vcxproj]
    dlib.vcxproj -> C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\dlib_build\Release\dlib19.22.0_release_64bit_msvc1929.lib
  C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Microsoft\VC\v160\Microsoft.CppBuild.targets(517,5): warning MSB8029: 中间目录或输出目录无法驻留在临时目录下，因为这可能会导致增量生成出现问题。 [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
    Building Custom Rule C:/Users/22366_gu/AppData/Local/Temp/pip-install-hsjixceq/dlib_680b8efad36243a0b84232cc3915f74b/tools/python/CMakeLists.txt
    dlib.cpp
    matrix.cpp
    vector.cpp
    svm_c_trainer.cpp
    svm_rank_trainer.cpp
    decision_functions.cpp
    other.cpp
    basic.cpp
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\image_transforms/morphological_operations.h(1,1): warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\vector.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
    cca.cpp
    sequence_segmenter.cpp
    svm_struct.cpp
    image.cpp
    image2.cpp
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\image_transforms/morphological_operations.h(1,1): warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\image.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\image_transforms/morphological_operations.h(1,1): warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\image2.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
    image3.cpp
    image4.cpp
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\image_transforms/morphological_operations.h(1,1): warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\image3.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
    rectangles.cpp
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\image_transforms/morphological_operations.h(1,1): warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\image4.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\..\dlib/image_transforms/assign_image.h(86,45): warning C4018: “>=”: 有符号/无符号不匹配 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\image3.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\..\dlib/image_transforms/assign_image.h(160): message : 查看对正在编译的函数 模板 实例化“void dlib::impl_assign_image_scaled<out_image_type,src_image_type>(dlib::image_view<out_image_type> &,const src_image_type &,const double)”的引用 [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
            with
            [
                out_image_type=dlib::numpy_image<int8_t>,
                src_image_type=dlib::matrix_op<dlib::op_image_to_mat<dlib::numpy_image<uint32_t>,uint32_t>>
            ] (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\image3.cpp)
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\..\dlib/image_transforms/assign_image.h(177): message : 查看对正在编译的函数 模板 实例化“void dlib::impl_assign_image_scaled<dest_image_type,dlib::matrix_op<dlib::op_image_to_mat<src_image_type,T>>>(dest_image_type &,const dlib::matrix_op<dlib::op_image_to_mat<src_image_type,T>> &,const double)”的引用 [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
            with
            [
                dest_image_type=dlib::numpy_image<int8_t>,
                src_image_type=dlib::numpy_image<uint32_t>,
                T=uint32_t
            ] (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\image3.cpp)
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\image3.cpp(27): message : 查看对正在编译的函数 模板 实例化“void dlib::assign_image_scaled<dlib::numpy_image<int8_t>,dlib::numpy_image<uint32_t>>(dest_image_type &,const src_image_type &,const double)”的引用 [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
            with
            [
                dest_image_type=dlib::numpy_image<int8_t>,
                src_image_type=dlib::numpy_image<uint32_t>
            ]
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\image3.cpp(435): message : 查看对正在编译的函数 模板 实例化“pybind11::array convert_image_scaled<uint32_t>(const dlib::numpy_image<uint32_t> &,const std::string &,const double)”的引用 [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\..\dlib/image_transforms/assign_image.h(87,45): warning C4018: “<=”: 有符号/无符号不匹配 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\image3.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
    object_detection.cpp
    shape_predictor.cpp
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\image_transforms/morphological_operations.h(1,1): warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\object_detection.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\image_transforms/morphological_operations.h(1,1): warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\shape_predictor.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
    correlation_tracker.cpp
    face_recognition.cpp
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\image_transforms/morphological_operations.h(1,1): warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\correlation_tracker.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
    cnn_face_detector.cpp
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\image_transforms/morphological_operations.h(1,1): warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\face_recognition.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
    global_optimization.cpp
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\image_transforms/morphological_operations.h(1,1): warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\cnn_face_detector.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
    image_dataset_metadata.cpp
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\image_transforms/morphological_operations.h(1,1): warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\image_dataset_metadata.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
    numpy_returns.cpp
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\image_transforms/morphological_operations.h(1,1): warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\numpy_returns.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
    line.cpp
    gui.cpp
  C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\dlib\image_transforms/morphological_operations.h(1,1): warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失 (编译源文件 C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\tools\python\src\gui.cpp) [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\_dlib_pybind11.vcxproj]
      正在创建库 C:/Users/22366_gu/AppData/Local/Temp/pip-install-hsjixceq/dlib_680b8efad36243a0b84232cc3915f74b/build/temp.win-amd64-3.9/Release/Release/_dlib_pybind11.lib 和对象 C:/Users/22366_gu/AppData/Local/Temp/pip-install-hsjixceq/dlib_680b8efad36243a0b84232cc3915f74b/build/temp.win-amd64-3.9/Release/Release/_dlib_pybind11.exp
    _dlib_pybind11.vcxproj -> C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\lib.win-amd64-3.9\_dlib_pybind11.cp39-win_amd64.pyd
  C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Microsoft\VC\v160\Microsoft.CppBuild.targets(517,5): warning MSB8029: 中间目录或输出目录无法驻留在临时目录下，因为这可能会导致增量生成出现问题。 [C:\Users\22366_gu\AppData\Local\Temp\pip-install-hsjixceq\dlib_680b8efad36243a0b84232cc3915f74b\build\temp.win-amd64-3.9\Release\ALL_BUILD.vcxproj]
    Building Custom Rule C:/Users/22366_gu/AppData/Local/Temp/pip-install-hsjixceq/dlib_680b8efad36243a0b84232cc3915f74b/tools/python/CMakeLists.txt
  installing to build\bdist.win-amd64\wheel
  running install
  running install_lib
  creating build\bdist.win-amd64
  creating build\bdist.win-amd64\wheel
  creating build\bdist.win-amd64\wheel\dlib
  copying build\lib.win-amd64-3.9\dlib\__init__.py -> build\bdist.win-amd64\wheel\.\dlib
  copying build\lib.win-amd64-3.9\_dlib_pybind11.cp39-win_amd64.pyd -> build\bdist.win-amd64\wheel\.
  running install_egg_info
  running egg_info
  writing tools/python\dlib.egg-info\PKG-INFO
  writing dependency_links to tools/python\dlib.egg-info\dependency_links.txt
  writing top-level names to tools/python\dlib.egg-info\top_level.txt
  reading manifest file 'tools/python\dlib.egg-info\SOURCES.txt'
  reading manifest template 'MANIFEST.in'
  no previously-included directories found matching 'tools\python\build*'
  no previously-included directories found matching 'dlib\test'
  writing manifest file 'tools/python\dlib.egg-info\SOURCES.txt'
  Copying tools/python\dlib.egg-info to build\bdist.win-amd64\wheel\.\dlib-19.22.0-py3.9.egg-info
  running install_scripts
  D:\Software\Anaconda\envs\test\lib\site-packages\wheel\bdist_wheel.py:80: RuntimeWarning: Config variable 'Py_DEBUG' is unset, Python ABI tag may be incorrect
    if get_flag('Py_DEBUG',
  creating build\bdist.win-amd64\wheel\dlib-19.22.0.dist-info\WHEEL
  creating 'C:\Users\22366_gu\AppData\Local\Temp\pip-wheel-l92n3vf8\dlib-19.22.0-cp39-cp39-win_amd64.whl' and adding 'build\bdist.win-amd64\wheel' to it
  adding '_dlib_pybind11.cp39-win_amd64.pyd'
  adding 'dlib/__init__.py'
  adding 'dlib-19.22.0.dist-info/METADATA'
  adding 'dlib-19.22.0.dist-info/WHEEL'
  adding 'dlib-19.22.0.dist-info/top_level.txt'
  adding 'dlib-19.22.0.dist-info/RECORD'
  removing build\bdist.win-amd64\wheel
  Building wheel for dlib (setup.py) ... done
  Created wheel for dlib: filename=dlib-19.22.0-cp39-cp39-win_amd64.whl size=2946201 sha256=4f0c103da0eadc711bf08b265141813b688229362f8fd24d3ad2a7ecace38a0b
  Stored in directory: c:\users\22366_gu\appdata\local\pip\cache\wheels\33\2c\16\6ceb1bda8e67571304aaf3cd80c8cb47dc8d5ab99e34bda88b
Successfully built dlib
Installing collected packages: dlib
Successfully installed dlib-19.22.0
```

&nbsp;

