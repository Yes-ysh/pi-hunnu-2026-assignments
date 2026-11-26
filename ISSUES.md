# 作业 Issue 提交文案（五条）

在课程作业仓库提交：https://github.com/hunnuOpenSourceSoftware2026/assignments/issues

每条 Issue 的标题直接复制下方"标题"，正文复制对应代码块。**提交前把 `[yangsihui]` 替换成自己的姓名拼音**（例如 `[zhangsan]`）。

---

## 实践课1

**标题**

```
实践课1： Pi Agent安装与使用 [yangsihui]
```

**正文**

```markdown
## 实践课1：Pi Agent 安装与使用

- 个人报告：https://github.com/Yes-ysh/pi-hunnu-2026-assignments/blob/main/practice/practice-01.md
- 小组代码仓库：https://github.com/aaa-course-project/pi-hunnu-2026

### 完成情况

| 任务 | 状态 |
|---|---|
| 1 本地安装并启动 Pi | 已完成，`pi --version` = 1.6.1 |
| 2 配置模型并完成一次会话 | 已完成，API Key 方式（DeepSeek） |
| 3 切换模型与思考级别 | 已完成，`deepseek-flash` ↔ `deepseek-v4-pro`，`--thinking high/low` |
| 4A `@` 引用代码文件 | 已完成，引用 `hello.py`，回复正确给出输出 `5` |
| 4B 粘贴小图片 | 已完成，截图见报告 |
| 5 安装并使用插件 | 已完成，`npm:pi-web-access`，安装 + 加载 + 抓取网页标题 |
| 6 安装 PPT Skill（选做） | 未做 |

### 说明

- 本机 PowerShell 原生 TLS（Schannel）报 `SEC_E_NO_CREDENTIALS`，无法执行 `irm https://pi.dev/install.ps1 | iex`，故按文档允许的官方 npm 方式安装：`npm install -g @earendil-works/pi-coding-agent`。
- 全局安装版本为 1.6.1，小组仓库源码构建版本为 1.5.2，两者版本号不同。
- 报告中未包含任何 API Key / `auth.json` 内容。

运行证据见仓库 `evidence/09`–`evidence/13`。
```

---

## 实践课2

**标题**

```
实践课2：组织建仓与源码分析 [yangsihui]
```

**正文**

```markdown
## 实践课2：组织建仓与源码分析

- 个人报告：https://github.com/Yes-ysh/pi-hunnu-2026-assignments/blob/main/practice/practice-02.md
- 小组代码仓库：https://github.com/aaa-course-project/pi-hunnu-2026
- 本人实际 clone 地址（文本）：https://github.com/aaa-course-project/pi-hunnu-2026.git
- 本地提交编号：`c3cc91632d0a1950fb122605b190233f2aa0006e`（分支 `main`）

### 报告包含

1. 源码获取：组织仓库信息、本人 clone 后的目录 / 远端 / 分支 / commit 截图、源码目录结构
2. 模块分析表：11 个构建相关子包的功能、build 实际动作与产物、直接内部依赖、依据文件与行号
3. 依赖关系图：Mermaid 图源（使用方 → 被依赖方，实线运行时 / 虚线仅类型），含层级划分依据
4. 三个问题的回答：根与子模块 build 各读什么配置、构建顺序与依赖的关系、一条涉及四模块的依赖路径分工

### 本人重点分析模块

`packages/agent`（Agent 执行层）

### 分析中发现与常见假设不符的地方

- 相邻构建不等于互相依赖：10 组相邻对中 6 组无依赖关系
- 根 build 只覆盖 11 个包，`packages/evals`、`apps/p2`、`apps/p3` 均无 build 脚本
- `ai` 包的 build 会写回源码树（`src/models.generated.ts`、`src/providers/data/`），不只写 `dist`
- `tui`/`env` 的原生部分由独立脚本构建，不在 `npm run build` 内
- 根 build 不做类型检查；`npm run check` 才做，且其 `paths` 指向 `src`，与 build 指向 `dist` 相反
```

---

## 实践课3

**标题**

```
实践课3：Pi Agent源码安装与编译 [yangsihui]
```

**正文**

```markdown
## 实践课3：Pi Agent 源码安装与编译

- 个人报告：https://github.com/Yes-ysh/pi-hunnu-2026-assignments/blob/main/practice/practice-03.md
- 小组代码仓库：https://github.com/aaa-course-project/pi-hunnu-2026

### 环境

```
Node.js  v24.22.3  (LTS)
npm      11.19.1
git      2.56.1.windows.1
```

### 关键结果

- `npm install --ignore-scripts`：`added 333 packages`，`found 0 vulnerabilities`，退出码 0，无 `EBADENGINE`
- `npm run build`：退出码 **0**，末尾 `Built packages\coding-agent\dist\bundle (84 files, 9.4 MiB)`
- `Test-Path .\packages\coding-agent\dist\bundle\cli.js` → `True`
- `node .\packages\coding-agent\dist\bundle\cli.js --version` → `1.5.2`
- `.\pi-test.bat --version` → `1.5.2`，并成功完成一次真实会话

### 遇到的困难与解决（报告第五节为详述）

1. Node 版本低于 `engines.node >=22.19.0` → 升级到 Node.js 24 LTS
2. 本机 `hosts` 把 `github.com` 等解析到 `127.0.0.1` → 改走镜像入口完成 clone
3. 原生 HTTPS 报 `schannel: SEC_E_NO_CREDENTIALS` → `git -c http.sslBackend=openssl` 改用 OpenSSL 后端
4. npm 缓存目录不可写（`EPERM`）→ `$env:npm_config_cache` 指向工作目录
5. 构建最后一步 `spawn EPERM`（esbuild 需 spawn 原生服务进程）→ 在允许子进程的环境中重跑，同一命令即通过

