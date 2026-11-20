# ZiruPDF 2.3.2

本版本在 2.3.1 基础上增加扫描件 OCR 顶部提醒的永久关闭功能；版本号统一来自 `version.py`。详见 [UPDATE_2.3.2.md](UPDATE_2.3.2.md)。

# ZiruPDF 2.3.0

**打印重做**：默认矢量打印（`print_backend.py`：PDFium `FPDF_RenderPage` 直接画到 Windows 打印机 DC），可选无损图像打印；
新打印窗口 `print_dialog.py`（整张纸预览、后台打印）；版面计算 `print_layout.py` 由预览和打印共用。
**统一 Logo**（`static/brand/`，由 `tools/build_brand_assets.py` 从 `zirupdf.ico` 生成）；**界面双向切换**（Ctrl+Shift+M）。
详见 UPDATE_2.3.0.md；Windows 实机验收清单 deploy/WINDOWS_ACCEPTANCE_2.3.0.md。

# ZiruPDF 2.2.5

**阅读器改为 SumatraPDF 的阅读方式**（只改阅读功能，其他功能不变）：一条顶部工具栏、灰色背景上连续排列页面、
默认连续显示 + 适合页面、六种显示方式（单页 / 双页 / 书本视图 × 连续与否）、SumatraPDF 缩放档位与快捷键、
目录树跟随阅读位置、工具栏查找并在页面上标出结果、后退 / 前进、中键自动滚动、Ctrl+拖动框选。
渲染改为“布局 + 按需渲染”：只有视口附近的页面在 DOM 中并持有画布，缩放先拉伸已有位图再清晰重绘。
另有“SumatraPDF 简洁界面”：只显示标签栏 + ☰ 菜单、一条工具栏和文档，与完整界面随时切换。
详见 UPDATE_2.2.5.md；新增 static/sumatra_reader.css、static/sumatra_mode.js，app.js §07 重写。


**功能拆分（ZiruImage）**：「批量图片处理」「照片信息」以及智能拼版中的照片排版、证件照排版，已移到独立软件
ZiruImage（另一个源码包）。ZiruPDF 的其他功能、界面、快捷键、语言和保存逻辑不变；智能拼版保留 PDF 多页拼版
（照片仍可与 PDF 一起排，每张照片算一页）。旧的已保存任务若含“批量图片处理”步骤，会明确提示并禁止执行，
删除该步骤后即可继续使用。详见 UPDATE_SPLIT_ZIRUIMAGE.md。

新增「智能拼版」「安全涂黑」「手写签名」。详见 UPDATE_2.2.0.md（其中照片信息 / 批量图片处理 / 照片排版部分已移到 ZiruImage）。
源码：imposition.py（拼版核心 + PDF 适配器）、redaction.py（真删除涂黑 + 独立验证）、phone_sign.py（手机签名会话，
复用扫描中心的局域网服务）、v22_api.py（接口）、static/v22_tools.js / v22_tools.css（界面）、
templates/phone_sign.html + static/phone_sign.js / .css（手机签名页）。

新增购物车式“常用任务”：从原功能页加入步骤，回原页面修改，预览后批量执行。详见 UPDATE_2.1.1.md。

# ZiruPDF 2.0.41 — 源码说明（内部）

ZiruPDF · PDF & Document Toolkit（Formerly ZiruDoc / FreeDoc）。Publisher：ZiruLab。

**内部源码，不对外公开。** ZiruPDF 是免费使用的专有软件（Freeware / Proprietary，见 `LICENSE.txt`），
对外只发布安装程序 `ZiruPDF_Setup_<版本>.exe`、`version.json`、发布说明和校验值；
源码、`.spec`、`tests`、`tools` 不对外提供。第三方组件的许可见 `THIRD_PARTY_NOTICE.txt`。

