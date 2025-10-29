# Blazor WASM Packager

## Introduction

Blazor WASM Packager is a resource integration tool designed specifically for Blazor WebAssembly applications. It compresses and merges core resources required for application execution (including framework files, business assemblies, configuration files, and custom scripts) into a single JavaScript file, significantly simplifying the deployment process and optimizing application loading performance.

This tool functions through a browser-based HTML page, requiring no backend service support. Simply place the HTML file in the root directory of your published application to use it.

## Benefits

1. **Simplified Deployment Process**
   - Merges dozens or even hundreds of scattered resource files into a single JS file, reducing the number of deployment files
   - Prevents application runtime errors caused by missing files
   - Reduces network request overhead when transmitting multiple files

2. **Improved Loading Performance**
   - Applies maximum-level GZIP compression to all resources, significantly reducing transmission size
   - Decreases HTTP request count, reducing the impact of network latency
   - Supports custom script minification to remove redundant code

3. **Enhanced Deployment Flexibility**
   - The generated single file can be easily integrated into various static resource hosting services
   - Supports micro-frontend architectures (MicroApp/ QianKun) environment adaptation
   - Facilitates version management and cache strategy implementation

4. **Overcoming Cross-Origin and Local Execution Restrictions**
   - Resolves cross-origin resource calling issues (CORS) in traditional Blazor WASM applications by embedding resources to avoid cross-origin requests
   - Enables direct application launch from the local file system (`file://` protocol) without relying on a web server
   - Suitable for offline scenarios or local demonstration environments without requiring server CORS configuration

5. **Ease of Use**
   - Pure frontend implementation with no need to install additional tools or dependencies
   - Provides visual progress feedback and detailed processing logs
   - Supports custom output file names and script merging rules


## Code Execution Process

### I. Initialization Phase

1. **Page Loading Preparation**
   - Initializes DOM element references (progress bar, log area, buttons and other interactive components)
   - Resets progress bar status to "Ready" (0%)
   - Clears log output area
   - Disables download button to prevent duplicate clicks

2. **User Parameter Collection**
   - Reads content from output file name input box (uses default naming rules when empty)
   - Parses custom JS file list (one path per line, relative to `_framework` directory)
   - Checks "Minify custom scripts" checkbox status (determines whether to remove comments and redundant spaces)


### II. Core Processing Flow

#### Step 1: Browser Compatibility Verification
- Detects if current browser supports `CompressionStream` API (used for GZIP compression)
- Terminates process and prompts: "Please use the latest version of Chrome, Edge, or Firefox browser" if not supported


#### Step 2: Parse Application Configuration File
1. **Download Configuration File**
   - Sends request to obtain `_framework/blazor.boot.json` (core configuration of Blazor application)
   - Records original size of configuration file and includes it in total statistics
   - Terminates process and displays specific error message in log if download fails (e.g., 404 error)

2. **Parse Configuration Content**
   - Parses JSON format configuration data
   - Extracts all resource paths from `resources` field (including runtime, assembly, jsModule, etc.)
   - Extracts `entryAssembly` value (used to generate default output file name)
   - Updates progress bar to 10% and logs "Configuration file parsing completed"


#### Step 3: Process Core Resource Files
1. **Task Preparation**
   - Collects list of resource files to be processed (including core files like `blazor.webassembly.js`)
   - Calculates progress proportion for each file based on total number of files (75% of total progress divided equally)

2. **Process Resources One by One**
   - Processes each resource file in a loop:
     1. Constructs complete URL: `_framework/[file name]`
     2. Downloads file and converts to Uint8Array format
     3. Records original file size (accumulates to total original size)
     4. Applies maximum-level (level:9) compression using `gzipCompress()` function
     5. Converts compressed data to Base64 encoding (processed in 129-byte chunks)
     6. Generates entry containing file name and Base64 content
   - Updates progress bar in real-time (e.g., "Processing 3/20: dotnet.wasm")
   - Logs and skips individual file processing failures without terminating the entire process


#### Step 4: Process Custom JS Files (Optional)
1. **Input Parsing**
   - Filters empty lines and invalid paths to obtain valid custom JS file list
   - Logs "No custom JS files specified, not generating Merge.js" and skips if list is empty

2. **Merging and Processing**
   - Downloads each specified custom JS file:
     - Records original size of each file
     - Applies `minifyJs()` processing when minification is enabled (preserves template strings and regular expressions)
     - Merges content (adds source comments unless minification is enabled)
   - Applies GZIP compression to merged content and converts to Base64 encoding
   - Generates `Merge.js` entry and adds to resource list


