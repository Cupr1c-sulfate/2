# iPhone 本机 DSH 启动诊断工程

这是实验工程源码，不是已经完成移植的 DSH Go，也不是已经编译成功的 IPA。
目标是用真实 iOS Node.js 运行库和真实 DSH npm 包，定位本机启动的第一个阻碍。
没有仿制聊天回复，没有连接电脑来代替本机运行。

## 已有内容与当前状态

- UIKit 原生诊断界面，Node 在独立线程运行。
- 使用社区 `fogtape/nodejs-mobile` 的 **24.22.1-0 full 预发布版**，下载时验证 SHA-256。
- DSH 固定为 `0.2.1-rc.4`，依赖锁来自 dsh-Go；它不是 dsh-Go 打过全部安卓补丁的引擎。
- 检查 Node 版本、沙箱文件读写、SQLite、回环 HTTP/fetch、真实 DSH 包和原生模块。
- 在 Worker 中先运行真实 `dsh --version`，再观察真实 `dsh web` 45 秒，保留错误。
- 所有尝试监听的 TCP 服务强制绑定回环地址，避免未配置引擎暴露到局域网；Unix socket 在诊断中禁用。这是本实验的附加约束，可能产生额外兼容错误。
- 原生桌面依赖没有被移植：安装时禁用 npm 生命周期脚本、跳过可选依赖，并排除桌面动态库。`koffi`、`node-pty`、`sharp` 等检查可能失败，失败不意味着整个 iOS 移植不可行。
- **未运行 Xcode 编译、iOS 模拟器或 iPhone 实机测试；未验证 DSH 启动成功或模型对话。**
- 本轮本地测试结果见 `VALIDATION.md`。Linux 测试不是手机验证。

## Windows + iPhone：如何得到测试包

1. 在你的 GitHub 中创建一个新仓库。将本压缩包解压后的**内部文件和目录**放到仓库根目录，包含 `.github/workflows/build-ios.yml`。不要只上传 ZIP，也不要再嵌套一层 `ios-local-probe/`。
   在 Windows 可使用 GitHub Desktop 的 Add local repository / Publish repository；资源管理器需显示隐藏项目以确认 `.github` 存在。
2. 向 `main` 推送工程代码会自动构建，也可以打开仓库 **Actions → Build iPhone local probe → Run workflow**。工作流使用 GitHub 提供的 macOS runner，不需要你有 Mac；账户的 Actions 权限、额度及计费规则仍适用。
3. 成功后下载运行页底部的 **DSHLocalProbe-unsigned** artifact，解压得到 `DSHLocalProbe-unsigned.ipa`。
   若失败，请发运行链接或第一处失败日志。不能把工作流文件存在当作编译成功。
4. Windows 从 **https://sideloadly.io/** 下载 Sideloadly；按它的官方安装指南准备 Apple 驱动。
5. 用数据线连接 iPhone 并信任电脑，在 Sideloadly 中选择 IPA，使用你自己的 Apple 账号签名安装。账号、密码和验证码只在你自己的设备上输入。
6. 根据 iOS 提示信任开发者，并在要求时启用开发者模式。免费账号侧载通常需要每 7 天续签，以实际工具和账号提示为准。
7. 打开“DSH 本机验证”，保持前台约一分钟。点击“分享报告”，把 `report.json` 发回用于继续定位。此轮不需要填写模型 API Key。

## 报告怎么判断

- `platform: "ios"` 且 `mobile` 有版本：确认运行的是移动端 Node，不能单独证明 DSH 可用。
- `checks`：每项分别显示 passed/failed；SQLite、文件或 HTTP 通过，仅代表基础能力通过。
- `versionCommand`：真实 CLI 是否能加载到版本命令。
- `engineAttempt`：真实 web 引擎退出、异常或观察窗口结束的情况。
- `events`：最近的 stdout/stderr 和监听事件。监听成功也不等于完整引擎或对话成功。
- `fullDSHVerified` 和 `modelConversationTested` 均保持 false；后续需要实际模型对话、工具调用和前后台切换验证才能改变结论。
- 如果 App 启动即退、没有报告，需要设备崩溃日志，而不是重新安装并假定成功。

报告会尽力遮盖常见 token/password 字段，但不是秘密泄漏防护的完整保证。第一轮不要配置任何真实凭据；已有日志若包含私人信息，分享前自行检查。

## 明确尚未实现的部分

没有 Flutter 聊天界面移植、安卓 PRoot/Ubuntu、可用本机 Bash、跨应用手机操控、系统悬浮桌宠或后台常驻。
运行库是社区预发布版本，其兼容性需实测；本工程不能代表原作者或 DeepSeek 的官方 iOS 产品。

## 固定来源

- dsh-Go: https://github.com/ccch1mneyyy/dsh-Go
- 依赖锁: https://github.com/ccch1mneyyy/dsh-Go/blob/main/runtime/locks/dsh-0.2.1-rc.4/package-lock.json
- DSH: https://github.com/deepseek-ai/deepseek-harness
- iOS runtime: https://github.com/fogtape/nodejs-mobile/releases/tag/v24.22.1-0
- runtime SHA-256: `3fa1b8fb0720768b01c2e0dae3a17e788476e8c1613a1d9a54e206793b45326e`
- 嵌入文档: https://github.com/fogtape/nodejs-mobile/blob/recipe/docs/EMBEDDING.md
- Windows 侧载: https://sideloadly.io/faq

测试代码为本次新编写，沿用 AGPL-3.0 许可交付。复制的依赖锁来源于 ccch1mneyyy/dsh-Go，保留原作者来源与许可；各 npm 包及 Node.js 保留其自身许可证。未复制项目品牌图标、Deepy 或鲸娘等素材。
