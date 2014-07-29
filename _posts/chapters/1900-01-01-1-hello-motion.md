---
layout: default
title: Hello Motion
description: Learn the installation and project setup of RubyMotion, then make your first iOS app
categories:
- chapters
---
# Hello Motion

##  安装

[RubyMotion][rm] 是 HipByte 公司的商业产品, 该公司由[MacRuby][macruby]负责人创建. 当你购买了[RubyMotion许可证][buy] , 你会收到一个key和一个安装程序. 同时你也需要从Mac App Store安装[Xcode][xcode]. Xcode 会安装一些RubyMotion所依赖的开发工具(比如iOS Simulator), 但是实际上你不会在Xcode中写RubyMotion项目的.

相反的, RubyMotion 会更多依赖命令行工具， 并且你可以选择使用任何你喜欢的编辑器. 可以利用[插件][packages]对主流编辑器扩展，用以辅助代码实现和编译集成等. RubyMotion建立在已有的Ruby工具上，例如RubyGems 和 Rake, 所以如果你已有Ruby经验，那么你会有种舒服到家的感觉.

现在，一切已经安装好了，已经准备好尝试RubyMotion了吧，好戏马上上演!

## 你的第一个应用

打开控制台, 切换到存放RubyMotion项目的目录. 执行以下命令:

```ruby
motion create HelloMotion
```

`motion` 命令，是RubyMotion提供的工具集命令之一; 它等价于 `rails` 命令, 你应该已经很熟悉`rails` 了. 该命令用来管理项目和RubyMotion工具本身(有时你可能会被提醒执行`motion update`).

`motion create` 创建了一个 `HelloMotion` 目录，这个目录内部同时也创建了一些文件, 那么大胆地 `cd` 进入这个目录 (`cd ./HelloMotion`). 所有后续口令都会在这个目录中执行，所以毫无疑问地, 我们需要对这个目录保留一个终端窗口/标签.

首先, 我们先介绍一下创建地两个文件: `Rakefile` 和 `./app/app_delegate.rb`.

`Rakefile` 用来处理项目的配置(例如应用名称, 包含资源等信息), 库文件的导入 (第三方gems或者本地资源). `rake` 命令会使用这个文件, 而 `rake` 就是RubyMotion整个工作流的另外一个口令. 对于RubyMotion 1.11, 生成的`Rakefile`应该是这个样子:

```ruby
$:.unshift("/Library/RubyMotion/lib")
require 'motion/project'

Motion::Project::App.setup do |app|
  # Use `rake config' to see complete project settings.
  app.name = 'HelloMotion'
end
```

如果你不是很熟悉Ruby的话, 你的第一感觉应该是, "等等...`$:.unshift` 这是什么飞机?" 确实看起来挺奇怪的. 这行代码用来告诉Ruby, "当我们使用`require`时候, 同时也遍历一下'/Library/RubyMotion/lib'这个目录, 看能否找到我们声明`require`的". 下方的require 'motion/project', 要是没有前面的 `$:.unshift` , Ruby就会找不到我们想要的!

声明了`require 'motion/project'` 之后, 我们才可以在项目中正确的访问RubyMotion, 以及在`.setup`代码块中设置我们的项目. 这里有`app`中所有的属性和参数, 执行`rake config`可以列出自动生成的注释. 默认下By default, RubyMotion 将 `.name` 设置成我们项目的名称, 嗯，看起来不错.

等等, 我们究竟为什么需要一个 `Rakefile` ? 好的, RubyMotion 使用 `rake` 来调用所有的方法, 而`rake` 命令会执行所在目录的 `Rakefile`.`Rakefile` 应当定义一组"tasks", 这些"task"可以用来附加到rake执行 (`rake <task>`), 但是当我们 `require "motion/project"`时候 , 这些都已经创建了.

试一试! 在你的控制台中执行 `rake` , 应当弹出一个空白的 iPhone 模拟器. 此外, 你的终端此刻应该运行着一个交互控制台, 里面可以让你在运行时执行新的代码.

![hello](images/0.png)

"牛逼!" 你可能会情不自禁地呼喊着...但是这是怎么发生地呢? 我们是如何通过 `app.name = 'HelloMotion'` 来弹出一个 iPhone 模拟器?

可以看出RubyMotion的 `App` 对象拥有许多明显的约定, 比如 (最重要地) 递归 `./app` 下所有地Ruby文件. 记得我们早先提及的 `app_delegate.rb` 吗? 很明显当我们编译我们的项目时候, 这家伙也包含进去了! 让我们来看看:

```ruby
class AppDelegate
  def application(application, didFinishLaunchingWithOptions:launchOptions)
    true
  end
