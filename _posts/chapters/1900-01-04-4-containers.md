---
layout: default
title: Containers
description: Learn the basic UINavigationController and UITabBarController container controllers.
categories:
- chapters
---

# 容器 Containers

现在我们有了一个白色屏幕, 让我们装饰一下吧.

如果你已经使用过一些iOS应用程序，你无疑会注意到它们视觉上有些相似。首先，许多应用程序有一个黑色底部标签栏, 和一个带有标题的蓝色顶部导航栏. 这些在iOS里都是标准 "容器" 

有一个标签和一个蓝色的顶杆与黑底栏标题。这些都是标准的“容器”在iOS，分别称为 `UITabBarController` 和 `UINavigationController`。我们马上就要来接触这些容器.

这些容器都是 `UIViewController` 的子类, 而实际上又管理着其他`UIViewController`. 很生猛, 对吧? 他们就像普通的控制器一样有一个`视图`, 而这些 "子控制器" 就会被当作子视图添加进去. 幸运的是, 我们从来都不需要真正关心这货*如何*运行的, 这些容器控制器都有友好的API, 可以用来处理控制器, 而不是视图.

## 导航控制器 UINavigationController

让我们从最常用的容器开始, 导航控制器 `UINavigationController`. 在对象管理上, 导航控制器使用了导航堆栈, 沿着一个可以看的到的水平的路径将它们进行推入或者弹出. Mail.app 就使用了这个流程: 账号 -> 某一个账号 -> 收件箱 -> 邮件. 导航控制器非常好用, 它会自动的替你操作回退按钮; 你所要做的事情就是推入或者弹出控制器.

![Navigation controller in Mail.app](images/nav_bar.png)

回到 `AppDelegate`, 将我们的 `rootViewController` 改为使用一个 `UINavigationController`:

```ruby
  ...
  controller = TapController.alloc.initWithNibName(nil, bundle: nil)
  @window.rootViewController = UINavigationController.alloc.initWithRootViewController(controller)
  ...
```

`initWithRootViewController` 会将controller传值进导航控制器, 并且还会随之启动一个栈.

在运行应用之前, 再在 `TapController` 多做点修改:

```ruby
  ...
  self.view.addSubview @label

  self.title = "Tap"
  ...
```

`rake` 然后再看看我们变得稍微有点好看的应用:

![Navigation controller](images/1.png)

不错. 现在让我们让它做点事情. 我们会添加一个导航栏按钮, 这个按钮会将更多的 `TapController` 实例推入到导航堆栈里.

实际上, 

你可以把按钮放在屏幕的顶部的导航栏。例如,邮件。应用这个“编辑”按钮。这些按钮是“UIBarButtonItem”的实例,有大量的配置选项(想使用文本吗?一个图像?一个系统图标?各种各样的有趣的东西)。

You can actually put buttons in the navigation bar at the top of the screen. For example, Mail.app does this for the "Edit" button. These buttons are instances of `UIBarButtonItem`, which has loads of configuration options (want to use text? an image? a system icon? all sorts of fun stuff).

In `TapController`, add the button in `viewDidLoad`:

```ruby
def viewDidLoad
  ...
  self.title = "Tap"

  right_button = UIBarButtonItem.alloc.initWithTitle("Push", style: UIBarButtonItemStyleBordered, target:self, action:'push')
  self.navigationItem.rightBarButtonItem = right_button
end
```

Should be pretty plain what this does. We create a `UIBarButtonItem` instance with a title and a style. The `style` property determines how our button looks (either plain, bordered, or "done"; play around to see the differences). We then set it as our controller's `navigationItem`'s `rightBarButtonItem`. Every `UIViewController` has a `navigationItem`, which is how we access the information displayed in the blue bar at the top. (NOTE that `UINavigationItem` *isn't* a `UIView`! so you can't arbitrarily add subviews to it).

What's the `target`/`action` business? Well, this is where the original Objective-C SDK leaks into Ruby-land =(. Up until very recently, you couldn't pass anonymous functions as callbacks in Objective-C; as an alternative, APIs would pass objects and the name of a function to call on that object. We `do` this operation in Ruby with blocks and lambdas, but sadly the older iOS APIs show their age.

Anyway, `target` is the object you want to call the `action` function on. Let's implement it in our `TapController` so you get a better idea:

```ruby
...
  def push
    new_controller = TapController.alloc.initWithNibName(nil, bundle: nil)
    self.navigationController.pushViewController(new_controller, animated: true)
  end
...
```

