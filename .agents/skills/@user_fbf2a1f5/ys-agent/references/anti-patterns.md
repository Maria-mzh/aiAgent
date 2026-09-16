# 反模式清单（Anti-Patterns）

> 每条都是历史上真实踩过的坑或本项目刻意要避免的写法。**写代码前过一遍，code review 时再过一遍**。
> 每条都给"长什么样 / 为什么不行 / 正确写法"三段；如果你的改动符合任一条，必须先矫正再提交。

---

## A1 · 在组件 / composable 里直接 import axios

**反模式**：
```ts
// ❌ views/skill/SkillSquareView.vue
import axios from 'axios'
const { data } = await axios.get('/api/v1/skills')
```

**为什么不行**：
- 绕过统一的 baseURL、token 注入、错误处理拦截器，每个组件各写一套，token 一改全爆炸
- 视图直接依赖 HTTP 细节，无法 mock、无法测试
- 同样的接口会被不同组件重复 fetch，没有 store 做缓存

**正确写法**：
```ts
// ✅ src/api/modules/skill.ts (一次)
import { skillClient } from '@/api/services'
export const skillApi = { list: (p) => skillClient.get<Paginated<SkillSummary>>('/skills', { params: p }) }

// ✅ src/stores/modules/skill.ts
async function fetchList() { list.value = (await skillApi.list(filters)).items }

// ✅ views/skill/SkillSquareView.vue
const store = useSkillStore(); onMounted(store.fetchList)
```

---

## A2 · 硬编码颜色 / 间距

**反模式**：
```vue
<style scoped>
.card { background: #4a90d9; padding: 12px 24px 16px 24px; border: 1px solid #e0e0e0; }
</style>
```

**为什么不行**：
- 换肤 / 暗色模式无从下手
- 设计 token 不集中，过几个迭代各种近似色互相冲突
- 改一个色要全局 grep

**正确写法**：
```vue
<style scoped lang="scss">
.card {
  background: var(--el-color-primary);
  padding: $spacing-md $spacing-lg;
  border: 1px solid var(--el-border-color);
}
</style>
```

---

## A3 · 手动 import Element Plus 组件

**反模式**：
```ts
import { ElButton, ElInput, ElTable } from 'element-plus'
```

**为什么不行**：
- 项目已配置 `unplugin-vue-components` + `ElementPlusResolver`，手动 import 等于关闭按需机制（多打包代码）
- 类型已经全局可用，没必要重复 import

**正确写法**：模板里直接 `<el-button>`、`<el-input>`、`<el-table>`。
**例外**：`ElMessage` / `ElMessageBox` / `ElNotification` 是函数式 API，必须显式 import。

---

## A4 · 路由静态 import

**反模式**：
```ts
import SkillSquareView from '@/views/skill/SkillSquareView.vue'
{ path: 'skills', component: SkillSquareView }
```

**为什么不行**：所有 view 打到首屏 bundle，首次加载慢。

**正确写法**：
```ts
{ path: 'skills', component: () => import('@/views/skill/SkillSquareView.vue') }
```

---

## A5 · 手动维护菜单数据

**反模式**：在 `SideMenu.vue` 写一份 `const menuItems = [...]` 配菜单。

**为什么不行**：菜单与路由两份数据，必然漂移；新人加路由忘了改菜单。

**正确写法**：菜单**只能**从 `routes.ts` 派生。需要展示控制就用 `meta.hideInMenu` / `meta.icon` / `meta.title`。

---

## A6 · store 解构丢响应式

**反模式**：
```ts
const { list, loading } = useSkillStore()  // ❌ 此时 list / loading 已不是 ref
```

**为什么不行**：解构后是普通值，模板访问不会重新渲染。

**正确写法**：
```ts
import { storeToRefs } from 'pinia'
const store = useSkillStore()
const { list, loading } = storeToRefs(store)   // ✅ state 走 storeToRefs
const { fetchList } = store                     // ✅ action 直接解构 OK
```

---

## A7 · 用 `any` 兜底

