# 开发规范

## Git 提交规范

!> Git 提交的原则是`每次提交只做一件事`，不要提交两个或两个以上功能的变更。`严谨`意义不明确的描述，如 "修改了一些bug"、"调整了样式" 等。

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
- merge: 合并分支
- release: 发布新版本
- sync: 同步操作
- other: 其它

##### 作用域

作用域是可选的，用于指定 commit 影响的范围。比如 login、user、api、i18n 等。我们的原则是表达通顺的前提下尽量使用它，这样有利于阅读。

##### 概要说明

概要说明必须是英文，不能以大写字母开头，表达清晰准确，不能包含中文。

### Description 的格式

`Description` 是可选项。必须是英文，表达清晰准确，不允许中文，`允许`换行，长度没有限制。