第 5 条是本次最值得记录的一条：报错出现在 `coding-agent` 的 workspace 位置，看起来像该包编译失败，实际是打包工具的子进程权限问题，源码无误。报告里按"编译错误 / 工具进程错误 / 环境缺失"三层做了辨析。

运行证据见仓库 `evidence/01`、`evidence/02`、`evidence/08`。
```

---

## 实践课4

**标题**

```
实践课4：Pi Agent模型调用与运行流程 [yangsihui]
```

**正文**

```markdown
## 实践课4：Pi Agent 的模型调用与运行流程

- 个人报告：https://github.com/Yes-ysh/pi-hunnu-2026-assignments/blob/main/practice/practice-04.md
- 小组代码仓库：https://github.com/aaa-course-project/pi-hunnu-2026

### 源码修改

`packages/agent/src/agent-loop.ts` 两处 `console.error`，行号与文档一致：

- 第 285 行（`currentContext.messages.push(result);` 之后）：`[P4] 工具结果已加入上下文`
- 第 413 行（`const response = await streamFunction(...)` 之前）：`[P4] Agent 即将请求模型`

差异见报告 1.3 节的 diff。

### 运行结果（同一模型 `deepseek/deepseek-flash`，各自新会话）

**会话 A：直接问答** —— "请直接用一句话解释什么是 Git 分支。"

```
[P4] Agent 即将请求模型
（模型一句话回答）
```

未调用工具，`即将请求模型` 1 条，1 轮模型调用。

**会话 B：读取文件** —— "请用 read 工具读取当前目录的 practice4.txt，并告诉我文件中的实验代号。"

```
[P4] Agent 即将请求模型
[P4] 工具结果已加入上下文
[P4] Agent 即将请求模型
文件 `practice4.txt` 中的实验代号是 **YSH-P4-7741**。
```

调用了 `read` 工具，`即将请求模型` 2 条、`工具结果已加入上下文` 1 条，2 轮模型调用，顺序与文档给出的典型顺序一致。

### 两个问题

- **谁实际读了文件**：不是模型。模型只输出 `toolCall`（name=`read`、参数为路径）；真正读文件的是本机 `packages/coding-agent/src/core/tools/read.ts` 的 `createReadToolDefinition()`，由 `agent-loop.ts` 的 `executePreparedToolCall()` 调用 `prepared.tool.execute(...)`。该工具流式优先、缓冲兜底，图片才整文件读取。
- **下一轮多带了什么**：多带一条 toolResult 消息（文件正文），由 `currentContext.messages.push(result)` 写入，下一轮连同上文一起发给模型。

报告 3.5 节附简图，标出 coding-agent、agent、ai 与远程模型服务的边界：文件系统只在 `coding-agent` 的工具里被访问，远程服务只收到工具返回的文本。
```

---

## 实践课5

**标题**

```
实践课5：Pi Agent工具调用与会话恢复 [yangsihui]
```

**正文**

```markdown
## 实践课5：Pi Agent 工具调用与会话恢复

- 个人报告：https://github.com/Yes-ysh/pi-hunnu-2026-assignments/blob/main/practice/practice-05.md
- 小组代码仓库：https://github.com/aaa-course-project/pi-hunnu-2026

> 文件名说明：文档要求 `practive-05.md`（原文缺 `c`），本仓库提交为 `practice-05.md` 以保持命名一致。

### 图 1：单轮工具调用的代码级时序图

见报告 A.3 节（Mermaid 图源，五条生命线：用户 / 交互与会话层 / Agent 执行层 / 模型请求层 / `read` 工具）。

实测一轮：`read` 读取 `handoff.txt`，日志为

```
[P4] Agent 即将请求模型
[P4] 工具结果已加入上下文
[P4] Agent 即将请求模型
```

### 图 2：恢复旧会话与新建会话的对照图

见报告 B.3 节（Mermaid 图源，两条分支）。

实测结果：

| 对比项 | `-c` 恢复旧会话 | 新建会话 |
|---|---|---|
| 提问 | "接着上次的待办，给 README 起草一条启动说明，写出交接单里的完整启动命令。只用这次会话已有的信息。不要调用工具。" | 完全相同 |
| 是否给出 A 阶段读到的命令 | **给出** `npm run demo -- --profile aaa-4821` | **没有**，只给占位符 |
| 是否调用工具 | 否 | 否 |

关键结论：`/new` 不是清空同一个 `Agent` 的历史，而是**替换会话**——新建空的 `SessionManager`、中止并 dispose 旧会话、再建全新 `AgentSession`/`Agent`。代码里 `Agent.reset()` 虽然存在，但 `packages/coding-agent/src` 下没有任何引用。

运行证据见仓库 `evidence/05`–`evidence/07`。
```

---

## 提交前自检

- [ ] 五条标题里的 `[yangsihui]` 都已替换
- [ ] 个人仓库已公开（否则教师无法访问链接）
- [ ] 报告里没有 API Key、`auth.json` 内容、授权回调地址
- [ ] 图片链接能正常显示（本仓库报告用 Mermaid 图源，GitHub 原生渲染）
- [ ] 每条 Issue 都在课程仓库 `hunnuOpenSourceSoftware2026/assignments` 下提交