**反模式**：
```ts
const data: any = await skillApi.detail(id)
data.foo.bar.baz  // 谁知道有没有
```

**为什么不行**：放弃了 TS 给的全部保护；接口一改不报错，运行时炸。

**正确写法**：先在 `types/api/<domain>.ts` 加上准确类型；不确定的字段用 `unknown` + 类型守卫，或加 `?`。

---

## A8 · 把"页面内组件"放到 `components/common/`

**反模式**：把 `SkillDetailVersionList.vue` 放到 `src/components/common/`。

**为什么不行**：`components/common/` 是**跨业务**复用区，混入业务组件会让"哪个组件可放心复用"的边界消失。

**正确写法**：业务组件放 `src/views/<域>/components/`；满足"≥2 个域用到"再升到 common。

---

## A9 · 直接改 `node_modules` / 改 Element Plus 源码

**反模式**：找不到 Element 主题色覆盖入口，去 `node_modules/element-plus/...` 改 SCSS。

**为什么不行**：下一次 `npm i` 直接覆盖回去；CI 永远复现不出本地效果。

**正确写法**：走 `src/styles/element-overrides.scss`：
```scss
@use 'element-plus/theme-chalk/src/common/var.scss' with (
  $colors: ('primary': ('base': #2563eb)),
);
```

---

## A10 · 留 `console.log` / `debugger` / 注释代码

**反模式**：提交里残留调试用 `console.log(...)`、`// console.log(...)` 一大堆。

**为什么不行**：污染生产 console；让 PR 噪音放大；后人不敢删怕有意义。

**正确写法**：提交前 `git diff` 自审，全部清掉。需要的日志走 utils 里统一封装的 logger。

---

## A11 · 在 vue 文件里 mix 业务 + 网络 + 渲染（god component）

**反模式**：`ChatView.vue` 里同时管 store、SSE 解析、Markdown 渲染、消息状态、UI——300 行起步。

**为什么不行**：单测无法做、复用无可能、bug 难定位。

**正确写法**：
- SSE 解析、消息拼装抽到 composable `useChatStream`
- Markdown 渲染抽到 `utils/markdown.ts`
- 状态搬到 store
- 视图只负责"绑数据 + 渲染 + 转事件"

> 但 demo 现有的 `ChatView.vue` 是过渡期资产，迁移到 playground 后允许保持现状，**不要重构成洁癖式**——见 SKILL.md §8 不在管辖范围。

---

## A12 · mock 返回值结构和真后端不一致

**反模式**：
```ts
return HttpResponse.json([{ id: 1 }, { id: 2 }])
```

**为什么不行**：拦截器期望 `{ code, msg, data }` 格式，直接拿到数组会全链路报错。

**正确写法**：
```ts
return HttpResponse.json({ code: 0, msg: 'ok', data: { items: [...], total: 2 } })
```

---

## A13 · 用相对路径上钻 ≥ 2 层

**反模式**：
```ts
import { SkillCard } from '../../../components/common/SkillCard.vue'
```

**为什么不行**：moves 时大面积破坏；可读性差。

**正确写法**：`import SkillCard from '@/components/common/SkillCard.vue'`。

---

## A14 · 重复造分页 / loading / debounce

**反模式**：每个广场页都自己写一遍 `currentPage / pageSize / total / loading` 与 fetch 逻辑。

**为什么不行**：N 份相似代码，bug 修 N 遍。

**正确写法**：抽 composable：`usePagination` / `useDebouncedRef`，统一行为。

---

## A15 · 把鉴权 / token 散落在多处

**反模式**：每个 view onMounted 自己读 localStorage 的 token 拼 header。

**为什么不行**：登出时清不干净；token 刷新逻辑无处下手。

**正确写法**：
- token 唯一来源：`useUserStore().token`（持久化）
- axios 请求拦截器统一注入 `Authorization`
- 401 拦截器统一清 token + 跳登录

---

## A16 · 漏 mock handler 导致 dev:mock 起不来

**反模式**：加了 api 模块和 store，却没补 `src/mocks/handlers/<domain>.ts`，`dev:mock` 下页面一片红。