本地 PDF 工具箱：阅读、编辑、页面整理、水印/图章、扫描、OCR、格式转换、对比、批量处理。
所有处理都在本机完成，文件不会上传。更新记录见 `CHANGELOG.md`。

## 目录结构（2.0.26 合并后）

```
run_desktop.bat        启动桌面版（双击）
run.bat                启动浏览器版（http://127.0.0.1:8848）
zirupdf_tools.bat      维护工具菜单：环境安装 / 引擎 / OCR 模型 / PDF.js / 检查 / 打包 …
                       也可直接执行：zirupdf_tools.bat <命令>，命令见下表

version.py             版本号（唯一需要改版本号的地方，见“发布新版本”）
desktop.py             桌面外壳（PySide6 / QtWebEngine）、菜单、单实例、保存到原文件
print_layout.py        打印版面（2.3.0）：页面范围、缩放、方向、每张纸多页；预览和打印共用
print_backend.py       打印后端（2.3.0）：Windows 矢量打印（PDFium → 打印机 DC）、无损图像打印、后台任务
print_dialog.py        打印窗口（2.3.0）：设置 + 整张纸预览
app.py                 本地引擎（Flask）：页面、全部工具接口
doc_tools.py           §1 页面组合（缩略图操作）  §2 智能 PDF 对比  §3 PDF→Word/Excel 版式还原
help_center.py         帮助中心接口：文档属性、检查更新、关于
phone_scan.py          扫描中心（手机局域网扫描、自动方向、PDF 导入）
watermark_engine.py    水印 / 图章（含文字变量）
smart_ocr.py           OCR 调度（PaddleOCR / Tesseract），图片 OCR / 表格等旧入口
scan_ocr.py            扫描页识别（2.0.40）：表格检测、按单元格识别、列表标记补回、段落结构
ocr_layer.py           PDF 侧（2.0.40）：逐页判断是否需要 OCR、坐标换算（旋转 / 裁剪框）、
                       可搜索文字层、OCR 基准（/PieceInfo）、扫描图像的局部像素修补
ocr_service.py         OCR 服务（2.0.40）：语言选择、按页识别、可搜索 PDF、编辑器 OCR 对象、Word 输出
pdf_protection.py      保护 PDF（2.0.41）：打开密码、权限限制、移除密码（pypdf）
digital_signature.py   数字签名（2.0.41）：.p12 / .pfx 证书签名与签名验证（pyHanko）
paddle_bridge.py       在独立的 .paddle_env 中运行 PaddleOCR（必须保持独立文件）
engine_manager.py      PDFium 线程锁 + 外部引擎查找（LibreOffice / Tesseract / qpdf / Paddle）
pdf_engine.py          PDF 编辑引擎适配层（native / MuPDF / Apryse）
run.py                 浏览器版启动器；python run.py test [unit|e2e|all] 运行测试

templates/index.html   主界面        templates/phone_scan.html  手机端页面
static/app.js          主界面全部脚本（按 §01…§25 分节，文件开头有目录）
static/app.css         主界面全部样式（按 §1…§8 分节，顺序决定覆盖关系）
static/sumatra_reader.css  阅读器（2.2.5，SumatraPDF 风格）：工具栏、灰色画布、页面、目录树、通知；在其他样式之后加载
static/sumatra_mode.js    阅读器的两种界面：完整界面 / SumatraPDF 简洁界面（标签栏、☰ 菜单、主页）
static/phone_scan.js / phone_scan.css / scan_editor.js   手机端页面与共用的裁边编辑器
static/tool_catalog.json  工具目录（工具面板、开始页、桌面菜单、工具数量都由它生成）
static/help/<语言>/manual.html  用户手册（zh-CN / ja / en，章节 id 相同）
static/i18n.js         界面语言（2.0.32）：按中文原文查 static/i18n/ja.json、en.json 显示日语 / 英语
static/i18n/           界面翻译字典：键 = 界面上的中文原文，{0}{1}… 代表数字、文件名等；
                       新增界面文字时在两个文件里加同一个键（tests 检查占位符和工具名称是否齐全）
ui_i18n.py             桌面版菜单和对话框使用同一份字典
ui_state.py            界面存储项（localStorage 中 freedoc.* / freedoc_*）的本机持久副本（2.0.35）
static/persist.js      桌面版：启动时把持久副本放回 localStorage，修改时同步回引擎；键名不变（浏览器版不启用）
static/vendor/         PDF.js、qrcode-generator（第三方，未修改）

tests/test_backend.py  后端测试        tests/e2e/test_ui.py  浏览器端到端测试
tests/assets.py        测试用文档生成  tools/devtools.py      开发工具（截图比对、CSS 审计、工具统计）
deploy/version.json    “检查更新”读取的版本文件（上传到 zirupdf.zirulab.org 根目录）
installer/             安装程序简体中文语言文件（Inno Setup 6.5 以前没有自带）
THIRD_PARTY_NOTICE.txt 第三方组件与许可（“帮助 → 关于”中显示，也随安装程序放在安装目录）
```

