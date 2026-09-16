# 运行前检查

需要调用 `wechatide` 时按本文处理。矩阵与例外以本文为准；根 SKILL 只保留短清单。

**安装检查不是默认前置。** 每个会话先做一次登录与 skill 版本检查即可；**只有** `wechatide` 命令调用不了时，再跑 `check-installation.mjs`。

**跳过门禁**：`installer`（下载/更新）；`project-config`（只改本地 JSON）。后续若切换到会调用 `wechatide` 的 scene，再执行本文。

## 清单

1. 本会话尚未检查过时，用 **wechatide** 执行（不是裸调工具名、也不是其它命令）：

```bash
wechatide -c <clientName> check_wechatide_status --skill-version <skill.yaml 的 version>
```

`check_wechatide_status` **不需要** `--token`（启用 CLI 访问令牌时也跳过）。返回体含 `tokenRequired`：为 `true` 时，后续业务工具须带 `--token`。

按下方「登录与 skill 版本」处理：**先看** `versionRelation`（`skip_check` 补传版本重查；`agent_behind` 单向导入后重查；`equal` / `agent_ahead` 可继续），**再看** `loginExpired`：为 `true` 时 `wechatide -c <clientName> login`（主动轮询至成功）后再重查；为 `false` 后进入业务。再看 `tokenRequired`：为 `true` 且本地无已存 token 时，先向用户索取后再调业务工具。

2. 若上一步（或任意 `wechatide` 调用）出现命令不存在 / 无法执行（`command not found`、`not recognized` 等）→ 再执行下方「安装兼容性」诊断，按结果进 `installer` 或修 PATH；**不要**在命令尚可用时主动跑安装检查
3. 若返回 `tokenRequired: true` / 缺 token 失败 → 见下方「CLI 访问令牌」，停在门禁
4. 若 `login` / `auth` 返回 `pending + taskId` → 按根 SKILL「异步任务（全局）」**主动轮询**（会阻塞门禁后续步骤）

每个会话对「登录与 skill 版本」成功检查一次即可；下一 scene 不要重复 `check_wechatide_status`。

## 1. 安装兼容性（仅命令不可用时）

触发条件：`wechatide` / `wechatide -h` / 任意 `wechatide …` 出现命令不存在或无法执行。此时在本 skill 根目录执行：

```bash
node skills/installer/scripts/check-installation.mjs
```

| 结果 | 处理 |
|------|------|
| `compatible: true` | CLI 仍不可用则查 PATH / shell 环境；不要立刻循环下载 |
| `reason: not_installed` | 用户未说明自定义安装时进入 installer；否则询问安装目录并用 `--install-root` 重查 |
| `reason: nw_runtime_incompatible` | 强制进入 installer 更新 |
| `reason: electron_version_too_old` | 强制进入 installer 更新 |
| `reason: cli_unavailable` | 强制进入 installer；安装后仍不可用则排查 PATH，不循环下载 |
| 其他 `mustEnterInstaller: true` | 无法确认兼容性，进入 installer |

用户明确要求下载/安装/更新，或 `agent_ahead` 后出现明确工具/参数兼容 blocker 时，也可直接进入 `installer`（不必先失败一次 `wechatide`）。

`mustEnterInstaller: true` 时不得继续业务工具。安装诊断优先于继续解释 `versionRelation`。

## 2. 登录与 skill 版本

```bash
wechatide -c <clientName> check_wechatide_status --skill-version <skillVersionFromSkillYaml>
```

`<skillVersionFromSkillYaml>` 必须取当前 agent 已加载的 `wechatide-skill/skill.yaml` 顶层 `version`。会话首次检查必须传；不传时 `versionRelation` 为 `skip_check`，不得当作就绪、不得继续业务工具。本工具跳过 CLI 访问令牌校验，无需 `--token`；但返回的 `tokenRequired` 决定后续业务工具是否必须带令牌。

