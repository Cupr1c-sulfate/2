# 验证记录

## 本次接入复核（2026-11-04）

- Windows / Node.js v24.12.0：5 项测试通过，0 项失败。
- Windows 受限执行沙箱最初阻止临时文件替换和回环请求；在普通本机执行环境复测全部通过，未修改断言。
- 已核对运行库 GitHub Release 中的 iOS 资源文件与 SHA-256，和脚本固定值一致。
- 工作流增加 main 分支代码推送触发及构建日志保存，仍支持手动运行。
- Xcode 编译结果以仓库 Actions 运行记录为准；这些本地测试不证明 iPhone 实机成功。

## 压缩包附带的前次记录

日期：2026-11-04。环境：Linux，Node.js v24.22.1，Python 3.12.14。

已执行：

- `node --test tests/probe.test.cjs`：5 项通过，0 项失败。
  覆盖敏感字段遮盖、Worker 非零退出记录、超时终止分类、回环绑定限制、依赖缺失时不误报成功。
- `bash -n scripts/prepare.sh`、`bash -n scripts/build.sh`：通过。
- `python3 -m py_compile scripts/native_inventory.py`：通过。
- JSON 依赖锁与项目配置的解析检查：通过。

未执行：

- iOS Node.js XCFramework 下载与二进制校验（由云端 prepare.sh 执行）。
- `npm ci` 安装真实 DSH（当前仅准备了原项目锁文件，未把本机测试冒充引擎测试）。
- Xcode 编译、签名安装、iOS 模拟器、iPhone 实机测试。
- DSH web 引擎成功启动、模型对话、工具调用、后台恢复。

因此当前产物是待编译的诊断源码工程，不是 IPA，也没有手机复现成功的结论。

已修正本地测试发现的一个问题：Worker 超时主动终止时，exit 事件可能先于 terminate Promise 到达；现在先记录超时原因再结束 Worker，避免误报引擎自行退出。
