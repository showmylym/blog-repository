---
layout: post
title: "图标管理控件的实现-类似于iOS主页的效果"
date: 2015-01-06 08:22:12 +0800
comments: true
categories: 
---
　　前段时间因工作需要，写了一个图标管理控件。日常开发中可能类似的需求不少，于是做成可复用的代码库，当作编写可复用控件的开端。    
　　　　　　　　　　　　　　　　　　　　　　　　　　　－－题记    
　　这个图标管理控件我为其命名为DraggableView，意为可拖动视图，它继承于UIView，与UITableView类似，它有自己的cell，cell中实现长按拖动(LongPressGesture)、点击选择(TapGesture)，以及删除功能。    
　　以下是控件演示：    
　　<!--embed src="http://player.youku.com/player.php/Type/Folder/Fid/23308555/Ob/1/sid/XODY0NTg4NTAw/v.swf" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" allowFullScreen="true" mode="transparent" type="application/x-shockwave-flash"></embed-->    
　　纵观iOS App市场，有类似实现效果的应用不少，如下截图：    
　　[图片2-1](http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/2/2-1.PNG "图片2-1")    
　　[图片2-2](http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/2/2-2.PNG "图片2-2")    
　　[图片2-3](http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/2/2-3.PNG "图片2-3")    
　　[图片2-4](http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/2/2-4.PNG "图片2-4")    
　　大致工作流程为：    
　　1、点击可选择，类似于UITableView的didSelect方法。    
　　2、长按一个cell，触发编辑状态。编辑状态的cell将放大显示，所有图标将以角度3°轻微旋转，表示当前的拖动编辑状态。处于编辑状态的cell可以被删除、被拖动重排序，但不能触发点击回调，两种操作都有回调暴露，在代理对象中编写具体实现。    
　　3、若数据源层级中添加了新的cell，只需[draggableView reloadData]调用一次即可刷新UI。刷新完成时，draggableView向代理对象发送didResizeWithFrame消息，代理实现可根据此回调适时改变视图中UIScrollView的contentSize，适应新内容大小。
　　大致使用流程为：    
　　1、初始化：    
　　`self.mainDraggableView = [[LYMDraggableView alloc] initWithFrame:CGRectMake(0.0, 0.0, self.screenSize.width, 0.0) layoutType:LYMDraggableViewLayoutByColumnNum horizontalMargin:0.0 verticalMargin:0.0 vSpace:4.0 maxColumn:4.0 cornerRadius:nil];`    
　　`self.mainDraggableView.delegate = self;`    
　　`self.mainDraggableView.dataSource = self;`    
　　`self.mainDraggableView.backgroundColor = [UIColor whiteColor];`    
　　`[self.mainScrollView addSubview:self.mainDraggableView];`    

　　一个多月没有发博客，其实积攒了很多材料，一直想把这一篇发出来后再写其他的。