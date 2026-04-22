# 版本发布流程

本文档描述如何为 pi-subagent 发布新版本。

## 版本号规则

遵循 [Semantic Versioning](https://semver.org/lang/zh-CN/)（语义化版本）：

| 版本类型 | 说明 | 示例 |
|---------|------|------|
| `MAJOR` | 破坏性变更，不向后兼容 | `2.0.0` |
| `MINOR` | 新增功能，向后兼容 | `1.1.0` |
| `PATCH` | 修复 bug，向后兼容 | `1.0.1` |

## 发布步骤

### 1. 确认变更已就绪

```bash
cd /path/to/pi-subagent

# 检查当前状态
git status

# 确保所有修改都已提交
git add -A
git commit -m "feat: 描述这次变更"
```

### 2. 更新版本号

使用 `npm version` 自动更新 `package.json` 并创建 git tag：

```bash
# 修复 bug（1.0.0 -> 1.0.1）
npm version patch

# 新增功能（1.0.1 -> 1.1.0）
npm version minor

# 破坏性变更（1.1.0 -> 2.0.0）
npm version major
```

> `npm version` 会自动：
> 1. 更新 `package.json` 中的 `version` 字段
> 2. 创建 `package-lock.json`（如果有）
> 3. 自动执行 `git commit`（提交版本变更）
> 4. 自动执行 `git tag`（创建版本标签）

### 3. 推送到远程仓库

```bash
# 推送代码
git push origin main

# 推送标签
git push origin --tags
```

### 4. 验证发布

在 GitHub 上检查：

1. 打开 https://github.com/ekil1100/pi-subagent/releases
2. 确认新版本标签已出现
3. 点击标签查看对应的代码

## 示例：完整的发布流程

```bash
cd /Users/like/workspace/pi-subagent

# Step 1: 开发并提交
git add -A
git commit -m "feat: 支持自定义最大并发数"

# Step 2: 更新版本号（假设是新增功能）
npm version minor

# Step 3: 推送
git push origin main
git push origin --tags

# 完成！用户现在可以安装 v1.1.0
```

## 用户如何更新

### 自动更新（未锁定版本）

如果用户安装时没有指定版本标签：

```bash
pi install git:github.com/ekil1100/pi-subagent
```

执行 `pi update` 即可自动拉取最新代码：

```bash
pi update
```

### 手动更新到特定版本

如果用户需要特定版本：

```bash
# 先移除旧版本
pi remove git:github.com/ekil1100/pi-subagent

# 安装指定版本
pi install git:github.com/ekil1100/pi-subagent@v1.1.0
```

### 锁定版本（推荐用于项目）

在团队项目中，建议在 `.pi/settings.json` 中锁定版本，避免自动更新引入意外变更：

```json
{
  "packages": [
    "git:github.com/ekil1100/pi-subagent@v1.0.0"
  ]
}
```

这样 `pi update` 会跳过已锁定的包，确保团队成员使用一致的版本。

## 查看历史版本

```bash
# 本地查看所有标签
git tag -l

# 查看带注释的标签信息
git show v1.0.0

# 查看版本历史
git log --oneline --decorate
```

## 撤销发布

如果发布了错误的版本：

```bash
# 删除本地标签
git tag -d v1.0.1

# 删除远程标签
git push origin --delete v1.0.1

# 如果需要，回退提交
git reset --hard HEAD~1
git push origin main --force
```

> ⚠️ **注意**：如果已有用户安装了该版本，删除标签不会影响已安装的代码。建议直接发布修复版本，而不是删除旧版本。
