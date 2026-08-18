<div align="center">
  <img src="docs/images/logo.png" alt="MediaPeek 图标" width="96">
  <h1>MediaPeek</h1>
  <p>一张藏在 Windows 屏幕顶部边缘的「正在播放」媒体卡片。<br>
  平时完全不占屏幕、点击穿透——你需要它的时候，它才出现。</p>
  <p><a href="README.md">English</a> | <b>中文</b></p>
  <img src="docs/images/edge-reveal.gif" alt="光标贴住屏幕顶边停留，卡片滑出" width="680">
</div>

## 为什么

想知道现在放到哪首歌，切一次窗口就打断一次心流。MediaPeek 把当前曲目（封面、
歌名、歌手）放在光标一抬就能够到的地方，置顶于一切窗口之上——包括全屏应用；
而闲置时它**不占用任何屏幕空间**，隐藏卡片的像素是真正点击穿透的。

它通过 Windows SMTC 与播放器通信，网易云音乐、QQ 音乐、Spotify、浏览器等常见
播放器装完即用，无需任何设置。

## 功能

### 贴边唤出

光标贴住屏幕顶边停留片刻（约 150ms，甩鼠标去点最大化窗口的关闭按钮不会误触），
卡片即刻滑出。悬停期间保持可见，离开约 600ms 后自动收起。

![贴边唤出](docs/images/edge-reveal.gif)

### 自动现身的「正在播放」

卡片也会自己冒头：网易云切歌时立即出现一次，之后播放中每分钟露脸 3.5 秒。
自动现身是两段式编排——先只滑出歌名，再把卡片拉长、展开封面与歌手。光标移上
自动现身中的卡片即可接管为常驻悬停。

![两段式自动现身](docs/images/auto-peek.gif)

卡片左侧的氛围光取自封面颜色，播放时边缘还有一道流光缓缓扫过——两者都严格
收敛在胶囊轮廓之内，不破坏点击穿透的承诺。

### 手势控制

默认没有任何按钮：点击卡片播放 / 暂停，左右横滑切歌（伴随一次横向回弹反馈），
切歌后卡片自动平滑过渡到新歌名的宽度。

![卡片手势](docs/images/gestures.gif)

### 按钮控制

不习惯手势？在托盘菜单勾选「按钮控制」，卡片右侧出现 上一首 / 播放暂停 /
下一首 按钮组。

![按钮控制](docs/images/buttons.gif)

### 常驻托盘

托盘图标左键唤出当前播放卡片，右键弹出菜单：开机启动、控制模式、界面语言、
检查更新（仅商店打包版）、重启与退出。

### 启动动画

每次启动都会播放一段从屏幕边缘长出的问候动画：首次安装运行是带使用引导的
欢迎卡，之后（含开机自启）是简短的迷你胶囊。

![首次运行欢迎卡](docs/images/welcome.gif)
![迷你问候胶囊](docs/images/hello.gif)

### 还有那些不吵的细节

- 始终置顶但不抢焦点，无任务栏按钮、不进 Alt+Tab——全屏看视频、打游戏时照样
  可以贴边唤出。
- 卡片宽度内容自适应：始终完整包裹歌名与歌手，不走马灯、不截断，切歌时宽度
  平滑过渡。
- 多语言界面（中文 / English）：默认跟随系统语言，随时可在托盘菜单切换。
- 单实例运行、异常兜底记日志，设置持久化在 `%AppData%\ImmersiveBar\settings.json`。

## 安装

- **微软商店（MSIX）**：上架准备中；打包版带自动更新与清单声明的开机自启任务。
  也可按[打包](#打包微软商店-msix)一节自行构建安装包。
- **绿色单文件 exe**：自包含单文件构建，放到哪都能跑——见[从源码构建](#从源码构建)。

要求 Windows 10 19041+（在 Windows 11 上开发）。

## 从源码构建

前置：.NET 8 SDK（含 Windows 桌面工作负载）。

```powershell
dotnet build ImmersiveBar.sln
dotnet test tests/ImmersiveBar.Tests
dotnet run --project src/ImmersiveBar
```

发布绿色单文件 exe：

```powershell
dotnet publish src/ImmersiveBar -c Release -r win-x64 --self-contained true `
  /p:PublishSingleFile=true /p:IncludeNativeLibrariesForSelfExtract=true
# 输出：src/ImmersiveBar/bin/Release/net8.0-windows10.0.19041.0/win-x64/publish/ImmersiveBar.exe
```

## 打包微软商店（MSIX）

```powershell
powershell -ExecutionPolicy Bypass -File packaging\Pack-Msix.ps1          # 未签名包（提交商店用）
powershell -ExecutionPolicy Bypass -File packaging\Pack-Msix.ps1 -Sign    # 开发证书签名（本机旁加载测试）
```

脚本会自动发布、拼装布局并校验打包出 `artifacts\msix\ImmersiveBar.msix`。
提交商店前：把 `packaging\Package.appxmanifest` 里 Identity 的 Name / Publisher /
PublisherDisplayName 换成 Partner Center「产品标识」页的正式值，并每次递增
`Identity@Version`。打包版的更新走商店通道，开机自启走清单 `StartupTask`
（而非注册表 Run 键）。

## 实现原理

- C# / WPF，.NET 8（`net8.0-windows10.0.19041.0`），**零第三方依赖**——WinRT
  SMTC 桥接与托盘图标都走原生 interop（`Shell_NotifyIcon`，不引 WinForms）。
- `ImmersiveBar.Core` 是纯逻辑层（视图模型 + SMTC 抽象，不引用 WPF，UI 调度靠
  注入），由 xUnit 单测覆盖；`ImmersiveBar` 是 WPF 壳（layered 点击穿透窗口、
  AppBar 停靠、动画编排、托盘、设置持久化、多语言）。
- 窗口铺满屏幕顶部但不预留任何空间（零高度 AppBar）；所有动画都是渲染线程
  驱动的 `DoubleAnimation`，高刷新率显示器上原生顺滑。
