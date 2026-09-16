---
name: figma-to-html
description: >
  将 Figma 设计稿转换为高保真 HTML 页面。使用 Figma MCP 工具获取设计数据和图片资源，
  结合用户提供的参考截图，生成响应式 HTML 文件。
  触发条件：用户提到「figma 转 html」「figma 还原」「figma to html」「设计稿转页面」
  「还原设计稿」「figma 切图」或提供 Figma 链接并要求生成 HTML 时使用。
---

# Figma to HTML

将 Figma 设计稿转换为高保真响应式 HTML 页面。

## 用户需提供

1. **Figma 地址** — 带 `node-id` 参数的完整 URL（如 `figma.com/design/<fileKey>/...?node-id=428-2244`）
2. **参考截图** — 用户上传的目标页面截图（用于视觉校准）
3. **需要下载的图片 node-id 列表**（可选）— 指定需要导出为本地图片的节点 ID

## 工作流程

按以下 4 个阶段顺序执行，每阶段完成后向用户确认再继续。

### 阶段 1：获取 Figma 数据

1. 从用户提供的 URL 中解析 `fileKey` 和 `nodeId`
   - URL 格式：`figma.com/(file|design)/<fileKey>/...?node-id=<nodeId>`
   - `nodeId` 中的 `-` 替换为 `:`（如 `428-2244` → `428:2244`）
2. 调用 `Figma-AI-Bridge__get_figma_data` 获取设计数据：
   ```
   fileKey: 从 URL 解析
   nodeId: 从 URL 解析（格式 1234:5678）
   ```
   - **已知问题**：传入 frame 节点时，本工具可能只返回根节点，`children` 全部丢失，且 `type` 被误报为 `IMAGE-SVG`；显式传 `depth` 也无法解决。传入叶子节点（单个图形）时则正常。
   - 遇到上述情况时，回退到 Figma REST API 一次取回完整子树：
     ```
     GET https://api.figma.com/v1/files/<fileKey>/nodes?ids=<nodeId>
     Header: X-Figma-Token: $FIGMA_API_KEY
     ```
     从 `document.children` 逐个读取 `absoluteBoundingBox`（位置与尺寸）与 `fills`（颜色）；矢量图形另取 `absoluteRenderBounds`——它才是实际可见范围，定位与导出尺寸以它为准。
   - REST API 有速率限制：短时间内重复调用会返回 HTTP 429。尽量一次取全；失败时按 10s、20s、30s 递增退避重试。
3. 将返回的完整 JSON 写入工作目录：
   ```
   <工作目录>/<nodeId>-figma.json
   ```
   - 文件名示例：`428-2244-figma.json`

### 阶段 2：下载图片资源

1. 在工作目录下创建 `images/` 文件夹（如不存在）
2. 从阶段 1 的 JSON 数据中识别需要下载的图片节点：
   - 含 `imageRef` 的填充（用户头像、背景图等）
   - 含 `IMAGE-SVG` 类型的节点（图标、矢量图）
   - 用户额外指定的 node-id
3. 调用 `Figma-AI-Bridge__download_figma_images` 批量下载：
   ```
   fileKey: 同上
   localPath: <工作目录>/images
   nodes: [
     { nodeId: "xxx:xxx", fileName: "<描述性名称>.png", imageRef: "..." },
     { nodeId: "xxx:xxx", fileName: "<描述性名称>.svg" }
   ]
   pngScale: 2
   ```
4. 命名规则：
   - 图标类：`icon-<功能描述>.svg`（如 `icon-bell.svg`、`icon-home.svg`）
   - 图片类：`img-<描述>.png`（如 `img-avatar.png`、`img-banner.png`）
   - 背景类：`bg-<描述>.png`（如 `bg-hero.png`）

### 阶段 3：生成 HTML

1. 读取 `<nodeId>-figma.json` 获取布局、样式、文本内容
2. 参考用户上传的截图确认视觉效果
3. 生成单文件 HTML（内联 CSS），遵循以下原则：

**布局规则：**
- 使用 Flexbox / CSS Grid 还原布局结构
- 从 figma.json 的 `layout` 字段提取：`mode`（row/column）、`gap`、`padding`、`justifyContent`、`alignItems`
- 尺寸使用相对单位（%, vw, rem），固定像素仅用于小元素（图标、间距）

**样式规则：**
- 从 `textStyle` 提取字体、字号、行高、字重
- 从 `fills` 提取颜色（支持纯色、渐变、图片填充）
- 从 `borderRadius`、`strokes`、`strokeWeight`、`effects` 提取圆角、边框、阴影/模糊
- 图片路径指向 `images/` 目录下的本地文件

