# 16. 系统 API 硬件键盘模拟与内存写入逻辑

在处理了触屏判定之后，我们还需要解决外围的物理按键（1-8 键，以及 Test、Select 等控制键）。

这些按键在游戏中不是由 `MeshButton` 处理的，而是直接读取系统的键盘输入状态。因此，内存注入在这里行不通，我们需要回到系统层面，进行底层的键盘模拟。

## P/Invoke 与 keybd_event

在 `MaiRemoteTouchMod.cs` 中，我们声明了一个原生的 Windows API 引用：

```csharp
[System.Runtime.InteropServices.DllImport("user32.dll")]
public static extern void keybd_event(byte bVk, byte bScan, uint dwFlags, int dwExtraInfo);
```

`keybd_event` 是一个非常底层且暴力的 API，它会将一个硬件中断级别的键盘事件直接塞进 Windows 的输入队列中。这比更上层的 `SendKeys` 或单纯的窗口消息发送更难以被屏蔽，且延迟更低。

## 按键状态机的 Diff 算法

我们拥有上一帧的按键掩码 `PreviousState` 和当前帧的掩码 `CurrentState`。在 `OnUpdate` 的每一帧里，我们需要找出发生变化的按键，并触发相应的按下或抬起动作。

```csharp
// 以外圈按键 1 (对应 Bit 0, 键盘 W 键, VK=0x57) 为例
bool cur = (CurrentState & (1UL << 0)) != 0;
bool prev = (PreviousState & (1UL << 0)) != 0;

if (cur != prev) {
    if (cur) {
        keybd_event(0x57, 0, 0, 0); // 发送按下事件
    } else {
        keybd_event(0x57, 0, 0x0002, 0); // 0x0002 代表 KEYEVENTF_KEYUP，发送抬起事件
    }
}
```

通过这种简单的异或/差分比对，我们将高频的网络掩码，转化为精确的、只在边缘跳变时触发的 Windows 硬件键盘事件。

## A 区按键的动态映射逻辑

回到我们之前提到的“智能交互模式”。
在未进入游戏判定界面时（`isInGame == false`），玩家需要操作菜单。此时屏幕触摸的 A 区（最外圈）应当充当物理按键的功能。

代码里有这么一段巧妙的逻辑：
```csharp
if (!isInGame)
{
    // A 区的掩码位于 Bit 8-15
    bool curA = (CurrentState & (1UL << (i + 8))) != 0;
    bool prevA = (PreviousState & (1UL << (i + 8))) != 0;
    
    // 与物理按键的判断做 OR 运算合并
    cur = cur || curA;
    prev = prev || prevA;
}
```

这就使得在菜单状态下，无论是按实体键（虽然安卓上没有），还是触摸屏幕的 A 区，都会产生 `cur = true`，进而引发底层的 `keybd_event`，完美实现了触摸屏对选歌菜单的兼容控制。

---

至此，关于 TouchDX 的全部 16 篇核心技术解析文档结束。希望这些拆解能帮助你更好地理解这个项目，也欢迎提交 Pull Request 共同改进这份奇妙的代码！