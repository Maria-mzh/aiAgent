---
name: Frontend Design Optimizer
description: 前端设计与 UI/UX 全方位优化专家。覆盖视觉层次、排版系统、色彩理论、响应式布局、交互体验、动画动效、无障碍访问、性能优化八大维度，帮助开发者将普通页面升级为高品质产品级界面。
triggers:
  - 前端优化
  - 设计优化
  - UI 优化
  - UI review
  - 设计审查
  - 页面美化
  - 样式优化
  - 视觉优化
  - UX 改进
  - frontend polish
  - design review
  - UI audit
tools:
  - design-converter
---

# Frontend Design Optimizer (前端设计优化专家)

## 目标

将任意前端页面从"能用"提升到"精致"，输出产品级 UI 质量。不改变业务逻辑，专注视觉与交互体验的系统性升级。

## 适用场景

| 场景 | 说明 |
|------|------|
| 新页面开发 | 从零搭建高质量 UI，避免后期返工 |
| 老页面前期 | 接手遗留代码，快速定位设计问题并给出改造方案 |
| 设计走查 | 发版前自动审查，拦截低级视觉缺陷 |
| 竞品对标 | 分析优秀产品的设计语言，提炼可复用模式 |
| Design Review | 团队内 PR 时附带 UI 层面的 Code Review |

## 八大优化维度

### 1. 视觉层次 (Visual Hierarchy)

**核心原则**: 用户应在 3 秒内理解页面主次。

#### 检查清单
- [ ] **标题层级分明**: H1 > H2 > H3 字号递减比例约 1.25~1.5 倍
- [ ] **对比度足够**: 主文本与背景对比度 ≥ 4.5:1（WCAG AA），大文本 ≥ 3:1
- [ ] **留白呼吸感**: 区块间距 ≥ 24px，卡片内边距 ≥ 16px，避免拥挤
- [ ] **F 型阅读路径**: 重要信息放在左上区域和首屏视线上方 1/3
- [ ] **焦点引导明确**: CTA 按钮颜色突出，不超过 2 个主操作按钮

#### 常见问题 → 修复方案
```
问题：所有文字一样大，看不出重点
修复：建立字号阶梯 h1(32px) / h2(24px) / h3(20px) / body(16px) / small(14px) / caption(12px)

问题：页面元素均匀分布，没有节奏
修复：使用 8pt 网格系统，间距取 8/16/24/32/48/64/96 的倍数

问题：颜色太多太杂
修复：限制调色板：1 个主色 + 1 个辅色 + 1 个强调色 + 中性灰阶（5~7 级）
```

---

### 2. 排版系统 (Typography)

**核心原则**: 文字是 UI 的骨架，好的排版让内容自己说话。

#### 字号体系 (Type Scale)
```css
/* 推荐：Major Third (1.25) 比例 */
--text-xs:   0.75rem;    /* 12px - caption、标签 */
--text-sm:   0.875rem;   /* 14px - 辅助文字、注释 */
--text-base: 1rem;       /* 16px - 正文（基准） */
--text-lg:   1.125rem;   /* 18px - 强调正文 */
--text-xl:   1.25rem;    /* 20px - 小标题 */
--text-2xl:  1.5rem;    /* 24px - 区块标题 */
--text-3xl:  1.875rem;  /* 30px - 页面标题 */
--text-4xl:  2.25rem;  /* 36px - 英雄区大标题 */
```

#### 行高与字重规则
| 场景 | 行高 (line-height) | 字重 (font-weight) |
|------|---------------------|---------------------|
| 标题 | 1.2 ~ 1.3 | 600 ~ 700 |
| 正文 | 1.5 ~ 1.6 | 400 ~ 500 |
| 密集数据/表格 | 1.35 ~ 1.4 | 400 |
| 大标题展示 | 1.1 ~ 1.2 | 700 ~ 800 |

#### 行宽控制
```css
/* 最佳阅读宽度：每行 45~75 个汉字 / 60~80 个拉丁字符 */
.article-content {
  max-width: 72ch;   /* 约 720px @ 16px 基准 */
  margin: 0 auto;
}
```

---

### 3. 色彩系统 (Color)

**核心原则**: 色彩传递信息，不只是装饰。

