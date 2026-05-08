<div align="center">

<h1>🌟 TouchDX 🌟</h1>

<p>
  <b>基于 Android 与 PC 桥接的 maimai DX 原生级触控外设方案</b>
</p>

<p>
  <a href="https://github.com/YubaiNya/TouchDX/blob/main/LICENSE">
    <img src="https://img.shields.io/badge/License-AGPL%203.0-blue.svg" alt="License">
  </a>
  <a href="https://github.com/YubaiNya/TouchDX-AndroidClient/releases">
    <img src="https://img.shields.io/github/v/release/YubaiNya/TouchDX-AndroidClient?label=Android%20Client&color=brightgreen" alt="Android Client Release">
  </a>
  <a href="https://github.com/YubaiNya/TouchDX-Mod/releases">
    <img src="https://img.shields.io/github/v/release/YubaiNya/TouchDX-Mod?label=PC%20Mod&color=orange" alt="PC Mod Release">
  </a>
</p>

<p>
  彻底告别 Windows 辣鸡手势干扰，让你的平板/手机化身最原汁原味的打歌面板。
</p>

</div>

<br/>

## 📖 项目简介

对于 PC 端的音游玩家而言，尤其是游玩 **maimai DX（舞萌 DX）** 时，Windows 默认的触摸屏机制常常伴随着各种手势操作导致判定丢失或游戏最小化。

**TouchDX** 专为此痛点而生！它将你的 Android 设备（平板或手机）变为一个专门的触控外围设备，捕捉原生的屏幕触摸与外部按键，再通过极低延迟的网络层（ADB 有线连接）将操作状态转发给 PC 端的专有 Mod。从而在 PC 游戏中实现 **1:1 原生级外设体验**，不再受到任何操作系统的手势影响。

<br/>

## ⚠️ 重要说明：画面串流机制

请注意：**TouchDX 本身仅负责“触摸判定与网络发送”，它不会捕获或传输任何游戏画面！** 
安卓客户端的本质是一个悬浮在系统最顶层的半透明/全透明触摸层。

为了在你的平板或手机上看到 PC 端的游戏画面，**你必须配合使用画面串流软件**。我们**强烈推荐使用 [Moonlight](https://moonlight-stream.org/)** 进行串流，以获得最低的画面延迟和最佳的画质体验。

**正确的食用方法**：
1. 启动 Moonlight，将 PC 的游戏画面串流到安卓设备上。
2. 将安卓设备切回桌面，打开 TouchDX 客户端并连接。
3. TouchDX 的悬浮面板会覆盖在 Moonlight 画面之上，你的触摸将由 TouchDX 接管并 0 延迟发送给 PC，而你的眼睛看的是底层的 Moonlight 画面。

<br/>

## 🧩 核心组件架构

TouchDX 是一个包含三大组件的完整生态，每个部分各司其职，无缝协作：

### 📱 [1. TouchDX-AndroidClient (安卓触控端)](https://github.com/YubaiNya/TouchDX-AndroidClient)
- **精准映射**：通过读取内置的 `maimai-layout.svg` 划定 A-E 屏幕触摸区域和 1-8 外围实体按键映射。
- **独立环境**：安卓原生无边框环境，完美截取所有触摸操作，无需担心误触系统手势。
- **高速编码**：将高达 64 位的触控掩码通过二进制流实时发送给 PC。

### 💻 [2. TouchDX-Mod (PC 注入端)](https://github.com/YubaiNya/TouchDX-Mod)
- **深度集成**：基于 **MelonLoader** 与 **Harmony**，在游戏运行时动态挂载并拦截原生 `InputManager` 和 `MeshButton`。
- **数据桥接**：在本地开启高速 TCP 端口 `4321`，直接将安卓端的物理级输入覆盖到游戏逻辑判定中。
- **物理按键支持**：同时模拟 `W/E/D/C/X/Z/A/Q` (1-8键)、`Select (3)`、`Test (F2)`、`Service (F3)`、`Coin (5)` 等控制指令。

### 🛠️ [3. TouchDX-DiagnosticTool (PC 诊断工具)](https://github.com/YubaiNya/TouchDX-DiagnosticTool)
- **可视化调试**：以控制台般简洁的界面，实时接收并输出当前的掩码及触摸状态。
- **连接排查**：在连接至正式游戏前，用于检验网络连通性、设备 IP 解析和 ADB 转发是否正常。

<br/>

## 🚀 极速上手体验

为了获得音游所必须的**极致零延迟**，我们强烈建议抛弃 Wi-Fi 环境，采用 **ADB 有线转发连接**。

### 第一步：准备 PC 环境
1. 确保电脑已安装 [Android Platform Tools (ADB)](https://developer.android.com/studio/releases/platform-tools)。
2. 将 **[TouchDX-Mod](https://github.com/YubaiNya/TouchDX-Mod/releases)** 对应的 `.dll` 文件丢入游戏的 `Package\Mods` 目录下（需要提前安装 MelonLoader）。
3. 使用数据线连接你的安卓设备到电脑，并开启设备的 **USB 调试** 模式。
4. 打开电脑命令行 (CMD / PowerShell)，执行以下核心转发命令：
   ```bash
   adb reverse tcp:4321 tcp:4321
   ```

### 第二步：配置 Android 设备
1. 下载并安装 **[TouchDX 安卓客户端](https://github.com/YubaiNya/TouchDX-AndroidClient/releases)** APK。
2. 打开 App，在连接地址中输入：
   ```text
   127.0.0.1
   ```
   *(得益于之前的 adb reverse 操作，访问本机端口即可直达 PC)*
3. 点击“连接”，提示连接成功即可。

### 第三步：起飞
1. 打开游戏，等待 Mod 自动接管系统判定。
2. 此时，在你的平板/手机上的任何正确区域的拍打与滑动，都将被游戏直接识别。
3. **享受没有延迟、没有误触的丝滑打歌体验！**

<br/>

## 🚨 问题反馈 (Issues)

**非常重要：** 为了方便统一管理和追踪，**请将所有有关本项目及其子项目（安卓端、PC Mod、诊断工具）的 Bug 反馈和功能建议，统一发布到本主仓库的 [Issues 页面](https://github.com/YubaiNya/TouchDX/issues)**。

如果您在各个子仓库下单独提交 Issue，我们可能会因为消息遗漏而无法及时回复与修复，感谢您的配合！

<br/>

##  赞助与支持

开源不易，为爱发电。如果您觉得 TouchDX 让您的打歌体验变得更好，或者这款工具对您有所帮助，欢迎通过**爱发电**请作者喝杯咖啡！

<div align="center">
  <a href="https://ifdian.net/a/YubaiNya" target="_blank">
    <img src="https://img.shields.io/badge/%E7%88%B1%E5%8F%91%E7%94%B5-%E8%B5%9E%E5%8A%A9%E4%BD%9C%E8%80%85-946ce6?style=for-the-badge&logo=buy-me-a-coffee&logoColor=white" alt="赞助作者" height="35">
  </a>
</div>

<br/>

## 🤝 参与贡献

这是一个源于热爱且持续迭代的项目，非常欢迎各路大佬加入：
- 你可以通过修改 `assets/maimai-layout.svg` 提交更好的视觉方案或面板比例。
- 也可以通过 Pull Request 提交性能优化与新功能。

<br/>

## 📜 开源许可声明

本项目及其所有分支组件均遵循 [AGPL-3.0 许可协议](LICENSE) 开源。

> **免责声明**：本项目仅供技术交流与学习，严禁用于任何商业牟利及非法用途。不对因使用本项目导致的任何账号封禁、数据损坏承担责任。