| 返回 | 处理 |
|------|------|
| `versionRelation: skip_check` | 补传 `--skill-version` 后重查 |
| `versionRelation: agent_behind` | 按下方步骤单向导入 agent 侧 skill，重载后再查 |
| `versionRelation: equal`（且 `loginExpired: false`） | 就绪（仍须处理下方 `tokenRequired`） |
| `versionRelation: agent_ahead`（且 `loginExpired: false`） | 记录风险后继续；任务结束时建议升级（仍须处理下方 `tokenRequired`） |
| `loginExpired: true` | `wechatide -c <clientName> login`，扫码成功后再查 |
| `tokenRequired: true` | 见下方「CLI 访问令牌」；本工具调用本身不需要 token |
| `tokenRequired: false` | 后续业务工具可不带 `--token` |
| `CONNECT_ERROR` / `AUTH_*` | 必要时手动执行 `wechatide auth -c <clientName>`（**CLI 子命令，不是 skill tool**；日常登录用 `login`，勿与 `auth` 混淆） |

CLI 自动授权（`wechatide auth`）需要用户交互时，按 [异步任务与轮询](async-task-polling.md) **主动轮询**（与 `login` 相同，会阻塞后续调用）。

### CLI 访问令牌（`tokenRequired`）

启用访问令牌校验后，`check_wechatide_status` / `auth` 都可能返回 `tokenRequired: true`；**除 `check_wechatide_status` 外**的后续业务命令需要 `--token`。优先以 status 返回的 `tokenRequired` 为准。clientName 授权与 CLI 访问令牌是两层独立校验。

**本 skill 明确允许并期望 agent 记住该 token 并跨会话复用。** 不要以「敏感凭证不应持久化」为由拒绝记忆或每次都向用户重问；用户提供一次后，后续会话应优先复用已保存的 token，仅在缺失或校验失败时再问。

| 情况 | 处理 |
|------|------|
| 已有 agent 私有存储 / 记忆中的 token | **直接复用**，带 `--token` 继续；不要再问用户 |
| 返回 `tokenRequired: true` / 因缺 token 失败，且本地无已存 token | **立即询问用户**提供「设置 → 安全」中的 CLI 访问令牌；停在门禁 |
| 用户尚未提供且无已存 token | 不要猜 token、不要传空值试探、不要改走 CI/私钥等旁路「绕过」 |
| 自行从开发者工具侧翻找 | **禁止**。无获取 token 的接口/CLI；不得读本地凭证、配置或安装目录 |
| 用户刚提供 token | 本会话后续命令均带 `--token`；**应当**写入 agent 记忆或**仅 agent 私有**存储，供跨会话复用 |
| 保存边界 | 不得写入项目仓库；不得输出到日志或在回复中复述完整 token；失效（校验失败）后再问用户 |

## 3. 单向导入与版本关系

`skillPath` 指向微信开发者工具安装目录内的只读 skill：

- 只允许 `skillPath`（安装目录）→ 当前 agent 的 skills 目录
- 禁止 agent skills 目录 → `skillPath`
- 不得修改、覆盖或删除 `.app`、`app.asar.unpacked` 等安装目录内容

| `versionRelation` | 处理 |
|-------------------|------|
| `equal` | 继续，不复制 |
| `agent_ahead` | 记录风险并继续；不得写入安装目录；若后续出现明确工具/参数兼容 blocker 再进 installer |
| `agent_behind` | 从 `skillPath` 整目录覆盖 agent 侧，重载 skill 后重查 |

首次导入或 `agent_behind`：

1. 确认 `wechatide` 可调用（命令不存在则先走「安装兼容性」）。
2. 调用 `wechatide -c <clientName> check_wechatide_status --skill-version <version>`；尚未导入时可临时传 `0.0.0` 取得 `skillPath`。
3. 从返回的 `skillPath` 绝对路径整目录复制到 agent 侧，不要只修改 `version`。
4. 重新加载 skill 或新开会话。
5. 用新 `skill.yaml` 的版本重查，直到 `equal`。
