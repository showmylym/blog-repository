---
layout: post
title: "屏幕适配二三事（一）—— 屏适的几种基本情况讨论"
date: 2015-05-10 08:22:12 +0800
comments: true
categories: 杂谈
---
###各种iOS设备分辨率一览
    设备              单倍分辨率     倍数  渲染分辨率      显示分辨率      缩放系数
    iPhone6P（标准）  {414 x 736}   3x   {1242 x 2208}  {1080 x 1920}  1/1.15
    iPhone6P（放大）  {375 x 667}   3x   {1125 x 2001}  {1080 x 1920}  0.96
    iPhone6（标准）   {375 x 667}   2x   {750  x 1334}  {750  x 1334}  1.0
    iPhone6（放大）   {320 x 568}   2x   {640  x 1136}  {750  x 1334}  1.171875
    iPhone5          {320 x 568}   2x   {640  x 1136}
    iPhone4          {320 x 480}   2x   {640  x 960}
    
    设备             单倍分辨率     倍数  实际分辨率
    iPad 2(mini)    {768 x 1024}  1x   {768  x 1024}
    iPad mini 2(3)  {768 x 1024}  2x   {1536 x 2048}
    iPad air 1(2)   {768 x 1024}  2x   {1536 x 2048}
    iPad 3(4)       {768 x 1024}  2x   {1536 x 2048}  
