# 仓库协作说明

本文档用于说明本仓库在 GitHub 上的分支管理规则、推荐的分支保护配置，以及 Pull Request（PR）的创建与合并流程。

当前仓库默认分支建议使用 `main` 作为稳定分支，所有代码变更通过 PR 合入，不直接向 `main` 推送。

## 1. 推荐分支策略

建议采用简单、易维护的分支模型：

- `main`：稳定分支，始终保持可发布状态
- `feature/<name>`：新功能开发分支
- `fix/<name>`：普通缺陷修复分支
- `hotfix/<name>`：线上紧急修复分支
- `chore/<name>`：工程配置、依赖升级、文档等非业务改动
- `release/<version>`：需要版本封板时再启用

分支命名示例：

```bash
feature/login-page
fix/header-style
hotfix/payment-timeout
chore/update-readme
release/v1.0.0
```

## 2. GitHub 分支规则配置

### 2.1 推荐优先使用 Rulesets

GitHub 当前更推荐使用 `Rulesets` 管理分支规则。相比传统的 `Branch protection rules`，`Rulesets` 更适合统一维护仓库规则。

配置入口：

```text
Repository -> Settings -> Rules -> Rulesets -> New ruleset -> New branch ruleset
```

如果你的仓库还在使用旧版配置，也可以在下面的位置设置分支保护：

```text
Repository -> Settings -> Branches -> Add branch protection rule
```

### 2.2 建议保护的分支

建议至少保护以下分支：

- `main`
- `release/*`（如果后续使用发布分支）
- `hotfix/*`（如果希望热修复也必须走审查）

### 2.3 `main` 分支推荐配置（按当前 Rulesets UI 对照）

建议创建一个名为 `protect-main` 的 `branch ruleset`，并按当前 GitHub Rulesets UI 配置以下项目：

| UI 位置 | 当前 UI 名称 | 建议值 | 说明 |
| --- | --- | --- | --- |
| 顶部基础信息 | Ruleset name | `protect-main` | 规则集名称，便于团队识别 |
| 顶部基础信息 | Enforcement status | `Active` | 正式生效；如果你在做元数据限制验证，当前 UI 里可能还会看到 `Evaluate` |
| 顶部基础信息 | Bypass list | 关键仓库建议留空，或仅保留极少数维护者 | Rulesets 里通过这里控制谁可以绕过规则 |
| 顶部基础信息 | Bypass list -> For pull requests only | 如需保留维护者绕过能力，优先选这个 | 允许特定角色只能通过 PR 绕过，不直接推送 |
| Target branches | Add a target -> Include default branch 或 Branch name pattern `main` | 开启 | 保护默认分支 |
| Restrictions | Restrict commit metadata | 可选 | 用于约束提交信息、作者邮箱等元数据，属于高级治理能力 |
| Restrictions | Restrict branch names | 可选 | 用于约束分支命名规范，例如强制 `feature/*`、`fix/*` |
| Branch protections | Restrict creations | 通常关闭 | 对已有 `main` 分支通常没必要，更多用于 `release/*` 之类模式分支 |
| Branch protections | Restrict updates | 通常关闭 | 标准 PR 流程一般不需要额外开启；仅在你要做“允许名单推送”时再启用 |
| Branch protections | Restrict deletions | 开启 | 防止误删主分支 |
| Branch protections | Require linear history | 推荐开启 | 保持提交历史清晰 |
| Branch protections | Require deployments to succeed before merging | 可选 | 有预发/测试环境时可开启 |
| Branch protections | Require signed commits | 可选 | 对提交签名有要求时开启 |
| Branch protections | Require a pull request before merging | 开启 | 所有改动必须通过 PR 合入 |
| Branch protections -> Require a pull request before merging -> Additional settings | Required approvals | `1` 或 `2` | 小团队可设 1，大团队建议 2 |
| Branch protections -> Require a pull request before merging -> Additional settings | Dismiss stale pull request approvals when new commits are pushed | 开启 | 提交变更后重新确认审查结果 |
| Branch protections -> Require a pull request before merging -> Additional settings | Require approval of the most recent reviewable push | 开启 | 保证最后一版代码也经过审查 |
| Branch protections -> Require a pull request before merging -> Additional settings | Require conversation resolution before merging | 开启 | 所有评论处理完成后才能合并 |
| Branch protections -> Require a pull request before merging -> Additional settings | Require review from Code Owners | 可选 | 如果仓库启用了 `CODEOWNERS`，建议开启 |
| Branch protections -> Require a pull request before merging -> Additional settings | Require merge type | 可选 | 想固定合并方式时可指定 `Squash` 或 `Rebase` |
| Branch protections -> Require a pull request before merging -> Required reviewers | 团队仓库按需开启 | 需要特定团队审批时使用；个人仓库通常不会看到 |
| Branch protections | Require status checks to pass before merging | 有 CI 时开启 | 保证自动检查通过 |
| Branch protections -> Require status checks to pass before merging -> Additional settings | Status checks | 选择实际在跑的 CI 检查项 | 例如 `build`、`test`、`lint` |
| Branch protections -> Require status checks to pass before merging -> Additional settings | Require branches to be up to date before merging | 有 CI 时推荐开启 | 这是当前 UI 中的附加项，不是顶层规则 |
| Branch protections | Block force pushes | 开启 | 防止改写历史 |
| Branch protections | Require code scanning results | 可选 | 已启用 Code Scanning 时可作为额外门禁 |
| Branch protections | Require code quality results | 可选 | 已启用 GitHub Code Quality 时可作为额外门禁 |

### 2.4 推荐配置说明

