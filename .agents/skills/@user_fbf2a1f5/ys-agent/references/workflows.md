# 标准工作流（Workflows）

> 10 类常见前端任务的"逐步操作清单"。**找到对应 workflow，照单全收**。
> 跨多个 workflow 的复合任务：拆成多个子任务，依次执行；不要并行执行造成相互冲突。

---

## W1 · 新增业务页面

适用：广场页 / 详情页 / 创建页 / 我的 X 页 等。

### W1.1 决定页面形态

| 形态 | 特征 | 模板 |
|------|------|------|
| 广场 (Square) | 列表 + 搜索 + 筛选 + 分页 | `templates/view-square.vue.tmpl` |
| 详情 (Detail) | Tabs / 卡片 / 单实体展示 | `templates/view-detail.vue.tmpl` |
| 创建 / 编辑 (Form) | 表单 + 提交 | 基于 `<el-form>` 自己起手 |
| 我的 (My) | 当前用户视角的双 Tab 列表 | 仿广场 + Tabs |

### W1.2 步骤

1. **类型先行**：在 `src/types/api/<domain>.ts` 添加 `XxxSummary` / `XxxDetail` / `XxxListQuery` 等类型。
2. **接口契约**：在 `src/api/modules/<domain>.ts` 添加方法（W2）。
3. **Store**：在 `src/stores/modules/<domain>.ts` 添加 state + action（W3）。
4. **页面文件**：
   - 路径：`src/views/<domain>/<XxxView>.vue`
   - 用 `templates/view-square.vue.tmpl` / `view-detail.vue.tmpl` 复制，替换占位符。
5. **挂路由**：在 `src/router/routes.ts` 的 children 数组追加一条，`meta` 至少含 `title`、可选 `icon`、`hideInMenu`。
6. **Mock handler**：补 `src/mocks/handlers/<domain>.ts`（W6）。
7. **浏览器实测**：`npm run dev:mock`，访问新路径，验证：
   - 列表能渲染
   - 搜索 / 筛选有反应
   - 跳详情 / 提交表单流程通
8. **自检命令**：`type-check && lint && lint:style && build` 全绿。

### W1.3 易错点
- 把"只在本视图用的小卡片组件"放到了 `src/components/common/` → 应该在 `src/views/<domain>/components/`
- 路由 `meta.title` 漏写 → 浏览器 tab 标题变成 undefined
- 详情页用 `<el-tabs>` 但忘了用 `<keep-alive>` 包 Tab 面板，切换 Tab 状态丢失

---

## W2 · 新增 API 模块

适用：对接后端任何一个新接口。

### W2.1 步骤

1. **类型**：在 `src/types/api/<domain>.ts` 加 `Request` / `Response` / 业务实体类型。`Response` 是 axios 拦截器解包后的实际数据形状（不含 `code` / `msg`）。
2. **模块文件**：
   - 路径：`src/api/modules/<domain>.ts`
   - 用 `templates/api-module.ts.tmpl` 复制。
   - 导出对象 `<domain>Api`，方法名动词驼峰：`list / detail / create / update / remove / search / upload`。
3. **使用**：只在 Pinia store action 里 import；视图层禁止 import api 模块。

### W2.2 模板结构（节选）

```ts
import { skillClient } from '@/api/services'
import type { SkillSummary, SkillDetail, SkillListQuery, SkillCreatePayload, Paged } from '@/types/api/skill'

export const skillApi = {
  list:   (params: SkillListQuery) => http.get<Paged<SkillSummary>>('/skills', { params }),
  detail: (id: string)             => http.get<SkillDetail>(`/skills/${id}`),
  create: (body: SkillCreatePayload) => http.post<SkillDetail>('/skills', body),
  remove: (id: string)             => http.delete<void>(`/skills/${id}`),
}
```

### W2.3 易错点
- 在 `Response` 类型里又包了一层 `{ code, data, msg }` → 拦截器已解包，类型只描述 `data` 形状
- 用 axios 默认实例 → 必须用 `src/api/client.ts` 导出的 service client
- 在 store 外（如组件、composable）直接调 api → 违反三层职责

---

## W3 · 新增 / 修改 Pinia Store

### W3.1 步骤