**为什么不行**：违反"前端可独立运行"原则。

**正确写法**：W6 工作流；handler 一并提交。

---

## A17 · 用 path 字符串拼跳转

**反模式**：
```ts
router.push(`/skills/${id}`)
```

**为什么不行**：path 变了要全局改；类型也兜不住。

**正确写法**：
```ts
router.push({ name: 'SkillDetail', params: { id } })
```

---

## A18 · `<style>` 不加 scoped

**反模式**：
```vue
<style>
.card { ... }
</style>
```

**为什么不行**：全局污染，命名冲突难排查。

**正确写法**：`<style scoped lang="scss">`。极少数确实需要穿透时用 `:deep(...)`。

---

## A19 · 把第三方大库整体 import

**反模式**：`import _ from 'lodash'`、`import * as echarts from 'echarts'`。

**为什么不行**：tree-shaking 失效，bundle 暴涨。

**正确写法**：按需 `import debounce from 'lodash-es/debounce'`、`import { BarChart } from 'echarts/charts'` + 手动 use。

---

## A20 · 用 `v-html` 渲染未消毒内容

**反模式**：
```vue
<div v-html="markdownRaw"></div>
```

**为什么不行**：XSS 风险，特别是 Skill 的 system_prompt / description 来自用户输入时。

**正确写法**：走 `utils/markdown.ts` 封装的 markdown-it 实例（开启 html: false + 自定义白名单），或先 `DOMPurify.sanitize`。

---

## A21 · 硬编码用户可见文本（不走 i18n）

**反模式**：
```vue
<!-- ❌ 模板里硬编码中文 -->
<h1>Skill 广场</h1>
<el-button>+ 新建 Skill</el-button>
<EmptyState text="暂无数据" />
```
```ts
// ❌ ElMessage 硬编码中文
ElMessage.success('登录成功')

// ❌ 校验规则硬编码中文
{ required: true, message: '请输入账号' }

// ❌ 路由 title 硬编码中文
{ path: 'skills', meta: { title: 'Skill 广场' } }
```

**为什么不行**：
- 无法切换语言，国际化形同虚设
- 文案散落在几十个文件里，改一个提示语要全局 grep
- 翻译团队无从下手，只能逐文件捞

**正确写法**：
```vue
<!-- ✅ 模板走 $t() -->
<h1>{{ $t('skill.square.title') }}</h1>
<el-button>{{ $t('skill.square.createBtn') }}</el-button>
<EmptyState :text="$t('common.empty.default')" />
```
```ts
// ✅ ElMessage 走 t()
import { useI18n } from 'vue-i18n'
const { t } = useI18n()
ElMessage.success(t('auth.login.successMsg'))

// ✅ 校验规则走 t()
{ required: true, message: t('auth.login.validation.accountRequired') }

// ✅ 路由 title 存 i18n key
{ path: 'skills', meta: { title: 'skill.square.title' } }
```

---

## A22 · 只改一个 locale 文件

**反模式**：在 `zh-CN.json` 加了 key，没有在 `en-US.json` 同步添加。

**为什么不行**：切英文时回退显示中文或裸 key，用户体验糟糕；后续补翻译时不知道哪些 key 是缺的。

**正确写法**：每次改 locale 文件，**两个文件同时改**，key 结构完全对齐。英文翻译暂时没把握可以先写 TODO 标记值（如 `"[TODO] Skill Square"`），但 key 必须存在。

---

## A23 · 在 defineProps 默认值里调 t()

**反模式**：
```ts
withDefaults(defineProps<{ text?: string }>(), {
  text: t('common.empty.default')  // ❌ 此时 i18n 上下文可能未就绪
})
```

**为什么不行**：`defineProps` 默认值在组件实例化前求值，`useI18n()` 的 `t` 函数依赖组件上下文，可能还没建立，导致运行时报错或翻译不生效。

**正确写法**：props 不设含翻译的默认值，在模板侧用 `||` 兜底：
```vue
<template>
  <span>{{ text || $t('common.empty.default') }}</span>
</template>
```