1. `main` 分支建议以 `Require a pull request before merging` 作为核心约束，确保所有改动都通过 PR 进入主分支。
2. `Required approvals`、`Dismiss stale pull request approvals when new commits are pushed`、`Require approval of the most recent reviewable push`、`Require conversation resolution before merging` 都属于 `Require a pull request before merging` 的附加设置，不会单独出现在顶层规则列表中。
3. `Require branches to be up to date before merging` 属于 `Require status checks to pass before merging` 的附加设置；如果没有先配置必需的状态检查，这个选项不会真正生效。
4. 如果仓库暂时没有 GitHub Actions 或其他 CI，不要立即开启 `Require status checks to pass before merging`，否则 PR 会被一直阻塞。
5. `Restrict who can push` 是旧版 `Branch protection rule` 的说法；在当前 `Rulesets` UI 中，对应思路通常是 `Restrict updates` 加 `Bypass list` 的组合。
6. `Do not allow bypassing the above settings` 也是旧版保护规则的说法；在当前 `Rulesets` UI 中没有这个同名开关。如果希望几乎没人能绕过规则，直接将 `Bypass list` 留空即可。
7. 如果开启 `Require linear history`，仓库层面的合并方式也要允许 `Squash merge` 或 `Rebase merge`，否则 PR 可能无法合并。
8. 如果团队对代码归属明确，建议同时配置 `CODEOWNERS`，再启用 `Require review from Code Owners`。
9. `Require code quality results` 是较新的规则项，只有在仓库实际启用了 GitHub Code Quality 时才建议开启。
10. `Restrict commit metadata` 和 `Restrict branch names` 位于当前 branch ruleset UI 的 `Restrictions` 区块，不属于传统分支保护项；如果只是先把主分支保护起来，可以暂时不启用。
11. `Restrict file paths`、`Restrict file path length`、`Restrict file extensions`、`Restrict file size` 属于 `Push ruleset`，不是 `Branch ruleset`；如果你当前创建的是 branch ruleset，在页面里不会看到这些选项。

## 3. 推荐开发流程

### 3.1 从 `main` 拉出功能分支

开发前先同步主分支，再创建新分支：

```bash
git checkout main
git pull origin main
git checkout -b feature/your-change-name
```

示例：

```bash
git checkout -b feature/add-readme-guide
```

### 3.2 本地开发并提交

开发完成后，按语义清晰的方式提交代码：

```bash
git add .
git commit -m "docs: add branch rules and pr process guide"
```

建议提交信息前缀：

- `feat:` 新功能
- `fix:` 缺陷修复
- `docs:` 文档更新
- `refactor:` 重构
- `test:` 测试相关
- `chore:` 工程维护

### 3.3 推送远程分支

```bash
git push -u origin feature/your-change-name
```

## 4. PR 创建流程

### 4.1 在 GitHub 页面创建 PR

推送分支后，可以在仓库页面通过以下流程创建 PR：

1. 打开仓库首页
2. 点击 `Compare & pull request`，或者进入 `Pull requests -> New pull request`
3. 选择目标分支 `base: main`
4. 选择来源分支 `compare: feature/your-change-name`
5. 填写 PR 标题和描述
6. 如果尚未完成，可创建 `Draft Pull Request`
7. 指定 reviewer、labels、assignee 后提交

### 4.2 使用 GitHub CLI 创建 PR

如果本地已安装 GitHub CLI，也可以使用命令创建：

```bash
gh pr create --base main --head feature/your-change-name --title "docs: add branch rules guide" --body "请审查本次文档更新"
```

## 5. PR 标题与描述建议

### 5.1 标题建议

建议标题简洁明确，直接体现本次改动目的：

```text
docs: 新增 GitHub 分支规则与 PR 流程说明
feat: 增加登录页表单校验
fix: 修复首页样式错位问题
```

### 5.2 描述模板

建议 PR 描述至少包含以下内容：

```md
## 变更背景

## 变更内容

## 测试说明

## 影响范围

## 回滚方案
```

## 6. PR 审查与合并流程

推荐采用以下流程：

1. 开发者提交 PR
2. CI 自动执行检查
3. Reviewer 审查代码并提出意见
4. 开发者根据意见继续提交修复 commit
5. 所有评论处理完成，状态检查通过
6. Reviewer 审批通过
7. 由负责人执行合并

### 6.1 推荐合并方式

建议优先使用 `Squash and merge`，优点是：

- 保持 `main` 历史简洁
- 一个 PR 最终对应一个合并提交
- 便于回溯功能变更

如果团队对提交粒度要求更高，也可以选择 `Rebase and merge`。

## 7. 合并后处理

PR 合并完成后，建议执行以下动作：

1. 删除远程功能分支
2. 本地切回 `main`
3. 拉取最新代码
4. 删除已合并的本地分支

示例：

```bash
git checkout main
git pull origin main
git branch -d feature/your-change-name
```

## 8. 最小可落地配置建议

如果当前仓库还比较简单，建议先启用下面这组最小配置：

1. 保护 `main`
2. 禁止 force push
3. 禁止删除 `main`
4. 必须通过 PR 合并
5. 至少 1 个 reviewer 审批
6. 所有评论必须 resolved

等仓库接入 CI 后，再补充：

1. 必须通过状态检查
2. 分支必须与 `main` 保持最新
3. 部署成功后才允许合并

## 9. 参考文档

- [GitHub Docs - About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [GitHub Docs - Available rules for rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)
- [GitHub Docs - Creating rulesets for a repository](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository)
- [GitHub Docs - Creating a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request)
