# WinUI 3 | Ep1. 初识WinUI 3 & 环境配置
![[Pasted image 20241127153149.png]]
简单概括，WinUI 3包含在Windows App SDK中，是微软为现代 Windows 应用开发而制作的最新的UI框架，也是Windows 11 24H2中许多部件的。它来源于已经坟头草三尺高的UWP平台的UI控件库，WinUI 2，带有一套完全符合Windows 11设计风格的控件，也是微软 Fluent Design System 设计语言的官方实现。

## 什么？又一个UI框架？

Windows 马上就要迎来它的40岁生日了（Windows 1.0 于 1985年11月20日问世），毫无疑问，Windows拥有超长的历史，并且更令人惊叹的是，在这40年间，它一直保持着极高的向后兼容性。即使是在今天的Windows 11 24H2系统上，你仍然可以运行来自于Windows 3.1上面的不少软件。

虽然它在相当程度上仍保持对远古软件的兼容，但作为一个一直保持活跃的平台，它总是要向前走的，不能只是停留在当下或者过去。

在这漫长的时间里，Windows平台的软件开发也经历了多次进化。微软自己的Windows平台原生应用开发框架从最早期Win32，到MFC，到WinForm、WPF，到Metro App、UWP，而后是发布于Windows 11前夕的2020年的Project Reunion，也就是Windows App SDK。

这么多不同的应用开发框架，它们都能良好的共存在最新的Windows上，如何选择成了一个棘手的问题。尤其是在Windows 10时期，发布的无数现代功能与特性（原生的触摸支持，现代的应用权限管理，应用安全性，MSIX安装包，Windows系统集成等等）全部是UWP Only的。UWP与旧世界的Windows完全撇清了关系，为实现微软心中全平台大一统的构想抛弃了一切包袱。但代价是，它与传统Windows应用开发之间巨大的割裂阻止了开发者去将自己的应用现代化并采用那些现代特性。最终，UWP平台因为开发难度太大，与旧平台完全割裂，生态不够齐全，随着Windows Phone的死去而宣告消亡。

### Project Reunion

UWP虽然失败了，但是下一代Windows依旧需要一套应用开发框架。
- 它需要一个能够轻松制作出符合Windows设计语言的UI框架。
- 它需要兼容现有的老程序，让它们能轻松加入Windows的新特性完成现代化。
- 它需要能继承UWP的遗产，因为Windows的相当多系统应用都是基于UWP的。
因此Project Reunion诞生了，它的使命就是完成整个Windows应用开发生态的统一，让无论是老旧的应用，还是新的应用，都能平等的享受Windows所提供的最新特性。UWP先进的UI也被分离出来，成为了WinUI 3。尽管桌面端开发已不再是微软发力的重点，但随着 WinUI 3 发展了几年，到今天日渐成熟。并且从Windows 11 24H2开始，越来越多的系统组件从UWP转向WinUI 3。

### 你说得对，但是为什么选择它？

它是现在和下个大版本Windows的主要应用开发框架。它有开箱即用的漂亮界面，能和Windows本身完美的融合在一起，原生支持各种Windows的现代特性。它的Composition层（也叫可视化层）将动画和特效与应用进程相分离，可以高性能的实现其它原生框架做不到的华丽的效果。如果你只是想制作一个在Windows上运行的应用，那么 WinUI 3 就是你的最佳选择。

好了，废话不说了，那我们开始吧！

## Quick Start | 第一个 WinUI 3 应用

在这篇教程中，我们将使用Visual Studio 2022与.NET 9，使用C#语言进行开发。

> 作为原生UI框架，WinUI 3除了主推的C#，亦支持C++（官方支持）和Rust等语言进行开发。 
> 如果你希望使用它们进行开发，这篇教程的部分内容对你可能不会有很大帮助。

## 配置开发环境

首先需要从安装Visual Studio 2022开始。WinUI 3应用在Visual Studio开发才能获得最好的调试体验与功能支持。

### 安装 Visual Studio 2022

如果你已经安装了Visual Studio 2022，可以跳至下一小节。

除了到官网下载或者去Microsoft Store下载，对于Windows 11，你还可以通过Windows的包管理器`winget`快速下载最新版的VS 2022。打开Windows Terminal，输入以下命令并执行以安装正式版的VS 2022。

```powershell
winget install Microsoft.VisualStudio.2022.Community
```

等待下载与安装结束，会自动打开Visual Studio Installer。

### 安装工作负载

编辑你的VS 2022安装，确保勾选
- .NET 桌面开发
- Windows 应用程序开发

![[Pasted image 20241127164841.png]]

> **可选：** 如果你的系统中没有安装Git for Windows，你可以在单个组件中寻找到”适用于 Windows 的 Git“，Visual Studio会在登陆微软账户与GitHub账号后，自动为你完成Git相关配置。

### 开启 Windows 开发人员模式

打开设置>系统>开发者模式，启用开发人员模式。按提示进行操作。

> 强烈推荐在这里将默认终端更换为`Windows Terminal`，如果你没有这么做过的话。  
> 也推荐你在这里启用允许本地未签名的Powershell脚本的执行。

![[Pasted image 20241127165503.png]]

### 创建新项目

打开Visual Studio 2022，创建一个新项目。选择"空白应用，已打包（桌面版中的 WinUI 3）"

![[Pasted image 20241127170403.png]]

创建后，你会看见这样的界面：

![[Pasted image 20241127171443.png]]

> 你的界面布局和我会有些许不同，不过这并不重要。

### 试着运行吧！

确保上方实心绿色三角形右侧的当前启动项目选择的是“你的项目名（Packaged）”，点击绿色实心三角形，在编译和部署完成后，就会开始运行与调试了。并且Visual Studio 会自动切换为调试界面布局。当你看见一个带有一个按钮的窗口的时候，代表着你完成了环境的配置。恭喜迈出开发的第一步！

![[Pasted image 20241127171913.png]]