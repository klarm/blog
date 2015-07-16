##概述
列表视图的下拉刷新是个很常见的功能，github上有很多开源的解决方案，EGOTableViewPullRefresh是其中使用很多的一个，下面来分析它的实现。
##UI组成
整体效果图如下图所示： 
![整体效果图]
(https://github.com/klarm/blog/blob/master/for-PullRefresh/%E6%95%B4%E4%BD%93%E7%A4%BA%E6%84%8F%E5%9B%BE.png)  
可以分为两个部分，列表视图部分和刷新指示视图部分。  
从UI结构上讲，没什么特别的，就是普通的subView关系：  
```objc
if (_refreshHeaderView == nil) {
		
		EGORefreshTableHeaderView *view = [[EGORefreshTableHeaderView alloc] initWithFrame:CGRectMake(0.0f, 0.0f - self.tableView.bounds.size.height, self.view.frame.size.width, self.tableView.bounds.size.height)];
		view.delegate = self;
		[self.tableView addSubview:view];
		_refreshHeaderView = view;
		[view release];
	}
```  
但怎么样实现在正常情况下，刷新指示视图不显示，在列表顶部下拉才显示的效果呢？关键在于坐标为负值的frame。从上面的代码来看，整个刷新指示视图在列表视图的正上方，大小和列表视图一样。继续看代码：
```objc
UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(0.0f, frame.size.height - 30.0f, self.frame.size.width, 20.0f)];

label = [[UILabel alloc] initWithFrame:CGRectMake(0.0f, frame.size.height - 48.0f, self.frame.size.width, 20.0f)];

layer.frame = CGRectMake(25.0f, frame.size.height - 65.0f, 30.0f, 55.0f);
```
发现刷新指示视图中，真正的内容部分，即显示提示文字和上次更新时间的lable、显示箭头的layer在刷新指示视图的下方，如下图所示：  
![整体示意图]
(https://github.com/klarm/blog/blob/master/for-PullRefresh/EGOTableViewPullRefresh%E8%A7%A3%E6%9E%90.png)  
那下拉的过程中，可视区域外的刷新指示视图是怎么显示出来的呢，加一条log：NSLog(@"scrollView.contentOffset.y=%f", scrollView.contentOffset.y);  
发现在列表下拉的过程中，刚好拉倒列表顶部的时候，scrollView.contentOffset.y为0，继续往下拉，scrollView.contentOffset.y变为负值，相对于scroolView的frame在y轴方向上为负的子视图就显示出来了。


##状态转换
代码里定义了几种状态  
```   
typedef enum{
	EGOOPullRefreshPulling = 0,
	EGOOPullRefreshNormal,
	EGOOPullRefreshLoading,	
} EGOPullRefreshState;
``` 
跟了一下，一个完整的刷新过程中的状态变化为normal-》pulling-》loading-》normal，如下图：
![状态转换示意图]
(https://github.com/klarm/blog/blob/master/for-PullRefresh/EGOTableViewPullRefresh%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2.png)   
这个也好理解，正常状态是normal，下来满足一定条件(下拉偏移量大于65)，进入pulling状态，触发刷新，进入loading状态，loading结束后，恢复normal状态。这里的关键是触发这些状态转换过程的时机：  
* normal状态-》pulling状态  开始滚动并且偏移量大于一定的负值，具体来讲用的是scrollViewDidScroll这个时机
* pulling状态-》loading状态  结束滚动并且偏移量大于一定的负值，具体来讲用的是scrollViewDidEndDragging这个时机
* loading状态-》normal状态 业务代码中刷新结束后，重置状态

每种状态下，需要改变刷新指示视图中文本及动画的状态，这个不赘述。

##总结
总体来讲EGOTableViewPullRefresh的代码还是挺简单的，关键在于上述两点：滚动视图中，frame坐标为负值的运用；利用滚动事件的回调，处理好状态转换。

	