# 贡献指南

## 为 freqtrade 做贡献

觉得我们的机器人缺少某个功能？我们欢迎您的 Pull Request！

标记为 [good first issue](https://github.com/freqtrade/freqtrade/labels/good%20first%20issue) 的问题是很好的首次贡献选择，有助于您熟悉代码库。

贡献须知：

- 请基于 `develop` 分支创建您的 PR，而不是 `stable` 分支。
- 提交信息、PR 描述、代码注释和变量名请使用英文。
- 新功能需要包含单元测试，必须通过 CI（运行 pre-commit 和 pytest 以获取早期反馈），并应在引入该功能的 PR 中附带文档说明。
- PR 可以标记为草稿（draft）——表示 Pull Request 仍在进行中。我们仍会尽量及时为草稿 PR 提供反馈。
- 如果您在 PR 中使用了 AI，请在 PR 描述中明确说明，并对生成的代码进行彻底审查。
  代码的最终责任在于 PR 作者，而非 AI，这也意味着提交必须关联到您的（个人）账户，而非某个通用的 AI 账户。

如果您不确定，请在我们的 [Discord 服务器](https://discord.gg/p7nuUNVfP7)或 [issue](https://github.com/freqtrade/freqtrade/issues) 中讨论该功能，然后再提交 Pull Request。

## 入门指南

最好先阅读[文档](https://www.freqtrade.io/)，了解机器人的功能，或者直接查看[开发者文档](https://www.freqtrade.io/en/latest/developer/)（编写中），它应该能帮助您快速上手。

## 提交 PR 之前

### 1. 运行单元测试

所有单元测试必须通过。如果某个单元测试失败了，请修改您的代码使其通过。这意味着您引入了一个回归问题。

#### 测试整个项目

```bash
pytest
```

#### 仅测试一个文件

```bash
pytest tests/test_<file_name>.py
```

#### 仅测试一个文件中的某个方法

```bash
pytest tests/test_<file_name>.py::test_<method_name>
```

### 2. 测试您的代码是否符合我们的代码风格指南

我们收到了很多无法通过 CI 初步检查的代码。
为了解决这个问题，我们鼓励贡献者安装 git pre-commit 钩子，这样在您尝试提交不符合规范的代码时会立即得到通知。

您可以使用 `pre-commit run -a` 手动运行 pre-commit，或使用 `pre-commit install` 安装 git 钩子，使其在每次提交时自动运行。

运行 `pre-commit run -a` 将运行所有检查，包括 `ruff`、`mypy` 和 `codespell`（以及其他工具）。

#### 额外的代码风格要求

- 所有公共方法都应有文档字符串（docstrings）
- 文档字符串使用双引号
- 多行文档字符串的缩进应与第一个引号对齐
- 文档字符串应遵循 reST 格式（`:param xxx: ...`、`:return: ...`、`:raises KeyError: ...`）

#### 手动运行各项检查

以下部分描述了如何手动运行作为 pre-commit 钩子一部分的各项检查。

##### 运行 ruff

使用 ruff 检查您的代码以确保其符合代码风格指南。

```bash
ruff check .
ruff format .
```

##### 运行 mypy

使用 mypy 检查您的代码以确保其符合类型注解规则。

``` bash
mypy freqtrade
```

## （核心）提交者指南

### 流程：Pull Requests

如何对 Pull Request 进行优先级排序，从最重要到最不重要：

1. 修复失败的测试。失败是指在任何支持的平台或 Python 版本上出现问题。
1. 添加额外的测试以覆盖边界情况。
1. 文档的小幅修改。
1. Bug 修复。
1. 文档的大幅修改。
1. 新功能。

确保每个 Pull Request 都满足贡献文档中的所有要求。

### 流程：Issues

如果某个 issue 是需要紧急修复的 bug，请将其标记为下一个补丁版本。然后要么修复它，要么标记为 please-help。

对于其他 issue：鼓励友好的讨论，调解辩论，分享您的想法。

### 流程：您自己的代码变更

所有代码变更，无论由谁完成，都需要由其他人审查和合并。此规则适用于所有核心提交者。

例外情况：

- 对他人提交的 Pull Request 进行的小修正和修复。
- 在正式发布期间，发布管理员可以进行必要的、适当的更改。
- 对现有内容进行强化的小型文档更改。最常见的是（但不限于）拼写和语法修正。

### 职责

- 确保每项被接受的变更都具有跨平台兼容性。Windows、Mac 和 Linux。
- 确保核心代码中不引入恶意代码。
- 为您希望进行的任何重大变更和增强功能创建 issue。透明地讨论事项并获取社区反馈。
- 保持功能 PR 尽可能小，最好每个 PR 只包含一个新功能。
- 欢迎新人，鼓励来自不同背景的新贡献者。请参阅 Python 社区行为准则 (https://www.python.org/psf/codeofconduct/)。

### 成为提交者

贡献者可能会被授予提交权限。以下条件将被优先考虑：

1. 对 Freqtrade 和其他相关开源项目的过往贡献。对 Freqtrade 的贡献包括代码（已接受和待处理的）以及在 issue 追踪器和 Pull Request 审查中的友好参与。数量和质量都会被考虑。
1. 其他核心提交者认为简洁、精简和整洁的编码风格。
1. 拥有跨平台开发和测试的资源。
1. 有时间定期投入到项目中。

出于安全原因，成为提交者并不会自动获得 `develop` 或 `stable` 分支的写入权限（用户将其交易所 API 密钥托付给 Freqtrade）。

在担任提交者一段时间后，提交者可能被任命为核心提交者，并获得完整的仓库访问权限。
