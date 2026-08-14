# Nebula Debugger - youzi 修改版

Nebula Debugger 是一个面向 Windows 微信小程序运行时（WMPF）的本地调试辅助工具。它通过 Electron 提供桌面操作界面，通过 Node.js 启动调试服务和 CDP 代理，并使用 Frida 在本机微信 WMPF 进程中注入 hook 脚本，从而让 Chrome DevTools 可以连接到小程序运行时。

## 主要功能

- 一键启动微信小程序调试核心。
- 启动本地 WebSocket 调试服务：ws://localhost:9421。
- 启动 Chrome DevTools Protocol 代理：ws://localhost:62000。
- 自动生成 DevTools 连接地址，可内置打开，也可复制到 Edge/Chrome 打开。
- 通过 Frida hook WMPF 运行时，配合版本偏移配置打开调试能力。
- 支持查看 DevTools 面板、Network 请求、Sources/JS 资源和运行时日志。
- 包含公众号调试辅助页面。

## 当前版本说明

- 软件目录：D:\SoftWare\wx_tools\Nebula Debugger - 4.1.12.55
- 主程序：Nebula Debugger.exe
- Electron 包：resources/app.asar
- 微信 PC 版本：4.1.12.55
- 已适配 WMPF 内核配置：20089、25364
- 本地显示：柚子修改，永久免费

## 快速开始

普通用户不需要克隆仓库，直接从 GitHub Releases 下载完整可运行包即可。

1. 打开项目的 GitHub 页面。
2. 进入右侧或顶部的 Releases 页面。
3. 在最新版本的 Assets 附件中下载完整 zip 包，例如 Nebula-Debugger-4.1.12.55-youzi.zip。
4. 不要点击 Code -> Download ZIP 来代替 Release 附件；那个下载的是仓库源码包，可能不包含大体积运行文件。
5. 解压 zip 到本地目录，例如 D:\SoftWare\wx_tools\Nebula Debugger - 4.1.12.55。
6. 先打开 Windows 版微信，并保持登录状态。
7. 双击运行 Nebula Debugger.exe。
8. 点击界面中的“启动调试”。
9. 等待日志出现类似以下信息：

~~~text
[server] debug server running on ws://localhost:9421
[server] proxy server running on ws://localhost:62000
[frida] script loaded, WMPF version: 25364
[frida] you can now open any miniapps
~~~

10. 手动打开需要调试的微信小程序。
11. 小程序连接成功后，内置 DevTools 会自动打开。如未自动打开，可点击界面中的内置调试、Edge、Chrome 或复制链接按钮。

## 目录结构

~~~text
Nebula Debugger - 4.1.12.55/
?? Nebula Debugger.exe                 # 桌面主程序
?? resources/
?  ?? app.asar                         # Electron 主包，包含界面、调试核心、Frida 脚本
?  ?? app.asar.unpacked/               # 未打包的原生依赖和资源
?  ?? app.asar.bak-local-single        # 本地单机版备份包
?  ?? elevate.exe                      # 辅助提权程序
?? locales/                            # Electron 语言包
?? *.dll / *.pak / *.bin               # Electron/Chromium 运行时文件
?? README.md                           # 本文档
~~~

app.asar 内部的关键模块：

~~~text
desktop/index.html                     # 桌面主界面
desktop/renderer.js                    # 桌面界面逻辑
desktop/main.js                        # Electron 主进程
desktop/official-account.*             # 公众号调试页面
dist/index.js                          # 调试核心，负责服务、代理和 Frida 注入
dist/cli.js                            # CLI 入口
frida/hook.js                          # Frida hook 脚本
frida/config/addresses.*.json          # 不同 WMPF 内核版本的偏移配置
~~~

## 技术实现概览

Nebula Debugger 由三部分组成：

1. **Electron 桌面层**  
   负责展示操作面板、运行日志、DevTools 打开按钮和状态显示。

2. **Node.js 调试服务层**  
   负责启动本地 WebSocket 服务和 CDP 代理，在 DevTools 与小程序运行时之间转发消息，并对部分 CDP 消息进行兼容处理。

3. **Frida 注入层**  
   根据 frida/config/addresses.<version>.json 中的偏移配置，将 frida/hook.js 注入到 WMPF 运行时进程。该层负责调整场景值、CDP 过滤条件和运行时连接行为，使 DevTools 能接收小程序调试数据。

## 本地修改点

- 启用本地单机模式，无需远程登录和授权检查。
- 移除主界面的 VIP/支付展示入口，改为“柚子修改，永久免费”。
- 清理了主界面的支付宝扫码、订单轮询和支付提示文案。
- 保留原有调试功能，包括调试服务、CDP 代理、内置 DevTools 打开和日志输出。
- 保留 WMPF 20089 和 25364 的适配配置。
- 对 Network 清空、Sources/JS 资源、CDP 过滤等问题做了兼容处理。

## 常见问题

### 1. 提示 WMPF 内核版本缺少适配配置

日志示例：

~~~text
[frida] 当前微信小程序内核版本 xxxxx 缺少适配配置。
~~~

说明当前微信 WMPF 内核版本没有对应的 frida/config/addresses.<version>.json。需要根据新内核重新适配偏移。

### 2. DevTools 没有自动打开

- 确认微信已启动并且已打开小程序。
- 确认日志中出现 [miniapp] miniapp client connected。
- 尝试点击“内置调试”或“复制链接”后手动打开。
- 检查 62000 端口是否被其他程序占用。

### 3. Frida 注入失败或小程序闪退

- 尝试重启微信和 Nebula Debugger。
- 确认安全软件没有拦截 Frida 相关行为。
- 避免同时开启多个调试工具实例。
- 如果微信内核升级，需要重新校准版本偏移。

### 4. Network 或 Sources 数据显示不完整

- 先启动调试工具，再打开小程序。
- DevTools 打开后等待小程序连接稳定，再进行刷新或页面切换。
- 如果页面卡死，先停止调试，关闭小程序，再重新启动调试流程。

## 打包和分发

完整可运行版请通过 GitHub Releases 附件分发，不要直接把超过 100MB 的大文件提交到普通 Git 仓库。GitHub 普通仓库会拒绝单个超过 100MB 的文件，而 Release 附件单文件上限为 2GiB。

推荐分发流程：

1. 确认本地 Nebula Debugger - 4.1.12.55 目录可以直接运行。
2. 将整个 Nebula Debugger - 4.1.12.55 目录压缩为 zip。
3. 在 GitHub 中新建 Release，把该 zip 作为 Assets 附件上传。
4. README 和仓库文件用于说明项目和引导下载，完整运行文件以 Release zip 为准。

当前 .gitignore 已忽略普通仓库无法上传或不建议直接提交的打包大文件，包括 Nebula Debugger.exe、resources/app.asar、resources/app.asar.unpacked/ 和压缩包等。

## 使用声明

本工具仅用于授权环境下的小程序调试、CTF 靶场、本地研究和学习交流。请勿用于未经授权的目标或生产环境。
