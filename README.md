# Eink Reframe — 即拍即显电子墨水相机



基于 **Seeed XIAO ESP32S3 Sense**（含 OV2640 摄像头）与 **2.9" C3_GDEM029C90 三色电子墨水屏**的便携式即拍即显相机。物理横置安装（128高×296长），按下按钮拍照，经 Floyd-Steinberg 三色误差扩散抖动后立即显示在墨水屏上，照片上叠加一句经典电视剧台词（逆时针90°旋转，沿长边显示）。拍完断电，零功耗待机。


## 设备预览

<div align="center">
  <table>
    <tr>
      <td align="center"><img src="https://raw.githubusercontent.com/jesse2023wang-collab/images/main/eink-reframe-04-display-content.jpg" width="400" /><br><sub>显示效果 — 经典台词叠加在照片上</sub></td>
      <td align="center"><img src="https://raw.githubusercontent.com/jesse2023wang-collab/images/main/eink-reframe-01-capturing.jpg" width="400" /><br><sub>拍摄中 — 屏幕显示「拍摄中 请稍后」</sub></td>
    </tr>
    <tr>
      <td align="center"><img src="https://raw.githubusercontent.com/jesse2023wang-collab/images/main/eink-reframe-02-display-front.jpg" width="400" /><br><sub>背面 — 墨水屏</sub></td>
      <td align="center"><img src="https://raw.githubusercontent.com/jesse2023wang-collab/images/main/eink-reframe-03-display-angle.jpg" width="400" /><br><sub>侧面 — 设备外观与接口</sub></td>
    </tr>
  </table>
</div>


## 硬件

| 组件 | 型号 |
|------|------|
| 主控板 | Seeed XIAO ESP32S3 Sense（8MB PSRAM, 8MB Flash） |
| 显示屏 | 2.9" C3_GDEM029C90 三色墨水屏 (128×296, 黑/白/红) |
| 摄像头 | OV2640（XIAO Sense 板载, DVP 接口） |
| 按钮 | 板载 BOOT 键 (GPIO0) + 外接拍照键 (GPIO6) |
| 状态 LED | 板载 LED (GPIO5, 高电平有效) |
| 电源 | 3.7V 锂电池 + 物理开关断电 |

## 接线

| 墨水屏 | XIAO 引脚 | GPIO | 说明 |
|--------|-----------|------|------|
| VCC | 3V3 | — | 供电 |
| GND | GND | — | 接地 |
| CLK | D8 | 7 | SPI SCK (硬件 SPI) |
| DIN | D10 | 9 | SPI MOSI (硬件 SPI) |
| CS | D1 | 2 | 片选 |
| DC | D2 | 3 | 数据/命令 |
| RST | D3 | 4 | 复位 |
| BUSY | D0 | 1 | 忙检测 |

| 外设 | XIAO 引脚 | GPIO | 说明 |
|------|-----------|------|------|
| 状态 LED | D4 | 5 | 常亮=就绪, 快闪=处理中, 灭=刷新中 |
| 拍照按键 | D5 | 6 | 外接快门键 (INPUT_PULLUP, 低电平触发) |
| BOOT 键 | BOOT | 0 | 板载按键 (INPUT_PULLUP, 低电平触发) |

> 摄像头使用 XiaoSense 板载引脚（GPIO10~18, 38~40, 47~48），与墨水屏引脚无冲突。

## 功能

1. **即拍即显** — 按下 BOOT 键或外接拍照键 → OV2640 采集 JPEG → 解码 RGB888 → 双线性插值缩放 → 三色 Floyd-Steinberg 误差扩散抖动 → 墨水屏显示
2. **中文台词叠加** — 每次拍照随机选取一句经典电视剧台词，使用 U8g2 12px 文泉驿中文字体叠加在照片上，`setFontDirection(3)` 逆时针90°旋转沿长边显示，`setFontMode(1)` 白色衬底确保可读性
3. **横屏显示** — 墨水屏物理横置安装，`setRotation(2)` 旋转坐标系，所有文本使用 U8g2 叠加层渲染以实现横屏文字方向
4. **开机画面（中文）** — `showReadyScreen()` 显示准备就绪提示、照片计数、存储用量和操作说明，全部中文文字
5. **拍摄提示（中文）** — `showMessage()` 在拍照时显示
6. **照片存储** — 拍照后 JPEG 文件保存至 LittleFS `/photos/` 目录，文件名按序号递增（`00001.jpg` 起），保留最近 50 张照片
7. **照片计数** — 开机画面显示已拍摄照片总数和存储使用量，存储空间不足时自动删除旧照片
8. **零功耗待机** — 拍摄完成后墨水屏静态显示，不耗电；物理电源开关断电
9. **WiFi 热点管理** — 长按拍照键 3 秒立即开启 WiFi AP（`Eink-Reframe`，无密码），支持 Captive Portal（手机连接后自动弹出浏览器页面），通过浏览器访问 `192.168.4.1` 浏览/下载/删除照片，支持一键清空所有照片，照片按从新到旧排序显示