1. 路径：`src/stores/modules/<domain>.ts`
2. 用 `templates/store-module.ts.tmpl` 复制。
3. setup store 风格：
   ```ts
   export const useSkillStore = defineStore('skill', () => {
     const list = ref<SkillSummary[]>([])
     const loading = ref(false)
     const filters = reactive<SkillListQuery>({ q: '', type: 'all' })
     async function fetchList() {
       loading.value = true
       try { list.value = (await skillApi.list(filters)).items }
       finally { loading.value = false }
     }
     return { list, loading, filters, fetchList }
   })
   ```
4. 持久化（如需）：在 `defineStore` 的第二个参数加 `{ persist: true }`，全局插件已注册。
5. 注册：无需手动；在视图调用 `useSkillStore()` 即可。

### W3.2 易错点
- 把 list 定义成 `reactive([])` → 后续 `list = []` 会丢响应式；用 `ref([])` 通过 `.value` 赋值
- store 直接持有 axios → 不允许；走 api 模块
- store 名重复（`'skill'` 用了两次）→ Pinia 会合并，造成奇怪 bug；保持唯一

---

## W4 · 新增公共组件

### W4.1 决定放哪里

- 同一组件在 **2 个以上 `views/<域>`** 用到 → 必须升到 `src/components/common/`
- 只在一个 `views/<域>` 内部用 → 留在 `src/views/<域>/components/`
- 业务无关的"纯展示"原子组件 → `src/components/common/`

### W4.2 步骤

1. 文件：`src/components/common/<Xxx>.vue`（自动注册，模板里直接用）
2. Props / Emits 用泛型声明，必填项必须显式标记
3. 写一个最小可用的"使用示例"作为顶部注释：
   ```vue
   <!--
   <SearchBox v-model="keyword" placeholder="搜索 Skill" :debounce="300" />
   -->
   ```
4. 样式 `<style scoped lang="scss">`，颜色用变量
5. 在用到的视图里直接 `<SearchBox ...>`，**不要手动 import**（unplugin-vue-components 已配）

### W4.3 易错点
- 漏加 scoped → 全局污染
- props 用 boolean 又给默认 true → 反直觉，默认应为 false 或不设
- 复杂状态硬塞进父子双向绑定 → 抽 composable

---

## W5 · 新增 / 修改路由

### W5.1 步骤

1. 仅改 `src/router/routes.ts`。**不要**改 `index.ts`（守卫定义）/ `guards.ts`，除非任务专门是改守卫。
2. 在 AppLayout 的 children 数组里追加：
   ```ts
   {
     path: 'tools/usage',
     name: 'ToolUsage',
     component: () => import('@/views/tool/ToolUsageView.vue'),
     meta: { title: 'tool.usage.title', icon: 'TrendCharts' },
   }
   ```
3. `meta` 必填：`title`。常用可选：`icon`（el-icon 名称）、`hideInMenu`、`requireAuth`（默认 true）、`keepAlive`。
4. 菜单和面包屑会自动生成，无需手动改 SideMenu / Breadcrumb。
5. 跳转：`router.push({ name: 'ToolUsage' })`（用 name，不用 path 字符串拼接）。

### W5.2 易错点
- 静态 import 组件 → 走 lazy（`() => import(...)`）保持分包
- `path` 开头加了 `/` → 嵌套路由内 children 不要前导斜杠
- 把详情页的 path 写成 `/skills/detail/:id` → 不符合 RESTful 习惯，应为 `/skills/:id`

---

## W6 · 新增 MSW Mock

### W6.1 何时必须写 mock

任何**前端先于后端**或**后端 API 还没冻结**的接口对接，必须先写 mock，否则 `npm run dev:mock` 起不来流程。

### W6.2 步骤

1. handler：`src/mocks/handlers/<domain>.ts`，用 `templates/mock-handler.ts.tmpl`
2. 假数据：`src/mocks/data/<domain>.ts`
3. 注册：`src/mocks/handlers/index.ts` 把 handler 数组聚合导出
4. 在 `npm run dev:mock` 下访问，浏览器 Network 应看到被 worker 拦截（`x-powered-by: msw`）

### W6.3 模板（节选）

