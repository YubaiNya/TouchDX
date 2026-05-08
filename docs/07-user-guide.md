# 07. 完整使用与配置指南 (User Guide)

欢迎使用 TouchDX！这份指南将教你如何从零开始，使用平板或手机控制电脑上的 maimai DX。

## 第 1 步：安装 PC 端前置要求
1.  **游戏环境**：确保你已经拥有一份可运行的 maimai DX (Windows) 游戏程序。
2.  **MelonLoader**：这是一个通用的游戏 Mod 框架。
    *   下载并安装最新版的 MelonLoader。
    *   选择你的游戏启动程序 (如 `Sinmai.exe`) 进行安装注入。

## 第 2 步：安装 TouchDX-Mod
1.  前往 GitHub 仓库 [YubaiNya/TouchDX-Mod](https://github.com/YubaiNya/TouchDX-Mod/releases) 下载最新的 Release 压缩包。
2.  解压得到 `MaiRemoteTouchMod.dll`。
3.  将该文件放入游戏根目录下的 `Mods` 文件夹中（如果在第一步中安装好了 MelonLoader，通常会有此文件夹；如果没有请手动创建）。

## 第 3 步：配置极速有线连接 (强烈推荐)
这是保证音游体验的关键步骤！请勿使用 Wi-Fi！
1.  电脑下载并安装 [ADB 工具包 (Platform Tools)](https://developer.android.com/studio/releases/platform-tools)。
2.  解压后，将该目录添加到 Windows 的系统环境变量 `Path` 中。
3.  用原装 USB 数据线将你的安卓设备连接到电脑。
4.  在安卓设备的“设置 -> 开发者选项”中，开启 **“USB 调试”**。
5.  打开电脑自带的 `cmd` 或 `PowerShell`，输入：
    ```cmd
    adb devices
    ```
    （此时手机可能会弹出授权提示，请点击允许。终端应显示一串字符后跟着 `device`）。
6.  **开启端口转发**。输入以下命令：
    ```cmd
    adb reverse tcp:4321 tcp:4321
    ```
    没有任何报错输出则表示成功。

## 第 4 步：连接手机并开始游戏
1.  在手机上安装 [TouchDX 安卓客户端](https://github.com/YubaiNya/TouchDX-AndroidClient/releases)。
2.  打开 App，在 IP 地址栏输入：`127.0.0.1` （因为经过了 adb 转发，本机端口即是电脑端口）。
3.  此时先在电脑上启动游戏，等待游戏加载完毕。
4.  点击手机上的“连接”按钮。
5.  大功告成！尽情体验丝滑的打歌面板吧！