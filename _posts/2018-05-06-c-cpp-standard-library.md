---
title: "【译】什么是 C/C++ 标准库"
comments: true
---

【译者序】 原文链接：<a href="https://www.internalpointers.com/post/c-c-standard-library" target="_blank">https://www.internalpointers.com/post/c-c-standard-library</a> on Apr 9, 2018 by TRIANGLES。这篇文章介绍了C/C++标准库在标准中的说明及各平台的实现，作者查阅了很多资料，对梳理标准库概念有所帮助，最后一段还提到一个有意思的比赛 : )

本文简要介绍了C/C++开发领域中标准库所扮演的角色，以及标准库在各操作系统中是如何实现的。

使用C++有一段时间了，一开始经常让我困惑的是其*内部原理*：我使用的核心函数和类来自哪里？是谁写的？它们打包放在我电脑上什么地方？是否有一份*官方手册*？

本文通过介绍C/C++语言本身到其实现来回答这些问题。

# C/C++怎么产生的

平常我们谈论的C/C++其实是指一个*规则的集合*，它定义了语言该做什么、应该表现成怎样以及提供的功能。C/C++编译器必须遵从这些规则来处理源程序并生成二进制应用程序。这看起来与HTML挺相近：浏览器遵从一组指示符来渲染网页。

与HTML一样，C/C++规则也是*理论*上的。<a href="https://www.iso.org/" target="_blank">国际标准化组织(ISO)</a>中有一部分人每年聚集在一块讨论语言的规则并记入文档。是的，C和C++是经过标准化了的。最终他们会编写一本叫**标准**的官方手册，可以在官网上买到。每次随着语言发展，会出版一本新的标准文档。所以大家会看到不同的C/C++版本：C99, C11, C++03, C++11, C++14等等，其中的数字表示出版年份。

标准是份及其详细的技术文档：反正我不会拿它用于手头使用。标准分两部分：
<!-- more -->
1. C/C++特性和功能；
2. C/C++ API - 即开发者在C/C++程序中使用的一组类、函数和宏，称为**标准库**。

例如，以下摘自<a href="http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf" target="_blank">C标准</a>第一部分中对`main`函数的定义：

<p style="text-align: center">
  <img src="https://www.internalpointers.com/img/2018/01/main-implementation-c-standard.png">
  <br />
  1. 程序开头调用<code>main</code>函数的定义
</p>

下面的例子摘自上面同样的文档，对C中某个API的说明 - `fmin`函数：
<p style="text-align: center">
  <img src="https://www.internalpointers.com/img/2018/01/fmin-function-c-standard.png">
  <br />
  2. 头文件<code>math.h</code>中<code>fmin</code>函数的定义
</p>

可以看到，标准中几乎不会涉及到代码。人们得阅读标准文档，把它转成计算机能消化的东西。这也就是编译器和语言实现的开发者在做的：前者做个工具读入并处理C/C++源代码，后者将标准库转成代码。下面深入看一下。

### C标准库
**C标准库**也叫**ISO C库**，定义了一组宏、类型和函数，用于输入/输出处理、字符串处理、内存管理、数学计算以及许多与操作系统服务相关的。它们在C标准（如C11标准）中做了规定。内容分布在各个头文件中，比如上面提到的`math.h`。

### C++标准库
含义与C标准库相同，不过特定于C++。**C++标准库**包括一组C++模板，提供了常用的数据结构和函数，例如list, stack, array, algorithm, iterator及其他你可以想到的C++组件。C++标准库也囊括了C标准库，在C++标准（如C++11标准）中规定。

# 实现C和C++标准库

从这部分开始讨论真实的代码。实现标准库的开发者先阅读官方ISO文档需求，然后把它们转成代码。实现时必须依赖特定操作系统提供的功能（读/写文件，分配内存，创建线程等等，通过所谓的**系统调用**），因此每个平台都要有自己的一套标准库实现。有时它是系统的一个核心部分，有时由另外的组件（编译器）提供，这个必须另外下载。

### GNU/Linux的实现
**GNU C**库即**glibc**，是GNU项目对C标准库的实现。并不是所有的标准C函数都能在glibc中找到：大部分数学函数其实由**libm**实现，它是另外的一个库。

目前glibc是Linux上使用最广泛的C库。不过在90年代有个glibc的竞争者叫**Linux libc**（或者就叫**libc**），它fork自glibc 1.x。有段时间内，Linux libc是许多Linux发行版的标准C库。

经过多年的开发，glibc被证明远远优于Linux libc，曾经使用libc的Linux发行版都转成使用glibc了。所以要是你的磁盘上有个文件叫`libc.so.6`请不要惊慌，它是个现代的glibc。版本号涨到6是为了避免与之前Linux libc的版本混淆（不能把它命名成`glibc.so.6`，因为所有的Linux库命名都必须以`lib`开头）。