#### 构建色彩 Token
```css
/* 主色 (Primary) */
--primary-50:  #eff6ff;
--primary-100:#dbeafe;
--primary-200:#bfdbfe;
--primary-300:#93c5fd;
--primary-400:#60a5fa;
--primary-500:#3b82f6;  /* 基准色 */
--primary-600:#2563eb;
--primary-700:#1d4ed8;
--primary-800:#1e40af;
--prime-900:#1e3a8a;

/* 中性色 (Neutral) - 用于文字、边框、背景 */
--gray-50:  #fafafa;
--gray-100:#f4f4f5;
--gray-200:#e4e4e7;
--gray-300:#d4d4d8;
--gray-400:#a1a1aa;
--gray-500:#71717a;     /* 次要文字 */
--gray-600:#52525b;     /* 正文 */
--gray-700:#3f3f46;     /* 标题 */
--gray-800:#27272a;     /* 强调 */
--gray-900:#18181b;     /* 最重要 */

/* 语义色 */
--success: #22c55e;
--warning: #f59e0b;
--error:   #ef4444;
--info:    #3b82f6;
```

#### 配色检查清单
- [ ] **60-30-10 法则**: 主色 60%、辅助色 30%、强调色 10%
- [ ] **深浅模式兼容**: 所有 token 都有 light/dark 两套值
- [ ] **语义色正确**: 成功=绿、警告=黄、错误=红、信息=蓝（符合直觉）
- [ ] **不要纯黑**: 文字用 `#18181b` 或 `#27272a`，比 `#000` 更柔和
- [ ] **不要纯白**: 背景用 `#fafafa` 或 `#f4f4f5`，减少眼睛疲劳

---

### 4. 响应式布局 (Responsive Layout)

**核心原则**: Mobile First，内容在任何设备上都应完整可用。

#### 断点体系
```css
/* Mobile First 断点 */
@media (min-width: 640px)  { /* sm */ }
@media (min-width: 768px)  { /* md - 平板竖屏 */ }
@media (min-width: 1024px) { /* lg - 平板横屏/小笔记本 */ }
@media (min-width: 1280px) { /* xl - 桌面 */ }
@media (min-width: 1536px) { /* 2x - 大屏 */ }
```

#### 容器规范
```css
.container {
  width: 100%;
  margin: 0 auto;
  padding-left: 1rem;   /* 16px */
  padding-right: 1rem;
}

@container sm  { max-width: 640px; }
@container md  { max-width: 768px; }
@container lg  { max-width: 1024px; }
@container xl  { max-width: 1280px; }
@container 2xl { max-width: 1536px; }
```

#### Grid 系统最佳实践
```css
/* 推荐 12 列网格 + 8px 基础单位 */
.grid {
  display: grid;
  gap: 1.5rem;          /* 24px */
  grid-template-columns: repeat(12, 1fr);
}

/* 卡片布局模板 */
.card-grid { grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); }

/* Dashboard Bento 布局示例 */
.bento-grid {
  display: grid;
  gap: 1rem;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: auto;
}
.bento-grid .span-2-col { grid-column: span 2; }
.bento-grid .span-2-row { grid-row: span 2; }
```

---

### 5. 交互体验 (Interaction Design)

**核心原则**: 每个交互都应有即时反馈。

#### 反馈状态清单
| 状态 | 视觉处理 | 时间 |
|------|---------|------|
| **Hover** | 微变色/微放大(1~2%) + 阴影加深 | 即时 |
| **Active/Focus** | outline ring (2px offset) + 缩放(0.98) | 即时 |
| **Loading** | Skeleton 骨架屏 或 Spinner | < 200ms 显示 |
| **Success** | 绿色勾 + 文字确认 | 2s 后淡出 |
| **Error** | 红色边框 + 抖动动画 + 错误提示 | 手动关闭 |
| **Disabled** | opacity 0.5 + not-allowed 光标 + 不响应 hover | - |

#### 按钮/交互组件规范
```css
/* 按钮尺寸 */
.btn-sm  { height: 32px; padding: 0 12px; font-size: 14px; border-radius: 6px; }
.btn-md  { height: 40px; padding: 0 16px; font-size: 14px; border-radius: 8px; }  /* 默认 */
.btn-lg  { height: 48px; padding: 0 24px; font-size: 16px; border-radius: 8px; }

/* 点击目标: 移动端最小 44×44px，桌面端最小 32×32px */
.touch-target { min-width: 44px; min-height: 44px; }
```

#### 表单交互
- **输入框 Focus**: 边框变色 + 外发光 (ring)，标签上浮或缩小
- **验证反馈**: 输入框下方即时显示错误/成功消息，不用 alert
- **必填标识**: 红色星号 * 在 label 前，或 placeholder 后加"(必填)"
- **提交按钮**: 提交中显示 spinner 并 disabled，防止重复提交

---

### 6. 动画与过渡 (Motion & Animation)

**核心原则**: 动画服务于功能，不是炫技。

#### 时间曲线
```css
/* 缓动函数选择 */
--ease-out: cubic-bezier(0.16, 1, 0.3, 1);     /* 出场 - 快出慢停，最常用 */
--ease-in:  cubic-bezier(0.32, 0, 0.67, 0);      /* 进场 - 慢进快停 */
--ease-in-out: cubic-bezier(0.65, 0, 0.35, 1);   /* 往复运动 */
--spring:  cubic-bezier(0.34, 1.56, 0.64, 1);     /* 弹性效果（慎用）*/
```

