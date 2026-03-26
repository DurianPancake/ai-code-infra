---
name: github-pr
description: 向 GitHub 仓库创建 Pull Request，支持 fork → upstream 跨仓库 PR。收集材料（需求文档、技术方案、截图等）、生成规范 PR 标题和描述、人工 Review 确认后输出可直接打开的 PR 链接。当用户说"创建PR"、"提交PR"、"pull request"、"pr到上游"、"提交到上游"、"合并到上游"、"推送到上游"、"fork pr"时使用此技能。不用于单纯的 git commit 操作。
---

# GitHub PR - Pull Request 创建工作流

完成从材料收集到 PR 链接生成的完整流程。

## 进度清单

开始时向用户展示进度清单，每完成一步打勾更新：

```
PR 进度：
- [ ] 步骤 1：确认仓库关系和目标分支
- [ ] 步骤 2：收集材料
- [ ] 步骤 3：分析变更内容
- [ ] 步骤 4：生成 PR 标题和描述
- [ ] 步骤 5：用户 Review 确认
- [ ] 步骤 6：输出 PR 链接
```

---

## 步骤 1：确认仓库关系和目标分支

```bash
# 检查是否有未提交的变更
git status --short
git diff --cached --name-only
# 仓库信息
git remote -v
git branch --show-current
git log --oneline upstream/dev..HEAD 2>/dev/null || git log --oneline -10
# 检查项目贡献规范
ls CONTRIBUTING.md README.md 2>/dev/null
```

**若检测到未提交变更**，不继续 PR 流程。分析变更内容，推断 commit type（feat/fix/refactor 等）和 scope，直接生成 commit message 供用户确认，然后执行 `git add -A && git commit`：

```bash
git diff --stat
git diff --name-only
```

展示格式：
```
检测到未提交变更，已为你生成 commit 信息，确认后将自动提交：

commit message: <type>(<scope>): <summary>

变更文件：
  <file list>

确认提交？或告诉我需要修改的地方。
```

用户确认后执行提交，再继续 PR 流程。若用户有 code-commit skill 且希望更完整的提交流程（统计、commit-info.md 等），告知可改用该 skill 完成提交步骤。

**判断仓库模式**（影响后续所有步骤）：

```bash
git remote | grep upstream
```

| 结果 | 模式 | PR 目标 |
|---|---|---|
| 有 `upstream` remote | **fork 模式** | upstream 仓库的目标分支 |
| 无 `upstream` remote | **自有项目模式** | origin 仓库的主分支（main/dev） |

**fork 模式 — 同步上游**：

```bash
git fetch upstream
git log HEAD..upstream/<target-branch> --oneline
```

若落后上游，询问用户是否 rebase：

```bash
git rebase upstream/<target-branch>
git push origin <source-branch> --force-with-lease
```

**自有项目模式** — 跳过同步上游步骤。

详见 [CONTRIBUTING-GUIDE.md](CONTRIBUTING-GUIDE.md)。

---

确认以下信息，不明确的**必须询问用户**：

| 信息 | fork 模式 | 自有项目模式 |
|---|---|---|
| 目标仓库 | upstream remote URL | origin remote URL |
| 目标分支 | upstream 主分支（dev/main） | origin 主分支（main/dev） |
| 源仓库 | origin remote | origin remote |
| 源分支 | 当前分支 | 当前分支 |

**贡献规范检查**：读取 `CONTRIBUTING.md`（如存在），提取：目标分支要求、提交格式约定、是否需要先提 Issue。见 [CONTRIBUTING-GUIDE.md](CONTRIBUTING-GUIDE.md)。

若改动范围较大（跨模块、API 变更），提醒用户是否已与维护者提前沟通（Issue / Draft PR）。

**跨仓库 PR 格式**（fork → upstream）：
```
https://github.com/<upstream-owner>/<repo>/compare/<target-branch>...<fork-owner>:<repo>:<source-branch>?expand=1
```

**同仓库 PR 格式**：
```
https://github.com/<owner>/<repo>/compare/<target-branch>...<source-branch>?expand=1
```

---