### 前端为什么是一个 app.js

所有脚本原本就是共享全局作用域的普通脚本，按固定顺序执行。合并后顺序不变（§01 → §25），
后面的分节可以使用前面分节定义的任何东西。修改时搜索 “§NN” 跳到对应分节；新增功能放进
相关分节，或在末尾新增一个分节并更新文件开头的目录。样式同理：`static/app.css` 中后面的分节
覆盖前面同等优先级的规则，不要随意调换分节顺序。

## zirupdf_tools.bat 命令

| 编号 | 命令 | 作用（原脚本） |
|---|---|---|
| 1 | setup-desktop | 桌面版：创建 / 更新 Python 环境，首次使用必须运行（setup_desktop.bat） |
| 2 | setup | 浏览器版：创建 .venv（setup.bat） |
| 3 | repair | 浏览器版：删除并重建 .venv（repair_environment.bat） |
| 4 | engines | 用 winget 安装 qpdf + LibreOffice + Tesseract（install_engine_suite.bat） |
| 5 | optional-tools | 用 winget 安装 LibreOffice + Tesseract（install_optional_tools.bat） |
| 6 | paddle | 安装 PaddleOCR / PP-Structure 到 .paddle_env（install_paddle_engine.bat） |
| 7 | ocr-models | 下载 Tesseract tessdata_best 高精度模型到 ocr_models\best（install_best_ocr_models.bat） |
| 8 | pdfjs | 下载 PDF.js 到 static\vendor\pdfjs（install_pdfjs.bat） |
| 9 | check-engines | 检查 PDF / Office / OCR 引擎（check_engines.bat） |
| 10 | check-components | 检查 LibreOffice 与 Tesseract 语言包（check_local_components.bat） |
| 11 | diagnose | 启动诊断：Python、.venv、8848 端口、依赖（diagnose_startup.bat） |
| 12 | build-exe | PyInstaller 打包 dist\ZiruPDF\ZiruPDF.exe（build_standalone.bat） |
| 13 | build-installer | 生成安装包（缺少 EXE 时先打包）（build_installer.bat） |
| 14 | installer-only | 用已有的 dist\ZiruPDF 生成安装包（build_installer_only.bat） |
| 15 | cleanup | 删除旧版本遗留文件（cleanup_to_1.3.66.bat） |
| 16 | open | 打开 http://127.0.0.1:8848（OPEN_LOCAL_URL.html） |

## 打开慢的排查（2.0.27 起）

每次打开 PDF、每次启动桌面版都会记录各步骤耗时：

