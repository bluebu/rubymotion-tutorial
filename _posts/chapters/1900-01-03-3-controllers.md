---
layout: default
title: Controllers
description: Use UIViewControllers to organize your app
categories:
- chapters
---

# 控制器 Controllers

我们已经完成了一些视图层的工作, 但是这些只是 iOS SDK采用的 MVC 范例当中的一部分. 这听起来真的很花哨，但它实际上是非常简单的.

简单来说，你的代码，你应该有三种类型：视图（是的，你已经看到过）, 模型（数据的表现和处理）, 还有…控制器.

那么, 什么是控制器? 它们是位于模型和视图之间的"层"， 处理从用户传递过来的事件, 更新模型并更新视图, 以此来作为响应。在一个完美的代码世界, 当你点击一个按钮, 控制器截获该事件，更新数据的属性，并改变视图以响应新的数据。

这听起来有点"大画面", 但对于控制器来说, 也有一些真正实用的原因:

- **视图重用**. 比方说, 我们有一个视图 `PostView`, 用来显示所有有关帖子的信息(内容, 作者, "喜欢", 等等). 我们想在不同屏幕上使用这个这个视图, 比如一个首页Feed, 用户个人资料页的Feed. 为了保证重用性, 视图 `PostView` 不应该处理如何获取信息; 相反, 控制器才应该考虑这些, 然后将处理后的数据传递到视图.
- **展现管理**. 有时候我们希望有一个视图占满整个屏幕, 又有些情况下我们想使用一个对话框来显示同样的内容(想象一下iPad和iPhone). 如果仅仅因为表现不同而写两个相同的视图类, 这么做是毫无意义的, 所以我们要使用控制器来调整视图的尺寸和交互.

并不是什么技术原因阻止你在模型内部和视图上做这些事情, 但是如果你遵守MVC规范的话, 它会使你的代码更健壮, 更便于管理.

在 iOS世界中, 控制器就是`UIViewController`. 它们伴随着一个 `view` 属性以及一些方法, 用来处理类似视图"生命周期"、定位发生变化等事务. 不要担心, 我们马上就会介绍生命周期的业务逻辑.

到目前为止, 我们知道了控制器是什么, 应该做什么, 那么 *不应该* 做什么呢?

- 直接查询或保存数据. 大家很容易想在控制器里面发送一批HTTP请求, 但这些最好都留给你的模型.
- 复杂视图的布局. 如果你想添加的子视图相对当前视图层级深度过大(大于1), 你应该重构下你的视图. 作为一个好的经验法则, 在你的控制器, 唯一的 `addSubview` 应该出现在 `self.view.addSubview`.

好了, 通篇的理论, 下面是老谋子, 苍老师的大片时间.

## 一切尽在Controllers之中

创建目录 `./app/controllers` (`mkdir ./app/controllers`),  然后在里面创建一个 `TapController.rb` 文件. 像这样来定义我们的控制器:

```ruby
class TapController < UIViewController
  def viewDidLoad
    super

    self.view.backgroundColor = UIColor.redColor
  end
end
```
`viewDidLoad` 是 `UIViewController` 生命周期方法中的一个, 当`self.view`已经创建好了, 准备添加子视图的时候, 系统会调用这个方法. 现在为止, 我们仅仅让它的背景颜色置为红色, 然后就收工.

在`viewDidLoad`里面, 你 *绝对*, *必须*, *毫无疑问地* 调用 `super` 方法, 否则一些不好的事情就要发生了. 懂了吗? Cool.

现在, 回到你的 `AppDelegate` , 干掉之前 `UIView` 的代码. 我们只需要加一行代码, 就像这样:

```ruby
class AppDelegate
  def application(application, didFinishLaunchingWithOptions:launchOptions)
    @window = UIWindow.alloc.initWithFrame(UIScreen.mainScreen.bounds)
    @window.makeKeyAndVisible

    # This is our new line!
    @window.rootViewController = TapController.alloc.initWithNibName(nil, bundle: nil)

    true
  end
end
```

看到我们调用 `rootViewController=` 了吗? window会关联给定的`UIViewController`, 并调整其视图的大小来适配整个窗口. 这是设置窗口的很好的办法（而不是到处 `window.addSubview` ）。

这行代码另一部分新面孔就是  `initWithNibName:bundle:` , 通常, 这是用来从一个 `NIB` 文件加载一个控制器. `NIB` 文件是Xcode的Interface Builder用来可视化构造你的视图. 由于我们没有为我们的控制器使用Interface Builder, 我们可以放心大胆地对这两个参数传nil值.

`initWithNibName:bundle:` 同样也是 `UIViewController` 指定的初始化方法. 无论你什么时候想创建一个控制器, 你 *必须*
 在某一时刻调用这个方法, 尤其是在自定义的初始化方法.


嗯, 有点跑题了, `rake` 然后 检查一下. 你应该看到:

![first controller](images/1.png)


Big things have small beginnings. 让我们对控制器做一个小修改:

```ruby
  def viewDidLoad
    super

    self.view.backgroundColor = UIColor.whiteColor

    @label = UILabel.alloc.initWithFrame(CGRectZero)
    @label.text = "Taps"
    @label.sizeToFit
    @label.center = CGPointMake(self.view.frame.size.width / 2, self.view.frame.size.height / 2)
    self.view.addSubview @label
  end
```

又来了一个 `UILabel` ! `UILabel` 同样是一种视图, 根据 `text` 属性显示静态文. 我们创建了一个, 将 text 设为 "Taps", 然后置为子视图.


我们使用 `CGRectZero` 来初始化它, 因为我们还不知道屏幕上的文字的精确尺寸, 然而, 当我们调用`sizeToFit`, label 会调整自身大小以完全适配其内容. 然后我们得心应手地使用 `center` 属性, 让它在我们控制器的视图居中.

再来一次 `rake`, 看到一个看起来不错的应用. 现在它什么都不能做, 但是下一章, 我们会给它加点功能的.

![with a label](images/2.png)

## 收工

这回我们学到了什么?

- iOS SDK 使用了 Model-View-Controller 规范.
- `UIViewController` 构成了 Controller 部分, 并且它应该被继承为一个子类来定制你的控制器.
- 当设置你的控制器时候, 使用 `viewDidLoad` , 并且 *不要忘了调用 *super*.
- `UIWindow` 有一个 `rootViewController` 属性, 用来显示控制器.

[撑杆跳到下一章 Container!](/4-containers)
