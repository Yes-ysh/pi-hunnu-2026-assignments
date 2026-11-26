# 实践课1：Pi Agent 安装与使用

**姓名拼音：** yangsihui
**学号：** 202430224005
**小组组织：** https://github.com/aaa-course-project

---

## 本次完成的六项任务

| 任务 | 状态 | 说明 |
|---|---|---|
| 1 本地安装并启动 Pi | 已完成 | 用官方 npm 方式安装，`pi --version` = 1.6.1 |
| 2 配置模型并完成一次会话 | 已完成 | API Key 方式，DeepSeek |
| 3 切换模型和思考级别 | 已完成 | 两个模型 + 两档思考级别实测 |
| 4A 使用 @ 引用代码文件 | 已完成 | `@pi-test/hello.py`，回复解释正确 |
| 4B 粘贴小图片 | **需手工完成** | 涉及剪贴板与终端按键，无法在纯文本环境取证 |
| 5 安装并使用一个插件 | 已完成 | `npm:pi-web-access`，安装 + 加载 + 实际调用 |
| 6 让 AI 安装 PPT Skill（选做） | 未做 | 选做项，本次未执行 |

---

## 任务一 本地安装并启动

### 1.1 安装方式说明

课程文档给的 Windows 安装命令是：

```powershell
powershell -c "irm https://pi.dev/install.ps1 | iex"
```

本机执行该命令会失败，原因是 **PowerShell 的原生 TLS（Schannel）在这台机器上无法取得凭据**：

```text
schannel: AcquireCredentialsHandle failed: SEC_E_NO_CREDENTIALS (0x8009030e)
```

这是本机证书/凭据层问题，与 pi.dev 是否可达无关。文档明确允许另一条官方路径——"已配置 Node.js 的同学也可使用官方 npm 安装方式，任选一种，无需重复安装"，因此改用 npm 安装：

```powershell
npm install -g @earendil-works/pi-coding-agent
```

安装结果：

```text
added 123 packages in 2m
9 packages are looking for funding
```

验证：

```powershell
pi --version
```

```text
1.6.1
```

### 1.2 一个需要特别注意的现象：全局版本与源码版本不一致

- 通过 npm 安装的全局 `pi`：**1.6.1**
- 小组仓库源码构建出来的是：**1.5.2**

所以直接输入 `pi` 启动的是**全局安装的 1.6.1**，不是我们仓库里的源码版本。这也是实践课3 强调必须用 `.\pi-test.bat` 或 `dist/cli.js` 才能证明"运行的是本地源码"的实际原因——两个版本号确实不同，可以直接用版本号区分。

### 1.3 启动

```powershell
mkdir pi-test
cd pi-test
pi
```

首次启动、尚未配置模型时会出现 `No models available`。这个提示说明**客户端已成功启动**，只是还没有模型凭据——它出现本身就可以作为"安装完成"的证据之一（配合版本号与工作目录）。

> **截图 S1：** 需自行截取 Pi 成功启动、版本信息与当前工作目录的画面。只有下载进度不能证明安装完成。

---

## 任务二 配置模型并完成一次会话

采用 **API Key** 方式（另一种"Sign in with an account"未使用）。

### 2.1 配置方式

文档给的是在 Pi 里输入 `/login` → `Sign in with an API key` → 选提供商 → 粘贴密钥。本次为便于复现，直接写入 Pi 的凭据文件 `~/.pi/agent/auth.json`：

```json
{
  "deepseek": {
    "type": "api_key",
    "key": "<自己的 API Key>"
  }
}
```

这个结构的依据是源码里的 `ApiKeyCredential` 类型（`packages/ai/src/auth/types.ts:17-21`）：`type: "api_key"` 加 `key` 字段，按 provider id 作为键。DeepSeek 的 provider id 是 `deepseek`。

> **注意：** `auth.json` 含明文密钥，**不要提交到任何仓库**。报告中一律不展示密钥内容。

### 2.2 完成一次会话

发送：

```text
只回复两个字：你好
```

实际回复：

```text
你好
```

会话能收到模型真实回复，说明模型配置成功。

**模型选择：** 本次使用 `deepseek/deepseek-flash`（DeepSeek V4.1 Flash）。它的模型目录信息显示支持 `reasoning: true`、输入含 `text` 与 `image`、上下文档 1M，适合本次任务三（思考级别）与任务四B（图片）。

> **截图 S2：** 需自行截取包含自己发出的提示词、模型实际回复和当前模型信息的画面。**不要提交 `auth.json`、授权回调链接或完整密钥。**

---

## 任务三 切换模型与模型思考级别

### 3.1 切换模型

Pi 的模型选择器是 `/model`（或 `Ctrl+L`）。本次用两种模型分别完成一次真实会话，记录切换前后的模型名。

