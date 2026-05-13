# ESP32 Web BLE OTA Tool

基于纯前端 Web Bluetooth API 开发的 **ESP32 极简蓝牙 OTA (空中升级) 工具**。
无需编写繁琐的 Android / iOS App，只需打开网页即可直接对 ESP32 进行蓝牙固件升级！

## ✨ 特性 (Features)
- 🚀 **免安装**：纯 HTML + JS，浏览器打开即用。
- 🛡️ **加密兼容**：完美兼容带有 CTKD 或 LESC 强加密配对的 ESP32 蓝牙设备。
- ⚡ **极速稳定**：采用无响应写入 (`Write Without Response`) 切片传输，防卡死防丢包机制。
- 📊 **可视化**：带有进度条和极客风的实时终端日志打印。
- ⚙️ **高度可定制**：支持在网页端直接修改 Service/Data/CMD UUID，以及调整分包大小和发送延迟，适配不同硬件环境。
- 🛠️ **全平台可用**：支持 Windows / Mac / Linux (Chrome & Edge)，以及 Android (Chrome)。*(注：iOS 因苹果系统限制，需使用第三方浏览器如 Bluefy)*

## 📖 在线体验 (Live Demo)
*(如果使用了 GitHub Pages 部署，把你的页面链接放在这里)*
🔗 **点击直接访问：[https://liuwentao12.github.io/esp32-ble-ota-web/]**

---

## ⚙️ ESP32 端协议对接规范

要使用本网页对你的 ESP32 进行 OTA 升级，你的 ESP32 固件需要按照以下“极简双通道”协议进行 GATT Server 配置：

### 1. 蓝牙通道定义 (UUID)
所有服务和通道采用以下 128-bit UUID（基于经典 0xFFF0 系列扩展）：

| 业务类型 | UUID (128-bit) | 属性 (Properties) | 说明 |
| :--- | :--- | :--- | :--- |
| **主服务 Service** | `e892fff0-e251-40b9-8c20-a10c2fc0ce6b` | - | OTA 主服务通道 |
| **OTA 数据通道** | `e892fff3-e251-40b9-8c20-a10c2fc0ce6b` | `Write / Write_NR` | 用于接收固件切片 |
| **OTA 控制通道** | `e892fff4-e251-40b9-8c20-a10c2fc0ce6b` | `Write` | 用于接收 **开始 / 结束** 指令 |

### 2. 交互指令与流程

本网页工具作为 GATT Client，严格遵循以下 3 步执行升级逻辑，ESP32 需配合处理：

1. **发起开始指令：** 
   网页向 **控制通道** (`...fff4`) 写入 `[0x01]`。
   👉 *设备端行为：收到 `0x01` 后，调用 `esp_ota_begin()` 擦除对应 Flash 分区。网页端会强制等待 2 秒钟等待设备擦除完毕。*
   
2. **切片发送固件：** 
   网页将 `.bin` 文件切分为 `240 Bytes` 一包，利用循环不断写入 **数据通道** (`...fff3`)（包之间自带 20ms 延时防丢包）。
   👉 *设备端行为：收到数据后，将收到的字节流通过 `esp_ota_write()` 写入 Flash。建议使用 Ringbuffer 缓冲避免写 Flash 时卡塞蓝牙底层。*
   
3. **发起结束指令：** 
   固件全部发完后，网页向 **控制通道** (`...fff4`) 写入 `[0x02]`。
   👉 *设备端行为：收到 `0x02` 后，调用 `esp_ota_end()` 校验固件完整性，随后调用 `esp_restart()` 重启设备，升级完成！*

---

## 🛠️ 快速原型调试建议 (Prototyping Tips)

本工具专为快速测试设计，如果您在调试过程中遇到问题，可以展开网页上的 **“高级设置”** 面板：

- **自定义 UUID**：如果您的设备使用了不同的 UUID，直接在网页修改即可，无需改动本工具代码。
- **调整分包大小 (Chunk Size)**：默认 240 字节。如果发现传输不稳定，可以尝试减小此值（如 128 或 64）。
- **调整发送间隔 (Delay)**：默认 20ms。如果 ESP32 来不及处理数据导致丢失，可以尝试增大此值（如 50ms）。
- **自动记忆**：所有修改都会自动保存在浏览器的 LocalStorage 中，下次打开无需重填。

---

## 💻 浏览器支持与运行环境

因 Web Bluetooth API 存在严格的安全限制，请注意以下运行条件：

1. **必须是 HTTPS 或 Localhost**：如果本地测试，直接双击打开本地 HTML 文件（`file:///` 协议）在大部分移动端浏览器会被禁用蓝牙功能。推荐使用 VSCode Live Server，或上传到 GitHub Pages / 任何 HTTPS 服务器上运行。
2. **PC 端**：推荐使用 Chrome / Edge 浏览器，确保电脑自带蓝牙且已开启。
3. **Android 端**：推荐使用系统级 Chrome / Edge 浏览器。微信/QQ内置浏览器阉割了该 API，无法使用。
4. **iOS / iPadOS 端**：由于苹果系统的严格限制，Safari/Chrome 原生不支持 WebBLE。请在 App Store 下载专属调试浏览器 **Bluefy** 或 **WebBLE** 打开此网页。

## 🤝 贡献与感谢
如果你在使用中发现 Bug 或者有更好的性能优化方案（比如动态 MTU 协商），欢迎提交 Issue 或 Pull Request！

*License: MIT License*