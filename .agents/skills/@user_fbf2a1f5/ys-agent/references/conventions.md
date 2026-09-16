# 工程约定（Conventions）

> 本文档是 SKILL.md 的延伸，定义"放哪里 / 叫什么 / 怎么写"。**遇到分歧以本文档为准**，未覆盖的情况回到 `docs/frontend-engineering-plan.md`。

---

## 1. 目录契约（必查表）

新增 / 移动文件前先在表里找到正确位置。**找不到对应行 = 暂停动手 + 问用户**。

| 我要做的事 | 唯一正确位置 | 文件命名 |
|-----------|-------------|----------|
| 调用某后端接口 | `src/api/modules/<domain>.ts` | 文件名小驼峰；导出对象 `<domain>Api` |
| 定义请求 / 响应类型 | `src/types/api/<domain>.ts` | 同 domain，导出接口名 `XxxRequest / XxxResponse / XxxSummary / XxxDetail` |
| 加一个全局状态 | `src/stores/modules/<domain>.ts` | 函数 `useXxxStore` |
| 加一个**业务页面** | `src/views/<domain>/XxxView.vue` | 文件以 `View.vue` 结尾 |
| 加一个**业务页面内部专用**子组件 | `src/views/<domain>/components/Xxx.vue` | PascalCase，不带 View 后缀 |
| 加一个**跨业务公共组件** | `src/components/common/Xxx.vue` | PascalCase；自动注册，不需手动 import |
| 加一个 composable hook | `src/composables/useXxx.ts` | 必须 `use` 前缀 |
| 加一个工具函数 | `src/utils/<topic>.ts` | 例：`format.ts`、`storage.ts`；纯函数 |
| 加一条路由 | `src/router/routes.ts` | 在合适 children 数组追加；不要新建文件 |
| 加一个 mock handler | `src/mocks/handlers/<domain>.ts` | domain 与 api 模块对齐 |
| 加 mock 假数据 | `src/mocks/data/<domain>.ts` | 导出 `mockXxx` |
| 接入新微服务 / 修改服务地址或 adapter | `src/api/services/index.ts` | 每个后端微服务一个 client 声明 |
| 修改 axios 公共拦截器 / 错误分发逻辑 | `src/api/client.ts` | 工厂函数 + adapters + dispatchError |
| 展示全局接口异常弹窗 | 调用 `useErrorDialogStore().push(event)` | 不要自己造 `ElMessageBox` |
| 加全局样式 / 变量 | `src/styles/variables.scss` / `index.scss` | 改已有文件，不要新建 |
| 加 Element 主题覆盖 | `src/styles/element-overrides.scss` | 唯一允许覆盖 Element 主题的地方 |
| 加 SVG 图标 / 静态图 | `src/assets/images/`（图） | 文件名 kebab-case |
| 加自定义指令 | `src/directives/<name>.ts` | 例：`permission.ts` |
| 加 / 改翻译文本 | `src/i18n/locales/zh-CN.json` + `en-US.json` | 两个文件**同时改**，key 结构完全对齐 |
| i18n 配置 / 插件 | `src/i18n/index.ts` | 唯一的 i18n 实例创建处 |

**禁止**：
- 在 `src/components/` 根目录建文件（必须落到 `common/` 或 `icons/`）
- 在 `views/` 根目录建 `.vue` 文件（必须先落到某个域子目录）
- 把"只在某个页面用一次"的组件放到 `components/common/`
- 把"跨多个页面复用"的组件放到 `views/<域>/components/`
- 在 `.vue` / `.ts` 文件中硬编码用户可见的中文或英文文本（必须走 `$t()` / `t()`）
- 只改 `zh-CN.json` 不改 `en-US.json`（两个文件的 key 必须同步）

---

## 2. 命名约定

| 对象 | 风格 | 示例 |
|------|------|------|
| Vue 组件文件 | PascalCase，view 加 `View` 后缀 | `SkillSquareView.vue`、`SkillCard.vue` |
| TS / JS 模块文件 | camelCase | `skillApi.ts`、`useTheme.ts` |
| TS 类型 / 接口 | PascalCase | `SkillSummary`、`McpModuleDetail` |
| 常量 | UPPER_SNAKE | `DEFAULT_PAGE_SIZE` |
| 变量 / 函数 | camelCase | `currentSkill`、`fetchSkillList` |
| Pinia store id | kebab-case，与文件名匹配 | `'skill'`、`'mcp-module'` |
| 路由 name | PascalCase | `'SkillSquare'`、`'McpDetail'` |
| 路由 path | kebab-case | `/skills`、`/my/skills` |
| CSS 类 | kebab-case，按 BEM 风格 | `.skill-card`、`.skill-card__title` |
| SCSS 变量 | `$kebab-case` | `$color-primary`、`$spacing-md` |
| 自定义事件 | kebab-case | `@update:value`、`@select-change` |
| Props | camelCase（模板用 kebab） | `<SkillCard :skill-id="..." />` |
| i18n key | `module.scope.semantic`，每段 camelCase | `skill.square.title`、`auth.login.validation.accountRequired` |

