# 贡献者规范参考

## 提交 PR 前的项目规范确认

### 必读文档（按优先级）

| 文件 | 内容 | 不存在时 |
|---|---|---|
| `CONTRIBUTING.md` | 代码风格、提交规范、PR 流程 | 参考维护者历史 PR 的评论风格 |
| `README.md` | 项目简介、目标分支说明 | 从 git 历史推断惯例 |
| `CODE_OF_CONDUCT.md` | 社区互动规范 | 保持专业礼貌即可 |

```bash
# 检查项目是否有贡献指南
ls CONTRIBUTING.md README.md CODE_OF_CONDUCT.md 2>/dev/null
```

### 目标分支确认规则

- 优先查看 `CONTRIBUTING.md` 中的明确说明
- 其次看上游仓库的默认分支（Settings → Default branch）
- 常见约定：`main` / `dev` / `develop` / `next`
- **不确定时必须询问用户**，不能假设

### 是否需要先提 Issue

- 功能改动较大（跨模块、API 变更、架构调整）→ **建议先提 Issue 或 Draft PR 与维护者对齐**
- 小型 bug 修复、文档更新 → 可直接提 PR
- 询问用户："这个改动是否已与维护者沟通过？"

---

## 提交前同步上游

PR 提交前确保分支基于最新上游，避免合并冲突：

```bash
git fetch upstream
git rebase upstream/<target-branch>
# 如已推送，强制推送（安全方式）
git push origin <source-branch> --force-with-lease
```

> `--force-with-lease` 比 `--force` 更安全：如果远端有你不知道的新提交会拒绝推送。

---

## PR 描述增强项

在基础模板之上，以下字段可显著提升 PR 质量：

### 关联 Issue

```markdown
Closes #<issue-number>
```

放在描述末尾，GitHub 会在 PR 合并后自动关闭对应 Issue。无关联 Issue 时省略。

### 风险与回滚

```markdown
## 风险与回滚

- **风险**：xxx 场景下可能受影响
- **回滚方式**：revert 此 PR 即可 / 需同时回滚数据库迁移
```

### 平台兼容性说明

```markdown
## 测试环境

- [x] macOS 14
- [x] Windows 11
- [ ] Linux（未验证）
```

---

## CI 检查

提交 PR 后，等待 CI 结果再催促维护者 Review：

```bash
# 本地预跑常见检查，避免 CI 失败
cargo clippy -- --deny warnings   # Rust
npm run lint && npm test           # Node
pytest                             # Python
```

CI 失败时直接修复并 `git push`，PR 会自动更新，无需关闭重开。

---

## 审核沟通礼仪

| 场景 | 建议做法 |
|---|---|
| 收到修改意见 | 每条评论回复确认，修改后标记 Resolved |
| 对某条意见有异议 | 解释理由，保持开放态度，最终尊重维护者决定 |
| PR 长时间无回应 | 礼貌 @ 维护者一次，附上简短说明 |
| 维护者要求 squash | 遵从，保持提交历史整洁 |

**核心原则**：一次只做一件事；尊重项目既有风格；保持谦逊。