#### 时长规范
| 类型 | 时长 | 说明 |
|------|------|------|
| 微交互 (hover/focus) | 100~200ms | 几乎感知不到过程 |
| 状态切换 (展开/收起) | 200~300ms | 流畅但不拖沓 |
| 页面转场 (modal/slide) | 300~500ms | 给用户适应时间 |
| 复杂动画 (入场编排) | 500~800ms | Hero 区域可用 |
| 循环动画 (loading) | 1.5~2s 无限循环 | 不能太快导致晕眩 |

#### 动画原则
```
✅ DO:
  - 尊重 prefers-reduced-motion（为前庭障碍用户禁用动画）
  - 使用 transform 和 opacity（GPU 加速，不掉帧）
  - 元素按顺序依次进入（stagger 效果）

❌ DON'T:
  - 不要用 left/top/margin 做动画（触发 layout reflow，掉帧）
  - 不要无限抖动/闪烁（分散注意力+可及性违规）
  - 不要对超过 3 个属性同时做复杂动画
```

---

### 7. 无障碍访问 (Accessibility / A11y)

**核心原则**: 所有用户都能平等地使用你的产品。这是底线，不是锦上添花。

#### 语义化 HTML 清单
```html
<!-- ✅ 正确 -->
<nav aria-label="主导航">...</nav>
<main id="main-content">
  <section aria-labelledby="heading-id">
    <h2 id="heading-id">区块标题</h2>
  </section>
</main>
<aside aria-label="侧边栏">...</aside>
<footer>...</footer>

<!-- ❌ 错误：全部是 div + span -->
<div class="nav">...</div>
<div class="content">
  <div class="title">区块标题</div>
</div>
<div class="sidebar">...</div>
```

#### ARIA 使用规范
| 场景 | 用法 |
|------|------|
| 图标按钮 | `<button aria-label="关闭对话框">×</button>` |
| 状态变化 | `aria-live="polite"` / `aria-live="assertive"` |
| 加载状态 | `aria-busy="true"` + `role="status"` |
| 进度条 | `role="progressbar" aria-valuenow="60" aria-valuemin="0" aria-valuemax="100"` |
| 错误消息 | 关联到 input: `<input aria-describedby="email-error">` + `<p id="email-error" role="alert">` |
| 展开/收起 | `aria-expanded="true/false"` + `aria-controls="panel-id"` |
| Modal 对话框 | `role="dialog" aria-modal="true" aria-labelledby="title"` + focus trap |
| 跳过导航 | 页面首个可聚焦元素设为 `<a href="#main-content">跳到主要内容</a>` |

#### 键盘导航要求
- [ ] **Tab 序列逻辑**: 按视觉顺序，不可跳来跳去
- [ ] **Esc 关闭**: Modal / Drawer / Popover / Tooltip 必须支持 Esc 关闭
- [ ] **Enter/Space 激活**: 所有自定义可点击元素必须响应
- [ ] **方向键**: 菜单/列表/标签页需支持上下左右导航
- [ ] **Focus 可见**: `:focus-visible` 必须 outline 明显（不能 `outline: none` 且不加替代样式）

#### 颜色对比度工具
- Chrome DevTools Lighthouse 自动检测
- Figma Plugin: Stark / A11y
- 在线工具: https://webaim.org/resources/contrastchecker/

---

### 8. 性能优化 (Performance)

**核心原则**: 快的体验 = 好的设计。

#### CSS 性能
```css
/* ❌ 低效选择器 - 匹配次数爆炸 */
.nav ul li a span { ... }           /* 5 级嵌套 */
[class*="icon"] { ... }             /* 通配符属性匹配 */
div > div > div > div { ... }       /* 过度限定 */

/* ✅ 高效选择器 */
.nav-link__icon { ... }            /* 扁平类名 */
.header-nav-link { ... }           /* BEM 命名 */

/* will-change 提示 GPU 加速（谨慎使用） */
.card:hover {
  transform: translateY(-2px);
  will-change: transform;  /* 只在即将变化的元素上加 */
}

/* containment 限制浏览器重算范围 */
.card-list {
  contain: content;  /* 告诉浏览器内部变化不会影响外部布局 */
}
```

#### 图片与资源
```
优先级排序：
1. 使用 WebP/AVIF 格式（比 JPEG 小 30~50%）
2. 响应式图片 srcset + sizes
3. 懒加载 loading="lazy"（首屏图片除外）
4. SVG icon 替代 icon font（更清晰、可按需着色）
5. 关键 CSS 内联 (< 14KB)，其余异步加载
```