**内容规则：**
- 从 `text` 字段提取所有文案，保持原文
- 保留 emoji 字符
- 交互元素（输入框、按钮）使用对应 HTML 标签

**语义化：**
- 顶部导航 → `<nav>`
- 侧边栏 → `<aside>`
- 主内容 → `<main>`
- 输入框 → `<input>` / `<textarea>`
- 可点击卡片 → `<button>` 或 `<a>`

4. 如有无法用 HTML/CSS 实现的 UI 元素（如复杂 SVG 动画、特殊滤镜），记录到：
   ```
   <工作目录>/unimplemented-ui.md
   ```
   格式：
   ```markdown
   # 未实现 UI 元素

   | 节点 ID | 名称 | 原因 | 建议替代方案 |
   |---------|------|------|-------------|
   | xxx:xxx | ... | ... | ... |
   ```

### 阶段 4：视觉校准与响应式适配

此阶段以 figma.json 数据为主要参考，用户上传的参考截图为辅助校验。

1. **间距还原（最高优先级）**：
   - 严格从 figma.json 提取所有间距数值：`padding`、`gap`、`margin`（来自 `layout` 和 `spacing` 字段）
   - 元素之间的间距必须与 Figma 设计稿一致，不可自行估算
   - 容器内边距、卡片间距、列表项间距等均以 figma.json 为准
   - 仅在 figma.json 缺失间距数据时，参考截图辅助判断
2. **图片尺寸还原（高优先级）**：
   - 从 figma.json 的节点 `width`、`height` 字段提取图片原始尺寸
   - 图片容器尺寸与 Figma 保持一致（使用 `max-width` + `height: auto` 保持比例）
   - 头像、图标等固定尺寸元素使用 Figma 中的精确像素值
   - 大图/Banner 使用相对单位但保持宽高比与 Figma 一致
3. **字体还原（高优先级）**：
   - 从 figma.json 的 `textStyle` 字段提取 `fontFamily`、`fontWeight`、`fontSize`、`lineHeight`
   - CSS 中的 `font-family`、`font-weight`、`font-size`、`line-height` 必须与 Figma 设计稿严格一致
   - 注意中文字体映射：如 `PingFang SC` 需提供合理的 fallback（`-apple-system, BlinkMacSystemFont, "PingFang SC", "Microsoft YaHei", sans-serif`）
   - `textAlignHorizontal` / `textAlignVertical` 对应 CSS 的 `text-align` 和垂直对齐方式
   - 不同层级文本（标题、正文、辅助文字）的字号、字重差异必须保留，不可统一化
4. **视觉对比**：将生成的 HTML 与参考截图逐区域对比
   - 整体布局比例
   - 颜色准确性
   - 字体大小和行高
   - 元素对齐方式
4. **美观性调整**：
   - 确保视觉层次清晰（标题 > 正文 > 辅助文字）
   - 颜色对比度满足可读性要求
5. **响应式适配（小屏友好）**：
   - 默认适配主屏幕宽度（参考 Figma 设计稿宽度）
   - 减少固定宽度：大容器、卡片等优先使用 `max-width` + `width: 100%`，避免硬编码 `px` 宽度
   - 添加断点适配小屏（≤768px）：
     - 侧边栏折叠或隐藏
     - 多列布局改为单列纵向排列
     - 卡片宽度改为 `100%`
     - 字号适当缩小（`clamp()` 或 `@media` 均可）
     - 输入框、按钮全宽
     - 水平间距适当收缩（可按比例缩小，但不可完全去除）
   - 添加超小屏断点（≤480px）处理极端情况
   - 使用 `@media` 查询实现
   - 图片添加 `max-width: 100%` 防止溢出
6. 输出最终 HTML 文件：
   ```
   <工作目录>/index.html
   ```

## 输出文件清单

```
<工作目录>/
├── <nodeId>-figma.json    # Figma 原始数据
├── index.html             # 最终 HTML 页面
├── unimplemented-ui.md    # 未实现元素记录（如有）
└── images/                # 图片资源
    ├── icon-*.svg
    ├── img-*.png
    └── bg-*.png
```

## 注意事项

- 工作目录默认为当前 workspace 下以项目名命名的文件夹，如用户未指定则用 `figma-output/`
- 单文件 HTML 便于预览和分享，CSS 全部内联在 `<style>` 标签中
- 图片路径使用相对路径 `images/xxx`
- 如果 Figma 数据中有组件实例（INSTANCE 类型），展开其子节点获取实际内容
- 渐变色从 `gradient` 字段直接提取 CSS `linear-gradient` 值
