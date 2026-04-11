# MarkItDown — AI 上下文文档

> 最后更新：2026-04-11
> 文档版本：1.0.0
> 项目状态：✅ 活跃维护

---

## 项目概览

**MarkItDown** 是由 Microsoft AutoGen Team 开发的轻量级 Python 文档转换工具，专门用于将各种文件格式转换为 Markdown，以供大语言模型（LLM）和文本分析管道使用。

### 核心价值
- 🎯 **LLM 优化**：输出格式专为 LLM 消费设计，保留关键文档结构
- 🔌 **MCP 集成**：提供 Model Context Protocol 服务器，可直接集成到 Claude Desktop 等 LLM 应用
- 🧩 **插件系统**：支持第三方扩展，易于定制
- 📊 **格式丰富**：支持 20+ 种文件格式转换

---

## 技术栈

### 核心技术
```yaml
语言: Python 3.10+
构建工具: Hatchling
包管理: pip / uv / conda
容器: Docker
测试框架: hatch
```

### 主要依赖

#### 核心依赖（必需）
```python
beautifulsoup4      # HTML 解析
requests            # HTTP 请求
markdownify         # HTML 转 Markdown
magika~=0.6.1       # 文件类型检测
charset-normalizer  # 字符编码检测
defusedxml          # 安全 XML 解析
```

#### 可选依赖（按需安装）
```bash
# Office 文档
[pptx]   python-pptx           # PowerPoint
[docx]   mammoth~=1.11.0       # Word
[xlsx]   pandas + openpyxl     # Excel (新格式)
[xls]    pandas + xlrd         # Excel (旧格式)

# PDF 处理
[pdf]    pdfminer.six>=20251230 + pdfplumber>=0.11.9

# 音频/视频
[outlook]              olefile                   # Outlook 邮件
[audio-transcription]  pydub + SpeechRecognition # 音频转文字
[youtube-transcription] youtube-transcript-api   # YouTube 字幕

# 云服务
[az-doc-intel]  azure-ai-documentintelligence + azure-identity
```

---

## 支持的文件格式

| 类别 | 支持格式 | 转换方式 |
|------|----------|----------|
| **文档** | PDF, DOCX, PPTX, XLSX, XLS | 内置解析器 |
| **网页** | HTML | markdownify |
| **数据** | CSV, JSON, XML | 文本解析 |
| **图片** | JPG, PNG, GIF, BMP, etc. | EXIF + OCR/LLM 描述 |
| **音频** | WAV, MP3 | EXIF + 语音识别 |
| **压缩** | ZIP | 递归解析内容 |
| **电子书** | EPUB | 文本提取 |
| **视频** | YouTube URL | 字幕提取 |

---

## 项目结构

```
markitdown/
├── packages/
│   ├── markitdown/              # 主包
│   │   └── src/markitdown/
│   │       ├── __main__.py      # CLI 入口
│   │       ├── __about__.py     # 版本信息
│   │       ├── _markitdown.py   # 核心逻辑
│   │       └── converters/      # 格式转换器
│   │
│   ├── markitdown-mcp/          # MCP 服务器
│   │   └── src/markitdown_mcp/
│   │       └── server.py        # MCP 协议实现
│   │
│   ├── markitdown-ocr/          # OCR 插件
│   │   └── src/markitdown_ocr/
│   │
│   └── markitdown-sample-plugin/ # 插件示例
│
├── README.md
└── CLAUDE.md
```

---

## 使用方式

### 1. 命令行使用

```bash
# 基础转换
markitdown path-to-file.pdf > document.md

# 指定输出文件
markitdown path-to-file.pdf -o document.md

# 管道输入
cat path-to-file.pdf | markitdown

# 列出插件
markitdown --list-plugins

# 启用插件
markitdown --use-plugins path-to-file.pdf
```

### 2. Python API

```python
from markitdown import MarkItDown

# 基础使用
md = MarkItDown(enable_plugins=False)
result = md.convert("test.xlsx")
print(result.text_content)

# Azure Document Intelligence
md = MarkItDown(docintel_endpoint="<endpoint>")
result = md.convert("test.pdf")

# LLM 图像描述
from openai import OpenAI
client = OpenAI()
md = MarkItDown(
    llm_client=client,
    llm_model="gpt-4o",
    llm_prompt="Describe this image"
)
result = md.convert("example.jpg")
```

