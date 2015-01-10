---
layout: post
title: "图标管理控件的实现-类似于iOS主页的效果"
date: 2015-01-06 08:22:12 +0800
comments: true
categories: 控件
---
###开发背景:    
　　前段时间因工作需要，写了一个图标管理控件。日常开发中可能类似的需求不少，于是做成可复用的代码库，当作编写可复用控件的开端。    
　　　　　　　　　　　　　　　　　　　　　　　　　　　－－题记    
　　这个图标管理控件我为其命名为DraggableView，意为可拖动视图，它继承于UIView，与UITableView类似，它有自己的cell，cell中实现长按拖动(LongPressGesture)、点击选择(TapGesture)，以及删除功能。    
　　以下是控件演示：    
　　<embed src="http://player.youku.com/player.php/Type/Folder/Fid/23308555/Ob/1/sid/XODY0NTg4NTAw/v.swf" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" allowFullScreen="true" mode="transparent" type="application/x-shockwave-flash"></embed>    
　　纵观iOS App市场，有类似实现效果的应用不少，如下截图：    
　　<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/2/2-1.PNG" title="图片2-1" alt="图片2-1"/>    
　　<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/2/2-2.PNG" title="图片2-2" alt="图片2-2"/>    
　　<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/2/2-3.PNG" title="图片2-3" alt="图片2-3"/>    
　　<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/2/2-4.PNG" title="图片2-4" alt="图片2-4"/>    
###大致工作流程为：    
　　1、点击可选择，类似于UITableView的didSelect方法。    
　　2、长按一个cell，触发编辑状态。编辑状态的cell将放大显示，所有图标将以角度3°轻微旋转，表示当前的拖动编辑状态。处于编辑状态的cell可以被删除、被拖动重排序，但不能触发点击回调，两种操作都有回调暴露，在代理对象中编写具体实现。    
　　3、若数据源层级中添加了新的cell，只需[draggableView reloadData]调用一次即可刷新UI。刷新完成时，draggableView向代理对象发送didResizeWithFrame消息，代理实现可根据此回调适时改变视图中UIScrollView的contentSize，适应新内容大小。    
　　下图为正常状态下各个部分的有效区域：    
　　<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/2/2-5.png" title="图片2-5" alt="图片2-5"/>    
　　下图为编辑状态下各个部分的有效区域：    
　　<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/2/2-6.png" title="图片2-6" alt="图片2-6"/>    
###大致使用流程为：    
####1、初始化：
　　`self.mainDraggableView = [[LYMDraggableView alloc] initWithFrame:CGRectMake(0.0, 0.0, screenWidth, draggableViewHeight) layoutType:LYMDraggableViewLayoutByColumnNum horizontalMargin:0.0 verticalMargin:0.0 vSpace:4.0 maxColumn:4.0 cornerRadius:nil];`    
　　`self.mainDraggableView.delegate = self;`    
　　`self.mainDraggableView.dataSource = self;`    
　　`self.mainDraggableView.backgroundColor = [UIColor whiteColor];`    
　　`[self.mainScrollView addSubview:self.mainDraggableView];`    
　　这里有个技术细节问题。对UIView及其子类初始化时，如果调用    
　　`[[UIView alloc] initWithFrame:rect]`    
　　方法，那么rect的宽或高都最好不要为0，可根据已有数据源，调用DraggableView中的类方法    
　　`+ (CGFloat)heightOfDraggableViewFromVMargin:(CGFloat)vmargin cellHeight:(CGFloat)cellHeight vSpace:(CGFloat)vSpace itemsCount:(NSUInteger)itemsCount;
`    
　　得到预计高度。实际情况证明，rect的宽高值有一项为0时，界面中的所有添加在此容器view中的subview，其autoresizing机制将出现异常（autolayout不知，没试过，类推情况应该类似）。    
　　<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/2/2-7.jpg" title="图片2-7" alt="图片2-7"/>    
　　如上图2-7，根据官方文档里对其中一种autoresizing类型——UIViewAutoresizingFlexibleLeftMargin的描述原文：The distance between the view’s left edge and the superview’s left edge grows or shrinks as needed.    
　　意为：view的左边与其superView的左边距离，可根据实际情况动态增大或减小。也就是说，如果启用此种autoresizing类型，view与superView之间的右边距是保持不变的。因此，若supeView没有设置正确的初始rect（例如width为0），view将无法自动管理与superView之间的左边距。    
　　在DraggableView的初始化方法中，会检测height是否为0。若为0，则自动将其赋值为1。下图为起初编写DraggableCell时未给予titleLabel合法（height > 0）的height的结果——label上的text无故消失。    
　　<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/2/2-8.png" title="图片2-8" alt="图片2-8"/>    
####２、实现数据源方法
　　`- (NSInteger)numberOfCellsInDraggableView:(LYMDraggableView *)draggableView; // 返回cell的个数`    
　　`- (CGSize)cellContentViewSizeInDraggableView:(LYMDraggableView *)draggableView; // 返回cell的contentSize，Demo中将宽高值都乘以了屏幕比例系数，以适配大屏`    
　　`- (CGSize)cellCornerBtnSizeInDraggableView:(LYMDraggableView *)draggableView; // 返回cell的边角按钮size，用于在编辑状态时显示删除按钮，可设定位置左上或右上，同样乘以屏幕比例系数`    
　　`- (CGSize)cellSizeInDraggableView:(LYMDraggableView *)draggableView; // 返回cell自身的size，宽度 = contentSize.width + 2 * cornerBtnSize.witdh / 2.0, 高度 = contentSize.height + cornerBtnSize.witdh / 2.0. 有效区域见上图2-5`    
　　`- (LYMDraggableViewCell *)draggableView:(LYMDraggableView *)draggableView cellForIndex:(NSUInteger)index; // 使用DraggableViewCell的初始化方法创建cell，cell不能像UITableViewCell那样复用，详情参照demo`    
　　`- (void)draggableView:(LYMDraggableView *)draggableView didSelectCellAtIndex:(NSUInteger)index; // 普通状态下选中一个项。例如作为查看item对应的详细信息的入口`    
　　`- (void)draggableView:(LYMDraggableView *)draggableView didResizeWithFrame:(CGRect)frame; // 每次调用DraggableView的实例方法[instance reloadData]刷新完自身frame后，利用此回调可适时调整其superView容器的contentSize。例如draggableView所在UIScollView或UITableViewCell的contentSize`    
　　`- (void)draggableView:(LYMDraggableView *)draggableView cornerBtnPressedAtIndex:(NSUInteger)index; //边角按钮点击回调。可用作删除操作确认，或其他自定义功能。`    
　　`- (void)draggableView:(LYMDraggableView *)draggableView didMoveCellFromIndex:(NSUInteger)fromIndex toIndex:(NSUInteger)toIndex; // 移动了一个cell，从fromIndex移动到toIndex。`    
　　`- (BOOL)draggableView:(LYMDraggableView *)draggableView canMoveAtIndex:(NSUInteger)index; // 判断某个index的cell，是否可以被移动`    
　　`- (BOOL)draggableView:(LYMDraggableView *)draggableView canShakeWhenEditingAtIndex:(NSUInteger)index; // 判断某个index的cell，是否可以使用抖动动画表示编辑状态`    
　　`- (BOOL)draggableView:(LYMDraggableView *)draggableView canEditingAtIndex:(NSUInteger)index; //判断某个index的cell，是否可以切换为编辑状态`    
　　上面这三个can...方法在控制首尾两个cell时没有问题，但对于其他位置的cell控制不是很方便。首次很好，被重新排序后需要reloadData才能重新计算各个cell的正确index。详情参见Demo中的`[ didMoveCellFromIndex]`协议方法实现。    
　　`- (CGFloat)draggableView:(LYMDraggableView *)draggableView cellEditingScaleUpFactorAtIndex:(NSUInteger)index; // 将长按cell的放大系数以回调方式暴露出来`    
　　`- (void)draggableViewBeginEditing:(LYMDraggableView *)draggableView; // draggableView进入编辑状态回调`    
　　`- (void)draggableViewEndEditing:(LYMDraggableView *)draggableView; // draggableView结束编辑状态回调`    
###最后，再简单说说几个关键算法的处理逻辑：    
####1、整体布局。    
　　布局基于320.0宽度的1x屏幕，640.0的2x屏可自动x2，但iPhone 6、iPhone6 Plus无法最佳自动适配，因此会在关键的contentSize后乘以屏比系数，该系数值为：UIScreenBoundsWidth/320.0。    
####2、图标排列。    
　　最初设计包含了两种排列类型，一种LYMDraggableViewLayoutByCellSize，保持cell的size不变，cell之间的列间距自适应。如果有足够空间，在当前行再插入一个cell；另一种为LYMDraggableViewLayoutByColumnNum，保持cell的size相对屏幕宽度成比例放大或缩小，列数不变，cell之间的列间距按照剩余空间自适应。边距仍然保持不变，使用初始值。    
　　第一种没有实现，目前代码中使用的是第二种。如果以后有需要，不排除实现第一种排列类型。    
####3、判定移动。    
　　水平方向上，取被拖动cell水平中心点的坐标x，长按手势触发后在每次手势的didMove回调中检查x的值是否在原来某个cell的区域内。如果在，则立即插入到此位置。因此过程计算量较大，特别需要注意将数据计算与界面绘制独立开，保证代码思路清晰，后续可维护。    
####4、界面绘制逻辑。    
　　用了些基本算法，先算出总共几行几列，之后依次由外至内循环，从行开始，逐列计算。算完一个cell，将其frame数据作为下一个cell的frame计算依据，全程使用相对坐标。详细过程参照draggableView中的resetLayout方法。    
###整体感悟：    
####1、业务逻辑与功能实现分离。    
　　这点非常重要，分离的好，可重用度高，可快速应用在新的场景中，节约开发成本。    
####2、数据计算与界面绘制相互独立。    
　　这点显著益处是维护耗时短，有问题易暴露、易定位。试想数据计算与界面绘制夹杂在一处，写代码时图了方便，过两天自己看自己写的代码也得花上好些时间才能理清头绪。我们写代码时，**要假想以后维护自己代码的人，是知道自己住处地址的精神病，那么写代码还会那么随意么？**    
　　距离上一篇博客发布已一个月有余，其实积攒了很多材料，一直想把这一篇写好，写到自己满意的程度再发。