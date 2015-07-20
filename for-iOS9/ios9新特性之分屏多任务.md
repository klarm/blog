##iOS9新特性之分屏多任务
###概述
分屏多任务也许是iOS9最大的变化之一（目前只支持iPad Air 2），从具体形态上来说，其有3种表现形式：    
* 滑动覆盖 (Slide Over)：临时调出另一个程序，如写文档的过程中，临时弹出聊天窗口，短暂划出后关闭  * 分割视图 (Split View)：真正的多任务，两个程序同时激活，两边的任务都可以进行实时操作，并且分屏比例可以调节* 画中画模式 (Picture in Picture)：视频在一个悬浮小窗口中播放，类似于以前的弹窗播放  ###背景知识
屏幕适配经历了代码计算frame -> auto resizing(父控件和子控件的关系ios6) -> auto layout(任何控件都可以产生关系ios7) 的演变过程，为了解决适配更丰富屏幕的问题，在iOS8中，引入了一个全新的概念，这就是Size Class，不再根据设备屏幕的具体尺寸来进行区分，也淡化了横屏和竖屏的概念，而是通过它们的感官表现，在每个方向将其分为普通 (Regular) 和紧密(Compact) 两个类别，主流设备在不同方向下的分类如下：  
iPhone4S,iPhone5/5s,iPhone6  竖屏：(w:Compact h:Regular)横屏：(w:Compact h:Compact)
iPhone6 Plus  竖屏：(w:Compact h:Regular)横屏：(w:Regular h:Compact)  iPad竖屏：(w:Regular h:Regular)横屏：(w:Regular h:Regular)
上述可以总结为下图：  
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/%E5%B1%8F%E5%B9%95%E5%88%92%E5%88%86.png)
在iPad上，iOS 9引入分割视图以后，可以通过调节分割视图的分割条来调节各子区域的大小。设备在横屏或竖屏的情况下，分割条拖到不同的位置，各区域的Size Class定义如下图所示：  
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/%E5%B1%8F%E5%B9%95%E5%88%92%E5%88%862.png)
引入Size Class最大的好处：1、所有不同设备屏幕统一；2、所有改变Window大小的行为统一（滑动分割窗口、旋转设备方向都统一为Size Class的变化）；在上述思想下，  - willRotateToInterfaceOrientation:duration:- willAnimateRotationToInterfaceOrientation:duration:- didRotateFromInterfaceOrientation:- shouldAutomaticallyForwardRotationMethods等与旋转有关的方法都将在8.0之后废弃，全部统一到-willTransitionToTraitCollection:withTransitionCoordinator: 和 -viewWillTransitionToSize:withTransitionCoordinator:，今后在程序代码层面将不再有旋转的概念，旋转与设备尺寸变化及分割窗口尺寸变化统一归结为Size Class的变化，这也是Size Class的设计哲学。  
###滑动覆盖 (Slide Over)、分割视图 (Split View)的适配方法  
官方文档：“Most apps should adopt Slide Over and Split View” 
即在iOS9时代，滑动覆盖和分割视图是很基本的特性，多数app需要支持。  
下图显示了旋转屏幕的过程中， ViewController收到的一系列的消息
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/%E6%97%8B%E8%BD%AC%E5%B1%8F%E5%B9%95%E7%9A%84%E5%9B%9E%E8%B0%83.png)
注意其中的Size Class变化对应的消息。 

开发者根据当前工作的 UIViewController 的 traitCollection 属性来设置合适的布局，并且在 -willTransitionToTraitCollection:withTransitionCoordinator: 和 -viewWillTransitionToSize:withTransitionCoordinator: 被调用时对 UI 布局做出正确的响应。如下图所示：
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/%E5%93%8D%E5%BA%94%E5%B1%8F%E5%B9%95%E5%8F%98%E5%8C%96.png)
###需要注意的问题
* 在分屏的情况下UIScreen.bounds和UIWindow.bounds可能不同，后者可能是前者的一半或者三分之一。* iPad的Size Class可能会发生变化。在iOS 8时代，iPad不论横屏还是竖屏，长宽都是Regular的，但是在iOS 9引入分屏之后，就会有多种情况（见上图）。总之，iOS 9时代不要对Size Class和Window Size做任何假定，因为你的app可能被作为从属app打开，你的app也可以作为主app打开其它从属app，并且在上述过程中用户还可以进行拖动，导致Size Class有各种组合，解决的办法是使你的app支持所有方向和相应方向对应的Size Class的组合。![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/guideline.png)
* 分屏情况下，用户移动分割条，系统回调delegate的applicationWillResignActive:方法，注意在这个过程中不要丢失相关信息，保证用户释放分隔条时，如果app窗口大小和原来一致，程序的状态（文本选择范围、滚动条位置等）和最初一致。* 分屏情况下，如果用户将分隔条从左侧滑动到最右侧，系统回调被完全覆盖的app的delegate的applicationDidEnterBackground:方法。
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/best%20practice.png)
###画中画模式 (Picture in Picture)
官方文档：“Picture in Picture (PiP) is for apps whose primary role is video playback. And PiP is best for playback of medium-to-long duration content”
我的理解是PiP特性需要根据app所在的行业特点来决定是否应该支持，对于播放类的应用，最好都能支持。 

在iOS9中，下面3个框架支持画中画模式： ![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/PiP1.png)
具体来说，根据下面的步骤来适配：
1、使用iOS 9 SDK；2、在工程配置中，打开后台模式；![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/PiP2.png)3、将AudioSession的Catogory 设置为合适的选项，比如 AVAudioSessionCategoryPlayback；4、如果使用的是AVPlayerViewController来播放视频，上述配置后进入全屏播放就支持画中画模式；5、如果使用AVPlayerLayer来构造自定义播放界面，需要使用AVPlayerLayer 来创建一个 AVPictureInPictureController：![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/PiP3.png)
AVPictureInPictureController提供了判断是否支持画中画模式、进入画中画模式、退出画中画模式的API：
![](https://raw.githubusercontent.com/klarm/blog/master/for-iOS9/PiP4.png)此外，AVPictureInPictureControllerDelegate还提供了一些回调方法来获取画中画的执行情况，如：pictureInPictureControllerDidStartPictureInPicturepictureInPictureControllerFailedToStartPictureInPicturepictureInPictureControllerWillStopPictureInPicture可以根据这些回调通知做一些业务性的功能。