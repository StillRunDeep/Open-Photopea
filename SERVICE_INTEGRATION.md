# 将 Open-Photopea 作为网页图片编辑服务调用

当前项目已经通过 GitHub Pages 发布，可作为一个独立的在线图片编辑器嵌入到其它网页中使用。

线上地址：

```text
https://ps.ethanmiao.cn/
```

典型目标流程：

```text
业务网页选择 / 获取图片
  ↓
通过 iframe 打开 https://ps.ethanmiao.cn/
  ↓
通过 postMessage 将图片传入编辑器
  ↓
用户在编辑器中编辑图片
  ↓
业务网页触发导出
  ↓
编辑器通过 postMessage 回传编辑结果
  ↓
业务网页保存 PSD / PNG / JPG 等文件
```

## 推荐方案

推荐使用 **iframe + postMessage + saveToOE()**。

这种方式不要求图片必须是公开 URL，也不依赖用户点击 Photopea 内部的 `File -> Save` 菜单。业务网页可以完全控制：

- 何时传入图片；
- 何时导出；
- 导出为什么格式；
- 回传后如何保存到业务系统。

## 基础接入方式

在业务网页中嵌入当前项目：

```html
<iframe
  id="photopea"
  src="https://ps.ethanmiao.cn/"
  style="width: 100%; height: 720px; border: 1px solid #ddd;"
></iframe>
```

然后通过 `postMessage` 和 iframe 内的编辑器通信。

## 完整示例

下面示例实现了：

1. 用户选择一张图片；
2. 将图片传入 `https://ps.ethanmiao.cn/`；
3. 用户编辑；
4. 点击“导出 PNG 预览”；
5. 点击“保存 PSD 源文件”；
6. 外层网页接收结果并可继续上传到服务器。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="utf-8" />
  <title>调用 Open-Photopea 编辑图片</title>
  <style>
    body {
      margin: 0;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    }

    .toolbar {
      display: flex;
      gap: 12px;
      align-items: center;
      padding: 12px;
      border-bottom: 1px solid #ddd;
    }

    #photopea {
      width: 100%;
      height: 720px;
      border: 0;
      display: block;
    }

    #preview {
      max-width: 320px;
      max-height: 240px;
      display: block;
      margin: 12px;
      border: 1px solid #ddd;
    }
  </style>
</head>
<body>
  <div class="toolbar">
    <input id="file" type="file" accept="image/*,.psd" />
    <button id="exportPng">导出 PNG 预览</button>
    <button id="exportPsd">保存 PSD 源文件</button>
  </div>

  <iframe
    id="photopea"
    src="https://ps.ethanmiao.cn/"
  ></iframe>

  <img id="preview" alt="导出预览" />

  <script>
    const photopeaOrigin = "https://ps.ethanmiao.cn";

    const iframe = document.getElementById("photopea");
    const fileInput = document.getElementById("file");
    const exportPngButton = document.getElementById("exportPng");
    const exportPsdButton = document.getElementById("exportPsd");
    const preview = document.getElementById("preview");

    let photopeaReady = false;
    let pendingExportFormat = null;

    window.addEventListener("message", async (event) => {
      // 生产环境必须校验来源，避免接收其它窗口伪造的数据。
      if (event.origin !== photopeaOrigin) return;
      if (event.source !== iframe.contentWindow) return;

      // Photopea 初始化完成、或某条命令执行完成后，会发送字符串 "done"。
      if (event.data === "done") {
        photopeaReady = true;
        console.log("Photopea ready / command done");
        return;
      }

      // saveToOE() 导出的文件会以 ArrayBuffer 形式回传。
      if (event.data instanceof ArrayBuffer) {
        const format = pendingExportFormat || "bin";
        pendingExportFormat = null;

        let mime = "application/octet-stream";
        let filename = `edited.${format}`;

        if (format === "png") {
          mime = "image/png";
          filename = "edited.png";
        }

        if (format === "jpg" || format === "jpeg") {
          mime = "image/jpeg";
          filename = "edited.jpg";
        }

        if (format === "psd") {
          mime = "image/vnd.adobe.photoshop";
          filename = "edited.psd";
        }

        const blob = new Blob([event.data], { type: mime });

        if (format === "png" || format === "jpg" || format === "jpeg") {
          preview.src = URL.createObjectURL(blob);
        }

        // 上传到业务服务端的示例：
        const formData = new FormData();
        formData.append("file", blob, filename);
        formData.append("format", format);

        // await fetch("/api/images/edited", {
        //   method: "POST",
        //   body: formData
        // });

        console.log("收到编辑结果", filename, blob);
      }
    });

    fileInput.addEventListener("change", async () => {
      const file = fileInput.files[0];
      if (!file) return;

      const buffer = await file.arrayBuffer();

      // 将图片 / PSD 二进制文件传入 Photopea。
      iframe.contentWindow.postMessage(buffer, photopeaOrigin);
    });

    exportPngButton.addEventListener("click", () => {
      pendingExportFormat = "png";

      iframe.contentWindow.postMessage(
        'app.activeDocument.saveToOE("png");',
        photopeaOrigin
      );
    });

    exportPsdButton.addEventListener("click", () => {
      pendingExportFormat = "psd";

      // 推荐保存为 PSD，保留图层，方便后续继续编辑。
      // psd:true 会生成更小的 PSD，但导出时间会更长。
      iframe.contentWindow.postMessage(
        'app.activeDocument.saveToOE("psd:true");',
        photopeaOrigin
      );
    });
  </script>