- 位置：`%LOCALAPPDATA%\FreeDoc\logs\freedoc_timing.log`（每行一条 JSON，超过 1 MB 自动轮换）
- 界面里：帮助 → 联系我们 → 诊断信息（最近 5 次打开 + 最近 3 次启动），可一键复制
- 打开记录的步骤：解析 PDF → 第一页显示 → 缩略图占位 → 首批缩略图 → 扫描件检查 → 缩略图全部；
  来源分为 打开 / 桌面打开（双击、用 ZiruPDF 打开、最近文件）/ 切换（标签切换、撤销重做）
- 启动记录的步骤：加载组件 → 本地引擎就绪 → 窗口显示 → 界面加载完成 → 文件交给界面

“第一页显示”慢 → PDF 本身大或复杂；“加载组件”慢 → 首次启动 / 杀毒软件扫描；
“缩略图全部”慢只影响左侧缩略图，不影响阅读。

## Windows 回归检查清单（2.0.27）

自动测试（`python run.py test all`）在 Linux + Chromium 上覆盖了下面大部分功能；
以下是需要在 Windows 桌面版上亲手确认的项目。建议用安装版（`zirupdf_tools.bat build-installer` 生成）测试。

### A. 启动、打开、保存
- [ ] 双击 PDF 启动 ZiruPDF：文件直接打开；诊断信息里“文件交给界面”有记录
- [ ] ZiruPDF 已打开时再双击另一个 PDF：在原窗口新标签打开，约 1 秒内出现
- [ ] 文件 → 最近打开：长文件名中间省略，名字里有 “&” 也正常显示
- [ ] 打开超长文件名的 PDF：底部状态栏文件名带 “…”，不压住页码；鼠标悬停显示全名；窗口标题不过长
- [ ] 200 页以上的 PDF：第一页立即显示，左侧缩略图先出当前附近的页，滚动缩略图时优先加载可见页
- [ ] 两个大文件之间切换标签：缩略图不再重新生成
- [ ] 旋转一页后保存（Ctrl+S）：原文件被覆盖；用其他软件打开确认已旋转
- [ ] 文件被其他程序占用 / 只读 / 在 OneDrive 文件夹：保存时给出明确提示，不丢数据
- [ ] 未保存时关闭标签、退出程序：出现提醒
- [ ] 150% / 200% 显示缩放下界面清晰、不错位

### B. 28 个工具（每个用一个真实文件跑一次，结果能打开）
- 编辑与注释
  - [ ] PDF 编辑：双击原文字修改（中文 / 日文输入法）、加文字和形状、保存后重新打开确认
  - [ ] 水印 / 图章：文字水印、变量 {filename}{page}{pages}、内置图章、保存“我的图章”、批量加水印、合并
  - [ ] 页码 / 页眉页脚
- 页面整理
  - [ ] 页面管理  - [ ] 合并 PDF  - [ ] 拆分 PDF  - [ ] 提取页面  - [ ] 旋转页面
  - [ ] 裁剪页面  - [ ] 删除空白页  - [ ] 统一页面尺寸
  - [ ] 缩略图拖动：插入线和“放到第 N 页之后”提示正确；多选拖动显示“N 页”；拖到列表边缘自动滚动
- 转为 PDF
  - [ ] 扫描中心：手机扫码连接（首次允许防火墙“专用网络”）、横拍自动转正、导入 PDF、生成后识别文字
  - [ ] Office → PDF：Word、Excel、PowerPoint 各一个（装 Office 时用 Office，未装时用 LibreOffice）
  - [ ] 图片 → PDF
- 从 PDF 转换
  - [ ] PDF → Word（版式还原，Word 打开检查段落 / 表格 / 双栏）  - [ ] PDF → Excel（数字可计算）
  - [ ] PDF → PowerPoint  - [ ] PDF → 图片  - [ ] 提取内容
- OCR 与扫描
  - [ ] 可搜索 PDF  - [ ] OCR → Word  - [ ] 图片 OCR  - [ ] 表格 → Excel  - [ ] 扫描件优化
  - [ ] 打开扫描件时出现“识别文字”提示，识别后能搜索
