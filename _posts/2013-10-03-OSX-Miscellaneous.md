---
layout: post
title:  OS杂记[3]
categories:
- MAC
tags:
- OS X
- Eclispe
- Mavericks

---

昨天把Air的系统从10.8升级到了10.9，体验了一把Mavericks的高端，但随之而来的是JDK的不能使用。原本在10.8时候从Oracle上下载下来的JDK1.7安装后需要做一个文件的链接，Eclipse才能正常运转（提示需要JDK1.6），就是把安装在Library下得JDK链接到System下JavaVirtualMachines中去（注意在MAC中还有一个地方史有JDK链接的，位置在System下得Frameworks.app下）。写到这里就要感叹一句苹果下好混乱的JDK安装位置。升级到Mavericks后，启动Eclipse提示说是无法打开Eclipse，需要安装JDK1.6，顿时我就想到了第一次安装JDK的情景，我立马做了一次链接，但是还是不行，然后网上各种搜索，各种解决方式，有的说需要把Library下Java中的Info.plist修改的，在JVMCapabilities的key下增加一些对Apple的选项支持，有的说再次做链接，到System下得Frameworks.app下，还有的说在Eclipse的安装目录下得eclipse.ini中增加-vm选项，然后把系统中javaws的路径写上（比较奇怪的是在windows下这个javaws貌似叫做javaw.exe，名字居然不同）,但是在jdk的bin路径下居然没有找到这个javaws，然后我比较了一下System下Frameworks.app下JAVA的javaws（具体在/System/Library/Frameworks/JavaVM.framework/Versions/A/Commands目录下）,结果版本一致，于是就把Command目录下得所有文件copy到了/Library/Java/JavaVirtualMachines/jdk1.7.0_21.jdk/Contents/Home/bin目录下。结果仍然是不能执行，还有个The JVM shared library "/System/Library/Frameworks/JavaVM.framework" does not contain the JNI_CreateJavaVM symbol.这样的问题出现。各种纠结蛋碎的心都有了，果断安装个新的jdk吧，于是官网下了个jdk1.7.45的版本，然后安装，结果提示安装失败。。。。然后，就只有删掉新安装的jdk，然后再恢复之前所有的修改，果然错误又回到了提示无法打开eclipse，需要安装jdk1.6的问题上了。。。然后我也把JAVA_HOME添加到了PATH上，但是仍然问题继续。

后面想想eclipse启动，肯定有个地方指定所使用的jdk版本，不然就在PATH下面搜索，虽然我之前更改了PATH是无效的，但是肯定有个地方可以指定jdk的版本，既然在eclipse.ini中通过-vm无法指定，但是在cd /Applications/eclipse/Eclipse.app/Contents目录下还有一个Info.plist,打开一看，果然有个地方来设置-vm及其参数，果断修改，熟悉的eclipse果断出现。。。。