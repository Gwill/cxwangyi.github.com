---
layout: post
title: "Install Hadoop YARN 2.2.0 on MacOS X Mavericks"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Go语言里有个有意思的话题，是如何用Go语言写GUI程序。因为Go的标准packages没有用来写GUI程序的，所以很多人在问，能否有一个标准GUI package？

2013年6月，有一个Go Authors的访谈。其中也有人问到上述问题。Rob Pike直接提到他年轻的时候设计的Newsqueak语言，一个专门用来写GUI程序的语言。Andrew Gerrand赶紧把话题拉回来，介绍了几个他看到的比较好的第三方GUI packages。这段访谈的缩减版可以在Go的主页里找到：http://blog.golang.org/a-conversation-with-the-go-team

我用过几个比较方便的GUI toolkit，包括GTK和Qt。根据这么点儿经验，我赞同Rob Pike对Go语言在GUI编程中的前景：

1. Go支持memory garbage collection。这样可以方便的管理widgets的memory消耗。用过GTK的人应该都被迫理解基于C设计的floating reference机制吧？

1. Go的concurrency应该能方便的让GUI不被后台处理拖累，保持界面反应的流畅。大家还记得当年BeOS（要不是Jobs介入，Apple本来要收购的；最后收购了NeXT）吗？它的一个重要特点就是使用multithreading技术保证GUI总是有反应。

虽然Andrew Gerrand当时举得例子go.uik的作者已经[把项目关掉了](https://github.com/skelterjohn/go.uik)，并且号称“如果再打开，一定要重写”：

> This project is closed. If I begin again, it will be from scratch and using lessons learned while writing go.uik. Of course, if I begin again it will still be called go.uik.

但是其他项目层出不穷。我上周在我的iMac（Mavericks）上尝试了[go-qt5](https://github.com/salviati/go-qt5)，这是一个Qt 5的Go语言binding。今天刚刚尝试了[gotk3](https://github.com/conformal/gotk3)，一个完全用Go的内存管理方式（idiomatic）写的GTK的binding。感觉都很不错。因为这两个项目都在快速演进中，它们主页上介绍的用法和实际操作略有不同。这里列一下我的操作。

## go-qt5

主页上的tutorial在我的环境下不完全好使，我的总结在[这里](http://cxwangyi.wordpress.com/2014/01/13/programming-qt-5-using-go/)。或者简单记录操作流程如下：

1. 用Homebrew安装5.2：
      
        brew update && brew doctor && brew install qt5

   这会提醒你说“目前世界上大部分人都在用Qt4，要不不要这么激进用Qt5啊？”（Homebrew真的很人性化。）但是很不好意思的，Qt4不支持最新版本的OS X Mavericks。
   
1. 检出go-qt5代码：

        mkdir -p /home/you/go-qt5
        export GOPATH=/home/you/go-qt5
        go get github.com/salviati/go-qt5
        
1. Build go-qt5

基本上可以按照go-qt5的[README](https://github.com/salviati/go-qt5)来操作。有几点要注意：因为Homebrew不会在/usr/local/bin下面创建指向Qt 5的symbolic links，所以你调用qmake的时候可能需要用全路径名：`/usr/local/Cellar/qt5/5.2.0/bin/qmake`。另外，运行自己写的Qt程序的时候，要记得把链接库`$GOPATH/src/github.com/salviati/go-qt5/lib/libgoqt5drv.1.0.0.dylib`和你的程序放在同一个目录下。

## gotk3

在Mac OS X上得安装和使用基本上就根据[这篇文章](https://github.com/conformal/gotk3/wiki/Installing-on-OSX)来，但是有两点需要注意：

如果build GTK3的时候，报错找不到`-lpython2.7`，可以通过LDFLAGS环境变量告诉jhbuild Python的库在哪里。比如：

    LDFLAGS=-L/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.8.sdk/usr/lib/ jhbuild build meta-gtk-osx-gtk3
    
另外，编译gotk3的时候，不需要用jhbuild shell启动一个新的shell，只需要设置一下环境变量：

    export PKG_CONFIG_PATH=/Users/wangyi/gtk/inst/lib/pkgconfig:/Users/wangyi/gtk/inst/share/pkgconfig:/usr/lib/pkgconfig    
    
然后就可以通过go get下载和编译gotk3了。

    go get github.com/conformal/gotk3/gtk
    
    