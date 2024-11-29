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

和控制台/ASP.NET Core C#项目不同，WinUI 3项目没有`Program.cs`和手写的`Main`方法。`App.xaml.cs`里面的`App`类是整个应用的起始点。当然，`Main`方法依旧存在，但是是由项目构建系统自动为我们生成的，它会为我们完成构建一个App实例，并且帮助我们在关闭的时候销毁它。

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

> XAML是一种用来描述用户界面的语法。在语法上，它是XML的扩展，拥有强大的表达能力。
> 
> XAML在用途上类似于编写网页使用的HTML，它取代了传统的使用代码构造界面，而使用一种更具有结构化的语言——XAML——来描述界面，并且根据XAML生成对应的代码，实现代码和界面的分离，让设计师能够不用去编写代码，而只需要通过XAML来设计界面。

好像有很多复杂的东西。不过我们先关注最后面的`StackPanel`和`Button`。

### `<Button ... />`

我们的窗口中的那个`Click Me`按钮正是由`Button`标签描述的。一个标签有标签类型、多个属性和内容，一个标签需要一对开始和结束，比如`Button`中的`<Button ...> ... </Button>`就是一对开始和结束标签。如果标签内容为空，你也可以写成自闭合的形式，就像`<Button />`

#### 内容

这个`Button`的内容为"Click Me"，它对应着窗口中按钮显示的内容
![[Pasted image 20241127180331.png]]
所有拥有内容的控件都继承自`ContentControl`。

#### 属性

除了内容之外，这个`Button`还具有两个属性“x:Name”和"Click"。属性，

##### `x:Name`

WinUI 3采用了代码与界面相分离的模式，界面在XAML中，而代码则在对应的后台代码（code-behind），即`*.xaml.cs`里。如果你希望在后台代码里访问一个控件，需要给这个控件取一个在C#代码中的名字，便是由`x:Name`完成的。它为你在MainWindow的分部类里生成了一个对应名字的private字段，以便你在代码中去访问它。你可以通过设置` x:FieldModifier`属性来让它生成public的字段。

打开后台代码， 在`myButton_Click`方法中`myButton.Content = "Clicked";`就使用了`x:Name`中的名字来修改`myButton`的内容。

##### `Click`

设置点击按钮时由哪个事件处理函数来处理。在这里每次点击按钮都会触发`myButton_Click`事件，修改`myButton`的内容为Clicked。

##### `Content`

我们不仅可以在开始和结束标签中间书写内容，实际上内容只是一个叫`Content`的属性，这就是为什么我们在点击事件里修改它的内容时使用的是`Content`。

![[Pasted image 20241127182018.png]]
这二者是等价的。并且你不能同时用这两种方式设置内容，因为一个属性只能被设置一次。

### `<StackPanel ... />`

`StackPanel`是一种控件的布局。布局可以用来描述控件如何被排列在界面上。`StackPanel`是一种将它里面的元素全部垂直或水平排列的布局，你可以试试多添加几个按钮：
![[Pasted image 20241127182815.png]]
它有一些常用属性：

| 属性                  | 作用          | 值                         |
| ------------------- | ----------- | ------------------------- |
| Orientation         | 水平或垂直排布控件   | Vertical（默认）, Horizontal  |
| HorizontalAlignment | 相对于父元素的水平位置 | Center,Left,Right,Stretch |
