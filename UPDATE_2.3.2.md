# ZiruPDF 2.3.2 — 扫描件 OCR 提示设置

## 功能

- 扫描 PDF 时顶部 OCR 蓝色提醒新增「不再提示」按钮，点击后关闭今后的自动提醒。
- 帮助 → 设置 → 文档：增加「打开扫描 PDF 时显示 OCR 提示」开关，允许重新开启。
- 原有 `×` 按钮保持只关闭当前文档的提醒。
- 使用原有本机设置 `freedoc.settings.v1` 的 `showOcrHints` 保存偏好，重启有效，默认开启。
- 关闭自动提醒不会禁用手动 OCR、OCR 进度及错误提示。
- 更新中文、日语、英语界面说明和帮助文档。

## 版本与安装

- `version.py`: `APP_VERSION = "2.3.2"`（程序唯一版本来源）。
- `ZiruPDF.spec` 根据 `version.py` 生成 Windows EXE 版本资源。
- `ZiruPDFInstaller.iss` 从 EXE 版本资源生成 2.3.2 安装程序文件名。
- 打包：Windows 环境运行 `zirupdf_tools.bat build-installer`，预期产物为 `ZiruPDF_Setup_2.3.2.exe`。
- `deploy/version.json` 为发布更新检查用的版本清单，只有新版安装程序已正式上线后才应上传到网站。

## 测试与限制

新增 `tests/test_ocr_hint_settings.cjs` 用于 OCR 提醒与持久化模拟测试。
已更新源码和发布元数据，未在此环境执行 Windows EXE/安装包实际构建与安装验收。
