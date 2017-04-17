# iOS Best Practices
这里总结一下iOS最佳实践，主要参考Futurice developers的文章[iOS Good Practices](https://github.com/futurice/ios-good-practices)，同时加上了一些自己的理解

## 目录
* [初始化项目](#setup_project)
* [iOS架构](#iOS_architecture)
* [API设计](#api_design)
* [高质量代码](#high_quality_code)
* [编码规范](#code_style)
* [iOS开源库](#iOS_open_source_lib)
* [iOS基本技能](#iOS_basic_skills)
* [自动化构建](#build_automation)
* [优化App](#optimiaze_app)

<b id="setup_project"></b>
## 初始化项目
### UI采用Interface Builder还是手写代码
当开始一个iOS项目时，一个常见的问题就是编写所有的views是采用手写代码的方式还是使用Interface Builder的Storyboard或XIB文件。两种方式都可以实现相同UI效果，但几点需要考虑：

#### 为什么使用代码？
* 当多人协作开发时，由于Storyboard复杂的XML结构，它更容易发生版本冲突，这比手写代码的方式更难合并代码。
* 手写代码更容易地结构化和重用views，因此使你的代码库保持[DRY](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself)。
* 所有的信息都在同一个地方。在Interface Builder中，你必须通过点击所有inspectors才能找到你想要的东西。

#### 为什么使用Storyboard？
* 为了更少的技术要求，Storyboard通过微调颜色或布局约束等方式来直接对项目做出贡献是一种很好的方式。但它需要设置工程并且需要时间来学习基础。
* 由于你无需构建项目就可以预览一些变化，迭代速度就变得更快了
* 在Xcode 6，自定义字体和UI元素最后都以可视化的方式在Storyboard上显示，当你设计最终外观时，给你更加好的灵感。
* 从iOS 8开始，[Size Class](http://blog.futurice.com/adaptive-view-ios8)允许你为不同的类型的设备和屏幕来设计并且没有重复

#### AsyncDisplayKit:保持UI流畅和快速响应
[AsyncDisplayKit](https://github.com/facebook/AsyncDisplayKit)是一个能让非常复杂的用户界面保持流畅和快速响应的iOS框架。作者[Scott Goodson](https://www.facebook.com/scottgoodson)在Facebook开发[Paper](https://www.facebook.com/paper) app时，由于它的用户界面和交互非常复杂，使用Paper过程中会出现卡顿，于是就开发出AsyncDisplayKit来解决UI的性能问题。同时它受[React](http://facebook.github.io/react/)启发，独立研发出性能比AutoLayout高的布局系统，详情可参考[AsyncDisplayKit 2.0: Intelligent User Interfaces - NSSpain 2015](https://www.youtube.com/watch?v=RY_X7l1g79Q)


### Git版本控制与.gitignore
创建工程之后，一个好的步骤就是使用[git](https://git-scm.com)对工程进行版本控制并且有个合适的`.gitignore`文件。这个文件的作用主要是忽略不想要的文件(用户设置、临时文件等)，不将它们存放在仓库。幸运的是，Github都提供[Objective-C](https://github.com/github/gitignore/blob/master/Objective-C.gitignore) 和 [Swift](https://github.com/github/gitignore/blob/master/Swift.gitignore)两种模板。

### Build设置与Faux Pas/Infer
为了提高**代码质量**，你可以配置一些Build Settings：

* 启用**Analyze During 'Build'**，在源文件运行Clang静态分析工具
![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/Enable-Build-Static-Analysis.png)

* 启用**Treat Warnings as Errors**，它会导致所有的警告都当做错误来处理
![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/Treat-Warnings-As-Errors.png)

除了配置build settings之外，你还可以通过工具[Faux Pas](http://fauxpasapp.com/)或[Facebook Infer](https://github.com/facebook/infer)来检测静态错误。它通过分析你代码库，然后查找出你意想不到的问题。在上架任何iOS或Mac的app之前使用它吧。

### 使用CocoaPods管理库
如果你打算引入额外的依赖(例如，第三方库)到你工程，[CocoaPods](http://www.cocoapods.org/)提供一种容易和快速的集成方式。下面简单介绍它的安装和使用

#### 安装CocoaPods
使用以下命令就能安装**CocoaPods**：

```
sudo gem install cocoapods
```
输入上述命令可能很久都没响应，因为CocoaPods也被墙了。所以使用淘宝的镜像会快不少，

```
gem sources --remove https://rubygems.org/
gem sources -a http://ruby.taobao.org/
```

#### 使用CocoaPods
* 使用`pod search`来搜索你需要安装的库。例如，你想安装`mantle`这个iOS库，使用这个命令来搜索：`pod search mantle`，显示以下结果：
![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/pod-search.png)
红色圈住的内容就是安装库时`Podfile`文件需要的配置信息

* 在工程目录下创建`Podfile`文件。这个文件主要保存依赖库信息，告诉CocoaPods去下载和管理哪些依赖库。创建之后，将`pod 'Mantle', '~> 1.5.4'`追加到Podfile文件
![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/Podfile-Demo.png)

* 使用`pod install --verbose --no-repo-update`安装第三方库。

* 如果在后来为了添加或删除第三库而修改`Podfile`文件，使用`pod update --verbose --no-repo-update`更新第三方库。

* 由于安装和更新库的命令太长，使用shell的命名别名来简化，以后只需输入`pod_install`和`pod_update`即可安装和更新第三方库了

 ```
 #setup pod update alias name
 alias pod_update='pod update --verbose --no-repo-update'
 alias pod_install='pod install --verbose --no-repo-update'
 ```
* 安装好第三方库之后，关闭Xcode，然后打开`.xcworkspace`而不是`.xcproject`文件，然后引入对应第三方库的头文件，你就可以使用第三方库了。

### 项目目录结构
为了组织目录里面的上百个源代码文件，最好的方式就是根据你的架构来设置目录结构。例如，你可以采用以下目录结构：
	
    ├─ AppDelegate
    ├─ General
    ├─ Macro			
    ├─ Sections
    ├─ Helpers
    ├─ Vendor
    ├─ Resources
   
* ####AppDelegate
这个目录主要存放AppDelegate.h(.m)文件，是整个app的入口文件，所以单独拿出来。

* ####General
这个目录放会被重用的Views、Controllers、Categories和DataSources等。里面大概是这样：

 ```
 General
     |- Cell
        |- ELMBaseCell
        ...
     |- Views
        |- ELMScollView
        |- ELMPullToRefresh
        ...
     |- Controllers
        |- ELMBaseViewController
        ...
     |- Categories
        |- UIViewController+Sizzle
        |- UIImageView+Downloader
        ...
     |- DataSources
        |- ArrayDataSource
        ...
 ```
这里的`ELM`是项目的首字母缩写

* ####Macro
这个目录下放了整个app会用到的宏定义，里面大概是这样：

 ```
 Macro
     |- AppMacro.h
     |- NotificationMacro.h
     |- VendorMacro.h
     |- UtilsMacro.h
     ...
 ```
如果有新的类型的宏定义，可以再新建一个相关的Macro.h。

* ####Sections
这个目录下面的文件对应的是app的具体单元，如导航、瀑布流等等。里面大概是这样：

 ```
 Sections
     |- Menu
     |- Setting
     |- Collection
     ...
 ```
 而每个具体单元对应存放几个部分`View`, `Controller`, `ViewModel`, `Model`, `Network`的类。
 
* ####Helpers
这个目录放一些帮助类，文件名与功能挂钩。里面大概是这样：

 ```
 Helpers
     |- ELMShareHelper
     |- ELMHelper
     |- ELMEmotionHelper
     ...
 ```
 帮助类的主要作用是帮助Controller瘦身，也可以提供一定程度的复用。

* ####Vendors
这个目录放第三方的类库/SDK，如UMeng、WeiboSDK、WeixinSDK等等。

* ####Resources
这个目录下放的是app会用到的一些资源，例如声音、数据，但**图片**存放在`Images.xcassets`目录。

项目结构的参考链接：[iOS项目的目录结构和开发流程](http://limboy.me/ios/2013/09/23/build-ios-application.html)

### Assets资源
[Asset catalogs](https://developer.apple.com/library/ios/recipes/xcode_help-image_catalog-1.0/Recipe.html)是管理你的项目所有可视化资源(visual assets)的最好方式。它们可以保存通用和特定设备(4寸iPhone，视网膜iPhone、iPad等)两种资源和会根据文件名自动提供合适的资源。

#### 使用Bitmap Images
Asset catalogs仅暴露图像集合的名字，在图像集合中抽离实际的文件名。这样话，很好地防止assset资源命名冲突，因为像`button_large@2x.png`这样的文件在它们的图像集合被划分命名空间。但当命名assets时遵循一些规则会更加方便。

```
IconCheckmarkHighlighted.png // Universal, non-Retina
IconCheckmarkHighlighted@2x.png // Universal, Retina
IconCheckmarkHighlighted~iphone.png // iPhone, non-Retina
IconCheckmarkHighlighted@2x~iphone.png // iPhone, Retina
IconCheckmarkHighlighted-568h@2x~iphone.png // iPhone, Retina, 4-inch
IconCheckmarkHighlighted~ipad.png // iPad, non-Retina
IconCheckmarkHighlighted@2x~ipad.png // iPad, Retina
```

修饰后缀`-568h`, `@2x`, `~iphone` 和 `~ipad`并不是必要的，但如果有了它之后，当拖动文件到图像集合会自动将它们放在合适的位置，因此，如果发生分配错误，这会很难发现它们。

#### 使用 Vector Images
你也可以把由设计师设计的原始[vector graphics (PDFs)](http://martiancraft.com/blog/2014/09/vector-images-xcode6/)添加到asset catalogs，然后Xcode会自动生成bitmaps。这样会减少工程的复杂性(管理更少的文件)。

### 杂项
#### App Icon和Launch Image
* __App Icon:__ 选中工程目录下的`Images.xcassets`文件，向AppIcon中添加Spotlight、Settings和App Icons

 ![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/AppIcons.png)
 （引用饿了么App图片）

* __Launch Image:__ Xcode6之后，每次创建工程之后都会创建`LaunchScreen.xib`文件来显示启动画面，如果你想采用之前的Launch Image方式，你需要在工程设置中`Geneal`的**App Icons and Launch Images**中设置Launch Images Source.
 ![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/Project-General-AppIcons-Settings.png)
 
 然后添加图片到LaunchImage中
 
 ![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/LaunchImages.png)
 （引用饿了么App图片）

#### Bundle Display Name
一般来说，Bundle Display Name跟工程名相同。例如， 你设置工程名为`MeiTuanWaiMai`，那么App显示Bundle Display Name就为`MeiTuanWaiMai`。
 ![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/Bundle-Display-Name1.png)
 
 但一般我们都需要在`Info.plist`文件中添加一个`Bundle display name`字段来修改名字。例如，我修改为**美团外卖**。
 
 ![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/Modify-Bundle-Display-Name.png)
 
 修改后效果如下：
 
 ![](https://github.com/samlaudev/iOS-Best-Practices/raw/master/ScreenShots/Bundle-Display-Name2.png)
 
#### 状态栏
状态栏默认是黑色，如果你想改为**白色**，首先在项目设置General中将`Status Bar Style`设置为`Light`
 
 ![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/Project-Settings-StatusBar-Light.png)

然后在`Info.plist`文件中添加多一个字段`View controller-based status bar appearance`，并设置为`NO`
 
 ![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/Info-Settings.png)
 


<b id="iOS_architecture"></b>
## iOS架构
* [Model-View-Controller-Store (MVCS)][mvcs]
    * 这是Apple默认的架构(MVC)，通过引入**Store层**使它能够访问Model实例和处理网络请求、缓存等
    * 每个Store对象暴露给view controller一个`RACSignal`对象或一个`void`返回值的自定义completion block方法
* [Model-View-ViewModel (MVVM)][mvvm]
    * 出于"重量级的view controllers": MVVM想将`UIViewController`子类那部分处理view之间状态和交互逻辑抽离到ViewModel，保持Controller的精简和让代码更加容易测试。
    * 对于很多Cocoa开发者都是新的概念，但可以参考文章[cocoasamurai-rac]和[raywenderlich-mvvm]
* [View-Interactor-Presenter-Entity-Routing (VIPER)][viper]
      * 它是比较特殊的架构，适用于大型的项目，当感觉连MVVM都太混乱了和难以测试，你是时候了解这个架构了。

[mvcs]: http://programmers.stackexchange.com/questions/184396/mvcs-model-view-controller-store
[mvvm]: http://www.objc.io/issue-13/mvvm.html
[cocoasamurai-rac]: http://cocoasamurai.blogspot.de/2013/03/basic-mvvm-with-reactivecocoa.html
[raywenderlich-mvvm]: http://www.raywenderlich.com/74106/mvvm-tutorial-with-reactivecocoa-part-1
[viper]: http://www.objc.io/issue-13/viper.html

### “Event”模式
这里有一些通知其他对象的常用方法：

* __Delegation:__ _(一对一)_ Apple经常地使用它(有些人会说，太多了)。当想从model view执行回调时，你就会使用它。
* __Callback blocks:__ _(一对一)_ 当相关代码段之间关系紧密时，它能使代码之间更加松耦合。当有很多的senders时，它比delegation更加容易扩展。
* __Notification Center:__ _(一对多)_ 这是多个对象发送"事件"给多个观察者的最常用方式。耦合度非常低 - notifications甚至在没有引用dispatching object的情况下被全局观察。
* __Key-Value Observing (KVO):__ _(一对多)_ 不需要被观察者显式地发送"事件"只要 _Key-Value Coding (KVC)_ 符合被观察的键(属性)。通常不推荐使用，因为它隐式的特性以及繁琐的API。
* __Signals:__ _(一对多)_ [ReactiveCocoa]的核心就是通过提供一些[callback hell][elm-escape-from-callback-hell]，允许将你和核心内容链接和组合.

[ReactiveCocoa]: https://github.com/ReactiveCocoa/ReactiveCocoa
[elm-escape-from-callback-hell]: http://elm-lang.org/learn/Escape-from-Callback-Hell.elm

### Models

### Views

### Controllers

#### 代码结构规范
```
#pragma mark - Lifecycle
- (instancetype)init {}
- (void)dealloc {}
- (void)viewDidLoad {}
- (void)viewWillAppear:(BOOL)animated {}
- (void)didReceiveMemoryWarning {}

#pragma mark - Custom Accessors
- (void)setCustomProperty:(id)value {}
- (id)customProperty {}

#pragma mark - IBActions/Event Response
- (IBAction)submitData:(id)sender {}
- (void)someButtonDidPressed:(UIButton*)button

#pragma mark - Protocol conformance
#pragma mark - UITextFieldDelegate
#pragma mark - UITableViewDataSource
#pragma mark - UITableViewDelegate

#pragma mark - Public
- (void)publicMethod {}

#pragma mark - Private
- (void)privateMethod {}

#pragma mark - NSCopying
- (id)copyWithZone:(NSZone *)zone {}

#pragma mark - NSObject
- (NSString *)description {}

```

#### 对controller"瘦身"

### 网络

### 数据存储

### iOS架构参考博客系列
#### [Casa Taloyum](http://casatwy.com)的iOS应用架构
* [iOS应用架构谈 开篇](http://casatwy.com/iosying-yong-jia-gou-tan-kai-pian.html)
* [iOS应用架构谈 view层的组织和调用方案](http://casatwy.com/iosying-yong-jia-gou-tan-viewceng-de-zu-zhi-he-diao-yong-fang-an.html) 
* [iOS应用架构谈 网络层设计方案](http://casatwy.com/iosying-yong-jia-gou-tan-wang-luo-ceng-she-ji-fang-an.html)
* [iOS应用架构谈 动态部署方案](http://casatwy.com/iosying-yong-jia-gou-tan-ben-di-chi-jiu-hua-fang-an-ji-dong-tai-bu-shu.html)
* [iOS应用架构谈 本地持久化方案](http://casatwy.com/iosying-yong-jia-gou-tan-ben-di-chi-jiu-hua-fang-an-ji-dong-tai-bu-shu.html)
* [iOS应用架构谈 组件化方案](http://casatwy.com/iOS-Modulization.html)

#### [Objc中国](http://objccn.io)的iOS架构
* [MVVM 介绍](http://objccn.io/issue-13-1/)
* [避免滥用单例](http://objccn.io/issue-13-2/)
* [iOS 中的行为](http://objccn.io/issue-13-3/)
* [子类](http://objccn.io/issue-13-4/)
* [使用VIPER构建iOS应用](http://objccn.io/issue-13-5/)


<b id="api_design"></b>
## API设计
### 面向对象基本概念
* 抽象
* 封装
* 继承
* 多态

### 面向对象设计原则
#### 单一职责原则(Single Responsibility Principle)
#### 开放封闭原则(Open Close Principle)
#### Liskov替换原则(Liskov Substitution Principle)
#### 接口隔离原则(Interface Segregation Principle)
#### 依赖倒置原则(Dependency Inversion Principle)

### 设计模式
### 重构
### AOP
[Aspects](https://github.com/steipete/Aspects)
### 依赖注入
[dyci-main](https://github.com/DyCI/dyci-main)


<b id="high_quality_code"></b>
## 高质量代码
### 简洁性
### 易读性
### 复用性
### 可测试性
### 健壮性

<b id="code_style"></b>
## 编码规范
关于Objective-C的编码规范，请参考我翻译的[raywenderlich.com Objective-C编码规范](https://github.com/samlaudev/Objective-C-Coding-Style)

### 注释
#### 类注释
#### 属性注释
#### 方法注释
#### 方法体注释

<b id="iOS_open_source_lib"></b>
## iOS开源库
* [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)
* [Aspects](https://github.com/steipete/Aspects)
* [AFNetworking](https://github.com/AFNetworking/AFNetworking)
* [Mantle](https://github.com/Mantle/Mantle)
* [Masonry](https://github.com/SnapKit/Masonry)
* [Classy](https://github.com/cloudkite/Classy)
* [ClassyLiveLayout](https://github.com/olegam/ClassyLiveLayout)
* [Kiwi](https://github.com/kiwi-bdd/Kiwi)
* [FMDB](https://github.com/ccgus/fmdb)
* [MTDates](https://github.com/mysterioustrousers/MTDates)
* [SDWebImage](https://github.com/rs/SDWebImage)

<b id="iOS_basic_skills"></b>
## iOS基本技能
### 多线程管理
### 内存管理
### AutoLayout适配不同屏幕
### 自定义复用控件
### 自定义动画
### 调试
### 崩溃报告分析
### 阅读开源项目

<b id="build_automation"></b>
## 自动化构建
### 优化编译速度
### 构建配置

### 分析

### 部署配置

<b id="optimiaze_app"></b>
## 优化App

### 可执行文件瘦身
### 图片压缩工具
* __[tinypng](https://tinypng.com):__ 是一个在线图片压缩工具，收费，不过压缩率比ImageOptim高6%左右，由于它没有mac客户端，可以通过Alfred的[Workflow](http://www.alfredforum.com/topic/1520-tiny-png-workflow-updated-to-v12/)来使用。
* __[ImageOptim](https://imageoptim.com):__是一个mac客户端图片压缩工具，免费，压缩率比tinypng低