**切换前：`deepseek/deepseek-flash`**

```text
请用一句话解释持续集成。
```

```text
持续集成（CI）是指开发者频繁地把代码合并到主干，每次合并都自动构建并运行测试，以便尽早发现集成错误。
```

**切换后：`deepseek/deepseek-v4-pro`**

```text
请用一句话解释持续集成。
```

```text
持续集成是一种开发实践，开发人员频繁地将代码变更合并到共享主干，每次合并都自动触发构建和测试，从而尽早发现并修复集成问题。
```

切换后仍能正常会话，说明模型切换生效。两个模型都指向 `api.deepseek.com`，但模型 id、定价和能力不同（`deepseek-flash` 支持图片输入与思考级别映射，`deepseek-v4-pro` 只支持文本）。

### 3.2 切换思考级别

Pi 里用 `/thinking` 选择，或 `Shift+Tab` 循环切换当前模型支持的级别。`deepseek-flash` 的思考级别映射支持 `low` / `high` / `max`。

**`--thinking high`**

```text
持续集成（CI）是指开发者频繁地把代码合并到主干，每次合并都自动构建并运行测试，以便尽早发现集成错误。
```

**`--thinking low`**

```text
持续集成（CI）是指开发者频繁地把代码合并到主干，每次合并都自动触发构建和测试，从而尽早发现集成错误。
```

两组都能正常回答，说明不同思考级别下会话仍然可用。

### 3.3 报告：选择了什么，以及任务复杂度如何影响选择

| 场景 | 我的选择 | 理由 |
|---|---|---|
| 日常问答、简单改写、格式转换 | `deepseek-flash` + `low` | 响应快、用量低；这类任务不需要多步推理，高思考级别只会增加等待时间 |
| 解释一段代码、写一条启动说明 | `deepseek-flash` + `high` | 需要一点推理和取舍，但仍属中等复杂度 |
| 跨文件分析、依赖关系梳理、找 bug | `deepseek-v4-pro` + `high`/`max` | 上下文长、需要更强的推理和指令遵循；这类任务一次答错的代价高于多花的时间和用量 |

要点澄清：**思考级别是模型的推理设置，不是换模型，也不是显示/隐藏思考文本。** 提高思考级别不保证每次答案更好，但会增加等待时间和 token 用量；部分模型不支持调整，遇到这种情况应换用支持该设置的模型（本次 `deepseek-v4-pro` 就不支持思考级别映射）。

> **截图 S3：** 需自行截取模型切换前后的名称，以及思考级别切换前后的状态（可用两张组合）。

---

## 任务四 引用代码并粘贴小图片

### 4A 使用 @ 引用代码文件

在 `pi-test` 文件夹创建 `hello.py`：

```python
def add(a, b):
    return a + b

print(add(2, 3))
```

在该文件夹启动 Pi，输入 `@` 后从候选列表选择 `hello.py`，再发送：

```text
请解释 @pi-test/hello.py 的功能，说明运行后输出什么。只解释，不修改文件。
```

实际回复：

```text
**`pi-test/hello.py`** 内容（3 行）：

def add(a, b):
    return a + b

print(add(2, 3))

**功能**：定义一个函数 `add(a, b)`，返回两数之和；随后调用 `add(2, 3)` 并打印结果。

**运行输出**：`5`
```

回复正确解释了 `add` 函数并给出输出 `5`，说明 `@` 引用生效：模型确实拿到了文件内容。

这一轮的 `[P4]` 日志也印证了引用是"读文件"实现的：

```text
[P4] Agent 即将请求模型
[P4] 工具结果已加入上下文
[P4] Agent 即将请求模型
[P4] 工具结果已加入上下文
[P4] Agent 即将请求模型
```

3 轮模型调用、2 次工具结果——`@` 文件引用在底层是让 Agent 调用读文件的工具，**不是**把文件内容拼进提示词字符串。注意：只打出 `@` 而没有从候选列表选中文件，不算完成引用。

> **截图 S4：** 需自行截取引用的文件名、问题和实际回复。

### 4B 粘贴小图片并测试（需手工完成）

这一项需要在**终端里把图片放进剪贴板再粘贴**，纯文本管道环境无法完成，因此本次没有可提交的图片证据。操作要点如下（按文档）：

1. 准备一张约 200×200 像素的小图（例如画图里画一个蓝色方块，旁边写数字 7）
2. **先截取图片内容到剪贴板**，不要只复制图片文件路径
3. 切换到明确支持图片输入的模型（`deepseek-flash` 的 `input` 含 `image`，可用）
4. 在 Pi 输入区按 `Alt+V` 粘贴（部分版本也支持 `Ctrl+V`；macOS 通常 `Ctrl+V`），以 `/hotkeys` 为准
5. 确认出现图片附件提示后发送：`请描述图片中的颜色、形状和数字。`
6. 检查回复是否说出"蓝色 / 方块 / 7"