---

## A25 · 所有 API 模块共用同一个 client（忽视微服务 baseURL 差异）

**反模式**：
```ts
// ❌ 所有模块都 import 同一个通用 http
import { http } from '@/api/request'
// skill.ts / mcp.ts / tool.ts / user.ts 统统指向同一 baseURL
export const skillApi = { list: (p) => http.get('/skills', { params: p }) }
export const mcpApi   = { list: (p) => http.get('/mcp/modules', { params: p }) }
```

**为什么不行**：
- 后端是微服务，不同服务可能部署在不同域名 / 端口
- 所有请求共用一个 baseURL 意味着无法独立指向各自的服务，环境切换时牵一发动全身
- 扩展新微服务时没有清晰的接入点，开发者会继续往同一个 client 里堆

**正确写法**：
```ts
// ✅ src/api/services/index.ts — 每个微服务一个 client
export const agentClient = createServiceClient({ baseURL: import.meta.env.VITE_AGENT_API_BASE, ... })
export const skillClient  = createServiceClient({ baseURL: import.meta.env.VITE_SKILL_API_BASE,  ... })
export const mcpClient    = createServiceClient({ baseURL: import.meta.env.VITE_MCP_API_BASE,    ... })

// ✅ 各模块按归属导入对应 client
// skill.ts
import { skillClient } from '@/api/services'
// mcp.ts
import { mcpClient } from '@/api/services'
```

接入新微服务时，按顺序执行以下四步：

1. **env 文件**：`.env.development` 加 `VITE_XXX_API_BASE`（dev 路径前缀，如 `/api-xxx`）和 `VITE_XXX_API_TARGET`（后端地址，如 `http://localhost:9000`）；`.env` / `.env.production` 加 `VITE_XXX_API_BASE`（值通常为 `/api`，由 Nginx 路由）。
2. **dev 代理**：`vite.config.ts` 照现有 `buildProxyEntry` 模式在 `proxy` 对象里追加一条——`[xxxProxyPath]: buildProxyEntry(xxxProxyPath, env.VITE_XXX_API_TARGET || fallback)`。dev server 会把 `/api-xxx/...` rewrite 成 `/api/...` 再转发给目标服务，**缺这步则 dev 环境请求 404**。
3. **service client**：`services/index.ts` 追加 `export const xxxClient = createServiceClient({ baseURL: import.meta.env.VITE_XXX_API_BASE, ... })`。
4. **API 模块**：新模块 import 该 client，其他文件无需改动。

---

## A26 · 用 `ElMessageBox` / `ElMessage` 自己弹接口错误弹窗

**反模式**：
```ts
// ❌ 在 store / composable / view 里自己捕错并弹窗
try {
  await skillApi.create(payload)
} catch (e) {
  ElMessageBox.alert(e.message, '请求失败', { type: 'error' })
  // 或
  ElMessage.error('服务器错误，请稍后重试')
}
```

**为什么不行**：
- 全局拦截器已经按策略（toast / dialog / silent）统一处理了错误，自己再弹等于双弹
- 各处自定义弹窗样式不一致，错误信息格式（httpStatus、traceId、detail）也不完整
- silent 模式下需要静默的请求，业务层却又弹出来，矛盾

**正确写法**：
- **不要** 在业务代码里 catch 接口错误后手动弹窗；让拦截器的默认策略（5xx→dialog，业务错误→toast）处理
- 确实需要自定义处理时，传 `errorMode: 'silent'` 静默请求，再在 catch 里做**业务逻辑**（如回滚状态），不要再弹窗：
```ts
// ✅ 默认：让拦截器自动弹
await skillApi.create(payload)

// ✅ 需要自己处理时：静默 + 只做业务逻辑，不手动弹窗
try {
  await skillApi.create(payload, { errorMode: 'silent' })
} catch (e) {
  // 只做业务：回滚本地状态、记录日志等
  formState.value = 'error'
}

// ✅ 需要主动推弹窗（极少数情况，如前端校验错误）时，走统一 store
import { useErrorDialogStore } from '@/stores/modules/errorDialog'
useErrorDialogStore().push({ id: nanoid(), code: 0, message: '操作失败：...' })
```