### 依赖库

- `lib-gxepd2` — 墨水屏驱动 (GxEPD2_290_T94)，内置 U8g2 中文字体支持（`u8g2_font_wqy12_t_gb2312a`）
- `lib-esp32cam` — OV2640 摄像头驱动
- `lib-esp32-littlefs` — 文件系统
- `lib-esp32-preferences` — 持久化存储
- `lib-esp32-wifi` — WiFi 热点功能（`#include <WiFi.h>`）
- `lib-esp32-webserver` — Web 服务器（`#include <WebServer.h>`，照片浏览/下载/删除）
- `lib-core-*` — 核心（serial/io/logic/loop/math/text/time/variables/custom）

---

## 版本日志

### v1.0.0 — 即拍即显相机 + 中文台词 + WiFi 热点 (2026-08-15)

#### 功能更新
- **项目灾后重建**：`project.abi` 文件损坏（丢失 `$ailyProjectData` schema 标记），通过 ABS 文件从零重建全部 215 个积木块，项目完全恢复
- **引脚配置对齐 dot 项目**：CS=D9(8), DC=D0(1), RST=D1(2), BUSY=D3(4)，与 dot_V1.0.1 保持一致
- **移除 WiFi/Web/OTA**：删除 WiFi 连接、WebServer、ConfigPortal、HTTPUpdateServer 等全部网络功能
- **移除电源管理**：删除深度睡眠、电池 ADC 检测、低电量告警，改用物理电源开关断电
- **简化按键逻辑**：从三功能（短拍/长睡/超长配网）简化为仅短按拍照
- **存储优化**：新增 MAX_PHOTOS=50 上限，超出自动删除最旧照片
- **新增启动画面**：`showReadyScreen()` 显示照片数、存储使用量和操作提示
- **清理依赖**：移除 6 个不再使用的库（arduinojson/httpupdateserver/sleep/webserver/wifi/sscma-micro-core）
- **JPEG 存储**：摄像头改为 JPEG 输出模式，照片直接保存为 JPEG（~10-15KB/张），比原 bitmap（37KB/张）节省 ~60% 空间
- **双线性插值缩放**：升级图像缩放算法为双线性插值，利用 PSRAM 解码全帧 RGB888 后做 4 邻域加权采样，画质显著提升
- **双键拍照**：新增板载 BOOT 键 (GPIO0) 作为第二拍照键，`checkButton()` 同时检测外接 D5 键和 BOOT 键，任意一个按下即可触发拍照
- **引脚重新分配**：CS=D1(2), DC=D2(3), RST=D3(4), BUSY=D0(1)，与硬件实际接线对齐
- **新增 .eink 文件存储**：`saveCurrentPhoto()` 在保存 JPEG 的同时，将墨水屏缓冲区（128×296）另存为 `.eink` 文件，保留原始照片和墨水屏显示副本
- **切换为黑白墨水屏**：面板从 `C3_GDEM029C90`（三色）改为 `BW_GDEM029T94`（黑白），Floyd-Steinberg 抖动从三色（黑/白/红）简化为双色（黑/白），移除红色通道和 `isReddish` 检测逻辑，`showReadyScreen()` 中的红色文字改为黑色
- **新增中文台词叠加**：拍照后随机从 20 条经典电视剧台词中选取一句，使用 `lib-gxepd2` 内置的 U8g2 12px 文泉驿中文字体（`u8g2_font_wqy12_t_gb2312a`）叠加在照片顶部，逐字白色衬底确保暗处可读性，不遮挡照片其他区域
- **随机种子初始化**：`setup()` 中调用 `randomSeed(esp_random())`，确保每次开机随机序列不同
- **恢复为三色屏**：面板从 `BW_GDEM029T94`（黑白）恢复为 `C3_GDEM029C90`（三色），恢复 Floyd-Steinberg 三色误差扩散抖动（黑/白/红），恢复 `isReddish` 红色检测逻辑和红色通道，`showReadyScreen()` 中操作提示恢复为红色文字
- **横屏显示改造**：墨水屏物理横置安装（128高×296长），`display.setRotation(2)` 旋转坐标系，`drawRect` 更新为 `(2, 2, 124, 292)` 适配横屏边框
- **U8g2 文本叠加层**：所有文本渲染统一使用 `u8g2Fonts` 叠加层（`gxepd2_u8g2_begin` 初始化），`setFontDirection(3)` 实现逆时针90°旋转沿长边显示，配合 `setFontMode(1)` 白色衬底确保可读性
- **开机画面中文化**：`showReadyScreen()` 全部改为中文文字，显示「准备就绪」「照片数」「存储使用量」「操作说明」等信息