若 `Alt+V` 无反应：确认剪贴板里确实是**图像**（不是文件路径）、检查终端快捷键冲突、用 `/hotkeys` 核对绑定。

> **截图 S5：** 需自行截取已附加图片的提示、问题与实际回复，并同时附上原始小图片便于核对识别结果。

---

## 任务五 安装并使用一个插件

### 5.1 选择插件

选用 **`pi-web-access`**（来源 npm 包 `pi-web-access`），作用：为 Pi 增加网络访问能力，让模型可以抓取网页内容作为工具结果。无需额外配置密钥。

### 5.2 安装

```powershell
pi install npm:pi-web-access
```

实际输出：

```text
Installing npm:pi-web-access...

added 211 packages in 19s
Installed npm:pi-web-access
```

这是**个人级安装**（未加 `-l`），因此对该用户的所有项目生效。若只想当前项目使用，应进入项目目录执行 `pi install npm:pi-web-access -l`（必要时加 `--approve` 确认信任该项目）。

### 5.3 核对是否真的加载

只用安装命令的截图不能证明加载成功。用 `pi list` 核对来源：

```text
User packages:
  npm:pi-web-access
    D:\Users\...\.pi\agent\npm\node_modules\pi-web-access
```

然后**退出原 Pi 会话，重新单独启动 `pi`**，启动信息里会出现 `[Extensions]` 区域并列出该插件。

> **截图 S6a：** 需自行截取安装成功输出。
> **截图 S6b：** 需自行截取重新运行 `pi` 后的 `[Extensions]` 区域与插件名称。

### 5.4 试用一个最小功能

```text
请获取 https://example.com 的页面标题
```

实际回复：

```text
页面标题是 **Example Domain**。

注：此环境没有可用的 bash（Git Bash 未安装），因此改用抓取工具获取了页面内容。
```

标题正确（example.com 的标题确实是 `Example Domain`），说明插件的抓取功能实际可用。

关于那句"没有 bash"的说明：Pi 的若干能力依赖 `bash`（Windows 上通常由 Git for Windows 提供），当前 `bash` 不在 PATH 上，因此模型改用了插件提供的抓取工具。这正好说明一个要点——**"插件加载成功"与"联网功能可用"是两件事**，应分别记录；本次两件都成立，只是走了抓取路径而非 shell 路径。

> **截图 S6c：** 需自行截取这次插件功能调用的工具调用记录与结果。

---

## 任务六 让 AI 安装一个 PPT Skill（选做，本次未执行）

选做项，本次未执行。若要做，要点是（供后续参考）：

- 提示词要求从 `https://github.com/anthropics/skills` 下载 **`skills/pptx` 整个目录**，安装到当前项目的 `.pi/skills/pptx`
- 必须保留 `SKILL.md`、`LICENSE.txt` 与 `scripts/` 等支持文件，并记录源 commit
- **不要**把 Skill 当作 npm 插件执行 `pi install npm:pptx`
- 装完要完全退出再启动 `pi`，检查 `[Skills]` 中是否出现 `pptx`，再用 `/skill:pptx` 实际调用
- 层次要分清：**安装成功、发现成功、调用成功、生成 PPT 成功**是四件事，前三个必做

---

## 附：本次实测命令与环境

```powershell
# 安装（npm 官方方式）
npm install -g @earendil-works/pi-coding-agent
pi --version            # 1.6.1

# 模型配置：写入 ~/.pi/agent/auth.json（provider id: deepseek, type: api_key）
# 一次会话
pi -p "只回复两个字：你好" --model deepseek/deepseek-flash

# 模型切换
pi -p "请用一句话解释持续集成。" --model deepseek/deepseek-v4-pro

# 思考级别
pi -p "请用一句话解释持续集成。" --model deepseek/deepseek-flash --thinking high
pi -p "请用一句话解释持续集成。" --model deepseek/deepseek-flash --thinking low

# @ 引用
pi -p "请解释 @pi-test/hello.py 的功能，说明运行后输出什么。只解释，不修改文件。"

# 插件
pi install npm:pi-web-access
pi list
pi -p "请获取 https://example.com 的页面标题"
```

> **安全提示：** 本报告不包含任何 API Key。提交前请检查截图里是否带出了密钥、`auth.json` 内容或授权回调地址。

---

## 提交信息

- 个人报告仓库：`https://github.com/Yes-ysh/pi-hunnu-2026-assignments`
- 小组代码仓库：`https://github.com/aaa-course-project/pi-hunnu-2026`
