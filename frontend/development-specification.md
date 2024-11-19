# 开发规范

## IDE

- 只能使用 vscode 或兼容编辑器（如 Cursor）进行开发。
- 项目的 .vscode/settings.json 文件中包含了 vscode 配置，请不要修改。
- 项目的 .vscode/extensions.json 文件中列出了插件列表，请按要求安装。

## 包管理器

- 使用 pnpm 作为包管理器。
- 项目根目录下的 pnpm-lock.yaml 文件是 pnpm 的锁文件，非项目负责人请不要修改。

## 代码规范

通过 eslint/stylelint 进行代码检查，通过 prettier 进行代码格式化，所有配置已经设置好，按 vscode 的提示编写代码即可。

## Git 配置

在首次克隆代码前，先使用以下命令配置 Git，以确保换行符一致：

```shell
git config --global core.autocrlf input
```

使用以下命令验证设置是否成功。如果输出 `input`，则说明设置成功。

```shell
git config --global --get core.autocrlf
```

## Git 提交规范

!> Git 提交的原则是`每次提交只做一件事`，不要提交两个或两个以上功能的变更。`严禁`意义不明确的描述，如 "修改了一些bug"、"调整了样式" 等。

Git 提交时可以输入 `Subject` 和 `Description` 两项描述，下面是 Fork 客户端的提交实例：

![Git Commit Example](./assets/git-commit-example.png ':class=doc-image')

下面是 Surge Admin 的提交记录截图：

![Surge Admin Commit Example](./assets/surge-admin-commit-example.png ':class=doc-image')

### Subject 的格式

`Subject` 必须是英文，表达清晰准确，不允许中文，不允许换行，不能超过100个字，格式只能是以下两种：

- `主题`:🈳`概要说明`
- `主题`(`作用域`):🈳`概要说明`

##### 主题

主题的种类以及含义如下：

- feat: 新功能
- fix: 修复问题
- perf: 性能优化
- docs: 文档更新
- style: 代码格式化（不是 css/less/scss 格式化）
- refactor: 代码重构
- test: 测试用例更新
- chore: 杂项更新
- build: 构建过程或辅助工具的变动
- ci: 持续集成相关
- revert: 回滚到某个 commit

##### 作用域

作用域是可选的，用于指定 commit 影响的范围。比如 login、user、api、i18n 等。我们的原则是表达通顺的前提下尽量使用它，这样有利于阅读。

##### 概要说明

概要说明必须是英文，不能以大写字母开头，表达清晰准确，不能包含中文。

### Description 的格式

`Description` 是可选项。必须是英文，表达清晰准确，不允许中文，`允许`换行，长度没有限制。