```ts
import { http, HttpResponse } from 'msw'
import { mockSkills } from '../data/skill'

export const skillHandlers = [
  http.get('/api/v1/skills', ({ request }) => {
    const url = new URL(request.url)
    const q = url.searchParams.get('q') ?? ''
    const items = mockSkills.filter(s => s.name.includes(q))
    return HttpResponse.json({ code: 0, msg: 'ok', data: { items, total: items.length } })
  }),
  http.get('/api/v1/skills/:id', ({ params }) => {
    const item = mockSkills.find(s => s.id === params.id)
    return item
      ? HttpResponse.json({ code: 0, msg: 'ok', data: item })
      : new HttpResponse(null, { status: 404 })
  }),
]
```

### W6.4 易错点
- 直接返回 data，没包 `{ code, msg, data }` → 拦截器会判错
- 路径写绝对 `https://api.xxx/...` → 走代理路径 `/api/v1/...`
- 改了 handler 没刷新浏览器 → MSW worker 需要硬刷新

---

## W7 · 新增 / 修改 TypeScript 类型

### W7.1 放哪里

| 类型 | 位置 |
|------|------|
| 接口请求 / 响应 | `src/types/api/<domain>.ts` |
| 业务实体 | 同上文件 |
| 全局通用（如 `Paged<T>`） | `src/types/common.ts` |
| 组件 Props 类型 | 紧挨组件，单独 `interface` 即可 |
| 环境变量 | `src/types/env.d.ts` |

### W7.2 步骤

1. 先看 `src/types/api/<domain>.ts` 是否已有相关类型，**优先扩展**
2. 命名：`XxxRequest` / `XxxResponse` / `XxxSummary`（列表项）/ `XxxDetail`（详情）
3. 不要把 `any` 当占位 → 用 `unknown` + 类型守卫
4. 改完后跑 `npm run type-check`，所有调用点都通过编译

### W7.3 易错点
- API 类型用 `class` → 用 `interface` 或 `type`
- 在多个 store 里复制粘贴同样的类型 → 集中到 `types/api/`

---

## W8 · 主题 / 样式调整

### W8.1 范围分类

| 任务 | 改哪里 |
|------|--------|
| 调主色 / 圆角 / 字号 | `src/styles/element-overrides.scss` |
| 加 / 改自定义变量（间距、阴影、自有色） | `src/styles/variables.scss` |
| 全站重置（去 margin、字体） | `src/styles/reset.scss` |
| 单页布局微调 | 视图自身 `<style scoped>` |
| 多页面共用样式 | 抽到 `src/styles/mixins.scss` 提供 mixin |

### W8.2 步骤

1. 判断范围，定位文件
2. 改完用 `git diff` 看影响面，必要时把变更面控制在最小
3. `npm run dev:mock` 抽 2-3 个有代表性页面（广场、详情、表单）目测一遍
4. `npm run lint:style` 通过

### W8.3 易错点
- 改了 `variables.scss` 但忘了 vite 已通过 `additionalData` 注入 → 直接用变量即可，不要再 `@use`
- 改 Element 主题用 CSS 变量但漏了 element-overrides 入口 → 走 SCSS `@use 'element-plus/.../var.scss' with (...)` 方案，CSS 变量是衍生输出

---

## W9 · 迁移 / 重构

### W9.1 适用场景

- 把旧 demo 代码搬到新目录（典型：现有 chat demo → `src/views/playground/`）
- 把一个组件从 `views/<域>/components/` 升级到 `components/common/`
- 拆解 god component

### W9.2 步骤

1. 在开干前列清单：哪些文件移动、哪些重命名、哪些 import 需要改
2. 一次只做一类迁移，commit 之间保证 build 通过
3. 移动时用 `git mv` 保留历史
4. 全局搜旧 import 路径（`@/components/sidebar/...`），改为新路径
5. 移动后跑 `type-check` + `build`，目测旧入口不再可达、新入口可用
6. 在 commit message 写清楚 from → to，便于他人定位

### W9.3 易错点
- 复制粘贴 + 删原文件 → 失去 git 历史；务必 `git mv`
- 移动后忘改 alias 别名 → 找不到模块；全局搜确认
- 把"自带状态"的组件升级到 common 但忘记把状态外置 → 跨页污染

---

## W10 · 国际化（i18n）