### 3. Docker 使用

```bash
docker build -t markitdown:latest .
docker run --rm -i markitdown:latest < input.pdf > output.md
```

---

## 核心功能

### 1. 文档结构保留

MarkItDown 在转换过程中尽可能保留原始文档的结构：
- ✅ 标题层级（H1-H6）
- ✅ 列表（有序/无序）
- ✅ 表格（包括合并单元格）
- ✅ 链接和引用
- ✅ 粗体/斜体等格式

### 2. MCP 服务器

```bash
# 安装
pip install 'markitdown-mcp[all]'

# 配置 Claude Desktop
# 在 claude_desktop_config.json 中添加：
{
  "mcpServers": {
    "markitdown": {
      "command": "uvx",
      "args": ["markitdown-mcp"]
    }
  }
}
```

### 3. 插件系统

```python
# 安装 OCR 插件
pip install markitdown-ocr
pip install openai  # 或其他 OpenAI 兼容客户端

# 使用
from markitdown import MarkItDown
from openai import OpenAI

md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),
    llm_model="gpt-4o",
)
result = md.convert("document_with_images.pdf")
print(result.text_content)
```

---

## 开发工作流

### 环境设置

```bash
# 克隆仓库
git clone git@github.com:microsoft/markitdown.git
cd markitdown

# 使用虚拟环境
python -m venv .venv
source .venv/bin/activate

# 安装（开发模式）
pip install -e 'packages/markitdown[all]'
```

### 测试

```bash
# 安装 hatch
pip install hatch

# 进入包目录
cd packages/markitdown

# 运行测试
hatch shell
hatch test

# 运行 pre-commit 检查
pre-commit run --all-files
```

### 贡献指南

1. 查看 [Issues](https://github.com/microsoft/markitdown/issues)
2. 寻找标记为 "open for contribution" 的问题
3. 提交 PR 前运行测试和 pre-commit 检查
4. 遵循 [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/)

---

## 最新更新（2026-04）

### 重要变更
- ✅ **修复 PDF 内存增长**：通过调用 `page.close()` 防止 O(n) 内存增长
- ✅ **新增 OCR 层服务**：支持嵌入图像和 PDF 扫描的 OCR
- ✅ **扩展表格支持**：改进宽表格支持
- ✅ **PDF 列表修复**：支持部分编号列表
- ✅ **更新 PDF 表格提取**：支持对齐的 Markdown

### 版本信息
- 当前版本：基于上游 main 分支
- Python 支持：3.10, 3.11, 3.12, 3.13
- 状态：Beta 阶段

---

## 常见问题

### Q: 与 textract 的区别？
**A:** MarkItDown 专注于保留文档结构并输出 Markdown，更适合 LLM 消费。textract 更适合通用文本提取。

### Q: 为什么选择 Markdown？
**A:**
1. LLM（如 GPT-4o）原生"理解"Markdown
2. 训练数据中 Markdown 占比大
3. Token 效率高
4. 接近纯文本，易于处理

### Q: 如何支持新格式？
**A:** 可以创建插件或直接贡献到主仓库。参考 `markitdown-sample-plugin`。

---

## 相关资源

- **GitHub**: https://github.com/microsoft/markitdown
- **PyPI**: https://pypi.org/project/markitdown/
- **MCP 服务器**: https://github.com/microsoft/markitdown/tree/main/packages/markitdown-mcp
- **OCR 插件**: https://github.com/microsoft/markitdown/tree/main/packages/markitdown-ocr
- **Issues**: https://github.com/microsoft/markitdown/issues
- **PRs**: https://github.com/microsoft/markitdown/pulls

---

## 本仓库信息

```yaml
上游仓库: git@github.com:microsoft/markitdown.git
分支: main
最后同步: 2026-04-11
个人分支: git@github.com:afanty2021/markitdown.git
```

---

*本文档由 AI 生成，最后更新于 2026-04-11*
