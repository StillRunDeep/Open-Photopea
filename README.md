# Open Photopea

Photopea is the best Photoshop alternative. Their program runs as a web app. Photopea is capable of running **PSD, XCF and Sketch** files.

Photopea is a closed source software. So I decide to extract the code and Open Source it, but if they ask me to remove it I will be 100% fine with their decision.

## 使用方法 / Usage

### 方式一：直接打开（推荐）

直接用浏览器打开 `index.html` 文件即可使用。

```bash
# macOS
open index.html

# Windows
start index.html

# Linux
xdg-open index.html
```

### 方式二：本地服务器

如果需要完整功能（如 Service Worker 等），建议通过本地服务器运行：

```bash
# 使用 Python 3
python3 -m http.server 8080

# 使用 Node.js (需要先安装 http-server)
npx http-server -p 8080

# 使用 PHP
php -S localhost:8080
```

然后在浏览器中打开 `http://localhost:8080`。

### 功能特性

- 支持打开和编辑 **PSD**、**XCF**、**Sketch**、**XD**、**CDR** 等格式文件
- 创建新图像或从电脑打开现有文件
- 保存为 PSD 格式（文件 - 另存为 PSD）或导出为 JPG / PNG / SVG（文件 - 导出）
- 高级图像编辑功能：图层、选区、滤镜、调整等

### 字体资源

项目已内置一组常用开源字体，方便文字工具直接显示中英文内容：

- 中文黑体：Noto Sans CJK SC Regular / Bold
- 中文宋体：Noto Serif CJK SC Regular / Bold
- 英文无衬线：Roboto Regular / Bold / Italic / BoldItalic
- 常见英文替代：Liberation Sans / Serif / Mono

字体文件位于 `rsrc/fonts/custom/` 和 `rsrc/fonts/fs/`。中文字体和常见中文系统字体名（如微软雅黑、宋体、苹方、黑体）会映射到 Noto CJK 字体；常见英文字体名会映射到 Roboto 或 Liberation 系列。

如果打开 PSD 时仍提示缺失字体，表示该字体不在当前内置集合中，程序会使用相近字体替代。新增字体时需要同时放入字体文件、更新 `code/FNTS.js`，必要时更新 `code/pp.js` 中的字体替代映射。字体来源和授权记录见 `rsrc/fonts/custom/README.md`。

### 项目结构

```
├── index.html          # 主页面入口
├── code/               # 核心 JS 代码
│   ├── pp.js           # 主程序逻辑
│   ├── PIMG.js         # 图像处理模块
│   ├── FNTS.js         # 字体模块
│   ├── LNG2.js         # 语言/国际化模块
│   └── external/       # 外部依赖
├── style/              # 样式文件
├── promo/              # 推广图片资源
├── learn/              # 学习教程
├── api/                # API 相关页面
└── img/                # 图片资源
```
