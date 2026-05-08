# 09. SVG 高精度解析与异形判定算法

maimai DX 的按键布局并非简单的网格，而是由内到外嵌套的扇形、圆环和切片组合而成。如何在安卓设备上精确判定用户点击了哪个不规则图形？这是 TouchDX 客户端的核心技术之一。

## 为什么不使用图片透明度检测？

最简单的异形按钮方案是：画一张包含所有按键的 PNG 图片，点击时获取对应坐标的像素颜色。但这有几个致命缺陷：
1.  **性能极低**：在 120Hz 下实时读取 Bitmap 像素会引发严重的掉帧。
2.  **缩放失真**：当玩家在平板上对面板进行缩放 (Scale) 时，图片会模糊，导致边界判定不再精确。
3.  **内存占用**：高分辨率的图片会占用大量内存。

## SVG (可缩放矢量图形) 的引入

为了完美解决以上问题，我们选择了 SVG。SVG 使用数学方程（如贝塞尔曲线、圆弧）来描述图形。

项目内部实现了一个 `SvgLoader` 和精简版的 `PathParser`。
它们的作用是读取 `assets/maimai-layout.svg` 文本文件，并将其中的 `<path d="M... C... Z">` 节点转换为安卓底层的 `android.graphics.Path` 对象。

## 异形判定算法：Region 相交测试

有了 `Path` 对象，我们依然无法直接用它做点击判定，因为 `Path` 是数学描述，不具备碰撞属性。

1.  **转换为 Region**：
    在解析出 `Path` 之后，我们利用安卓底层的 `Region` API。
    ```java
    RectF rectF = new RectF();
    path.computeBounds(rectF, true);
    Region clip = new Region((int) rectF.left, (int) rectF.top, (int) rectF.right, (int) rectF.bottom);
    Region region = new Region();
    region.setPath(path, clip); // 将数学路径光栅化为可计算的区域
    ```
    
2.  **点阵膨胀测试 (Touch Radius)**：
    人的手指不是一个数学上的“点”，而是一个“面”。如果玩家手指按在两个区块的交界线上，理论上应该同时触发两个区块。
    为了实现这种“模糊判定”，我们在 `TouchView` 中引入了 `touchRadius` (触摸半径)。
    
    判定算法不仅仅检查触控中心点 `(x, y)` 是否在 `Region` 内。它还会以 `(x, y)` 为圆心，`touchRadius` 为半径，在一个圆周上取 8 个采样点进行碰撞测试。
    只要这 9 个点（中心 + 8个边缘点）中有一个落在了某个 `Region` 内部，我们就认为该区域被按下了。

3.  **纯净的数学缩放**：
    当玩家调整面板大小时，我们只需修改 Canvas 的 `scale` 矩阵，`Path` 和 `Region` 会在数学层面上自动适配，永远保持边缘的锐利和判定的绝对精确。