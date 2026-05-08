# 11. 悬浮窗 Overlay 架构与 UI 交互设计

为了让安卓设备能一边显示游戏画面（通过串流），一边响应触摸，TouchDX 的客户端被设计为一个 `Service` 驱动的全局悬浮窗应用。

## 重要概念警告：必须配合串流使用！

**🚨 TouchDX 本身绝对不包含任何游戏画面的捕获或传输功能！**

它仅仅是一个**纯黑的/半透明的触摸遮罩层**。要在平板上看到游戏画面，你**必须**在后台运行额外的串流软件（如 **MoonLight**、Steam Link 等）。

正确的使用姿势：
1. 打开 MoonLight 串流你的电脑桌面，确保你能看到游戏画面。
2. 切出 MoonLight，打开 TouchDX。
3. TouchDX 会在 MoonLight 的画面之上覆盖一层悬浮窗（Overlay）。
4. 你透过 TouchDX 的半透明 UI 看到下方的串流画面，你的手指点击在 TouchDX 上，由它将坐标处理后发给电脑。

## OverlayService 架构

所有的核心逻辑都运行在 `OverlayService` 中。它通过 `WindowManager` 直接向系统请求 `TYPE_APPLICATION_OVERLAY` 权限，将自己绘制在系统最顶层。

它包含两个独立的 Window：
1.  **TouchView Window**：这是全屏的打歌面板。配置了 `FLAG_NOT_TOUCH_MODAL`。
2.  **UI Container Window**：这是包含设置面板和那个半透明 `M` 悬浮球的窗口。

## 触摸穿透技术 (Lock Touch)

在悬浮窗的设置中有一个“锁定触摸 (穿透到桌面)”按钮。
当你点击它时，代码会动态修改 TouchView 的 Window 参数，为其加上 `FLAG_NOT_TOUCHABLE` 标志：

```java
touchParams.flags |= WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE;
windowManager.updateViewLayout(touchView, touchParams);
```

加上这个标志后，Android 系统就不再把触摸事件派发给 TouchDX，而是直接穿透给下方的应用（比如你的微信，或者你的 MoonLight 串流端）。这就允许你在不关闭控制器的前提下，直接操控手机桌面。

再次点击时，移除该标志，重新接管触控。

## 游玩透明度 (Alpha) 的实现

悬浮窗设置中还有一个滑动条用于调节“游玩透明度”。
其原理是通过 `Paint.setAlpha()` 动态修改 SVG 绘制时的不透明度。如果你将 Alpha 调到 0，整个 TouchView 在视觉上会完全消失，呈现出下方的原版游戏画面，但它的判定逻辑依然在默默运行，这就是大佬们追求的“盲打”境界。