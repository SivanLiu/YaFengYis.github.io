---
title: Android反编译工具
date: 2016/8/29 1:12:42 
tags:
---

本文主要介绍当下比较流行的几种反编译Android apk的方式，包括：

**1. apktool和dex2jar**
 
[apk官方地址](https://ibotpeaches.github.io/Apktool/)：解压apk后便于资源文件获取，可以提取出图片文件和布局文件进行使用查看。

[dex2jar](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/innlab/dex2jar-0.0.7.11-SNAPSHOT.zip)：将apk反编译成java源码（classes.dex转化成jar文件）

[jd-gui](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/innlab/jd-gui-0.3.3.windows.zip):查看APK中classes.dex转化成出的jar文件，即源码文件

以QQ为例，反编译过程如下：

1）反编译apk得到程序的源代码、图片、XML配置、语言资源等文件:
Ctrl+R ——> cmd，定位到apktool文件夹，根据apktool d -f [apk文件] [输出文件夹]）输入以下命令：

    E:\Assemble\apktool>apktool d E:\Assemble\com.tencent.mobileqq_6.5.3_liqucn.com.apk 
qq文件中的内容如下，可以看到QQ反编译是不成功的，只能部分反编译：

![](http://ococzubug.bkt.clouddn.com/1.png?attname=&e=1472484427&token=Eyl9DyX0WeWOeyWwoCcYlqx5dmMI4_EgUq5C2s8G:51w5dCIYuFpKG_2foGGFegwIBN0)

2）Apk反编译得到Java源代码，将要反编译的APK后缀名改为.rar或则 .zip，并解压得到其中的classes.dex文件（该文件就是java文件编译再通过dx工具打包而成的），利用dex2jar工具将获取到的classes.dex进行反编译，命令如下：

    E:\Assemble\dex2jar-2.0>d2j-dex2jar.bat E:\Assemble\com.tencent.mobileqq_6.5.3_liqucn.com\classes.dex
最后生成的文件目录如下，其中包含dex文件和各种资源文件：

![](http://ococzubug.bkt.clouddn.com/2.png?attname=&e=1472484427&token=Eyl9DyX0WeWOeyWwoCcYlqx5dmMI4_EgUq5C2s8G:vMCH4aRj2SGTz_Eg0pg27O0d9cc)

3）经过第2步之后便会生成一个classes_dex2jar.jar的文件，利用jd-gui工具打开该文件，便可以看到源码了，效果如下：

![](http://ococzubug.bkt.clouddn.com/3.png?attname=&e=1472484427&token=Eyl9DyX0WeWOeyWwoCcYlqx5dmMI4_EgUq5C2s8G:iiBXZkkbfFhK86jINALbS1QJSe0)

**2. [jadx](https://github.com/skylot/jadx)**

前些天在github上寻觅到了一个新的反编译工具Jadx，方便实用，用法超级简单，命令行直接搞定，个人认为比apktool+dex2jar+jd-gui要方便很多；jadx可以直接反编译.apk文件，也可以反编译解压出的classes.dex文件。使用过程如下：

1）安装：
[直接下载jadx,解压](https://github.com/skylot/jadx/archive/master.zip)

解压后通过cmd命令行cd到jadx目录中，执行gradlew dist命令（针对windows），gradlew任务执行完毕之后，会生成一个build文件夹,至此安装完毕。

2）直接反编译.apk文件

进入到jadx\build\jadx\bin中：

执行反编译apk命令：

    E:\Assemble\jadx\build\jadx\bin>jadx-gui E:\Assemble\com.tencent.mobileqq_6.5.3_
liqucn.com.apk

查看dex文件命令：

    E:\Assemble\jadx\build\jadx\bin>jadx-gui E:\Assemble\com.tencent.mobileqq_6.5.3_
liqucn.com\classes.dex

结果和jd-gui一样，但是jadx功能更加丰富，如下

![](http://ococzubug.bkt.clouddn.com/4.png?attname=&e=1472484427&token=Eyl9DyX0WeWOeyWwoCcYlqx5dmMI4_EgUq5C2s8G:5ePdkks2E_5EFQXClek0dqtzsdQ)

**3. [Androguard](https://code.google.com/archive/p/androguard/)**

1)使用DAD作为反编译器

2)可以分析恶意软件

3)有python api，可以写扩展

4)支持可视化 

**4. [Enjarfy](https://github.com/google/enjarify)**

enjarify是由Google官方新出品的基于Python3开发，类似dex2jar的一个将Dalvik字节码转换成相对应的Java字节码开源工具，官方宣称有比dex2jar更优秀的兼容性，准确性及更高的效率。

1)可以将dalvik bytecode转化为java bytecode

2)比dex2jar支持case更多

**5. [JEB](https://www.pnfsoftware.com/)**

 JEB是一个功能强大的为安全专业人士设计的Android应用程序的反编译工具。用于逆向工程或审计APK文件，可以提高效率减少许多工程师的分析时间。

1)全面的Dalvik反编译器:JEB的独特功能是，其Dalvik字节码反编译为Java源代码的能力。无需DEX-JAR转换工具。我们公司内部的反编译器需要考虑的Dalvik的细微之处，并明智地使用目前在DEX文件的元数据。

2)交互性：分析师需要灵活的工具，特别是当他们处理混淆的或受保护的代码块。JEB的强大的用户界面，使您可以检查交叉引用，重命名的方法，字段，类，代码和数据之间导航，做笔记，添加注释，以及更多。

3)可全面测试APK文件内容：检查解压缩的资源和资产，证书，字符串和常量等。追踪您的进展情况。
不要让研究的工时浪费。

4)支持多平台（Win/Linux/Mac）

**6. [codeinspect](https://codeinspect.sit.fraunhofer.de/)**

CodeInspect是由德国Paderborn University和TU Darmstadt的软件安全团队研发的一款基于Jimple的Android应用分析工具。CodeInspect的软件是Eclipse框架开发的，所以我们可以像使用IDE一样对Android应用进行分析和调试，也可以直接插入或者修改Java代码。
CodeInspect是一款强大的分析工具，拥有源码级别的调试和修改功能，当然也有一些缺点，比如某些加固过的APK无法正常反编译，没有类似JEB的JAVA级别的代码，不支持Native层的调试等。

**7. [bytecodeviewer](https://github.com/konloch/bytecode-viewer/releases)**

BytecodeViewer是一个先进的轻量级Java字节码查看器，它是一款基于图形界面的Java反编译器，Java字节码编辑器，APK编辑器，Dex编辑器，APK反编译器，DEX反编译器，Procyon Java反编译器，CFR Java反编译器，以及FernFlower Java反编译器。不仅如此，它还是一款Hex查看器，代码搜索器和代码调试器。除此之外，它还具备Smali和Baksmali等汇编器的相关功能。

**8. [ClassyShark](https://github.com/google/android-classyshark)**

ClassyShark是一款可以查看Android可执行文件的浏览工具，支持.dex, .aar, .so, .apk, .jar, .class, .xml 等文件格式，分析里面的内容包括classes.dex文件，包、方法数量、类、字符串、使用的NativeLibrary等。

参考网址：

[https://github.com/Juude/droidReverse](https://github.com/Juude/droidReverse "https://github.com/Juude/droidReverse")