---

## 3. 代码风格

### 3.1 Vue 组件
- 一律 `<script setup lang="ts">`，禁止 Options API。
- 顺序：`<script setup>` → `<template>` → `<style scoped lang="scss">`。
- Props 与 Emits 都用泛型声明：

  ```ts
  const props = defineProps<{
    skill: SkillSummary
    selected?: boolean
  }>()
  const emit = defineEmits<{
    (e: 'select', id: string): void
  }>()
  ```
- 不要混用 `defineProps` 的 runtime 写法与泛型写法。
- 默认值用 `withDefaults(defineProps<...>(), { ... })`。

### 3.2 Pinia
- 一律 setup store 风格（函数返回对象），不用 options store。
- 状态用 `ref` / `reactive`；计算属性用 `computed`；操作是普通 async function。
- 视图侧解构必须 `storeToRefs(store)` 包裹响应式数据，function 直接从 store 上取。

### 3.3 TypeScript
- `tsconfig.app.json` 的 strict 选项保持开启。
- 公共类型放 `src/types/`，业务局部类型放紧挨着使用处的同名 `.ts`。
- 严格禁止 `any`，需要时优先用 `unknown` 然后类型守卫。
- API 响应类型必须从 `src/types/api/` 导入，不要在 store / view 里重新声明。

### 3.4 Import 顺序
1. node / vite 内置（`'vue'`、`'pinia'`、`'vue-router'`）
2. 第三方（`'element-plus'`、`'axios'`、...）
3. `@/api`、`@/stores`、`@/composables`、`@/utils`
4. `@/types`
5. 相对路径（`./` / `../`）

每组之间空行。Prettier 不自动整理 import，**人工保持秩序**。

### 3.5 路径
- `import` 路径**禁止** `../../`；向上一层用 `../`，向上 ≥ 2 层一律 `@/...`。

---

## 4. Element Plus 用法约定

| 项 | 约定 |
|----|------|
| 组件引入 | 模板里直接 `<el-button>`，**不要** `import { ElButton }`；按需引入由 unplugin 处理 |
| 图标 | `import { Edit } from '@element-plus/icons-vue'`，模板里 `<el-icon><Edit /></el-icon>` |
| ElMessage / ElMessageBox / ElNotification | 在 ts 里 `import { ElMessage } from 'element-plus'`（这是函数，必须显式 import） |
| 表单校验 | 用 `<el-form :rules>` 内置校验，复杂校验抽到组件同目录的 `validators.ts` |
| 弹窗 | 用 `<el-dialog v-model:visible>`；禁止用全局函数式 dialog 拼业务 |
| 按钮 type | 主操作 `type="primary"`，危险操作 `type="danger"`，次操作不设 type |
| 表格 | 优先 `<el-table>` + `<el-pagination>` 组合；通用分页用 composable `usePagination` |
| 主题色 | 通过 `<style>` 里 `var(--el-color-primary)` 引用，**不要写死颜色** |

---

## 5. 样式约定

- 组件 `<style>` 必须加 `scoped`，例外要在 PR 说明。
- 全局样式只放在 `src/styles/index.scss`（重置）+ `element-overrides.scss`（Element token）。
- 业务组件**禁止**：
  - 写死颜色（`#xxxxxx`、`rgb(...)`、`rgba(...)`）
  - 写死间距常数（`padding: 12px` 这种 ok，但 `margin: 24px 32px 16px 24px` 等"四个魔法值"需要抽成变量）
- `@/styles/variables.scss` 已通过 vite `additionalData` 自动注入到所有 SCSS 文件，**不要在组件里再 `@use`**。
- 暗色模式预留：颜色全部走 CSS 变量（`var(--el-color-*)`），主题色覆盖在 `element-overrides.scss`，业务组件无需关心。

---

## 6. API / Store / View 三层职责

```
┌── View ──────────────────────────────────────┐
│  只读 store.state / 调 store.action          │
│  禁止 import axios、禁止直接 await api.xxx() │
└──────────────────────────────────────────────┘
                    ↓
┌── Store (Pinia) ─────────────────────────────┐
│  持有 state、暴露 actions                    │
│  action 内 await xxxApi.yyy()，处理 loading  │
│  状态错误 / 业务错误转换为可消费的 state     │
└──────────────────────────────────────────────┘
                    ↓
┌── API (axios) ───────────────────────────────┐
│  纯请求；只关心 URL / method / params / body │
│  返回类型来自 src/types/api/                 │
└──────────────────────────────────────────────┘
```

**典型违反**：
- View 里 `const data = await skillApi.list()` → 应该改为 store action 并在 store 里维护 list state
- Store 里 `import axios` 自己造请求 → 应该走 `src/api/client.ts` 的 service client

---

## 7. 环境变量