## 步骤 2：收集材料

材料用于生成准确的 PR 描述。按优先级查找：

1. **用户明确提供的路径** — 直接读取，无需询问
2. **项目根目录** — 查找 `commit-info.md`、`CHANGELOG.md`、`docs/` 下相关文档
3. **用户指定的外部目录** — 读取需求文档（prd.md）、技术方案（trd.md）、测试文档、截图说明

```bash
# 查找常见材料文件
ls -la commit-info.md 2>/dev/null
find . -name 'prd.md' -o -name 'trd.md' -o -name 'CHANGELOG.md' 2>/dev/null | head -10
```

**没有找到任何材料时**，不停下等待，直接基于 git log 生成初版草稿，在步骤 4 展示时注明：
> 暂未找到需求/技术文档，以下描述基于 git log 生成，请补充或修改后确认。

用户可在步骤 5 Review 时补充材料路径，AI 重新生成后再确认。

---

## 步骤 3：分析变更内容

根据步骤 1 判断的模式获取本次 PR 包含的提交：

```bash
# fork 模式
git log --oneline upstream/<target-branch>..HEAD 2>/dev/null
# 自有项目模式
git log --oneline origin/<target-branch>..HEAD 2>/dev/null
# 兜底（两者均失败）
git log --oneline -10
```

逐条分析提交，识别：
- 主功能提交（feat）
- 修复提交（fix）
- 合并上游的 merge commit（排除在 PR 变更描述之外）

**推断 PR 类型**（用于步骤 4 标题）：
- 所有提交均为 `fix` → type = fix
- 有 `feat` 提交 → type = feat
- 仅重构/文档 → type = refactor / docs
- 混合类型 → 以主功能提交的 type 为准

---

## 步骤 4：生成 PR 标题和描述

见 [PR-TEMPLATE.md](PR-TEMPLATE.md) 获取完整模板。

### 标题规则

```
<type>(<scope>): <summary>
```

- type: feat / fix / refactor / docs / chore
- scope: 主要变更模块
- summary: 20 字以内，中文

### 描述原则

- 以材料内容为准，不凭空捏造细节
- 有截图说明时，用文字描述演示效果（不能嵌入本地图片）
- 合并注意事项必须包含：影响功能点、验证方式、不兼容变更（如有）
- 技术细节从 commit-info.md 或 trd.md 提取，保持准确

---

## 步骤 5：用户 Review 确认

**必须**向用户展示完整 PR 信息供审核，不得跳过：

```
请确认以下 PR 信息：

模式:     fork 模式 / 自有项目模式
目标仓库: <upstream 或 origin 的 owner/repo>
目标分支: <target-branch>
源分支:   <fork-owner>:<repo>:<source-branch>   ← fork 模式
          <source-branch>                        ← 自有项目模式

--- PR 标题 ---
<type>(<scope>): <summary>

--- PR 描述 ---
<description>

是否确认？可以告诉我需要修改的地方。
```

用户要求修改时，更新对应部分后重新展示，直到用户确认。

---

## 步骤 6：输出 PR 链接

用户确认后，输出：

1. **可直接打开的 PR 创建链接**（带 `?expand=1` 参数，自动展开描述输入框）
2. **完整 PR 描述**（供用户粘贴到 GitHub 页面）

```
PR 链接（点击打开，标题和描述需手动填入）：
<url>

--- 复制以下内容粘贴到 PR 描述框 ---
<description>
```

**自有项目模式 PR 链接**：
```
https://github.com/<owner>/<repo>/compare/<target-branch>...<source-branch>?expand=1
```

> GitHub API 方式（通过 `gh` CLI 直接创建）：见 [GH-CLI.md](GH-CLI.md)

---

## 注意事项

- 目标仓库和目标分支**必须明确**，不能假设
- PR 描述不能凭空捏造需求细节，必须基于材料或 git log
- 用户 Review 步骤不可跳过
- fork 内部的 PR（源和目标同一仓库）没有实际意义，需提醒用户
- 合并 commit（`Merge remote-tracking branch ...`）不计入 PR 变更内容
