# COMPONENTS.md 模板

`COMPONENTS.md` 是阶段 2 的产物。它把页面里所有可复用元素列出来、分类、明确构建顺序。

## 模板结构

```markdown
# 组件树 — <页面名>

## Tokens
已经在 `PLAN.md` 里捕获完毕。Token 文件是构建顺序里的第一个产物。

## 原子组件 (Primitives)
最小单元，没有子组件，只有 props。

| 组件 | 来源节点 | 复用状态 | Props / 变体 | 备注 |
| --- | --- | --- | --- | --- |
| `Button` | 12:813（基准） | wrap `<el-button>` | variant: primary \| secondary \| text；size: sm \| md；disabled、loading | 状态变体在 frame 13:200 找到 |
| `Tag` | 12:512 | 直接用 `<el-tag>` | tone: info \| warn \| success | 颜色映射在备注 |
| `Avatar` | 12:580 | 自建 | size: sm (32) \| md (48)；fallback 显示首字 | |
| `Icon` | 多处 | 直接用 `lucide-vue-next` | name、size | |

**复用状态**字段三选一：
- **直接用 `<xxx>`** — 现有组件库的 API 完全够用，PLAN/Component 都不新建文件
- **wrap `<xxx>`** — 在现有组件外包一层改样式或调 props 默认值，新建一个组件文件
- **自建** — 现有库没有，从零写

## 复合组件 (Composites)
由原子组件组成，封装一个重复的布局单位。

| 组件 | 来源节点 | 由谁组成 | Props | 备注 |
| --- | --- | --- | --- | --- |
| `DoctorCard` | 12:586 | Avatar + 标题 + Tag + Button | doctor: Doctor | 在模块 3 用了 4 次 |
| `ArticleListItem` | 12:640 | 缩略图 + 标题 + 元信息 | article: Article | 在模块 4 用了 6 次 |
| `ServiceTile` | 12:518 | Icon + 文案 | name、icon | 模块 2 的 2×4 宫格 |

## 模块 (Modules)
页面级分区，一般不跨页复用。

| 模块 | 来源节点 | 由谁组成 | 备注 |
| --- | --- | --- | --- |
| `TopBar` | 12:446 | Icon + 自定义搜索框 | sticky；适配安全区上 |
| `ServiceGrid` | 12:512 | ServiceTile × 8 | |
| `DoctorList` | 12:580 | DoctorCard × n | 横向滚动，snap 对齐 |
| `ArticleSection` | 12:631 | 区块标题 + ArticleListItem × n | |
| `TabBar` | 12:700 | Icon + 文案 × 5 | sticky 底部；适配安全区下 |

## 构建顺序

1. **Tokens** — 写 token 文件
2. **原子** — `Icon`、`Avatar`、`Tag`、`Button`（依赖少的先写）
3. **复合** — `ServiceTile`、`DoctorCard`、`ArticleListItem`
4. **模块** — `TopBar`、`ServiceGrid`、`DoctorList`、`ArticleSection`、`TabBar`
5. **整页** — 按 `PLAN.md` 的布局组装所有模块
```

## 命名约定

- **统一用 PascalCase**，不管最终框架的命名习惯是什么——生成具体文件时再按框架转。这一份索引里 PascalCase 最好扫读。
- **挑一个"基准节点"**——选最干净的那个实例。其它出现的位置写在备注里，方便你后面验证它们是否真的等价。
- **不要过度组件化**。在某个区块只出现一次的 UI 是模块自身的内部结构，不是单独的组件。等到第二次出现、或者用户明确要求复用，再提升为组件。

## 怎么判断"同一个组件的不同变体"

两段 UI 是同一个组件，前提是它们都共享：
- 布局结构（同样的子元素以同样的方式排布）
- Token（同样的字体、颜色、圆角尺度）

…只在以下方面有差别：
- 内容（文字、图片）
- 一两个看起来像真实变体的视觉属性（实心 vs 描边、主色 vs 次色）

如果布局结构或 token 用法有差别，那就是不同的组件，即使长得有点像也别强行合并。

## 阶段 3 中途要补充组件时怎么办

阶段 3 写代码时，可能会发现一个阶段 2 没识别出来的可复用模式（比如模块 2 里的"标签"其实和模块 5 里的"标记"是同一个）。这时：

1. **停下当前模块**。
2. 把新发现的组件加到 `COMPONENTS.md` 里，写清来源和备注。
3. **先把它做出来**（按构建顺序里的合理位置——通常作为原子，在用到它的模块之前）。
4. 再回来继续原来的模块。

为了重构花的 5 分钟，比后期发现重复实现要便宜得多。
