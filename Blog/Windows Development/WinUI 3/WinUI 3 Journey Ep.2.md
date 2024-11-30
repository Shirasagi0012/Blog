# WinUI 3 | Ep2. 认识 WinUI 3 项目

在上一节中，我们完成了环境的配置，并且创建并运行了第一个 WinUI 3 项目。接下来，让我们来看看它的构成：

![项目构成](./Pasted%20image%2020241127173014.png)

## 入口点

打开`App.xaml.cs`（这个文件在App.xaml的下级）

```csharp
namespace Tutorial
{
    public partial class App : Application
    {
        public App()
        {
            this.InitializeComponent();
        }

        protected override void OnLaunched(Microsoft.UI.Xaml.LaunchActivatedEventArgs args)
        {
            m_window = new MainWindow();
            m_window.Activate();
        }

        private Window? m_window;
    }
}
```

App类会为我们完成整个应用的初始化，包括它需要的各种资源的准备，初始化完成后会在启动时执行`OnLaunched`方法，创建并展示一个窗口。

和控制台/ASP.NET Core C#项目不同，WinUI 3项目没有`Program.cs`和手写的`Main`方法。`App.xaml.cs`里面的`App`类是整个应用的起始点。

> 当然，`Main`方法依旧存在，但是是由项目构建系统自动为我们生成的。

## 主窗口

WinUI 3和它的前辈UWP，WPF一样，使用了XAML作为编写界面的语法。

打开`MainWindow.xaml`，让我们看看XAML长什么样：

```xml
<?xml version="1.0" encoding="utf-8"?>
<Window
    x:Class="Tutorial.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:Tutorial"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d"
    Title="Tutorial">

    <StackPanel Orientation="Horizontal" HorizontalAlignment="Center" VerticalAlignment="Center">
        <Button x:Name="myButton" Click="myButton_Click">Click Me</Button>
    </StackPanel>
</Window>
```

## XAML

> XAML是一种用来描述用户界面的语法。它取代了传统的使用代码构造界面，而使用声明式的语法来描述界面，实现代码逻辑和界面声明的分离，让设计师能够不编写代码，只需要通过XAML来设计界面。

XAML使用了与XML相似的语法。如果你已经熟悉了XML，那么下面一部分对你而言应该是基础知识：

### 对象元素

在C#之中，一切皆对象。XAML中可以非常方便的声明一个对象元素：

```xml
<Button>Hello!</Button>
```

简单的一行，我们就声明了一个Button对象，它大致相当于下面这段C#代码

```csharp
Button button1 = new Button()
{
    Content = "Hello!"
};
```

XAML中用`< >`包裹起来的被称作一个标签，和HTML类似。标签有头也有尾，结束标签在最开头有一个`/`。

每一个XAML元素都对应着C#中的一个类。

#### 内容

起始标签和结束标签中间夹着的被称为内容(Content)，在这里它是一段文字"Hello!"，但你也可以使用另一个标签形成嵌套：

```xml
<Button>
    <Image Source="/Assets/WinUI3.png" />
</Button>
```

自闭合标签：如果一个标签内容部分为空，可以省略掉结束标签，而在起始标签的结尾处加上一个`/`表示这个标签是自闭合的，eg.`<Button />`

这段XAML相当于：

```csharp
Button button1 = new Button()
{
    Content = new Image()
    {
        Source = "/Assets/WinUI3.png",
    }
};
```

#### 属性

上面的例子中，`<Image Source="/Assets/WinUI3.png" />`的`Source="..."`就是设置一个属性的值的语法。对于一个元素，它对应的类的可访问属性都能通过这种语法赋值。

内容也是一种属性，叫`Content`，所以以下两种写法是等价的：

```xml
<Button>Hello!</Button>
<Button Content="Hello!" /> 
```

当然，这样写的话，你就不能设置内容的值为另一个标签了。前面`Button`嵌套一个`Image`的例子就不能改写成这样。

如果你希望使用XAML元素设置一个属性的值也是可行的。这种方式虽然更繁琐，但是能使用另一个XAML元素作为它的内容。这两种写法是等价的，可见，使用属性语法会让代码更简洁

```xml
<Button>
    <Button.FontSize>
        24
    </Button.FontSize>
    Hello!
</Button>

<Button FontSize="24" Content="Hello!" />
```


