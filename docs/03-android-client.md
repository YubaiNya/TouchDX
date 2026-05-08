# 03. 安卓客户端实现原理 (Android Client)

TouchDX 的安卓端 (`MaiNative`) 肩负着将物理触摸转化为精准数据的重任。为了实现极致性能，应用规避了繁重的 UI 框架，直接使用原生的 View 体系。

## 触摸区域的定义与 SVG 渲染

maimai DX 的屏幕分为 A, B, C, D, E 五个大区，每个大区又分为多个子区域。为了准确映射这些不规则的几何图形，我们使用了 **SVG (可缩放矢量图形)**。

1.  **路径解析**：项目使用了从 AOSP (Android Open Source Project) 提取的高性能 `PathParser` 类。它能够将 `maimai-layout.svg` 中定义的 `<path>` 数据转换 Android 原生的 `android.graphics.Path` 对象。
2.  **区域判定**：每个按键对应一个 `Path`。当发生触摸事件时，客户端会通过判断触摸点的 `(x, y)` 坐标是否包含在某个 `Path` 的轮廓内，来确定触发了哪个按键。

## 触摸事件处理 (Event Handling)

Android 提供了 `MotionEvent` 来处理触摸。`MaiNative` 能够同时处理多个 `pointer`（触控点）：

*   **ACTION_DOWN / ACTION_POINTER_DOWN**：手指按下时，计算坐标所属的区域，并将对应区域的状态置为 `true`。
*   **ACTION_MOVE**：手指在屏幕上滑动时，由于 maimai 支持滑动触发（如星星），客户端需要实时重新计算所有触控点当前所在的区域，如果跨越了边界，则触发新区域的按下，释放旧区域。
*   **ACTION_UP / ACTION_POINTER_UP**：手指抬起，清除该触控点带来的按键状态。

## 状态压缩与网络发送

为了减少网络带宽占用和传输延迟，所有的按键状态（多达 40+ 个独立的感应区和按键）并没有被打包成冗长的 JSON 或文本，而是被压缩成了一个 **64 位的无符号长整型 (`ulong`)**。

每一个 bit (位) 代表一个按键或区域的状态（0为松开，1为按下）。

例如（伪代码）：
```
Bit 0-7  : 实体按键 1-8
Bit 8-15 : 触摸 A1-A8
Bit 16-23: 触摸 B1-B8
Bit 24-25: 触摸 C1-C2
...
Bit 42   : Select 键
```

一个后台的高优先级网络线程 (`NetworkThread`) 会以极高的频率读取这个 `ulong` 的当前值，一旦发现状态发生变化，就立即通过 TCP Socket 将这 8 个字节的二进制流 `write` 给 PC。