- 所有运行时配置走 `import.meta.env.VITE_XXX`，必须在 `src/types/env.d.ts` 加类型。
- `.env` 里只放公共默认；`.env.development` 与 `.env.production` 各管各的。
- 不要在代码里写 `process.env`（vite 不支持）。

---

## 8. Git 与提交

- 分支：`feat/S{编号}-{slug}` 或 `feat/frontend-{slug}`
- commit message：`S{编号}: 简述` 或 `frontend: 简述`
- 一个 PR 一个主题；不混合"加一个页面 + 重构 store + 改路由"
- frontend 改动 PR 描述里必须勾选：
  - [ ] `npm run type-check` 通过
  - [ ] `npm run lint` 通过
  - [ ] `npm run lint:style` 通过
  - [ ] `npm run build` 通过
  - [ ] 浏览器实测（`npm run dev:mock`）通过

---

## 9. 兼容性预设

- 目标浏览器：Chrome / Edge / Safari 近 2 年版本；不考虑 IE。
- 移动端：响应式适配最小 768px 宽度；后台管理系统不强求手机良好体验。
- Node：本地开发 ≥ 20.x。

---

## 10. 国际化（i18n）约定

> 详细技术方案见 `docs/frontend-i18n-plan.md`。本节是 agent 日常开发时的速查规则。

### 10.1 核心规则

- **所有用户可见文本必须走 i18n**，包括：模板文本、按钮、标题、描述、placeholder、ElMessage 提示、表单校验 message、路由 meta.title、组件 props 默认值展示文本。
- **例外情况**（不走 i18n，但需在代码行尾加 `// i18n-ignore` 注释并简述原因）：
  - mock 数据中的业务模拟文本（如虚构的 skill 名称 `'SQL 查询助手'`），位于 `src/mocks/` 目录
  - **品牌名和产品固有专有名词**：`ys-agent`、`Skill`、`MCP`、`Tool`、`Playground` 等——这些是产品术语，各语言保持原文不翻译，可以直接硬编码
  - 纯技术标识：日志 tag（如 `'[mock]'`）、API path、CSS 类名等
- 翻译文件位于 `src/i18n/locales/zh-CN.json` 和 `en-US.json`，**两个文件的 key 结构必须完全同步**。

### 10.2 Key 命名规范

格式：`{module}.{scope}.{semantic}`

| 层级 | 说明 | 常用值 |
|------|------|--------|
| module | 业务域 | `common`、`auth`、`skill`、`mcp`、`tool`、`chat`、`nav`、`error`、`employee`、`my` |
| scope | 页面/组件/功能块 | `login`、`square`、`detail`、`header`、`sidebar`、`form`、`validation`、`filter`、`action` |
| semantic | 具体语义 | `title`、`description`、`placeholder`、`submitBtn`、`emptyText`、`successMsg`、`errorMsg` |

**命名禁忌**：
- 禁止中文 key、纯数字 key
- 禁止超过 4 级嵌套
- 禁止复用同一 key 表达不同含义
- key 段用 camelCase

**示例**：
```
skill.square.title         → "Skill 广场"
auth.login.validation.accountRequired → "请输入账号"
common.error.network       → "网络错误，请稍后重试"
nav.header.logout          → "退出登录"
```

### 10.3 各场景用法

**模板中**：
```vue
<h1>{{ $t('skill.square.title') }}</h1>
<el-input :placeholder="$t('chat.inputPlaceholder')" />
<EmptyState :text="$t('common.empty.default')" />
```

**`<script setup>` 中**：
```ts
import { useI18n } from 'vue-i18n'
const { t } = useI18n()
ElMessage.success(t('auth.login.successMsg'))
```

**表单校验规则**：
```ts
const rules = {
  account: [{ required: true, message: t('auth.login.validation.accountRequired'), trigger: 'blur' }],
}
```

**路由 meta.title**：
```ts
// routes.ts — 存 i18n key，不存中文
{ path: 'skills', meta: { title: 'skill.square.title' } }

// guards.ts — 守卫中翻译
import i18n from '@/i18n'
document.title = i18n.global.t(to.meta.title as string)
```

**非组件 TS 文件**（如 `client.ts`）：
```ts
import i18n from '@/i18n'
const t = i18n.global.t
ElMessage.error(t('common.error.network'))
```

**组件 props 默认值**——不在 `defineProps` 默认值里调 `t()`，改为模板侧兜底：
```vue
<template>
  <span>{{ text || $t('common.empty.default') }}</span>
</template>
```

### 10.4 新增文本的标准流程

1. 先在 `zh-CN.json` 中按命名规范添加 key 和中文值
2. 在 `en-US.json` 中添加相同 key 和英文值
3. 在代码中通过 `$t('key')` / `t('key')` 引用
4. 自检：grep 确认无新增硬编码中文

### 10.5 带参数的翻译

使用 `{name}` 占位符：
```jsonc
// zh-CN.json
"auth.login.validation.passwordMinLength": "密码长度不能少于 {min} 位"
```
```ts
t('auth.login.validation.passwordMinLength', { min: 4 })
```