---

## A24 · i18n key 命名不规范

**反模式**：
```jsonc
{
  "广场标题": "Skill 广场",           // ❌ 中文 key
  "skill_square_title": "Skill 广场", // ❌ snake_case
  "a.b.c.d.e.f": "...",               // ❌ 超过 4 级
  "title": "Skill 广场"               // ❌ 无模块前缀，必然冲突
}
```

**为什么不行**：中文 key 在代码中不好输入和搜索；命名风格不统一后续维护成本极高；过深嵌套不可读；无前缀的扁平 key 必然和其他模块冲突。

**正确写法**：遵守 `{module}.{scope}.{semantic}` 三级结构，每段 camelCase：
```jsonc
{
  "skill": {
    "square": {
      "title": "Skill 广场"
    }
  }
}
```

---

## A27 · 用 `resolveComponent` 获取 Element Plus 图标组件

**反模式**：
```ts
// ❌ 以为 app.use(ElementPlus) 会全局注册图标，所以用 resolveComponent 获取
import { resolveComponent, type Component } from 'vue'
const checkIcon = resolveComponent('Check') as Component  // 返回字符串 'Check'，不是组件！

// 然后传给 :icon prop
// <el-button :icon="checkIcon"> → 图标静默不渲染，没有报错
```

**为什么不行**：
- `@element-plus/icons-vue` 是**独立 npm 包**，不属于 `element-plus` 主包
- `app.use(ElementPlus)` **不会**全局注册图标组件（只注册 `el-*` 组件）
- `resolveComponent('Check')` 找不到注册项，在 Vue 3 中**静默返回字符串** `'Check'`，不抛错
- `el-button` 内部用 `resolveDynamicComponent(icon)` 渲染图标；字符串 `'Check'` 同样找不到组件，图标**完全不渲染**，按钮看起来正常，但图标消失
- 这个 bug 在开发环境 console 有警告但很容易被忽略，在生产环境完全静默

**正确写法**：图标作为 JS 值（`:icon` prop）时，直接 import：
```ts
// ✅ 直接 import，作为 prop 值传入
import { Check, VideoPlay } from '@element-plus/icons-vue'

// <el-button type="primary" :loading="submitting" :icon="Check">发布</el-button>
// <el-button type="primary" :icon="VideoPlay">在 Playground 测试</el-button>
```

**注意**：图标有两种用法，规则不同：

| 用法 | 正确写法 | 说明 |
|------|---------|------|
| 模板中直接展示（`<el-icon>` 内） | `import { Check }` + `<Check />` 静态 tag | `unplugin-vue-components` 识别 PascalCase 静态 tag，自动处理 |
| 传给 `:icon` prop（JS 值） | `import { Check }` + `:icon="Check"` | 必须显式 import，不能用 `resolveComponent` |

**结论：不管哪种场景，图标都要 `import`。** 区别只是模板里写 `<Check />` 还是 `:icon="Check"`。

**`<component :is="'IconName'" />` string literal 的陷阱**：和 `resolveComponent` 一样——`unplugin-vue-components` 无法静态分析字符串形式，图标同样不渲染。`<component :is="...">` 只在 `someRef` 是通过 `import` 拿到的真实组件引用时才有效，不能是字符串。

---

## A28 · 在视图 `onMounted` 里用"有数据就跳过"做缓存

**反模式**：
```ts
// ❌ 以为这是"优化"，实际上是让页面永远显示旧数据
onMounted(() => {
  if (list.value.length === 0) {   // ← 只有列表为空才请求
    void store.fetchList()
  }
})
```

**为什么不行**：
- 用户从其他页面导航回来时，列表已有旧数据，条件不触发，页面永远不刷新
- 用户在 A 页面拉取了列表，切到 B 页面新建了一条记录，回到 A 看到的还是没有新记录的旧数据
- 与 `onActivated` 语义重叠但行为更差：`onActivated` 是 keep-alive 场景下的正确钩子；这种写法在普通路由（无 keep-alive）下也会阻止刷新

