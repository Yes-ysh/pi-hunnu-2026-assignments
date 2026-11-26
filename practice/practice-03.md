# 实践课3：Pi Agent 源码安装与编译

**姓名拼音：** yangsihui
**学号：** 202430224005
**小组仓库：** https://github.com/aaa-course-project/pi-hunnu-2026
**提交 commit：** `c3cc91632d0a1950fb122605b190233f2aa0006e`

---

## 任务一 实验准备：检查并安装开发工具

### 1.1 现有环境检查

在 PowerShell 中逐条执行 `node --version`、`npm --version`、`git --version`：

```text
node  v24.22.3
npm   11.19.1
git   git version 2.56.1.windows.1
```

> **截图 S1：** 需自行截取当前终端显示这三个版本号的画面。

### 1.2 版本选择说明

根 `package.json` 声明 `"engines": { "node": ">=22.19.0" }`。本机最初的 Node 版本低于该要求，无法通过 `engines` 检查，因此按课程建议切换到 **Node.js 24 LTS**，实际使用 **v24.22.3**（LTS 代号 Krypton），npm 随附为 **11.19.1**。

选 24 而不是"刚好满足 22.19"的原因：部分工作区依赖对 Node 版本的要求比根项目更高，只满足根声明会在安装阶段触发 `EBADENGINE` 警告。本次在 Node 24 下安装，**没有出现任何 `EBADENGINE` 警告**（见任务四）。

Node 与 npm 来自 Node.js 官方 Windows x64 发行包，npm 由安装包自带，未单独安装。

---

## 任务二 下载课程仓库源码

### 2.1 下载源码

从**小组组织**内仓库下载（不是教师组织，也不是个人账号）：

```powershell
git clone --branch main https://github.com/aaa-course-project/pi-hunnu-2026.git
cd pi-hunnu-2026
```

### 2.2 确认源码来源和版本

```text
D:\projects\study\pi-hunnu-2026

origin  https://github.com/aaa-course-project/pi-hunnu-2026.git (fetch)
origin  https://github.com/aaa-course-project/pi-hunnu-2026.git (push)

main

c3cc91632d0a1950fb122605b190233f2aa0006e
```

- `origin` 指向**小组组织**的仓库，符合要求
- 当前分支 `main`
- 完整提交编号：`c3cc91632d0a1950fb122605b190233f2aa0006e`
- 工作区在开始任务前是干净的（`git status --short` 无输出）

后续所有安装与构建命令均在此仓库根目录执行。

> **截图 S2：** 需自行截取包含当前目录、`git remote -v`、分支、commit 编号的终端画面。

---

## 任务三 阅读配置，确认安装与编译的依据

### 3.1 根 package.json 的关键字段

```json
"engines": {
  "node": ">=22.19.0"
}
```

`scripts.build` 是一条串行 shell 链，没有使用 turbo/nx 之类的任务编排器：

```
cd packages/telemetry && npm run build
&& cd ../schema && npm run build
&& cd ../chord && npm run build
&& cd ../tui && npm run build
&& cd ../codemode && npm run build
&& cd ../mcp && npm run build
&& cd ../ai && npm run build
&& cd ../durable && npm run build
&& cd ../env && npm run build
&& cd ../agent && npm run build
&& cd ../coding-agent && npm run build
```

另外还有 `build:offline`，链条完全相同，只有 `ai` 那一步换成 `npm run build:offline`（跳过模型目录 codegen）。

### 3.2 报告：Node.js 版本要求在哪里？`npm run build` 实际从哪里读取命令？

**Node.js 版本要求**写在根 `package.json` 的 `engines.node` 字段（`>=22.19.0`）。这只是一个声明，npm 会在安装时检查并给出 `EBADENGINE` 警告；它不代表每个工作区依赖都只要求同一个版本——依赖包可以声明更高的下限。

**`npm run build` 的命令来源**是根 `package.json` 的 `scripts.build` 字段。npm 自己不"编译"任何东西，它只是把该字段的值交给 shell 执行。真正干活的是每个子包自己的 `scripts.build`，它们读各自的 `tsconfig.build.json`（该文件 `extends ../../tsconfig.base.json`，并覆盖 `outDir: ./dist`、`rootDir: ./src`）。