适用：新增任何用户可见文本、修改现有文案、新增语言包。

> ⚠️ **W10 是伴生 workflow**：W1-W9 中只要涉及用户可见文本，都必须同时执行本节步骤。不要"先写硬编码，回头再改"。

### W10.1 新增用户可见文本（最常见场景）

1. **确定 key 命名**：按 `conventions.md` §10.2 的 `{module}.{scope}.{semantic}` 规范。先在 `zh-CN.json` 中找相邻 key，保持命名风格一致。
2. **写入 locale 文件**：
   - 在 `src/i18n/locales/zh-CN.json` 中添加 key + 中文值
   - 在 `src/i18n/locales/en-US.json` 中添加相同 key + 英文值
   - 两个文件**同时改**，key 结构完全对齐
3. **代码中引用**：
   - 模板：`{{ $t('key') }}` 或 `:prop="$t('key')"`
   - script：`const { t } = useI18n()` → `t('key')`
   - 非组件 TS：`import i18n from '@/i18n'` → `i18n.global.t('key')`
4. **带参数的文本**：JSON 中用 `{name}` 占位，调用时传对象 `t('key', { name: value })`
5. **自检**（命令与 SKILL.md §3 Step 4 保持一致）：
   ```bash
   grep -rn '[一-鿿]' src/ --include='*.vue' --include='*.ts' --exclude-dir='mocks' | grep -v '.json' | grep -v '/i18n/' | grep -v '// i18n-ignore'
   ```
   确认无新增硬编码中文。`src/mocks/` 中的业务模拟数据和标注了 `// i18n-ignore` 的品牌/专有名词除外。

### W10.2 修改现有文案

1. 找到对应的 i18n key（在 locale JSON 中搜中文值）
2. 修改 `zh-CN.json` 和 `en-US.json` 中的值
3. **不要改 key 名**（除非语义变了），因为其他地方可能引用

### W10.3 新增语言

1. 在 `src/i18n/locales/` 下新建 `<lang>.json`（如 `id-ID.json`），结构完全复制 `zh-CN.json`
2. 在 `src/i18n/index.ts` 中 import 并注册到 `messages`
3. 在 `App.vue` 的 `elementLocale` computed 中加一个分支
4. 在语言切换 UI 中加一个选项

### W10.4 路由 meta.title 和 meta.group 的 i18n

路由 `meta.title` 和 `meta.group` **必须**存储 i18n key（不是中文文本）：

```ts
// ✅ 正确
{ path: 'skills', meta: { title: 'skill.square.title' } }
// group 默认为 'nav.group.square'，需要分到其他组时显式指定：
{ path: 'my/skills', meta: { title: 'my.skills.title', group: 'nav.group.my' } }

// ❌ 错误
{ path: 'skills', meta: { title: 'Skill 广场' } }
{ path: 'my/skills', meta: { title: '我的 Skill', group: '我的' } }
```

路由守卫（`guards.ts`）负责在运行时翻译 title。菜单组件（`SideMenu.vue`）从 `meta.title` 和 `meta.group` 读 key 后均通过 `$t()` 翻译展示。

### W10.5 组件 props 默认值

**不要**在 `defineProps` 的默认值里调 `t()`（时序问题），改为模板侧兜底：

```vue
<!-- ✅ 正确 -->
<template>
  <span>{{ text || $t('common.empty.default') }}</span>
</template>

<!-- ❌ 错误 -->
<script setup>
withDefaults(defineProps<{ text?: string }>(), {
  text: t('common.empty.default')  // 此时 useI18n 可能未就绪
})
</script>
```

### W10.6 易错点

- 只改了 `zh-CN.json` 没改 `en-US.json` → 切换英文时显示 fallback 中文或 key 名
- 在 `defineProps` 默认值里调 `t()` → 运行时报错或翻译不生效
- 路由 `meta.title` 存中文而非 i18n key → 语言切换后菜单/标签页标题不变
- 忘了在 ElMessage / ElMessageBox 的文案中用 `t()` → 提示信息永远中文
- key 命名不遵守 `module.scope.semantic` 规范 → 后续难以维护、key 冲突
- 复用了含义不同的同名 key（如两个页面的"标题"虽然都叫 title 但含义不同）→ 改一个影响另一个
