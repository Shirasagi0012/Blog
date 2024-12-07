# WinUI 3 | Ep.5 实现MVVM

接下来，我们一起制作一个实现MVVM架构的简单音乐播放器应用。

新建一个项目。首先，我们安装几个需要用到的NuGet包：
- `TagLibSharp` - 用于读取音乐标签
- `CommunityToolkit.Mvvm` - 可以帮助我们实现MVVM架构，减少代码量。如果你使用最新的.NET 9，还可以将语言版本调整为`preview`，享受基于部分属性的`CommunityToolkit.Mvvm`源生成器

回顾MVVM架构，其由三个部分组成，Model、ViewModel、View。MVVM实现了分层，在这三部分中，View依赖于ViewModel，ViewModel依赖于Model。因此我们可以先从最底层的，应用业务逻辑所在的Model着手。

为了更好地体现MVVM分层的逻辑关系，我们可以为每一层建立一个文件夹。我们可以新建三个文件夹：View、ViewModel、Model。但是，由于View和ViewModel通常是一对一的关系，且紧密关联，我们也可以将View和ViewModel合二为一，避免在两个文件夹中来回切换。

## Model层

首先是Model文件夹中的Song模型：

```csharp
public partial class Song (string path) // 由于CsWinRT源生成器，可能需要partial
{
    public string Path { get; set; } = path;
    public string? Title { get; set; }
    public string? Artist { get; set; }
    public string? Album { get; set; }
    public TimeSpan Duration { get; set; }
    BitmapImage Picture { get; set; }
    
	public void ReadData ( )
	{
		using var tagFile = TagLib.File.Create(Path);
		Title = tagFile.Tag.Title;
		Artist = tagFile.Tag.FirstPerformer;
		Album = tagFile.Tag.Album;
		Duration = tagFile.Properties.Duration;
	
		var image = (tagFile.Tag.Pictures.Length > 0) 
						? tagFile.Tag.Pictures[0].Data.Data 
						: null;
		var bitmap = new BitmapImage();
		if ( image != null )
		{
			using var stream = new InMemoryRandomAccessStream();
			using var writer = new DataWriter(stream.GetOutputStreamAt(0));
			writer.WriteBytes(image);
			writer.StoreAsync().GetResults();
			bitmap.SetSource(stream);
		}
		Picture = bitmap;
	}
}
```

这里使用TagLib完成音乐文件标签的读取。读取完标签后，将图片转换为BitmapImage需要的数据流，供显示用。

除此之外，我们还需要一个Player模型。

```csharp
public partial class Player // 由于CsWinRT源生成器，可能需要partial
{
	public Song NowPlaying { get; set; }
	public List<Song> Playlist { get; }
}
```

这就是整个Model层的设计。由于我们严格遵循了MVVM架构，我们的模型完全不会被UI界面相关的东西污染，非常干净。

## ViewModel 与 View 层

接着是ViewModel层和View层。我们将这二层放在同一个文件夹中。在上一节中，我们直接在MainWindow中编写界面，但实际上不应该这么做，因为Window和其它的控件是不同的，它不继承自`FrameworkElement`类，存在局限性。在现代的应用中，我们使用`页面 Page`来承载控件。而Window则应该作为Page的载体。

因此，我们先新建一个页面`MainView.xaml`和它对应的ViewModel类 `MainViewModel.cs`。在VS2022的新建菜单中，可以找到WinUI 3的页面。

View的用途之一是展示ViewModel。所以我们要在View的后台代码中获取一个对应的ViewModel。

```csharp
public sealed partial class MainView : Page
{
    public MainViewModel ViewModel;

    public MainView ( )
    {
        ViewModel = new();
        this.InitializeComponent();
    }
}
```

### 显示页面

在编写View和ViewModel前，先得让程序能显示出来我们的页面。由于MainWindow已经不再需要显示控件和包含程序逻辑了，和一个普通的，空白的Window没啥区别，所以我们可以直接删掉它，然后在`App.xaml.cs`中简单地new一个Window即可。

然后设置这个Window的内容为前面创建的页面，顺便利用对象初始化器语法设置其它属性。

```csharp
public partial class App : Application
{
    public App ( )
    {
        this.InitializeComponent();
    }

    protected override void OnLaunched (Microsoft.UI.Xaml.LaunchActivatedEventArgs args)
    {
        MainWindow = new Window()
        {
            Content = new MainView(),
            Title = "WinUI 3 Journey",
            SystemBackdrop = new MicaBackdrop(),
            ExtendsContentIntoTitleBar = true
        };
        MainWindow.Activate();
    }

    public static Window MainWindow { get; private set; } = null!;
}
```

在页面中随便写点东西，运行，可以看见能正常显示页面。

![[Pasted image 20241206160018.png]]

> **注意**
> VS 2022的页面新建模板中，页面有默认背景颜色，会掩盖MicaBackdrop。可以将背景颜色设置为Transparent。

### 实现页面到ViewModel的简单数据绑定

成功让页面显示出来，接着我们来探索如何让页面能展示ViewModel中的数据。在ViewModel中随便写一个字符串属性`public string Title => "Hello from ViewModel!";`，然后在页面XAML中使用`{x:Bind ...}`绑定到它。

```xml
<TextBlock Style="{ThemeResource TitleLargeTextBlockStyle}"
		   VerticalAlignment="Center" HorizontalAlignment="Center" 
		   Text="{x:Bind ViewModel.Title}}"/>
```