因此完整链条是：**根 scripts.build → 子包 scripts.build → 子包 tsconfig.build.json → tsc（个别包再加 esbuild 或 codegen）**。

还要区分另一套配置：根 `tsconfig.json` 只服务 `npm run check` 的 `tsc --noEmit`，它把 `@earendil-works/*` 的 `paths` 指向 `packages/*/src/*.ts`（走源码）；而 `tsconfig.build.json` 把 `paths` 指向上游的 `dist/*.d.ts`（走产物）。所以 **`check` 不需要先 build，`build` 必须先 build 上游**。

> **截图 S3：** 需自行截取 `engines.node` 与 `scripts.build` 的定义（可分两张）。

---

## 任务四 安装项目依赖

### 4.1 执行安装

在仓库根目录（确认存在根 `package.json`）执行：

```powershell
npm install --ignore-scripts
```

实际输出（末尾）：

```text
added 333 packages, and audited 356 packages in 1m

57 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

退出码 `0`，耗时约 1 分 7 秒。

> **截图 S4：** 需自行截取 `npm install --ignore-scripts` 结束时的输出与退出码，要能区分「警告」与「导致失败的错误」。

### 4.2 这条命令做了什么

- 根据 `package.json` 与 `package-lock.json` 解析、下载依赖；锁文件与声明一致时使用锁定版本
- 在本地建立 `node_modules`，并通过 workspace 机制把 `packages/*` 互相链接（`node_modules/@earendil-works/*` 是指向 `packages/*` 的链接）
- 安装编译器与打包工具（typescript、esbuild、biome 等）
- `--ignore-scripts` 跳过了安装阶段自动触发的生命周期脚本（`postinstall`、`prepare`），但**不影响**后续主动执行 `npm run build`
- 没有 `-g`，装的是**当前项目**的依赖，不会把本地修改后的 Pi 装成全局 `pi` 命令

### 4.3 警告分析

本次只出现两条 `deprecated` 类警告，均**不影响安装结果**：

```text
npm warn deprecated prebuild-install@7.1.3: No longer maintained.
npm warn deprecated node-domexception@1.0.0: Use your platform's native DOMException instead.
```

| 警告类型 | 本次是否出现 | 含义与处理 |
|---|---|---|
| `EBADENGINE` | **未出现** | Node 版本不满足某个包的要求。本次用 Node 24，已消除该问题 |
| `deprecated` | 出现 2 条 | 依赖包已弃用，只是提示，不等同于安装失败。不修改依赖版本 |
| `vulnerabilities` | `found 0 vulnerabilities` | 本次审计未报告已知漏洞。**这不等于构建成功**，也不是安全证明 |

注意：`found 0 vulnerabilities` 只表示该次审计的结果。不应据此执行 `npm audit fix --force`，那会改动课程使用的依赖版本。

---

## 任务五 本地编译与构建产物验证（重点：遇到的困难）

### 5.1 首次构建

```powershell
npm run build
```

构建日志依次进入 11 个子包。最终成功时的末尾输出：

```text
> @earendil-works/pi-coding-agent@1.5.2 copy-assets
> node ../../scripts/package-files.mjs copy "src/modes/interactive/theme/*.json" ...

Built packages\coding-agent\dist\bundle (84 files, 9.4 MiB)
```

退出码 `0`，耗时约 53 秒。

### 5.2 本次遇到的困难与解决过程（重点）

#### 困难 1：Node 版本低于项目要求

**现象：** 本机初始 Node 为 v20.x，根 `package.json` 要求 `>=22.19.0`。

**为什么必须处理：** 不只是警告问题。下游依赖可能声明更高的下限，版本不足时安装阶段就会出现 `EBADENGINE`，后续 tsc 也会因语法/API 差异失败。

**解决：** 安装 Node.js 24 LTS（v24.22.3），关闭旧终端后重新打开，用 `node --version` 确认切换成功。重新安装依赖后 `EBADENGINE` 消失。

#### 困难 2：无法直接访问 github.com

**现象：** `git clone https://github.com/...` 失败，域名被本机 `hosts` 解析到 `127.0.0.1`。

**排查：** 查看 `C:\Windows\System32\drivers\etc\hosts`，发现 `github.com`、`api.github.com`、`raw.githubusercontent.com` 等一批域名都被映射到 `127.0.0.1`；`Resolve-DnsName github.com` 返回 `127.0.0.1`，确认是 hosts 拦截而非网络故障。

**解决：** 先用镜像入口把源码拉下来，再执行 `git remote set-url origin https://github.com/aaa-course-project/pi-hunnu-2026.git` 把远端改回小组组织的正式地址，因此 `git remote -v` 显示的是真实 URL 而不是镜像地址。

#### 困难 3：原生 HTTPS 报 `schannel: SEC_E_NO_CREDENTIALS`

**现象：** 修好访问路径后，`git clone` 与 `curl` 仍失败：

```text
schannel: AcquireCredentialsHandle failed: SEC_E_NO_CREDENTIALS (0x8009030e)
```

**排查：** 报错来自 Windows 原生 TLS 后端（Schannel）无法取得凭据，与仓库地址无关；验证方法是换成 Git 自带的 OpenSSL 后端再试。

**解决：** 让 git 使用 OpenSSL：

```powershell
git -c http.sslBackend=openssl clone <url>
```

该参数生效后 clone 成功。这一步说明：**同样的 URL，换 TLS 后端能通，就说明问题在本机证书/凭据层，而不是网络或仓库权限。**

#### 困难 4：npm 缓存目录不可写（EPERM）

**现象：** `npm view` 报 `EPERM ... open 'C:\Users\<用户>\AppData\Local\npm-cache\...'`。

**原因：** 默认缓存目录在当前进程可写范围之外。

**解决：** 把缓存指到项目工作目录：

```powershell
$env:npm_config_cache = "<工作目录>\.npm-cache"
```

之后 `npm install` 与 `npm run build` 均正常。

#### 困难 5：构建在最后一步失败——`spawn EPERM`（esbuild）

**现象：** 前 10 个子包全部构建成功，最后卡在 `coding-agent` 的打包步骤：

```text
> @earendil-works/pi-coding-agent@1.5.2 build
> npm run build:unbundled && node ../../scripts/build-coding-agent-bundle.mjs
...
Error: spawn EPERM
    at Object.spawn (node:child_process:813:9)
    at ensureServiceIsRunning (...\node_modules\esbuild\lib\main.js:2272:29)
    at build (...\node_modules\esbuild\lib\main.js:2170:26)
    at file:///.../scripts/build-coding-agent-bundle.mjs:157:26
```

**排查：** 关键线索是 `ensureServiceIsRunning`。esbuild 的 JS 包不是自己完成打包，而是 **spawn 一个原生 esbuild 子进程**做实际工作，并通过管道与它通信。`EPERM` 来自子进程创建被环境禁止，不是 esbuild 版本问题、也不是代码错误——证据是前面 10 个包的 tsc 全部成功，且 `coding-agent` 的 `tsc` 与 `copy-assets` 步骤也已成功，只有最后的 bundle 步骤失败。

**为什么容易误判：** 报错位置在 `npm error workspace @earendil-works/pi-coding-agent`，看起来像"coding-agent 编译失败"。实际是**打包工具的运行权限**问题，源码本身没有问题。

**解决：** 在允许子进程运行的环境中重新执行 `npm run build`，同一条命令即通过：

```text
Built packages\coding-agent\dist\bundle (84 files, 9.4 MiB)
```

**经验：** 判断"构建失败"要区分三个层次——① 编译错误（tsc 报具体文件/行）② 打包/工具进程错误（spawn/EPERM/EACCES）③ 环境依赖缺失（找不到模块）。本次属于第 ② 类，重跑时不要盲目去改源码。

### 5.3 检查并运行构建产物

```powershell
Test-Path .\packages\coding-agent\dist\bundle\cli.js
node .\packages\coding-agent\dist\bundle\cli.js --version
```

实际结果：

```text
True
1.5.2
```

同时确认 tsc 产物入口也存在（实践课4 使用这个入口）：

```powershell
Test-Path .\packages\coding-agent\dist\cli.js
node .\packages\coding-agent\dist\cli.js --version
```

```text
True
1.5.2
```

`dist` 下的产物目录：

```text
dist/
├── bundle/      esbuild 打包产物，dist/bundle/cli.js 是 bin: pi 的启动壳
├── cli/  cli.js  core/  modes/  extensions/  utils/   tsc 逐文件产物
└── bun/
```

**层级关系要分清：** `dist/cli.js` 是 tsc 直接编译出的入口；`dist/bundle/cli.js` 是 esbuild 把 agent/ai/tui/codemode/mcp 等内联后生成的单文件启动壳。两者都指向同一版本 1.5.2，但前者依赖 `node_modules` 里的 workspace 包，后者是自包含的。

以后修改程序应改 `src` 下的源码再重新构建；直接改 `dist` 会在下次构建时被覆盖。

### 5.4 后续重新构建

首次构建成功、模型目录数据已准备好且仍有效时可用：

```powershell
npm run build:offline
```

它仍会重新编译相关包，但复用已有模型目录数据，跳过 `ai` 包的 codegen 联网步骤。

> **截图 S5：** 需自行截取完整构建结束的最后一段输出、`$LASTEXITCODE` 为 `0`、`Test-Path` 为 `True`，以及通过本地路径运行得到的版本号。

---

## 任务六 启动本地源码版本并完成一次会话

### 6.1 验证源码入口

```powershell
.\pi-test.bat --version
```

```text
1.5.2
```

这个脚本名字里有 `test`，但它是**运行本地源码的入口**，不是执行整个测试套件。当前版本的 `.bat` 调用 `.ps1`，由 Node 加载源码解析器运行 `packages/coding-agent/src/cli.ts`。

### 6.2 启动并完成一次会话

```powershell
.\pi-test.bat
```

发送：

```text
请用一句话说明源码和编译产物的区别。只回答问题，不修改文件。
```

实际收到的回复：

```text
源码是人类编写的原始代码文件；编译产物是源码经编译器转换后生成的机器可执行或可直接运行的输出文件。
```

**报告：本次如何确保运行的是本地源码？**

有三条独立证据：

1. 启动命令是 `.\pi-test.bat`，其内部走 `packages/coding-agent/src/cli.ts`，**不经过全局安装的 `pi` 命令**
2. 该次会话的终端输出里出现了只存在于本地修改中的 `[P4]` 日志（本报告实践课4 新增），全局安装的版本不会有这行
3. `--version` 输出 `1.5.2`，与本地仓库 `packages/coding-agent/package.json` 的版本号一致

**源码入口与 `dist/bundle/cli.js` 分别适合什么场景？**

| 入口 | 适合场景 | 原因 |
|---|---|---|
| `.\pi-test.bat`（源码入口） | 修改代码后的调试、验证刚改的逻辑 | 直接由 Node 解析 `src/*.ts`，改完即生效，不必先构建 |
| `node .\packages\coding-agent\dist\cli.js` | 验证 tsc 编译结果、贴近发布结构 | 走 `dist` 产物，能发现"源码能跑但编译产物不能跑"的问题 |
| `node .\packages\coding-agent\dist\bundle\cli.js` | 模拟用户真实安装的 `pi` 命令 | 自包含单文件，不依赖 `node_modules` 中的 workspace 包 |

本次练习要证明"运行的是本地源码版本"，因此用 `pi-test.bat`；后续实践课4要观察编译产物中的日志，则用 `dist/cli.js`。

> **截图 S6：** 需自行截取 `.\pi-test.bat` 启动命令、Pi 界面、自己发出的提示词与实际模型回复。**不要提交 API Key、`auth.json` 或授权回调地址。**

---

## 附：完整命令记录

```powershell
# 1. 环境
node --version        # v24.22.3
npm --version         # 11.19.1
git --version         # git version 2.56.1.windows.1

# 2. 获取源码
git clone --branch main https://github.com/aaa-course-project/pi-hunnu-2026.git
cd pi-hunnu-2026
git remote -v
git branch --show-current
git rev-parse HEAD

# 3. 安装依赖
npm install --ignore-scripts

# 4. 构建
npm run build

# 5. 验证产物
Test-Path .\packages\coding-agent\dist\bundle\cli.js
node .\packages\coding-agent\dist\bundle\cli.js --version

# 6. 运行源码版本
.\pi-test.bat --version
.\pi-test.bat
```

---

## 提交信息

- 个人报告仓库：`https://github.com/Yes-ysh/pi-hunnu-2026-assignments`
- 小组代码仓库：`https://github.com/aaa-course-project/pi-hunnu-2026`