#### Step 5: Generate Output File
1. **Content Construction**
   - Adds generation time comment on first line (format: `// Generated on: YYYY-MM-DD HH:MM:SS`)
   - Creates JavaScript object containing all resources (keys are file names, values are Base64 content)
   - Generates self-executing function containing:
     - Resource decompression logic (uses `DecompressionStream` to process GZIP data)
     - Blazor startup configuration (overrides `loadBootResource` method to inject local resources)
     - Temporary URL cleanup logic (releases memory usage)

2. **File Name Processing**
   - Prioritizes user-specified file name (must include `.js` extension)
   - Uses rule: `[entryAssembly].published.js` when not specified
   - Defaults to `blazor-resources.published.js` when `entryAssembly` is unavailable


#### Step 6: Generate Download Link
1. **Statistical Information Calculation**
   - Total original size: sum of original sizes of all resources + configuration files + custom scripts
   - Generated file size: UTF-8 encoded byte count of final JS file
   - Compression ratio: `(generated file size / total original size) × 100%`

2. **Create Download**
   - Converts JS content to Blob object (MIME type: `application/javascript`)
   - Generates temporary download URL (`URL.createObjectURL`)
   - Automatically triggers download and displays summary report (including size statistics and compression ratio)
   - Provides manual download link (for scenarios where automatic download fails)


### III. Error Handling and Optimization

1. **Error Handling Mechanism**
   - Individual resource download failure: logs and skips (continues processing other files)
   - Critical file (e.g., `blazor.boot.json`) failure: terminates process and displays error
   - Cleans up temporary resource links when page unloads (`URL.revokeObjectURL`)

2. **Performance Optimization Details**
   - Uses streaming compression for large files to reduce memory usage
   - Processes Base64 encoding in chunks to avoid excessively long lines
   - Adapts to .NET 8/9 runtime versions
   - Preserves key syntax structures in custom JS minification (template strings, regular expressions)


---

# Blazor WASM 程序打包器

## 简介

Blazor WASM 程序打包器是一款专为 Blazor WebAssembly 应用设计的资源整合工具。它能够将应用运行所需的核心资源（包括框架文件、业务程序集、配置文件及自定义脚本）压缩并合并为单个 JavaScript 文件，大幅简化部署流程并优化应用加载性能。

该工具通过浏览器端直接运行的 HTML 页面实现功能，无需后端服务支持，只需将 HTML 文件放置在发布后的应用根目录即可使用。

## 带来的好处

1. **简化部署流程**
   - 将数十甚至上百个分散的资源文件合并为单个 JS 文件，减少部署文件数量
   - 避免因文件遗漏导致的应用运行异常
   - 降低多文件传输时的网络请求开销

2. **提升加载性能**
   - 对所有资源进行最大级别 GZIP 压缩，显著减小传输体积
   - 减少 HTTP 请求次数，降低网络延迟影响
   - 支持自定义脚本精简，移除冗余代码

3. **增强部署灵活性**
   - 生成的单一文件可轻松集成到各种静态资源托管服务
   - 支持微前端架构（MicroApp/ QianKun）的环境适配
   - 便于版本管理和缓存策略实施

4. **突破跨域与本地运行限制**
   - 解决传统 Blazor WASM 应用的跨域资源调用问题（CORS），通过资源内嵌避免跨域请求
   - 支持直接从本地文件系统启动应用（`file://` 协议），无需依赖 Web 服务器
   - 适合离线场景或本地演示环境，无需配置服务器跨域策略

5. **使用便捷性**
   - 纯前端实现，无需安装额外工具或依赖
   - 提供可视化进度反馈和详细处理日志
   - 支持自定义输出文件名和脚本合并规则


## 代码执行过程

### 一、初始化阶段

1. **页面加载准备**
   - 初始化 DOM 元素引用（进度条、日志区、按钮等交互组件）
   - 重置进度条状态为"准备就绪"（0%）
   - 清空日志输出区域
   - 禁用下载按钮防止重复点击

2. **用户参数收集**
   - 读取输出文件名输入框内容（为空时将使用默认命名规则）
   - 解析自定义 JS 文件列表（每行一个路径，相对 `_framework` 目录）
   - 检查"精简自定义脚本"复选框状态（决定是否移除注释和冗余空格）


### 二、核心处理流程

