# 湖南师范大学 2026《系统分析与设计》实践课作业

**姓名拼音：** （待填）
**学号：** （待填）
**班级：** 24 级软件工程（待填班级）
**小组组织：** [aaa-course-project](https://github.com/aaa-course-project)
**小组代码仓库：** [aaa-course-project/pi-hunnu-2026](https://github.com/aaa-course-project/pi-hunnu-2026)

本仓库用于提交《系统分析与设计》课程的实践课作业。课程基于开源项目 [Pi Agent](https://github.com/earendil-works/pi) 进行系统分析与设计学习。

---

## 作业目录

| 实践课 | 报告 | 主要内容 |
|---|---|---|
| 实践课1 | [Pi Agent 安装与使用](practice/practice-01.md) | 客户端安装、模型配置、模型与思考级别切换、`@` 引用文件、插件安装与调用 |
| 实践课2 | [组织建仓与源码分析](practice/practice-02.md) | 小组仓库获取、11 个子包功能与依赖分析、模块层级依赖图、构建顺序与依赖关系 |
| 实践课3 | [Pi Agent 源码安装与编译](practice/practice-03.md) | 环境准备、依赖安装、`npm run build`、构建产物验证、困难与解决过程 |
| 实践课4 | [模型调用与运行流程](practice/practice-04.md) | `agent-loop.ts` 加日志、重新编译、A/B 会话对比 `[P4]` 日志顺序 |
| 实践课5 | [工具调用与会话恢复](practice/practice-05.md) | 单轮工具调用代码级时序图、恢复会话与新建会话对照图 |

> 实践课5 的课程文档要求文件名是 `practive-05.md`（原文缺 `c`）。本仓库提交为 `practice-05.md` 以保持命名一致；若需严格按原文，可改名。

## 提交文案

[`ISSUES.md`](ISSUES.md) 保存了五条课程 Issue 的标题与正文，可直接复制提交到课程仓库；提交前把标题里的 `[姓名拼音]` 替换成自己的姓名拼音。

## 原始证据

`evidence/` 目录保存了实际运行的终端记录（纯文本），用于佐证报告中的结论：

| 文件 | 内容 |
|---|---|
| `01-npm-install.log` | `npm install --ignore-scripts` 的完整输出与退出码 |
| `02-npm-build-summary.log` | `npm run build` 的末尾输出、产物规模与退出码 |
| `03-sessionA.log` | 实践课4 会话 A（直接问答）的 `[P4]` 日志与回复 |
| `04-sessionB.log` | 实践课4 会话 B（`read` 工具）的 `[P4]` 日志与回复 |
| `05-p5-A.log` | 实践课5 A 阶段：`read` 读取 `handoff.txt` |
| `06-p5-B1-resume.log` | 实践课5 B 阶段：`-c` 恢复会话后的回答 |
| `07-p5-B2-new.log` | 实践课5 B 阶段：新建会话的对照回答 |
| `08-p3-pi-test-session.log` | 实践课3：`.\pi-test.bat` 运行本地源码并完成一次会话 |
| `09-p1-at-file.log` | 实践课1：`@hello.py` 引用的实际回复 |
| `10-p1-model-switch.log` | 实践课1：切换到 `deepseek-v4-pro` 后的会话 |
| `11-p1-thinking-high.log` / `12-p1-thinking-low.log` | 实践课1：思考级别 high / low 的对比 |
| `13-p1-plugin-test.log` | 实践课1：`pi-web-access` 插件抓取网页标题 |

## 关于本机环境的两点说明

1. **GitHub 直连受限。** 本机 `hosts` 将 `github.com` 等域名解析到 `127.0.0.1`，因此 clone 与 push 通过镜像入口完成，`origin` 记录为镜像地址；报告中同时写明了真实仓库地址。
2. **原生 HTTPS 报错。** 本机 Git/curl 走 Schannel 时报 `SEC_E_NO_CREDENTIALS`，通过 `git -c http.sslBackend=openssl` 改用 OpenSSL 后端后恢复正常。这类现象说明问题在**本机证书/凭据层**，不是网络或仓库权限问题。

## 环境版本

```text
Node.js  v24.22.3   (LTS, Krypton)
npm      11.19.1
git      2.56.1.windows.1
```

本地源码构建版本：`1.5.2`（`packages/coding-agent`）
npm 全局安装版本：`1.6.1`

两者版本号不同，正说明"直接输入 `pi`"跑的是全局版本，要证明运行本地源码必须用 `.\pi-test.bat` 或 `node packages/coding-agent/dist/cli.js`。