再来看看C++库，实现于**libstdc++**，或者叫**GNU标准C++库**。它是个一直持续着的项目，在GNU/Linux上实现了标准C++库。一般常见的Linux发行版默认使用libstdc++。

### Mac和iOS的实现
在Mac和iOS上，C标准库的实现是核心库**libSystem**的一部分，位于`/usr/lib/libSystem.dylib`。LibSystem还包括了数学库、线程库及其他底层组件。

至于C++标准库，Mac在OS X Mavericks(10.9 版本)前默认使用libstdc++，与现代Linux系操作系统使用相同的。自OS X Mavericks之后，苹果转而使用**libc++**，它由Mac官方编译器框架**LLVM项目**实现，用以替代GNU libstdc++标准库。

iOS开发者通过**iOS SDK(Software Development Kit)**使用标准库。iOS SDK是一组用于创建移动App的工具集。

### Windows的实现
Windows上标准库的实现与微软官方编译器**Visual Studio**严格绑在一起，称为**C/C++运行时库(CRT)**，它包括了标准库的实现。

最初将CRT实现为**CRTDLL.DLL**库（我猜那时不含C++标准库）。从Windows 95开始，微软把它实现在**MSVCR[版本号].DLL**（MSVCR20.DLL, MSVCR70.DLL等）中，猜测也包含了C++标准库。1997年左右，他们决定把文件名简化为**MSVCRT.DLL**，这不幸导致了臭名昭著的<a href="https://blogs.msdn.microsoft.com/oldnewthing/20140411-00/?p=1273" target="_blank">DLL mess</a>。这也就是为什么从Visual Studio 7.0后又转回每个版本搭载DLLs。

Visual Studio 2015对CRT做了大规模重构，C/C++标准库实现在一个新的库，名为**统一C运行时库(Universal CRT or UCRT)**，编译成**UCRTBASE.DLL**。UCRT目前已成为Windows的组件，从Windows 10开始作为操作系统的一部分。

### Android的实现
**Bionic**是谷歌为其安卓操作系统实现的C库，安卓系统内部直接使用它。第三方开发者可通过**Android Native Development Kit (NDK)**使用Bionic，NDK是一套工具集，允许开发者使用C和C++编写安卓应用。

C++方面，NDK提供了几种实现：
 - **libc++**，自从Lollipop（译者注：谷歌于2014年发布的第五代安卓系统）开始使用的安卓官方C++标准库，现代Mac操作系统上也使用的它。自NDK release 17开始它将成为NDK中唯一的C++标准库实现。

 - **gnustl**，是libstdc++的别名，在GNU/Linux上两者几乎相同。现在不建议使用它，在NDK release 18将被移除。

 - **STLport**，由STLport项目实现的第三方C++标准库，2008年后不再维护。与gnustl一样，在NDK release 18将被移除。

# 能用其他实现代替默认的吗？
如果你在一个资源极度受限的系统上做开发，可能你想用个不同的C标准库实现。<a href="https://uclibc-ng.org/" target="_blank">uClibc-ng</a>, <a href="http://www.musl-libc.org/" target="_blank">musl libc</a>和<a href="http://www.fefe.de/dietlibc/" target="_blank">diet libc</a>是其中一些，它们面向嵌入式Linux系统的开发，提供了更小的二进制文件和内存使用。

C++标准库也有其他的实现：<a href="http://stdcxx.apache.org/" target="_blank">Apache C++ Standard Library</a>, <a href="https://msharov.github.io/ustl/" target="_blank">uSTL</a>和<a href="https://github.com/electronicarts/EASTL" target="_blank">EASTL</a>是其中一些，后两个实际上只关注模板部分，并不是整个库，它们考虑的是运行速度。Apache版本聚焦在可移植性。

# 不用标准库会怎样？
不用标准库很简单：不要在程序中引进任何标准头文件就可以了。不过为了程序执行得有意义，必须或多或少地通过系统调用与操作系统交互。如之前所说，这是标准库中的函数/方法底层实现所用的方式。很可能得调汇编程序与硬件设备交互。

如果你对这些很兴奋，网上一些人则想着不用标准库来创建<a href="http://weeb.ddns.net/0/programming/c_without_standard_library_linux.txt" target="_blank">可用的程序</a>，这样的程序不具可移植性，因为依赖了特定操作系统提供的功能。但这种艰难会让你学到很多，你在使用高层库时将明白程序具体做了什么。

除了学习标准库，在嵌入式系统上工作时你不会想着使用标准库：内存受限，每个字节都要精打细算，所以往往要写很多汇编，反正代码不需要有可移植性。在<a href="https://en.wikipedia.org/wiki/Demoscene" target="_blank">demoscene</a>上，人们极力地把美妙的音视频塞进大小受限的二进制程序中 - 4K还不是最低界限：有的队伍拿1K、256B、64B甚至32B的参赛。在那可不允许使用标准库！

