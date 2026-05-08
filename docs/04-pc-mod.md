# 04. PC 端 Mod 注入与按键模拟 (PC Mod)

PC 端的 `TouchDX-Mod` 是连接虚拟与现实的桥梁。它本质上是一个 **MelonLoader** 插件。

## MelonLoader 生命周期

Mod 继承自 `MelonMod`，并主要利用了以下生命周期方法：
*   `OnInitializeMelon()`：游戏启动时执行。这里我们初始化了 TCP Server (`4321` 端口) 以接收安卓端的连接，并初始化了 `Harmony` 框架以挂载钩子。
*   `OnUpdate()`：跟随游戏主循环每帧执行。这里用于处理从网络线程拿到的按键状态，并模拟物理键盘的动作。
*   `OnApplicationQuit()`：游戏退出时清理资源，关闭 Socket。

## 实体按键的模拟 (键盘映射)

游戏不仅需要屏幕触摸，还需要外围的八个实体按键（1P）以及测试、服务等控制键。

在 `OnUpdate()` 中，我们通过对比上一帧和当前帧的 64 位掩码状态，提取出物理按键对应的 Bit 位（如 Bit 0-7）。
一旦检测到变化（0->1 或 1->0），就会调用 Windows 底层的 `user32.dll` API `keybd_event` 来生成系统级的硬件键盘中断。

映射关系如下：
*   **外圈 1-8** -> `W, E, D, C, X, Z, A, Q`
*   **Select (Bit 42)** -> `3`
*   **Test (Bit 43)** -> `F2`
*   **Service (Bit 44)** -> `F3`
*   **Coin (Bit 45)** -> `5`

> **注意**：在游戏处于未完全进入打歌界面的状态时，安卓端 A 区的触摸也会被自动映射为外圈实体按键的触发，以方便玩家进行菜单选择。

## 屏幕触摸的内存注入 (Harmony Patch)

这是最核心的技术。普通的鼠标模拟无法实现多点触控，因此必须在游戏代码层面做手脚。

游戏内部使用 `InputManager` 和 `MeshButton` 类来处理触摸屏判定。我们使用 `Harmony` 的 `Postfix` (后置补丁) 功能，拦截了 `MeshButton.Update()` 方法。

```csharp
[HarmonyPatch(typeof(MeshButton), "Update", new System.Type[0])]
[HarmonyPostfix]
public static void MeshButtonUpdatePostfix(MeshButton __instance, ref InputManager.TouchPanelArea ___touchArea, ...)
```

在原版方法执行完毕后，我们的补丁会获取到当前处理的触摸区块 ID (`___touchArea`)，然后计算它在我们 64 位掩码中对应的 Bit 位。如果该 Bit 为 1，我们就强行调用游戏的 `___onTouchEvent.Invoke()` 和 `___onTouchDownEvent.Invoke()` 委托。

通过这种“降维打击”，游戏会完全相信这是一次来自合法硬件设备的触控。