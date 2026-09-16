---
name: ys-agent-frontend-dev
description: ys-agent 前端（frontend/）功能迭代的工程规范守门 skill。当任何 agent 接到 ys-agent 前端相关任务时——新增页面、新增组件、加 API、加 Pinia store、改路由、写 mock、调整布局、修改样式、对接后端接口、迁移 demo 代码、做主题、加权限、写公共组件、做数据可视化、做表单、做弹窗、做表格、做搜索过滤、调 Element Plus 用法、做响应式、做性能优化，或仅仅是在 frontend/ 目录下编辑任何 .vue / .ts / .scss / .css / vite.config / tsconfig / package.json 文件——都必须先调用本 skill，按统一的工程规范实施，避免不同 agent 各自发挥导致目录混乱、风格漂移、重复造轮子。即使任务听起来很小（"加个按钮""改个文案"），只要发生在 frontend/ 下，也要先读本 skill，确认正确的放置位置、命名、引入方式后再动手。
---

# ys-agent 前端开发规范 skill

为 ys-agent 项目的 `frontend/` 模块提供一套**强约束**的工程规范，让每个接手前端迭代的 agent 都按同一套思路落地，杜绝"我以为可以这样做"。

本 skill 是 `docs/frontend-engineering-plan.md` 的可执行延伸：plan 描述"为什么"，本 skill 提供"怎么做"。

---

## 0. 触发判断（先读完再决定动不动）

只要满足任一条件，就在动手前完整读完本 SKILL.md 并按工作流执行：

- 任务描述出现 frontend / Vue / Element Plus / Pinia / 路由 / 组件 / 接口对接 / mock / 样式 / 主题 / 布局 等关键词
- 任务路径涉及 `frontend/`、`docs/sessions/S13-*`、`docs/frontend-engineering-plan.md`
- 用户引用了 S13 文档里出现的页面（Skill 广场 / MCP 广场 / Tool 广场 / 我的 Skill / 我的 MCP / Playground）
- 任何代码改动落在 `frontend/src/` 下，无论改一行还是改一个目录

不确定就读，本 skill 是项目级"前端开发指南"，读它的成本永远低于偏离规范后返工的成本。

---

## 1. 核心原则（铁律）

这 9 条是任何 agent 都不能绕开的底线。**违反任意一条 = PR 必须返工**。

1. **目录即契约**：每类文件有且只有一个正确位置，详见 `references/conventions.md`。新增文件前先翻"目录契约"表，再下手。
2. **组件不直接调 axios**：HTTP 一律走 `src/api/`；视图通过 Pinia store action 拿数据；store 调 `api/modules/`；axios 只在 `src/api/client.ts` 里实例化；各微服务实例在 `src/api/services/index.ts` 声明；模块文件 import 对应 service client，不直接使用 axios。**新增微服务时**，除了在 `services/index.ts` 加 client，还必须同步：① `.env.development` 声明独立路径前缀（如 `/api-xxx`）和 `VITE_XXX_API_TARGET`；② `vite.config.ts` 追加对应 dev proxy rewrite 条目。开发环境多个 `VITE_*_API_BASE` 禁止共用同一个 `/api`，否则无法按微服务独立路由。完整四步见 `references/anti-patterns.md` A25。
3. **类型先行**：每个接口都要在 `src/types/api/<域>.ts` 定义 `Request / Response` 类型；store / view 直接复用类型，不允许 `any`（确需 `any` 必须 PR 写理由）。
4. **Element Plus 按需 + 自动注册**：直接 `<el-button>` 用即可，**禁止** `import { ElButton } from 'element-plus'`。图标分两种场景，规则不同：
   - **模板中作为子元素展示**（如 `<el-icon>` 内）：直接 `import { IconName } from '@element-plus/icons-vue'` 后在模板里写 `<IconName />`，`unplugin-vue-components` 会识别 PascalCase 静态 tag 并自动处理。**不要用** `<component :is="'IconName'" />`（string literal），该形式无法被静态分析器解析，图标同样不渲染。
   - **作为 JavaScript 值传给 `:icon` prop**（如 `<el-button :icon="...">`）：**必须**直接 `import { IconName } from '@element-plus/icons-vue'` 后传入，`resolveComponent('IconName')` 在此项目中**不可用**（`@element-plus/icons-vue` 是独立包，`app.use(ElementPlus)` 不全局注册图标组件，`resolveComponent` 会返回字符串而非组件，导致图标静默不渲染）。详见 `references/anti-patterns.md` A27。