</body>
</html>
```

## 使用流程

### 1. 业务网页加载编辑器

业务网页通过 iframe 加载：

```text
https://ps.ethanmiao.cn/
```

示例：

```html
<iframe id="photopea" src="https://ps.ethanmiao.cn/"></iframe>
```

### 2. 等待编辑器就绪

编辑器初始化完成后，会向外层网页发送：

```text
done
```

外层网页监听：

```js
window.addEventListener("message", (event) => {
  if (event.origin !== "https://ps.ethanmiao.cn") return;

  if (event.data === "done") {
    console.log("编辑器已就绪");
  }
});
```

### 3. 传入图片

如果图片来自用户本地选择：

```js
const file = fileInput.files[0];
const buffer = await file.arrayBuffer();

iframe.contentWindow.postMessage(
  buffer,
  "https://ps.ethanmiao.cn"
);
```

如果图片来自业务系统接口：

```js
const response = await fetch("/api/images/source/123");
const blob = await response.blob();
const buffer = await blob.arrayBuffer();

iframe.contentWindow.postMessage(
  buffer,
  "https://ps.ethanmiao.cn"
);
```

### 4. 用户编辑图片

用户在 iframe 中使用 Photopea 编辑图片。

业务系统可以把 `https://ps.ethanmiao.cn/` 当作一个在线编辑器组件。

### 5. 导出图片

导出 PNG：

```js
iframe.contentWindow.postMessage(
  'app.activeDocument.saveToOE("png");',
  "https://ps.ethanmiao.cn"
);
```

导出 JPG，质量 0.9：

```js
iframe.contentWindow.postMessage(
  'app.activeDocument.saveToOE("jpg:0.9");',
  "https://ps.ethanmiao.cn"
);
```

导出 PSD，保留图层：

```js
iframe.contentWindow.postMessage(
  'app.activeDocument.saveToOE("psd:true");',
  "https://ps.ethanmiao.cn"
);
```

### 6. 接收回传文件

```js
window.addEventListener("message", async (event) => {
  if (event.origin !== "https://ps.ethanmiao.cn") return;

  if (event.data instanceof ArrayBuffer) {
    const blob = new Blob([event.data], {
      type: "image/png"
    });

    const formData = new FormData();
    formData.append("file", blob, "edited.png");

    await fetch("/api/images/edited", {
      method: "POST",
      body: formData
    });
  }
});
```

## 推荐保存格式

为了方便持续编辑，建议保存 **带图层的 PSD 格式**。

推荐策略：

```text
PSD：作为源文件保存，保留图层，方便下次继续编辑。
PNG / JPG：作为预览图、发布图、缩略图或业务展示图。
```

也就是说，业务系统中最好保存两类文件：

| 文件 | 用途 |
| --- | --- |
| `edited.psd` | 源文件，保留图层，后续继续编辑 |
| `edited.png` 或 `edited.jpg` | 预览、展示、下载、发布 |

推荐导出 PSD：

```js
iframe.contentWindow.postMessage(
  'app.activeDocument.saveToOE("psd:true");',
  "https://ps.ethanmiao.cn"
);
```

其中：

```text
psd:true
```

表示生成更小的 PSD 文件，适合网络传输和长期保存。代价是导出会更慢。

如果需要最高兼容性，也可以使用：

```js
iframe.contentWindow.postMessage(
  'app.activeDocument.saveToOE("psd");',
  "https://ps.ethanmiao.cn"
);
```

## 自动回传方案

### 方案 A：用户点击业务网页按钮后回传

推荐。

```js
saveButton.addEventListener("click", () => {
  iframe.contentWindow.postMessage(
    'app.activeDocument.saveToOE("psd:true");',
    "https://ps.ethanmiao.cn"
  );
});
```

