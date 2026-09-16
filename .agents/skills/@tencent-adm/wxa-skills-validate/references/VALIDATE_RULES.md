# validate.mjs 内置规则详解（V001~V021）

> 本文件列出 `scripts/validate.mjs` 内置的所有校验规则。当 `validate-report.json` 中出现未知的 `id` 时按本文定位。
>
> 自定义规则通过 `--rules <path>` 合并（相同 id 覆盖内置）。
>
> **规则编号稳定、不回收**：编号一经使用即固定，废弃规则直接留空（当前空档：V004），不重排后续编号，以保证 `validate-report.json` 的 id、`--rules` 覆盖与历史引用不错位。

---

## 目录

- [单文件规则（regex 扫描）](#单文件规则regex-扫描)
  - [V001 禁止依赖主包](#v001-禁止依赖主包)
  - [V002 已注册接口必须为 async function](#v002-已注册接口必须为-async-function)
  - [V003 WXML 组件白名单](#v003-wxml-组件白名单)
  - [V005 CSS 禁止属性](#v005-css-禁止属性)
  - [V006 CSS 禁止选择器](#v006-css-禁止选择器)
- [跨文件规则](#跨文件规则)
  - [V007 定义-注册一致性](#v007-定义-注册一致性)
  - [V008 注册-实现一致性](#v008-注册-实现一致性)
  - [V009 接口返回值-outputSchema 一致性](#v009-接口返回值-outputschema-一致性)
  - [V010 组件取值-接口返回一致性](#v010-组件取值-接口返回一致性)
  - [V011 setData-WXML 绑定一致性](#v011-setdata-wxml-绑定一致性)
  - [V012 原子接口若关联原子组件则需文件齐全](#v012-原子接口若关联原子组件则需文件齐全)
  - [V013 mcp.json 体积限制](#v013-mcpjson-体积限制)
  - [V014 SKILL.md 必须存在且文件名严格大写](#v014-skillmd-必须存在且文件名严格大写)
  - [V015 原子组件必须配置关联小程序页面](#v015-原子组件必须配置关联小程序页面)
  - [V016 app.json 的 agent.skills[].description 必须存在且非空](#v016-appjson-的-agentskillsdescription-必须存在且非空)
  - [V017 handoff 接力页 pagePath 校验](#v017-handoff-接力页-pagepath-校验)
  - [V018 handoff query 参数名与接力页 onLoad 一致性](#v018-handoff-query-参数名与接力页-onload-一致性)
  - [V019 敏感接口初筛待模型判断](#v019-敏感接口初筛待模型判断)
  - [V020 体积与数量上限](#v020-体积与数量上限)
  - [V021 静态原子组件禁用网络请求与定时器](#v021-静态原子组件禁用网络请求与定时器)
- [规则与错误类型映射](#规则与错误类型映射)

---

## 单文件规则（regex 扫描）

### V001 禁止依赖主包

- **阶段**：`registration`，**级别**：`error`，**目标**：`**/*.js`
- **字面量禁用**：`getApp()`、`import ... from '@/...'`（命中即报错）
- **越界检查**（`require/import` 中以 `.` 开头的相对路径）：以 `app.json` 的 `subPackages[].root` 作为边界
  - 落在分包根子树内 → ✅ 合法（**含分包根下 `_shared/` 等公共目录**）
  - 落到分包根之外 → ❌ "超出 skill 分包边界"
  - 落入兄弟 skill 私有子树 → ❌ "落入另一个 skill 的私有目录"
- **典型修复**：跨 skill 复用工具 → 抽到 `skills/_shared/`，用 `require('../../_shared/xxx')`；越界相对路径 → 把目标模块挪入分包，或改为入参/JSAPI

### V002 已注册接口必须为 async function

- **阶段**：`registration`
- **级别**：`error`
- **类型**：跨文件校验（基于 `mcp.json` 的注册列表）
- **目标**：仅 `mcp.json` 中 `apis[].name` 对应的实现文件（在 `apis/` / `tools/services/` / `tools/` 下按同名 `.js` 解析）

规则会读取 `mcp.json` 列出的 API 名称，对每个已注册接口，校验其实现文件内是否存在以下任一形态：

- `async function <name>(...)`
- `const <name> = async (...) => ...` / `let` / `var`
- `<name> = async ...`（如 `module.exports.<name>`）
- `{ <name>: async (...) => ... }`
- `{ async <name>(...) { } }`

`apis/` 目录下未被 `mcp.json` 注册的工具函数**不会**被检查，避免对辅助模块的误判。

**典型修复**：把注册接口改为 `async function` 或 `async () => {}`；若目标函数只是工具函数，请将其移到同级 `utils/` 目录并在 `mcp.json` 中取消其注册。

### V003 WXML 组件白名单

- **阶段**：`component`
- **级别**：`error`
- **类型**：`regex_absent`（以下模式**不允许出现**）
- **目标**：每个 `mcp.json.apis[]._meta.ui.componentPath` 对应的 `<componentPath>.wxml`

| 正则 | 含义 |
|------|------|
| `<(?!view\|text\|image\|map\|button\|canvas\|scroll-view\|collapsible-view\|block\|template\|\\/\|!--)[a-zA-Z]` | 仅允许 `view` / `text` / `image` / `map` / `button` / `canvas` / `scroll-view` / `collapsible-view` 八种内置组件（含 `block` / `template` 和注释）。禁用 `navigator` / `swiper` / `input` / `textarea` / `picker` / `checkbox` / `radio` / `form` / `slider` / `switch` / `editor` / `rich-text` / `icon` / `progress` / `web-view` 等 |
| `<button[^>]*\sopen-type\s*=` | 原子组件的 `<button>` 不支持 `open-type` 属性 |
| `<scroll-view(?![^>]*\sscroll-x(?:[\s=>]\|$))` | `<scroll-view>` 必须显式声明 `scroll-x`（仅支持横向滚动） |
| `<scroll-view[^>]*\sscroll-y\s*=\s*["']?(?:true\|\{\{\s*true\s*\}\})` | `<scroll-view>` 不支持 `scroll-y="true"`（纵向滚动不支持） |
| `<image[^>]*\ssrc\s*=\s*["'][^"'{]*\.(?:webp\|gif\|svg\|bmp\|avif)\b` | `image` 仅支持 png / jpg 格式（且仅网络地址） |
| `\sbind(?::)?(?:touchstart\|touchmove\|touchend\|touchcancel\|longpress\|longtap\|input\|change\|submit\|scroll\|blur\|focus\|confirm)\s*=` | 原子组件仅支持 `tap` 事件与 `image` 的 `load` / `error` 事件 |

**典型修复**：
- 纵向内容过多 → `<collapsible-view>` 展开**半屏原子组件**（把每一项作为其直接子节点）；它仍属于原子组件，不是另开半屏页面。或减少展示条数 + 上行"查看更多"`api/call`
- 横向超长内容 → `<scroll-view scroll-x="true">` 包裹横向列表（如商品横滚卡片）
- `<scroll-view scroll-y>` 纵向滚动 → 改为 `collapsible-view`（半屏原子组件）或减少展示条数
- `<navigator>` → `<view>` 带 `bindtap`（小程序 AI 不在页面栈内导航时直接删除）
- `<swiper>` → 用 `<view>` 列表平铺，或 `<scroll-view scroll-x>` 横滚展示
- `<input>` / `<textarea>` / `<picker>` → 交互类不适合原子组件，删除后由小程序 AI 对话收集入参
- `<button open-type="getPhoneNumber" bindgetphonenumber="...">` → 手机号只能在原子接口侧取，组件侧删除该按钮或改为上行 `api/call`
- `<button open-type="share">` → `<button bindtap="onShare">` + 在 handler 内调 `wx.shareAppMessage(...)`
- `bindtouchstart` / `bindlongpress` 等 → 改为 `bindtap`；`image` 的加载态用 `binderror` / `bindload`
- `.webp` / `.gif` 图片 → 换成 png / jpg 地址

### V005 CSS 禁止属性

- **阶段**：`component`
- **级别**：`error`
- **类型**：`regex_absent`
- **目标**：每个 `mcp.json.apis[]._meta.ui.componentPath` 对应的 `<componentPath>.wxss`

> 判定基准是官方「运行机制 - WXSS 属性支持范围」那张白名单表，**表里没有的属性一律当作不支持**。

| 正则 | 禁用内容 |
|------|----------|
| `position\s*:\s*fixed` | `position: fixed` |
| `position\s*:\s*sticky` | `position: sticky` |
| `z-index\s*:` | `z-index` |
| `display\s*:\s*grid` | `display: grid` |
| `display\s*:\s*table` | `display: table` |
| `display\s*:\s*inline-flex` | `display: inline-flex` |
| `display\s*:\s*-webkit-box` | `display: -webkit-box`（多行截断改用 `-wx-line-clamp`） |
| `float\s*:` | `float` |
| `text-decoration\s*:` | `text-decoration` |
| `--[a-zA-Z][\w-]*\s*:` | CSS 变量 `--*` |
| `(?:^\|[;\{\s])transition(?:-[a-z]+)?\s*:` | `transition`（**原子组件不支持动画**） |
| `(?:^\|[;\{\s])animation(?:-[a-z]+)?\s*:` | `animation` |
| `@keyframes\b` | `@keyframes` |
| `(?:^\|[;\{\s])overflow(?:-x\|-y)?\s*:` | `overflow` / `overflow-x` / `overflow-y` |
| `-webkit-line-clamp\s*:` | `-webkit-line-clamp`（改用 `-wx-line-clamp`） |
| `-webkit-box-orient\s*:` | `-webkit-box-orient` |
| `(?:^\|[;\{\s])(?:row-\|column-)?gap\s*:` | `gap` / `row-gap` / `column-gap` |
| `cursor\s*:` | `cursor` |

**典型修复**：
- `transition` / `animation` → 删除，**没有动画**；状态变化直接 `setData` 切类名，按下反馈用组件的 `hover-class`
- `overflow: hidden` → 直接删除；裁剪由宿主容器完成，组件根节点也不要写 `height` / `max-height` / `min-height`（写了会让 `NotificationType.Overflow` 回调失效）
- `-webkit-line-clamp: 2` + `display:-webkit-box` + `-webkit-box-orient` 三件套 → 一行 `-wx-line-clamp: 2`
- 单行省略 → `white-space: nowrap` + `text-overflow: ellipsis`（不要再加 `overflow: hidden`）
- `gap` → 用 `margin` 控制间距
- `position: fixed` → `position: absolute`；`display: grid` → `flex`；自定义变量改常量

### V006 CSS 禁止选择器

- **阶段**：`component`
- **级别**：`error`
- **类型**：`regex_absent`
- **目标**：每个 `mcp.json.apis[]._meta.ui.componentPath` 对应的 `<componentPath>.wxss`

禁：子选择器 `>`、相邻兄弟 `+`、通用兄弟 `~`、伪元素 `::*`、伪类 `:hover`/`:focus`/`:active`/`:checked`/`:disabled`/`:first-child`/`:last-child`/`:nth-child`、属性选择器 `[attr=]`。

**典型修复**：全部用**类名选择器**替代；状态用类切换；奇偶行用 JS 预计算类名。

---

## 跨文件规则

### V007 定义-注册一致性

- **阶段**：`registration`
- **级别**：`error`

比对 `<skill>/mcp.json` 的 `apis[].name` 与 `<skill>/index.js` 中的 `wx.modelContext.registerAPI('name', fn)`。

**典型 fail**：

- `mcp.json` 定义了 `searchItems`，但 `index.js` 未注册 → 补 `wx.modelContext.registerAPI('searchItems', searchItems)`
- `index.js` 注册了 `searchItems`，但 `mcp.json` 未定义 → 在 `mcp.json` 中补 `apis[]` 条目

### V008 注册-实现一致性

- **阶段**：`registration`
- **级别**：`error`

比对 `index.js` 的 `require('./apis/xxx')` 与实际 `apis/xxx.js` 文件是否存在。

**典型 fail**：`require('./apis/searchItems')` 但 `apis/searchItems.js` 文件不存在 → 创建该文件。

### V009 接口返回值-outputSchema 一致性

- **阶段**：`output`
- **级别**：`error`

比对 `apis/<name>.js` 中 `structuredContent: { ... }` 字面量的字段名与 `mcp.json` 中 `outputSchema.properties` 的字段名。

**典型 fail**：

- 接口返回了 `{ items: [] }`，但 outputSchema 未声明 `items` → 在 `mcp.json` 的 `outputSchema.properties` 补 `items`
- `outputSchema.required` 声明了 `items`，但接口未返回 → 在 `structuredContent` 补 `items`

### V010 组件取值-接口返回一致性

- **阶段**：`component`
- **级别**：`error`

比对组件 `components/<name>/index.js` 中的 `result.structuredContent.xxx` 与对应接口的 `structuredContent` 字段。

**关联规则**：
- 优先按组件 JS 中 `atomicApi: 'xxx'` 元信息匹配接口
- 其次按 `apis/*.js` 中找字段全集匹配

**典型 fail**：组件读 `result.structuredContent.items`，但接口返回的是 `list` → 在 `structuredContent` 补 `items`（或改组件读 `list`）。

### V011 setData-WXML 绑定一致性

- **阶段**：`component`
- **级别**：`error`

双向校验组件 `index.js` 的 `setData({ field: ... })` 与 `index.wxml` 的 `{{field}}`：

- `setData` 有 `x` 但 WXML 未用 `{{x}}` → fail（冗余 setData）
- WXML 用 `{{x}}` 但 `setData` / `properties` / 忽略名单中都没有 `x` → fail（未定义字段）

**忽略名单**：`item`、`index`、`wx`（`wx:for` 内部变量）。

### V012 原子接口若关联原子组件则需文件齐全

- **阶段**：`component`
- **级别**：`error`

`mcp.json` 中的 `apis[]` **按需**声明 `_meta.ui.componentPath`（纯操作型/中间态数据接口可不声明，仅负责执行）。**若已声明**，则该路径对应的组件文件必须完整。

**检查项**：

- `_meta.ui.componentPath` 未声明 → 直接 pass，跳过组件文件检查
- 若已声明：
  - 为相对 skill 目录的组件基路径
  - 同基路径存在 `.js` / `.json` / `.wxml` / `.wxss` 四个文件

**典型 fail**：

- `componentPath` 不是相对 skill 目录的基路径 → 改为相对路径，例如 `"components/<name>/index"`
- 组件缺 `<componentPath>.wxss` → 创建该文件

### V013 mcp.json 体积限制

- **阶段**：`registration`，**级别**：`error`（另含 `warning` 子项）

平台对 `mcp.json` 有 **24000 字符**的硬上限，超出会在预览 / 上传时被拒绝。**计算时会先去掉所有 `apis[].outputSchema` 字段**再序列化，因此 `outputSchema` 写得详细不占额度，`description` / `title` / `inputSchema` 才占。

**检查项**：
- 去掉 `outputSchema` 后的 JSON 长度 > 24000 → error
- 长度 ≥ 上限的 90%（21600）→ warning（提前预警，可带着进入下一阶段）
- JSON 解析失败 → error

**典型修复**：压缩 `description` / `inputSchema.properties[].description` 的描述文字；接口多到难以精简时按职责把一个 skill 拆成多个 skill 分包。**不要把示例、枚举值硬塞进 `outputSchema` 来规避统计**——那样虽然不计入长度，但会让模型拿到错误的返回结构描述。

### V014 SKILL.md 必须存在且文件名严格大写

- **阶段**：`registration`，**级别**：`error`

每个 skill 目录必须存在文件名**严格为** `SKILL.md` 的文件。`skill.md` / `Skill.md` 等大小写变体一律 fail（macOS / Windows 的默认文件系统通常不区分大小写，本地编辑容易绕过检查，但 Linux / CI / 后台严格区分）。

**典型修复**：把文件重命名为 `SKILL.md`。在默认不区分大小写的系统上直接改大小写可能"改了等于没改"，可先改成临时名再改回：终端执行 `mv skill.md tmp && mv tmp SKILL.md`（用 git 管理时也可 `git mv skill.md SKILL.md`）。文件不存在则按 wxa-skills-generate `references/CODE_TEMPLATES.md` 第五节模板新建。

### V015 原子组件必须配置关联小程序页面

- **阶段**：`component`，**级别**：`error`

原子组件渲染出的 GUI 卡片，标题栏右上角有一个"进入小程序"的入口，靠 `mcp.json` 顶层 `components[]` 里的 `relatedPage` 配置，**这是平台的必填项**——缺失会在 `cli preview` / 上传时被强校验拦下。通过该入口进小程序的场景值为 1442 / 1443。

对 `mcp.json.apis[]` 中每个声明了 `_meta.ui.componentPath` 的接口（未声明则跳过），要求：

1. `mcp.json` 顶层 `components[]` 中存在一条 `path` 与该 `componentPath` **字符串完全相等**（含末尾 `/index`，严格相等、不做归一化）的条目；
2. 该条目的 `relatedPage` 存在且 trim 后非空；
3. `relatedPage` **必须以 `/` 开头**（绝对路径）；
4. 去掉前导 `/`、去掉 query 后，存在于项目 `app.json` 的可路由页面集合中（主包 `pages[]` ∪ 所有分包 `root + '/' + page`）。读不到 `app.json` 时跳过第 4 条。

**运行时配套**：组件可在 `created` 中保存 `this._viewCtx = wx.modelContext.getViewContext(this)`；收到 `NotificationType.Result` 后调 `this._viewCtx.setRelatedPage({ query })` 把业务参数拼进去（只传 `query` 时路径取 `relatedPage`；组件可能关联多个页面时才额外传 `path`）。也可按使用点获取 `viewCtx` 后调用。

**典型修复**：在 `mcp.json` 顶层 `components[]` 追加 `{ "path": "<与接口 _meta.ui.componentPath 完全一致的字符串>", "relatedPage": "/<主包 pages[] 中的页面 或 分包 root+page 拼接>" }`（`relatedPage` 前导 `/` 必填）；业务上没有对应页面时**兜底填首页 `/<app.json.pages[0]>`**。详见 wxa-skills-generate `SKILL.md` D.3。

### V016 app.json 的 agent.skills[].description 必须存在且非空

- **阶段**：`registration`，**级别**：`error`
- **类型**：项目级校验（仅执行一次，不按 skill 目录循环）

每个 `app.json` 中 `agent.skills[]` 的条目必须包含**非空的 `description` 字段**。这是后台的硬性要求，若缺失将导致 skill 无法正常注册。

**检查项**：
- `agent.skills[]` 数组存在且非空
- 每个条目中 `description` 字段存在，且 `trim()` 后非空

**典型 fail**：
- `agent.skills` 条目只有 `{ "path": "skills/xxx" }`，缺少 `description`
- `{ "name": "xxx", "path": "skills/xxx", "description": "" }` — `description` 为空字符串

**典型修复**：在 `app.json` 的 `agent.skills` 中为每个条目补上非空的 `description`，如：`{ "name": "xxx", "description": "该 skill 的业务描述", "path": "skills/xxx" }`。

---

### V017 handoff 接力页 pagePath 校验

- **阶段**：`registration`，**级别**：`error`（另含 `warning` 子项）

进小程序场景下，原子接口通过 `_meta.ui.pagePath` 声明"用户点卡片后进入的接力业务页"，并在返回值顶层的 `handoff` 中传递 query / payload。本规则对**声明了 `_meta.ui.pagePath` 的接口**校验（未声明则跳过，`pagePath` 按需）：

1. `pagePath` **非空**（trim 后非空字符串）；
2. **必须以 `/` 开头**（绝对路径）；
3. **不应带 query**（`?`）——query 由返回值 `handoff.query` 传递，`pagePath` 只写页面路径；
4. 去掉前导 `/` 后，**存在于项目 `app.json` 的可路由页面集合**中（主包 `pages[]` ∪ 所有分包 `subPackages[]` / `subpackages[]` 的 `root + '/' + page`）。读不到 `app.json` 时跳过第 4 条。

**warning 子项**：声明了 `pagePath` 但该接口实现文件（`apis/` / `tools/services/` / `tools/` 下同名 `.js`）中**未出现 `handoff`** 时，提示补返回值——用户点卡片进接力页会缺少 query / payload 传递。

**典型修复**：
- `pagePath` 带 query（如 `/pages/drug/detail?id=1`）→ 改为 `"/pages/drug/detail"`，把 `id=1` 放到返回值 `handoff.query`
- `pagePath` 不以 `/` 开头 → 补前导 `/`
- 页面不存在 → 改为 `app.json` 中真实页面
- 实现未返回 `handoff` → 在返回值顶层增加 `handoff: { query, payload? }`（详见 wxa-skills-generate `SKILL.md` D.6）

---

### V018 handoff query 参数名与接力页 onLoad 一致性

- **阶段**：`registration`，**级别**：`error`

对声明了 `_meta.ui.pagePath` 且返回值 `handoff.query` 含**非空参数名** 的接口，读取接力页 JS 文件，检查 `handoff.query` 的每个参数名是否在其中作为属性被引用。

**背景**：handoff 的 `query`（string，如 `'drugId=xxx'`）会被框架原样注入接力页 `onLoad(query)` 与 `wx.onAgentHandoff` 回调。如果 `handoff.query` 的参数名与页面实际读取的参数名不一致（如 handoff 传 `matchId` 但页面读 `uni_id`），用户点卡片进接力页时 handoff 意图丢失——页面拿不到预期参数，展示错误内容或空白。

**检查项**：

1. 从接口实现文件中提取 `handoff.query` 的 string 值，解析出参数名集合（空 string `''` → 跳过）
2. 读取 `<projectRoot>/<pagePath>.js`（接力页 JS）
3. 在接力页 JS 中搜索 `.<key>` 或 `['<key>']` 模式（不依赖 onLoad 参数名：key 不作为属性出现即说明页面未消费）
4. 若**全部** key 均未被引用 → error（参数名不匹配或页面忽略 handoff query）
5. 若**至少一个** key 被引用 → pass（页面可能只消费部分参数）

**跳过条件**（不报 error，输出 pass）：

- 接口无 `pagePath`（V017 管）
- `handoff.query` 为空 string `''` 或无参数名（无参接力）
- 接力页 JS 文件不存在（V017 管页面存在性）
- `handoff.query` 不是 string 字面量（如变量引用，无法静态提取参数名）

**典型 fail**：

- `searchPlayers` 返回 `handoff: { query: 'matchId=' + params.matchId }`，接力页 `uni_match.js` 的 `onLoad: function(t)` 读取 `t.uni_id`——`t.matchId` 未出现在文件中 → error
- `viewMatchScorecard` 返回 `handoff: { query: `matchId=${params.matchId}` }`，接力页 `ScoreCard.js` 的 `onLoad: function(a)` 完全不读 query——`a.matchId` 未出现在文件中 → error

**典型修复**：读接力页 `onLoad` 确认其读取的参数名，将 `apis/<name>.js` 中 `handoff.query` 的参数名改为页面实际读取的名称。例如 `handoff: { query: 'matchId=' + params.matchId }` → `handoff: { query: 'uni_id=' + params.matchId }`（当页面读 `t.uni_id` 时）

---

### V019 敏感接口初筛待模型判断

- **阶段**：`registration`，**级别**：`error`
- **类型**：项目级校验（仅执行一次，不按 skill 目录循环）

会产生**不可逆副作用**的敏感接口（注销/删除/解绑/解散/踢出/退出等）在 execute 阶段默认拒绝执行。本规则做**两阶段判定**：脚本初筛 + 模型判断。

**脚本初筛**（`checkV019` 自动）：扫描所有 `mcp.json`，把 `name`/`description` 命中 `DESTRUCTIVE_KEYWORDS` 的接口写入 `cli-agent-run/destructive-manifest.json`，每个候选 `destructive: null`（待判断）。有 `null` → 报 **error**（阻断 build），逼模型判断。

**模型判断**（人工/模型编辑 manifest）：逐个候选填 `destructive` 为 `true`（真敏感，execute 跳过）或 `false`（误判，execute 放行）+ `destructiveReason`。填完重跑 validate，V019 读 manifest 保留已有判断、检查无 `null` → pass。

**关键词清单**（定义于 `scripts/lib.mjs` 的 `DESTRUCTIVE_KEYWORDS`）：
[
  "logoff", "logout", "注销", "销户", "close",
  "cancel", "delete", "删除", "remove", "移除", "移出", "取消", "清空", "clear",
  "unbind", "解绑", "unsubscribe", "退订",
  "dissolve", "解散", "kick", "踢出", "quit", "exit", "退出",
]

**manifest 结构**：

```jsonc
{
  "generatedAt": "2026-07-22T...Z",
  "project": "<abs-path>",
  "apis": [
    {
      "skill": "demo", "apiName": "deleteAccount",
      "hitKeyword": "delete", "hitField": "name",
      "destructive": null,          // null=待判断(脚本初筛) / true=真敏感 / false=误判(模型填)
      "destructiveReason": ""       // 模型填的判断理由
    }
  ]
}
```

**execute 如何读 manifest**（`isDestructiveApi`）：manifest 存在时以 manifest 为准（不回退关键词）：`destructive=true` → exit 3 拒绝；`destructive=false` → 放行；`destructive=null` → 拒绝（待判断，安全第一）；接口不在 manifest → 放行（V019 已扫过，不命中关键词）。manifest 不存在时回退关键词判定。

**典型流程**：

1. 首次跑 validate → V019 初筛，`deleteAccount`/`getDeletedOrders` 命中 `delete`，manifest 落盘 `destructive:null`，报 error
2. 模型编辑 manifest：`deleteAccount` → `destructive:true`+"注销账号不可逆"；`getDeletedOrders` → `destructive:false`+"只读查询"
3. 重跑 validate → V019 保留判断，无 null → pass；execute 时 `deleteAccount` 拦截、`getDeletedOrders` 放行

**典型修复**：编辑 `cli-agent-run/destructive-manifest.json`，将 `destructive:null` 填为 `true`+`destructiveReason`（真敏感）或 `false`+`destructiveReason`（误判如只读查询）。

> **设计说明**：脚本只做关键词初筛（不漏），模型做语义判断（不误伤）。`_meta.destructive` 不放 `mcp.json`（最终产物纯净）；生成阶段 destructive 标记写 `probe/plan.json`（probe 跳过用）；校验阶段 V019 初筛 + 模型判断落盘 manifest，execute 读 manifest。`validate` 不依赖 `.ai-mode-skills/`，保持独立性。

---

### V020 体积与数量上限

- **阶段**：`registration`，**级别**：`error`
- **类型**：项目级校验（仅执行一次，不按 skill 目录循环）

平台对配置文件大小和 SKILL 数量有硬上限，超出会在预览 / 上传时被拒绝。**按 UTF-8 字节数计算**（中文一个字约 3 字节，不是字符数）。

| 检查对象 | 上限 | 来源 |
|---|---|---|
| 每个 skill 的 `SKILL.md` | 16000 字节 | 遍历所有 skill 分包目录 |
| `AGENTS.md`（全局提示词） | 10000 字节 | `app.json` 的 `agent.instruction` 指向的文件 |
| `page-meta.json`（页面元数据） | 8000 字节 | `app.json` 的 `agent.pageMetadata` 指向的文件 |
| `agent.skills[]` 条目数 | 30 | `app.json` |

**跳过条件**：`agent.instruction` / `agent.pageMetadata` 未配置时，对应检查整项跳过（两者都是可选配置）。配置了但文件读不到 → error（配置与文件不一致）。`page-meta.json` 额外做一次 JSON 可解析校验。

`mcp.json` 的 24000 上限由 V013 单独负责，不在本规则内重复。

**典型修复**：

- `SKILL.md` 超限 → 精简为 5 节结构（能力域定位 / 触发场景 / 不适用范围 / 前置条件 / 使用顺序），把接口契约挪回 `mcp.json`。注意 `SKILL.md` 只支持单文件，**不能拆成多个 md 引用**
- `AGENTS.md` 超限 → 只保留跨 skill 的全局信息（服务边界、skill 之间怎么选、回答风格），单个 skill 的路由说明回各自 `SKILL.md`
- `page-meta.json` 超限 → 只保留真正需要"账号卡片"引导的页面；接口已能返回页面路径的场景改用 `outputSchema` 的 `format: "page-link"`
- SKILL 数量超 30 → 合并职责相近的 skill

### V021 静态原子组件禁用网络请求与定时器

- **阶段**：`component`，**级别**：`error`

普通（静态）原子组件**默认不支持登录、网络请求和定时器**；声明 `scope.dynamic` 后可使用。`wx.cloud.*` 在普通和实时动态原子组件中都不支持，数据必须在原子接口中获取。

**检查项**：对每个声明了 `_meta.ui.componentPath` 的接口，读其 `<componentPath>.js` 及相对引用的 JavaScript 模块，扫描 `wx.login(` / `wx.checkSession(` / `wx.request(` / `wx.cloud` / `setTimeout(` / `setInterval(`。

**动态组件处理**：声明 `permissions["scope.dynamic"]` 时，仅放行 `wx.login` / `wx.checkSession` / `wx.request` 与定时器；发现 `wx.cloud.*` 仍报错。组件文件不存在 → 跳过（由 V012 负责）。

**典型修复**：

- 组件里 `wx.cloud.*` → 移到原子接口，数据经 `NotificationType.Result` 下发
- 组件里 `wx.login` / `wx.checkSession` / `wx.request` 或定时器且没有实时需求 → 删除，改为原子接口取数或一次性渲染
- 确有实时刷新或登录需求 → 在 `mcp.json` 的 `components[]` 该条目加 `"permissions": { "scope.dynamic": { "desc": "<说明使用场景>" } }`。注意此能力需单独审核（正常提审即可），非必要场景不建议用

---

## 规则与错误类型映射

| 规则 id | 错误类型（SKILL.md 中的分类） | 典型修复路径 |
|---------|------------------------------|-------------|
| V001 | T5 合规性违规 | 去除主包依赖，数据通过入参传入 |
| V002 | T1 命名/结构 | 函数头补 `async` |
| V003 / V005 / V006 | T5 合规性违规 | 白名单内等价实现 |
| V007 | T6 注册缺失 | 补 `registerAPI` 或在 `mcp.json` 补 `apis[]` |
| V008 | T6 注册缺失 | 创建缺失的 `apis/<name>.js` |
| V009 | T2 Schema 不一致 | 对齐 `structuredContent` 与 `outputSchema` |
| V010 | T4 组件取值路径错 | 修 `result.structuredContent.xxx` 访问路径 |
| V011 | T3 组件绑定不一致 | 对齐 `setData` 与 WXML `{{}}` |
| V012 | T6 注册缺失（组件维度） | 若已声明 `componentPath`，补齐同基路径 `.js` / `.json` / `.wxml` / `.wxss` 文件 |
| V013 | T-mcp-size | 压缩 description/inputSchema 描述文字，或按职责拆分 skill 分包 |
| V014 | T1 命名/结构 | `SKILL.md` 文件名严格大写 |
| V015 | T-relatedPage 关联页面缺失 / path 不一致 / 缺前导 `/` | 在 `mcp.json.components[]` 补 `{ path, relatedPage }`，`path` 与接口 `_meta.ui.componentPath` **字符串完全相等**，`relatedPage` **必须以 `/` 开头**，无业务对应页面时填首页 |
| V016 | T-skill-description skill 描述缺失 | 在 `app.json` 的 `agent.skills[]` 中为该条目补 `description` 字段 |
| V017 | T-handoff 接力页配置错误 | 修正 `_meta.ui.pagePath`（以 `/` 开头、不含 query、页面真实存在）；补返回值 `handoff` |
| V018 | T-handoff-query handoff query 参数名不匹配 | 读接力页 onLoad 确认参数名，将 handoff.query 的参数名改为页面实际读取的名称 |
| V019 | T-destructive | 编辑 manifest 填 `destructive=true`（真敏感）/`false`（误判）+ `destructiveReason` |
| V020 | T-limits 体积/数量超限 | 精简 `SKILL.md` / `AGENTS.md` / `page-meta.json`，或合并 skill |
| V021 | T-dynamic 静态组件用了动态能力或任意组件用了云开发 | 静态组件删除 `wx.login` / `wx.checkSession` / `wx.request` / 定时器或声明 `scope.dynamic`；任意组件的云开发移到原子接口 |

---

## 自定义规则

通过 `--rules <path>` 合并自定义规则 JSON：

```json
{
  "rules": [
    {
      "id": "V100",
      "name": "自定义规则示例",
      "stage": "registration",
      "level": "warning",
      "type": "regex_absent",
      "targets": ["*/apis/*.js", "*/utils/*.js"],
      "patterns": [
        { "regex": "console\\.log", "message": "生产代码不应有 console.log" }
      ]
    }
  ],
  "crossFileRules": []
}
```

相同 `id` 会覆盖内置规则。
