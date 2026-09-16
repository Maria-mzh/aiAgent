# MCP 工具调用范式

Figma 和 Mastergo 的 MCP 服务器暴露的工具名和参数形态略有差异，但概念上的几类操作是一样的。这份文档描述如何探查已连接的服务器，以及四个阶段里常见的调用范式。

## 第 0 步 — 探查可用工具

不要凭空假设工具名。会话开始时，列出所有名字像设计 server 的工具：

- Figma MCP — 一般前缀是 `mcp__figma__` 或 `figma:get_file`、`figma:get_node`、`figma:export_image` 之类
- Mastergo MCP — 一般前缀是 `mcp__mastergo__` 或 `mastergo:*`

如果都找不到，跑 `tool_search(keywords=["figma", "design", "mastergo", "mockup"])`，然后通过 `suggest_connectors` 把结果给用户挑。

## 几类核心工具的角色

不同 server 的工具名不一样，但通常会覆盖以下几类。

### `get_file` / `get_document`
返回整个文件的节点树（JSON）。开销大——只在阶段 1 开头调用一次，且只在没有更精确的节点获取工具时才用。

### `get_node` / `get_node_info`
返回单个节点和它的子树，通常有遍历深度参数。**贯穿全流程的主力工具**：
- 阶段 1、2 用浅层（depth=2 或 3）
- 阶段 3 用全深度

### `render_image` / `export_image`
渲染节点为 PNG/JPG/SVG，可指定缩放：
- `scale=0.5` 或 `1`：阶段 1 整页概览
- `scale=2`：阶段 3 模块级参考图（diff 时更清晰）

把字节流存到本地再 view，不要试图直接解读二进制。

### `get_styles` / `get_variables`
返回设计文件里定义的命名 token。**如果存在就一定要用**——这是权威来源，比从节点 fill 反推 token 准确得多。

### `get_code` / `export_code`
有些 server 提供 CSS/React/Vue 代码导出。**当作起点参考，不当最终交付**。永远要对照渲染图核对。常见漂移：写死了应该是 token 的值；缺响应式逻辑；组件命名不规范。

## 阶段 1 调用范式

```
# 1. 整页渲染
render_image(node_id=<顶层>, scale=0.5)
# 存到 ./design-refs/page-overview.png

# 2. 浅层节点树
get_node(node_id=<顶层>, depth=2)
# 看 children 列表，那就是模块候选

# 3. 如果有 token 定义就拉
get_styles()  # 或 get_variables()
```

## 阶段 2 调用范式

对每个阶段 1 识别出的模块候选：

```
# 浅层看模块内部结构
get_node(node_id=<模块节点>, depth=2)
```

这一步主要是看 JSON 树、不是看像素，一般不需要渲染。

## 阶段 3 调用范式

每个要构建的组件或模块：

```
# 1. 完整子树
get_node(node_id=<目标>, depth=full)

# 2. 参考渲染图
render_image(node_id=<目标>, scale=2)
# 存到 ./design-refs/<模块名>.png

# 3. 可选的代码起点
get_code(node_id=<目标>, framework=<react|vue|css>)
```

文件系统里：
- 参考图放 `./design-refs/`（或用户指定的路径）
- 你的产物截图放 `./build-refs/`
- 用 `view` 工具并排比对

## 阶段 4 调用范式

```
# 整页按目标视口渲染做最终 diff
render_image(node_id=<顶层>, scale=1)
```

和你拼好的整页比对。

## MCP 错误处理

- **"节点不存在"** — 用户给的 URL 里 node id 可能过期了，或者文件后来被改了。重新问一下当前 id。
- **"权限不足" / 鉴权错** — MCP 需要重新授权，通过 `suggest_connectors` 传 server uuid 让用户重新连。
- **渲染返回空白图或极小图** — 试试别的 scale，有些 server 有尺寸上限。或者拉父节点用心眼裁剪。
- **`get_code` 输出乱七八糟** — 直接忽略，回到 渲染图 + 节点 JSON 来工作。代码导出是便利，不是契约。

## MCP 拉不到完整数据时怎么办

有些 MCP（特别是早期或功能受限的 Mastergo connector）不会在节点 JSON 里返回行高、字间距等属性。这时：

1. 从 2x 渲染图肉眼估值，作为 best-effort
2. **明确告诉用户**有哪些属性是估出来的、不是从文件读的，方便他们重点核对
3. 别把估的值伪装成"从 MCP 拿的"

## 不要让 MCP 做它不擅长的事

- **不要直接让 MCP "把整个 React 应用生成出来"**——即使有 `get_code`，也只把它当一类输入，真正的还原仍然要自己按四阶段走。否则你避开的还原度问题会通过导出工具回流。
- **不要指望 MCP 给动效规格**——大多数都不暴露。交互行为单独问用户要规格说明。