end
```

嗯…, 这家伙所做的事情就是声明了一个 包含一个方法的`AppDelegate` 类. 甚至连一个父类都没有, 有毛用啊?!

好吧, 快速执行一下 `rake config` 命令. 它将打印出一坨设置和信息, 但是我们感兴趣的是这个:

```ruby
...
delegate_class         : "AppDelegate"
...
```

哇哦, 宝贝, 我们这 "不重要的" AppDelegate 在这呢! RubyMotion 实际上会寻找这个delegate_class同名的类, 然后启动app时候调用这个类. 我们也可以把这个类改为 `SuperAppDelegate` , 如下修改我们的 `Rakefile`:

```ruby
Motion::Project::App.setup do |app|
  app.name = 'HelloMotion'
  app.delegate_class = "SuperAppDelegate"
end
```

那么, delegate 是什么? 在 IOS世界, 当用户启动app时候, 提供一些应用程序级别的状态的回调. 我们需要给操作系统提供一个对象, 这个对象在此过程中可以响应不同的事件; 我们将这个对象，称之为"应用程序委托类"(application delegate). 当app处于启动, 终止, 转为后台运行, 接收到一个推送通知, 等这些预定义情况下, 系统就会回调这个对象来完成相应的任务.

在 `motion` 生成的代码里, 我们仅仅实现了 `def application(application, didFinishLaunchingWithOptions: launchOptions)`. 这厮看起来有点和普通Ruby写法不太一样啊, 都是因为夹杂在中间 `didFinishLaunchingWithOptions` 这货...为什么? 好吧, 事情是这样的, 很久很久以前....

大多数语言里, 方法看起来是这个样子: `obj.makeBox(origin, size)`. 这样有些令人讨厌, 因为你需要去阅读下这个方法的具体实现, 然后找出里面有哪些变量, 并指向何处. Objective-C 使用 "命名参数" 去解决这个问题. 在 Objective-C 里面, 这个方法则变成这样: `[obj makeBoxWithOrigin: origin andSize: size];`.将参数相关信息附加到方法名中, 看清楚是如何来避免不确定性的了吧? 相当聪明. 我们参照这些方法, 并通过使用冒号来替代变量名, 就变成这样: `makeBoxWithOrigin:andSize:`.

在Ruby里面, 根本没有"命名参数"; 它使用的传统的方法, 就像 `obj.make_box(origin, size)`. RubyMotion 决定将"命名参数"添加到Ruby里,  这样就可以兼容原生的苹果API, 像这样: `obj.makeBox(origin, andSize: size)`. 那个 `andSize` 不是语法糖; 确确实实是方法名的一部分. `obj.call_method(var1, withParam: var2)` 和 `obj.call_method(var1, withMagic: var2)` 是完全不一样的, 尽管从正常的Ruby形式看起来像是 `obj.call_method`. [Ruby 2.0 确实存在命名参数, 但是RubyMotion 目前兼容的是1.9.3]

不管怎样, 回到我们的主线来. 当系统完成程序启动时, `application:didFinishLaunchingWithOptions:` 会被调用, 而且即将运行我们所创建的代码:

```ruby
def application(application, didFinishLaunchingWithOptions:launchOptions)
  true
end
```

到现在, 仅仅假设程序永远返回 `true`. 一些应用可能想使用 `launchOptions` 来决定应用是否应该开始运行, 但是大多说情况下你不会那样做的.

现在我们理解了项目如何启动的, 并且找到了项目开发的切入点. 要不…让我们来做点事情? 修改你的 `application:didFinishLaunchingWithOptions:` 如下:

```ruby
def application(application, didFinishLaunchingWithOptions:launchOptions)
  alert = UIAlertView.new
  alert.message = "Hello Motion!"
  alert.show

  puts "Hello again!"

  true
end
```

来看看我们都添加什么了? 首先, 有3行关于`UIAlertView`的代码. `UIAlertView`s 就是有时候使用IOS看到的蓝色弹层 (登录到iTunes Store, pre-iOS5 推送信息, 等等). 我们创建了一个, 并设置一个message, 然后 show 出来.

接下来, `puts` 是最基本的声明日志的方式. 你可以传递一个普通的字符串打印出来, 或者传递一个普通的对象, 来获取这个对象更多的信息.

再一次执行 `rake`, 然后...BAM, 一个无法关闭的蓝色弹层!  而且在你的终端窗口, 你应该在控制台里看到 "Hello again!". 并没有非常大的进步, 但是...至少现在我们知道了它是如何实现的.

![hello](images/1.png)

学到了什么?

- 通过 `motion create <ProjectName>` 创建 RubyMotion 应用
- 在`Rakefile`里, 配置你的应用, 导入依赖库
- 应用程序需要一个委托, RubyMotion 需要你在 `Rakefile` 设置它的值(或者使用默认值)
- AppDelegate 使用 `application:didFinishLaunchingWithOptions:` 作为首个进入点.
- 在项目目录中使用 `rake` 运行你的应用.

[前往下一章: 在屏幕上添加Views!](/2-views)

[rm]: http://www.rubymotion.com/

[macruby]: http://macruby.org/

[xcode]: http://itunes.apple.com/us/app/xcode/id497799835?mt=12

[buy]: http://sites.fastspring.com/hipbyte/product/rubymotion

[packages]: http://www.rubymotion.com/developer-center/articles/editors/