#### 步骤1：浏览器兼容性验证
- 检测当前浏览器是否支持 `CompressionStream` API（用于 GZIP 压缩）
- 不支持时终止流程并提示："请使用最新版 Chrome、Edge 或 Firefox 浏览器"


#### 步骤2：解析应用配置文件
1. **下载配置文件**
   - 发送请求获取 `_framework/blazor.boot.json`（Blazor 应用的核心配置）
   - 记录配置文件原始大小并计入总统计
   - 下载失败（如 404 错误）时终止流程，日志显示具体错误信息

2. **解析配置内容**
   - 解析 JSON 格式的配置数据
   - 提取 `resources` 字段中所有资源路径（包括 runtime、assembly、jsModule 等类型）
   - 提取 `entryAssembly` 值（用于生成默认输出文件名）
   - 进度条更新至 10%，日志显示"配置文件解析完成"


#### 步骤3：处理核心资源文件
1. **任务准备**
   - 收集需处理的资源文件列表（包含 `blazor.webassembly.js` 等核心文件）
   - 根据文件数量计算单个文件的进度占比（总进度 75% 平分）

2. **逐个处理资源**
   - 循环处理每个资源文件：
     1. 构造完整 URL：`_framework/[文件名]`
     2. 下载文件并转换为 Uint8Array 格式
     3. 记录原始文件大小（累加至总原始大小）
     4. 使用 `gzipCompress()` 函数进行最大级别（level:9）压缩
     5. 将压缩后的数据转换为 Base64 编码（按 129 字节分块处理）
     6. 生成包含文件名和 Base64 内容的条目
   - 实时更新进度条（如"处理 3/20：dotnet.wasm"）
   - 单个文件处理失败时记录日志并跳过（不终止整体流程）


#### 步骤4：处理自定义JS文件（可选）
1. **输入解析**
   - 过滤空行和无效路径，获取有效自定义 JS 文件列表
   - 列表为空时日志显示"未指定自定义 JS 文件，不生成 Merge.js"并跳过

2. **合并与处理**
   - 逐个下载指定的自定义 JS 文件：
     - 记录每个文件原始大小
     - 启用精简时使用 `minifyJs()` 处理（保留模板字符串和正则表达式）
     - 合并内容（添加来源注释，除非启用精简）
   - 对合并内容进行 GZIP 压缩并转换为 Base64 编码
   - 生成 `Merge.js` 条目并添加到资源列表


#### 步骤5：生成输出文件
1. **内容构建**
   - 首行添加生成时间注释（格式：`// Generated on: YYYY-MM-DD HH:MM:SS`）
   - 创建包含所有资源的 JavaScript 对象（键为文件名，值为 Base64 内容）
   - 生成自执行函数，包含：
     - 资源解压逻辑（使用 `DecompressionStream` 处理 GZIP 数据）
     - Blazor 启动配置（覆盖 `loadBootResource` 方法注入本地资源）
     - 临时 URL 清理逻辑（释放内存占用）

2. **文件名处理**
   - 优先使用用户指定文件名（需包含 `.js` 扩展名）
   - 未指定时使用规则：`[entryAssembly].published.js`
   - 无 `entryAssembly` 时默认使用 `blazor-resources.published.js`


#### 步骤6：生成下载链接
1. **统计信息计算**
   - 原始总大小：所有资源+配置文件+自定义脚本的原始大小总和
   - 生成文件大小：最终 JS 文件的 UTF-8 编码字节数
   - 压缩比率：`(生成文件大小 / 原始总大小) × 100%`

2. **创建下载**
   - 将 JS 内容转换为 Blob 对象（MIME 类型：`application/javascript`）
   - 生成临时下载 URL（`URL.createObjectURL`）
   - 自动触发下载并显示总结报告（含大小统计和压缩比率）
   - 提供手动下载链接（应对自动下载失败场景）


### 三、异常处理与优化

1. **错误处理机制**
   - 单个资源下载失败：记录日志并跳过（继续处理其他文件）
   - 关键文件（如 `blazor.boot.json`）失败：终止流程并显示错误
   - 页面卸载时清理临时资源链接（`URL.revokeObjectURL`）

2. **性能优化细节**
   - 大文件采用流式压缩，降低内存占用
   - Base64 编码分块处理，避免单行过长
   - 针对 .NET 8/9 版本的 runtime 做适配处理
   - 自定义 JS 精简保留关键语法结构（模板字符串、正则表达式）
