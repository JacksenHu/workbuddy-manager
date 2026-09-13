# 启用 Actions 并发布 v1.0.17 —— 操作步骤

> 背景：`JacksenHu/workbuddy-manager` 是 `ithtelab/workbuddy-manager` 的 **fork**。
> GitHub 对 fork 有硬性策略：**工作流默认不运行，必须在网页上人工点一次启用**。
> 这一步无法用 API 或 token 代做（已实测：`workflow_dispatch` 返回 422），
> 所以需要你本人操作。预计耗时 1 分钟。

---

## 第 1 步：打开 Actions 页面

浏览器访问：

```
https://github.com/JacksenHu/workbuddy-manager/actions
```

你会看到一个提示条，大意是：

> **Workflows aren't being run on this forked repository**
> （此 fork 仓库的工作流未运行）

点这个提示条里的绿色按钮：

```
I understand my workflows, go ahead and enable them
（我了解我的工作流，继续启用）
```

**这一步就是关键动作**，点完 fork 的 Actions 限制即解除。

---

## 第 2 步：手动触发 Release（补发 v1.0.17）

启用后，左侧栏应能看到 **Release** 工作流。

1. 点左侧栏的 **Release**
2. 右侧会出现一个下拉按钮 **Run workflow**
3. 点开它，在 **tag** 输入框里填写：

   ```
   v1.0.17
   ```

   > ⚠️ 这个输入框**必填**，不填直接点运行会失败。

4. 点绿色的 **Run workflow** 按钮

几秒后刷新页面，会看到一条新的运行记录，状态为 `Queued` → `In progress`。

---

## 第 3 步：等它跑完

整条流水线大约 **3–6 分钟**，依次执行：

| 步骤 | 做什么 |
|---|---|
| 解析版本号 | 从 tag 提取 `v1.0.17` |
| 安装 Node | Node 20 |
| 构建前端静态导出 | `npm ci` + `npm run build:export` |
| 校验构建产物 | 确认 `web/out/index.html` 存在 |
| 后端语法与导入自检 | `compileall` |
| **后端单元测试** | 162 项（你本地已验证全绿） |
| 组装发布目录 | 打包 `server/` + `web/out/` + `deploy/` + `docs/` |
| 生成发布说明 | 从 CHANGELOG 提取 v1.0.17 段落 |
| **创建 Release 并上传产物** | 生成 `.tar.gz` / `.zip` |

全部变绿（✅）即发布成功。

---

## 第 4 步：验证结果

发布成功后：

- **Release 页面**：https://github.com/JacksenHu/workbuddy-manager/releases
  应能看到 `v1.0.17`，附带两个可下载文件：
  - `workbuddy-manager-v1.0.17.tar.gz`
  - `workbuddy-manager-v1.0.17.zip`

- **管理端「一键更新」** 从此能正确识别你自己的仓库版本了
  （之前查的是原作者仓库，永远发现不了你的改动 —— 这是本次修复的核心问题）。

---

## 如果遇到问题

### 情况 A：Actions 页面没有「I understand...」提示条

说明 Actions 可能已启用过，或权限设置有问题。直接看第 2 步能否找到「Run workflow」按钮。

### 情况 B：运行失败在「构建前端」步骤

大概率是 npm 依赖问题。把失败步骤的日志（红色部分）发我，我来定位。

### 情况 C：失败在「创建 Release」步骤

报权限错误的话告诉我 —— 虽然 `release.yml` 已声明 `permissions: contents: write`，
但仓库的 Actions 全局权限设置也可能需要调整（Settings → Actions → General →
Workflow permissions → 勾选 **Read and write permissions**）。

### 情况 D：想换成非 fork 的独立仓库

如果觉得 fork 的限制太麻烦（虽然只有这一次人工步骤），
可以考虑把仓库脱离 fork 关系，或用 GitHub 的 "Leave fork network" 功能。
需要的话我帮你操作。

---

## 附：为什么我无法代做这一步

已尝试并确认无效的路径（留档，避免重复踩）：

| 尝试 | 结果 |
|---|---|
| `PUT /repos/.../actions/workflows/{id}/enable` | 返回 200，但无实际作用（workflow 本来就是 `active`） |
| `POST /repos/.../actions/workflows/{id}/dispatches` | **422 Unprocessable Entity** |
| 查询 `actions/runs` | 始终 `total_count = 0` |

GitHub 官方文档明确规定：

> Workflows don't run in forked repositories by default.
> **You must enable GitHub Actions in the Actions tab of the forked repository.**

这是平台级的安全设计（防止 fork 后自动执行他人代码），**任何 token 都绕不过**。
