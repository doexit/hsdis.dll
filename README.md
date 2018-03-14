## 编译步骤
[How to build hsdis-amd64.dll and hsdis-i386.dll on Windows](https://dropzone.nfshost.com/hsdis.htm) 这篇文章说明了如何在Windows系统下编译hsdis，基本步骤如下：
1. 安装[Cygwin](https://www.cygwin.com/)，Cygwin的主要目的是通过重新编译，将POSIX系统（例如Linux、BSD，以及其他Unix系统）上的软件（以 GNU 工具为代表）移植到Windows上，具体安装步骤
可参考[Win下Cygwin的安装](http://blog.csdn.net/yupu56/article/details/53186410)，其中选择安装模块的步骤中，一定要安装Devel这个部分的模块，这个模块包含了各种开发所用到的工具或模块，最好全部安装，以免后续流程出现各种奇怪问题
2. 运行Cygwin终端，在你的安装目录中会创建`\home\username`文件夹
3. 下载[binutils](http://ftp.jaist.ac.jp/pub/GNU/binutils/), `binutils-2.24.tar.bz2`或其它最新版本，解压到主目录`\home\username`下
4. 下载[OpenJDK源码](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/tags)，选择与您安装的JRE版本对应的标记，并单击bz2。提取hsdis目录(在src/share/tools中)到Cygwin主目录`\home\username`
5. 在Cygwin终端，输入`cd hsdis`
6. 获取hsdis-amd64.dll，输入`make OS=Linux MINGW=x86_64-w64-mingw32 'AR=$(MINGW)-ar' BINUTILS=~/ BINUTILS -2.24`，获取hsdis-i386.dll，输入`make OS=Linux MINGW=i686-w64-mingw32 'AR=$(MINGW)-ar' BINUTILS=~/ BINUTILS -2.24`。在这两条命令中，用你下载的binutils版本替换2.24
7. 如果编译成功的话，可以在`hsdis\build\Linux-amd64`或者 `hsdis\build\Linux-i586`目录中找到我们想要的hsdis-amd64.dll和hsdis-i386.dll，将它放到JRE安装目录的`bin\server`或者`bin\client`文件夹即可

## 编译错误解决

在第6步编译提示
```
x86_64-w64-mingw32-gcc -o build/Linux-amd64/hsdis-amd64.dll -Ibuild/Linux-amd64/include -I/home/Administrator/binutils-2.24/include -I/home/Administrator/binutils-2.24/bfd -Ibuild/Linux-amd64/bfd -DLIBARCH_amd64 -DLIBARCH=\"amd64\" -DLIB_EXT=\".dll\" -O hsdis.c -shared build/Linux-amd64/bfd/libbfd.a build/Linux-amd64/opcodes/libopcodes.a build/Linux-amd64/libiberty/libiberty.a
build/Linux-amd64/bfd/libbfd.a(compress.o):compress.c:(.text+0x15)：对‘compressBound’未定义的引用
build/Linux-amd64/bfd/libbfd.a(compress.o):compress.c:(.text+0x48)：对‘compress’未定义的引用
build/Linux-amd64/bfd/libbfd.a(compress.o):compress.c:(.text+0x2a7)：对‘inflateInit_’未定义的引用
build/Linux-amd64/bfd/libbfd.a(compress.o):compress.c:(.text+0x2e1)：对‘inflate’未定义的引用
build/Linux-amd64/bfd/libbfd.a(compress.o):compress.c:(.text+0x2f0)：对‘inflateReset’未定义的引用
build/Linux-amd64/bfd/libbfd.a(compress.o):compress.c:(.text+0x30f)：对‘inflateEnd’未定义的引用
collect2: 错误：ld 返回 1
make: *** [Makefile:198：build/Linux-amd64/hsdis-amd64.dll] 错误 1
```
错误提示`对‘compressBound’未定义的引用`指链接的时候没有加上zlib的库，需要添加`-lz`后缀解决，打开hsdis下的Makefile文件，找到
```
$(TARGET): $(SOURCE) $(LIBS) $(LIBRARIES) $(TARGET_DIR)
	$(CC) $(OUTFLAGS) $(CPPFLAGS) $(CFLAGS) $(SOURCE) $(DLDFLAGS) $(LIBRARIES)
```
在后面添加`-static -lz`
```
$(TARGET): $(SOURCE) $(LIBS) $(LIBRARIES) $(TARGET_DIR)
	$(CC) $(OUTFLAGS) $(CPPFLAGS) $(CFLAGS) $(SOURCE) $(DLDFLAGS) $(LIBRARIES) -static -lz
```
然后，在第6步执行make命令，即可编译成功