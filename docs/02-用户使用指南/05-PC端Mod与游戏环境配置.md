# 05. PC端Mod与游戏环境配置

TouchDX-Mod 是让游戏理解外部触摸信号的核心桥梁。它作为一个 MelonLoader 插件运行。

## 1. 安装前置框架：MelonLoader

如果你之前没有为游戏安装过 Mod，你需要先安装 MelonLoader。

1.  下载 [MelonLoader (推荐版本 v0.5.7 或以上)](https://github.com/LavaGang/MelonLoader/releases)。
2.  运行 MelonLoader 安装程序。
3.  点击 **"SELECT"**，找到你的游戏主程序 (例如 `Sinmai.exe`)。
4.  取消勾选 "Latest"（如果需要指定特定版本），然后点击 **"INSTALL"**。
5.  安装成功后，第一次运行游戏时会比较慢（它在生成缓存），当看到带有 MelonLoader 字符的黑框启动并成功进入游戏后，关闭游戏。
6.  此时你的游戏根目录下会多出一个 `Mods` 文件夹。

## 2. 安装 TouchDX-Mod

1.  前往 [YubaiNya/TouchDX-Mod Releases](https://github.com/YubaiNya/TouchDX-Mod/releases) 下载最新的压缩包。
2.  解压缩后得到 `MaiRemoteTouchMod.dll`。
3.  将此 DLL 文件放入游戏根目录的 `Mods` 文件夹内。

## 3. 验证是否生效

1.  启动游戏。
2.  观察伴随游戏启动的那个黑色控制台窗口。
3.  在杂乱的日志中，寻找是否有绿色的文字写着：
    `[MaiRemoteTouchMod] Starting MaiRemoteTouch TCP Server on port 4321...`
    `[MaiRemoteTouchMod] Harmony Patches applied!`
4.  如果看到了这两行，说明 Mod 已经成功注入，正在乖乖等待你的安卓设备连接。