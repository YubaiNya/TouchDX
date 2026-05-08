# 08. 原生 View 与多点触控生命周期

在 Android 系统中，处理游戏级别的超高频多点触控，如果使用标准的 UI 按钮组件（Button）会导致灾难性的延迟和事件冲突。因此，TouchDX 的客户端采用继承原生的 `View` 类（即 `TouchView.java`）来接管最底层的分发。

## MotionEvent 的机制
Android 将每一次屏幕接触都封装在 `MotionEvent` 对象中。该对象不仅包含坐标 `(x, y)`，还包含动作类型 `Action` 以及触摸点 ID `PointerId`。

为了支持十指齐飞的极限操作，我们重点处理了以下四个 Action：
*   `ACTION_DOWN`：屏幕上第一根手指落下。
*   `ACTION_POINTER_DOWN`：在已有手指按下的情况下，后续的额外手指落下。
*   `ACTION_UP`：最后一根手指抬起。
*   `ACTION_POINTER_UP`：松开多根手指中的某几根。
*   `ACTION_MOVE`：任意一根或多根手指在屏幕上移动。

## 事件分发与收集算法

每一帧接收到 `onTouchEvent` 回调时，客户端并没有逐个处理单一事件，而是进行一次“快照收集”：

1.  **清空状态缓冲**：每次事件触发，先将所有 `Pointer` 的活跃状态数组重置。
2.  **遍历所有活跃 Pointer**：利用 `motionEvent.getPointerCount()` 循环。如果当前该 Pointer 不是刚好在这一帧被抬起（UP / POINTER_UP），我们就读取它的 `getX(i)` 和 `getY(i)`。
3.  **坐标转换与缩放修正**：手机屏幕的分辨率千差万别，且玩家可能对其进行了双指缩放（Scale）或平移（Offset）。在拿到绝对物理坐标后，需要逆向应用矩阵变换：
    ```java
    int actualX = (int) ((x - offsetXScreen) / currentTotalScale);
    int actualY = (int) ((y - offsetYScreen) / currentTotalScale);
    ```
4.  **碰撞测试分发**：将计算出的相对坐标传入碰撞测试引擎，匹配出哪些 SVG 区域被点亮。
5.  **增量更新比对**：将当前帧计算出的区域激活状态，与上一帧的状态数组进行按位比对。只有当发生真正的电平翻转（即某个区块从 0 变 1，或 1 变 0）时，才会标记需要向网络层提交更新。

这一套流程被优化在了一个极短的执行路径中，以确保能跑满大部分高端平板 120Hz 甚至更高采样率的报点极限。