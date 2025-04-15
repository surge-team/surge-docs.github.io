# Surge H5 And MiniApp

为 PC Web、Mobile Web 和 MiniApp(HLS) 应用提供的开发框架，基于Surge Framework开发：

- 技术层面：技术栈包括 Surge Framework、Vue3、TailwindCSS 3.x、flyonui 1.x 和 i18n 等
- 兼容性：支持chrome/60等低版本浏览器，Android 7、iOS 14 以上移动设备

## 分支说明

Git中存在两个分支：
- `main`：主版本，如果是纯 PC Web 应用，则使用此分支
- `earlier-version-of-android-webview`：低版本 Android WebView 兼容，如果是 Mobile Web、MiniApp(HLS) 应用，则使用此分支

这两个分支主要区别在于兼容性，如果使用了错误的分支，开发过程中也可以很容易的切换，但需要修改一些配置文件。参考 4aaa36d 和 25f1a2c 两次提交。

## 项目结构

!> 该文件结构仅供参考，最新版 Surge Admin 可能会有所不同。

- `.husky` Git 提交规范
- `.vscode` VSCode 开发规范
- `config` Vite 配置
- `library` 库文件
  - `mini` MiniSDK for JS 核心代码，包含Miniapp(HLS)接口和打包工具
  - `surge` Surge Framework 核心代码
- `src` 项目源码
  - `assets` 静态资源，包括图片、字体、全局样式表等
    - `images` 图片
    - `style` 全局样式表
  - `config` 存放App配置或默认参数
    - `routes` 路由配置
    - `appSettings.ts` App特性开关
    - `defaultSettings.ts` 用户个性化设置的默认值
  - `i18n` 国际化配置
    - `en-US` 英文配置
    - `ja-JP` 日文配置
    - `zh-CN` 中文配置
    - `index.ts` 国际化配置入口
  - `library` Hooks、状态管理、Utils、Vue指令等各类与视图无关的代码
    - `errorHandler` 错误处理工具
    - `routerManager` 路由实例及管理工具
  - `services` 存放第三方服务，如API、WebSocket、WebWorker等
    - `customMiniapp` 泛用 DataChannel API
    - `dataChannel` 标准 DataChannel API
    - `private` 本App私有API（Git代码中没有创建该文件夹，可以参考Surge Admin）
  - `views` 页面视图组件
    - `_components` 公共组件库
    - `_not-found` 404页面
    - `home` 首页
    - `...` 其他页面
- `.env.development` 开发环境变量
- `.env.production` 生产环境变量
- `.env.stg` STG环境变量
- `.env.test` 测试环境变量
- `.env.uat` UAT环境变量
- `.eslintignore` ESLint 忽略文件
- `.eslintrc.js` ESLint 配置
- `.gitignore` Git 忽略文件
- `.prettierignore` Prettier 忽略文件
- `.prettierrc.js` Prettier 配置
- `.stylelintrc.js` Stylelint 配置
- `babel.config.js` Babel 配置
- `commitlint.config.js` Commitlint 配置
- `components.d.ts` VUE 全局组件类型声明
- `index.html` 入口文件
- `package.json` 项目配置
- `pnpm-lock.yaml` pnpm 锁定文件
- `shims-tsx.d.ts` TSX 类型声明
- `tailwind.config.js` Tailwind 配置
- `tsconfig.json` TS 配置

## 路由

Surge H5 使用 Vue Router 作为路由管理器。

主要涉及以下文件：

- `src/config/routes/index.ts` 路由配置的主入口文件
- `src/config/routes/constants.ts` 路由相关常量定义，供 `src/library/routerManager` 使用
- `src/library/routerManager` 路由管理核心模块
  - `index.ts` 路由实例的主入口
  - `constants.ts` 常用路由名称的常量定义（如 LOGIN_ROUTE.name、REDIRECT_ROUTE.name），方便开发使用

## 国际化

Surge H5 采用 vue-i18n 的国际化方案，支持中、英、日三种语言，支持日期、时间、数字、货币、百分比等多种格式化方式。

更多实现细节请参考 `src/i18n` 目录中代码。

## 错误处理

Surge H5 继承了 Surge Framework 的[通用错误处理](surge-framework#通用错误处理)机制。

更多实现细节请参考 `src/library/errorHandler` 目录中代码。

## 时区处理

前后端统一采用 UNIX 时间戳进行数据传输，从根本上规避时区问题

## 页面响应式布局

Surge H5 默认采用 TailwindCSS 3.x 进行响应式布局，基于 flyonui 1.x UI库，同时支持移动端和 PC 端。

## Services

Web应用的本质是与第三方服务进行交互，它们不只局限于REST API，还包括GraphQL API、WebSocket、WebWorker、WebAssembly等等。Surge Admin 在 `src/services` 目录下实现了对这些服务的统一管理。

#### REST API

Surge H5 使用 Surge Framework 提供的 [Http 请求库](surge-framework#http-请求)来实现 REST API 调用。具体开发流程如下：

1. 在 `src/services` 目录下新建 API 服务目录。例如 `src/services/private` (private 表示本项目的 API)。根据不同的提供商这个名字也不相同，例如 `src/services/aws` 表示 AWS API。
2. 在 `src/services/private/index.ts` 文件中实例化并导出 Http 类，完成基础配置和拦截器定义。
3. 在 `src/services/private/entities/` 目录下定义 API 相关的类型声明，包括请求参数和响应数据类型。
4. 将功能相近的 API 封装到同一个对象中，并将其保存在 `src/services/private/` 目录下的独立文件中。

具体请参考 `src/services/private` 目录下的代码。

#### WebSocket Client

Surge H5 使用 Surge Framework 提供的 [WebSocket 客户端](surge-framework#websocket-客户端) 来实现与 WebSocket 服务器的通信。业务代码应该存放到 `src/services` 目录下，以 `ws` 为前缀的文件夹名称表示该文件夹下的代码是 WebSocket 相关的。
