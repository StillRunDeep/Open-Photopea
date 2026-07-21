# 补充中英常用开源字体设计

日期：2026-07-21

## 背景

Open-Photopea 当前通过 `code/FNTS.js` 维护字体清单，通过 `code/pp.js` 按 `rsrc/fonts/<字体路径>` 加载字体资源。仓库目前只包含 `rsrc/fonts/fonts.png`，缺少实际字体文件，导致文字工具在选择或回放字体时容易请求 404。中文字体尤其缺失，因此中文输入或打开含中文文字图层时可能出现空白、方块或 fallback 不稳定。

## 目标

- 补充一组中英常用、开源可再分发的字体。
- 优先解决中文文字工具显示问题。
- 兼顾常见英文无衬线、衬线、等宽字体场景。
- 保持仓库体积和维护成本可控。
- 不尝试一次性补完整 Photopea 原字体库。

## 非目标

- 不补齐 `code/FNTS.js` 中全部 `fs/...`、`gf/...` 字体。
- 不提交商业字体或系统专有字体。
- 不重写文字渲染引擎。
- 不改变现有 PSD / SVG / 导出流程，只补充字体资源和映射。

## 字体集合

首批内置字体如下：

| 用途 | 字体 | 样式 |
| --- | --- | --- |
| 中文黑体 | Noto Sans SC | Regular, Bold |
| 中文宋体 | Noto Serif SC | Regular, Bold |
| 英文无衬线 | Roboto | Regular, Bold, Italic, BoldItalic |
| 英文衬线 | Liberation Serif | Regular, Bold, Italic, BoldItalic |
| 英文等宽 | Liberation Mono | Regular, Bold, Italic, BoldItalic |

中文字体优先保证 Regular 和 Bold；不加入 Light、Medium、Black 等额外字重，以控制仓库体积和首次加载成本。

## 文件结构

字体放在当前加载器已经使用的 `rsrc/fonts/` 下，使用 `custom/` 命名空间，避免和原始 `fs/`、`gf/` 清单混淆：

```text
rsrc/fonts/
  custom/
    noto-sans-sc/
      0
      1
    noto-serif-sc/
      0
      1
    roboto/
      0
      1
      2
      3
    liberation-serif/
      0
      1
      2
      3
    liberation-mono/
      0
      1
      2
      3
```

这些文件内容为实际 `.ttf` / `.otf` 字体二进制，但文件名不带扩展名，以匹配现有加载代码的路径约定。

## 代码改动

### `code/FNTS.js`

在 `FNTS.list` 末尾追加字体记录。每条记录继续沿用现有结构：

```js
[
  familyName,
  styleName,
  postScriptName,
  subsetMask,
  categoryIndex,
  resourcePath
]
```

示例：

```js
[
  "Noto Sans SC",
  "Regular",
  "NotoSansSC-Regular",
  257,
  12,
  "custom/noto-sans-sc/0"
]
```

字符集标记按现有 `FNTS.subsetNames` 的位图规则处理：

- `Latin-1` 为 `1`；
- `Chi-Jap-Kor` 为 `256`；
- 中文字体使用 `257`，表示同时覆盖 `Latin-1` 和 `Chi-Jap-Kor`；
- 英文字体使用 `1` 或包含扩展拉丁的组合值。

分类按现有 `FNTS.cats` 使用：

- 无衬线：`Sans Serif`，索引 `12`；
- 衬线：`Serif`，索引 `14`；
- 等宽：`Monospaced`，索引 `7`。

### `code/pp.js`

保留现有加载流程，只在字体替代映射中补充常见名称到开源字体的映射。重点映射：

```text
MicrosoftYaHei / Microsoft YaHei / 微软雅黑 -> NotoSansSC-Regular
SimSun / 宋体 -> NotoSerifSC-Regular
PingFangSC-Regular / PingFang SC -> NotoSansSC-Regular
HeitiSC / STHeiti -> NotoSansSC-Regular
ArialMT / Arial -> Roboto-Regular
TimesNewRomanPSMT / Times New Roman -> LiberationSerif-Regular
CourierNewPSMT / Courier New -> LiberationMono-Regular
```

由于 `pp.js` 是压缩后的代码，改动要尽量小，只修改 `rr.prototype.Dx` 映射对象，避免影响其它渲染逻辑。

## 加载流程

1. 用户创建或打开文字图层。
2. 文字样式引用某个 PostScriptName。
3. `rr.prototype.jq(fontName)` 查询 `FNTS.map`。
4. 命中后读取 `resourcePath`。
5. 加载器请求 `rsrc/fonts/<resourcePath>`。
6. 字体解析成功后进入 `this.FQ[postScriptName]` 缓存。
7. 文字工具使用该字体绘制文本。

## 错误处理

- 如果字体文件缺失，保留当前 fallback 逻辑，不让应用崩溃。
- 如果某个 PSD 引用了未内置字体，继续映射到最接近的常用开源字体。
- 如果中文字体下载失败，浏览器 Network 会出现明确的 404，可通过文档说明排查。
- 不隐藏现有字体缺失提示，避免用户以为原字体已经完整安装。

## 文档更新

更新 `README.md`，增加“字体资源”说明：

- 本项目内置一组开源中英文字体；
- 字体文件位于 `rsrc/fonts/custom/`；
- 新增字体时必须同时更新 `code/FNTS.js`；
- 若打开 PSD 时提示缺失字体，说明该字体不在当前内置集合中，会使用替代字体。

增加 `rsrc/fonts/custom/README.md`，记录每个字体的来源、授权和下载地址。

## 验证计划

手工验证：

1. 打开 `index.html` 或本地服务器页面。
2. 新建图像。
3. 使用文字工具输入中文：`你好，Open Photopea`。
4. 使用文字工具输入英文：`Hello Open Photopea`。
5. 分别切换 Noto Sans SC、Noto Serif SC、Roboto、Liberation Serif、Liberation Mono。
6. 打开浏览器 Network 面板，确认新增字体请求返回 200，不再 404。
7. 导出 PNG，确认中英文都能正常显示。

回归验证：

1. 打开普通图片并编辑。
2. 打开无文字图层的文件，确认启动不受字体资源影响。
3. 打开含常见英文字体的 PSD，确认 Arial / Times / Courier 类字体有合理替代。

## 风险与取舍

- 中文 CJK 字体体积较大，可能增加仓库和首次加载成本；因此首批只放 Regular / Bold。
- `pp.js` 是压缩文件，直接编辑可维护性较差；本次只做最小映射变更。
- 字体名称、PostScriptName 和实际字体内部名称必须一致或可被解析，否则加载后可能无法命中预期名称。
- 开源字体仍需保留授权说明，避免后续混入不可分发字体。

## 成功标准

- `rsrc/fonts/custom/` 下存在首批字体文件。
- `code/FNTS.js` 能列出新增字体。
- 常见中文文本可在文字工具中正常显示。
- 常见英文衬线、无衬线、等宽文本可正常显示。
- 浏览器 Network 中新增字体资源返回 200。
- README 说明字体来源和后续扩展方式。
