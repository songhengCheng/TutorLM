# TutorLM

围绕学习资料展开阅读、追问和整理的学习工作台。

读论文、教材或课程讲义时，一个问题往往会带出几个更基础的问题。TutorLM 把这些追问保留在原文旁：你可以从一段文字建立节点，在解释中继续追问，再把子问题的解释带回上层问题。文档、问题之间的关系和生成过的解释都留在工作区，方便回看和继续学习。

项目包含浏览器工作台、文档与模型任务后端，以及管理用户、持久项目和临时计算实例的控制面。本地可以用离线模型体验工作流；接入真实模型后，解释、回讲和串讲由配置的模型服务生成。

[本地开发](#本地开发) · [工作台功能](#工作台功能) · [文档格式](#文档格式) · [数据与分享](#数据与分享) · [开发与测试](#开发与测试) · [协作入口](#协作入口)

## 工作台功能

### 在原文旁提问，保留问题的来处

导入资料后，由你决定哪些内容值得展开。选中一段文字并提问，工作台会建立与原文关联的节点；之后可以从节点返回对应位置，查看问题最初出现的上下文。文档上传不会自动生成一整张知识图。

工作区分为三个区域：左侧管理文档，中间阅读原文，右侧查看节点及其连接。阅读区支持搜索和来源定位，知识图可以缩放、平移和调整布局，也可以跟随阅读区滚动。

### 把追问展开成分支

遇到仍不理解的术语或推导，可以在解释中继续圈选并追问，建立下一层节点。每个节点可以保存自己的问题和解释，父子关系保留这次追问与上层问题的联系。重新生成的解释会保留历史版本，便于比较不同讲法。

例如，阅读一个证明时，可以先对“这一步为什么成立”提问，再把其中的定义和适用条件分别展开。弄清这些子问题后，回到原来的推导，检查它们如何共同支持这一步。

### 回到上层问题，整理已经展开的解释

“回讲”会读取父节点已有的解释和已解释的子节点，生成一版连接这些内容的新解释。首次使用需要设置学习档案；父节点没有解释、没有子节点，或子节点尚未得到解释时，不能直接完成回讲。

部分子节点尚未解释时，系统会标记结果不完整。生成了回讲，也不代表学习者已经掌握相关知识；工作台中的掌握状态由用户自己判断和标记。

### 将节点整理成讲稿，带走学习记录

选择节点后，可以生成分段串讲，查看历史结果，并将讲稿复制、下载为 Markdown 或保存为项目文档。工作台也支持导出当前文档与节点解释的 Markdown，以及知识图的 JSON 数据。

这里的串讲产物是文本讲稿，包含段落和时长信息；当前没有交付语音合成或音频播放能力。

### 用自己的副本继续学习

接入控制面后，用户可以保存多个项目，并通过分享字符串让别人导入一份独立副本。接收者可以继续提问和修改，双方的内容互不覆盖。分享的权限、计算申请和数据保留方式见下方[数据与分享](#数据与分享)。

## 本地开发

第一次运行，建议先启动**独立工作台**：实际应用后端负责文档、节点和任务，模型使用确定性的离线 `fake` provider。这个模式不需要 Docker 或模型 API Key，适合检查界面和业务流程；离线回答不代表真实模型的教学效果。

### 环境要求

| 工具 | 要求 | 用途 |
| --- | --- | --- |
| Python | 3.11 或更高版本 | 应用后端、控制面及本地静态服务器 |
| Node.js | 22.13.0 或更高版本 | PDF.js 依赖、前端构建与测试 |
| npm | 随 Node.js 安装 | 按锁文件安装前端依赖 |
| Git | 可用即可 | 获取源码 |
| PowerShell | 用于以下启动脚本 | 仓库提供的本地开发入口 |

下面的命令面向 **Windows PowerShell**。除明确切换目录的步骤外，均在仓库根目录执行。当前 Python CI 使用 3.12。

### 1. 获取源码并安装依赖

```powershell
git clone https://github.com/namenotfound64/TutorLM.git
cd TutorLM

python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r instance-app/backend/requirements-dev.txt

Push-Location instance-app/frontend
npm ci
Pop-Location
```

前端依赖中包含锁定版本的 `pdfjs-dist`。本地服务器会读取该目录中的 PDF.js 文件，因此第一次运行 PDF 阅读器前需要完成 `npm ci`。

### 2. 启动应用后端

在第一个终端运行：

```powershell
.\scripts\dev-instance-app.ps1
```

默认地址为 `http://127.0.0.1:8091`。脚本使用离线模型，将业务数据写入 `var/instance-app/data/`，将应用会话写入独立的 `var/instance-app/sessions/s01/`。这些目录不纳入 Git。

启动后可用以下地址检查服务：

| 地址 | 用途 |
| --- | --- |
| `http://127.0.0.1:8091/health` | 服务存活检查 |
| `http://127.0.0.1:8091/health/ready` | 就绪检查 |
| `http://127.0.0.1:8091/api/runtime` | 运行时及能力信息 |

应用后端没有 `/docs` 页面。完整路由说明见[应用 API 文档](docs/instance-app-api.md)。

### 3. 启动工作台

保持后端运行，在第二个终端进入仓库根目录：

```powershell
.\.venv\Scripts\python.exe instance-app/workbench/scripts/dev-server.py --port 8092 --api http://127.0.0.1:8091
```

打开 **http://127.0.0.1:8092/**，使用本地演示码 **`TLM-DEV-LOCAL`** 进入。该字符串是开发脚本提供的演示值，不是线上邀请码。

可以按下面的顺序检查一次完整流程：

1. 导入一份 Markdown 或 TXT，或新建文档并粘贴正文。仓库提供了 [Markdown 示例](instance-app/workbench/scripts/fixtures/markdown-support.md)。
2. 选中一段文字并提问，查看生成的节点及其原文位置。
3. 在解释中继续追问，建立子节点，再尝试回讲。
4. 选择节点生成串讲，或导出 Markdown / JSON。
5. 若要检查 PDF 阅读，上传自己的文本型 PDF，或使用[仓库中的测试 PDF](output/pdf/TutorLM-PDF-reader-test.pdf)。

本地服务器将页面、Reader、PDF.js 和 `/api` 代理放在同一来源下。请通过上述地址访问工作台，不要直接双击 HTML 文件打开页面。

### 4. 按需启动控制面

如果只修改阅读器、节点交互或应用业务逻辑，可以先不运行控制面。开发用户准入、容量调度和租约生命周期时，再安装并启动它：

```powershell
.\.venv\Scripts\python.exe -m pip install -r runtime-host/control-plane/requirements-dev.txt
.\scripts\dev-runtime-host.ps1
```

未指定外部配置时，该脚本在 `http://127.0.0.1:8090` 启动使用 `fake` adapter 的控制面：

- `/docs`：控制面接口文档。
- `/control/capacity`：容量信息。

**两个 `fake` 的含义不同**：应用的 `fake` provider 生成离线模型结果，实际文档服务仍在运行；控制面的 `fake` adapter 只模拟实例生命周期，不会启动真实应用容器。只运行控制面，或把它接到工作台代理上，都不能替代完整的受控实例联调。

### 开发参数与端口

```powershell
.\scripts\dev-instance-app.ps1 -Help
.\scripts\dev-instance-app.ps1 -Port 8093
```

修改后端端口后，同步调整工作台服务器的 `--api`。端口被占用时，应用启动脚本会报错退出，不会结束占用端口的进程。

`-AllowImplicitSession` 可显式启用用于开发的 scratch 工作区及隐式会话，具体参数见脚本帮助。常规体验使用默认的演示码流程即可。

## 文档格式

| 格式 | 当前能力 | 使用边界 |
| --- | --- | --- |
| Markdown / TXT | 上传、版本管理、文本抽取、搜索、选区提问与来源定位 | 文本使用 UTF-8；Markdown 原始 HTML 不执行，远程图片显示替代文本 |
| PDF | 保存原件、抽取文本层；通过 PDF.js 显示原始页面、文本层、搜索与选区 | 当前没有 OCR，扫描件不能按有文本层的 PDF 使用 |
| TeX | 后端支持单文件及 ZIP 工程校验、源码引用，并提供通用源码 Reader 模块 | 尚未形成完整的在线编辑与编译工作流 |
| TeX 编译产物 | 已有构建与产物接口设计 | 未提供安全编译执行器，编译请求返回 503 |

Markdown 阅读和解释卡片支持标题、表格、列表、任务列表、代码块、引用及 KaTeX 数学公式。PDF.js 的脚本、worker 和字体等资源由项目自托管。

上传的新内容会形成新的文档版本。节点来源引用绑定具体版本，不会因为原文件更新就悄悄改指另一段文字。来源解析和 Reader 的细节见[文档阅读器设计](docs/document-readers.md)。

## 模型与配置

### 应用模型

应用实现了三种 provider：

| Provider | 用途 |
| --- | --- |
| `fake` | 不调用远端模型，生成确定性结果，用于本地开发和回归 |
| `openai_compatible` | 通过 Chat Completions 协议调用配置的模型服务 |
| `gemini` | 通过原生 `generateContent` 协议调用 Gemini 服务 |

接入远端模型时，应用读取以下环境变量；模型名称和接口地址由实际服务决定：

| 变量 | 含义 |
| --- | --- |
| `TUTORLM_PROVIDER` | 选择 provider |
| `TUTORLM_MODEL_API_BASE` | 模型服务地址 |
| `TUTORLM_MODEL_API_KEY` | 模型服务密钥，仅在服务端提供 |
| `TUTORLM_MODEL_NAME` | 服务支持的模型名称 |
| `TUTORLM_MODEL_TIMEOUT_SECONDS` | 请求超时，默认 30 秒 |
| `TUTORLM_MODEL_MAX_TOKENS` | 模型输出 token 配置，默认 512 |

`dev-instance-app.ps1` 会强制使用 `fake` 并清除继承的模型地址和密钥，单独设置上述变量不会使该脚本调用真实模型。需要真实模型时，应按[应用后端说明](instance-app/backend/README.md)配置应用进程，并通过 `python -m backend` 或应用容器入口启动，同时设置正确的数据目录和会话目录。配置字段以 [HostConfig](instance-app/backend/host/config.py) 为准。

远端模型会接收生成任务所需的文档片段、节点或学习上下文，并可能产生服务费用。本地部署不代表模型推理也在本地。模型请求失败会报告错误，不会自动换成离线答案伪装成功。

解释、串讲和回讲采用后台任务及轮询方式。任务可以查询和取消；当前没有交付 HTTP SSE 或逐 token 流式回答，事件接口返回 501。

### 控制面配置

控制面使用独立的 `CONTROL_PLANE_*` 配置，可以通过 TOML 管理准入策略、槽位数量、计算期限和项目保留期限。配置优先级为：

**内置默认值 → TOML 文件 → 环境变量 → 显式代码覆盖**，后面的值覆盖前面的值。

根目录 [config.example.toml](config.example.toml) 是示例，不会因文件存在而自动加载。通过 `CONTROL_PLANE_CONFIG`、CLI 的 `--config`，或开发脚本的 `-ConfigPath` 指定文件。详细字段见[配置说明](docs/configuration.md)，管理命令见[控制面 CLI](docs/cli.md)。

应用的 `TUTORLM_*` 与控制面的 `CONTROL_PLANE_*` 分别负责两层配置。不要把控制面 TOML 当作应用直接读取的配置文件，也不要提交包含真实密钥的配置。

## 数据与分享

### 项目数据与计算分别管理

受控运行模式下，一个用户可以拥有多个持久项目，每个项目使用独立数据卷。容器提供临时计算，租约结束或页面关闭后回收计算，不等于立即删除项目数据。

项目保留时间由 `CONTROL_PLANE_WORKSPACE_IDLE_TTL_SECONDS` 决定：**代码默认值为 14400 秒（4 小时）**，也支持配置为 `inf`。具体环境可能覆盖该值，因此不要把“持久项目”理解为默认永久保存。匿名临时租约使用另一套临时数据清理策略。

用户创建时会得到 UUID 和恢复凭据；请私下保存恢复凭据，用于重新登录。当前没有邮箱、密码或 OAuth 登录，也没有恢复凭据丢失后的自动找回流程。

### 分享的是独立副本

项目所有者可以生成可复制、可撤销的分享字符串。接收者导入后得到自己的项目和数据卷，随后再申请计算资源。分享字符串不授予源项目的编辑权限，也不能替代系统邀请码或提高调度优先级。

在邀请模式下，导入副本后仍需有效系统邀请码才能申请计算。为复制已落盘的数据，导入过程可能停止源项目的计算，进行中的任务可能中断；适合在完成当前生成任务后分享。

该流程需要真实控制面、应用实例和对应网关集成；前面的独立工作台演示不提供完整的托管分享环境。

### 哪些内容会随项目保存

| 内容 | 保存范围 | 随持久项目分享复制 |
| --- | --- | --- |
| 文档、版本、节点、连接、解释、串讲和学习数据 | 应用的项目数据目录及数据库 | 是 |
| 用户身份、租约、邀请码等管理数据 | 控制面数据库 | 否 |
| 应用会话和鉴权数据 | 项目卷外的会话目录 | 否 |
| 阅读偏好、部分视图状态、本地高亮与自由绘图笔迹 | 浏览器本地存储 | 否 |

绘图和本地高亮目前不能随项目跨设备同步。项目复制、租约和会话的完整约定见[运行协议 v4](contracts/instance-runtime-v4.md)及[持久工作区说明](docs/workspaces.md)。

## 代码结构

项目将应用业务和实例调度分开维护，通过版本化 HTTP 契约协作。

| 目录 | 主要职责 |
| --- | --- |
| `instance-app/workbench/` | 完整工作台页面、交互脚本、样式和本地静态服务器 |
| `instance-app/frontend/` | 通用文档 Reader、PDF.js、邀请码入口和租约生命周期模块 |
| `instance-app/backend/` | 应用 HTTP API、文档与节点、解释任务、插件和存储 |
| `instance-app/deploy/` | 应用镜像构建 |
| `runtime-host/control-plane/` | 用户、持久项目、邀请码、容量、租约、实例生命周期和分享复制 |
| `runtime-host/gateway/` | Nginx 同源入口和受控 API 转发 |
| `runtime-host/deploy/` | 控制面与网关的部署模板 |
| `contracts/` | 应用与控制面之间的版本化协议 |
| `docs/` | 架构、配置、API、Reader 与开发约定 |
| `scripts/` | 本地启动及架构边界检查 |

“容器外”是相对于用户应用实例而言，控制面本身也可以容器化。`instance-app/backend/host/` 是应用内部的装配与运行模块，不是外层控制面。两侧不共享数据库，也不导入对方的私有模块。

应用内置 `documents`、`explanations`、`knowledge-narration` 三个插件，解释、串讲、回讲分别由 `explanation.v1`、`narration.v1`、`retelling.v1` 工作流处理。扩展插件时，使用 [Plugin SDK](docs/plugin-sdk.md) 提供的接口，由 Host 校验并提交结果。插件是随应用构建交付的可信 Python 代码，不是用于运行任意用户代码的沙箱。

## 开发与测试

按修改范围运行相关测试。应用后端和控制面的测试应在**不同 pytest 进程**中执行，避免两个测试包相互影响。以下命令均在仓库根目录执行。

### 应用后端

```powershell
.\.venv\Scripts\python.exe -m pip install -r instance-app/backend/requirements-dev.txt
.\.venv\Scripts\python.exe -m pip install pyflakes
$env:PYTHONPATH = (Resolve-Path instance-app).Path
.\.venv\Scripts\python.exe -m pytest instance-app/backend/tests -q
.\.venv\Scripts\python.exe -m pyflakes instance-app/backend
```

### 前端模块与工作台

```powershell
Push-Location instance-app/frontend
npm ci
npm test
Pop-Location

node --test instance-app/workbench/scripts/test-access.cjs instance-app/workbench/scripts/test-workspace-view.cjs instance-app/workbench/scripts/test-markdown.cjs
```

通用 Reader 与 PDF.js 制品通过以下命令构建：

```powershell
Push-Location instance-app/frontend
npm run build
Pop-Location
```

这个构建目标只覆盖通用前端制品，不会自动打包完整的 `workbench/` 页面。

### 控制面与架构边界

```powershell
.\.venv\Scripts\python.exe -m pip install -r runtime-host/control-plane/requirements-dev.txt
.\.venv\Scripts\python.exe -m pytest runtime-host/control-plane/tests -q
.\.venv\Scripts\python.exe scripts/check-boundaries.py
```

普通控制面测试使用 fake adapter。真实 Docker 集成测试需要显式开启，并使用隔离的本地环境；不要针对已有用户数据的服务运行测试。浏览器集成检查的环境变量和启动方式见[工作台开发说明](instance-app/workbench/README.md)。

CI 分别定义在 [instance-app.yml](.github/workflows/instance-app.yml) 和 [runtime-host.yml](.github/workflows/runtime-host.yml)。修改跨层行为前，先确认契约及两侧责任范围；具体流程见[开发约定](docs/development.md)。

## 部署范围与当前限制

仓库提供应用和控制面的 Dockerfile，以及控制面与 Nginx 网关的 Compose 模板。应用实例由控制面动态创建，Compose 本身不直接列出所有用户实例。

部署时需要同时交付工作台页面、通用 Reader 和 PDF.js 资源，并由网关同源提供。当前应用镜像构建会导出通用前端制品，但没有把完整工作台页面一并打包；仅构建应用镜像不能得到可直接访问的完整站点。构建与接线说明见[应用部署](instance-app/deploy/README.md)和[外层部署](runtime-host/deploy/README.md)。

当前还没有交付以下能力：

- 扫描 PDF 的 OCR、安全隔离的 TeX 编译器和完整在线 TeX 编辑工作流。
- HTTP SSE 与模型流式回答。
- 浏览器绘图、本地高亮的服务端保存及跨设备同步。
- 邮箱或 OAuth 登录、恢复凭据找回。
- 旧匿名临时工作区到持久项目的一键迁移。
- 应用级可信 drain/flush、无损任务迁移及完整活动任务探针。

本地开发服务器只用于开发。公网 TLS、访问权限、Cookie 配置、容量和长时间运行需要在实际部署环境中单独验证；源码中的实现和本地测试记录不能证明某个线上环境已经启用这些能力。

## 协作入口

| 想了解什么 | 从这里开始 |
| --- | --- |
| 工作台启动、页面结构与浏览器测试 | [Workbench README](instance-app/workbench/README.md) |
| 已有能力和阶段性验收记录 | [功能总览](docs/feature-status.md) |
| 两层架构与协作规则 | [架构](docs/architecture.md)、[开发流程](docs/development.md)、[AGENTS.md](AGENTS.md) |
| 应用分层、HTTP API 与插件 | [后端 README](instance-app/backend/README.md)、[设计](docs/instance-app-design.md)、[API](docs/instance-app-api.md)、[SDK](docs/plugin-sdk.md) |
| 文档来源、Reader 与 TeX 安全边界 | [文档阅读器](docs/document-readers.md)、[TeX 编译设计](docs/tex-compilation-security.md) |
| 用户、持久项目、邀请码和分享 | [工作区](docs/workspaces.md)、[运行协议 v4](contracts/instance-runtime-v4.md) |
| 控制面配置和管理操作 | [配置](docs/configuration.md)、[CLI](docs/cli.md)、[配置示例](config.example.toml) |
| 协议版本与兼容性 | [契约目录](contracts/README.md)、[实例协议 v1](contracts/instance-runtime-v1.md) / [v2](contracts/instance-runtime-v2.md) / [v3](contracts/instance-runtime-v3.md)、[网关协议 v1](contracts/gateway-lease-v1.md) / [v2](contracts/gateway-lease-v2.md) |

部分设计文档保留了早期阶段的描述；涉及接口字段、默认值和实际能力时，请同时核对当前配置、路由与测试。报告问题时，附上启动方式、运行模式、相关错误和复现步骤，便于定位到工作台、应用后端或控制面。

凭据、本地 `.env`、真实配置、数据库、运行数据、日志、镜像归档及 `reference/` 不纳入版本控制。仓库提交和推送由维护者明确执行。

## 许可证

项目采用 [MIT License](LICENSE)。随仓库提供的第三方资源保留各自的许可证，例如 [KaTeX](instance-app/workbench/assets/katex/LICENSE) 和 [markdown-it](instance-app/workbench/assets/markdown-it/LICENSE)。