`x:Bind` 会在后台代码中寻找指定的属性。目前我们的后台代码中有一个ViewModel属性，而ViewModel中有Title属性，使用和C#中一样的`.`连接即可。

![[Pasted image 20241206161825.png]]

当然，我们还没实现数据绑定的更新通知。在上一节中，我们手动实现了`INotifyPropertyChanged`接口以实现通知，但是那样太麻烦了。若是每一个需要更改通知的属性都得写成这样
```csharp
private int clickCount = 0;

int ClickCount
{
	get => clickCount;
	set
	{
		clickCount = value;
		OnPropertyChanged(nameof(TitleText));
	}
}
```

但是在C#中，有一个强大的功能叫做源生成器。它在你编写代码的时候和编译时执行，作用就是对你的代码作一些修改。在开头安装的`CommunityToolkit.Mvvm`这个社区MVVM工具包中，就提供了一个用来自动生成带有更改通知的属性的源生成器。要让它为我们生成对应的属性很简单，只需要在一个字段前加上特性`[ObservableProperty]`即可。同时，这个类也需要继承自社区MVVM工具包中的`ObservableObject`。

```csharp
public class MainViewModel : ObservableObject
{
    [ObservableProperty]
    private string _title = "Hello from ViewModel!";
}
```

它会自动生成一个`public string Title {get ...; set ...; }`属性，并提供更改通知功能。不过，字段的命名必须满足下划线开头或小写字母或`m_`开头，社区MVVM工具包会根据这个名字生成对应的`public`属性

由于C#不支持多重继承，在不能直接继承自`ObservableObject`的场合，可以使用`[ObservableObject]`特性：

```csharp
[ObservableObject]
public class MainPage : Page
{
	// ...
	
    [ObservableProperty]
    private string _title = "Hello from ViewModel!";
}
```

>**提示**
>
>在.NET 9中加入了半自动属性（部分属性 partial property），这让我们可以不用再写所谓的backing field，只需要一个属性即可。最新版本的`CommunityToolkit.Mvvm`已经能够利用这一点，不再需要遵守字段的命名规范了
>
>```csharp
>public partial class MainViewModel : ObservableObject
>{
>    [ObservableProperty]
>    public partial string Title { get; set; } = "WinUI 3 Journey";
>}
>```

### 命令绑定

此前，为了响应用户点击按钮的操作，我们的做法是去监听按钮点击的事件。但这种做法存在一定的不足：
1. 事件处理函数只能位于后台代码。这意味着很可能会包含用于直接操作 UI 的代码，造成业务逻辑和界面逻辑的混杂
3. 事件处理函数不能动态更换。
4. 代码冗杂。对于不同的事件，它们对应着不同的委托，我们需要多个事件处理函数来做同一件事。
5. 复用困难。由于处理事件的代码在页面的后台代码中，你很难从其他页面使用。

为了解决这个问题，MVVM架构中，用户的操作所执行的业务逻辑被抽象成命令（保存文件，删除一个项目，暂停视频……），而并不关系用户如何触发命令，那是View层该考虑的事情，从而实现了Model和View的解耦。

比如程序有一个保存功能。按下`Ctrl-s`，点击保存按钮，在关闭程序时弹出的对话框里点击保存，定时自动保存——有非常多的触发方式，如果我们使用事件，那么对于每一种触发方式都要单独处理。但是命令并不关心是谁触发的，只要被执行就是相同的逻辑。

在WinUI 3中，命令是一个接口`ICommand`。其包含三个参数。

```csharp
event EventHandler? CanExecuteChanged;
bool CanExecute(object? parameter);
void Execute(object? parameter);
```

要实现一个自己的命令，只需要继承它，实现这几个接口就行了。

很显然每个命令都要继承自`ICommand`接口会使代码非常冗杂且写起来很麻烦。庆幸的是在`CommunityToolkit.Mvvm`中提供了一个`ICommand`的通用实现：`RelayCommand`类，可以让我们方便的定义一个命令。这是它的构造函数：

```csharp
public RelayCommand (Action execute) { }

public RelayCommand (Action execute, Func<bool> canExecute) { }
```

在ViewModel里使用也很简单：

```csharp
[ObservableProperty] private RelayCommand _changeTitleCommand;

[ObservableProperty] private string _title = "WinUI 3 Journey";

public MainViewModel ( )
{
	_changeTitleCommand = new RelayCommand(
		( ) =>
		{
			Title = "Clicked! And the button cannot be clicked again.";
			c.NotifyCanExecuteChanged();
		},
		( ) => Title == "WinUI 3 Journey"
	);
}
```

对于按钮，要绑定到`ICommand`，只需要设置其`Command`参数`Command="{x:Bind ViewModel.ChangeTitleCommand}"`。

如果你的ICommand并不会改变，不需要属性更改通知，你也可以不使用`[ObservableProperty]`，简单的get-only属性即可。

>**注意**
>命令和事件并不冲突。命令是为了将业务逻辑从原来的事件处理函数中分离出来，进一步完善解耦。而对界面的操作，比如按下按钮后播放动画，仍然需要处于后台代码的事件处理函数。

### 集合绑定

前面完成了简单的绑定。但是我们绑定到的都是单个属性，集合同样能被绑定。但是集合的绑定和普通属性的绑定不同。在集合中添加/删除/修改并不会像修改属性的值那样能产生通知，我们需要一种能在发生更改时产生通知的集合。

在WinUI 3中内置了`ObservableCollection<>`这一集合类型，使用它即可。