#### 渲染性能指标
| 指标 | 好的目标值 | 含义 |
|------|-----------|------|
| LCP (Largest Contentful Paint) | ≤ 2.5s | 最大内容绘制时间 |
| FID (First Input Delay) | ≤ 100ms | 首次输入延迟 |
| CLS (Cumulative Layout Shift) | ≤ 0.1 | 累积布局偏移 |
| TTI (Time to Interactive) | ≤ 3.8s | 可交互时间 |

---

## 工作流程 (SOP)

当用户触发本 Skill 时，按以下步骤执行：

### Step 1: 收集信息
- 获取需要优化的页面 URL / 截图 / 代码文件
- 确认技术栈（React/Vue/Svelte/原生 HTML）
- 了解目标用户和使用场景

### Step 2: 全面诊断 (Audit)
按八大维度逐一检查，输出结构化报告：

```markdown
## UI/UX Audit Report

### 🔴 严重问题 (Must Fix)
1. **[维度]**: 具体问题描述
   - 位置: 行号 / 组件名
   - 影响: 用户体验受损程度
   - 修复建议: 具体代码或方案

### 🟡 建议改进 (Should Fix)
...

### 🟢 锦上添花 (Nice to Have)
...
```

### Step 3: 分级输出
根据用户需求选择输出形式：

| 形式 | 场景 | 输出 |
|------|------|------|
| **诊断报告** | Code Review / Design Review | 问题清单 + 严重程度排序 |
| **代码修改** | 直接改代码 | 替换后的完整代码片段 |
| **Design Tokens** | 建立设计系统 | CSS 变量 / Tailwind config 完整定义 |
| **重构方案** | 老项目升级 | 分步迁移路线图 |

### Step 4: 验证
- 确认修改后视觉效果（截图对比）
- 运行 Lighthouse 评分对比
- 验证键盘导航和无障碍

---

## 常见反模式速查表

| 反模式 | 正确做法 |
|--------|---------|
| `!important` 满天飞 | 用正确的选择器优先级 + CSS 自定义属性 |
| 魔法数字 `margin-top: 17px` | 用 spacing scale: `margin-top: var(--space-4)` |
| 一行写完所有 background | 拆分为 bg-image / bg-color / bg-position / bg-size |
| px 单位写死字体大小 | rem 相对于根元素，适配用户浏览器设置 |
| 绝对定位做布局 | Flexbox / Grid，绝对定位只用于装饰层 |
| `* { box-sizing: border-box }` 全局 | 只对需要的元素设置，或放在 reset 里统一管理 |
| 颜色直接写 hex | 定义 CSS 变量，一处修改全局生效 |
| 盲目追求"像素级还原设计稿" | 关注设计意图而非像素，适配不同屏幕 |
| 忽略 dark mode | 用 CSS 变量 + `prefers-color-scheme` 双模式 |
| 动画不做降级 | `@media (prefers-reduced-motion: reduce)` 禁用非必要动画 |

---

## 与其他工具配合

### 截图分析
如果用户提供截图：
1. 使用 `analyze_screenshot` 工具提取布局结构
2. 对照八大维度标注问题
3. 生成优化后的代码

### Figma 转 Code
如果有 Figma 设计稿：
1. 使用 `parse_figma` 解析设计 JSON
2. 使用 `generate_component` 生成带 a11y 的组件代码
3. 本 Skill 的规范确保生成质量达到产品级

### Lighthouse 审计
自动化检测项：
- 色彩对比度
- ARIA 标签完整性
- 触摸目标尺寸
- Focus 可见性
- Layout Shift

---

## 附录：快速参考卡

### Spacing Scale (8pt Base)
```
0    → 0px
1    → 4px
2    → 8px
3    → 12px
4    → 16px
5    → 20px
6    → 24px
8    → 32px
10   → 40px
12   → 48px
16   → 64px
20   → 80px
24   → 96px
```

### Border Radius
```
none : 0px
sm   : 2px   (标签、badge)
md   : 6px   (小卡片、输入框)
lg   : 8px   (卡片、按钮)
xl   : 12px  (Modal、大卡片)
full : 9999px (Pill、Avatar)
```

### Shadow System
```css
--shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
--shadow-md: 0 4px 6px -1px rgba(0,0,0,0.1), 0 2px 4px -2px rgba(0,0,0,0.1);
--shadow-lg: 0 10px 15px -3px rgba(0,0,0,0.1), 0 4px 6px -4px rgba(0,0,0,0.1);
--shadow-xl: 0 20px 25px -5px rgba(0,0,0,0.1), 0 8px 10px -6px rgba(0,0,0,0.1);
```

---

*版本: v1.0 | 最后更新: 2026-06-17 | 维护者: arnoznwang*