- 优化与比较
  - [ ] 压缩 PDF  - [ ] 对比 PDF（增删页面的两个版本）  - [ ] 图片批处理  - [ ] 批量处理（含 ZIP 下载）

### C. 打印与其他
- [ ] 打印（Ctrl+P）：预览、选择打印机、页面范围
- [ ] 帮助：F1、使用说明、文档属性（长文件名完整显示）、检查更新
- [ ] zirupdf_tools.bat：双击出现菜单；选 9（check-engines）、11（diagnose）能正常运行并返回菜单

发现问题时，请附上：帮助 → 联系我们 → 诊断信息（复制）和出问题的文件（如可以）。

## 开发者：运行与测试

```
zirupdf_tools.bat setup-desktop     首次：安装桌面版环境（%LOCALAPPDATA%\FreeDocToolbox\venv）
run_desktop.bat                     启动桌面版
python run.py test                  后端测试（几秒）
python run.py test e2e              浏览器端到端测试（需要 pip install playwright; playwright install chromium）
python run.py test all              全部
python tools/devtools.py --help     开发工具（界面状态截图 / 样式比对 / CSS 审计 / 工具统计）
```

## 打包安装程序（原 BUILD_INSTALLER_README）

目标：生成普通的 Windows 安装程序，最终用户不需要 Python、pip、PySide6、Flask 或任何 .bat。

打包电脑需要：Windows 10/11 x64、Python 3.11–3.14 x64、首次打包时联网（pip 依赖）、
Inno Setup 6（生成最终安装程序时）。

1. 把源码解压到较短的路径，例如 `C:\ZiruPDF`。
2. 运行 `zirupdf_tools.bat build-installer`。
3. 第一步用 PyInstaller 生成 `dist\ZiruPDF\ZiruPDF.exe`。
4. Inno Setup 再生成 `dist_installer\ZiruPDF_Setup_<版本>.exe`。

没有安装 Inno Setup 时，`zirupdf_tools.bat build-exe` 仍然可以生成独立的程序文件夹。
只需要把 `ZiruPDF_Setup_<版本>.exe` 发给用户。

打包内容：Python 运行时、PySide6 / QtWebEngine、Flask + Waitress 本地引擎、pypdf / pypdfium2 /
pdfplumber / ReportLab、OpenCV / NumPy / Pillow、DOCX/PPTX/XLSX 库、ZiruPDF 界面、PDF 引擎适配层、
OCR 支持文件、内置水印 / 图章工作台。

在用户电脑上自动检测（不会被 ZiruPDF 静默安装）：Microsoft Office 自动化、LibreOffice、可选的外部 OCR 引擎。

公开 / 商业发布前请检查 `THIRD_PARTY_NOTICE.txt` 以及所有打包组件的许可证；建议对安装程序做代码签名，
以减少 SmartScreen 警告。

## 最终用户安装（原 USER_INSTALL_README）

1. 双击 `ZiruPDF_Setup_<版本>.exe`，按提示安装。
2. 从开始菜单启动 ZiruPDF。

用户电脑不需要 Python、pip、PySide6。可选能力：装有 Microsoft Office 时 Office→PDF 效果更好；
可用时会使用 LibreOffice 作为备选；部分 OCR / 高级外部引擎为可选，会自动检测。

## 精简源码包（原 SOURCE_LITE_README）

为减小体积，源码包不包含可选的高精度 OCR 语言模型
（ocr_models/best 下的 jpn / chi_sim / eng / osd.traineddata）。OCR 代码和功能入口都没有删除。
需要时运行 `zirupdf_tools.bat ocr-models` 下载这四个官方 tessdata_best 模型到 `ocr_models\best`，然后重启 ZiruPDF。

## 检查更新（原 deploy/README_UPDATE_CHECK）

“帮助 → 检查更新”读取 `https://zirupdf.zirulab.org/version.json`
（可用环境变量 `FREEDOC_UPDATE_URL` 改成别的地址，方便测试）。

