# 代码质量检查

本项目使用自动化代码质量检查工具来确保代码符合规范。在提交代码前，请确保通过所有检查。

## 自动检查

### CI 中的检查

每次 Push 到 `main` 分支或创建 Pull Request 时，GitHub Actions 会自动运行代码检查：

- **Lint 检查**：使用 Ruff 检查代码风格和潜在问题
- **格式化检查**：确保代码格式符合规范
- **类型检查**（可选）：使用 mypy 进行类型检查

如果检查失败，PR 将无法合并。请修复所有问题后重新提交。

### 本地检查

在提交代码前，建议在本地运行检查：

```bash
# 安装开发依赖
pip install -r requirements-dev.txt

# 运行 Ruff linting
ruff check .

# 运行格式化检查
ruff format --check .

# 自动修复可修复的问题
ruff check . --fix
ruff format .
```

## 使用 Pre-commit Hooks（推荐）

Pre-commit hooks 会在 `git commit` 时自动运行检查，避免提交不符合规范的代码。

### 安装

```bash
# 安装 pre-commit
pip install pre-commit

# 安装 hooks
pre-commit install

# 手动运行所有文件的检查
pre-commit run --all-files
```

### 使用

安装后，每次 `git commit` 时会自动运行检查。如果检查失败，提交会被阻止，你需要修复问题后重新提交。

## 配置说明

代码检查的配置位于 `pyproject.toml` 文件中，主要包括：

- **Ruff 配置**：代码风格和检查规则
- **Mypy 配置**：类型检查规则（可选）

### 常见问题修复

#### 导入未使用

```python
# 错误：导入了但未使用
import unused_module

# 修复：删除未使用的导入
```

#### 行太长

```python
# 错误：超过 100 个字符
very_long_variable_name = some_function_with_many_parameters(param1, param2, param3, param4, param5)

# 修复：分行
very_long_variable_name = some_function_with_many_parameters(
    param1, param2, param3, param4, param5
)
```

#### 格式化问题

```bash
# 自动格式化所有文件
ruff format .
```

## GitHub 分支保护

为了确保代码质量，建议在 GitHub 仓库设置中配置分支保护规则：

1. 进入仓库 Settings → Branches
2. 添加或编辑 `main` 分支的保护规则
3. 启用以下选项：
   - ✅ Require a pull request before merging
   - ✅ Require status checks to pass before merging
   - ✅ Require branches to be up to date before merging
   - 在 "Status checks" 中选择 `Lint Code`

这样，所有 PR 必须通过代码检查才能合并。

