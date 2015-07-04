---
layout: post
title: "Swift与Objective-c混编"
date: 2015-07-04 12:39:12 +0800
comments: true
categories: Swift
---
###背景
　　适时搞点新鲜玩意，增加挑战性，同时学以致用。  
###混编第一步：创建桥接头文件，让Swift调用OC
　　添加一个swift文件到工程，这时会提示是否创建一个桥接头文件。如下图所示：  
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/6%2F6.png">  
　　自动创建的桥接文件命名格式为ProductName-Bridging-Header.h，默认在当前添加Swift文件的同一目录（如要更换文件存放目录，在project settings中搜索，修改指定项的值给予新路径即可）。该文件中用于导入给swift文件使用的OC类，使用#import "xxx.h"的OC语法导入，文件名及内容如下所示。（导入OC头文件一直都没有提示，But it works.）  
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/6%2F1.png">  
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/6%2F2.png">  
　　导入后，在swift文件中，便可用swift的方式调用。如下图
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/6%2F3.png">
###混编第二步：导入swift头文件，让OC调用Swift
　　第一步完成后可在Swift中调用OC类，而反向调用应该怎么做呢？答案是，导入Xcode自动生成的隐式Swift类的头文件。如下图：  
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/6%2F4.png">  
　　这个隐式文件由Xcode自动生成，且只需导入它，便可调用所有swift类。需要注意的是，在必要的.m文件中再导入，防止头文件引用循环，也加快编译速度。  
　　基本规则如下表所示：  

<table border="1", class="table table-bordered table-striped table-condensed">
    <tr>
        <td></td>
        <td>调用Swift代码</td>
        <td>调用OC代码</td>
    </tr>
    <tr>
        <td>Swift代码</td>
        <td>无需声明</td>
        <td>依赖ProductModuleName-Bridging-Header.h文件</td>
    </tr>
    <tr>
        <td>OC代码</td>
        <td>#import "ProductModuleName-Swift.h"</td>
        <td>#import "Header.h"</td>
    </tr>
</table>  
 
###混编第三步：定义类和协议遵从
　　使用Swift创建一个UIBaseViewController的子类，需要使用如下代码声明class。

    class MLF_Tab_Me_MyCardsViewController: UIBaseViewController {
    
    }
　　如果要实现某个协议，则继续在UIBaseViewController后面用逗号分隔协议，比如：  

    class MLF_Tab_Me_MyCardsViewController: UIBaseViewController, UITableViewDelegate, UITableViewDatasource {
        var mainTableView : UITableView!
    }
　　传统OC中，若采用纯代码初始化界面，一般都是在viewDidLoad中完成；但swift默认必须在调用super.init构造方法前就初始化新增成员。不过像上述代码中添加mainTableView那样，使用implicitly unwrapped optional（隐式解析可选）类型，可以延迟到viewDidLoad时初始化，默认初始值为nil。  
###混编第四步：适应新的Json反序列化机制
　　Swift的Json反序列化，对类型要求十分严格。由于Swift的原生String类型没有足够的方法转数字类型，因此很有必要将原生对象转换为NSString或NSNumber。而这样的类型转换，必须对原生的可选类型对象做force unwrapping(强制解析)才能type casting(类型转换)。对一个nil值做force unwrapping操作，会引发运行时错误。

    fatal error: unexpectedly found nil while unwrapping an Optional value
　　不知道会不会有人问，为什么要用可选类型？因为只有可选类型才能被赋值为nil。假若给非可选类型显式赋值nil，编译时报错；隐式赋值nil，引发运行时错误。如下图，程序运行完31行后就crash了。  
<img src="http://showmylym-blog.oss-cn-shenzhen.aliyuncs.com/6%2F5.png">  
　　从程序健壮性角度而论，客户端是不能假定某个key一定可以取到value的，因此反序列化后对象为nil是常有的现象。OC中的运行时动态类型机制，决定了类型检查相对宽松，给nil发送消息，运行时会自动忽略该条消息。  
　　另外，OC中NSString和NSNumber几乎有相同的取值方法，且基于运行时动态类型，json反序列化时无需检查对象类型便可直接调用那些同名的取值方法。比如给NSString或NSNumber对象发送longLongValue消息，都能作为int64_t类型取出值；对象为nil也没关系，取出的值为0。但在Swift中，类型如果错误，会直接报运行时错，导致crash。  
　　综上述，这样的规则可以让人感觉出，Swift更加注重安全性，要求更多地用代码显式表明行为。个人认为，如何在遵循Swift新规则的前提下，减少条件表达式的代码量，将是一项值得研究的议题，也可能是Swift后续版本可能优化的地方。