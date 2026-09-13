# 贡献指南

感谢愿意为本项目出力。为了让改动顺利合入，请先花两分钟读完本文。

## 核心约定（重要）

**本项目不修改上游 [`workbuddy2api`](https://github.com/Sliverkiss/workbuddy2api) 的任何代码。**
账号轮询、并发调度、熔断、令牌刷新都由上游负责，管理端只通过 HTTP 接口与共享的
`auths/` 目录、`config.json` 与其协作。

因此**不要**在本仓库里提交针对上游 Go 代码的补丁。若问题出在上游，请到其仓库反馈。

## 环境准备

```bash
# 后端（Python 3.11+）
python -m pip install -r server/requirements.txt

# 前端（Node 20+）
cd web && npm install
```

本地无真实上游时，起一个模拟上游即可看到完整界面：

```bash
python dev/mock_upstream.py            # 监听 127.0.0.1:7863

# 终端 A：后端
WB_ADMIN_PASSWORD=admin123 \
WB_DATA_DIR=./data \
WB_AUTH_DIR=/opt/workbuddy2api/auths \
python -m uvicorn server.main:app --reload --port 7864

# 终端 B：前端（next dev 会把 /api、/v1 反代到 :7864）
cd web && npm run dev
```

## 提交前必做

### 1. 后端测试必须全绿

```bash
python -m unittest discover -s server/tests -t . -v
```

测试用临时目录模拟上游 `config.json`，不触碰真实配置，可安全反复运行。
改动配置读写逻辑时，**请一并补充回归测试** —— 历史上曾两次写坏上游配置。

### 2. 前端构建必须通过

```bash
cd web && npm run build:export
```

本项目生产环境是 `output: 'export'` 静态导出，由 FastAPI 托管，没有 Next.js 服务端。

## 代码风格

- **注释写「为什么」，不写「是什么」**。本项目注释密度较高，重点解释被否决的方案、
  边界条件与踩过的坑。改逻辑时请同步更新注释，避免注释与实现脱节。
- 后端：类型标注 + `from __future__ import annotations`，中文注释与错误文案。
- 前端：TypeScript + 函数组件，样式用 Tailwind v4 与 CSS 变量（设计令牌见
  `web/app/globals.css`）。
- 面向用户的错误信息要**可照做**（说清是什么、怎么改），而不是只抛一个状态码。

## 提交信息

采用约定式提交，中文描述：

```
fix(tasklog): 适配上游日志格式变更
feat(settings): 新增 XX 配置项
docs: README 同步 XX
security(settings): 修复 XX 明文下发
```

## Pull Request

- 一个 PR 解决一件事，便于审阅与回滚
- 描述里写清：**问题是什么 → 为什么这么改 → 怎么验证的**
- 涉及界面改动请附截图（含窄屏表现，本项目对移动端有专门适配）
- 涉及配置字段改动，请说明是否与上游 `config.json` 结构对齐
- **不要**在 PR 里包含密钥、Token、账号授权文件

## 关于 CHANGELOG

维护者会在发版时更新 `CHANGELOG.md`。你无需手动改它，但若你的改动涉及
用户可感知的行为变化，在 PR 描述里写一句「建议记入更新日志」会很有帮助。

## 发布流程（维护者）

打 tag 即自动构建并发布：

```bash
git tag v1.0.1 && git push origin v1.0.1
```

CI 会构建前端、跑测试、打包产物、从 CHANGELOG 提取对应版本段落作为发布说明，
并创建 Release 附带 `.tar.gz` / `.zip`。