Make more sense now? When the user taps the bar button, `push` gets called on the `target`, which in this case is our controller.

In addition to `navigationItem`, `UIViewController`s also have a property for their `navigationController`, if available. On the nav controller we call `pushViewController` which pushes the passed controller onto the stack. By default, the navigation controller will also show a back button which handles popping the current controller for us (normally we call `popViewControllerAnimated:` on the navigation controller). Kind of neat, right?

Let's have one last bit of fun before we run the app. Have the controller's title reflect its position in the navigation stack, like so:

```ruby
  def viewDidLoad
    ...
    self.title = "Tap (#{self.navigationController.viewControllers.count})"
    ...
  end
```

Now `rake` and observe our dynamic controllers!

![pushable controllers](images/2.png)

Now, I said we'd cover `UITabController`s too, so let's get to it.

## UITabBarController

![Navigation controller in Mail.app](images/tab_bar.png)

Tab controllers are a lot like `UINavigationController` and other container controllers: it has a list of `viewControllers` which are presented within the container's "chrome" (the black bar). However, unlike the other containers, `UITabBarController`s are only supposed to act as the `rootViewController` of the window (i.e. you shouldn't push a tab bar controller inside a navigation controller).

In `AppDelegate`, let's make a small change:

```ruby
  ...
  controller = TapController.alloc.initWithNibName(nil, bundle: nil)
  nav_controller = UINavigationController.alloc.initWithRootViewController(controller)

  tab_controller = UITabBarController.alloc.initWithNibName(nil, bundle: nil)
  tab_controller.viewControllers = [nav_controller]
  @window.rootViewController = tab_controller
  ...
```

Unlike other code examples, this doesn't have anything new! We create the `UITabBarController` like a normal `UIViewController` and set its `viewControllers` to an array with our navigation controller.

`rake` and check it out!

![tab bar controller](images/3.png)

Kind of...underwhelming, I guess. Let's start to pretty it up by adding an icon to our tab bar.

Much like `navigationItem`, every `UIViewController` has a `tabBarItem` property, which accepts an instance of `UITabBarItem`. We can use this object to customize the icon, title, and other appearance options of our controller's tab.

Override `initWithNibName:bundle:` in TapController, and go ahead and create such an object:

```ruby
  ...
  def initWithNibName(name, bundle: bundle)
    super
    self.tabBarItem = UITabBarItem.alloc.initWithTabBarSystemItem(UITabBarSystemItemFavorites, tag: 1)
    self
  end
  ...
```

This is one initializer for `UITabBarItem`; you can also use `initWithTitle:image:tag:` if you want to supply a custom image and title. If you do use a custom image, it needs to be a 30x30 black and transparent icon.

`initWithTabBarSystemItem:` makes our lives a little easier for demonstrating a tab icon, but it will force the title to correspond to the system's image (in this case, "Favorites").

Why do we put it in `initWithNibName:bundle:`? Because we want it to create the `tabBarItem` as soon as the controller exists, regardless of whether or not its `view` exists. If you put it in `viewDidLoad`, then it might not get created when the app launches (tab bar controllers only load each child controller when its first accessed by the user).

One more thing! Let's make another tab. We don't really have anything to do in this other tab yet, so let's just make an empty `UIViewController` with a different background color in `AppDelegate`:

```ruby
  ...
  other_controller = UIViewController.alloc.initWithNibName(nil, bundle: nil)
  other_controller.title = "Other"
  other_controller.view.backgroundColor = UIColor.purpleColor

  tab_controller = UITabBarController.alloc.initWithNibName(nil, bundle: nil)
  tab_controller.viewControllers = [nav_controller, other_controller]
  @window.rootViewController = tab_controller
  ...
```

`rake` and voila! A whole bunch of container controllers! They don't do a whole lot, but you can easily see how these few classes form the building blocks of 80% of iOS apps.

![multiple tabs](images/4.png)

## Subway Up

Let's rundown what we covered:

- The iOS SDK uses `UINavigationController`s and `UITabBarController`s for containing "child" view controllers.
- `UINavigationController` uses 'pushViewController:animated:' and 'popViewControllerAnimated:' to control the stack.
- Use `controller.navigationItem` to change the buttons and other options of the top blue bar for a controller.
- `UITabBarController` uses `viewControllers=` to control its children; note that `UITabBarController` should *only* be used as the `rootViewController` of a window.
- Use `controller.tabBarItem` to change the tab icon and title displayed for that controller.

[Simply walk to Mordor/the next chapter to learn about Tables!](/5-tables)
