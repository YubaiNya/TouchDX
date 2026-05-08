# 14. 心跳包同步与诊断 Ping 测试实现

TouchDX 并非只有从手机到 PC 的单向数据流。为了实现“智能状态切换”和“网络质量诊断”，PC 端也会向手机端发送指令。

## 游戏状态心跳包 (Heartbeat)

正如《智能状态切换》一章所述，手机端需要知道当前是否处于打歌状态。

PC 端的 `MaiRemoteTouchMod.cs` 在没有按键状态需要更新时，会作为心跳包定期向手机发送 8 字节的数据：
```csharp
byte[] packet = new byte[8] { 0x47, 0x41, 0x4D, 0x45, 0, 0, 0, (byte)(isInGame ? 1 : 0) };
stream.Write(packet, 0, 8);
```
前 4 个字节是 ASCII 码的 `G`, `A`, `M`, `E` 作为数据包的魔数头。最后一个字节的 `1` 或 `0` 代表了当前的 `isInGame` 状态。安卓端依靠这个心跳包来实时调整自己的 UI 和触发策略。

## 诊断握手 (DIAG)

当安卓端连接到并非游戏的 `TouchDX-DiagnosticTool` 时，为了让 UI 变成黄色的测试模式并解锁功能，诊断工具在建立连接后第一时间会发送一段握手指令：

```csharp
byte[] handshake = System.Text.Encoding.ASCII.GetBytes("DIAG");
stream.Write(handshake, 0, handshake.Length);
```
安卓端一旦读取到包头是 `D`, `I`, `A`, `G`，就会将自身的 `isDiagnostic` 标志位置为 true。

## 极限 Ping 测试的奥秘

在测试模式下，当你点击“测网络延迟”按钮时，安卓端会记录下当前的精确时间，并向 PC 发送一个极其特殊的 64 位整型数字：`-1229782938247303442L`（其十六进制表示刚好是 `0xEEEEEEEEEEEEEEEE`）。

PC 端（诊断工具）的逻辑非常简单粗暴：如果收到的数据是 `0xEEEEEEEEEEEEEEEE`，它什么也不做，直接原封不动地把它 `Write` 回去。

安卓端在读取循环中一旦收到了这个魔法数字，就会用当前时间减去刚才记录的时间：
```java
long l2 = System.currentTimeMillis() - this.lastPingTime;
```
这就得到了一个精确的往返时间 (RTT, Round-Trip Time)，也就是我们常说的 Ping 延迟。这套机制能够精准地揪出那些声称自己“网络很好”却在用垃圾路由器 Wi-Fi 玩音游的幻觉。