# 参考资料
Mac OS X for Unix Geeks - <em>5.2. The System Library: libSystem</em> (<a href="https://docstore.mik.ua/orelly/unix3/mac/ch05_02.htm" target="_blank">link</a>)<br>
libcxx.llvm.org - <em>"libc++" C++ Standard Library</em> (<a href="http://libcxx.llvm.org/" target="_blank">link</a>)<br>
blogs.msdn.microsoft.com - <em>Windows is not a Microsoft Visual C/C++ Run-Time delivery channel</em> (<a href="https://blogs.msdn.microsoft.com/oldnewthing/20140411-00/?p=1273" target="_blank">link</a>)<br>
malsmith.net - <em>A visual history of Visual C++</em> (<a href="http://www.malsmith.net/blog/visual-c-visual-history/" target="_blank">link</a>)<br>
docs.microsoft.com - <em>CRT Library Features</em> (<a href="https://docs.microsoft.com/en-us/cpp/c-runtime-library/crt-library-features" target="_blank">link</a>)<br>
MicrosoftDocs @ GitHub - <em>Upgrade your code to the Universal CRT</em> (<a href="https://github.com/MicrosoftDocs/cpp-docs/blob/master/docs/porting/upgrade-your-code-to-the-universal-crt.md" target="_blank">link</a>)<br>
Stackoverflow  - <em>What the role of libc(glibc) in our linux app?</em> (<a href="https://stackoverflow.com/questions/11372872/what-the-role-of-libcglibc-in-our-linux-app" target="_blank">link</a>)<br>
Stackoverflow  - <em>Where is the standard C library on Mac OS X?</em> (<a href="https://stackoverflow.com/questions/6240639/where-is-the-standard-c-library-on-mac-os-x" target="_blank">link</a>)<br>
Stackoverflow  - <em>When is it necessary to use use the flag -stdlib=libstdc++?</em> (<a href="https://stackoverflow.com/questions/19774778/when-is-it-necessary-to-use-use-the-flag-stdlib-libstdc" target="_blank">link</a>)<br>
Stackoverflow - <em>Where is the C/C++ Standard Library in Android and iOS?</em> (<a href="https://stackoverflow.com/questions/48067940/where-is-the-c-c-standard-library-in-android-and-ios" target="_blank">link</a>)<br>
Stackoverflow - <em>iOS help: math.h? Where is this?</em> (<a href="https://stackoverflow.com/questions/6613516/ios-help-math-h-where-is-this" target="_blank">link</a>)<br>
Stackoverflow - <em>What can you do in C without “std” includes? Are they part of “C,” or just libraries?</em> (<a href="https://stackoverflow.com/questions/2572988/what-can-you-do-in-c-without-std-includes-are-they-part-of-c-or-just-libra" target="_blank">link</a>)<br>
developer.android.com - <em>C++ Library Support</em> (<a href="https://developer.android.com/ndk/guides/cpp-support.html" target="_blank">link</a>)<br>
Wikipedia - <em>Bionic</em> (<a href="https://en.wikipedia.org/wiki/Bionic_(software" target="_blank">link</a>))<br>
Wikipedia - <em>C standard library</em> (<a href="https://en.wikipedia.org/wiki/C_standard_library" target="_blank">link</a>)<br>
Wikipedia - <em>GNU C Library</em> (<a href="https://en.wikipedia.org/wiki/GNU_C_Library" target="_blank">link</a>)<br>
Wikipedia - <em>Microsoft Windows library files</em> (<a href="https://en.wikipedia.org/wiki/Microsoft_Windows_library_files" target="_blank">link</a>)<br>
Wikipedia - <em>Demoscene</em> (<a href="https://en.wikipedia.org/wiki/Demoscene" target="_blank">link</a>)<br>
android.googlesource.com - <em>NDK Roadmap</em> (<a href="https://android.googlesource.com/platform/ndk/+/master/docs/Roadmap.md" target="_blank">link</a>)<br>
Reddit - <em>What is the relationship between gcc, libstdc++, glibc, binutils?</em> (<a href="https://www.reddit.com/r/linuxquestions/comments/1tghjd/what_is_the_relationship_between_gcc_libstdc/" target="_blank">link</a>)<br>
man7.org - <em>LIBC(7)</em> (<a href="http://man7.org/linux/man-pages/man7/libc.7.html" target="_blank">link</a>)<br>
gnu.org - <em>The GNU C Library (glibc)</em> (<a href="https://www.gnu.org/software/libc/" target="_blank">link</a>)<br>

(End)
