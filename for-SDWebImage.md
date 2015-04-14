#概述
简单来讲，SDWebImage是一个用来异步加载网络图片的库，在使用者的角度，设置url以及占位图片后，其他的就不用做了，SDWebImage自动开始下载图片，并且在图片下载完成之前在相应的位置显示占位图片。

#实现形式  
通过为预定义控件添加类别方法来实现，具体来说，为以下控件做了扩展：  
UIImage  
UIButton  
UIImageView  
MKAnnotationView  

#代码结构
如图：   
![如图:](https://github.com/klarm/blog/raw/master/res/SDWebImage_proj.png)


主要由下面几个类组成：  
- SDWebImageDownloader
图片下载器
- SDImageCache
图片cache（包括disk和memory的cache）
- SDWebImageManager
大内总管，扮演协调downloader和cache之间关系的角色

#主要类的设计
SDWebImageManager主要成员：  
- SDWebImageDownloader *imageDownloader管理下载  
- SDImageCache *imageCache管理缓存  
- id <SDWebImageManagerDelegate> delegate回调通知

SDWebImageDownloader主要成员：  
- NSOperationQueue *downloadQueue存放下载任务队列
（SDWebImageDownloaderOperation派生自NSOperation）

SDImageCache主要成员：  
- NSCache *memCache存放内存cache  
- NSString *diskCachePath存放磁盘cache的路径  
- dispatch_queue_t ioQueue表示io队列

#几个关键点
- 单例的实现
SDWebImage中SDWebImageManager、SDWebImageManager和SDImageCache都是以单例的形式存在，实现上用了类似的方法：  
	+ (id)sharedManager {
    	static dispatch_once_t once;
    	static id instance;
    	dispatch_once(&once, ^{
        	instance = [self new];
    	});
    	return instance;
	}
简单来说，就是通过dispatch_once来实现单例。这种方法利用了dispatch_once可以保证block里的代码只执行一次，并且是线程安全的特性。[可以参考这里。](http://blog.csdn.net/ryantang03/article/details/8622415)

- 磁盘cache的实现  
这里使用了UIImagePNGRepresentation和UIImageJPEGRepresentation读取图片的data，保存在NSData中，然后通过createFileAtPath保存在磁盘中。