适合：

- 用户明确点击“保存”；
- 文件较大；
- 需要减少无效导出；
- 需要同时保存 PSD 源文件和 PNG 预览图。

### 方案 B：定时自动保存

可以每隔一段时间自动导出 PSD：

```js
setInterval(() => {
  iframe.contentWindow.postMessage(
    'app.activeDocument.saveToOE("psd:true");',
    "https://ps.ethanmiao.cn"
  );
}, 60000);
```

不建议间隔太短。PSD 文件较大，频繁导出会影响性能。

### 方案 C：离开页面前提示保存

可以在用户关闭页面前提示：

```js
window.addEventListener("beforeunload", (event) => {
  event.preventDefault();
  event.returnValue = "图片可能还没有保存，确定离开吗？";
});
```

浏览器通常不允许在 `beforeunload` 中可靠执行异步上传，所以不要依赖这个事件做最终保存。

## 通过 URL 自动打开图片

如果图片已经有可公开访问的 URL，也可以通过 URL hash 初始化：

```js
const config = {
  files: [
    "https://example.com/uploads/source.png"
  ],
  environment: {
    showbranding: false
  }
};

const editorUrl =
  "https://ps.ethanmiao.cn/#" +
  encodeURI(JSON.stringify(config));

iframe.src = editorUrl;
```

注意：图片服务器必须允许跨域访问：

```http
Access-Control-Allow-Origin: https://ps.ethanmiao.cn
```

或者：

```http
Access-Control-Allow-Origin: *
```

如果图片需要鉴权，推荐由业务网页自己 `fetch` 图片，再用 `ArrayBuffer` 传给 iframe。

## 服务端保存接口示例

前端拿到 `Blob` 后，可以用 `FormData` 上传：

```js
const formData = new FormData();
formData.append("file", blob, "edited.psd");
formData.append("sourceImageId", "123");
formData.append("format", "psd");

await fetch("/api/images/edited", {
  method: "POST",
  body: formData
});
```

服务端需要保存：

- 文件二进制；
- 文件格式；
- 原始图片 ID；
- 用户 ID；
- 编辑版本号；
- 预览图 URL；
- PSD 源文件 URL。

推荐数据结构：

```json
{
  "id": "edit_123",
  "sourceImageId": "img_123",
  "sourceFileUrl": "https://cdn.example.com/images/img_123.png",
  "psdFileUrl": "https://cdn.example.com/edits/edit_123.psd",
  "previewFileUrl": "https://cdn.example.com/edits/edit_123.png",
  "createdBy": "user_123",
  "createdAt": "2026-07-16T00:00:00Z"
}
```

## 安全注意事项

### 1. 校验 message 来源

不要直接接收所有窗口消息。

推荐：

```js
if (event.origin !== "https://ps.ethanmiao.cn") return;
if (event.source !== iframe.contentWindow) return;
```

发送消息时也不要使用 `*`，推荐明确目标域名：

```js
iframe.contentWindow.postMessage(buffer, "https://ps.ethanmiao.cn");
```

### 2. 控制上传文件大小

图片和 PSD 可能很大。建议业务系统限制：

- 原图最大体积；
- 导出 PSD 最大体积；
- 上传超时时间；
- 用户存储配额。

### 3. 保存源文件和预览图

不要只保存 PNG / JPG。这样会丢失图层，后续无法继续编辑。

推荐：

```text
必须保存：PSD
按需保存：PNG / JPG / WEBP
```

### 4. 文件版本管理

建议每次保存生成一个新版本，而不是直接覆盖旧文件。

例如：

```text
edit_123_v1.psd
edit_123_v2.psd
edit_123_v3.psd
```

这样可以支持撤回、历史版本和恢复。

## 推荐产品交互

业务网页可以提供这些按钮：

```text
选择图片
保存源文件 PSD
导出预览 PNG
完成并返回业务系统
```

推荐流程：

```text
用户打开业务编辑页
  ↓
系统加载 https://ps.ethanmiao.cn/
  ↓
用户选择或系统传入图片
  ↓
用户编辑
  ↓
点击“保存”
  ↓
先导出 PSD 保存源文件
  ↓
再导出 PNG 保存预览图
  ↓
业务系统记录两个文件地址
```

## 一句话总结

将当前项目作为服务调用时，推荐通过 iframe 嵌入 `https://ps.ethanmiao.cn/`，用 `postMessage` 传入图片，用 `app.activeDocument.saveToOE()` 导出结果；保存时优先保存 `psd:true` 作为带图层源文件，同时保存 PNG / JPG 作为预览或发布图。
