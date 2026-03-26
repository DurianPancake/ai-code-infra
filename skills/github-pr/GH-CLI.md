# 通过 gh CLI 直接创建 PR

适用于已安装并登录 `gh` CLI 的场景，可跳过手动填写网页表单。

## 前置检查

```bash
gh auth status
```

未登录时提示用户执行 `gh auth login`，**不要代替用户执行**。

## 创建命令

### 同仓库 PR

```bash
gh pr create \
  --title "<title>" \
  --body "<description>" \
  --base <target-branch> \
  --head <source-branch>
```

### 跨仓库 PR（fork → upstream）

```bash
gh pr create \
  --repo <upstream-owner>/<repo> \
  --title "<title>" \
  --body "<description>" \
  --base <target-branch> \
  --head <fork-owner>:<source-branch>
```

## 注意事项

- `gh` CLI 方式**在用户确认 PR 信息后**才执行，不提前运行
- 描述包含多行时，使用 `--body-file <file>` 传入临时文件更可靠
- 创建成功后 `gh` 会输出 PR URL，直接展示给用户

## 使用临时文件传入描述

```bash
cat > /tmp/pr-body.md << 'PREOF'
<description 内容>
PREOF

gh pr create \
  --repo <upstream-owner>/<repo> \
  --title "<title>" \
  --body-file /tmp/pr-body.md \
  --base <target-branch> \
  --head <fork-owner>:<source-branch>

rm /tmp/pr-body.md
```