**混淆的两件事**：

| 目标 | 正确位置 | 正确手段 |
|------|---------|---------|
| 页面数据始终最新 | 视图层 `onMounted` | 无条件调 fetch，让数据从服务端刷新 |
| 防止同一时刻多个组件并发重复请求 | **store action 内部** | 用 `if (loading.value) return` 在 action 里去重 |

**正确写法**：
```ts
// ✅ 视图层：每次挂载都刷新，数据保鲜由接口保证
onMounted(() => {
  void store.fetchList()
})

// ✅ store action 内部：如需防并发，在 action 里加 loading 守卫
async function fetchList() {
  if (loading.value) return   // 已在请求中，跳过并发重复调用
  loading.value = true
  try {
    list.value = await api.getList()
  } finally {
    loading.value = false
  }
}
```

**补充**：如果页面是 keep-alive 缓存的（路由 meta 配了 `keepAlive: true`），则用 `onActivated` 替代 `onMounted` 触发刷新；两者不混用。

---

## A29 · 对接真实接口时用"后端没有这个字段"为由删除原型视觉元素

**背景**：页面先做了原型视觉还原，之后接入真实后端 API 时，发现部分原型字段（如表单里的请求方法、请求 URL、超时配置）在后端 `CreatePayload` 里没有对应字段，于是**直接把这些字段从表单里删掉**。

**反模式**：
```ts
// ❌ 接入 API 时，把"后端没有"的字段全部删除
// 原型里有：displayName / server / method / endpoint / timeout / returns
// 后端只有：toolName / description / domain / riskLevel / inputSchema

const form = reactive({
  toolName: '',       // 只保留后端有的字段
  description: '',
  domain: '',
  riskLevel: 'low',
  params: [],
  // displayName、server、method、endpoint、timeout、returns ← 全删了
})

async function submit() {
  await api.create({
    toolName: form.toolName,
    description: form.description,
    // ... 直接提交，原型字段就此消失
  })
}
```

**为什么不行**：
- 原型是产品设计的视觉契约，不是技术实现的草稿。"后端暂不支持"不等于"产品不需要"
- 删掉字段 = 视觉回退，用户失去了填写这些字段的入口，后续后端补全接口后还要重新做一遍表单
- 这个反模式往往发生在"原型视觉还原"和"真实接口接入"被合并成一个步骤时，缺少了视觉设计与数据层的分离意识

**正确做法**：表单展示所有原型字段；提交时用**白名单函数**只提取后端已支持的字段；原型专有字段加注释标明等待后端。

```ts
// ✅ 表单保留全部原型字段
const form = reactive<ToolCreateForm>({
  displayName: '',   // 展示名（原型有，后端暂无独立字段，待后端补充）
  toolName: '',      // Tool Key → 对应后端 toolName
  server: '',        // 所属 Server（原型有，对应后端 mcpServerId，待映射）
  method: 'POST',    // 请求方法（原型有，后端暂无，提交时不发送）
  endpoint: '',      // 请求 URL（原型有，后端暂无，提交时不发送）
  timeout: 10000,    // 超时（原型有，后端 policy 只读，提交时不发送）
  description: '',
  domain: '',
  riskLevel: 'low',
  params: [],
  returns: [],       // 返回字段（对应后端 outputSchema）
})

// ✅ 白名单函数：只产出后端已定义的入参，其余字段静默忽略
function toCreatePayload(form: ToolCreateForm): ApiPayload {
  return {
    toolName: form.toolName,
    description: form.description,
    domain: form.domain,
    riskLevel: form.riskLevel,
    inputSchema: buildInputSchema(form.params),
    outputSchema: buildOutputSchema(form.returns),
    // method / endpoint / timeout ← 后端待补充，暂不发送
  }
}
```

**核心原则**：**视觉设计与数据提交解耦**。表单字段由原型决定，提交字段由后端接口决定，两者通过白名单映射函数连接，互不干扰。后端补充字段时，只需修改映射函数，不需要重新设计表单。