　　由于iPhone 6系设备（包括6 plus）情况特殊，存在标准和放大两种显示模式，导致渲染时分辨率略有不同，不过渲染后最终其UI在缩放显示时的屏幕分辨率是确定的([详情戳这里](http://www.paintcodeapp.com/news/ultimate-guide-to-iphone-resolutions "iPhone分辨率详解"))。特别是iPhone 6 Plus，由于屏幕硬件限制，会强行缩放为{1080 x 1920}显示，故本文中的屏幕截图全部取自标准模式和1x（严格意义上说，放大模式和标准模式在app测试时都不能放过）。  

###背景
　　屏幕适配一直是老生常谈的话题，也一直是移动端app的重点之一。Apple每次新设备的发布，都会给iOS开发者带来一场灾难。如何最高效率地完成所有适配，成为一个新的讨论议题。  
　　闲话稍叙，看场景。  

###场景一 —— 微信聊天“表情选取栏”
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/3%2F%E5%BE%AE%E4%BF%A1%E8%A1%A8%E6%83%85%E9%A6%96%E9%A1%B55c.jpg" title="微信聊天“表情选取栏”" alt="微信聊天“表情选取栏”"/>  
###场景一解析
　　场景一属于散列布局，UI整体由众多相同类型的元素排列而成，适配时存在调整margin和space的可能性。因其元素本身较小，为了保证整体的保真度，无论何种屏幕，必须优先表情主体突出的原则，避免放大space，喧宾夺主。因此最佳的适配手段，是保持原有各项元素尺寸和space不变，挨个排入，不足一个元素单位的空间并入margin，适配对比如下图。  
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/3%2F%E5%BE%AE%E4%BF%A1%E8%A1%A8%E6%83%85%E9%A6%96%E9%A1%B5%E5%AF%B9%E6%AF%94.png" title="微信聊天“表情选取栏”首页对比" alt="微信聊天“表情选取栏”首页对比"/>  

###场景二 —— 支付宝宫格样式首页
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/3%2F%E6%94%AF%E4%BB%98%E5%AE%9D%E5%AE%AB%E6%A0%BC%E9%A6%96%E9%A1%B55c.jpg" title="支付宝首页" alt="支付宝首页"/>  
###场景二解析
　　场景二属于全屏式布局，space不存在，看到的无内容区域属于矩形元素内部的margin，矩形元素整体仍被等分划分，它们瓜分屏幕，任何矩形元素之间不留外部margin。这种布局有两个适配要点，一是单行的矩形元素数量保持不变；二是矩形元素内的主体成比例放大，放大的量级与屏幕放大系数（大屏宽除以小屏宽，我是这么定义的）相关。众所周知，5c屏幕宽度为320（单倍像素，下同），屏幕系数为2；而6p上屏幕宽度为414，屏幕系数为3。因此，放大系数就是 414.0 / 320.0 ≈ 1.29 。如果4个矩形元素组成一行，那么5c上{80 x 120}的矩形元素size，在6p上就是{103.5 x 155.25}，内部文本字体大小x1.29，icon同字体放大倍数。为了印证观点，排除系统自动缩放的效果，将6p的支付宝首页截图缩放为原始尺寸的33.3%，5c的缩放为50%，关键部分如下图，可观察到6p截图中内容的字体仍然略大，由此可知字体适配的确是与屏幕放大系数相关。  
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/3%2F%E6%94%AF%E4%BB%98%E5%AE%9D%E5%AE%AB%E6%A0%BC%E9%A6%96%E9%A1%B5%E5%AF%B9%E6%AF%94.png" title="支付宝首页对比" alt="支付宝首页对比"/>  

###场景三 —— 有信照片墙
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/3%2F%E6%9C%89%E4%BF%A1%E7%85%A7%E7%89%87%E5%A2%995c.png" title="有信照片墙" alt="有信照片墙"/>  
　　场景三是我参与开发“有信”时碰到的，与场景二类似但略有不同。整体也是按照原有的宽高比乘以屏幕放大系数来等分屏幕，但margin和space保持不变。这样适配的原因是，照片墙突出的是图片，在屏幕有更多空间显示时，应当优先放大图片，弥补小屏幕浏览图片的先天缺陷——内容展示空间十分有限。那么很明显，效果应该是下图所示的样子。  
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/3%2F%E6%9C%89%E4%BF%A1%E7%85%A7%E7%89%87%E5%A2%996p.png" title="有信照片墙" alt="有信照片墙"/>  

###场景四 —— 微信聊天气泡
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/3%2F%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A9%E6%B0%94%E6%B3%A15c.png" title="微信聊天页面气泡UI" alt="微信聊天页面气泡UI"/>  
　　情况开始变得复杂了。前面说的都是等比等宽缩放，像这种上下左右都需要动态计算的情况如何处理呢？别急，先看看微信是如何处理的，不过在此之前依旧排除屏幕自动缩放的干扰，1倍size对比，如下图所示。  
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/3%2F%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A9%E6%B0%94%E6%B3%A1%E5%AF%B9%E6%AF%94_5c.png" title="微信聊天“5c气泡UI”" alt="微信聊天“5c气泡UI”"/>  
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/3%2F%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A9%E6%B0%94%E6%B3%A1%E5%AF%B9%E6%AF%94_6p.png" title="微信聊天“6p气泡UI”" alt="微信聊天“6p气泡UI”"/>  
　　问题来了。假设以气泡箭头方向为正，那么气泡正向margin为64px，反向margin有两种情况：1、5c上对方消息margin为49px，己方消息margin为45px；2、6p上对方消息margin为90px，己方消息margin为86px。尽管反向margin值有出入，但其在两种情况下两方消息的margin差值一致为4，由此可推断，这个页面的气泡UI是用纯代码写的(为什么？见下一回分晓)。另外，对于对方消息的气泡，5c上，气泡的宽度为207(320.0 - 64.0正margin - 49.0反margin)，而6p上，气泡的宽度为260(414.0 - 64.0正margin - 90.0反margin)。  
　　这相当奇怪，既然保持了正margin不变，调整气泡宽度，那就应该同时保持反margin也不变，类似于场景三，让多出来的屏幕水平空间展示更多的气泡文字，这才发挥了大屏的作用。  
　　手里头没有iPhone6的设备，根据其在iPhone6标准显示模式下的表现，应当能准确推断其实现方式。个人推测很可能iPhone6刚出来时，开发人员按照5c的正反margin值和6的屏宽375px，口算得出气泡宽度值262(375.0 - 64.0正margin - 49.0反margin)，然后写死在代码里，导致6以后的设备气泡宽度都是260±。  

###场景五 —— 搜狗输入法
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/3%2F%E6%90%9C%E7%8B%97%E9%94%AE%E7%9B%985c.png" title="搜狗输入法" alt="搜狗输入法"/>  
　　搜狗输入法这个场景好似不遵从任何适配规则。如下图对比数据所示，看起来像是针对每一个元素做了判断逻辑，做了条件处理。推测结果  

    明显差值为24px = (50 * 3 + 46) - (44 * 3 + 40)  

　　还有6px，可能在未标注的行间距中，每个间距1像素。  
　　实际上，输入法键盘这类元素众多的UI，适配起来也可大致遵从场景二的方法，水平方向：原始总宽度320.0 * 屏幕放大系数(6p为1.29，6为1.17)，包含的所有控件宽度也都乘以同一系数；竖直方向：原始总高度同样乘以屏幕放大系数，包含的控件同理。这样，做到不失真地放大，连带以后新出的设备屏幕也一并兼容了。  
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/3%2F%E6%90%9C%E7%8B%97%E9%94%AE%E7%9B%985c%E6%A0%87%E6%B3%A8.png" title="搜狗输入法5c标注" alt="搜狗输入法5c标注"/>  
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/3%2F%E6%90%9C%E7%8B%97%E9%94%AE%E7%9B%986p%E6%A0%87%E6%B3%A8.png" title="搜狗输入法6p标注" alt="搜狗输入法6p标注"/>  

###综述
　　综上五类场景所述，缩放和对齐，是屏幕适配需求的基本方法。但它们都不能完美解决问题，有时需要将二者结合使用，再配合字符串高度计算方法，完整得出元素的frame。将构造好的元素frame全部计算完毕存入数组，之后一次性取用、刷新UI，既满足了坐标动态化，又不至于让数据计算和UI展示逻辑混淆，易编写、易维护。  
　　下一回，奉上一个Demo，“从0到1”实战演练。

###下集预告：
　　屏幕适配二三事（二）—— UI构建两大武器 纯代码 pk 纯autolayout
　　