5. **样式走 SCSS 变量**：颜色、间距、圆角、阴影、字号一律从 `@/styles/variables.scss` 取；业务组件**禁止**写死十六进制颜色。Element 主题色覆盖只能改 `@/styles/element-overrides.scss`。
6. **Mock 与真后端零代码切换**：开发期通过 `VITE_USE_MOCK=true` 启用 MSW；任何接口对接前先写 mock handler，再写真接口对接，保证前端可独立运行。
7. **路由 = 单一来源**：菜单、面包屑、title 全部从 `routes` 的 `meta` 派生，**禁止**在 SideMenu / Breadcrumb 里手写菜单数据。路由 `meta.title` 和 `meta.group` 必须存储 i18n key（如 `'skill.square.title'`、`'nav.group.square'`），**禁止**存储中文文本。SideMenu 读取后通过 `$t()` 翻译展示。
8. **可保留资产不删**：旧 demo（`ChatView` / `chat store` / 三栏布局）已迁移到 `src/views/playground/`，**禁止**因为"看着乱"就删除；任何与流式 SSE 协议相关的代码改动需在 commit message 写明。
9. **所有用户可见文本必须走 i18n**：模板文本、ElMessage 提示、表单校验 message、组件 props 默认值、路由 title——任何用户能看到的字符串**禁止**硬编码中文或英文，必须通过 `$t('key')` 或 `t('key')` 引用。翻译 key 集中在 `src/i18n/locales/*.json`，命名规范见 `references/conventions.md` §10。**例外情况**（不走 i18n，在代码行尾加 `// i18n-ignore` 并说明原因）：① mock 数据中的业务模拟文本（如虚构的 skill 名称）；② 品牌名和产品固有专有名词（如 `ys-agent`、`Skill`、`MCP`、`Tool`、`Playground`）——这些是产品术语，各语言保持原文，直接写在代码中即可；③ 纯技术标识（如日志 tag、API path）。

---

## 2. 你接到任务后的第一步：选 workflow

打开 `references/workflows.md`，找到与任务最匹配的那个 workflow 并完整跟随。Workflow 覆盖 10 类常见任务：

| ID | 任务类型 | 触发短语 |
|----|---------|---------|
| W1 | 新增业务页面（广场 / 详情 / 创建 / 我的） | "加一个 X 页面"、"做 Y 详情页"、"S13 的 Z 页面" |
| W2 | 新增 API 模块 | "对接 X 接口"、"调用后端 Y"、"加个请求" |
| W3 | 新增 / 修改 Pinia store | "把 X 状态存起来"、"做个全局 Y"、"按 store 重构" |
| W4 | 新增公共组件 | "封装一个 X 组件"、"做个复用的 Y"、"抽公共组件" |
| W5 | 新增 / 修改路由 | "加路由"、"改菜单"、"调跳转" |
| W6 | 新增 MSW mock | "前端先跑通"、"后端没好"、"补 mock" |
| W7 | 新增 / 修改类型 | "TS 类型对不上"、"接口改了"、"加类型" |
| W8 | 主题 / 样式调整 | "换颜色"、"暗色模式"、"全站调 X 风格" |
| W9 | 迁移 / 重构 | "把 X 挪到 Y"、"重构 Z" |
| W10 | 国际化（i18n） | "加翻译"、"多语言"、"改文案"、"新增语言"、新增任何用户可见文本 |

**没匹配上的任务**：不要硬套，回到本 SKILL.md 的"通用工作流"节走。

> ⚠️ **重要**：W1-W9 的任何 workflow 只要涉及用户可见文本（标题、按钮、提示、校验消息、路由 title 等），都必须**同时执行 W10 的文本国际化步骤**。W10 不是可选的独立 workflow，而是其他 workflow 的**伴生要求**。

---

## 3. 通用工作流（任何任务都先走一遍）

执行任何前端改动前，至少做完以下 4 步：

### Step 1 · 定位
1. 用 Read 打开 `docs/frontend-engineering-plan.md`，确认任务在改造方案里的位置（哪个目录、哪个文件、哪个阶段）。
2. 用 Glob/Grep 检查 `frontend/src/` 下是否已有相关文件，**优先 Edit 已有文件而不是新建**。
3. 如果是新文件，**对照 `references/conventions.md` 的"目录契约"表**确认放置位置。