#### WiFi 热点功能（2026-08-15 新增，持续更新）

- **长按 3 秒立即开启 WiFi 热点**：`checkButtonLongPress()` 返回 int（0=无按键/1=短按/2=长按），长按检测到 ≥3 秒后立即返回 2，不等用户松开按键，触发 `startHotspot()` 开启 WiFi AP（SSID: `Eink-Reframe`，无密码），墨水屏显示 SSID、IP 地址和操作说明
- **Captive Portal（强制门户）**：集成 `DNSServer` 库，DNS 拦截所有域名查询并重定向到 ESP32，手机连接 WiFi 后自动弹出浏览器页面，无需手动输入 IP
- **内嵌 HTML 网页浏览**：WebServer 注册 `/`（HTML 相册）、`/list`（JSON 列表）、`/photo/`（下载原图）、`/delete/`（删除）、`/clear-all`（清空所有照片）路由，手机连接热点后通过浏览器管理所有照片
- **照片排序从新到旧**：网页端照片列表按文件名降序排列（最新照片排最前），方便查看最近拍摄的照片
- **网页图片旋转修正**：CSS 添加 `transform:rotate(180deg)` 修复网页预览图上下颠倒问题
- **一键清空所有照片**：网页底部新增「🗑️ 清空所有照片」按钮，点击确认后删除全部照片并重置计数器
- **热点快捷关闭**：热点开启时，任意按键（短按或长按）触发 `stopHotspot()` 关闭 WiFi 和 WebServer，自动调用 `showLastPhoto()` 显示最近一张照片（含台词文字叠加层）
- **启动页长按进入热点**：在开机启动画面（`showReadyScreen()`）显示时，长按拍照键同样可进入热点模式
- **新增依赖**：添加 `lib-esp32-wifi`、`lib-esp32-webserver` 和 `DNSServer` 库，通过 `custom_library` 块生成 `#include <WiFi.h>`、`#include <WebServer.h>` 和 `#include <DNSServer.h>`
- **全局变量**：`WebServer server(80)`、`DNSServer dnsServer` 和 `bool hotspotActive = false` 通过 `custom_insert_code` 块声明
- **构建验证**：编译成功，代码无语法或链接错误

#### 崩溃修复与稳定性优化

- **WiFi TX 功率降低**：`WiFi.setTxPower(WIFI_POWER_8_5dBm)` 将发射功率从默认 19.5dBm 降至 8.5dBm，减少摄像头 + 墨水屏 + WiFi 同时工作时的电流尖峰，防止电压跌落导致 ESP32 重启
- **栈缓冲区溢出修复**：`String names[50]` 扩容为 `names[52]`，并增加 `total < 52` 边界检查，修复 `/photos` 目录中 50 张编号照片 + `current.jpg` 共 51 个 `.jpg` 文件时数组越界问题
- **路由注册防重复**：`static bool handlersRegistered` 守卫变量防止 `server.on()` 在多次 `startHotspot()`/`stopHotspot()` 循环中重复注册导致崩溃

#### 照片回看功能

- **新增 `showLastPhoto()` 函数**：退出热点时从 `/photos/current.jpg` 读取最近一张照片，执行与 `captureAndDisplay()` 相同的 JPEG 解码 → Floyd-Steinberg 三色抖动 → 墨水屏显示流程，并叠加随机台词文字，确保退出热点后看到完整照片

#### 经验教训

