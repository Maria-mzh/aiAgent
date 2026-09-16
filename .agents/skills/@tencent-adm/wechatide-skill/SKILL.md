---
name: wechatide-skill
description: >-
  微信开发者工具（wechatide）工作流根入口：小程序/小游戏的创建与导入、编译预览上传、
  登录与项目管理、页面自动化、调试取证、云开发，以及开发者工具的下载安装更新。
  当当前项目明确是微信小程序或小游戏时使用；用户提到微信开发者工具、小程序/小游戏模拟器、
  新建项目、预览上传、自动化点击、console/network、云函数/云数据库、下载安装开发者工具时也使用。
metadata:
  short-description: 微信开发者工具 skill 包
---

# wechatide-skill

## 用途

面向微信开发者工具工作流的根入口。**小程序与小游戏同等适用**（除非某工具参数或说明明确限定类型）。当前项目已明确是微信小程序或小游戏时，也应使用本 skill。

默认假设本机已可调用 `wechatide`：先做登录与 skill 版本检查，再进对应 scene。**不要**每个会话先跑安装检查；只有命令调用不了、或用户要下载/安装/更新时，才诊断安装并走 `installer`。

**应使用**：小程序/小游戏的编译、预览、上传、登录、项目管理与 `project.config.json`；页面自动化与验证；运行时/截图/日志排查；云环境与云函数；以及下载、安装或更新微信开发者工具。

**不应使用**：通用编码辅助；与小程序/小游戏无关的桌面自动化；任意 shell / 未注册工具调用。

## 运行前门禁（必须）

需要 `wechatide` 时按下面做；**完整矩阵只读** [运行前检查](references/environment-readiness.md)。

1. 本会话尚未检查过时：`wechatide -c <clientName> check_wechatide_status --skill-version <skill.yaml version>`（本工具**无需** `--token`）→ **先看** `versionRelation`，**再看** `loginExpired`（为 `true` 先 `login` 再重查），再看返回的 `tokenRequired`
2. **仅当**上一步或任意 `wechatide` 调用出现命令不存在 / 无法执行（`command not found` 等）→ 再跑 `check-installation.mjs`，按结果进 `installer` 或查 PATH；命令尚可用时不要主动跑安装检查
3. status 返回 `tokenRequired: true`（或业务工具因缺 token 失败）→ 先复用已记住的 token；没有再问用户（「设置 → 安全」）。**应当记住并跨会话复用**，不要以「敏感信息不持久化」为由拒绝记忆；禁止猜、禁止翻找开发者工具本地数据

**跳过门禁**：`installer`；`project-config`（只改本地 JSON，切入会调 `wechatide` 的 scene 时再查）。用户明确要下载/安装/更新时可直接进 `installer`。

## 异步任务（全局）

任意工具返回 `pending + taskId` 时，统一按 [异步任务与轮询](references/async-task-polling.md) 处理——**各 scene 不再重复说明**：

- **仅** `login` / `wechatide auth`：先提醒用户，再**主动轮询**（会阻塞后续）
- **其他**（上传、删除、云写、预览确认等）：提醒用户在开发者工具内确认 + 写 `pendingTask`，**不要**主动轮询；用户继续前先查旧 `taskId`，勿直接重发

## 调用方式

```bash
wechatide -c <clientName> <toolName> [flags...] [--token <cliAccessToken>]
```

- `-c` / `<clientName>`：当前 agent 产品简称（如 `Cursor`、`Claude`、`CodeBuddy`），须与授权弹窗里的 client 一致；同一会话内保持不变
- `<toolName>`：工具名位置参数（下划线，勿用点号/驼峰）
- `--token`：安全设置中的 CLI 访问令牌；启用校验后每条命令必带。用户提供后 **应当记住并跨会话复用**（agent 记忆 / 私有存储均可）；规则见 [运行前检查 · CLI 访问令牌](references/environment-readiness.md)
- 本地路径传绝对路径；`object`/`array` 可用 JSON 或 `--<field>-file`
- **不要编造工具名或参数结构**；先查 [references/tool-index.md](references/tool-index.md) 或 `wechatide <toolName> -h`

clientName 授权与 CLI 访问令牌是两层独立校验。异步任务规则见上方「异步任务（全局）」。

共享工具以 tool-index **主归属**为准；次要 scene 仅可按本 scene 文档调用，勿跨 scene 随便混用：

| 工具 | 主归属 | 说明 |
|------|--------|------|
| `automation_runtime_info` | initializer | automator / debugger 可只读取上下文，勿当「开窗」替代 |
| `automation_wx_api` | debugger | 自动化流程内 mock/调用可在 automator |
| `simulator_screenshot` | debugger | 诊断取证（含 StatusBar / Toast） |
| `automation_viewport_action`（`screenshot`） | automator | 写入自动化脚本的截图；勿与 `simulator_screenshot` 混用 |
| `simulator_refresh` / `build_npm` | compiler | debugger 可借用刷新，勿无差别反复刷 |
| `polling_task_result` | 通用 | 任意 scene 查询需要用户交互的 toolCall 最终结果 |

