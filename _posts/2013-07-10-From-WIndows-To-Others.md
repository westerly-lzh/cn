---
layout: post
title: OS杂记[1]
categories:
- OS
tags:
- Linux
- Mac
- Windows

---

####1.制作USB启动盘
最近在学校机房做作业，大家也都知道国内学校的那些机房，都是windows的天下，而且每台电脑上还都安装上了还原精灵之类的还原卡，貌似很方便的样子，我现在遇到的问题是，前几天把机房中一台台式机给换成了Ubuntu系统（前提是把主机的还原卡给拆下了），用的挺爽得，后面有一天脑子抽风，又把还原卡给装上了，一启动，这还原卡果然威武，立马就把我这Ubuntu系统的引导分区给覆盖了，可怜我好不容易配好的Ubuntu立马就进不去了。现在手头又没有Ubuntu的安装光盘，只有制作一个usb的启动盘来恢复引导区了。

自己的电脑是用的mac系统，按照网络上的说法，可以使用下面的命令过程来实现一个usb盘的制作。

1. 将iso转换为img文件

	$ hdiutil convert -format UDRW -o /path/to/generate/img/file /path/to/your/iso/file 
	
	该命令会生成一个.img的磁盘镜像文件，但是mac osx会默认追加一个.dmg，即生成的文件后缀是.img.dmg，这个后缀没关系，可以忽略
	
2. 查看USB的盘符

	$ diskutil list

	该命令查看当前系统上挂载的磁盘，其中/dev/disk1是我的USB磁盘。不同的系统disk后的数字可能不一样，但一般都是diskN的模式

3. 卸载USB磁盘

	$ diskutil unmountDisk /dev/disk1
	
	Unmount of all volumes on disk1 was successful
	使用diskutil unmountDisk卸载USB磁盘，注意卸载（umount）与弹出(eject)的区别:)

4. 将镜像写入USB

	$ sudo dd if=ubuntu.img.dmg of=/dev/rdisk1 bs=1m
	将第二步生成的img文件写入到USB磁盘/dev/rdisk1。

5. 弹出USB

	$ diskutil eject /dev/disk1
	
这个过程在ubuntu下如下：

假如u盘挂在/media/jeff/mydisk上

1. 卸载

	sudo unmount /media/jeff/mydisk
	
2. 查看u盘位置

	sudo fdisk -l
	
	发现u盘挂在/dev/sdb位置
	
3. 格式化
	
	sudo kmfs.vfat -I /dev/sdb
	其他格式
	在对u盘进行格式化时，可以选择其他格式，如: mkfs.ext2,mkfs.ext3,mkfs.ext4,mkfs.ntfs…
	
4. 将镜像写入U盘

	sudo dd if=/path/to/file of=/path/to/usb
	当然也可以使用ubuntu上自带的可视化工具进行
	
在windows上制作usb启动盘
这个过程比较简单，使用一个UltraISO工具就可，十分简单。

上面在mac系统上制作的usb启动盘，在其他的非mac系统上根本不识别，这是我后面才发现的，所以才有了在ubuntu上制作的经历，而windows上制作usb启动盘，是我最先接触的，而且那个UltraISO工具在我使用windows系统的那会儿一直是我的装机必备工具。

下面的过程就是把usb启动盘插在我们需要的电脑上启动了。现在的主板一般都是支持usb启动的，但也有一些比较变态的主板，在启动设备上会提供usb-fdd,usb-cdrom,usb-zip,usb-hdd等很多选项，这个大家就要一个个的试了，看看我们制作的usb启动盘能在那种设备模式下启动。比较可惜的是我制作出来的usb启动盘在所有模式下都无法启动。


####2.日常工具的非Windows替代版
这几年Ubuntu发展的如火如荼，其下的软件已经非常丰富，现在从windows阵营转移到Linux阵营已经没有什么障碍了(网银除外)。就我自己现在的体验来说，在Ubuntu下日常使用及办公已经没有什么问题，而且现在Ubuntu下得OpenOffice也是兼容Windows的Office系列的。我本人不怎么玩游戏，所以在国内这些严重依赖Windows系统的游戏方面，我没有什么障碍。学习上需要的软件，基本上Windows下的收费系列我在Ubuntu下基本有开源免费的替代品，这对于我们这些苦逼学生族来说是比较好的，主要是现在很多人的版权意识都已经形成了，实在不好意思总是对别人说我是用的破解版吧。我这里罗列一些自己常用的软件在windows，linux和mac下不同的替代品

 Windows下	| Linux下 | Mac下
 ---		| ---	  | ---
 Office		| OpenOffice| iWork
 QQ	/Skype	   | QQ或者Longene-QQ/Skype|QQ/Skype
 IE/Chrome/Firefox | Chrome/FireFox|Safari/Chrome/Firefox
 SPSS/Matlab/SAS |R/Octave|R/Octave
 迅雷/快车	|uget/aria2|iGetter/迅雷
 有道/灵格斯字典|goldenDict | 词典
 
 很多人不用Linux是因为他们的commandline，但是如果纯粹是为了娱乐，那么，确实老老实实的呆在Windows下，可以有很多好玩的，但如果是为了工作学习，我觉得还是尽早转到Linux下吧，那些小命令和方便的小工具能极大的提高我们的效率。
 
 
	
