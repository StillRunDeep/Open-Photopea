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
