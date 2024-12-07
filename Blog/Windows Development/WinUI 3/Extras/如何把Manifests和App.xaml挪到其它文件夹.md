#WinUI #Tips #HowTo

通常，我们会使用VS2022自带的WinUI 3单项目打包应用模板来创建项目。这个空白项目由一个塞了些对MSIX包必要的图像资源，还有几个文件——`App.xaml`、`MainWindow.xaml`、`app.manifest`、`Package.appxmanifest`。

它们总是没啥逻辑地堆在项目根目录下，根目录下文件夹多了之后看着让我很烦躁，因此我便琢磨着怎样才能给它们几个文件挪个窝。

# 1. `MainWindow.xaml`

这个很好处理，它并非什么关键文件，删掉便是。我们可以直接在`App`类的`OnLaunch()`方法下`new`一个`Window`出来，然后设置这个`Window`的`Content`、`SystemBackdrop`等，完全没必要单独搞个XAML文件。

# 2. `app.manifest` & `Package.appxmanifest`

`app.manifest`是应用程序清单，包含了一些应用程序的能力的声明，比如`DpiAwareness`和支持的系统版本。要给它挪位置只需要在项目文件中，将已经存在的`<ApplicationManifest>`的值更改为移动后`app.manifest`的路径

`Package.appxmanifest`则是MSIX包的清单，它包含了应用的权限，安装包的图标等等，对于打包应用而言也是必不可少的。不过对于非打包应用而言，它并不是必须的。要移动它，需要在项目文件的：
- `ItemsGroup`中新增一条`<AppxManifest Include="[Package.appxmanifest的路径]" />`。
- `PropertyGroup`中手动指定`<WindowsPackageType>MSIX</WindowsPackageType>`。估计是存在Bug，会导致项目根目录下链接两个与Windows App SDK部署有关的`.cs`文件，而这两个文件本应是只在非打包模式（即`WindowsPackageType`值为`None`）时才会生成。不过这没有什么影响，似乎也没什么好的办法删除掉它们，忽略即可。

> **注意**
>
> 设置`<WindowsPackageType>MSIX</WindowsPackageType>`会导致非打包模式不可用。

# 3. `App.xaml`

`App.xaml`也比较诡异。在根目录下，编译时会自动生成一个`App.i.g.cs`，其中包含应用的`Program`类和`Main()`方法，应用就是从这里启动的。但是给它挪个窝后就不会生成了😡，矫情。

这可能是因为移动后`App.xaml`的生成操作从默认的`应用程序定义`变为了`页`导致的。但是我把它改回`应用程序定义`后报错了，提示：

```
XAML files 'Application\App.xaml' and 'Application\App.xaml' have the same project path 'Application\App.xaml'.  Each file must have a unique project path.
```

😇奇怪的报错，What can I say. 在互联网上一番查询后可以确认将`App.xaml`挪到其它地方并不是什么好主意，但既然我都写了这篇文章了，那为何不去研究下怎样才能实现呢？毕竟，它虽然诡异，说到底也只是个要被编译的XAML文件。

找了半天资料，最终找到了报错的原因。所有的XAML文件，除了根目录下的`App.xaml`，生成操作都默认为页，导致冲突。因此，在项目文件`ItemGroup`中，先删掉它的生成操作就好了😋，添加`<Page Remove="Application\App.xaml" />`


![[Pasted image 20241206011622.png]]

最终战绩：~~根目录下三个文件换两个文件，优势在我~~