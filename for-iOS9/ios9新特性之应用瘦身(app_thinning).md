##iOS9新特性之应用瘦身(App thinning)
###概述
针对app体积越来越大的问题，iOS9引入了3种新技术：  
* 应用切片（App  Slicing）  
* 资源按需加载（On Demand Resources）
* Bitcode
###应用切片(App Slicing)：
现在的app为了兼容不同设备，同时包含了32 bit 和 64 bit 两个 slice；图片资源方面，1x，2x、3x的资源一应俱全，但是对具体用户来讲，其设备是特定的，为什么要让只需要3x分辨率的设备去下载2x和1x资源？为什么要让只能运行32bit的设备下载64bit的Architecture？
App Slicing的直观理解，就是在开发者上传的完整包中，根据设备情况切取最小可用的单元，并部署到实际设备，如下图所示： 
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/AppThinng1.png)

iOS9之前的app的完整包的组成如下图所示，对具体设备而言，存在冗余内容：
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/AppThinng2.png)
以iPhone 6 Plus为例，下图说明了它实际需要的内容，除此之外的都是冗余的内容：
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/AppThinng3.png)
有了App Slices，开发者就可以根据设备添加资产标签，当用户从iTunes下载应用时，它将仅下载你的设备需要的资产。因为苹果已经将整个过程设计得非常简单，所以相信很多应用很快就会开始支持这项特性。

以前做资源的延迟下载，一般都是自己搭建server放置那些app以后用到的资源，然后在某个时机触发多线程下载，之后进行后续的处理。现在Apple用自己的server来实现这个功能， 不需要自己搭建server等繁琐的一套，只需要给资源打上tag，剩下的工作apple帮你完成。（XCode7有相应的支持，个人理解就是通过tag，进行分组下载）

应用场景：•	初始资源的延迟加载。app有一些资源是主要功能要用到的，但在启动时并不需要。将这些资源标记为“初始需要”。操作系统在app启动时会自动下载这些资源。例如，图片编辑app有许多不常用的滤镜。•	app资源的延迟加载。app有一些只在特定情景下使用的资源，当应用可能要进入这些场景时，会请求这些资源。例如，在一个有很多关卡的游戏中，用户只需要当前关卡和下一关卡的资源。 •	不常用资源的远程存储。app有一些很少使用的资源，当需要这些资源时会去请求它们。例如，当app第一次打开时会展示一个教程，而这个教程之后就可能不会在用到。app在第一次启动时请求教程的资源，这之后只在需要展示教程或者添加了新功能才去请求该资源。•	应用内购买资源的远程存储。app提供包含额外资源的应用内购买。app会在启动完成后请求已购买模块的资源。例如，用户在一个键盘app内购买的表情包。应用程序会在启动完成后请求表情包的资源。•	第一次启动时必需资源的加载。app有一些资源只在第一次启动时需要，之后的启动不再需要。例如，app有一个只在第一次启动时展示的教程。App包含一系列的资源，并且根据程序的执行流程，可以分级，如下图：
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/AppThinng5.png)
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/AppThinng6.png)资源按需加载的思想是，最初安装到设备上的程序只包含必要的资源，非必要的资源放在云端，如下图：
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/AppThinng9.png)
按需加载资源的流程：
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/AppThinng7.png)
###如何使用该特性：
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/AppThinng8.png)
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/AppThinng.png)
tag的分类：Initial install tags：该tag的资源与app同时下载Prefetch tag order：app安装后开始下载Dowloaded only on demand：需要显式请求、并且无cache时才开始下载
涉及到的类：NSBundleResourceRequestNSBundle 使用流程：•	分配NSBundleResourceRequest，根据tag发出资源请求；•	开始下载，监测进度与下载过程中可能出现的错误；•	设置资源在本地保存的优先级；•	使用资源；•	告知系统不再需要资源，以便系统在必要的时候释放空间。何时开始发起下载：不幸的是，在这方面系统能做的不多，大体上需要开发者根据程序逻辑自己决定，系统提供了以下有限的几个影响资源下载的方式：•	在下载开始前设置下载优先级；•	监控下载过程。其一，开发者可以根据下载情况动态调整优先级，其二，可以通过UI将进度反馈给用户###几个要注意的点：
处理空间不足的警告[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(lowDiskSpace:) name:NSBundleResourceRequestLowDiskSpaceNotification object:nil];注意应对tag所标示的资源的各种可能的状态
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/AppThinng.png)
好处：•	提高存储空间的应用效率：比如多级游戏，用户需要的通常都是他们当前的级数以及下一级。ODR意味着用户可以下载他们需要的几级游戏。随着你的级数不断增加，应用再下载其他级数，并将用户成功过关的级数给删掉；•	对于延迟下载的资料，不用搭建自己的服务器，这点对于个人开发者更重要。不足：•	需要自己控制下载时机，处理下载过程中出现的各种情况；•	消耗更多移动流量；•	在使用app的时候，如果触发了较多的资源下载，用户需要等待，甚至block住。比如你坐十几个小时飞机，期间一直在玩游戏，不小心你就一路过关斩将，但是因为没有下载所以不能继续玩下去，这种时候会非常无奈；•	网络的不可控性，特别是app store的访问速度。Bitcode•	开发者上传应用程序时提交的是中间码，App Slicing可以根据用户需求，来判断是需要32位还是64位。•	也就是说，在用户下载应用之前，App Store在自动编译应用程序。这样，即使开发者没有给代码添加标签，应用也能够执行App Slicing的部分功能，仅下载设备需要的32或64位代码。•	也意味着如果苹果完善编译器提高代码效率，用户下载应用时苹果进行的完善会自动整合进去。