### Step 2 · 类型先行 + 文本先行
1. 涉及接口的，先在 `src/types/api/<域>.ts` 加类型。
2. 涉及组件 Props/Emits 的，先在 `<script setup>` 顶部用 `defineProps<T>()` / `defineEmits<T>()` 声明。
3. 涉及 store state 的，先在 store 里用接口标注 state 形状。
4. **涉及用户可见文本的，先在 `src/i18n/locales/zh-CN.json`（和 `en-US.json`）中添加 key-value**，按 `references/conventions.md` §10 的命名规范命名。先写 key，再在代码中引用——不要先写硬编码中文"回头再改"。

### Step 3 · 实施
1. 优先复用 `templates/` 下的模板（cp 一份再改），不要"白板写"。
2. 严格遵守第 1 节的 9 条铁律。
3. 命令式 import 路径一律走 `@/` 别名，不写相对路径上下钻 2 层以上。
4. **所有用户可见文本走 `$t('key')` / `t('key')`**，具体用法见 `references/conventions.md` §10 和 `references/workflows.md` W10。

### Step 4 · 自检（在交还控制权前）
执行以下检查，**全部通过**才算完成：

```bash
cd frontend
npm run type-check     # vue-tsc 零错误
npm run lint           # ESLint 零错误
npm run lint:style     # Stylelint 零错误
npm run build          # 必须通过
```

5. 国际化检查：`grep -rn '[一-鿿]' src/ --include='*.vue' --include='*.ts' --exclude-dir='mocks' | grep -v '.json' | grep -v '/i18n/' | grep -v '// i18n-ignore'` 应**无命中**。排除范围说明：`src/mocks/` 目录（mock 业务模拟数据允许中文）、i18n locale 文件、以及标注了 `// i18n-ignore` 的行（需在注释中说明原因，如品牌名/专有名词）。

无法通过时：**不要降级规则**（不要加 `// @ts-ignore`、`eslint-disable-next-line` 来绕），先修代码再说。确实需要例外的，在同行注释里写明原因。

---

## 4. 模板使用指南

`templates/` 下提供 5 个可直接复用的骨架。**首选复用，不要重新打字。**

| 模板 | 用途 | 何时用 |
|------|------|--------|
| `templates/view-square.vue.tmpl` | 广场页（列表 + 筛选 + 搜索） | W1 中"X 广场"形态 |
| `templates/view-detail.vue.tmpl` | 详情页（Tabs 容器） | W1 中"X 详情" |
| `templates/api-module.ts.tmpl` | API 模块（CRUD 五件套） | W2 |
| `templates/store-module.ts.tmpl` | Pinia store（state/getters/actions） | W3 |
| `templates/mock-handler.ts.tmpl` | MSW handler（list/detail/create） | W6 |

用法：
```bash
cp docs/skill/ys-agent-frontend-dev/templates/api-module.ts.tmpl \
   frontend/src/api/modules/<domain>.ts
# 把模板里所有 __PLACEHOLDER__ 改成实际值
```

模板里以 `__XXX__` 开头的占位符**必须全部替换**，留任何一个未替换的占位符提交 = 视为未完成。

> **`api-module.ts.tmpl` 专项提示**：模板 import 行为 `import { __SERVICE_CLIENT__ } from '@/api/services'`，替换时填入该域名对应的 client 名称（`skillClient` / `mcpClient` / `agentClient` 等），并在 `src/api/services/index.ts` 确认该 client 已声明。

---

## 5. 反模式（看到立即矫正）

完整清单见 `references/anti-patterns.md`，高发的前 5 条放在这里以便速查：

