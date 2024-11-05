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

## GIT提交规范

### 格式

git commit 格式只能是以下两种：

- `subject`:🈳`description`
- `subject`(`scope`):🈳`description`

### subject

subject 种类以及含义如下：

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
- merge: 合并分支
- release: 发布新版本
- sync: 同步操作
- other: 其它

### scope

scope 是可选的，用于指定 commit 影响的范围。比如 login、user、api、i18n 等。我们的原则是表达通顺的情况下尽量使用它，这样有利于阅读。

### description

description 必须是英文，不能包含中文，不能以大写字母开头。

### 下面是几个例子

```
feat: add login page
feat(login): login page add forgot password
fix(login): login page forgot password reset password button not working
fix(login): the login page color is confused in dark mode
fix(currentAccount): correct the "errorCollector" import error
docs: update the project README
style(api): format the code
chore: update the project dependencies
refactor(i18n): adjust "error" to the first row
feat(timezone): supports dark mode
```

### Surge Admin 实例

下面是 Surge Admin 的提交实例截图：

![Surge Admin Commit Example](./assets/surge-admin-commit-example.png ':class=doc-image')
