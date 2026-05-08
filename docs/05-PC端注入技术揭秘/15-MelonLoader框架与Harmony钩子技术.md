# 15. MelonLoader 框架与 Harmony 钩子技术

要让基于 Unity 引擎的 maimai DX 理解我们的网络掩码，我们必须把手伸进游戏运行时的内存里。TouchDX-Mod 选用的手术刀就是 **MelonLoader** 与 **Harmony**。

## MelonLoader 的角色

MelonLoader 是一个强大的通用型 Mod 启动器。
由于 maimai DX 是通过 Il2Cpp 或 Mono 编译的 Unity 游戏，其代码被封装在大量的 `.dll` 中。MelonLoader 能够在游戏启动的最早期（引擎初始化阶段）截获执行流，加载我们编写的 `MaiRemoteTouchMod.dll`。

我们继承了 `MelonMod` 基类，这使得我们可以重写 `OnInitializeMelon`（初始化）和 `OnUpdate`（每帧刷新）方法。

## Harmony 内存钩子 (Hooking)

如果在外部模拟鼠标点击，不仅受限于单点触控，而且坐标换算极其复杂且不稳定。通过阅读游戏反编译出的源码，我们发现游戏对触摸屏的判定最终都会汇聚到 `InputManager.TouchPanelArea` 枚举和 `MeshButton` 组件上。

**Harmony** 是一个可以让我们在运行时替换或干涉 .NET 方法执行的库。

```csharp
[HarmonyPatch(typeof(MeshButton), "Update", new System.Type[0])]
[HarmonyPostfix]
public static void MeshButtonUpdatePostfix(MeshButton __instance, ref InputManager.TouchPanelArea ___touchArea, ref Action<InputManager.TouchPanelArea> ___onTouchEvent, ...)
```

这段代码的含义是：
1. 找到游戏里名为 `MeshButton` 的类，以及它无参数的 `Update` 方法。
2. 注入一个 `Postfix`（后置钩子）。这意味着每次游戏执行完自己的 `Update` 逻辑后，都会立刻顺带执行一遍我们写的 `MeshButtonUpdatePostfix` 代码。
3. 通过 `ref` 参数和 `___` 变量命名规则（这是 Harmony 的黑魔法），我们强行获取了 `MeshButton` 实例的私有字段：`___touchArea`（该按钮代表的区域）和 `___onTouchEvent`（按下时要执行的委托函数）。

接下来，我们只需查表判断这个 `___touchArea` 对应的 Bit 在我们接收到的 64 位掩码中是否为 1。如果是 1，不管玩家有没有真的摸屏幕，我们都通过 `___onTouchEvent.Invoke()` 强行告诉游戏：“这个区域被按下了！”。

这就是所谓的内存级注入。不依赖系统级鼠标事件，绝对的底层、绝对的精准。