| 反模式 | 正确做法 |
|--------|---------|
| 组件里 `import axios` | 走 `import { skillApi } from '@/api/modules/skill'`，或调 store action |
| `style` 里写 `#4a90d9` 等魔法色值 | 改用 `$color-primary` / `var(--el-color-primary)` |
| Element Plus 手动 `import { ElButton }` | 直接在模板用 `<el-button>`，auto-import 已配置 |
| `el-button :icon` 用 `resolveComponent('Check')` | `@element-plus/icons-vue` 未全局注册，会静默不渲染；改用 `import { Check } from '@element-plus/icons-vue'` 后直接传值 `:icon="Check"` |
| `onMounted` 里 `if (list.length === 0) fetch()` | 混淆了"数据保鲜"与"防并发重复"；视图层应无条件调 fetch，防并发去重在 store action 里用 `if (loading) return` 实现（见 A28） |
| 接入真实接口时删除"后端没有"的原型表单字段 | 视觉设计与数据提交应解耦：表单字段由原型决定，提交字段由白名单映射函数决定；后端暂不支持的字段保留在表单、不发送即可（见 A29） |
| 新增页面忘了挂菜单 | 路由 `meta` 加 `title`（i18n key）即可，菜单自动生成；不要去 SideMenu.vue 加 |
| 新增 store 后忘了在视图用 `storeToRefs` | 解构出来的 ref 会丢响应式，`storeToRefs(store)` 是默认姿势 |
| 模板 / ElMessage 里硬编码中文 | 走 `$t('key')` / `t('key')`，先在 locale JSON 加 key 再引用 |
| 所有 API 模块共用同一 client（`import { http } from '@/api/request'`） | 按微服务归属 import 对应 client：`skillClient` / `mcpClient` / `agentClient`（见 A25） |
| 接口出错后自己 `ElMessageBox.alert(...)` / `ElMessage.error(...)` | 让拦截器统一处理；需静默时传 `{ errorMode: 'silent' }`；主动推弹窗走 `useErrorDialogStore().push()`（见 A26） |

---

## 6. 上下文参考

以下文件按需 Read，**不要一开始就全部读**，按当前任务需要读：

| 文件 | 读取时机 |
|------|---------|
| `docs/frontend-engineering-plan.md` | 任何任务都至少扫一眼第 4、6、7、10 节 |
| `docs/frontend-i18n-plan.md` | 涉及国际化、文本变更、新增语言时 |
| `docs/sessions/S13-square-frontend.md` | 涉及业务页面时（广场/详情/创建/我的） |
| `references/conventions.md` | 不确定放置位置 / 命名 / 引入方式 / i18n key 命名时 |
| `references/workflows.md` | 任务匹配到 W1-W10 时 |
| `references/anti-patterns.md` | 写完一段后回头自查 / code review 时 |
| `templates/*.tmpl` | 真正动手实施时 |

---

## 7. 验收门控（Definition of Done）

任何前端改动在交付（commit / PR）前必须满足：

1. 自检命令全绿（见 §3 Step 4，含 i18n 硬编码扫描）
2. 新增的接口在 `mocks/handlers/` 里有对应 handler
3. 新增 / 修改的 view 在浏览器实测：
   - `npm run dev:mock` 启动后可访问
   - 路由跳转、搜索、筛选、详情、提交均有响应
4. **新增的用户可见文本在 `zh-CN.json` 和 `en-US.json` 中都有对应 key**
5. commit message 用 `S{编号}: 简述` 或 `frontend: 简述`
6. 不留 `console.log`、不留 `// TODO: ...`（除非配 issue 链接）
7. 不留未替换的模板占位符（grep `__[A-Z_]\+__` 应无命中）

---

## 8. 不在本 skill 管辖范围内的事

为避免 scope 蔓延，以下事不在本 skill 处理范围，遇到时**先停下找用户确认**：

- 调整 `frontend/` 之外的代码（Java 后端、CLI、MCP server）
- 引入 `package.json` 里没列过的新重量级依赖（任何新依赖需 PR review）
- 修改根 `pom.xml` / 后端 `application.yml`
- 删除 `src/views/playground/` 下的任何文件
- 调整 ys-agent 的核心架构（AgentLoop、ToolExecutor 等）

---

## 9. 自我检查（每次任务结束前问自己 6 个问题）

1. 我新建的文件是否完全符合 §1 第 1 条"目录契约"？
2. 我写的代码是否引入了 §5 任何一条反模式？
3. 我是否补了对应的 mock handler、可以 `npm run dev:mock` 单独跑通？
4. `type-check / lint / lint:style / build` 是否全绿？
5. 我是否在 commit 前把所有 `__PLACEHOLDER__`、`console.log`、`// TODO` 都清理了？
6. **我新增的用户可见文本是否全部走了 i18n（`$t` / `t`），并在 `zh-CN.json` 和 `en-US.json` 中都添加了对应 key？**

6 个全 yes 才能交付。任何一个 no，回到第 3 节通用工作流重新跑。