发布新版本时：

0. 只改 `version.py` 里的 `APP_VERSION`。ZiruPDF.exe 的文件版本、窗口标题、关于、安装包版本和文件名
   （ZiruPDFInstaller.iss 从 ZiruPDF.exe 读取）、zirupdf_tools.bat 的提示都会自动跟着变。
   打安装包前要重新执行 build-exe（build-installer 发现已有 dist\ZiruPDF 时会跳过打包）。

1. 修改 `deploy/version.json`：`latest` 新版本号（只比较数字，2.0.100 > 2.0.25）、`released` 发布日期、
   `url` 下载页地址、`notes` 更新说明（按语言写，也可以只写一个字符串）。
2. 把 version.json 上传到 zirupdf.zirulab.org 网站根目录（Cloudflare Pages：放进发布目录后重新部署）。
3. 浏览器打开 https://zirupdf.zirulab.org/version.json 确认能看到内容。

请求由本机 ZiruPDF 引擎发出（不是网页），不需要 CORS；只带 User-Agent `ZiruPDF/<版本> (update-check)`，
不含任何文件或用户信息；自动检查每天最多一次，可在“帮助 → 设置”关闭；服务器上没有这个文件时只提示
“无法检查更新”，不影响其他功能。

## 架构备忘

### 桌面画布（原 DESKTOP_ARCHITECTURE）

Qt/PySide6 桌面外壳 → QtWebEngine 界面 → 原生 PDF 画布交互层 → 本地 Python PDF 引擎 → 原生内容流保存。
画布坐标直接使用 PDF 页面尺寸（`viewBox=0 0 page.width page.height`），字形、选区和批注都以 PDF 坐标表示，
CSS 只负责显示。

### PDF Engine 2.0（原 PDF_ENGINE_2_ARCHITECTURE，代码在 pdf_engine.py）

`UI -> ZiruPDF Editing API -> Engine Adapter -> Native / MuPDF / Apryse`，界面不依赖具体引擎。

原生编辑流程：渲染页面背景 → 提取文字与精确字符框 → 解析内容流并跟踪字体资源 → 解码 `Tj/TJ`
（简单字体按字节；Type0/CID 字体用 ToUnicode CMap；标准 `Uni*-UCS2-*` CMap 用 UTF-16BE）→
把可见的行与内容流中的短连续片段对应 → 每个块给出 `native_editable`、`native_mode`、`native_refs` 和字形框 →
保存时确认修改后的文字能用原字体资源编码 → 替换原内容流里的显示文字操作数 →
无法安全原生编码时，该对象改用兼容覆盖层。

安全规则：ZiruPDF 不臆造 CID 值。只有当 Unicode→编码映射能从已有字体 / CMap 反推，或字体使用标准
Uni*-UCS2 映射时，Type0 编辑才走原生方式，避免损坏子集字体。

后续方向：Identity-H/V CID 反向映射、字体子集扩展 / 新字形嵌入、竖排 CJK、文字矩阵与几何修改、
段落边界推断与有限重排、原生图片替换、原生矢量编辑、更强的增量保存与对象级撤销。

### 清理记录（原 CLEANUP_REPORT，1.3.66）

1.3.66 删除了外部水印工具包（约 41 MB）、旧水印源码副本、外部水印启动器、缓存和一次性补丁说明；
保留了内置 watermark_engine.py 及全部水印功能、OCR、PDF.js、各工具、测试、打包脚本和第三方声明。
2.0.25 完成了“去除从未生效的 CSS 规则”和前端拆分；2.0.26 又把前端合并回 `static/app.js` / `static/app.css`
（按分节组织，便于维护），把十几个 .bat 合并为 `zirupdf_tools.bat`。尚未进行的建议：把开发 / 打包脚本与
最终用户安装内容分开；OCR 语言包按需安装。
