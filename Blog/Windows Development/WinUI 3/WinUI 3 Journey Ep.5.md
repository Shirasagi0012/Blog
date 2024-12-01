# WinUI 3 | Ep.5 实现MVVM

接下来，我们一起制作一个实现MVVM架构的简单音乐播放器应用。

新建一个项目。首先，我们安装几个需要用到的NuGet包：
- TagLibSharp - 用于读取音乐标签
- CommunityToolkit.Mvvm - 可以帮助我们实现MVVM架构，减少代码量

这次，我们先不做界面，而是从ViewModel着手。
### 创建 ViewModel 类

按照惯例，我们会将所有ViewModel放在项目里的ViewModels文件夹内，Model放在Models文件夹，而XAML页面放在Pages文件夹。创建一个`PlayerViewModel.cs`的类

```csharp
public partial class PlayerViewModel : ObservableObject
{
    [ObservableProperty]
    Song? _nowPlaying;
}
```

在这里，我们使ViewModel继承自ObservableObject。这是CommunityToolkit.Mvvm中的东西，提供了数据绑定更新需要的基础设施，包括实现INotifyPropertyChanged接口，不需要像上一个应用一样手动实现。对于需要提供更新通知的属性，我们只需要写它的backing field，加上ObservableProperty特性就可以了。CommunityToolkit.Mvvm会自动生成对应的属性。

以及我们需要的Model `Song.cs`：

```csharp
public class Song (string path)
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

这里使用TagLib完成音乐文件标签的读取。
### Data Binding 数据绑定

数据绑定使UI元素和数据源之间建立连接，从而实现数据的自动更新和显示。因此我们需要有一种方式通知数据被更改了，需要更新。这就是为什么在上一节我们让MainWindow实现了INotifyPropertyChanged接口。