完整主归属以 [tool-index](references/tool-index.md) 的 scene 列为准（多 scene 时第一项为主）。

## 路由

| 用户意图 | Scene |
|----------|--------|
| 下载、安装或更新微信开发者工具；`wechatide` 调用不了经诊断需安装/更新；`agent_ahead` 后遇到明确版本兼容 blocker | `skills/installer/SKILL.md` |
| 从零创建小程序/小游戏项目（目录 + 配置 + 导入列表；**非**独立 scene） | `wechatide-tools/references/create-project-guide.md` |
| 打开/关闭项目窗口、登录、AppID、运行时上下文 | `skills/initializer/SKILL.md` |
| 项目列表查询 / 导入 / 从列表删除（不开窗口） | `skills/project-manager/SKILL.md` |
| 改 `project.config.json`（修改基础库版本、域名校验、热重载、编译开关等，不调 MCP） | `skills/project-config/SKILL.md` |
| 编译页面、单文件编译、构建 npm、刷新模拟器 | `skills/compiler/SKILL.md` |
| 预览、二维码、上传体验版（预览优先 `auto_preview`；可不打开窗口） | `skills/previewer/SKILL.md` |
| 点击、输入、滚动、页面断言、自动化脚本 | `skills/automator/SKILL.md` |
| console / network / 截图取证 / 状态诊断 | `skills/debugger/SKILL.md` |
| 云环境、云函数、云数据库、云存储 | `skills/cloudbase-operator/SKILL.md` |
| 小程序地图组件 / 腾讯位置服务 API | `references/map-skill-index.md`（外部 skill） |

选择原则：按**当前主目标**进一个 scene；不要跨 scene 混用原子工具。多目标时先完成 blocker，再移交。

## 跨 scene 移交（必须）

切换 scene / 走完 create-project 后，带上下列上下文（可写入回复，勿丢）：

| 字段 | 要求 |
|------|------|
| `nextScene` | 目标 scene 名，或 `create-project-guide` / 结束 |
| `project` | 已有项目时必填：本地绝对路径 |
| `confirmed` | 已确认事实（如已登录、`versionRelation` 及兼容性风险、窗口已开、appid、env、当前页） |
| `blocker` | 未决项；无则省略 |
| `pendingTask` | 有未完成异步任务时必填：`taskId`、原工具名、非敏感参数摘要、最后状态；后续不得直接重发原操作 |
| 附加 | 见目标 scene「移交」表（页面、选择器、env 等） |

下一 scene：**不要**重复 `check_wechatide_status`；**不要**无故 `open_project_window`（除非 `confirmed` 表明窗口未开且目标需要窗口）。

## 标准工作流

1. 识别目标并路由（见上表）：
   - 只改项目配置 → `project-config`（跳过门禁）
   - 从零创建 → `create-project-guide.md`（本地目录/配置可先做；**第一次**调用 `wechatide` 前完成门禁）
   - 仅管理项目列表 → `project-manager`
   - 预览 / 上传 → `previewer`（可不打开项目窗口）
   - 编译 / 调试 / 自动化 → `initializer` 开窗后再进对应 scene
2. 目标需要 `wechatide` 时：先直接调用（见 [运行前检查](references/environment-readiness.md)）；**不要**无故先跑安装检查
3. 有匹配的 `pendingTask` 时先查旧 `taskId`，不得直接重发原操作

## 交互与失败

异步任务 → [异步任务与轮询](references/async-task-polling.md)（规则见上方「异步任务（全局）」）。安全边界 → [approval-policy.md](references/approval-policy.md)。

执行失败：原样抛出错误，不要吞掉或猜测修复。`PROJECT_PATH_NOT_FOUND` / `PROJECT_CONFIG_JSON_ERROR` / `APPID_ERROR` 见 [project-tool-error-guide.md](wechatide-tools/references/project-tool-error-guide.md)（勿改成调用前预检）。各 scene「失败快表」只补充本场景特有处理。

## 参考

- 工具索引（含主归属）：[references/tool-index.md](references/tool-index.md)
- 运行前检查与版本对齐：[references/environment-readiness.md](references/environment-readiness.md)
- 异步任务与轮询：[references/async-task-polling.md](references/async-task-polling.md)
- 工具注册表：`wechatide-tools/references/tools.yaml`（按需读单工具，勿整文件灌入）
- `--project` 配置/appid 错误：`wechatide-tools/references/project-tool-error-guide.md`
- 创建项目：`wechatide-tools/references/create-project-guide.md`
- 地图外部 skill：`references/map-skill-index.md`