1. **`"320,240"` 引号包裹**：`esp32cam_init` 的 RESOLUTION 参数需要用引号包裹（如 `"320,240"`），不能用 `320,240`，否则会被解析为两个独立参数导致导入失败。
2. **墨水屏刷新时间**：三色屏全屏刷新需要 15~27 秒（SSD1680 硬件限制），应在刷新前显示"处理中"提示。
3. **SPI 引脚重定义**：墨水屏与摄像头共用 SPI 总线时，需在初始化前调用 `SPI.end(); SPI.begin(7, -1, 9, -1);` 重新配置引脚，MISO 设为 -1（墨水屏不需要读取）。
4. **JPEG → RGB888 解码**：`fmt2rgb888()` 来自 `img_converters.h`（esp32-camera 库内置），可直接将 JPEG 帧解码为 RGB888 用于图像处理，避免使用 RGB565 模式占用双倍内存。
5. **双线性插值 vs 最近邻**：最近邻缩放在低分辨率（128×296）上锯齿明显，双线性插值虽需额外 PSRAM（camW×camH×3 bytes）但画质提升显著，ESP32S3 8MB PSRAM 完全够用。
6. **ABI 文件损坏与恢复**：`project.abi` 因未知原因丢失 `$ailyProjectData` 外部 schema 标记，导致项目加载失败。通过 ABS 导出/导入工作流从零重建了全部积木块（6 个全局变量、1 个库引用、5 个自定义函数、setup/loop），验证了 ABS 文件作为项目备份恢复手段的可靠性。建议定期 `sync_abs_file(export)` 导出 ABS 作为快照备份。
7. **ABS 重建大型项目**：含大量 `custom_code` 块的项目通过 ABS 重建时，需注意 `custom_code` 的 CODE 字段中嵌入的引号、反斜杠等特殊字符需正确转义；`custom_function` 的 `@BODY:` 语句体内可串联任意数量的 `custom_code` 块，但缩进必须一致（4 空格）。
8. **`BOOT_PIN` 宏冲突**：ESP32 Arduino 核心库 `esp32-hal.h` 已预定义 `BOOT_PIN` 宏（`static const uint8_t BOOT_PIN = 0`），自定义同名 `const int BOOT_PIN = 0` 会导致 `conflicting declaration` 编译错误。解决方案：改用 `BOOT_BTN` 避开核心库保留名称。
9. **`abi_set_field` 逗号拆分**：`abi_set_field` 对包含逗号的字符串值会按逗号拆分为数组，导致代码片段损坏（如 `pinMode(LED_PIN, OUTPUT)` 被拆为 `LED_PIN` 和 `OUTPUT)`）。解决方案：传单元素数组 `["完整代码"]` 或使用无逗号写法。
10. **长按检测的延迟陷阱**：`checkButtonLongPress()` 在等待按键释放时使用 `while` 循环 + `delay(10)`，若等待时间过长（超过 3.5 秒上限）且用户仍按住不放，函数会返回 2（长按）。但 `delay(10)` 在循环中累计可能产生微小偏差，建议长按检测使用 `millis()` 差值判断而非计数器。
11. **WiFi 库选择**：ESP32 的 WiFi 和 WebServer 功能在 ESP32 Arduino SDK 中作为内置库提供（`#include <WiFi.h>` 和 `#include <WebServer.h>`），无需额外安装第三方库。Aily Blockly 的 `lib-esp32-wifi` 和 `lib-esp32-webserver` 是对 SDK 内置库的 block 封装，安装后即可在 Blockly 中使用，但生成的 C++ 代码直接使用原生 SDK API。
12. **PROGMEM 原始字符串字面量**：嵌入式 HTML 页面使用 `R"rawliteral(...)rawliteral"` 原始字符串字面量存储，避免手动转义 HTML 中的引号和特殊字符。注意选择 `rawliteral` 作为分隔符时，HTML 内容中不能出现 `)rawliteral"` 序列，否则会提前结束字符串。
13. **延时与按键响应平衡**：`loop()` 末尾的 `delay(50)` 用于控制循环频率，但热点模式下 `server.handleClient()` 需要频繁调用以保持 Web 响应。50ms 延时在低负载下足够，若网页响应慢可减少至 10ms 或移除，但会增加 CPU 占用。
14. **定期备份 ABI 文件**：通过直接编辑 `project.abi` 添加大量 blocks 时，每一步操作前建议备份原始文件。`project.abi` 是 JSON 格式，深嵌套的对象链（如 `inputs.BODY.block.next.block...`）容易在手动编辑时出错，使用脚本化操作（Node.js 修改 JSON）比手动编辑更可靠。
15. **U8g2 中文字体内置在 lib-gxepd2 中**：`lib-gxepd2` 库已内置 U8g2 字体子系统，通过 `gxepd2_u8g2_begin($display)` 初始化后即可使用 `u8g2Fonts` 对象渲染中文。字体 `u8g2_font_wqy12_t_gb2312a` 为 12px 文泉驿点阵字库，支持 GB2312 编码的简体中文汉字，无需额外安装任何库。
16. **U8g2 字体叠加模式**：`u8g2Fonts.setFontMode(0)` 为不透明模式，每个字符自带白色背景衬底，仅覆盖字符本身像素区域，不影响照片其他区域。`setFontMode(1)` 为透明模式，字符直接叠加在照片上，暗色区域文字不可读，因此字幕类应用应使用 mode 0。
17. **Node.js 批量修改 project.abi**：当需要修改大量嵌套的 `custom_code` 积木块时，使用 Node.js 脚本递归遍历 `project.abi` 的 JSON 树比逐块调用 `abi_set_field` 更可靠。注意特殊字符（如 `$`、`` ` ``、`%`）在 JavaScript 字符串和 shell 命令中的双重转义问题。
18. **Captive Portal 实现**：ESP32 热点模式下的 Captive Portal 通过 `DNSServer` 库实现 — `dnsServer.start(53, "*", WiFi.softAPIP())` 拦截所有 DNS 查询，`server.onNotFound()` 返回 302 重定向到 ESP32 主页，手机 OS 的连通性检测请求被重定向后自动弹出浏览器页面。
19. **WiFi TX 功率与电源稳定性**：ESP32 在摄像头 + 墨水屏 + WiFi 同时工作时电流峰值可达 500mA+，若电池或 USB 供电能力不足，`WiFi.setTxPower(WIFI_POWER_8_5dBm)` 将发射功率从 19.5dBm 降至 8.5dBm 可显著降低瞬间电流尖峰（约降 150mA），有效防止电压跌落引起的重启崩溃。
20. **栈数组边界检查**：LittleFS 的 `dir.openNextFile()` 返回的文件名包含完整路径前缀，排序后计数时需注意 `current.jpg` 是否被排除。当文件数接近数组上限时，必须同时检查 `total < arraySize` 以防止数组越界，否则栈缓冲区溢出会导致随机崩溃。
21. **`server.on()` 重复注册崩溃**：ESP32 的 `WebServer.on()` 在已注册同路径路由后再次调用会导致崩溃。解决方案：使用 `static bool handlersRegistered` 守卫变量，首次注册后将标记置为 true，后续 `startHotspot()` 直接调用 `server.begin()` 重启服务而不重新注册路由。
22. **`f.name()` 返回完整路径**：LittleFS 的 `File.name()` 返回的是打开时的完整路径（如 `/photos/00001.jpg`），而非仅文件名。删除文件时应直接使用 `LittleFS.remove(fn)` 而非 `LittleFS.remove("/" + fn)`，否则会生成错误路径 `//photos/00001.jpg`。
23. **`project_build` 与 .temp 缓存**：`project_build` 依赖 `.temp/sketch/sketch.ino` 已生成。若手动删除 `.temp` 目录，需先通过 `app_reload` 重新加载项目以重新生成代码文件，否则构建会失败并报 `Missing Blockly generated file`。`project_save` 只写入 `project.abi` 和 `project.abs`，不生成 `.temp` 中的代码。
24. **`abi_set_field` 的 CODE 换行限制**：`custom_code` 块的 CODE 字段在通过 `abi_set_field` 写入时，换行符 `\n` 会被转换为逗号，导致多行代码损坏。单行复杂代码虽然可读性差，但在 `abi_set_field` 工作流中是最可靠的写法。若需要多行，应使用 `custom_function` 的 `@BODY` 语句体串联多个 `custom_code` 块，每个块一行。
25. **CSS `transform:rotate()` 仅影响网页渲染**：在网页 HTML 中通过 CSS 旋转图片（`transform:rotate(180deg)`）仅修改浏览器端的显示效果，不影响原始 JPEG 文件或墨水屏渲染，是修复网页预览方向问题的零成本方案。
26. **零填充文件名排序**：照片编号文件使用零填充命名（`00000.jpg`、`00001.jpg`...），按字典序排序即可正确反映时间顺序（逆序为从新到旧）。零填充确保了排序的稳定性，无需解析数字部分。
